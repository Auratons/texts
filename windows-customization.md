# Customizing Windows 10

## Disable system beep

Go to **Control Panel** -> **Sound** -> **Sounds**, within **Program Events**
search for **Default Beep** in **Windows** section. Set its **Sounds** (located
in the bottom) to **None** and you are done. Taken from
[TheWindowsClub](https://www.thewindowsclub.com/disable-system-beep-windows-7-8).



## Get rid of pre-login animation on a key press
Thanks to
[superuser](https://superuser.com/questions/979239/is-there-a-work-around-for-the-extra-key-press-and-delay-at-login-screen/979242),
you need to: 

  1. Type **WinKey + R**, enter `gpedit.msc` & press Enter.
  2. Navigate to **Computer Configuration** -> **Administrative Templates** ->
     **Control Panel** -> **Personalization** -> **Do not display Lock Screen**,
     right-click and go to **Properties**.
  3. Set the value to **Enabled**.



## Automatically mount a file system with fstab in WSL

If you use a Veracrypt-encrypted drive that is automatically mounted upon login
to your Windows account, you may notice that it is not automatically mounted
into `/mnt/<drive_letter>` as C: is. Today, one can use the `/etc/fstab`
configuration that is processed by Windows each time you start an Ubuntu window.
(If it does not do that, take a look at [`wsl` utility
documentation](https://docs.microsoft.com/en-us/windows/wsl/wsl-config#set-wsl-launch-settings)
that is ultimately used by Windows to run the WSL.) The subsystem uses `drvfs`
file system to ensure Windows-Linux interoperability, see
[this](https://docs.microsoft.com/en-us/archive/blogs/wsl/wsl-file-system-support)
blog post. The initial issue that requested such a feature is to be found
[here](https://github.com/microsoft/WSL/issues/2636).

In my case, when having a dual boot with a shared encrypted drive that is
automatically mounted on both OSes, on Linux with the combination of `fstab` and
`crypttab`, whereas on Windows it is mounted upon login by Veracrypt (see [my
other text](https://github.com/Auratons/texts/blob/master/dual-booting.md)), I
want to reuse as many mount options from proper Ubuntu's fstab as I can. The
drvfs file system after some testing with `sudo mount -t drvfs -o ...` supports
the following WSL `fstab` line:

    S:   /mnt/s  drvfs   defaults,metadata,noatime,uid=1000,gid=1000,dmask=027    0   0

## Git running on a Windows-Linux shared NTFS filesystem
https://git-scm.com/docs/git-config
https://github.com/microsoft/WSL/issues/184
https://github.com/hangxingliu/wslgit
https://medium.com/faun/how-to-use-git-and-other-linux-tools-in-wsl-on-windows-4c0bffb68b35

crlf nema cenu, ale 
 git config --system core.filemode false
je fakt potreba

https://medium.com/@securitystreak/veracrypt-full-disk-drive-encryption-fde-157eacbf0b61
https://wiki.gentoo.org/wiki/GRUB2/Chainloading
https://github.com/octetz/arch-windows-encrypted-uefi-install
