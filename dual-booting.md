# How to setup fully encrypted Windows-Linux dual-boot machine with NVIDIA GPU, UEFI, GPT, and Secure Boot enabled

The ultimate goal is to have a 'fully', meaning all partitions, the more towards
bootloaders, the better, encrypted machine with two operating systems &ndash;
Windows 10 Pro and Ubuntu 19.10 sharing an encrypted disk partition that is
automatically mounted upon boot or login. The icing on the cake for this setup
includes redirecting user folders such as Documents, Downloads, etc.
transparently to the so-called equivalents on the shared partition.

## Discussion

Starting from pressing Power Button, we encounter UEFI with Secure Boot.
Luckily, nowadays most of the most common Linux distributions support both
technologies. Further, UEFI as a modern successor of BIOS expects GPT
partitioning. Likewise, Linux has no problem with this.

Talking about 1 TB disk partitioning, we would like to have the following
partitions:

1. EFI System Partition (ESP) for Windows Bootloader and GRUB2. Since we are
   going to take inspiration from [official Ubuntu guide][ubuntu-guide], we are
   going to use sgdisk utility for the actual partitioning. In its
   documentation, we are advised to reserve **550 MB** for this partition.

2. BIOS-mode GRUB core image partition, according to [official Ubuntu
   guide][ubuntu-guide]:

    > A small bios_boot (**2 MB**) partition for BIOS-mode GRUB's core image.

    I'm not sure if it gets used since we do not support BIOS mode, after all,
    see the next chapters.

3. The /boot partition for Linux kernels. The smaller, the more often a user has
   to clean old kernels, so let's have **2 GB** for it.

    - Microsoft Reserved Partition (**16 MB**) that **gets added automatically**
      within Windows 10 installation. Its position in the partition table is
      caused by the fact that the first three partitions are placed right next
      after each other.

4. Encrypted LVM with root, home, and swap logical volumes. I believe that with
   the shared partition, **180 GB** will be enough here. I won't need
   hibernation, so the swap is only for sake of having it, **4 GB** is way more
   than enough since I have 32 GB RAM. The rest is split equally between the
   root and home with LVM-soft boundary between. Some software can reside in a
   user's home directory (plus configurations, fonts, etc.), some in the root
   file system. That is the reason I reserve so much for the home. Separating
   home from the root is a classic way how to preserve data stored there across
   different Linux distribution installations. The Intel's Clear Linux project
   is the only OS that I would possibly like to try out and that is not yet able
   to have the root file system on an LVM volume.

5. Windows system partition with ca. **240 GB** (mainly for a few games there
   hh).

    - Microsoft Windows Recovery Environment (**650 MB**) that **gets added
      automatically** after Windows 10 installation using Windows 10 partition's
      space (it can probably be deleted).

6. Shared NTFS formatted partition.

Also, according to [sgdisk team's
advice](http://www.rodsbooks.com/gdisk/advice.html), **128 MB** of unallocated
space should be left between partitions bigger than 2 GB.

UEFI specification does not require ESP to be the first partition on a disk, the
same for the /boot. Windows *recommends* that, as well as placing the shared
partition and Windows being the last and the last but one partition,
respectively.

According to
[HowToGeek](https://www.howtogeek.com/howto/35807/how-to-harmonize-your-dual-boot-setup-for-windows-and-ubuntu/),
you cannot have an NTFS formatted partition mounted as /home because of
different concepts of access rights. That is the reason why we use ext4
formatted /home on the LVM partition in addition to the shared file system. We
can then change locations of user folders via .config/user-dirs.dirs on Linux
and via Location property on Windows.

Taking encryption into account, LUKS is the right way for Linux-based systems.
As usual, Windows does not support the technology, so we must use something else
for the shared partition. Veracrypt can do the job as it is multi-platform,
open-source, and to some extent also independently audited software. A question
of the R/W speeds affection arises, although since it is going to be used only
for shared files, it should be OK. For Windows *system*, we can choose to use
Veracrypt as well. However, because of R/W speed dropout and the fact that
Veracrypt bootloader does not go well Secure Boot, we should use Microsoft's
proprietary BitLocker that is pronounced to be considerably faster and works
with Secure Boot.

## Steps

The primary source for this manual is [official Ubuntu guide][ubuntu-guide].
These steps include some shortcomings, but I didn't repeat the process so that I
could find the most optimal solution.

It is claimed that installing Windows first is better because GRUB then detects
Windows and takes care of dual-booting correctly.

1. Create the Live USB for both operating systems.

2. Boot to Linux ('Try Ubuntu without installing'), but be sure you boot in UEFI
   mode. The option for that can differ from machine to machine, mine has the
   option to disable Legacy Boot.

    You can simply check it &ndash; once Linux starts, presence of the efivarfs
    file system means the system booted in UEFI mode:

        $ mount | grep efivars
        efivarfs on /sys/firmware/efi/efivars type efivarfs (rw,nosuid,nodev,noexec,relatime)

    Another sign is booting via GRUB, not via Syslinux that uses a screen with
    options and a big Ubuntu logo above them.

    If the machine supports NVIDIA Optimus, it may happen that after a few
    minutes of running Live USB the system freezes and nothing but restart
    helps. In such a case, press 'e' on GRUB's boot screen before choosing 'Try
    Ubuntu without installing' and edit line starting with `linux` changing
    `quiet squash` to `nomodeset`. Then boot from the updated option with F10.
    See
    [1](https://help.ubuntu.com/community/Grub2/Troubleshooting#Editing_the_GRUB_2_Menu_During_Boot),
    [2](https://wiki.ubuntu.cz/grub2),
    [3](https://ubuntuforums.org/showthread.php?t=2230389&p=13370808#post13370808)
    (search for 'nomodeset').

3. We will create GPT partitioning on the drive. Below you can find the result
   after partitioning, but before installing operating systems. I will refer to
   the result in the commands' comments.

    |Number | Start (sector) | End (sector) | Size      | Code | Name       |
    |:-----:|---------------:|-------------:|----------:|:----:|:-----------|
    |   1   | 2048           | 1128447      | 550.0 MiB | EF00 | EFI-SP     |
    |   2   | 1128448        | 1132543      | 2.0 MiB   | EF02 | GRUB-BIOS  |
    |   3   | 1132544        | 5228543      | 2.0 GiB   | 8301 | /boot      |
    |   4   | 5492736        | 159092735    | 175.8 GiB | 8301 | linuxlvm   |
    |   5   | 379430912      | 883238911    | 242.2 GiB | 0700 | windows    |
    |   6   | 883503104      | 1953260942   | 510.6 GiB | 0700 | sharedfs   |

    Please note that Start and End sectors do not necessarily follow one right
    after each other since there are those unallocated blocks. Commands:

        export DEV="/dev/nvme0n1"
        export DM="${DEV##*/}"

        sgdisk --print $DEV
        sgdick --list-types # See type codes.
        sgdisk --zap-all $DEV

        sgdisk --new=0:0:+550M $DEV # EFI-SP
        sgdisk --new=0:0:+2M $DEV # GRUB-BIOS
        sgdisk --new=0:0:+2000M $DEV # /boot
        sgdisk --new=0:+128M:+180000M $DEV # linuxlvm
        sgdisk --new=0:+128M:+248000M $DEV # windows
        sgdisk --new=0:+128M:-128M $DEV # shardefs

        sgdisk -t=1:ef00 $DEV # EFI System
        sgdisk -t=2:ef02 $DEV # BIOS boot partition
        sgdisk -t=3:8301 $DEV # BIOS boot partition
        sgdisk -t=4:8301 $DEV # BIOS boot partition
        sgdisk -t=5:0700 $DEV # Microsoft basic data
        sgdisk -t=6:0700 $DEV # Microsoft basic data

        sgdisk --change-name=1:EFI-SP $DEV
        sgdisk --change-name=2:GRUB-BIOS $DEV
        sgdisk --change-name=3:/boot $DEV
        sgdisk --change-name=4:linuxlvm $DEV
        sgdisk --change-name=5:windows $DEV
        sgdisk --change-name=6:sharedfs $DEV

    The [official Ubuntu guide][ubuntu-guide] proposes creation of hybrid MBR
    within GPT tables as support for BIOS-mode booting. I find this worthless.
    Windows installer then perceives the disk as MBR and do not want to install
    onto it. Also, the most common Linux distributions can be installed on GPT,
    making MBR support via `sgdisk --hybrid 1:2:3 $DEV` useless.

4. Install Windows in their intended partition.

    In my setting, removing Live USB destructed LUKS and LVM stuff. Installing
    Windows added Microsoft Reserved Partition, thus shifting all partition
    numbers after 3 by one. So I needed to re-do the following commands with
    shifted numbering after Windows installation.

5. Create encrypted Linux partitions. Boot into Live USB, possibly again
   modifying `quiet splash` to `nomodeset`. We use luks1 format because GRUB2 at
   the moment does not support luks2 type for /boot encryption.

        cryptsetup --key-size 512 --hash sha512 luksFormat --type=luks1 ${DEV}p3 # boot, create strong passphrase
        cryptsetup --key-size 512 --hash sha512 luksFormat --type=luks1 ${DEV}p5 # linuxlvm, create strong passphrase

        cryptsetup open ${DEV}p3 LUKS_BOOT
        cryptsetup open ${DEV}p5 LVM_crypt

        mkfs.ext4 -L boot /dev/mapper/LUKS_BOOT

    Unlike [official Ubuntu guide][ubuntu-guide], we do not need to create FAT
    file system in EFI-SP partition since it is already formatted by Windows.

6. Create LVM volumes with 4 % of free space from (175.8 - 4) GiB:

        pvcreate /dev/mapper/LVM_crypt
        vgcreate vg00-ubuntu /dev/mapper/LVM_crypt
        lvcreate -L 4G -n swap vg00-ubuntu
        lvcreate -l 48%FREE -n root vg00-ubuntu
        lvcreate -l 48%FREE -n home vg00-ubuntu

7. Install Ubuntu from the running Live USB. Do not restart the machine, just
   use the installer on desktop. Otherwise, you would lose prepared LUKS and LVM
   volumes. Choose 'Something else' and mount the correct file system points to
   the right volumes and partitions. Namely /boot, /, /home, swap.

    After running Install and filling up 'Who are you' page, again open Terminal
    and run:

        echo "GRUB_ENABLE_CRYPTODISK=y" >> /target/etc/default/grub

    *If you don't make it before the installer gets to working on the Install
    Bootloader stage, the installation fails and you have to repeat it. It is
    better to prepare command to a file on another USB and use it for
    copy-paste.*

8. Set automatic mounting of LVM partition when booting. Being still in the
   running Live USB system, we can run the following (another and not tested way
   is to create keyfile via Veracrypt tool and do the following steps from the
   installed Ubuntu afterward):

        # mount /dev/mapper/ubuntu--vg-root /target
        # for n in proc sys dev etc/resolv.conf; do mount --rbind /$n /target/$n; done 
        # chroot /target
        # mount -a
        # apt install -y cryptsetup-initramfs

    We are forced to use initramfs because we use encrypted LVM volume for the
    root file system.

        # echo "KEYFILE_PATTERN=/etc/luks/*.keyfile" >> /etc/cryptsetup-initramfs/conf-hook 
        # echo "UMASK=0077" >> /etc/initramfs-tools/initramfs.conf 

        # mkdir /etc/luks
        # dd if=/dev/urandom of=/etc/luks/boot_os.keyfile bs=4096 count=1
        # chmod u=rx,go-rwx /etc/luks
        # chmod u=r,go-rwx /etc/luks/boot_os.keyfile

        # cryptsetup luksAddKey ${DEV}p1 /etc/luks/boot_os.keyfile 
        # cryptsetup luksAddKey ${DEV}p5 /etc/luks/boot_os.keyfile

        # echo "LUKS_BOOT UUID=$(blkid -s UUID -o value ${DEV}p1) /etc/luks/boot_os.keyfile luks,discard" >> /etc/crypttab
        # echo "${DM}5_crypt UUID=$(blkid -s UUID -o value ${DEV}p5) /etc/luks/boot_os.keyfile luks,discard" >> /etc/crypttab

        # update-initramfs -u -k all

After these steps, we have a dual boot with encrypted GRUB and Ubuntu installed.
Windows and the shared partition are still unencrypted.

We also end up having Microsoft Reserved Partition which gets created right
after /boot partition in 128 MB unallocated space. I'm not sure how it would
work in case of not having that space. Maybe it would just take space from
Windows partition as Microsoft Windows Recovery Environment does.

9. Change the boot order in UEFI. In my case, it turned out that Windows cannot
   encrypt their partition using BitLocker because of Secure Boot if not booted
   by Windows Bootloader.

10. Encrypt Windows 10 with BitLocker. Surprisingly enough, encrypted Windows in
    this setting cannot be booted from GRUB's options list. It is easier to
    change the order of bootloaders in UEFI and either to use F10, as in my
    case, during boot to choose a bootable device. Alternatively, one can use
    GRUB only for booting Ubuntu and Windows Bootloader with Ubuntu boot option
    added as the primary bootloader.

11. Encrypt and automatically mount the shared partition. Install Veracrypt
    either on Windows (my case) or Ubuntu and encrypt the partition only with
    randomly generated keyfiles as NTFS. Keep PIM's default settings because
    automatic mounting on Linux does not support a user-defined value yet. Do
    not also forget to appropriately set keyfiles' access rights in both
    operating systems.

    For on-login automatic mounting on Windows, you must set default keyfiles,
    trying an empty password automatically and unsurprisingly to automatically
    mount given partition on login.

    For boot-time mounting on Ubuntu, we must add a line per /etc/crypttab and
    /etc/fstab files. Since we are not manipulating with the root file system
    here, we do not need to update initramfs.

    `/etc/crypttab` (/dev/null for no passphrase):

        SHAREDFS /dev/disk/by-id/nvme-Samsung_SSD_970_EVO_1TB-part8 /dev/null tcrypt-veracrypt,tcrypt-keyfile=<path_to_file1>,tcrypt-keyfile=<path_to_file2>, ...

    `/etc/fstab` (pass is 0 because fsck is useless for NTFS):

        /dev/mapper/SHAREDFS /mnt/sharedfs ntfs-3g defaults,nls=utf8,uid=1000,gid=1000,dmask=027,windows_names 0 0

    On Windows, in Properties of user folders such as Downloads, Documents,
    Pictures, ... go to Location and change it to the shared volume's
    counterpart. It is better to rename them also to lowercase for consistency.

    In Ubuntu, rename folders to lowercase for consistency, remap them to
    /mnt/sharedfs in `~/.config/user-dirs.dirs`, remove all old user folders and
    make symlinks to /mnt/sharedfs, then run `xdg-user-dirs-update` and
    `xdg-user-dirs-gtk-update`. You might also want to specifically change these
    paths in some file explorers, such as Nemo on Ubuntu GNOME.

    Note: If you are using snaps more extensively, you are going to have
    problems with snaps accessing data in /mnt/sharedfs because of their
    inherent isolation. You may solve this by installing those specific snaps
    with --devmode option that disables the security measures. Otherwise, you
    can mount the shared file system to your home directory in /etc/fstab. That
    is an even better solution from the security point of view. (Visual Studio
    Code's snap 'code' does not suffer from this behavior since it asks for
    higher privileges from the very beginning of the installation.)

## Conclusion & Future Work

Now we have the desired dual boot system with two flaws:

- When we want to boot to Ubuntu, we need to press F10 in the early boot stage
  and we have in GRUB's table unnecessary option for Windows that does not work.
  This can be solved by removing Windows option from the table and setting table
  on-screen time to 0 and by adding Linux option to UEFI's primary Windows
  Bootloader.

- Also, the booting process takes ages on Ubuntu because of decryption. Windows
  gets to login screen much faster. That could be possibly solved by removing or
  changing the order of LVM's passphrase and keyfile location. Another option is
  to check that all encryption libraries are loaded during boot time to speed
  decryption up. There may be missing some, after boot the encryption is not
  noticeable.

Next possible hardening could include using motherboard's TPM chip even for
Linux GRUB's passphrase storing as in the case of BitLocker, but that is a more
complicated process that needs to be repeated with each new kernel version.

[ubuntu-guide]:
https://help.ubuntu.com/community/Full_Disk_Encryption_Howto_2019
