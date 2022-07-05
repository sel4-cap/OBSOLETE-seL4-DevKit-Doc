# Hardware Requirements

This section details the hardware related to the target platform that the guide assumes is available to the developer. Before following the guide the developer will need to have access to all items marked as mandatory.

As detailed in the [Execution on Target Platform](execution_on_target_platform.md) section, there are multiple methods of transferring a compiled seL4 binary from the host machine to the target platform. Where a hardware item is required to support only one potential transfer method it is marked as 'optional'. The developer should decide which transfer method(s) are to be used to determine which items need to be available.

The following table includes order codes and hyperlinks for the [Farnell UK store](https://uk.farnell.com) correct as of 20th of January 2022.

| Item                                 | Notes                         | Order Code                                                                                                          |
| ------------------------------------ | ----------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| Avnet MaaXBoard (AES-MC-SBC-IMX8M-G) | Mandatory                     | [3436577](https://uk.farnell.com/avnet/aes-mc-sbc-imx8m-g/sbc-quad-arm-cortex-a53-cortex/dp/3436577?ost=3436577)    |
| USB to TTL Serial Cable              | Mandatory                     | [2147356](https://uk.farnell.com/ftdi/ttl-232r-rpi/cable-debug-ttl-232-usb-rpi/dp/2147356)               |
| 16GB Micro SD Card                   | Mandatory                     | [3498607](https://uk.farnell.com/integral/inmsdh16g10-90u1/16gb-ultimapro-microsd-c10-90/dp/3498607?ost=3498607)    |
| USB Micro SD Card Reader / Writer    | Mandatory                     | [3493850](https://uk.farnell.com/tripp-lite/u452-000-sd-a/usb-c-memory-card-reader-sd-micro/dp/3493850?ost=3493850) |
| 15W USB-C Power Adapter              | Mandatory                     | [3106255](https://uk.farnell.com/stontronics/t7725dv/adapter-ac-dc-1-o-p-5-1v-3a/dp/3106255?ost=3106255)            |
| USB Flash Drive                      | Optional - USB transfer only  | [3410011](https://uk.farnell.com/hama/90891/usb-memory-stick-rotate-8gb/dp/3410011?st=3410011)                      |
| Ethernet Cable                       | Optional - TFTP transfer only | [1526063](https://uk.farnell.com/videk/2961-2/patch-lead-cat5e-utp-beige-2m/dp/1526063?ost=1526063)                 |

This guide assumes the following basic hardware capabilities of the user's development environment:

1. The ability to connect the MaaXBoard to a wired network to which the host machine is also connected. Note that this is only required if the user wishes to perform TFTP transfer of executables from the host environment to the target platform (see the [Bootloader](bootloader.md) section).

2. The ability to connect USB devices (i.e. the USB flash drive and USB SD card reader / writer) to the host machine.
