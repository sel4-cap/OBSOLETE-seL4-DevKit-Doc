# Worked Example - Adding a Device Driver

A helpful way to show the process of adding a driver is to work through an example step-by-step. The I<sup>2</sup>C driver is relatively straightforward and can be readily tested on the MaaXBoard as there is a power management IC (BD71837MWV) already installed on the board's I<sup>2</sup>C bus. In this worked example, we shall not go as far as installing a driver for the BD71837MWV device itself, but we can probe the I<sup>2</sup>C bus, identify the address of the BD71837MWV device, and perform a sample memory read.

## Establishing the driver

The first place to look is the relevant source code for the driver in U-Boot. Using the example of the I<sup>2</sup>C driver, for the i.MX series (the SoC used in the MaaXBoard), the relevant file is `mxc_i2c.c`, found within the directory at [https://github.com/u-boot/u-boot/tree/master/drivers/i2c](https://github.com/u-boot/u-boot/tree/master/drivers/i2c). This file may be identified as follows.

A `dm tree` command from the U-Boot prompt[^2] should reveal an `i2c` entry, which uses driver `i2c_mxc`.

```text
 Class     Index  Probed  Driver                Name
-----------------------------------------------------------
       ...
 i2c           0  [   ]   i2c_mxc                   |   |-- i2c@30a20000
 i2c           1  [   ]   i2c_mxc                   |   |-- i2c@30a30000
```

[^2]: If the MaaXBoard is powered on and autoboot is interrupted (e.g. press the `Enter` key during the countdown shown in line 37 of the output in the [First Boot](first_boot.md#boot-to-u-boot-prompt) section), then the `dm tree` command may be entered at the U-Boot prompt.

A search for `i2c_mxc` within U-Boot's github repository [https://github.com/u-boot/u-boot](https://github.com/u-boot/u-boot) reveals the file referenced above.

At the end of the file `mxc_i2c.c`, we can see the driver's name and that it relies on the I2C uclass:

```c
U_BOOT_DRIVER(i2c_mxc) = {
    .name = "i2c_mxc",
    .id = UCLASS_I2C,
    ...
```

The I2C uclass is provided by the file `i2c-uclass.c` in the same directory. At the end of `i2c-uclass.c`, we can see that driver and the other relevant drivers:

```c
UCLASS_DRIVER(i2c) = {
    .id = UCLASS_I2C,
    .name = "i2c",
    ...

UCLASS_DRIVER(i2c_generic) = {
    .id = UCLASS_I2C_GENERIC,
    .name = "i2c_generic",
    ...

U_BOOT_DRIVER(i2c_generic_chip_drv) = {
    .name = "i2c_generic_chip_drv",
    .id = UCLASS_I2C_GENERIC,
    ...
```

This provides the driver references that we need to add to `libubootdrivers` via `projects_libs/libubootdrivers/include/plat/maaxboard/plat_driver_data.h`:

```c
/* Define the uclass drivers to be used on this platform */
...
extern struct uclass_driver _u_boot_uclass_driver__i2c;
extern struct uclass_driver _u_boot_uclass_driver__i2c_generic;

/* Define the drivers to be used on this platform */
...
extern struct driver _u_boot_driver__i2c_mxc;
extern struct driver _u_boot_driver__i2c_generic_chip_drv;
...
```

and `projects_libs/libubootdrivers/src/plat/maaxboard/plat_driver_data.c`:

```c
void initialise_driver_data(void) {
    ...
    driver_data.uclass_driver_array[17] = _u_boot_uclass_driver__i2c;
    driver_data.uclass_driver_array[18] = _u_boot_uclass_driver__i2c_generic;
    ...
    driver_data.driver_array[18] = _u_boot_driver__i2c_mxc;
    driver_data.driver_array[19] = _u_boot_driver__i2c_generic_chip_drv;
    ...
```

We can see that the latter change has added to the arrays. In this example, the two arrays had previously extended to elements [16] and [17] respectively. The increased sizes need to be reflected back in `libubootdrivers/include/plat/maaxboard/plat_driver_data.h` by increasing the size counts:

```c
/* Define the number of different driver elements to be used on this platform */
#define _u_boot_uclass_driver_count  19  // In this example previous count was 17, so +2
#define _u_boot_driver_count         20  // In this example previous count was 18, so +2
...
```

We need to add the relevant source code for the i2c drivers from U-Boot, so we create a new `i2c` directory thus: `libubootdrivers/src/uboot/src/drivers/i2c/`. From the U-Boot path that we identified earlier ([https://github.com/u-boot/u-boot/tree/master/drivers/i2c](https://github.com/u-boot/u-boot/tree/master/drivers/i2c)), we copy `mxc_i2c.c` and `i2c-uclass.c` into our new directory.

In order to build these new files, we need to add them to the `libubootdrivers/CMakeLists.txt` makefile. There are also some U-Boot configuration switches relevant to I<sup>2</sup>C that are needed, which are also set within the makefile (see `CONFIG_SYS_*I2C*` definitions below)[^2]. The extract from the makefile that is relevant to I<sup>2</sup>C is shown below.

[^2]: _Working TBD note: How to identify the relevant switches from U-Boot?_

```makefile
############################
# Settings for I2C drivers #
############################

if(i2c_driver MATCHES "none")
    # Nothing to do
else()
    # Enable I2C support
    add_definitions("-DCONFIG_DM_I2C=1")
    # Generic I2C source files
    list(APPEND uboot_deps src/uboot/src/drivers/i2c/i2c-uclass.c)

    # Driver specific settings / files
    if(i2c_driver MATCHES "i2c_mxc")
        add_definitions("-DCONFIG_SYS_I2C_MXC=1")
        add_definitions("-DCONFIG_SYS_I2C_MXC_I2C1=1")
        add_definitions("-DCONFIG_SYS_I2C_MXC_I2C2=1")
        add_definitions("-DCONFIG_SYS_I2C_MXC_I2C3=1")
        add_definitions("-DCONFIG_SYS_I2C_MXC_I2C4=1")
        add_definitions("-DCONFIG_SYS_MXC_I2C1_SPEED=100000")
        add_definitions("-DCONFIG_SYS_MXC_I2C1_SLAVE=0")
        add_definitions("-DCONFIG_SYS_MXC_I2C2_SPEED=100000")
        add_definitions("-DCONFIG_SYS_MXC_I2C2_SLAVE=0")
        add_definitions("-DCONFIG_SYS_MXC_I2C3_SPEED=100000")
        add_definitions("-DCONFIG_SYS_MXC_I2C3_SLAVE=0")
        add_definitions("-DCONFIG_SYS_MXC_I2C4_SPEED=100000")
        add_definitions("-DCONFIG_SYS_MXC_I2C4_SLAVE=0")

        list(APPEND uboot_deps src/uboot/src/drivers/i2c/mxc_i2c.c)
    else()
        message(FATAL_ERROR "Unrecognised I2C driver. Aborting.")
    endif()
endif()
```

We can now try building to identify the extra header files that we need. Within our `build` directory, after `init_build` (to pick up the new files that we have added so far) and `ninja`, we see some missing files that are terminating the compilation:

`... /libubootdrivers/src/uboot/src/drivers/i2c/i2c-uclass.c:23:10: fatal error: acpi_i2c.h: No such file or directory`

and

`... /libubootdrivers/src/uboot/src/drivers/i2c/mxc_i2c.c:25:10: fatal error: asm/mach-imx/mxc_i2c.h: No such file or directory`

We find that `acpi_i2c.h` is at the same U-Boot path as the `.c` files were ([https://github.com/u-boot/u-boot/tree/master/drivers/i2c](https://github.com/u-boot/u-boot/tree/master/drivers/i2c)), so this can be copied to `libubootdrivers/src/uboot/src/drivers/i2c/`.

`mxc_i2c.h` can be found at [https://github.com/u-boot/u-boot/tree/master/arch/arm/include/asm/mach-imx](https://github.com/u-boot/u-boot/tree/master/arch/arm/include/asm/mach-imx) and we copy it to `libubootdrivers/include/uboot/arch/imx8m/asm/mach-imx/`.

Another `ninja` will reveal a further missing header file:

`... /libubootdrivers/include/uboot/arch/imx8m/asm/mach-imx/mxc_i2c.h:8:10: fatal error: asm/mach-imx/iomux-v3.h: No such file or directory`

`iomux-v3.h` is found in and copied to the same locations as `mxc_i2c.h`.

A further `ninja` should result in a clean build.

## Integrating the driver with CAmkES

We have established the underlying driver code, but it is not yet integrated within the CAmkES component that we shall be using. Assuming that we use the `uboot-driver-example` test application introduced earlier – see [Using the U-Boot Driver Library](uboot_driver_usage.md#instructions-for-running-the-uboot-driver-example-test) – we need to modify the file `camkes/apps/uboot-driver-example/include/plat/maaxboard/platform_devices.h` as follows.

Firstly, we need to add path definitions so that the devices can be located in the device tree:

```c
#define I2C_0_PATH      "/soc@0/bus@30800000/i2c@30a20000"
#define I2C_1_PATH      "/soc@0/bus@30800000/i2c@30a30000"
#define I2C_2_PATH      "/soc@0/bus@30800000/i2c@30a40000"
#define I2C_3_PATH      "/soc@0/bus@30800000/i2c@30a50000"
```

These need to be added to `DEVICE_PATHS` and then `DEVICE_PATHS_LENGTH` should be modified accordingly:

```c
#define DEVICE_PATHS {  \
    ...                 \
    I2C_0_PATH,         \
    I2C_1_PATH,         \
    I2C_2_PATH,         \
    I2C_3_PATH,         \
    ...
    };
#define DEVICE_PATHS_LENGTH 22 // In this example previous size was 18 so +4
```

Entries for the devices need to be added to `HARDWARE_INTERFACES`:

```c
#define HARDWARE_INTERFACES  \
    ...                      \
    consumes Dummy i2c_0;    \
    consumes Dummy i2c_1;    \
    consumes Dummy i2c_2;    \
    consumes Dummy i2c_3;    \
    ...                      \
    emits Dummy dummy_source;
```

And also added to `HARDWARE_COMPOSITION`:

```c
#define HARDWARE_COMPOSITION                                             \
    ...                                                                  \
    connection seL4DTBHardware i2c_0_conn(from dummy_source, to i2c_0);  \
    connection seL4DTBHardware i2c_1_conn(from dummy_source, to i2c_1);  \
    connection seL4DTBHardware i2c_2_conn(from dummy_source, to i2c_2);  \
    connection seL4DTBHardware i2c_3_conn(from dummy_source, to i2c_3);  \
    ...
```

And also added to `HARDWARE_CONFIGURATION`:

```c
#define HARDWARE_CONFIGURATION                                                  \
    ...                                            \
    i2c_0.dtb     = dtb({ "path" : I2C_0_PATH });  \
    i2c_1.dtb     = dtb({ "path" : I2C_1_PATH });  \
    i2c_2.dtb     = dtb({ "path" : I2C_2_PATH });  \
    i2c_3.dtb     = dtb({ "path" : I2C_3_PATH });  \
    ...
```

## Establishing the driver API

We now need to work on the driver's API. We are going to access the driver via the U-Boot `i2c` command, such as is available at the U-Boot command line:

```text
u-boot=> i2c
i2c - I2C sub-system

Usage:
i2c bus [muxtype:muxaddr:muxchannel] - show I2C bus info
i2c crc32 chip address[.0, .1, .2] count - compute CRC32 checksum
i2c dev [dev] - show or set current I2C bus
i2c loop chip address[.0, .1, .2] [# of objects] - looping read of device
i2c md chip address[.0, .1, .2] [# of objects] - read from I2C device
i2c mm chip address[.0, .1, .2] - write to I2C device (auto-incrementing)
i2c mw chip address[.0, .1, .2] value [count] - write to I2C device (fill)
i2c nm chip address[.0, .1, .2] - write to I2C device (constant address)
i2c probe [address] - test for and show device(s) on the I2C bus
i2c read chip address[.0, .1, .2] length memaddress - read to memory
i2c write memaddress chip address[.0, .1, .2] length [-s] - write memory
          to I2C; the -s option selects bulk write in a single transaction
i2c flags chip [flags] - set or get chip flags
i2c olen chip [offset_length] - set or get chip offset length
i2c reset - re-init the I2C Controller
i2c speed [speed] - show or set I2C bus speed
u-boot=>
```

We return to `libubootdrivers/src/plat/maaxboard/plat_driver_data.c` to add a new entry to the `cmd_array`:

```c
void initialise_driver_data(void) {
    ...
    driver_data.cmd_array[16] = _u_boot_cmd__i2c;
    ...
```

Corresponding changes are made in `libubootdrivers/include/plat/maaxboard/plat_driver_data.h`:

```c
/* Define the number of different driver elements to be used on this platform */
...
#define _u_boot_cmd_count  17  // In this example previous count was 16, so +1
...

/* Define the u-boot commands to be used on this platform */
...c
extern struct cmd_tbl _u_boot_cmd__i2c;
...
```

We need to import the U-Boot source code for the `i2c` command, copying the file `i2c.c` from [https://github.com/u-boot/u-boot/tree/master/cmd](https://github.com/u-boot/u-boot/tree/master/cmd) to `libubootdrivers/src/uboot/src/cmd/`.

As with the previous addition of `.c` files, this needs to be added to the `libubootdrivers/CMakeLists.txt` makefile, along with a CMD configuration switch. The earlier extract from the makefile is repeated below, with two additional lines marked.

```makefile
    ############################
    # Settings for I2C drivers #
    ############################

    if(i2c_driver MATCHES "none")
        # Nothing to do
    else()
        # Enable I2C support
+       add_definitions("-DCONFIG_CMD_I2C=1")
        add_definitions("-DCONFIG_DM_I2C=1")
        # Generic I2C source files
+       list(APPEND uboot_deps src/uboot/src/cmd/i2c.c)
        list(APPEND uboot_deps src/uboot/src/drivers/i2c/i2c-uclass.c)

        # Driver specific settings / files
        if(i2c_driver MATCHES "i2c_mxc")
            add_definitions("-DCONFIG_SYS_I2C_MXC=1")
            add_definitions("-DCONFIG_SYS_I2C_MXC_I2C1=1")
            add_definitions("-DCONFIG_SYS_I2C_MXC_I2C2=1")
            add_definitions("-DCONFIG_SYS_I2C_MXC_I2C3=1")
            add_definitions("-DCONFIG_SYS_I2C_MXC_I2C4=1")
            add_definitions("-DCONFIG_SYS_MXC_I2C1_SPEED=100000")
            add_definitions("-DCONFIG_SYS_MXC_I2C1_SLAVE=0")
            add_definitions("-DCONFIG_SYS_MXC_I2C2_SPEED=100000")
            add_definitions("-DCONFIG_SYS_MXC_I2C2_SLAVE=0")
            add_definitions("-DCONFIG_SYS_MXC_I2C3_SPEED=100000")
            add_definitions("-DCONFIG_SYS_MXC_I2C3_SLAVE=0")
            add_definitions("-DCONFIG_SYS_MXC_I2C4_SPEED=100000")
            add_definitions("-DCONFIG_SYS_MXC_I2C4_SLAVE=0")

            list(APPEND uboot_deps src/uboot/src/drivers/i2c/mxc_i2c.c)
        else()
            message(FATAL_ERROR "Unrecognised I2C driver. Aborting.")
        endif()
    endif()
```

We can again try building to identify any extra header files that we need. Within our `build` directory, after `init_build` and `ninja`, we see a missing file that is terminating the compilation:

`... /libubootdrivers/src/uboot/src/cmd/i2c.c:73:10: fatal error: edid.h: No such file or directory`

The missing `edid.h` file can be copied from [https://github.com/u-boot/u-boot/tree/master/include](https://github.com/u-boot/u-boot/tree/master/include) to `projects_libs/libubootdrivers/include/uboot/include/`.

This should then build successfully.

## Testing the driver

Now that we have the I<sup>2</sup>C driver and its API, we can call `i2c` commands from our test application. For example, the following lines can be used within a CAmkES component (in our example, these would be added to file `camkes/apps/uboot-driver-example/components/Test/src/test.c`):

```c
    // I2C test
    run_uboot_command("i2c dev 0"); // Set current i2c bus to zero
    run_uboot_command("i2c probe"); // Probe i2c bus 0
```

On the MaaXBoard, the `i2c probe` command returns `Valid chip addresses: 4B`, which is the address of the BD71837MWV power management IC on the bus. Extending the test case as follows:

```c
    // I2C test
    run_uboot_command("i2c dev 0"); // Set current i2c bus to zero
    run_uboot_command("i2c probe"); // Probe i2c bus 0
    run_uboot_command("i2c md 0x4b 0x0.1 0x20"); // Read 32 bytes from device at address 0x4b
    run_uboot_command("dm tree");
```

reads the first 32 bytes from the BD71837MWV, displaying:

```text
0000: a3 04 03 a2 00 40 40 44 44 00 00 00 00 14 14 14    .....@@DD.......
0010: 1e 14 1e 1e 02 03 03 28 03 00 00 00 00 00 0f 48    .......(.......H
```

Before the `i2c probe`, a `dm tree` command would reveal the presence of the `i2c_mxc` driver:

```text
 Class     Index  Probed  Driver                Name
-----------------------------------------------------------
       ...
 i2c           0  [   ]   i2c_mxc               |   |   |-- i2c@30a20000
 i2c           1  [   ]   i2c_mxc               |   |   |-- i2c@30a30000
 i2c           2  [   ]   i2c_mxc               |   |   |-- i2c@30a40000
 i2c           3  [   ]   i2c_mxc               |   |   |-- i2c@30a50000
```

Following the `i2c probe`, the `dm tree` command shows the instantiation of the `i2c_generic_chip_drv` driver that we ported earlier:

```text
 Class     Index  Probed  Driver                Name
-----------------------------------------------------------
       ...
 i2c           0  [ + ]   i2c_mxc               |   |   |-- i2c@30a20000
 i2c_generi    0  [ + ]   i2c_generic_chip_drv  |   |   |   `-- generic_4b
 i2c           1  [   ]   i2c_mxc               |   |   |-- i2c@30a30000
 i2c           2  [   ]   i2c_mxc               |   |   |-- i2c@30a40000
 i2c           3  [   ]   i2c_mxc               |   |   |-- i2c@30a50000
```