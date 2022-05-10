# U-Boot Driver Library Overview

To achieve the goals specified in [device driver introduction](device_driver_intro.md) section an extensible library of device drivers ported form U-Boot has been created. This library provides an extensive set of drivers for the Avnet MaaXBoard platform as a worked example of its usage supported by documentation in the following sections on usage and maintenance of the library and guidance on the extension of the library to support additional platforms and devices.

The remainder of this section provides information on the design and structure of the library. Before attempting to

## Design Summary

The primary goal of the library is to allow drivers from U-boot to be used within seL4 with minimal or no code changes. To support this goal the library is comprised of:

- A set of U-Boot drivers.

- The [Driver Model](https://u-boot.readthedocs.io/en/latest/develop/driver-model/index.html) driver framework and its associated subsystems required to support device drivers.

- Stubbed versions of U-Boot subsystems providing a compatibility / conversion layer between the U-Boot source code and underlying seL4 libraries.

- A wrapper around the U-Boot code to provide an API for users of the library to interact with devices and manage library initialisation / shutdown.

## Detailed Design Information

The following sections provide details on the design of stub or wrapper elements that are significant to support the functioning of the drivers within the Driver Model framework.

### Initialisation

TBC

### Linker Lists

To allow variable levels of functionality to be built into a U-Boot executable, U-Boot utilises a concept of [linker-generated arrays](https://u-boot.readthedocs.io/en/latest/api/linker_lists.html). These are arrays of objects (e.g. drivers, supported commands, driver classes, etc) which are stored by U-Boot within dedicated linker sections to be queried at runtime.

Access to the objects stored within these linker sections is provided by U-Boot through a set of macros defined in the ```linker_lists.h``` header file.

To mimic this functionality within the library the following approach has been taken:

1. Rather than storing the objects within linker sections of the executable, a global variable named ```driver_data``` is declared within ```driver_data.h``` which contains the equivalent object arrays.

2. During initialisation of the library, the platform-dependant contents of the ```driver_data``` global variable are set up.

3. A stubbed version of the ```linker_lists.h``` header file is provided. Rather than accessing data from the executable's linker sections as performed by the original version, the stubbed version accesses data from the arrays within the ```driver_data``` global variable.

### Memory Mapped IO

One of the core mechanisms for providing a software interface to a hardware device is through the use of [memory mapped IO](https://en.wikipedia.org/wiki/Memory-mapped_I/O).

The memory addresses which a U-Boot driver uses for communication with a device can be either read from the device tree (i.e. from a devices ```reg``` property) or in some cases can be hard-coded into the driver. At this pointed it should be noted that U-Boot source code is intended to be executed in an environment with direct access to the system's physical address space whilst, however the library is executed within a virtual address space provided by seL4.

As such the addresses used by U-Boot drivers are *physical* addresses which are not directly accessible by the library. To work around this issue:

1. During library initialisation the list of all platform-dependent devices that need to be accessed must be provided. For each device:
   1. The device's physical address range is read from the device tree.
   2. The device's physical address range is mapped into the virtual address space through use of the IO mapping routines provided by seL4's platform support library.
   3. The mapping between the physical address range and the mapped virtual address range is stored within the ```sel4_io_map``` package (part of the libraries wrapper functionality).

2. When a driver attempts to perform memory mapped IO it calls architecture-dependent routines from the ```io.h``` header, e.g. ```readl``` or ```writel```. A stubbed version of the ```io.h``` header is provided that uses the ```sel4_io_map``` package to translate the physical addresses provided by the driver to the equivalent address mapped into the libraries virtual address space.

Using the approach provided above U-Boot drivers can seamlessly perform memory mapped IO without the need for any modifications being made to the driver itself.


## TODO

- Design decisions

- Major issues to overcome
  - Initialisation
  - "Linker Lists" - DONE
  - Running in a virtual address space
    - Memory mapped IO - DONE
    - DMA
  - Clock / time. Need to provide a monotonic high-frequency clock.

- General structure:
  - Code structure
  - U-Boot code, minimally modified.
  - Stub
    - delay / time.
    - print / debug.
  - Wrapper
    - Mimic of u-boot start-up code.
    - Memory mapping.
    - DMA.



- Linker lists.
- Env / console.
  - Output wrappers around printf / ZF_LOGxxx routines.
- Memory mapped IO.

- Different types of hardware interfaces:

  - Drivers using memory mapped IO with addresses should just work.
  - Drivers requiring DMA may need to be modified. For XHCI USB all of the required modification have been made in the XHCI stack so other drivers should not require changes. Drivers using the DMA interface exported by linux/dma-mapping.h should work; an seL4 specific implementation of this API has been provided by the library. Drivers not using either of these mechanisms will need to be updated to use the API exported by sel_dma.h (see the fec_mxc.c ethernet driver as an example).
  - Interrupt handling is currently not used / supported/

- Limitations:

  - Not thread safe ( uses musclib that are not thread safe).
  - Don’t expect amazing performance.
  - Unable to control ARM power domains (requires calls to the ATF).
  - Only works with drivers that have been updated to work with U-Boot’s “live tree” functionality. Such for uses of devfdt_xxx routines and replace with the equivalent dev_xxx routine. See pinctrl-imx.c for an example.
