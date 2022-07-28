# Manually Writing U-Boot to an SD card

**Please note, the SD card must be correctly partitioned before following these instructions.**

1. If you are continuing on from the [partitioning appendix](./partitioning_sd_card.md), skip straight to step 5.

2. Connect your SD card reader to your host machine and insert your SD card.

3. Find the disk identifier for your SD card (e.g. `/dev/disk6` for macOS, `/dev/sdb` for Linux).
    - **macOS:** This can be done by running `diskutil list`. This command should present a list of disks and their partitions, `/dev/disk0` and `/dev/disk1` are usually used for the internal SSD/HDD on your Mac, so the newly inserted storage device will typically be at the bottom, assuming it was the last storage device attached to your machine.
    - **Linux**: You can list all disk drives and their identifiers using either `lsblk` or `df -h`, where `/dev/sda` is usually the system drive.

4. You may need to unmount any volumes associated with the SD card. In both cases, `X` is the disk identifer and `Y` is the volume identifier.
    - **macOS:** You can do this either from Disk Utility or by using `diskutil unmount /dev/diskXsY`.
    - **Linux:** unmouting of volumes on the disk can be performing using `sudo umount /dev/sdXY`.

5. Navigate to the folder containing your U-Boot `flash.bin` file. This file could have been generated from the [Building U-Boot](./building_uboot.md) appendix, or it is also available from the [maaxboard-prebuilt](https://github.com/sel4devkit/maaxboard-prebuilt) repository.

    > **WARNING: The next step uses the `dd` command line utility, which is used for writing images to disks. IT WILL OVERWRITE ANY DATA ON THE DISK IT IS SPECIFIED TO WRITE TO! Improper usage WILL cause data loss, corruption and potentially render your system inoperable. Please ensure you are familiar with the use of the command, as well as the disk identifiers on your system, and that you are writing to the disk you intend to, and not your system drive!**

6. From that folder run the following command, replacing `/dev/diskX` with the disk identifier of your SD card. You may be asked to enter your password.

    ```sh
    sudo dd if=flash.bin of=/dev/diskX bs=1k seek=33
    ```

7. The image should now be written to your SD card and should be bootable by the MaaXBoard.
