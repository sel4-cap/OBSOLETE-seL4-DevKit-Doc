# Library Extension - New Driver

This section provides guidance on the addition of a new device driver to support a device on an existing platform.

Adding support for a new device can be broken down into logical steps:

1. [Identification of U-Boot Device Driver](#identification-of-u-boot-device-driver)

2. [Updating the library `CMakeLists.txt` to support the device driver](#updating-cmakeliststxt)

3. [Associating the device driver with an existing platform](#associating-driver-with-platform)

4. [Resolving compilation issues](#resolving-compilation-issues)

5. [Updating an seL4 application to access the device](#updating-sel4-application)

To further support this topic, appendices have been added to provide [a worked example](./appendices/add_driver_worked_example.md) and to discuss [troubleshooting and common issues](./appendices/driver_troubleshooting.md) that may be encountered.

## Identification of U-Boot Device Driver

The first step in adding a new device driver to the library is to positively identify the U-Boot source file providing the required support. U-Boot declares drivers through use of a macro of the following format:

```c
U_BOOT_DRIVER(<driver_id>) = {
    .name = "<driver_name>",
    .id = UCLASS_<driver_uclass_id>,
    ...
};
```

An example of such a declaration from the GPIO driver for the Avnet MaaXBoard can be found in the U-Boot source code within file `drivers/gpio/mxc_gpio.c`.

If a build of U-Boot with support for the desired device is available, the name of the device driver can be obtained through use of U-Boot's `dm tree` command.

For example, to determine the GPIO device driver for a platform the following `dm tree` output would identify that the U-Boot driver with a `<driver_name>` of `gpio_mxc` provides the desired support. Note that the name of the device (as per the platform's device tree) is provided in the `Name` column whilst the `<driver_name>` is provided in the `Driver` column.

```text
u-boot=> dm tree
 Class     Index  Probed  Driver                Name
-----------------------------------------------------------
 root          0  [ + ]   root_driver           root_driver
 simple_bus    0  [ + ]   simple_bus            `-- soc@0
 simple_bus    1  [ + ]   simple_bus                |-- bus@30000000
 gpio          0  [ + ]   gpio_mxc                  |   |-- gpio@30200000
 gpio          1  [   ]   gpio_mxc                  |   |-- gpio@30210000
 gpio          2  [   ]   gpio_mxc                  |   |-- gpio@30220000
 gpio          3  [   ]   gpio_mxc                  |   |-- gpio@30230000
 gpio          4  [   ]   gpio_mxc                  |   |-- gpio@30240000
 ...
```

If no such build of U-Boot is available, the device driver can be obtained through manual matching of the `compatible` strings between the platform's device tree and a device driver.

For example, from the following device tree excerpt it can be seen that the GPIO device is compatible with a device driver that advertises compatibility to either `fsl,imx8mq-gpio` or `fsl,imx35-gpio` devices.

```text
gpio@30200000 {
    compatible = "fsl,imx8mq-gpio\0fsl,imx35-gpio";
    ...
};
```

Performing a textual search for these compatibility strings within the U-Boot source code provides the following match for the `fsl,imx35-gpio` compatibility string:

```c
static const struct udevice_id mxc_gpio_ids[] = {
    { .compatible = "fsl,imx35-gpio" },
    { }
};

U_BOOT_DRIVER(gpio_mxc) = {
    .name = "gpio_mxc",
    .id = UCLASS_GPIO,
    ...
    .of_match = mxc_gpio_ids,
    ...
};
```

Through either method, the U-Boot source file declaring the driver can be identified.

## Updating `CMakeLists.txt`

Once the source file declaring the driver has been established, this information can be captured in the library CMake file (`CMakeLists.txt`).

The CMake file contains sections devoted to each class of driver (also known as a `UCLASS` in U-Boot), e.g. Ethernet drivers, MMC drivers, etc. If the driver to be added is of a previously unsupported class then a new section will need to be added; this will typically have the following structure (using the GPIO driver class as an example supporting a single driver):

```makefile
#############################
# Settings for GPIO drivers #
#############################

if(gpio_driver MATCHES "none")
    # Nothing to do
else()
    # Enable GPIO support
    add_definitions("-DCONFIG_DM_GPIO=1")
    add_definitions("-DCONFIG_GPIO_EXTRA_HEADER=1")
    add_definitions("-DCONFIG_CMD_GPIO_READ=1")
    # Generic GPIO source files
    list(APPEND uboot_deps uboot/cmd/gpio.c)
    list(APPEND uboot_deps uboot/drivers/gpio/gpio-uclass.c)

    # Driver specific settings / files
    if(gpio_driver MATCHES "gpio_mxc")
        list(APPEND uboot_deps uboot/drivers/gpio/mxc_gpio.c)
    else()
        message(FATAL_ERROR "Unrecognised GPIO driver. Aborting.")
    endif()
endif()
```

This structure allows configuration settings to be defined, such as configuration macros and source files, which are set regardless of the driver to be used, as well as configuration settings specific to each driver.

To identify the required configuration macros (using `add_definitions("-D<macro_name>=1")`) it is suggested that the macros generated by a build of U-Boot are examined (contained in the `.config` file following configuration of U-Boot). If a build of U-Boot is unavailable, the `defconfig` file for the platform can be examined (see U-Boot files `config/<platform>_defconfig`). Alternatively, how the configuration macros themselves are used within the U-Boot source code can be examined (see instances of `#if ...`).

For a new driver class, it is recommended that two source files are listed for all drivers (using `list(APPEND uboot_deps <file>)`):

1. The source file declaring U-Boot commands related to the driver class. For example, U-Boot commands related to GPIO are stored in the `uboot/cmd/gpio.c` file.
2. The source file declaring the UCLASS driver for the driver class. A UCLASS driver is declared in U-Boot using the `UCLASS_DRIVER` macro. For example, the U-Boot GPIO UCLASS driver is declared in the `uboot/drivers/gpio/gpio-uclass.c` file.

For a new driver it is recommended that the single source file declaring the driver (as identified in the [previous step](#identification-of-u-boot-device-driver)) is set.

It is accepted at this stage that the set of source files may not be complete; this is resolved in a later step.

Note that by convention, the driver naming used in `CMakeLists.txt` matches the driver name used by U-Boot, e.g. in its `dm tree` output (e.g. `gpio_mxc`).

## Associating Driver With Platform

To associate the driver with the platform, the following changes need to be made.

### `CMakeLists.txt`

Within the platform specific section of the library `CMakeLists.txt` file, the variable holding the selected driver for the driver class must be set. For example, if a GPIO driver named `gpio_foo` has been added for platform `foo` then the following would be added:

```makefile
    # Set up the applicable drivers and platform dependent configuration.
    if(KernelPlatImx8mq)
        ...
    elseif("${KernelPlatform}" STREQUAL "foo")
        # Platform specific settings for the Foo board.
+       set(gpio_driver "gpio_foo")
    else()
        message(FATAL_ERROR "Unsupported platform. Aborting.")
    endif()
```

### `plat_driver_data.h`

The `plat_driver_data.h` and `plat_driver_data.c` files are used to create arrays of objects, such as drivers or commands, that U-Boot normally stores in dedicated linker sections (see [documentation on linker lists](uboot_driver_library.md#linker-lists) for details). The objects are declared in U-Boot source files via macros of the form `U_BOOT_<object_type> = { ... };`. All objects that the library needs to access must be declared and enumerated in  `plat_driver_data.h` and added to the `driver_data` structure within `plat_driver_data.c`.

Within the platform's `include/plat/<platform>/plat_driver_data.h` file, the new objects from the added source files (e.g. driver, UCLASS driver, commands, etc.) need to be declared, and the counts of each class of object needs to be updated. It should be noted that sometimes multiple drivers or UCLASS drivers are required to support a single device.

For example, if a GPIO driver were to be added then it is expected that the GPIO driver, the GPIO UCLASS driver, and the GPIO command objects would be added as follows:

```c
extern struct uclass_driver _u_boot_uclass_driver__gpio;
extern struct driver        _u_boot_driver__gpio_foo;
extern struct cmd_tbl       _u_boot_cmd__gpio;
```

Each declaration is of the form `extern struct <type> _u_boot_<class>__<name>` to match the declaration generated by the expansion of the `U_BOOT_<object_Type>` macro, where:

- `<name>` is the parameter provided to the macro that declared the object, e.g. a driver is declared with `U_BOOT_DRIVER(<name>)`;
- `<type>` and `<class>` directly correspond to the type of object. Worked examples of the correct format for each object type are provided in the Avnet MaaXBoard's `plat_driver_data.h` file. Alternatively, the format for each type of object can be determined from the expansion of the macro for that object type; e.g. for a driver, see the expansion of `#define U_BOOT_DRIVER`.

### `plat_driver_data.c`

Within the platform's `include/plat/<platform>/plat_driver_data.c` file, the objects declared in the `plat_driver_data.h` file need to be added to the `driver_data` global structure. A worked example covering multiple object types is provided in the Avnet MaaXBoard's `plat_driver_data.c` file.

## Resolving Compilation Issues

At this stage, compilation of the library should be attempted, e.g. through compilation of the `uboot-driver-example` test application.

It is expected that the set of source files referenced in the library `CMakeLists.txt` may not be complete, leading to compilation or linker errors;  whilst the source files declaring the driver, UCLASS driver and commands have been added, these may rely upon routines from source files not currently referenced.

For each error some judgement needs to be made:

- If the required routine is fundamental to correct operation of the driver (this is the normal case) then the source file should be added to the `CMakeLists.txt` section for the driver class. It must be determined by the developer whether the source file is likely to be required by all drivers of the driver class or is specific to the driver.

- If the required routine is not required, e.g. it can be seen that it should never be called or a 'null' implementation would be sufficient, then the routine can be added to the wrapper file `src/wrapper/unimplemented.c`.

- If an alternative implementation of the routine / source file needs to be provided, e.g. because the original implementation is incompatible with use within the context of an seL4 user-mode application, then a stub version of the source file can be added to the `uboot_stub` folder.

## Updating seL4 Application

At this stage, the library with the newly added driver should compile cleanly.

What remains is to update the application utilising the library to:

1. Ensure that seL4 permits access to the necessary devices from the platform's device tree; this is performed through the configuration of the application's CAmkES project file. For a worked example, see the  `uboot-driver-example.camkes` file from the `uboot-driver-example` [test application](uboot_driver_usage.md#test-application-uboot-driver-example).

2. Provide the identity of those devices from the platform's device tree to the library; this is performed by providing the names of the devices when initialising the library through the `initialise_uboot_drivers` interface on the library's public API. Note that only those devices from the device tree listed in the call to `initialise_uboot_drivers` will be used by the library; all other devices in the device tree will be considered to be disabled.

## Appendices

- [Worked Example - Adding a Device Driver](./appendices/add_driver_worked_example.md)
- [Library Extension - Troubleshooting](./appendices/driver_troubleshooting.md)
