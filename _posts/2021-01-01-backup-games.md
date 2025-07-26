---
title: "Backing Up Games to OneDrive"
date: "2021-01-01"
tags: []
# header:
#     overlay_image: ""
#     show_overlay_excerpt: false
published: true
---

Standards are hard... I get it. But it would be really nice if I could actually get all my games into the same place (ideally `$Env:USERPROFILE/OneDrive/Documents/My Games` on Windows). That way that they are included in my automatic OneDrive Backup of my `$Env:USERPROFILE/OneDrive/Documents` directory.

Enter Windows [Junctions](https://docs.microsoft.com/en-us/sysinternals/downloads/junction).

## Background

Windows [Junctions](https://docs.microsoft.com/en-us/sysinternals/downloads/junction) are effectively hard links for directories. They all Windows to reference the same physical storage from different logical directories.

By creating a Junction in OneDrive pointing to the directory where the files *really* live, this will allow OneDrive to sync the files and keep them backed up in to the cloud on a regular basis.

Below are the example snippets (in Powershell) showing how to configure things. Note, for my setup, I'll be storing things in the base path (in OneDrive) of: `$Env:USERPROFILE/OneDrive/Documents/My Games/`

## Satisfactory

[Satisfactory](https://www.satisfactorygame.com/) was the first game that I did this for, and there I (for what ever reason) chose to only store [the saved game folder](https://satisfactory.gamepedia.com/Save_files), and not all the game files.

```powershell
# Make the Junction in Satisfactory Save Folder point to OneDrive backed up version of the files
New-Item -ItemType Junction -Path "$Env:LOCALAPPDATA/FactoryGame/Saved/SaveGames/" -Target "$Env:USERPROFILE/OneDrive/Documents/My Games/FactoryGame/SaveGames/"
```

## Factorio

For [Factorio](https://factorio.com/) the attempt was to store the saves AND the all the blueprint files, so therefore I linked [the entire Factorio directory](https://wiki.factorio.com/Application_directory).

```powershell
# Make the Junction in OneDrive pointing to the Factorio user data
New-Item -ItemType Junction -Path  "$Env:USERPROFILE/OneDrive/Documents/My Games/Factorio/" -Target "$Env:APPDATA/Factorio/"
```

## References

- [Create Junctions using Powershell](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/new-item?view=powershell-7.1#examples)
