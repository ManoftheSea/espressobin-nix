# NixOS on the Marvell ESPRESSObin
http://espressobin.net/

## U-Boot
Different versions of the ESPRESSObin board have different memory layouts, which require different builds of the bootloader.

Build the bootloader with `nix-build -A pkgsCross.aarch64-multiplatform.ubootEspressobinImages_<configuration>`, replacing `<configuration>` with one of:
- `512MB `
- `V5_1GB_2CS `
- `V5_1GB_1CS `
- `V7_1GB `
- `V7_2GB `
- `V5_2GB `

eg. To build for the V7 1GB board, run `nix-build -A pkgsCross.aarch64-multiplatform.ubootEspressobinImages_V7_1GB`

### Flashing
Adapted from the USB stick instructions from http://wiki.espressobin.net/tiki-index.php?page=Update+the+Bootloader

1. Copy `result/flash-image.bin` to a USB stick.
1. Connect a USB cable to the serial port on the ESPRESSObin and boot it with the current bootloader.
1. Press enter to stop autoboot and get to the shell.
1. Run `bubt flash-image.bin spi usb`
1. Run `reset` to reset the board and boot into the new bootloader.

### Extra steps after flashing
Stop autoboot, load the default U-Boot environment with `env default -a`, save it with `env save`, then `reset`.

The goal is to get the `fdtfile` U-Boot variable to be automatically set correctly. I've seen corruption the first time I run those commands where the `m` in `marvell` is missing (see below). Running the commands again seems to set it correctly.

To see the current value, run `echo $fdtfile`.

- On an ESPRESSObin V5, the correct value is `marvell/armada-3720-espressobin.dtb`.
- On an ESPRESSObin V7, the correct value is `marvell/armada-3720-espressobin-v7.dtb`.

### Recovering from a bad flash
See https://github.com/MarvellEmbeddedProcessors/u-boot-marvell/blob/u-boot-2017.03-armada-17.06/doc/mvebu/uart_boot.txt

`WtpDownload_linux` is built as part of `ubootEspressobinImages`.

## SD image
The default Aarch64 NixOS image will boot unmodified on the ESPRESSObin, but there will be no serial output by default.

To get serial output over USB in Linux, `console=ttyMV0,115200n8` needs to be added to the kernel command line.

To build an SD card image with this already set, run `nix-build '<nixpkgs/nixos>' -A config.system.build.sdImage -I nixos-config=./sd-image.nix`\
TODO: This probably requires an Aarch64 host, figure out how to cross-compile.

## TODO
- Fix MAC addresses not being passed to Linux.

## Links
- U-Boot documentation for Marvell SoCs: https://gitlab.denx.de/u-boot/u-boot/blob/master/doc/README.marvell
- ARM trusted firmware documentation for Marvell Armada: https://github.com/ARM-software/arm-trusted-firmware/blob/master/docs/plat/marvell/armada/build.rst

## Hardware notes
- The V7 schematics (page 9) imply you can boot U-Boot from SD card by setting the three mode jumpers to 1. The [wiki page](http://wiki.espressobin.net/tiki-index.php?page=Ports+and+Interfaces) table seems to be wrong, all modes except SPI NOR flash are the same.
