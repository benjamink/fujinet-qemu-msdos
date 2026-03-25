# QEMU with FujiNet

## Using run-qemu

`run-qemu` starts FujiNet and launches QEMU with the correct serial configuration. Before using it, build the firmware first (see below).

```sh
./run-qemu
```

Key options (all can also be set via environment variables of the same name):

| Flag | Env var | Default | Description |
|---|---|---|---|
| `-m`, `--memory` | `MEMORY` | `64` | RAM in MB for the QEMU machine |
| `--hda` | `HDA` | `msdos.qcow2` | Hard disk image path |
| `--fda` | `FDA` | _(none)_ | Floppy disk image path |
| `--boot` | `BOOT` | `c` | Boot device (`c` = hard disk, `a` = floppy) |
| `--fujinet-path` | `FUJINET_FIRMWARE_PATH` | `~/code/fujinet-firmware` | Path to the fujinet-firmware repo |
| `--no-pkill` | `PKILL_ENABLED=false` | _(pkill enabled)_ | Skip killing existing fujinet processes |
| — | `FUJINET_PORT` | `65504` | FujiNet network port |

Examples:

```sh
# Boot from floppy
./run-qemu --fda disks/Disk1.img --boot a

# Use a custom firmware path, more memory, and skip pkill
./run-qemu --fujinet-path /opt/fujinet --memory 128 --no-pkill
```

## Build fujinet-firmware for Serial (RS232)

In the `fujinet-firmware` cloned source directory, run:

```sh
./build.sh -p RS232
```

The resulting binary and supporting files will be placed in `distfiles/`, which `run-qemu` uses via the `fujinet/` symlink.

## Setting up QEMU

### Create a hard disk 

```sh
qemu-img create -f qcow msdos.qcow2 200M
```

>**_NOTE:_** This repository contains a pre-build MS-DOS 6.22 200MB hard disk image for convenience.  If a new hard disk is created then it will need the operating system, FujiNet driver, & any other FujiNet commands installed to it before it will work with the virtual FujiNet device that runs alongside QEMU.

### Install MS-DOS 

```sh
qemu-system-i386 -hda msdos.disk -m 64 -L  -fda disks/Disk1.img -boot a
```

When the installer prompts to insert Disk #2... 

1. `ctrl-alt-2` - get to the Qemu console
1. `eject floppy0` - eject the floppy
1. `change floppy0 disks/Disk2.img` - insert Disk #2
1. `ctrl-alt-1` - change back to the emulator screen

### Booting to the Hard Drive

```sh
qemu-system-i386 -hda msdos.qcow2 -m 64 -L . -serial tcp:localhost:65504,reconnect=1 -boot c
```
