# How to have terminal experience consistent across OSes

After many experiments with different terminal emulators and settings
in Linux, Windows, and macOS, the following emulators proved to be able
to function in a consistent setting:
  - Windows: WSL + Windows Terminal,
  - Linux: Konsole,
  - macOS: iTerm2.

In my setting, all programs above use monospace Hack Nerd Font together with a
certain material color theme. I use the Z shell with Oh My Zsh framework,
Powerlevel10k theme flavored by custom dotfiles, all running in a Tmux session.

The combination of terminal emulators and their visual appearance came up from
the need for a clean and functional look that spans over operating systems. It
took a lot of experimentation with applications and their settings so that they
can display all the colors and glyphs.

Tools running inside of these terminal emulators follow this desire as well. I was not able to set everything up with other than a monospace font. Z shell is just great, together
with Oh My Zsh they are a killer combination for faster, more convenient, and
effective work. The material color scheme just looks amazing and it is simple to read.
I use the one taken from [here](https://medium.com/@rafavinnce/iterm2-zsh-oh-my-zsh-material-design-the-most-power-full-terminal-on-macos-332b1ee364a5).
(Where it is possible, I use terminal.sexy to translate config file formats between
tools further in this text.) The visual appearance is wrapped by Powerlevel10k zsh
theme that is a successor of popular Powerlevel9k, I found that on Windows, Powerlevel9k
makes the whole command line extremely slow. Powerlevel10k reimplements the theme so that
the command line reacts instantaneously again. The theme uses Powerline glyphs to provide useful
information in concise visual language. For that, [Hack](https://github.com/ryanoasis/nerd-fonts/tree/master/patched-fonts/Hack)
font, a part of [Nerd Fonts](https://github.com/ryanoasis/nerd-fonts) is used.
It is monospace, patched (easier to use and set up than fallback strategy with `fontconfig`)
font and it contains all necessary Powerline glyphs used by Powerlevel10k.

## Linux

Install Konsole, it looks great even in classic Ubuntu with small tweaking.
Use Shift+Ctrl+M to "Show/Hide Menu Bar", that does not go well with Ubuntu
visual theme. Further, I use customized tabs look in Settings -> Configure
Konsole -> TabBar. User-defined stylesheet is:

    ~/.local/share/konsole/dark-tabs.css:

    /* Based on codemedic's work (https://gist.github.com/codemedic/f11cc460b8d9544f9afc) */
    QWidget, QTabWidget::pane, QTabWidget::tab-bar {
        background-color: #383c4a;
    }
    QTabBar::tab {
        color: #777; 
        background-color: #383c4a;
        font-size: 11px;
        height: 16px;
        padding: 2px;
        border: 0px;
        border-bottom: 3px solid
    }
    QTabBar::tab:selected, QTabBar::tab:hover {
        color: #d3dae3;
        background-color: palette(window);
        border: 0px;
        border-bottom-right-radius: 5px;
        border-bottom-left-radius: 5px;
        font-weight: bold;
        background: qlineargradient(x1: 0, y1: 1, x2: 0, y2: 0, stop: 0 #d3dae3, stop: 0.001 #d3dae3, stop: 0.07 #383c4a);
    }

#### Font & Color Scheme

Download Hack Nerd Fonts from Github and store them whenever under
`~/.local/share/fonts/`. Then, run `fc-cache` to update user-wide fonts.
One can verify the successful installation with `fc-list | grep Hack`.

Color scheme file for Konsole:

    ~/.local/share/konsole/material.colorscheme:
    
    # --- special colors ---
    [Background]
    Color=29,38,42
    [BackgroundIntense]
    Color=29,38,42
    [Foreground]
    Color=231,235,237
    [ForegroundIntense]
    Color=231,235,237
    Bold=true

    # --- standard colors ---
    [Color0]
    Color=67,91,103
    [Color0Intense]
    Color=161,176,184

    [Color1]
    Color=252,56,65
    [Color1Intense]
    Color=252,116,109

    [Color2]
    Color=92,241,158
    [Color2Intense]
    Color=173,247,190

    [Color3]
    Color=254,208,50
    [Color3Intense]
    Color=254,225,108

    [Color4]
    Color=55,182,255
    [Color4Intense]
    Color=112,207,255

    [Color5]
    Color=252,34,110
    [Color5Intense]
    Color=252,102,155

    [Color6]
    Color=89,255,209
    [Color6Intense]
    Color=154,255,230

    [Color7]
    Color=255,255,255
    [Color7Intense]
    Color=255,255,255

    # --- general options ---
    [General]
    Description=terminal.sexy
    Opacity=1
    Wallpaper=

Finally, the whole Konsole theme file:

    ~/.local/share/konsole/Material\ Powerlevel10k.profile

    [Appearance]
    ColorScheme=material
    Font=Hack Nerd Font,10,-1,5,50,0,0,0,0,0,Regular

    [General]
    Name=Material Powerlevel10k
    Parent=FALLBACK/

## Windows

Install Windows Subsystem for Linux (WSL) and/or Windows
Terminal (WT). Windows Pro or Education is necessary for
Ubuntu in Windows as it needs virtualization support enabled.

#### Font & Color Scheme

Install [the Hack Regular Nerd Font Complete Windows
Compatible
font](https://github.com/ryanoasis/nerd-fonts/blob/master/patched-fonts/Hack/Regular/complete/)
as Administrator.

In my case, the font was installed as 'Hack NF', one can find it in **Font
settings**. The font also has a second, longer, name, but WSL nor WT registers
the longer name.

For WT, the process is pretty straightforward. Just open **Settings** (Ctrl + ,)
and add `"fontFace": "Hack NF"` to the Ubuntu profile. One can also add
`fontSize` key with an integer value. Set `defaultProfile` to Ubuntu's `guid`.
Finally, material color scheme here is:

    {
      // https://raw.githubusercontent.com/MartinSeeler/iterm2-material-design/master/material-design-colors.itermcolors
      // terminal.sexy
      "name": "material-colors",
      "foreground": "#e7ebed",
      "background": "#1d262a",
      "black": "#435b67",
      "red": "#fc3841",
      "green": "#5cf19e",
      "yellow": "#fed032",
      "blue": "#37b6ff",
      "purple": "#fc226e",
      "cyan": "#59ffd1",
      "white": "#ffffff",
      "brightBlack": "#a1b0b8",
      "brightRed": "#fc746d",
      "brightGreen": "#adf7be",
      "brightYellow": "#fee16c",
      "brightBlue": "#70cfff",
      "brightPurple": "#fc669b",
      "brightCyan": "#9affe6",
      "brightWhite": "#ffffff"
    }

For WSL, the process is more tricky since it incorporates using **Registry
Editor**:

  1. Type **WinKey + R**, enter `regedit` & press Enter.
  2. Navigate to **Computer** -> **HKEY_CURRENT_USER** -> **Console** ->
     **C:_Program
     Files_WindowsApps_CanonicalGroupLimited.UbuntuonWindows_1804.2019.521.0_x64__79rhkp1fndgsc_ubuntu.exe**.

     > Because of an amazing Microsoft update culture, this path to WSL settings changes with every bigger update. So one probably needs to search for Ubuntu settings
     each time WSL stops looking nice and neat.
  3. Change **FaceName** to `Hack NF` and **FontSize** to decimal size, if
     wanted.

One can automate setting the right values, which is manually pretty tedious, by using the
following regedit script (**!!with the correct path to Ubuntu settings!!**):

    Windows Registry Editor Version 5.00
    ; Material color scheme
    ; for Windows command prompt.

    ; Values stored as 00-BB-GG-RR
    ; Be aware of THAT, https://terminal.sexy uses RGB order.
    [HKEY_CURRENT_USER\Console\C:_Program Files_WindowsApps_CanonicalGroupLimited.UbuntuonWindows_1804.2019.521.0_x64__79rhkp1fndgsc_ubuntu.exe]
    ; BLACK DGRAY
    "ColorTable00"=dword:00675b43
    "ColorTable08"=dword:00b8b0a1
    ; BLUE LBLUE
    "ColorTable01"=dword:00ffb637
    "ColorTable09"=dword:00ffcf70
    ; GREEN LGREEN
    "ColorTable02"=dword:009ef15c
    "ColorTable10"=dword:00bef7ad
    ; CYAN LCYAN
    "ColorTable03"=dword:00d1ff59
    "ColorTable11"=dword:00e6ff9a
    ; RED LRED
    "ColorTable04"=dword:004138fc
    "ColorTable12"=dword:006d74fc
    ; MAGENTA LMAGENTA
    "ColorTable05"=dword:006e22fc
    "ColorTable13"=dword:009b66fc
    ; YELLOW LYELLOW
    "ColorTable06"=dword:0032d0fe
    "ColorTable14"=dword:006ce1fe
    ; LGRAY WHITE
    "ColorTable07"=dword:00ffffff
    "ColorTable15"=dword:00ffffff
    ; BG FG
    "DefaultBackground"=dword:002a261d
    "DefaultForeground"=dword:00edebe7

Unfortunately, this workaround means that one is not able to change anything
related to fonts in WSL's settings. Such a change would immediately remove
the user-set font in the Registry resulting in the necessity of re-doing the steps
above. Inspiration for this setup was taken from [WSL's
Github](https://github.com/Microsoft/WSL/issues/757).

For Visual Studio Code, use:

    "workbench.colorCustomizations": {
        "terminalCursor.background":"#A89984",
        "terminalCursor.foreground":"#A89984",
        "terminal.foreground": "#e7ebed",
        "terminal.background": "#1d262a",
        "terminal.ansiBlack": "#435b67",
        "terminal.ansiRed": "#fc3841",
        "terminal.ansiGreen": "#5cf19e",
        "terminal.ansiYellow": "#fed032",
        "terminal.ansiBlue": "#37b6ff",
        "terminal.ansiMagenta": "#fc226e",
        "terminal.ansiCyan": "#59ffd1",
        "terminal.ansiWhite": "#ffffff",
        "terminal.ansiBrightBlack": "#a1b0b8",
        "terminal.ansiBrightRed": "#fc746d",
        "terminal.ansiBrightGreen": "#adf7be",
        "terminal.ansiBrightYellow": "#fee16c",
        "terminal.ansiBrightBlue": "#70cfff",
        "terminal.ansiBrightMagenta": "#fc669b",
        "terminal.ansiBrightCyan": "#9affe6",
        "terminal.ansiBrightWhite": "#ffffff"
      }

## macOS

Install [iTerm2](https://www.code2bits.com/how-to-install-iterm2-on-macos-using-homebrew/)
and [Hack Nerd Font](https://github.com/ryanoasis/nerd-fonts#option-4-homebrew-fonts)
with brew package manager (best option). According to aforementioned
[text](https://medium.com/@rafavinnce/iterm2-zsh-oh-my-zsh-material-design-the-most-power-full-terminal-on-macos-332b1ee364a5),
do:
1. Open terminal and paste.
    $ cd Downloads
    $ curl -O https://raw.githubusercontent.com/MartinSeeler/iterm2-material-design/master/material-design-colors.itermcolors
2. Open iTerm2, go to iTerm2 -> Preferences -> Profiles -> Colors Tab.
3. Click Color Presets… at the bottom right.
4. Click Import…
5. Select material-design-colors.itermcolors file.
6. Select the material-design-colors from Load Presets…
7. Go to iTerm2 > Preferences > Profiles > Text Tab.
8. Use Hack Nerd Font in the drop-down list.
