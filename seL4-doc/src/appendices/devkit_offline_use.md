# Using the Developer Kit Offline
If you need to use the developer kit in situations where public internet access is unavailable, you can download the required files for building the developer kit for offline use.

- [Downloading the Docker image to a file](#downloading-the-docker-image-to-a-file)
- [Downloading source code for offline use](#downloading-source-code-for-offline-use)
- [Building U-Boot offline](#building-u-boot-offline)
- [Downloading the pre-prepared SD card images](#downloading-the-pre-prepared-sd-card-images)

## Downloading the Docker image to a file
The Docker image contains all the required toolchain for building seL4 and seL4 applications, usually the command `docker pull` is used to download an image into your Docker desktop application, however this won’t be possible on a computer without internet access. The following steps detail how the Docker image can be downloaded and saved to a file which can then be transferred to a machine without internet access:

1. On a machine with internet access and Docker installed, run: `docker pull ghcr.io/sel4devkit/maaxboard:latest`. The image will now be downloaded. During download of the image progress is reported in the following format:

```
% docker pull ghcr.io/sel4devkit/maaxboard:latest
latest: Pulling from sel4devkit/maaxboard
0e29546d541c: Pull complete 
c2db25fafa50: Pull complete 
00af15d3de8e: Downloading [==========>                                ]  241.7MB/1.188GB
c6749363a298: Download complete 
521655c15371: Download complete 
```

2. Once the download has completed, the image can now be saved to a file, to do this run `docker save ghcr.io/sel4devkit/maaxboard:latest  > sel4devkit_docker.tar`. This will save the docker image to a file called `sel4devkit_docker.tar` in your current folder. **NOTE:** the image file produced will be quite large (around 4GB), so it is recommend that you compress the file before transferring it to the machine where it will be used offline. This can be done directly from the `docker save` command by piping the output to a compression utility such as `gzip` as follows: `docker save ghcr.io/sel4devkit/maaxboard:latest | gzip > sel4devkit_docker.tar.gz`. You can now transfer the saved image to the offline machine.

3. On the offline machine, decompress the image (if applicable), and save the  tar file in an appropriate location. From the terminal, navigate to the folder where the tar file is saved, and run the following command `docker load -i sel4devkit_docker.tar`.  Docker will now load the image from the tarball, this may take some time and it may appear that Docker has frozen, but once Docker has loaded the file, you should see the following output: 
```
% docker load -i sel4devkit_docker.tar
Loaded image: ghcr.io/sel4devkit/maaxboard:latest
```

4. The docker image can now be used as detailed throughout the documentation, for more information, see [Build Environment Setup](build_environment_setup.md).

## Downloading source code for offline use
The source code for seL4 and the seL4 developer kit is stored across multiple repositories. The `repo` command line tool is used alongside manifest files to make downloading and managing the code from these multiple repositories easier. A manifest file details what repos and versions of those repos are required for a certain project. There are two main manifests for the seL4 developer kit:

* `camkes-manifest` - used to download the source for building CAmkES applications
* `sel4test-manifest` - used for downloading the source for building the seL4 test program for the Avnet MaaXBoard

In order to download source code for offline use, you will need an internet connected computer with the `repo` tool installed, see [here](https://gerrit.googlesource.com/git-repo/) for details on how to install `repo`.

1. From a computer with the repo command line tool installed, create a new folder to store the source code and navigate into it from the command line.
2. In the new folder, run the following command to initialise the repo tool in this folder: `repo init -u https://github.com/sel4devkit/camkes-manifest.git`, replacing `cakes-manifest.git`with `sel4test-manifest.git` if you wish to download the source for seL4 test. Repo will now perform the required initialisation. If prompted, enter your GitHub credentials 
3. Once the initialisation has completed, a sync must be performed in order to download the source code. To do this run `repo sync` from the same folder. Repo will now download the required files from the various repositories. If prompted, enter your GitHub credentials. 
4. Once the sync has completed, all the required source files will now be contained within the folder you created, and this folder can now be moved to the offline machine. Instructions for building can be followed exactly as detailed in the rest of the documentation.
5. If you need to download updated source code, the folder created can be moved back to a machine with internet access and the repo tool, and `repo sync` can be run again.

## Building U-Boot offline
1. Clone the maaxboard-uboot repo from https://github.com/sel4devkit/maaxboard-uboot.git
2. On a computer with an internet connection, run the `clone.sh` script to download the required files for building U-Boot
3. Once the clone is complete, you should see the following message:
```
+----------------------------------------------------------------+
|  Source code for compiling U-Boot now cloned.                  |
|  This folder can now be transferred to an offline computer     |
|  and U-Boot built by running build-offline.sh.                 |
|  If you ran build.sh, build will start now.                    |
+----------------------------------------------------------------+
```

4. You can now transfer the cloned repository folder to a offline machine, and build using `build-offline.sh`

## Downloading the pre-prepared SD card images
Two SD card images are availible for the Maaxboard as part of the developer kit, one containing a precompiled U-Boot image with a BOOT partition for holding seL4 executables and a FILESYS partition for use as a filesystem, and another containing the above in addtion to a precompiled seL4 test executable. These can be downloaded from the disk_images folder of the [maaxboard-prebuilt](https://github.com/sel4devkit/maaxboard-prebuilt/tree/master/disk_images) repository. Instructions for writing these two and SD card can be found in the [SD Card preparation](../sd_card_preparation.md) chapter.
