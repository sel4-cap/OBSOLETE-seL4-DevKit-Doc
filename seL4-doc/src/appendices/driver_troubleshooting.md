# Library Extension - Troubleshooting

This section attempts to provide guidance to help resolve issues that may be encountered when adding new device drivers to the library.

## Increase logging

Whenever an issue is encountered, one of the simplest actions that can be taken to provide additional data is to increase the verbosity of the library's output.

The library's logging level is controlled by the `LIB_UBOOT_LOGGING_LEVEL` variable in the library `CMakeLists.txt` file:

```makefile
set(LIB_UBOOT_LOGGING_LEVEL "ZF_LOG_INFO")
```

This can be increased by changing the default `ZF_LOG_INFO` level to a higher logging level, e.g. `ZF_LOG_DEBUG` or `ZF_LOG_VERBOSE`.

## DMA

As documented in the [library overview](../uboot_driver_library.md#dma) section, drivers utilising DMA may need manual modifications.

### Detecting DMA usage

When adding support for a new device and associated driver, it should be determined whether the driver utilises DMA.

The single most reliable method for determining DMA usage is to read and understand the device reference manual; however, there are a number of strong indicators from the U-Boot source code that can assist, such as:

1. Search the driver source code for references to `DMA`.

2. Allocation of aligned memory (i.e. usage of `memalign`). Memory allocated with the intent of being used for DMA transfers tends to be aligned on cache line boundaries. Examples of U-Boot source code allocating memory for DMA are `ptr = memalign(cachline_size, ...)` and `tmp = memalign(ARCH_DMA_MINALIGN, ...)`.

3. Data cache flushing or invalidation (i.e. usage of `flush_dcache_range` and `invalidate_dcache_range`). Such manipulation of the data cache is a strong indicator that the associated memory is utilised for communication with a device.

### Resolving DMA issues

Once it has been determined that an area of memory is being used by the driver for DMA purposes, it must be converted to use an seL4 DMA region.

Where the memory is managed through standard `malloc`, `memalign`, and `free`, these must be converted to the directly equivalent routines `sel4_dma_malloc`, `sel4_dma_memalign`, and `sel4_dma_free` provided by the library wrapper. Note that no changes are required to usage of `flush_dcache_range` and `invalidate_dcache_range`; these routines are automatically mapped to the seL4 DMA equivalents by the library wrapper.

In cases where the existing memory allocation cannot be easily modified, e.g. the memory was allocated by a variable declaration on the stack or is only used for DMA in limited cases, then an alternative approach can be taken. Prior to communicating the DMA address to the device (usually performed via memory-mapped IO) a DMA region can be allocated to mirror the original memory and the data manually copied between them as required.

If a driver performs conversions between the DMA region's virtual and physical addresses, such conversions must be replaced with usage of the `sel4_dma_virt_to_phys` and `sel4_dma_phys_to_virt` routines provided by the library wrapper.

### Worked Examples

All of the above techniques have been used in the modifications made to the Ethernet driver file `drivers/net/fec_mxc.c` in [this Git commit](https://github.com/sel4devkit/u-boot/commit/6a4512f1d3b8427a4e192a14c52319a6228c7bbe):

- Within routine `fec_alloc_descs`, calls to `memalign` have been replaced with `sel4_dma_memalign`.
- Within routine `fec_free_descs`, the corresponding calls to `free` have been replaced with `sel4_dma_free`.
- Within routine `fecmxc_send`, an seL4 DMA region is allocated to mirror the contents of the provided packet (and subsequently freed in `fecmxc_free_pkt`).
- Address translations using `sel4_dma_virt_to_phys` and `sel4_dma_phys_to_virt` are performed around use of the `.data_pointer` pointers into the receive buffers (which are communicated with the device) to ensure that the device is only sent the physical address of the DMA region.

## Live Device Tree support

As documented in the [library overview](../uboot_driver_library.md#library-limitations) section, the library only supports drivers that are compatible with U-Boot's live tree format for holding the device tree.

Guidance on porting an old driver to the live device tree interface is provided [here](https://u-boot.readthedocs.io/en/latest/develop/driver-model/livetree.html#porting-drivers).

A worked example of porting can be seen in the Pin Multiplexing driver file `drivers/pinctrl/nxp/pinctrl-imx.c` in [this Git commit](https://github.com/sel4devkit/u-boot/commit/6a4512f1d3b8427a4e192a14c52319a6228c7bbe).

## Missing Initialisation

When a new device class has been added to the library, it may be necessary to perform explicit initialisation of the associated U-Boot subsystems. Currently this has only been required for the Ethernet and MMC device classes as shown below in the extract from `uboot_wrapper.c`:

```c
int initialise_uboot_wrapper(char* fdt_blob)
{
    ...

#ifdef CONFIG_DM_MMC
    // Initialize the MMC system.
    ret = mmc_initialize(NULL);
    if (0 != ret)
        goto error;
#endif

#ifdef CONFIG_NET
    // Initialize the ethernet system.
    puts("Net:   ");
    eth_initialize();
#ifdef CONFIG_RESET_PHY_R
    debug("Reset Ethernet PHY\n");
    reset_phy();
#endif
#endif

    ...
}
```

Whether initialisation is required, and the required initialisation, can be determined by referencing U-Boot's own initialisation code held in U-Boot source file `common/board_r.c`.

## Environment Variables Not Working

Setting or changing certain environment variables in U-Boot can trigger callback routines. If these callback routines are not called then the setting of the environment variable will appear to have no effect.

Environment variable callbacks are declared by the U-Boot macro `U_BOOT_ENV_CALLBACK` and need to be referenced in the platform's `plat_driver_data.h` and `plat_driver_data.c` files, if required.

An example of this are the callbacks associated with networking related environment variables (e.g. setting of the IP address). The following excerpts from the Avnet MaaXBoard's `plat_driver_data.h` and `plat_driver_data.c` enable all networking related callbacks:

```c
#define _u_boot_env_clbk_count 8
...
extern struct env_clbk_tbl _u_boot_env_clbk__ethaddr;
extern struct env_clbk_tbl _u_boot_env_clbk__ipaddr;
extern struct env_clbk_tbl _u_boot_env_clbk__gatewayip;
extern struct env_clbk_tbl _u_boot_env_clbk__netmask;
extern struct env_clbk_tbl _u_boot_env_clbk__serverip;
extern struct env_clbk_tbl _u_boot_env_clbk__nvlan;
extern struct env_clbk_tbl _u_boot_env_clbk__vlan;
extern struct env_clbk_tbl _u_boot_env_clbk__dnsip;
```

```c
void initialise_driver_data(void) {
    ...
    driver_data.env_clbk_array[0] = _u_boot_env_clbk__ethaddr;
    driver_data.env_clbk_array[1] = _u_boot_env_clbk__ipaddr;
    driver_data.env_clbk_array[2] = _u_boot_env_clbk__gatewayip;
    driver_data.env_clbk_array[3] = _u_boot_env_clbk__netmask;
    driver_data.env_clbk_array[4] = _u_boot_env_clbk__serverip;
    driver_data.env_clbk_array[5] = _u_boot_env_clbk__nvlan;
    driver_data.env_clbk_array[6] = _u_boot_env_clbk__vlan;
    driver_data.env_clbk_array[7] = _u_boot_env_clbk__dnsip;
}
```

## Incomplete Device Tree

Device drivers commonly access settings stored within the platform's Device Tree. If the device does not appear to be functioning, it can be related to information missing from the Device Tree; such issues can be resolved by including the missing information in the platform's Device Tree overlay.

If the Device Tree is suspected of being incomplete it is suggested that the equivalent entries could be compared from Device Tree files for similar platforms (e.g. those using the same or similar SoC) held within the Linux and U-Boot repositories.

### Example - DWC3 Generic USB Driver

The widely used DWC3 USB device is supported by the DWC3 Generic USB driver. This driver has been found to expect a specific format of the USB device entries in the device tree with some settings held within a sub-node. Without the correct format, the USB device will not function.

An example of the required format is provided in the [overlay file for the Avnet MaaXBoard](https://github.com/seL4/seL4/blob/master/src/plat/maaxboard/overlay-maaxboard.dts).

### Example - Missing Pin Multiplexing Settings

Another potential reason for a non-functioning device is a failure to correctly configure the SoC's pin multiplexer, thereby failing to correctly connect the device to the external pins / pads on the SoC. This issue was encountered when adding SPI support for the Avnet MaaXBoard.

Pin multiplexing settings are held within `pinctrl-<index>` properties in the Device Tree, for example:

```text
i2c@30a20000 {
    compatible = "fsl,imx8mq-i2c\0fsl,imx21-i2c";
    ...
    pinctrl-names = "default";
    pinctrl-0 = <0x25>;
```

Note that not all devices require pin multiplexer settings; e.g. device outputs may be routed to dedicated SoC pins / pads. The SoC's reference manual should be consulted to determine whether configuration is required for a device.
