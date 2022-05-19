# Library Extension - Troubleshooting

This section attempts to provide guidance to help resolve issues that may be encountered When adding new device drivers to the library.

## Increase logging

Whenever an issue is encountered one of the simplest actions that can be taken to provide additional data is to increase the verbosity of the library's output.

The library's logging level is controlled by the `LIB_UBOOT_LOGGING_LEVEL` variable in the library `CMakeLists.txt` file:

```makefile
set(LIB_UBOOT_LOGGING_LEVEL "ZF_LOG_INFO")
```

This can be increased by changing the default `ZF_LOG_INFO` level to a higher logging level, e.g. `ZF_LOG_DEBUG` or `ZF_LOG_VERBOSE`.

## DMA

As documented in the [library overview](../uboot_driver_library.md#dma) section, drivers utilising DMA may need manual modifications.

### Detecting DMA usage

When adding support for a new device and associated driver it should be determined whether the driver utilising DMA.

The single most reliable method for determining DMA usage is to read and understand the device reference manual, however there are a number of strong indicators from the U-Boot source code that can assist:

1. Search the driver source code for references to `DMA`.

2. Allocation of aligned memory (i.e. usage of `memalign`). Memory allocated with the intent of being used for DMA transfers tend to aligned on cache line boundaries. Examples of U-Boot source code allocating memory for DMA are `ptr = memalign(cachline_size, ...);` and `tmp = memalign(ARCH_DMA_MINALIGN, ...);`.

3. Data cache flushing or invalidation (i.e. usage of `flush_dcache_range` and `invalidate_dcache_range`). Such manipulation of the data cache is a strong indicator that the associated memory is utilised for communication with a device.

### Resolving DMA issues

Once it has been determined that an area of memory is being used by the driver for DMA purposes it must be converted to use an seL4 DMA region.

Where the memory managed through standard `malloc`, `memalign` and `free` these must be converted to the directly equivalent `sel4_dma_malloc`, `sel4_dma_memalign` and `sel4_dma_free` routines provided by the library wrapper. Note that no changes are required to usage of `flush_dcache_range` and `invalidate_dcache_range`; these routines are automatically mapped to the seL4 DMA equivalents by the library wrapper.

In cases where the existing memory allocation cannot be easily modified, e.g. the memory was allocated by a variable declaration on the stack or is only used for DMA in limited cases, then an alternative approach can be taken. Prior to communicating the DMA address to the device (usually performed via memory-mapped IO) a DMA region can be allocated to mirror the original memory and the data manually copied between them as required.

If a driver performs conversions between the DMA region's virtual and physical addresses such conversions must be replaced with usage of the `sel4_dma_virt_to_phys` and `sel4_dma_phys_to_virt` routines provided by the library wrapper.

### Worked Examples

All of the above techniques have been used in the modifications made to the Ethernet driver file `drivers/net/fec_mxc.c` in [this Git commit](https://github.com/sel4devkit/u-boot/commit/6a4512f1d3b8427a4e192a14c52319a6228c7bbe):

- Within routine `fec_alloc_descs` calls to `memalign` have been replaced with `sel4_dma_memalign`.
- Within routine `fec_free_descs` the corresponding calls to `free` have been replaced with `sel4_dma_free`.
- Within routine `fecmxc_send` an seL4 DMA region is allocated to mirror the contents of the provided packet (and subsequently freed in `fecmxc_free_pkt`).
- Address translations using `sel4_dma_virt_to_phys` and `sel4_dma_phys_to_virt` are performed around use of the `.data_pointer` pointers into the receive buffers (which are communicated with the device) to ensure the device is only sent the physical address of the DMA region.

## Live Tree support

xx

## Missing Initialisation

TBC:

- May need to add new initialisation code. Look in u-boot’s common/board_r.c file to see if any is required.

## Environment Variables Not Working

TBC:

- Environment variables not having expected effect - check for callbacks.

## Incomplete Device Tree

TBC:

- DWC3 USB example.
- Missing Pin Multiplexing settings
