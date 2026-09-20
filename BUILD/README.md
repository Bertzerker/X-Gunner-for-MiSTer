# X-GUNNER MiSTer Test Build

This folder is the current install package. It is meant to be copied to a MiSTer SD card without searching through the repository.

Back up your existing MiSTer files before replacing anything.

## Main MiSTer

Copy this file to the MiSTer root as `/media/fat/MiSTer`:

```text
MiSTer-xgunner-lightgun-only-20260920
```

You can also keep the filename and point an alternate MiSTer setup at it manually, but replacing `/media/fat/MiSTer` is the normal test path.

## RetroAchievements MiSTer

Copy this file to the MiSTer root:

```text
MiSTer_RA-xgunner-lightgun-only-20260907
```

To use it for RetroAchievements launches, set this in `MiSTer.ini`:

```ini
[RA_*]
main=MiSTer_RA-xgunner-lightgun-only-20260907
```

## Config, Maps, And Script

Copy the contents of `config/` to `/media/fat/config/`.

Copy the unified helper script to `/media/fat/Scripts/`:

```text
scripts/xgunner_map.sh
```

Run it on MiSTer when you want to switch profile files:

```sh
sh /media/fat/Scripts/xgunner_map.sh
sh /media/fat/Scripts/xgunner_map.sh psx-guncon
sh /media/fat/Scripts/xgunner_map.sh psx-justifier
sh /media/fat/Scripts/xgunner_map.sh saturn
```

The package does not include an X-GUNNER keyboard OSD map. Bind the OSD/menu button through MiSTer's normal input remap, and remap the gun's physical Enter button in the X-GUNNER GUI before binding it in MiSTer.
