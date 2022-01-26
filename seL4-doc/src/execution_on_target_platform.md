# Execution on Target Platform

## Prerequisites

This assumes the following, building on previous sections:

- The MaaXBoard is powered and connected via the USB to TTL serial cable to the host machine.
- The host machine is running CoolTerm (or equivalent), configured for communication with the MaaXBoard.
- An SD card has been prepared containing U-Boot.
- The SD card's `BOOT` partition contains an appropriate version of `uEnv.txt`. If the chosen mechanism for transferring the executable is TFTP, this assumes that the `serverip` and `ipaddr` IP addresses have been edited correctly; ascertaining these is covered in [First Boot](first_boot.md).
- The seL4Test application has been built.

## Preparing the seL4Test Application

In the [Building Applications](building_applications.md) section, the `sel4test-driver-image-arm-maaxboard` executable was produced. The rest of this section is written for a more general case of executing a file named `sel4_image`, so within the Docker container simply copy or rename the seL4Test executable from `sel4test-driver-image-arm-maaxboard` to `sel4_image` using `cp` or `mv`.

In [Build Environment Setup](build_environment_setup.md), the Docker container's host directory was mapped to `/scratch/seL4` on the host machine; therefore `sel4_image` should be available at `/scratch/seL4/seL4Test/build-MaaXBoard-AArch64/images/sel4_image` (for an AArch64 build).

## Choosing the Bootloader Mechanism

In [Bootloader](bootloader.md), three transfer mechanisms were introduced:
1. SD card;
2. USB flash drive;
3. TFTP.

No changes to `uEnv.txt` are required for options 1 or 2, since they are prioritised, if found. Consequently, for TFTP to be used by the supplied `uEnv.txt`, `sel4_image` must not be present on either the SD card or the USB flash drive.

### SD Card

To use this method, mount the SD card onto the host machine using the USB Micro SD card reader / writer and copy `sel4_image` to the `BOOT` partition using the normal means of file copying (e.g. it does not need to flashed using Etcher, as it is already a formatted drive). Then return the SD card to the MaaXBoard.

### USB Flash Drive

To use this method, mount the USB flash drive onto the host machine and ensure that it has been formatted as FAT; then copy `sel4_image` to the flash drive. Then insert the USB flash drive into one of the USB ports on the MaaXBoard.

### TFTP

If Ethernet is intended to be used then an Ethernet cable must be connected either directly to the host machine or to a network to which the host machine is connected. This is described further in [Bootloader](bootloader.md) and [First Boot](first_boot.md).

The TFTP Server application must be running on the host machine, and the _Change Path_ menu option is used to navigate to the directory containing `sel4_image`; in the example above, this path would be `/scratch/seL4/seL4Test/build-MaaXBoard-AArch64/images/`. Remember also to start the server using the _Start TFTP_ menu option.

## Executing seL4Test

When all the items above are correctly set, upon powering up the MaaXBoard, CoolTerm will show U-Boot initialising the board and then loading `sel4_image` into RAM from the chosen medium. The final `bootelf` command within `uEnv.txt` will run the executable.

As introduced in [Building Applications](building_applications.md), [seL4Test](https://docs.sel4.systems/projects/sel4test) is a test suite for seL4 developed and maintained by the seL4 Foundation. It comprises over 100 tests that exercise the seL4 microkernel. Its progress as it runs can be followed in the CoolTerm window. Upon successful completion, the AArch64 version should finish with:
```
Test suite passed. 129 tests passed. 42 tests disabled.
All is well in the universe
```
The AArch32 version completes with a similar message, having run a few more tests:
```
Test suite passed. 138 tests passed. 42 tests disabled.
All is well in the universe
```

Once seL4Test has completed, no further interaction is possible and the MaaXBoard will have to be powered off. As a general point, there is no reset switch or button on the MaaXBoard, so our recommendation is to power cycle the MaaXBoard using the mains adapter of the power supply unit, rather than repeatedly disturbing the USB-C power connector on the board itself, which is more sensitive to wear and tear.