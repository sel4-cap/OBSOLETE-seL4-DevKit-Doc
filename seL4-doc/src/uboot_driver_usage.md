# Using the U-Boot Driver Library

## xxx

The ported U-Boot drivers in the library have been made accessible via U-Boot commands, i.e. a subset of those available at the [U-Boot prompt](first_boot.md#boot-to-u-boot-prompt). For example, in the [I<sup>2</sup>C worked example](uboot_library_add_driver.md#establishing-the-driver-api), it is shown how the U-Boot `i2c` command is added and used, e.g. to probe the bus. 

Our CAmkeS test application `uboot-driver-example` shows how to access the drivers that have been ported so far.

## Instructions for running the uboot-driver-example test

As usual, this assumes that the user is already running a Docker container within the [build environment](build_environment_setup.md), where we can create a directory and clone the code and dependencies.


```bash
mkdir /host/uboot_test
cd /host/uboot_test
```

 _(Note: below is currently on maaxboard-usb branch; will ultimately change)_

```bash
repo init -u https://github.com/sel4devkit/camkes-manifest.git -b maaxboard-usb
```

```bash
repo sync
```

_Temporary workaround of musllibc bug until [seL4_libs PR#63](https://github.com/seL4/seL4_libs/pull/63) is merged: Copy `memalign.c` from [https://github.com/stephen-williams-capgemini/musllibc/tree/issue_17/src/malloc](https://github.com/stephen-williams-capgemini/musllibc/tree/issue_17/src/malloc) and overwrite the version in `/host/uboot_test/projects/musllibc/src/malloc/`_

The test application includes an Ethernet operation (ping) with hard-coded IP addresses; these need to be customised for an individual's environment. The following lines of the source file `projects/camkes/apps/uboot-driver-example/components/Test/src/test.c` should be edited:

```
run_uboot_command("setenv ipaddr xxx.xxx.xxx.xxx"); // IP address to allocate to MaaXBoard e.g. 192.168.0.111
run_uboot_command("setenv gatewayip xxx.xxx.xxx.xxx"); // IP address of router e.g. 192.168.0.1; only required to ping beyond the local network
run_uboot_command("setenv netmask 255.255.255.0");
run_uboot_command("ping xxx.xxx.xxx.xxx"); // IP address of host machine e.g. 192.168.0.11
run_uboot_command("ping 8.8.8.8"); // An example internet IP address (Google DNS); requires gatewayip to have been set
```

From the `/host/uboot_test` directory, execute the following commands:

```bash
mkdir build
cd build
```

```bash
../init-build.sh -DCAMKES_APP=uboot-driver-example -DPLATFORM=maaxboard -DSIMULATION=FALSE
```

```bash
ninja
```

A successful build will result in an executable file called `capdl-loader-image-arm-maaxboard` in the `images` subdirectory. This should be copied to a file named `sel4_image` and then made available to the preferred loading mechanism, such as TFTP, as per [Execution on Target Platform](execution_on_target_platform.md).

_Incomplete, and remember to refer to new appendix for BMP280_