# Hardware Requirements

This section details the hardware related to the target platform that the guide assumes is available to the developer. Before following the guide the developer will need to have access to all items marked as mandatory.

As detailed in the [Execution on Target Platform](execution_on_target_platform.md) section, there are multiple methods of transferring a compiled seL4 binary from the host machine to the target platform. Where a hardware item is required to support only one potential transfer method it is marked as 'optional'. The developer should decide which transfer method(s) are to be used to determine which items need to be available.

For convenience, the following table includes order codes and hyperlinks for the [Farnell UK store](https://uk.farnell.com) correct as of 20th of January 2022; clearly, equivalent items are available from many other sources.

| Item                                 | Notes                         | Order Code                                                                                                          |
| ------------------------------------ | ----------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| Avnet MaaXBoard (AES-MC-SBC-IMX8M-G) | Mandatory                     | [3436577](https://uk.farnell.com/avnet/aes-mc-sbc-imx8m-g/sbc-quad-arm-cortex-a53-cortex/dp/3436577?ost=3436577)    |
| USB-to-TTL Serial UART Cable         | Mandatory                     | [2147356](https://uk.farnell.com/ftdi/ttl-232r-rpi/cable-debug-ttl-232-usb-rpi/dp/2147356)               |
| 16GB Micro SD Card                   | Mandatory                     | [3498607](https://uk.farnell.com/integral/inmsdh16g10-90u1/16gb-ultimapro-microsd-c10-90/dp/3498607?ost=3498607)    |
| USB Micro SD Card Reader/Writer    | Mandatory                     | [3493850](https://uk.farnell.com/tripp-lite/u452-000-sd-a/usb-c-memory-card-reader-sd-micro/dp/3493850?ost=3493850) |
| 15W USB-C Power Adapter              | Mandatory                     | [3106255](https://uk.farnell.com/stontronics/t7725dv/adapter-ac-dc-1-o-p-5-1v-3a/dp/3106255?ost=3106255)            |
| USB Flash Drive                      | Optional - USB transfer only  | General[^1]                      |
| Ethernet Cable                       | Optional - TFTP transfer and some of the test applications | General[^1] |
| USB Keyboard | Optional - some of the test applications | [1848111](https://uk.farnell.com/logitech/920-002524/keyboard-k120-business-logitech/dp/1848111)[^2] |
| SPI Bus Pressure Sensor | Optional - test application only | [See Appendix](appendices/spi_bmp280.md) |

[^1]Where items are considered to be ubiquitous with no special requirements, no example order code is given.

[^2]Although USB keyboards are fairly ubiquitous, experience leads us to recommend a 'basic' model such as this, which works with our keyboard driver; the driver may not work with more feature-rich models (e.g. a keyboard with an integral USB hub, nor does it work with [this compact keyboard](https://uk.farnell.com/a4-tech/rp011/keyboard-mini-slim-black-uk/dp/2113614)).

This guide assumes the following basic hardware capabilities of the user's development environment:

1. The ability to connect the MaaXBoard to a wired network to which the host machine is also connected. This is required if the user wishes to perform TFTP transfer of executables from the host environment to the target platform (see the [Bootloader](bootloader.md) section). It is also required for full demonstration of the [case study application](case_study_intro.md) and to execute the [picoserver test application](uboot_driver_usage.md#test-application-picoserver_uboot).

2. The ability to connect USB devices (i.e. the USB flash drive and USB SD card reader / writer) to the host machine.
