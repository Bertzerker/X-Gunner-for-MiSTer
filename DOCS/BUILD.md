# How To Build The Patched Main_MiSTer Binary

Most testers can use the prebuilt binary in this repository:

```sh
Main_MiSTer/binaries/MiSTer-xgunner-lightgun-only-20260920
```

Build from source if you want to review the patch, test against a newer Main_MiSTer revision, or make changes.

## Requirements

Build host:

- Linux or WSL.
- `git`
- `bash`
- `make`
- `wget`
- `tar`
- `sed`
- ARM Linux cross compiler using the `arm-none-linux-gnueabihf` prefix.

The Main_MiSTer tree includes `setup_default_toolchain.sh`, which can download and configure the expected Arm toolchain for a local build shell.

MiSTer test hardware:

- MiSTer FPGA.
- X-GUNNER receiver and gun.
- A way to copy the built `MiSTer` binary to the MiSTer SD card.
- Back up the existing `/media/fat/MiSTer` binary before replacing it.

Python helper notes:

- See `DOCS/REQUIREMENTS.txt`.
- No pip install step is needed.

## Patch Base

The included patch was prepared against this Main_MiSTer commit:

```sh
5b3ae64440f94c7a73c9e7fc4cc3a42e4af156d3
```

It may still apply to newer Main_MiSTer revisions, but if upstream `input.cpp` or `menu.cpp` changes around the lightgun code, the patch may need a small manual refresh.

## Build Steps

Clone Main_MiSTer:

```sh
git clone https://github.com/MiSTer-devel/Main_MiSTer.git
cd Main_MiSTer
```

Check out the tested base revision:

```sh
git checkout 5b3ae64440f94c7a73c9e7fc4cc3a42e4af156d3
```

Apply the X-GUNNER patch from this repository:

```sh
git apply /path/to/X-Gunner/Main_MiSTer/patches/Main_MiSTer-xgunner-lightgun.patch
```

Set up the toolchain if `arm-none-linux-gnueabihf-gcc` is not already on your `PATH`:

```sh
source ./setup_default_toolchain.sh
```

Build:

```sh
make clean
make
```

The `20260920` binary was built in Docker using Main_MiSTer's `.devcontainer/Dockerfile` because this Windows host did not have native `make`, WSL access, or the Arm toolchain available.

The `MiSTer_RA-xgunner-lightgun-only-20260920` binary was built from `odelot/Main_MiSTer` tag `v1.12.2` with the same X-GUNNER patch applied.

The built binary will be:

```sh
bin/MiSTer
```

Copy `bin/MiSTer` to the MiSTer SD card as `/media/fat/MiSTer`, then reboot or restart Main_MiSTer.

## Repository Layout

Files are grouped by purpose and by MiSTer install location where possible:

- `BUILD/` contains the current install package.
- `BUILD/config/` contains `.CFG` files for `/media/fat/config/`.
- `BUILD/config/inputs/` contains input `.map` files for `/media/fat/config/inputs/`.
- `BUILD/scripts/` contains the unified helper script for `/media/fat/Scripts/`.
- `DOCS/` contains build notes, issue notes, task notes, and the control table.
- `Main_MiSTer/binaries/` contains prebuilt Main_MiSTer test binaries.
- `Main_MiSTer/patches/` contains the current source patch for Main_MiSTer.
- `Main_MiSTer/patches/backup/` contains older patch attempts kept for review.
- `DOCS/LIGHTGUN_CORES.md` lists the included normal and RetroAchievements core profiles.

## Notes

The upstream `build.sh` can build and deploy over the network, but it is meant for a local developer setup. For public testing, the safer path is to build with `make`, copy the binary manually, and keep any local host files or connection details out of commits.

Do not commit private test reports, local host files, SSH keys, API tokens, or full device inventories.
