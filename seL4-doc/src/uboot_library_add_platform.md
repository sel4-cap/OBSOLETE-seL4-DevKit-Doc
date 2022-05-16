# Library Extension - New Platform

This section documents the required actions and guidance for adding support for a new platform to the library.

By the end of this section an seL4 executable will be built for the new platform that can initialise the library (although the library may not support any of the platform's devices). Later sections of this guide cover the required actions to add driver support into the library for the new platform.

## Add basic support to library

To allow the library to be successfully compiled for a new platform the following changes will need to be made to the library:

### Update the library's CMake file to support the platform

The library's `CMakeLists.txt` file contains a The section titled ```Platform specific settings``` to control the settings for each platform. This section:

1. Declares a set of variables to control which drivers and optional capabilities are to be built for each platform. The defaults for these variables will produce a build including only a dummy timer driver, this is the minimum necessary to allow the library to be built.

2. Provides a conditional block of settings associated with each supported platform, e.g. to control which of the supported drivers to build. If no conditional block is supplied for a platform the compilation of the library will fail and an error message will be returned.

The minimal changes changes to this section to allow the library to be built is therefore the inclusion of a conditional block for the platform, even if that section makes to changes from the default settings. For example, to add support for a platform that is named  ```foo``` within the seL4 build system, the following changes would be required:

```cmake
    # Set up the applicable drivers and platform dependent configuration.
    if(KernelPlatImx8mq)
        ...
+   elseif("${KernelPlatform}" STREQUAL "foo") )
+       # Platform specific settings for the Foo board.
    else()
        message(FATAL_ERROR "Unsupported platform. Aborting.")
    endif()
```

Note that where settings are shared across multiple platforms this logic can be simplified to reflect the shared settings. An example of this is provided in the library's `CMakeLists.txt` for the ```maaxboard``` and ```imx8mq-evk``` platforms which both use the iMX8MQ SoC.

It should also be noted that U-Boot drivers can support multiple devices. For example, many of the drivers added to support the Avnet MaaXBoard support multiple iMX SoCs. If previously supported drivers are compatible wih the new platform this support can be added now, for example if platform `foo` has a GPIO device supported by the `gpio_mxc` driver (as used by the Avnet MaaXBoard), then the line ```set(gpio_driver "gpio_mxc")``` could be added to the `foo` platform's settings to enable support.

### Define platform specific Linker Lists data structure

As documented within the [Linker Lists section of the library overview](uboot_driver_library.md#linker-lists), the library requires a global data structure named ```driver_data``` to be declared to allow the U-Boot source code to access optional functionality included in the executable.

To define ```driver_data``` for the new platform new platform specific files must be added to the library. The code blocks that follow provide minimal templates for these files that include the base device classes, drivers for those classes, and base commands that are required by all platforms.

File `inlcude/plat/foo/plat_driver_data.h`:

```c
/*
 * This file defines which drivers, driver classes, driver entries and commands
 * are to be included in the compiled library for this platform.
 *
 * This allows only the drivers compatible with the targeted platform to be
 * included with all non-compatible drivers excluded.
 *
 * It should be noted that some of these are fundamental to allowing the U-Boot
 * driver model to function (e.g. the nop, root and simple bus drivers).
 */

/* Define the number of different driver elements to be used on this platform */
#define _u_boot_uclass_driver_count     5
#define _u_boot_driver_count            2
#define _u_boot_usb_driver_entry_count  0
#define _u_boot_part_driver_count       0
#define _u_boot_cmd_count               3
#define _u_boot_env_driver_count        0
#define _u_boot_env_clbk_count          0
#define _u_boot_driver_info_count       0
#define _u_boot_udevice_count           0

/* Define the uclass drivers to be used on this platform */
extern struct uclass_driver _u_boot_uclass_driver__nop;
extern struct uclass_driver _u_boot_uclass_driver__root;
extern struct uclass_driver _u_boot_uclass_driver__simple_bus;
extern struct uclass_driver _u_boot_uclass_driver__phy;
extern struct uclass_driver _u_boot_uclass_driver__blk;

/* Define the drivers to be used on this platform */
extern struct driver _u_boot_driver__root_driver;
extern struct driver _u_boot_driver__simple_bus;

/* Define the driver entries to be used on this platform */

/* Define the disk partition types to be used */

/* Define the u-boot commands to be used on this platform */
extern struct cmd_tbl _u_boot_cmd__dm;
extern struct cmd_tbl _u_boot_cmd__env;
extern struct cmd_tbl _u_boot_cmd__setenv;

/* Define the u-boot environment variables callbacks to be used on this platform */
```

File `src/plat/foo/plat_driver_data.c`:

```c
#include <uboot_helper.h>

#include <dm/device.h>
#include <dm/uclass.h>
#include <dm/platdata.h>
#include <usb.h>
#include <part.h>

void initialise_driver_data(void) {
    driver_data.uclass_driver_array[0]  = _u_boot_uclass_driver__nop;
    driver_data.uclass_driver_array[1]  = _u_boot_uclass_driver__root;
    driver_data.uclass_driver_array[2]  = _u_boot_uclass_driver__simple_bus;
    driver_data.uclass_driver_array[3]  = _u_boot_uclass_driver__phy;
    driver_data.uclass_driver_array[4]  = _u_boot_uclass_driver__blk;

    driver_data.driver_array[0]  = _u_boot_driver__root_driver;
    driver_data.driver_array[1]  = _u_boot_driver__simple_bus;

    driver_data.cmd_array[0]  = _u_boot_cmd__dm;
    driver_data.cmd_array[2]  = _u_boot_cmd__env;
    driver_data.cmd_array[3]  = _u_boot_cmd__setenv;
}
```

As support for additional optional objects (e.g. drivers, driver classes, commands, etc) are added for a platform these files will be updated to reference the associated objects. See the files for the Avnet MaaXBoard platform for an example of these files providing more extensive support.

## Add support to example application

Section [Using the U-Boot Driver Library](uboot_driver_usage.md) introduced demonstration application  `uboot-driver-example`. This section documents the changes required to the `uboot-driver-example` application to allow it to be built for the new platform.

The following empty template file needs to be added to the application `inlcude/plat/platform_devices.h`:

```c
#pragma once

#define DEVICE_PATHS {};
#define DEVICE_PATHS_LENGTH 0

#define HARDWARE_INTERFACES

#define HARDWARE_COMPOSITION

#define HARDWARE_CONFIGURATION
```

As support for devices are added for a platform this file will be updated to reference those devices from the platform's device tree. See the file for the Avnet MaaXBoard platform for an example providing more extensive support.

## TODO

Demonstrate how to add a new platform and build the example application.

- Example application:
  - Add folder to include/plat for the new platform.
  - ```platform_devices.h``` provide an empty file.

- Library:
  - Update the CMake file (currently any KernelPlatImx8mq based devices)
  - Add platform specific folders to include/plat and src/plat. Can use the existing entry for 'maaxboard' as a template.
    - ```plat_uboot_config.h``` is expected to start empty and be added to as drivers are added.
    - ```plat_driver_data.h``` provide an empty example file.
    - ```plat_driver_data.c``` provide an empty example file.
  - Need a monotonic clock driver.
    - Needs to provide the same routines as the timer provided for the maaxboard (see ```timer_imx8mq.c```)
  - Perform a build of a simple test app that just initialises then shuts down the library (provide example).
    - Add any required platform specific headers and source files into the existing structure based upon any resulting compiler and linker errors.
  - When everything runs fine progress to adding drivers.
    - Suggested order of drivers. The existence of clock, GPIO and iomux drivers is expected and relied upon by most drivers (provide examples) so suggest adding these first. Some basic drivers can be supported without any of these or through use of workarounds (e.g. setup performed by the bootloader).
  - Add in support from existing drivers.

- New architecture? Already covered ARM.