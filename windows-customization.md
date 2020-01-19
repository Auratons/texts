# Disable system beep

Go to **Control Panel** -> **Sound** -> **Sounds**, within **Program Events**
search for **Default Beep** in **Windows** section. Set its **Sounds** (located
in the bottom) to **None** and you are done. Taken from
[TheWindowsClub](https://www.thewindowsclub.com/disable-system-beep-windows-7-8).



# Get rid of pre-login animation on a key press
Thanks to
[superuser](https://superuser.com/questions/979239/is-there-a-work-around-for-the-extra-key-press-and-delay-at-login-screen/979242),
you need to: 

  1. Type **WinKey + R**, enter `gpedit.msc` & press Enter.
  2. Navigate to **Computer Configuration** -> **Administrative Templates** ->
     **Control Panel** -> **Personalization** -> **Do not display Lock Screen**,
     right-click and go to **Properties**.
  3. Set the value to **Enabled**.



# Use Hack Nerd Font in WSL and/or WT

Firstly, to be able to set the Windows Subsystem for Linux (WSL) and/or Windows
Terminal (WT) up correctly, install [the Hack Regular Nerd Font Complete Windows
Compatible
font](https://github.com/ryanoasis/nerd-fonts/blob/master/patched-fonts/Hack/Regular/complete/)
as Administrator.

In my case, the font was installed as 'Hack NF', one can find it in **Font
settings**. The font also has a second, longer, name, but WSL nor WT registers
the longer name.

For WT, the process is pretty straightforward. Just open **Settings** (Ctrl + ,)
and add `"fontFace": "Hack NF"` to the Ubuntu profile. You can also add
`fontSize` key with an integer value. Finally, you can set `defaultProfile` to
Ubuntu's `guid`.

For WSL, the process is more tricky since it incorporates using **Registry
Editor**:

  1. Type **WinKey + R**, enter `regedit` & press Enter.
  2. Navigate to **Computer** -> **HKEY_CURRENT_USER** -> **Console** ->
     **C:_Program
     Files_WindowsApps_CanonicalGroupLimited.UbuntuonWindows_1804.2019.521.0_x64__79rhkp1fndgsc_ubuntu.exe**.
  3. Change **FaceName** to `Hack NF` and **FontSize** to decimal size, if
     wanted.

Unfortunately, this workaround means that you are not able to change anything
related to fonts in WSL's settings. Such a change would immediately remove
user-set font in the Registry resulting in necessity of re-doing the steps
above. Inspiration for this setup was taken from [WSL's
Github](https://github.com/Microsoft/WSL/issues/757).

Why bothering with WSL when having WT? In my case, WT is extremely slow out of
the box. WSL as well, but not as much as WT.



# Automatically mount a file system with fstab in WSL

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

    S:   /mnt/s  drvfs   defaults,noatime,uid=1000,gid=1000,dmask=027    0   0
