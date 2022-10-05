# U-Boot Driver Library Overview

To achieve the goals specified in [device driver introduction](device_driver_intro.md) section, an extensible library of device drivers ported from U-Boot has been created. This library provides an extensive set of drivers for the Avnet MaaXBoard platform as a worked example of its usage. The library is supported by documentation in the following sections on usage, maintenance, and guidance on the extension of the library to support additional platforms and devices.

Information on the design and structure of the library is provided under the following sections:

- [Design Summary](#design-summary)
- [Build System](#build-system)
- [Library Limitations](#library-limitations)
- [Code Structure](#code-structure)

## Design Summary

The primary goal of the library is to allow drivers from U-Boot to be used within seL4 with minimal or no code changes. To support this goal, the library comprises:

- A set of U-Boot drivers.

- The U-Boot [Driver Model](https://u-boot.readthedocs.io/en/latest/develop/driver-model/index.html) driver framework and associated subsystems required to support device drivers.

- Stubbed versions of U-Boot subsystems providing a compatibility / conversion layer between the U-Boot source code and underlying seL4 libraries.

- A wrapper around the U-Boot code to provide an API for users of the library to interact with devices and manage library initialisation / shutdown.

It should be noted that the library works with the intended security and isolation of seL4; capabilities to permit access to hardware devices and provide sufficient resources (e.g. heap and DMA memory) must be granted to components using the library.

## Detailed Design Information

The following sections provide details on the design of stub or wrapper elements that are significant to support the functioning of the drivers within the Driver Model framework.

### Linker Lists

To allow variable levels of functionality to be built into a U-Boot executable, U-Boot utilises a concept of [linker-generated arrays](https://u-boot.readthedocs.io/en/latest/api/linker_lists.html). These are arrays of objects (e.g. drivers, supported commands, driver classes, etc.) which are stored by U-Boot within dedicated linker sections to be queried at runtime.

Access to the objects stored within these linker sections is provided by U-Boot through a set of macros defined in the `linker_lists.h` header file.

To mimic this functionality within the library, the following approach has been taken:

1. Rather than storing the objects within linker sections of the executable, a global variable named `driver_data` is declared within `driver_data.h` which contains the equivalent object arrays.

2. During initialisation of the library, the platform-dependent contents of the `driver_data` global variable are set up.

3. A stubbed version of the `linker_lists.h` header file is provided. Rather than accessing data from the executable's linker sections as performed by the original version, the stubbed version accesses data from the arrays within the `driver_data` global variable.

### Global Data

U-Boot maintains a [global data](https://u-boot.readthedocs.io/en/latest/develop/global_data.html) structure for the storage of globally required fields.

The macro `DECLARE_GLOBAL_DATA_PTR` is used by U-Boot to provide a pointer to this data structure. On most architectures this pointer is stored in a dedicated register (e.g. register `x18` on ARM64); such a mechanism is not compatible with seL4.

Within the library the following approach has been taken to global data:

1. A stub of the `global_data.h` header file has been provided to provide the macro `DECLARE_GLOBAL_DATA_PTR` as a pointer to a global variable.

2. At initialisation of the library, memory for the global data structure is allocated and initialised as follows:
   - Most elements are unused and set to null values.
   - The global status `flags` are set up to mimic an instance of U-Boot that has been relocated to, and is executing from, RAM.
   - Pointers to the flattened device tree are setup to point to seL4's device tree.
   - A [live device tree](https://u-boot.readthedocs.io/en/latest/develop/driver-model/livetree.html) is built from the flattened device tree and referenced.

### Timer

The U-Boot code expects to have access to a high-frequency, monotonic timer. The timer is accessed by the U-Boot source code through calls to routines `get_ticks`, `get_timer_*` and `timer_get_*`.

Provision of such a timer is platform dependent. For the Avnet MaaXBoard the timer has been implemented through use of the System Counter (SYS_CON) device provided by the iMX8MQ SoC; see file `timer_imx8mq.c` for details.

### Memory Mapped IO

One of the core mechanisms for providing a software interface to a hardware device is through the use of [memory mapped IO](https://en.wikipedia.org/wiki/Memory-mapped_I/O).

The memory addresses that a U-Boot driver uses for communication with a device can be either read from the device tree (i.e. from a device's `reg` property) or in some cases can be hard-coded into the driver.

At this point it should be noted that U-Boot source code is intended to be executed in an environment with direct access to the system's physical address space; however, the library is executed within a virtual address space provided by seL4. As such, the addresses used by U-Boot drivers are *physical* addresses which are not directly accessible by the library. To work around this issue:

1. During library initialisation the list of all platform-dependent devices that need to be accessed must be provided. For each device:
   - The device's physical address range is read from the device tree.
   - The device's physical address range is mapped into the virtual address space through use of the IO mapping routines provided by seL4's platform support library.
   - The mapping between the physical address range and the mapped virtual address range is stored within the `sel4_io_map` package (part of the library's wrapper functionality).

2. When a driver attempts to perform memory mapped IO it calls architecture-dependent routines from the `io.h` header, e.g. `readl` or `writel`. A stubbed version of the `io.h` header is provided that uses the `sel4_io_map` package to translate the physical addresses provided by the driver to the equivalent address mapped into the library's virtual address space.

U-Boot drivers performing memory mapped IO should perform seamlessly without the need for any modifications to the driver.

### DMA

As for [memory mapped IO](#memory-mapped-io), it should be noted that U-Boot source code expects to be executed in an environment with direct access to the system's physical address space; however the library is executed within a virtual address space provided by seL4.

This can lead to U-Boot drivers that are executed within seL4 erroneously providing a *virtual* memory address to devices as the address to use for a DMA transfer; DMA transfers performed by devices can only be performed to *physical* addresses.

To support this issue, the library:

1. Provides the `sel4_dma` package as part of its wrapper, which:
   - Provides memory allocation / deallocation routines that utilise the underlying DMA functions provided by seL4's platform support library;
   - Maintains a mapping between the physical and virtual addresses, and provides routines to perform address translations;
   - Provides routines to perform flushing and invalidation of the allocated DMA regions.

2. Provides an implementation of Linux's `dma_mapping.h` interface via the `sel4_dma` package.

3. Provides modifications to the XHCI USB stack, as used by USB 3.0 devices, intended to cover all DMA issues related to *users* of the USB stack (e.g. keyboard, mouse, or mass storage devices).

It is an unfortunate fact that drivers utilising DMA may require manual modifications to function correctly:

- Drivers that use the DMA interface exported by `dma-mapping.h` should work without modification; drivers ported to U-Boot from Linux have been seen to use this interface. Examples of device drivers for the Avnet MaaXBoard using the `dma-mapping.h` DMA interface are the USB driver and the SD/MMC driver.

- Drivers that do not use the DMA interface exported by `dma-mapping.h` will need to be modified to use the API exported by the `sel4_dma` wrapper package. An example of a device driver for the Avnet MaaXBoard that required manual modifications is the Ethernet driver (see `fec_mxc.c`).

### Console

A minimal stub of U-Boot's console subsystem has been provided (see `console.c`). This stub provides the subset of functionality required to allow input and output devices to be registered and accessed by the driver library.

It should be noted that the console subsystems's `stdout` file is not used by the library; instead all output is routed directly to the C library output routines.

The console subsystem's `stdin` file, however, is used. For example, if a USB keyboard is registered with the console as the `stdin` device then subsequent calls to retrieve input from `stdin` will return keypresses that are input from the USB keyboard.

### Standard Output and Logging

The wrapper header file `uboot_print.h` provides a set of macros that map:

- All U-Boot standard output routines onto calls to the C library `printf` routine.
- All U-Boot logging routines onto the seL4 platform support library `ZF_LOG*` routines at an equivalent logging level.

### Initialisation

As part of library initialisation, any U-Boot subsystems that require explicit initialisation are handled (see `uboot_wrapper.c:initialise_uboot_wrapper`). This mimics the initialisation that would normally be performed by [U-Boot's start-up routine](https://github.com/u-boot/u-boot/blob/master/common/board_r.c).

Initialisation of the library comprises:

- Initialisation of the memory mapped IO and DMA wrappers;
- Initialisation of the monotonic timer;
- Initialisation of the Linker Lists data structure;
- Initialisation of the Global Data data structure;
- Initialisation of U-Boot's Environment subsystem (manages storage of environment variables);
- Initialisation of U-Boot's Driver Model subsystem;
- Initialisation of U-Boot's MMC subsystem (if SD/MMC drivers are used by the platform);
- Initialisation of U-Boot's Network subsystem (if Ethernet drivers are used by the platform).

When calling the `initialise_uboot_wrapper` routine the following must be provided:

- The I/O operations data structure from the platform support library. These routines enable the library to access the seL4 device tree and to perform memory mapping and DMA operations. The routines are provided by CAmkES on creation of a single threaded component.
- The list of device tree paths containing physical addresses to be memory mapped (through the `reg` property).
- The list of device tree paths for the devices to enable. Note that all sub-nodes of the device tree paths will be automatically enabled; only the root node for the required devices need to be listed. All other nodes in the device tree will be marked as disabled.

A worked example for use of the `initialise_uboot_wrapper` routine is provided by the [`uboot-driver-example` test application](uboot_driver_usage.md#test-application-uboot-driver-example) for the Avnet MaaXBoard.

## Build System

U-Boot is, by necessity, highly configurable in terms of which functionality is included in a build and the configuration of default values / settings. The U-Boot source code relies upon a KConfig build system that results in the definition of a set of macros (typically named `CONFIG_xxx`), together with the identification of the required source code files, to manage this configuration; however, seL4 relies upon the CMake build system. To resolve this issue the following approach has been taken:

1. Macros that are expected to be consistent across all platforms, e.g. those supporting the basic U-Boot subsystem configuration that the library relies upon, are defined in the ```uboot_helper.h``` header file within the library's wrapper.

2. All other configuration, i.e. macros, includes and sources files, are controlled by the library's CMake file (```CMakeLists.txt```).

As such all architecture and platform dependent configuration is encapsulated within the library's CMake file. The general structure of this file is as follows:

- A section controlling architecture dependent settings. The settings necessary for ARM-based platforms (both ARMv7 and ARMv8) have been provided.

- A section controlling platform dependent settings. This contains platform-specific macros, identification of drivers supporting the platform, and setup of platform specific header files.

- One section for each class of device (e.g. clock devices, USB devices, etc.). For each class of device the generic settings are provided (i.e. settings required irrespective of the chosen driver) as well as the settings provided for each supported driver.

- A section controlling configuration consistent across all architectures and platforms, e.g. the set of source files required by all platforms.

This modular structure of the CMake file is intended to allow for the library to be incrementally extended, i.e. through the addition of support for new architectures, platforms, device classes and drivers.

## Library Limitations

Users of the library should be aware of its limitations, and potential workarounds for those limitations.

1. **Thread safety**: The library is not thread safe; as such it is the responsibility of the user to serialise access to any single instance of the library. Note, however, that multiple instances of the library may be used. For example, two instances of the library could be used concurrently, each held within separate CAmkES components. IF multiple instances of the library are used, it is the responsibility of the user to ensure that each instance is using disjoint devices, i.e. two instances of the library would not both be able to access the same USB device; however, it should be possible for one instance to access an Ethernet device whilst a second instance accesses a USB device (see the [case study application](case_study_intro.md) for an example of this).

2. **Performance**: Do not expect great performance from the library. The underlying U-Boot drivers have tended to prioritise simplicity over performance; for example the SPI driver for the Avnet MaaXBoard does not support the use of DMA transfers even though the underlying device can perform DMA transfers. Additionally, the library wrapper adds additional layers of address translations and data copying (e.g. in its support of memory mapped IO and DMA) as part of the trade-off for minimising changes necessary to the U-Boot drivers.

3. **U-Boot Live Device Tree**: The decision has been taken for the library to utilise U-Boot's modern [live device tree](https://u-boot.readthedocs.io/en/latest/develop/driver-model/livetree.html) functionality to read device tree properties rather than the historical `fdtdec` interface. Whilst this should result in improved long term support for the library, it may be necessary to make minor changes to a driver when porting an old driver to the live device tree interface. Porting is a simple activity and guidance is provided [here](https://u-boot.readthedocs.io/en/latest/develop/driver-model/livetree.html#porting-drivers).

4. **Interrupt Handling**: The device drivers provided by U-Boot generally do not support interrupt handling; instead they rely on busy-waiting / polling of devices. There is, however, no inherent reason preventing the use of interrupt handlers. Should use of interrupt handling be required then it would need to be added by the user to the driver and the library's API would need to be enhanced to support such functionality.

5. **ARM Power Domains**: ARM power domains are controlled through calls to the ARM Trusted Firmware (ATF). Accessing the ATF from within seL4 is not currently supported due to the need for elevated privileges. It is instead suggested that the power domains should be set as required during execution of the bootloader prior to seL4's startup. On the Avnet MaaXBoard, for example, power domains need to be enabled to power the USB PHY; therefore the USB devices need to be probed from the U-Boot bootloader prior to starting seL4 to ensure they are powered.

## Code Structure

The library code is held within the following structure:

```text
libubootdrivers
│
├───include
│   ├───plat
│   ├───public_api
│   └───wrapper
│
├───src
│   ├───plat
│   ├───timer
│   └───wrapper
│
├───uboot
│
└───uboot_stub
```

The following conventions are maintained for each folder name:

- **plat**: Folders holding platform specific source files. Contains one subfolder per platform.
- **public_api**: Holds header files defining the library's publicly accessible API.
- **wrapper**: Bespoke code written for the library.
- **timer**: Timer driver source code.
- **uboot**: A symlink to an unmodified, or minimally modified, fork of the U-Boot project. Note that the build system is responsible for creation of the symlink; for the [example test applications](uboot_driver_usage.md) the symlink is created by the `repo` tool.
- **uboot_stub**: Holds library specific replacements for U-Boot source code. Internal folder structure mirrors U-Boot code structure.
