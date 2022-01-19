
# SD Card preparation 
## Partitioning the SD card
### MacOS Instructions
The SD card must be petitioned correctly in order to contain U-Boot, seL4 and space for a filesystem, this can be done as follows.
1. Insert your SD card into an SD card reader connected to your computer.
2. Find the disk identifier (e.g  `/dev/disk6` ) for your SD card, on MacOS this can be done by running `diskutil list` . This command should present a list of disks and their petitions, `/dev/disk0`  and  `/dev/disk1`  are usually used for the internal SSD/HDD on your Mac, so the SD card will usually be at the bottom, assuming it was the last storage device attached to your machine.
3.  On MacOS, you may need to unmount any volumes associated with the SD card. You can do this either from Disk Utility or by using `diskutil unmount /dev/diskXsY` where `X` is the disk identifier and `Y` is the volume identifier.
4. From terminal run the following command, replacing X in `/dev/diskX` with your disk identifier as found earlier
```
diskutil partitionDisk /dev/diskX 3 MBR \
   FREE Bootloader 10M \
   FAT32 BOOT 256M \
   FAT32 FILESYS R

```
This creates 3 partitions on the disk, a 10 megabyte bootloader partition for U-Boot, a 256 megabyte partition to hold the seL4 image, and finally the remainder of the disk is provided for the file system.


## Writing the Maaxboard U-Boot image to an SD card.
1. If you are continuing on from the partitioning section, skip straight to step 5.
2. Insert your SD card into an SD card reader connected to your computer.
3. Find the disk identifier (e.g  `/dev/disk6` ) for your SD card, on MacOS this can be done by running `diskutil list` . This command should present a list of disks and their petitions, `/dev/disk0`  and  `/dev/disk1`  are usually used for the internal SSD/HDD on your Mac, so the SD card will usually be at the bottom, assuming it was the last storage device attached to your machine.
4. On MacOS, you may need to unmount any volumes associated with the SD card. You can do this either from Disk Utility or by using `diskutil unmount /dev/diskXsY` where `X` is the disk identifier and `Y` is the volume identifier.
5. Using the terminal, navigate to the folder containing your U-Boot  `flash.bin`  file.
6. From that folder run the following command: `sudo dd if=flash.bin of=/dev/diskX bs=1k seek=33` , replacing `/dev/diskX` with the disk identifier of your SD card respectively. You may be asked to enter your password.
7. The image should now be written to your SD card and should be bootable by the Maaxboard.



## Setting up U-Boot config
