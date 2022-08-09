# Execution on Target Platform

## Prerequisites

This assumes the following, building on previous sections:

- The MaaXBoard is powered and connected via the USB-to-TTL serial UART cable to the host machine.
- The host machine is running CoolTerm (or equivalent), configured for communication with the MaaXBoard.
- An SD card has been prepared containing U-Boot.
- The SD card's `BOOT` partition contains an appropriate version of `uEnv.txt`. If the chosen mechanism for transferring the executable is TFTP, this assumes that the `serverip`, `ipaddr` and `netmask` environment variables have been configured correctly for the user's network.
- The seL4Test application has been built.

## Preparing the seL4Test Application

In the [Building Applications](building_applications.md) section, the `sel4test-driver-image-arm-maaxboard` executable was produced. The rest of this section, and indeed the commands in the supplied `uEnv.txt` U-Boot configuration file, expect a file named `sel4_image`. Therefore the `sel4test-driver-image-arm-maaxboard` executable must be renamed to `sel4_image` prior to execution using the mechanisms described below.

In [Build Environment Setup](build_environment_setup.md), the Docker container's host directory was mapped to `/scratch/seL4` on the host machine; therefore `sel4_image` should be available at `/scratch/seL4/seL4Test/build-MaaXBoard-AArch64/images/sel4_image` (for an AArch64 build) on the host machine.

## Choosing the Bootloader Mechanism

In the [Bootloader](bootloader.md) section, three transfer mechanisms, and the order in which these mechanisms will be attempted to access the `sel4_image` binary, were introduced:

1. USB flash drive;
2. SD card;
3. TFTP.

The following subsections provide guidance on how to use each of these mechanisms.

### USB Flash Drive

To use this method, mount the USB flash drive onto the host machine and ensure that it has been formatted as FAT32; then copy `sel4_image` to the root of the flash drive. Then insert the USB flash drive into the top USB port[^1] on the MaaXBoard prior to applying power to the MaaXBoard.

[^1]: Note: Currently, only the upper USB port on the Avnet MaaXBoard is active (i.e. the port furthest away from the PCB); the lower USB port does not function. This is a feature of the power domains on the board, not the USB driver.

### SD Card

To use this method, mount the SD card onto the host machine using the USB Micro SD card reader / writer and copy `sel4_image` to the root of the `BOOT` partition. Then return the SD card to the MaaXBoard prior to applying power to the MaaXBoard.

### TFTP

To use the TFTP mechanism, the MaaXBoard must be connected to the same network as the host machine and the network related environment variables within the `uEnv.txt` (i.e. `serverip`, `ipaddr` and `netmask`) are configured.

The TFTP Server application must be running on the host machine and serving the `sel4_image` binary prior to applying power to the MaaXBoard.

## Executing seL4Test

When guidance above has been followed for the chosen mechanism, upon powering up the MaaXBoard, CoolTerm will show U-Boot initialising the board and then loading `sel4_image` into RAM from the chosen medium. The final `bootelf` command within `uEnv.txt` will run the executable.

As introduced in [Building Applications](building_applications.md), [seL4Test](https://docs.sel4.systems/projects/sel4test) is a test suite for seL4 developed and maintained by the seL4 Foundation. It comprises tests that exercise the seL4 microkernel. Its progress as it runs can be followed in the CoolTerm window. Upon successful completion the seL4Test executable outputs a message of the following format:

```text
Test suite passed. 129 tests passed. 42 tests disabled.
All is well in the universe
```

Once seL4Test has completed, no further interaction is possible and the MaaXBoard will have to be powered off.

As a general point, there is no reset switch or button on the MaaXBoard, so our recommendation is to power cycle the MaaXBoard using the mains adapter of the power supply unit, rather than repeatedly disturbing the USB-C power connector on the board itself, which is more sensitive to wear and tear.
