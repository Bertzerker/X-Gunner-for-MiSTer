# X-GUNNER on MiSTer FPGA

## Why This Patch Exists

The X-GUNNER receiver appears to MiSTer as a composite USB device. The tested P1 receiver exposes separate keyboard, mouse, and joystick-style input interfaces. The aiming data is reported through the mouse interface as absolute coordinates, while several physical buttons are reported as keyboard or mouse-button events.

That layout does not work cleanly with stock MiSTer:

- The absolute mouse interface needs to be recognized as a MiSTer lightgun before cores can use it like Sinden, Gun4IR, or other supported lightguns.
- A normal mouse is not enough to operate the MiSTer OSD or activate the gun as a player input device.
- Keyboard keys that MiSTer already uses for OSD control are awkward to reuse as game buttons. In particular, `Enter` and `Escape` are used as Finish and Cancel in MiSTer's keyboard remap menu, so remapping the X-GUNNER's physical Enter button inside MiSTer is unreliable.

For current testing, the recommended setup is to use the X-GUNNER GUI to remap the gun's physical Enter button to a harmless key such as `M`, then bind that new key in MiSTer like a normal game button. Bind the OSD/menu button through MiSTer's normal input remap flow.

This is a public test package for using the X-GUNNER LCD lightgun on MiSTer FPGA.

The included Main_MiSTer patch adds native detection for the X-GUNNER USB IDs and treats the gun's absolute mouse interface as a MiSTer lightgun. The package also includes map files and one unified helper script for the cores that have been mapped so far.

The current install package is in `BUILD/`. Build notes and issue notes are in `DOCS/`. Console lightgun profile notes are in `DOCS/LIGHTGUN_CORES.md`.

## Status

Implemented and packaged:

- Native Main_MiSTer lightgun detection patch for X-GUNNER P1-P4 USB IDs.
- A current patched Main_MiSTer binary for testers: `Main_MiSTer/binaries/MiSTer-xgunner-lightgun-only-20260920`.
- A current patched RetroAchievements Main_MiSTer binary for testers: `Main_MiSTer/binaries/MiSTer_RA-xgunner-lightgun-only-20260907`.
- PSX GunCon and Justifier input maps.
- Normal PSX GunCon and Justifier config profiles.
- PSX 2X CPU GunCon and Justifier input maps for both `PSX_2XCPU` and `PSX2XCPU` launch names.
- Saturn Virtua Gun input maps.
- Saturn `.CFG` files used during testing.
- NES Zapper configs and input maps for normal and RetroAchievements cores.
- SNES Super Scope and Justifier configs and input maps for normal and RetroAchievements cores.
- Genesis/Mega Drive, MegaCD/Sega CD, and S32X lightgun configs and input maps for normal and RetroAchievements cores.
- SMS Phaser configs and input maps for normal and RetroAchievements cores.
- Atari 7800 XG-1 configs and input maps for normal and RetroAchievements cores.
- Arcade input maps for Laser Ghost, N.Y. Captor, Colt, Bronx, Operation Wolf variants, and Point Blank 2 / Gunbarl variants.

Not included:

- The separate input control mapper script. That tool is still experimental and is not part of this repository.
- Old experimental binaries. Current binaries are kept in `Main_MiSTer/binaries/` and copied into `BUILD/`; old patch attempts are kept under `Main_MiSTer/patches/backup/` instead.
- MiSTer console core `.rbf` files. The config and map files target normal and RetroAchievements core names, but users should install cores from their normal MiSTer and RetroAchievements sources.

## X-GUNNER Device IDs

The X-GUNNER manual lists one USB product ID per player:

- P1: `1209:0001`
- P2: `1209:0002`
- P3: `1209:0003`
- P4: `1209:0004`

The tested P1 gun appeared as these Linux input interfaces:

- `HONGWEIHUA XGUNNER-P1 Keyboard`
- `HONGWEIHUA XGUNNER-P1 Mouse`
- `HONGWEIHUA XGUNNER-P1`

The important interface is the `Mouse` device. It reports absolute `ABS_X` and `ABS_Y` coordinates from `0` to `32767`, plus mouse button events.

Firmware GUI note:

- Remapping the physical X-GUNNER Enter button to `M` in the X-GUNNER GUI was tested working. This is the preferred way to make that physical button available to MiSTer; do not use MiSTer's keyboard remap to turn Enter into an OSD key.
- The X-GUNNER GUI can update the mapping over the serial connection without reflashing the device. A future MiSTer-side helper may be able to send the same serial commands.

## How The Main_MiSTer Patch Works

The patch is in `Main_MiSTer/patches/Main_MiSTer-xgunner-lightgun.patch`.

It changes `input.cpp` and `menu.cpp` in Main_MiSTer:

- Adds `input_is_xgunner()` to recognize VID `1209` with PID `0001` through `0004`.
- Detects X-GUNNER by VID `1209` and PID `0001` through `0004`, without depending on the reported device name. This keeps detection working after firmware updates that change the USB interface layout.
- Adds `input_xgunner_setup()` to mark the device as `QUIRK_LIGHTGUN_MOUSE`.
- Marks the device as a MiSTer lightgun and assigns the player number from the product ID.
- Sets the default calibration range to X/Y `0` through `32767`, matching the observed absolute axis range.
- Calls the X-GUNNER setup both during normal input detection and after MiSTer merges related input interfaces.
- Leaves keyboard-button mapping to MiSTer config or to the X-GUNNER GUI. The patch does not translate X-GUNNER keyboard keys into gamepad buttons.
- Does not ship a `kbd_1209_0001.map` keyboard remap. Do not remap `Enter` or `Escape`; MiSTer's keyboard remap menu uses them as Finish and Cancel.
- Stops raw X-GUNNER absolute-axis events from advancing the lightgun calibration menu before a button press.
- Adds a calibration debounce timer: 750 ms when entering calibration, then 350 ms after each accepted calibration edge.

The debounce matters because the gun can report tracking and button transitions immediately when the calibration screen appears. Without the delay, MiSTer can skip through calibration points too quickly.

## Current Test Binary

Use this binary for the latest packaged native test:

```sh
Main_MiSTer/binaries/MiSTer-xgunner-lightgun-only-20260920
```

SHA-256:

```sh
87a19270e08abafa40c477e782afbd08abd369d6ca47fa5cde8af8e9eadc6a10
```

The same current binary is also copied into `BUILD/`, which is the folder testers should use when installing the package.

RetroAchievements Main_MiSTer test binary:

```sh
Main_MiSTer/binaries/MiSTer_RA-xgunner-lightgun-only-20260907
```

SHA-256:

```sh
096aebbc13fb2b8d27468bc3395f7a93b23280de1c9d53e34246ebf02063be84
```

## Tested And Mapped Cores

PSX and Saturn have direct local testing notes below. The additional NES, SNES, Genesis/Mega Drive, MegaCD/Sega CD, S32X, SMS, and Atari 7800 profiles are configured from MiSTer's published lightgun documentation and core option strings, and need wider testing on real games.

### PSX

Mapped profiles:

- `config/inputs/PSX_input_1209_0001_v3.guncon.map`
- `config/inputs/PSX_input_1209_0001_v3.justifier.map`
- `config/inputs/RA_PSX_input_1209_0001_v3.guncon.map`
- `config/inputs/RA_PSX_input_1209_0001_v3.justifier.map`
- `config/inputs/PSX_2XCPU_input_1209_0001_v3.guncon.map`
- `config/inputs/PSX_2XCPU_input_1209_0001_v3.justifier.map`
- `config/inputs/PSX2XCPU_input_1209_0001_v3.guncon.map`
- `config/inputs/PSX2XCPU_input_1209_0001_v3.justifier.map`

Active/default map in the package:

- `config/inputs/PSX_input_1209_0001_v3.map`
- `config/inputs/RA_PSX_input_1209_0001_v3.map`
- `config/inputs/PSX_2XCPU_input_1209_0001_v3.map`
- `config/inputs/PSX2XCPU_input_1209_0001_v3.map`

Normal PSX config profiles:

- `config/PSX.guncon.CFG`
- `config/PSX.justifier.CFG`
- `config/PSX.CFG`

RetroAchievements PSX config profiles:

- `config/RA_PSX.guncon.CFG`
- `config/RA_PSX.justifier.CFG`
- `config/RA_PSX.CFG`

PlayStation 2X CPU config profiles:

- `config/PSX_2XCPU.guncon.CFG`
- `config/PSX_2XCPU.justifier.CFG`
- `config/PSX_2XCPU.CFG`
- `config/PSX2XCPU.guncon.CFG`
- `config/PSX2XCPU.justifier.CFG`
- `config/PSX2XCPU.CFG`

Helper scripts:

```sh
sh /media/fat/Scripts/xgunner_map.sh
sh /media/fat/Scripts/xgunner_map.sh psx-guncon
sh /media/fat/Scripts/xgunner_map.sh psx-justifier
```

GunCon profile:

- Trigger: `Circle` / shoot
- Mouse 2: `Start` / GunCon A
- Mouse 3: reserved for reload/offscreen shot
- `5`/`e`: `X` / GunCon B

Justifier profile:

- Trigger: `Circle` / shoot
- Mouse 2: `X` / special
- Mouse 3: reserved for reload/offscreen shot
- Bind `Start` through MiSTer's normal input remap flow or through a harmless key assigned in the X-GUNNER GUI.

In the PSX core OSD, set `Pad1` to `GunCon` or `Justifier` to match the game, then use the matching helper script or copy the matching `.map` file into place.

PSX test note:

- The PSX mappings were tested with `PSX_unstable_20260821_19f225.rbf`.
- The official stable core `PSX_20260807.rbf` was downloaded from `MiSTer-devel/Distribution_MiSTer` and installed on the test MiSTer, but core `.rbf` files are not included in this repository.
- `_Console/PlayStation (2X CPU).mgl` uses setname `PSX_2XCPU`.
- `_Other/PSX2XCPU_20260413.rbf` may use `PSX2XCPU` when launched directly.
- The PSX helper scripts update normal PSX, RA PSX, and both 2X CPU launch names together.

### Saturn

Mapped profile:

- `config/inputs/Saturn_input_1209_0001_v3.virtua_gun.map`

Active/default maps in the package:

- `config/inputs/Saturn_input_1209_0001_v3.map`
- `config/inputs/RA_Saturn_input_1209_0001_v3.map`
- `config/inputs/A0CD-Saturn_input_1209_0001_v3.map`

Helper script:

```sh
sh /media/fat/Scripts/xgunner_map.sh saturn
```

Virtua Gun profile:

- Trigger: `A` / shoot
- Mouse 2: `B`
- Mouse 3: reserved for reload/offscreen shot
- `5`/`e`: `C`
- Bind `Start` through MiSTer's normal input remap flow or through a harmless key assigned in the X-GUNNER GUI.

Included Saturn config files:

- `config/Saturn.CFG`
- `config/RA_Saturn.CFG`
- `config/A0CD-Saturn.CFG`
- `config/Saturn_20260713.CFG`

Saturn core compatibility note:

- `Saturn_20251003.rbf` was tested working with Virtua Cop 2 and X-GUNNER.
- `Saturn_20260713.rbf` was tested not working for X-GUNNER lightgun input.
- The RetroAchievements Saturn `poc` core was tested not working for X-GUNNER lightgun input.
- Core files are not included in this repository.

RetroAchievements Saturn note:

- The RA Saturn `poc` core needs the upstream lightgun reset fix from MiSTer-devel/Saturn_MiSTer commit `526332d4c06291e4b402ace3c753a4baf08f5074`.
- The local source patch is `RA_Saturn_lightgun_reset_fix.patch`.
- Without a rebuilt RA Saturn `.rbf`, maps and Main_MiSTer changes alone are not enough to fix the missing crosshair/trigger behavior.

### Additional Console Lightgun Profiles

See `DOCS/LIGHTGUN_CORES.md` for the full profile matrix, OSD settings, and button mappings.

New mapped/configured profiles:

- NES Zapper: `NES` and `RA_NES`.
- SNES Super Scope: `SNES` and `RA_SNES`.
- SNES Justifier: `SNES` and `RA_SNES`.
- Genesis / Mega Drive lightgun mode: normal `Genesis` and RA `RA_MegaDrive`.
- MegaCD / Sega CD Justifier: `MegaCD` and `RA_MegaCD`.
- MegaCD / Sega CD Menacer: `MegaCD` and `RA_MegaCD`.
- S32X lightgun mode: `S32X` and `RA_S32X`.
- SMS Phaser: `SMS` and `RA_SMS`.
- Atari 7800 XG-1: `Atari7800` and `RA_Atari7800`.

Helper scripts:

```sh
sh /media/fat/Scripts/xgunner_map.sh
sh /media/fat/Scripts/xgunner_map.sh nes
sh /media/fat/Scripts/xgunner_map.sh snes-super-scope
sh /media/fat/Scripts/xgunner_map.sh snes-justifier
sh /media/fat/Scripts/xgunner_map.sh genesis
sh /media/fat/Scripts/xgunner_map.sh megacd-justifier
sh /media/fat/Scripts/xgunner_map.sh megacd-menacer
sh /media/fat/Scripts/xgunner_map.sh s32x
sh /media/fat/Scripts/xgunner_map.sh sms
sh /media/fat/Scripts/xgunner_map.sh atari7800
```

Mega Jet and Mega Gun are not included as separate named profiles because they were not exposed as current MiSTer gun-mode choices in the documentation or core option strings checked for this update.

NES note:

- The default NES profile now uses `Zapper(Joy1)` because the tested X-GUNNER P1 receiver is MiSTer player 1.
- The NES core still feeds the emulated Zapper to the NES game as port 2. The Joy1/Joy2 choice selects which MiSTer input source supplies the lightgun coordinates.
- Use `sh /media/fat/Scripts/xgunner_map.sh nes-zapper-joy2` only if you manually assign the X-GUNNER as MiSTer player 2.

### Arcade Lightgun Profiles

The following arcade MRA setnames now have X-GUNNER maps:

- `lghost`, `lghostj`, `lghostu`: Laser Ghost.
- `nycaptor`, `colt`, `bronx`: N.Y. Captor / related bootlegs.
- `opwolf`, `opwolfa`, `opwolfu`, `opwolfj`, `opwolfjsc`, `opwolfp`: Operation Wolf.
- `ptblank2a`, `ptblank2b`, `ptblank2c`, `ptblank2ua`, `gunbarla`: Point Blank 2 / Gunbarl.

Arcade mapping:

- Trigger: `A`, main gun/fire.
- Mouse 2: `B`, secondary input such as grenade or special weapon.
- Mouse 3: reserved for reload/offscreen shot.
- Bind `Start` through MiSTer's normal input remap flow or through a harmless key assigned in the X-GUNNER GUI.
- `5`/`e`: third game button where the core exposes one.
- `s`: `Coin` / `Select`.
- `w`: `Pause` where the core exposes one.

`Oh! Bakyuuun` is not included because the installed MRA says light gun is not supported by that core yet.

Arcade test notes:

- Operation Wolf works correctly with X-GUNNER.
- Bronx input works, but its vertical aiming axes are rotated +90 degrees.
- Point Blank 2 receives aim input, but does not work correctly in absolute lightgun mode; this appears to be a SYSTEM11 core issue.
- Laser Ghost starts and the crosshair moves, but the game itself did not work correctly in testing.
- The test MiSTer has a convenience launcher folder at `_Arcade/X-GUNNER Lightgun/`.

## Calibration Notes

Calibrate in this order:

1. Put the X-GUNNER in light gun mode.
2. Calibrate the X-GUNNER with its own five-point process.
3. Start the patched Main_MiSTer binary.
4. Open the target MiSTer core.
5. Open the OSD and press `F10` to run MiSTer's lightgun calibration.

Useful X-GUNNER hotkeys from the manual:

- Calibration pause mode: `Space + Start`, then pull trigger once.
- Calibration sequence: center, up, down, left, right.
- Off-screen trigger as right-click on: `Space + 5 + S`.
- Off-screen trigger as right-click off: `Space + 5 + W`.
- Light gun mode: `Space + 5 + Joystick Up`, or COM command `G`.
- 4:3 mode: `Space + A`, or COM command `Q`.
- 16:9 mode: `Space + D`, or COM command `V`.

## References

- X-GUNNER official site: https://hwhxg.com/
- MiSTer controller and lightgun docs: https://mister-devel.github.io/MkDocs_MiSTer/basics/input/
- Main_MiSTer source: https://github.com/MiSTer-devel/Main_MiSTer
