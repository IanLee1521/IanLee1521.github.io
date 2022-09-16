---
title: "Steam Deck: Setting up Emudeck"
date: "2022-09-15"
tags: ["steamdeck", "emudeck"]
# header:
#     overlay_image: ""
#     show_overlay_excerpt: false
published: false
---

I had the opportunity to borrow a friend's Steam Deck for about a month in July, but
now I have my own!

## Overview

Here are the steps that I went through to installing and configuring EmuDeck for my
emulation needs.

### Installing EmuDeck

- Boot in to desktop mode
- Install Firefox
- Google for `emudeck`
    - end up at <https://emudeck.com/>
- Open file browser
    - execute the `EmuDeck.desktop` file
    - I selected `Easy Mode` during installation
- Ends with:

    > Remember to add your games here:
    > `/home/deck/Emulation/roms`
    >
    > And your Bios (PS1, PS2, Yuzu, Ryujinx) here:
    > `/home/deck/Emulation/bios`
- Open ROM manager


### Set `deck` user password

- Open `Konsole` app
- type `passwd` and set a new password

### Installing Emulators / Files

- Turn on sshd with `sudo systemctl start sshd`
    - Check your ip address with `ip a`
- Sync roms and bios files from main computer

    ```bash
    cd $PATH_TO_ROMS  # "/mnt/c/Documents and Settings/Ian Lee/OneDrive/Documents/Emulation/"
    rsync -av ./bios/ deck@192.168.1.50:Emulation/bios/
    rsync -av ./roms/ deck@192.168.1.50:Emulation/roms/
    rsync -av ./saves/ deck@192.168.1.50:Emulation/saves/
    ```

### Configure Emulators

- Exit Steam
- Open `Steam ROM Manager`
- Enable / Disable any parsers (emulators) you don't want down the left sidebar
- Go to "Preview"
    - Click "Generate app list"
    - Adjust any game art you want
    - Click "Save app list"
- Close installer window (if not already)
- Execute "Return to Gaming Mode" from the desktop

From: <https://www.emudeck.com/#how_to_install> --

> 1. If you plan to store your roms on a SD card..Format your SD Card in Steam UI. SD Cards need to be on ext4 (or btrfs) to work on EmuDeck. Then go into Desktop mode by pressing the STEAM button, Power -> Switch to Desktop
> 2. Download the Installer down below, copy that file to your Deck's Desktop and run it.
> 3. Now close Steam and run Steam Rom Manager when asked by the app.
> 4. Click on Preview, then Generate App list, wait for all the images to download and then click Save App list. The first time it could take some minutes, check on the Event Log tab to know when the process is finished.
> 5. Close Steam Rom Manager and the Installer window, click on "Return to game mode" on your desktop and you are good to go!
>
> BONUS: You can use EmuDeck with EmulationStation right out of the box, no configuration needed! After you run Steam Rom Manager you will see it on Steam in a new Emulation Collection
