# Library Extension - New Driver

_This may not be the right structure. For now, just some markdown notes of content that will end up somewhere._

_Expect we need to cover to following:_
- Known problem: DMA
- Known problem: Live tree
- Talk about how to determine the required config macros. Look at .config generated by a build of u-boot.
- Guidance on how to identify the correct driver within U-Boot.

## Worked example - I<sup>2</sup>C

A helpful way to show the process of adding a driver is to work through an example step-by-step. The I<sup>2</sup>C driver is relatively straightforward and can be readily tested on the MaaXBoard as there is a power management IC (BD71837MWV) already installed on the board's I<sup>2</sup>C bus. In this worked example, we shall not go as far as installing a driver for the BD71837MWV itself, but we can probe the I<sup>2</sup>C bus, identify the address of the BD71837MWV, and perform a sample memory read.

### Establishing the driver

The first place to look is the relevant source code for the driver in U-Boot. Using the example of the I<sup>2</sup>C driver, for the i.MX series (the SoC used in the MaaXBoard), the relevant file is `mxc_i2c.c`, found within the directory at [https://github.com/u-boot/u-boot/tree/master/drivers/i2c](https://github.com/u-boot/u-boot/tree/master/drivers/i2c)

At the end of the file `mxc_i2c.c`, we can see the driver's name and that it relies on the I2C uclass:
```
U_BOOT_DRIVER(i2c_mxc) = {
    .name = "i2c_mxc",
    .id = UCLASS_I2C,
    ...
```

The I2C uclass is provided by the file `i2c-uclass.c` in the same directory. At the end of `i2c-uclass.c`, we can see that driver and the other relevant drivers:
```
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

```
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

```
void initialise_driver_data(void) {
    ...
    driver_data.uclass_driver_array[17] = _u_boot_uclass_driver__i2c;
    driver_data.uclass_driver_array[18] = _u_boot_uclass_driver__i2c_generic;
    ...
    driver_data.driver_array[18] = _u_boot_driver__i2c_mxc;
    driver_data.driver_array[19] = _u_boot_driver__i2c_generic_chip_drv;
    ...
```

We can see that the latter change has added to the arrays. In this example, the two arrays had previously extended to elements [16] and [17] respectively. The increased sizes need to be reflected back in `projects_libs/libubootdrivers/include/plat/maaxboard/plat_driver_data.h` by increasing the size counts:

```
/* Define the number of different driver elements to be used on this platform */
#define _u_boot_uclass_driver_count  19  // In this example previous count was 17, so +2
#define _u_boot_driver_count         20  // In this example previous count was 18, so +2
...
```

We need to add the relevant source code for the i2c drivers from U-Boot, so we create a new `i2c` directory thus: `project_libs/libubootdrivers/src/uboot/src/drivers/i2c/`. From the U-Boot path that we identified earlier ([https://github.com/u-boot/u-boot/tree/master/drivers/i2c](https://github.com/u-boot/u-boot/tree/master/drivers/i2c)), we copy `mxc_i2c.c` and `i2c-uclass.c` into our new directory.

We can now try building to identify the extra header files that we need. Within our `build` directory, after `init_build` (to pick up the new files that we have added so far) and `ninja`, we see some missing files that are terminating the compilation:

`... /projects_libs/libubootdrivers/src/uboot/src/drivers/i2c/i2c-uclass.c:23:10: fatal error: acpi_i2c.h: No such file or directory`

and

`... /projects_libs/libubootdrivers/src/uboot/src/drivers/i2c/mxc_i2c.c:25:10: fatal error: asm/mach-imx/mxc_i2c.h: No such file or directory`

We find that `acpi_i2c.h` is at the same U-Boot path as the `.c` files were ([https://github.com/u-boot/u-boot/tree/master/drivers/i2c](https://github.com/u-boot/u-boot/tree/master/drivers/i2c)), so this can be copied to `project_libs/libubootdrivers/src/uboot/src/drivers/i2c/`.

`mxc_i2c.h` can be found at [https://github.com/u-boot/u-boot/tree/master/arch/arm/include/asm/mach-imx](https://github.com/u-boot/u-boot/tree/master/arch/arm/include/asm/mach-imx) and we copy it to `project_libs/libubootdrivers/include/uboot/arch/imx8m/asm/mach-imx/`.

Another `ninja` will reveal a further missing header file:

`... /projects_libs/libubootdrivers/include/uboot/arch/imx8m/asm/mach-imx/mxc_i2c.h:8:10: fatal error: asm/mach-imx/iomux-v3.h: No such file or directory`

`iomux-v3.h` is found in and copied to the same locations as `mxc_i2c.h`.

A further `ninja` compiles but generates a linker error for an undefined reference to `_u_boot_driver__i2c_mxc` in `plat_driver_data.c`, an addition we made earlier. There are some U-Boot configuration switches that we need to set. The following are related to I<sup>2</sup>C and we add these lines to `projects_libs/libubootdrivers/include/plat/maaxboard/plat_uboot_config.h`:

```
/* Enable I2C */
#define CONFIG_DM_I2C                   1
#define CONFIG_CMD_I2C                  1
#define CONFIG_SYS_I2C_MXC              1
#define CONFIG_SYS_I2C_MXC_I2C1         1
#define CONFIG_SYS_I2C_MXC_I2C2         1
#define CONFIG_SYS_I2C_MXC_I2C3         1
#define CONFIG_SYS_I2C_MXC_I2C4         1
```

This results in a successful build for which a `dm tree` command reveals the presence of the `i2c_mxc` driver:

```
 Class     Index  Probed  Driver                Name
-----------------------------------------------------------
       ...
 i2c           0  [   ]   i2c_mxc               |   |   |-- i2c@30a20000
 i2c           1  [   ]   i2c_mxc               |   |   |-- i2c@30a30000
 i2c           2  [   ]   i2c_mxc               |   |   |-- i2c@30a40000
 i2c           3  [   ]   i2c_mxc               |   |   |-- i2c@30a50000
```

### Establishing the driver API

This has established the driver but now we need to work on its API. We are going to access the driver via the U-Boot `i2c` command, such as is available at a U-Boot command line:

```
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

We return to `projects_libs/libubootdrivers/src/plat/maaxboard/plat_driver_data.c` to add a new entry to the `cmd_array`:

```
void initialise_driver_data(void) {
    ...
    driver_data.cmd_array[16] = _u_boot_cmd__i2c;
    ...
```

Corresponding changes are made in `projects_libs/libubootdrivers/include/plat/maaxboard/plat_driver_data.h`:

```
/* Define the number of different driver elements to be used on this platform */
...
#define _u_boot_cmd_count  17  // In this example previous count was 16, so +1
...

/* Define the u-boot commands to be used on this platform */
...
extern struct cmd_tbl _u_boot_cmd__i2c;
...
```

We need to import the U-Boot source code for the i2c command, copying the file `i2c.c` from [https://github.com/u-boot/u-boot/tree/master/cmd](https://github.com/u-boot/u-boot/tree/master/cmd) to `projects_libs/libubootdrivers/src/uboot/src/cmd/`.

We can now try building to identify any extra header files that we need. Within our `build` directory, after `init_build` and `ninja`, we see a missing file that is terminating the compilation:

`... /projects_libs/libubootdrivers/src/uboot/src/cmd/i2c.c:73:10: fatal error: edid.h: No such file or directory`

The missing `edid.h` file can be copied from [https://github.com/u-boot/u-boot/tree/master/include](https://github.com/u-boot/u-boot/tree/master/include) to `projects_libs/libubootdrivers/include/uboot/include/`.

This should then build successfully.

### Testing the driver

Now that we have the I<sup>2</sup>C driver and its API, we can call `i2c` commands from our test program. For example, the following lines can be used within a CAmkES component:

```
    // I2C test
    run_uboot_command("i2c dev 0"); // Set current i2c bus to zero
    run_uboot_command("i2c probe"); // Probe i2c bus 0
```

On the MaaXBoard, the `i2c probe` command returns `Valid chip addresses: 4B`, which is the address of the BD71837MWV power management IC on the bus. Extending the test case as follows:

```
    // I2C test
    run_uboot_command("i2c dev 0"); // Set current i2c bus to zero
    run_uboot_command("i2c probe"); // Probe i2c bus 0
    run_uboot_command("i2c md 0x4b 0x0.1 0x20"); // Read 32 bytes from device at address 0x4b
    run_uboot_command("dm tree");
```

reads the first 32 bytes from the BD71837MWV, displaying:

```
0000: a3 04 03 a2 00 40 40 44 44 00 00 00 00 14 14 14    .....@@DD.......
0010: 1e 14 1e 1e 02 03 03 28 03 00 00 00 00 00 0f 48    .......(.......H
```

and following the `i2c probe` the `dm tree` command shows the instantiation of the `i2c_generic_chip_drv` driver that we ported earlier:

```
 Class     Index  Probed  Driver                Name
-----------------------------------------------------------
       ...
 i2c           0  [ + ]   i2c_mxc               |   |   |-- i2c@30a20000
 i2c_generi    0  [ + ]   i2c_generic_chip_drv  |   |   |   `-- generic_4b
 i2c           1  [   ]   i2c_mxc               |   |   |-- i2c@30a30000
 i2c           2  [   ]   i2c_mxc               |   |   |-- i2c@30a40000
 i2c           3  [   ]   i2c_mxc               |   |   |-- i2c@30a50000
```