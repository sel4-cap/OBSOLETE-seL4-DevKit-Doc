# First Boot

## U-Boot Only

Prerequisites: Note to self - list/reference other sections

Note that the terminal output does not include the line numbers; those have just been added afterwards for ease of reference.


```
     1	U-Boot SPL 2021.04-00002-gf752480a4c (Jan 20 2022 - 10:32:23 +0000)
     2	power_bd71837_init
     3	set buck8 to 1.2v for DDR4
     4	DDRINFO: start DRAM init
     5	DDRINFO: DRAM rate 2400MTS
     6	DDRINFO:ddrphy calibration done
     7	DDRINFO: ddrmix config done
     8	Normal Boot
     9	Trying to boot from MMC1
    10	
    11	
    12	U-Boot 2021.04-00002-gf752480a4c (Jan 20 2022 - 10:32:23 +0000)
    13	
    14	CPU:   i.MX8MQ rev2.1 1500 MHz (running at 1000 MHz)
    15	CPU:   Commercial temperature grade (0C to 95C) at 25C
    16	Reset cause: POR
    17	Model: Avnet Maaxboard
    18	DRAM:  2 GiB
    19	MMC:   FSL_SDHC: 0
    20	Loading Environment from MMC... *** Warning - bad CRC, using default environment
    21	
    22	In:    serial
    23	Out:   serial
    24	Err:   serial
    25	
    26	 BuildInfo:
    27	  - ATF d801fd9
    28	
    29	switch to partitions #0, OK
    30	mmc0 is current device
    31	flash target is MMC:0
    32	Net:   
    33	Warning: ethernet@30be0000 (eth0) using random MAC address - 9a:e7:b5:44:da:ab
    34	eth0: ethernet@30be0000
    35	Fastboot: Normal
    36	Normal Boot
    37	Hit any key to stop autoboot:  2 ... 1 ... 0 
    38	starting USB...
    39	Bus usb@38100000: Register 2000140 NbrPorts 2
    40	Starting the controller
    41	USB XHCI 1.10
    42	Bus usb@38200000: Register 2000140 NbrPorts 2
    43	Starting the controller
    44	USB XHCI 1.10
    45	scanning bus usb@38100000 for devices... 1 USB Device(s) found
    46	scanning bus usb@38200000 for devices... 1 USB Device(s) found
    47	       scanning usb for storage devices... 0 Storage Device(s) found
    48	
    49	Device 0: unknown device
    50	MMC Device 1 not found
    51	no mmc device at slot 1
    52	MMC Device 2 not found
    53	no mmc device at slot 2
    54	switch to partitions #0, OK
    55	mmc0 is current device
    56	SD/MMC found on device 0
    57	Failed to load 'uEnv.txt'
    58	switch to partitions #0, OK
    59	mmc0 is current device
    60	Failed to load 'boot.scr'
    61	Failed to load 'Image'
    62	Booting from net ...
    63	BOOTP broadcast 1
    64	BOOTP broadcast 2
    65	DHCP client bound to address 192.168.0.56 (1003 ms)
    66	*** ERROR: `serverip' not set
    67	Cannot autoload with TFTPGET
    68	BOOTP broadcast 1
    69	BOOTP broadcast 2
    70	DHCP client bound to address 192.168.0.56 (892 ms)
    71	*** ERROR: `serverip' not set
    72	Cannot autoload with TFTPGET
    73	WARN: Cannot load the DT
    74	u-boot=> 
```

continue...


