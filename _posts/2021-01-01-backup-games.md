---
title: "Backing Up Games to OneDrive"
date: "2021-01-01"
tags: []
# header:
#     overlay_image: ""
#     show_overlay_excerpt: false
published: true
---

Standards are hard... I get it. But it would be really nice if I could actually get all my games into the same place (ideally `%USERPROFILE%/Documents/My Games` on Windows). That way that they are included in my automatic OneDrive Backup of my `%USERPROFILE%/Documents` directory.

Enter Windows [Junctions](https://docs.microsoft.com/en-us/sysinternals/downloads/junction).

## Satisfactory

[Satisfactory](https://www.satisfactorygame.com/) was the first game that I did this for, and there I (for what ever reason) chose to only store the saved game folder, and not all the game files.

```powershell
# Make the Satisfactory top level directory in My Games
mkdir "$Env:LOCALAPPDATA/FactoryGame/"

# Move the Save Game from the "main" (original) location
Move-Item -Path "$Env:LOCALAPPDATA/FactoryGame/Saved" -Destination "$Env:USERPROFILE/My Games/FactoryGame/"

# Make the Junction in the main / original path pointing to the My Games version
New-Item -ItemType Junction -Path "$Env:LOCALAPPDATA/FactoryGame/Saved/" -Target "$Env:USERPROFILE/My Games/FactoryGame/Saved"
```

## Factorio

For [Factorio](https://factorio.com/) the attempt was to store the saves AND the all the blueprint files, so therefore I linked the entire Factorio directory.

```powershell
# Make the Satisfactory top level directory in My Games
mkdir "$Env:APPDATA/Factorio/"

# Move the Save Game from the "main" (original) location
Move-Item -Path "$Env:APPDATA/Factorio" -Destination "$Env:USERPROFILE/My Games/"

# Make the Junction in the main / original path pointing to the My Games version
New-Item -ItemType Junction -Path "$Env:APPDATA/Factorio/" -Target "$Env:USERPROFILE/My Games/Factorio/"
```

## References

- [Create Junctions using Powershell](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/new-item?view=powershell-7.1#examples)
