# Device Driver Introduction

This section seeks to provide guidance on adding device drivers to seL4.

It should be noted that seL4 runs all device drivers in user mode; device support is therefore not the kernel’s problem.

This guide focuses on methods, complete with an extensive worked example, for adding device support to seL4.

The following sections describe the creation of an extensible library for seL4 allowing U-Boot device drivers to function under seL4 with either minimal or no modifications. As an example of its usage, the library provides an extensive set of device drivers for the Avnet MaaXBoard (support for USB, Ethernet, SD/MMC, I<sup>2</sup>C, GPIO, IOMUX, Clock and SPI devices). Guidance on the extension of this library to support other platforms and devices is also provided.

## Goals

The goal of this guide is to lower the barrier to entry for the use of seL4. One such barrier is device driver support for seL4, both the relatively limited device driver support as well as the difficulty in the creation of new device drivers.

To help overcome this barrier, this guide attempts to provide a route for adding device support to seL4 with minimal effort and technical challenges. This goal has resulted in the following decisions and priorities being taken:

1. A focus on porting existing open source device drivers from other projects (e.g. Linux or U-Boot) over the writing of new device drivers; detailed guidance on the writing of new device drivers is outside the scope of this guide.

2. Methods of porting that allow devices to be supported with minimal effort, e.g. minimising changes to driver source code, have been prioritised over factors such as performance and support for all features.

3. Consideration for device driver formal verification has not been applied. No guarantee of correctness, security, or anything else is given for the drivers ported, or the method described.

## Porting Drivers

Device drivers can be ported from any source where the device driver source code is available under a suitable license. The most obvious such sources are Linux and U-Boot, both of which contain a very large selection of open source device drivers. It is suggested that porting drivers from U-Boot is easier than porting drivers from Linux for the following reasons:

- Porting a device driver requires either the driver to be made to function independently from the driver framework of its source project, or for the source project's driver framework to be made to function within seL4. The U-Boot driver framework is significantly less complex and extensive than the Linux driver framework.

- U-Boot device drivers tend to be significantly simpler than device drivers found in Linux. This is because the use cases of a bootloader such as U-Boot do not require high performance, fully-featured drivers. Indeed it can be seen that many of the U-Boot device drivers are in fact actively simplified versions of Linux drivers that have been ported to the U-Boot driver framework.

In line with the goals stated above this guide therefore focuses on the porting of drivers from U-Boot to seL4.
