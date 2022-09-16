---
title: "Steam Deck: Setting up Emudeck"
date: "2022-09-15"
tags: ["steamdeck", "emudeck"]
# header:
#     overlay_image: ""
#     show_overlay_excerpt: false
published: true
---

I had the opportunity to borrow a friend's Steam Deck for about a month in July, but now I have my own!

## Overview

Here are the steps that I went through to installing and configuring EmuDeck for my emulation needs.

From: <https://www.emudeck.com/#how_to_install> --

> 1. If you plan to store your roms on a SD card..Format your SD Card in Steam UI. SD Cards need to be on ext4 (or btrfs) to work on EmuDeck. Then go into Desktop mode by pressing the STEAM button, Power -> Switch to Desktop
> 2. Download the Installer down below, copy that file to your Deck's Desktop and run it.
> 3. Now close Steam and run Steam Rom Manager when asked by the app.
> 4. Click on Preview, then Generate App list, wait for all the images to download and then click Save App list. The first time it could take some minutes, check on the Event Log tab to know when the process is finished.
> 5. Close Steam Rom Manager and the Installer window, click on "Return to game mode" on your desktop and you are good to go!
>
> BONUS: You can use EmuDeck with EmulationStation right out of the box, no configuration needed! After you run Steam Rom Manager you will see it on Steam in a new Emulation Collection

### Installing EmuDeck

- Boot in to desktop mode
- Install `Firefox`
    - Click on the icon in the taskbar at the bottom of the screen
- Go to <https://www.emudeck.com/#download> and click "Download app"
- Open file browser
    - Execute the `EmuDeck.desktop` file (probably in your `$HOME/Downloads` folder)
    - I selected `Easy Mode` during installation
- Ends with:
    > Remember to add your games here:
    > `/home/deck/Emulation/roms`
    >
    > And your Bios (PS1, PS2, Yuzu, Ryujinx) here:
    > `/home/deck/Emulation/bios`


### Setup `deck` user / Enable sshd

Rather than use a microSD card for moving things around, I opted to use the fact that the Steam Deck is running Linux, and simply use the built in SSH daemon.

- Open `Konsole` app
- type `passwd` and set a new password
- Turn on `sshd` with `sudo systemctl start sshd`
    - Check your ip address with `ip a`

### Installing Emulators / Files

Sync bios, roms, and saves files from main computer.

```bash
cd $PATH_TO_ROMS  # /mnt/c/Documents and Settings/Ian Lee/OneDrive/Documents/Emulation/
rsync -av ./bios/ deck@192.168.1.50:Emulation/bios/
rsync -av ./roms/ deck@192.168.1.50:Emulation/roms/
rsync -av ./saves/ deck@192.168.1.50:Emulation/saves/
```

### Configure Emulators

- In the taskbar, right click the Steam icon and `Exit Steam`
- Open `Steam ROM Manager`
- Enable / Disable any parsers (emulators) you don't want down the left sidebar
- Go to `Preview`
    - Click `Generate app list`
    - Adjust any game art you want
    - Click `Save app list`
- Close installer window (if not already)
- Execute `Return to Gaming Mode` from the desktop
