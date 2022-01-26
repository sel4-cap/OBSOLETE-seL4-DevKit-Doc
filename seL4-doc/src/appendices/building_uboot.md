# Building U-Boot for the MaaXBoard

In order to build U-Boot, the seL4devkit Docker build environment is required, please see [Build Environment Setup](./build_environment_setup.md) to setup this up if you haven't already done so.

1. In a suitable location on your host machine, create a new directory with a relevant name, e.g. `maaxboard-uboot-build`

2. Using a terminal, run the following command to start the seL4devkit Docker environment, changing `/your/working/folder/path/here` to the absolute path of the folder you just created:

    - `docker run -it --rm -v /your/working/folder/path/here:/host:z ghcr.io/sel4devkit/maaxboard:latest`

    - For more information on starting the seL4devkit build environment, refer to the [usage section of 'Build Environment Setup'](../build_environment_setup#Usage).

3. Once the bash shell in your build environment has loaded, you can now clone the [maaxboard-uboot](https://github.com/sel4devkit/maaxboard-uboot) repository from `https://github.com/sel4devkit/maaxboard-uboot.git` using git.

4. Once git has successfully cloned the repository, a new folder called `maaxboard-uboot` should be created, containing a README file, a `build.sh` build script and a `firmware` folder.

5. Once you have verified you have the correct files, run the build script using `./build.sh`

6. `build.sh` will clone a number of git repositories and extract necessary files from them, after which you will be presented with a license agreement for the NXP firmware for the i.MX8. You can navigate this agreement with the up and down arrow keys. Assuming you are happy to accept the agreement, when you reach the end, when prompted type `y` to accept. *Note: if you decline the EULA, the build process will be terminated, since the firmware is required to build U-Boot*

7. `build.sh` will now clone some additional repositories and complete the build process. If this is successful you should see the following:
![successful-uboot-build](../figures/successful-uboot-build.png)

8. The generated `flash.bin` file is now ready to write to storage media. Please see [SD Card Preparation](./writing_uboot_to_media.md) for details on how to do this.
