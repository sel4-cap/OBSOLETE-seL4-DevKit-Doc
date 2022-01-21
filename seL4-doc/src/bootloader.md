# Boot Loader

## Overview

When a computer is turned off, its software remains stored in non-volatile memory. When the computer is powered on, a bootstrap loader is required: a small, low-level program to configure the computer's memory and devices sufficient to then be able to load further software, such as a bare-metal application or operating system, into RAM and then call it.

**Das U-Boot** (known as "the Universal Boot Loader" and often shortened to U-Boot) is an open-source boot loader commonly used in embedded devices. For such low-level operations, it has to be customised for the hardware on which it is running. As part of this Developer Kit we have provided a U-Boot build suitable for the MaaXBoard.

## Building U-Boot for the MaaXBoard

**Josh**

## Loading the Application

After U-Boot has configured the MaaXBoard's memory and devices, it is able to load an application into RAM and then execute it. It is possible for a user to do this interactively using U-Boot commands via the serial terminal on the host machine. It is also possible and convenient to provide a U-Boot configuration file `uEnv.txt` that runs automatically; both options are documented below.

### Methods of Loading

We cover the three primary mechanisms for loading the application into RAM by U-Boot below:

- from SD card;
- from USB flash drive;
- via TFTP.

Additional mechanisms are available, such as downloading over the serial cable; however, this would be much slower compared with the options above and is not considered further here. Downloading from on-board flash memory is another possible mechanism, but that is not applicable to the MaaXBoard.

The remainder of this section assumes that you have an application in the form of an executable ELF file called `sel4_image`. We will create this as our test application [later in this documentation](building_applications.md), but this section is concerned with the loading mechanisms rather than the executable itself.

#### Loading from SD Card

The SD card is used to store U-Boot, the boot loader. The SD card may also be used to store the application.

Firstly, you need to write the executable ELF file to the SD card. If you have partitioned your SD card as described in the [SD Card Preparation](sd_card_preparation.md) section, you should have a `BOOT` partition. Simply copy the application file (`sel4_image` in our example) to the `BOOT` partition of the SD card.

