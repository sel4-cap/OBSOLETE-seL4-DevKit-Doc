# Partitioning the SD card

## macOS Instructions

The SD card must be partitioned correctly in order to contain U-Boot, seL4 and space for a filesystem, this can be done as follows.

1. Connect your SD card reader to your host machine and insert your SD card.

2. Find the disk identifier (e.g  `/dev/disk6` ) for your SD card, on macOS this can be done by running `diskutil list` . This command should present a list of disks and their petitions, `/dev/disk0`  and  `/dev/disk1`  are usually used for the internal SSD/HDD on your Mac, so the SD card will usually be at the bottom, assuming it was the last storage device attached to your machine.

3. On macOS, you may need to unmount any volumes associated with the SD card. You can do this either from Disk Utility or by using `diskutil unmount /dev/diskXsY` where `X` is the disk identifier and `Y` is the volume identifier.

4. From terminal run the following command, replacing X in `/dev/diskX` with your disk identifier as found earlier. *Note: the `15GB` size mentioned for the`FILESYS` partition can be changed to `R` which will use the remainder of the disk. The instructions here use `15GB` to reflect what was used to create the prebuilt images in the [maaxboard-prebuilt](https://github.com/sel4devkit/maaxboard-prebuilt] repo. `15GB` is used as "16GB" SD cards/flash drives are not all truly "16GB" due to formatting, so this is done to account for minor differences in size.*

```sh
diskutil partitionDisk /dev/diskX 3 MBR \
   FREE Bootloader 10M \
   FAT32 BOOT 256M \
   FAT32 FILESYS 15GB

```

This creates 3 partitions on the disk, a 10 megabyte bootloader partition for U-Boot, a 256 megabyte partition to hold the seL4 image, and finally the remainder of the disk is provided for the file system.
