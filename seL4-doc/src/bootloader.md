# Boot Loader

## Overview

When a computer is turned off, its software remains stored on non-volatile memory. When the computer is powered on, a bootstrap loader is required: a small, low-level program to configure the computer's memory and devices sufficient to then be able to load further software, such as a bare-metal application or operating system, into RAM and then call it.

**Das U-Boot** (subtitled "the Universal Boot Loader" and often shortened to U-Boot) is an open-source boot loader commonly used in embedded devices. For such low-level operations, clearly it has to be customised for the hardware on which it is running. As part of this Developer Kit we have provided a U-Boot build suitable for the MaaXBoard.

## Building U-Boot for the MaaXBoard

**Josh**

## Loading the Application

After U-Boot has configured the computer and its devices, ...
