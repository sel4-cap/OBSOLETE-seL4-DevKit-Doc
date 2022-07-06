# First Boot

First boot aims to demonstrate correct operation of the prepared SD card and communication over the serial interface. This ensures all steps have been followed correctly prior to proceeding with compilation and execution of an seL4 binary.

## Prerequisites

Prior to powering the MaaXBoard ensure the following setup is in place, building on previous sections:

1. The SD card has been prepared as per the instructions in the [SD Card Preparation](sd_card_preparation.md) section.

2. The MaaXBoard has been set up as per the [Target Platform Setup](target_platform_setup.md) section with only the USB-to-TTL cable connected and the SD card inserted.

3. The USB-to-TTL cable is connected to the host machine with CoolTerm configured as per the [Host Machine Setup](host_machine_setup.md) section.

## Boot to U-Boot Prompt

At this stage power should be supplied to the MaaXBoard by connecting the USB-C power adapter. On supply of power the user should see the output of the bootloader displayed in CoolTerm. As no `sel4_image` binary has been supplied the user should be dropped to the U-Boot command prompt.

The log below shows the serial terminal output on the host machine when the MaaXBoard boots with no access to a binary to execute.

Note that the serial terminal output does not include the line numbers shown in the example logs shown below; those line numbers have only been added afterwards for ease of reference in this documentation.

```text
     1  U-Boot SPL 2021.04-00002-gf752480a4c (Jan 20 2022 - 10:32:23 +0000)
     2  power_bd71837_init
     3  set buck8 to 1.2v for DDR4
     4  DDRINFO: start DRAM init
     5  DDRINFO: DRAM rate 2400MTS
     6  DDRINFO:ddrphy calibration done
     7  DDRINFO: ddrmix config done
     8  Normal Boot
     9  Trying to boot from MMC1
    10  
    11  
    12  U-Boot 2021.04-00002-gf752480a4c (Jan 20 2022 - 10:32:23 +0000)
    13  
    14  CPU:   i.MX8MQ rev2.1 1500 MHz (running at 1000 MHz)
    15  CPU:   Commercial temperature grade (0C to 95C) at 24C
    16  Reset cause: POR
    17  Model: Avnet Maaxboard
    18  DRAM:  2 GiB
    19  MMC:   FSL_SDHC: 0
    20  Loading Environment from MMC... *** Warning - bad CRC, using default environment
    21  
    22  In:    serial
    23  Out:   serial
    24  Err:   serial
    25  
    26   BuildInfo:
    27    - ATF d801fd9
    28  
    29  switch to partitions #0, OK
    30  mmc0 is current device
    31  flash target is MMC:0
    32  Net:   
    33  Warning: ethernet@30be0000 (eth0) using random MAC address - 5a:15:1f:fd:43:19
    34  eth0: ethernet@30be0000
    35  Fastboot: Normal
    36  Normal Boot
    37  Hit any key to stop autoboot:  2 ... 1 ... 0 
    38  starting USB...
    39  Bus usb@38100000: Register 2000140 NbrPorts 2
    40  Starting the controller
    41  USB XHCI 1.10
    42  Bus usb@38200000: Register 2000140 NbrPorts 2
    43  Starting the controller
    44  USB XHCI 1.10
    45  scanning bus usb@38100000 for devices... 1 USB Device(s) found
    46  scanning bus usb@38200000 for devices... 1 USB Device(s) found
    47         scanning usb for storage devices... 0 Storage Device(s) found
    48  
    49  Device 0: unknown device
    50  MMC Device 1 not found
    51  no mmc device at slot 1
    52  MMC Device 2 not found
    53  no mmc device at slot 2
    54  switch to partitions #0, OK
    55  mmc0 is current device
    56  SD/MMC found on device 0
    57  1493 bytes read in 1 ms (1.4 MiB/s)
    58  Loaded env from uEnv.txt
    59  Importing environment from mmc0 ...
    60  Running uenvcmd ...
    61
    62  Device 0: unknown device
    63
    64  Device 1: unknown device
    65  switch to partitions #0, OK
    66  mmc0 is current device
    67  Booting ELF binary from mmc 0 ...
    68  Failed to load 'sel4_image'
    69  ## No elf image at address 0x40480000
    70  MMC Device 1 not found
    71  no mmc device at slot 1
    72  Using statically defined IP address
    73  Booting ELF binary from TFTP ...
    74  ethernet@30be0000 Waiting for PHY auto negotiation to
        complete......................................... TIMEOUT !
    75  Could not initialize PHY ethernet@30be0000
    76  Using ethernet@30be0000 device
    77  TFTP from server 192.168.100.3; our IP address is 192.168.100.50
    78  Filename 'sel4_image'.
    79  Load address: 0x40480000
    80  Loading: *.
    81  ARP Retry count exceeded; starting again
    82  ## No elf image at address 0x40480000
    83  switch to partitions #0, OK
    84  mmc0 is current device
    85  Failed to load 'boot.scr'
    86  Failed to load 'Image'
    87  Booting from net ...
    88  ethernet@30be0000 Waiting for PHY auto negotiation to
      complete......................................... TIMEOUT !
    89  Could not initialize PHY ethernet@30be0000
    90  BOOTP broadcast 1
    91  BOOTP broadcast 2
    92  BOOTP broadcast 3
    93  BOOTP broadcast 4
    94  BOOTP broadcast 5
    95  BOOTP broadcast 6
    96  BOOTP broadcast 7
    97  BOOTP broadcast 8
    98  BOOTP broadcast 9
    99  BOOTP broadcast 10
   100  BOOTP broadcast 11
   101  BOOTP broadcast 12
   102  BOOTP broadcast 13
   103  BOOTP broadcast 14
   104  BOOTP broadcast 15
   105  BOOTP broadcast 16
   106  BOOTP broadcast 17
   107  
   108  Retry time exceeded; starting again
   109  ethernet@30be0000 Waiting for PHY auto negotiation to
      complete......................................... TIMEOUT !
   110  Could not initialize PHY ethernet@30be0000
   111  BOOTP broadcast 1
   112  BOOTP broadcast 2
   113  BOOTP broadcast 3
   114  BOOTP broadcast 4
   115  BOOTP broadcast 5
   116  BOOTP broadcast 6
   117  BOOTP broadcast 7
   118  BOOTP broadcast 8
   119  BOOTP broadcast 9
   120  BOOTP broadcast 10
   121  BOOTP broadcast 11
   122  BOOTP broadcast 12
   123  BOOTP broadcast 13
   124  BOOTP broadcast 14
   125  BOOTP broadcast 15
   126  BOOTP broadcast 16
   127  BOOTP broadcast 17
   128  
   129  Retry time exceeded; starting again
   130  WARN: Cannot load the DT
   131  u-boot=> 
```

_Note: The CRC warning at line 20 is not an important issue; it is just an artefact from the basic U-Boot image, and the default environment variables are sufficient._

Once at the U-Boot prompt, the user can enter interactive U-Boot commands. For example:

- `help` to display the available commands;
- `printenv` to display the environment variables;
- `setenv` to set environment variables;
- `reset` to reset the CPU instead of physically cycling the power.

A full reference to U-Boot commands can be found [here](https://u-boot.readthedocs.io/en/latest/usage/index.html?highlight=shell) but we only use a few of them directly in this developer kit.

The following sections build upon this step by demonstrating the building and execution of an seL4 application on the MaaXBoard.
