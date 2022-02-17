# Guide to Porting seL4

The level of work required and complexity of porting seL4 to a new platform is directly related to the level of similarity between the system to be ported and systems already supported by seL4. Porting seL4 to a new system based upon a SoC (system-on-chip) already supported by seL4 is likely to relatively straightforward whilst porting seL4 to a system with a currently unsupported SoC is likely to require more work, e.g. requiring the addition of new serial and timer drivers.

As a starting point the user should read and understand the excellent guidance on porting supplied by the seL4 Foundation at [this link](https://docs.sel4.systems/projects/sel4/porting.html).

The remainder of this section does not seek to repeat any information in the seL4 documentation, instead it provides more detailed guidance and worked examples based upon the experience of porting seL4 to the Avnet MaaXBoard.

## Device Tree (DTS)

A device tree (DTS file) contains plain text data that describes a platform's hardware. A key requirement in porting seL4 to a new platform is the provision of a device tree describing the platform. A device tree can normally either be sourced from the Linux kernel or from a Linux distribution supplied by the platform manufacturer.

Device tree files are stored within the Linux kernel within the `/arch/<arch_identifier>/boot/dts` folders. For example, for ARM based devices DTS files may be retrieved from [this link](https://github.com/torvalds/linux/tree/master/arch/arm/boot/dts).

Support has not been incorporated into the mainline Linux kernel for the Avnet MaaXBoard, and therefore the device tree needed to be sourced from the platform-specific Linux distribution supplied by Avnet at [this link](https://github.com/Avnet/linux-imx/blob/maaxboard_5.4.24_2.1.0/arch/arm64/boot/dts/freescale/maaxboard-base.dts).

Once a DTS file has been located it needs to be processed into a form suitable for inclusion in seL4, e.g. to remove use of C-style includes and macros. For the Avnet MaaXBoard this processing was performed using the following commands to generate DTS file named `maaxboard.dts`. It is expected the commands should be easily modifiable for other boards and sources of the Linux kernel.

```sh
git clone https://github.com/Avnet/linux-imx.git
cd linux-imx
cpp -nostdinc -I include -I arch -undef -x assembler-with-cpp \
   arch/arm64/boot/dts/freescale/maaxboard-base.dts temp.dts
dtc temp.dts temp.dtb
dtc -I dtb -O dts temp.dtb > maaxboard.dts
```

Following creation of the platform's DTS file it should be stored in the `/tools/dts` folder of the `seL4` git repository and the `update-dts.sh` script (located in the same location) updated to reference the new DTS file.

## Previous Worked Examples

To help fully understand the guidance supplied by the seL4 documentation it is considered to be informative to examine the changes made to the seL4 repositories made previously to support new platforms. To that end the following links are to commits in the seL4 git repositories where support for platforms was added. It should be noted to perform a basic port of seL4 to a new platform, i.e. sufficient to run and pass the seL4Test application, will require modification to the [`sel4`](https://github.com/seL4/seL4), [`seL4_tools`](https://github.com/seL4/seL4_tools) and [`util_libs`](https://github.com/seL4/util_libs) git repositories.

- Avnet MaaXBoard - Port to platform with existing SoC support
  - TBC

- ODroid C4 - Port to platform using an SoC very similar to previously supported SoC
  - [seL4 commit](https://github.com/seL4/seL4/commit/76b1de0670fc09df279883be570f4a518bad4745)
  - [seL4_tools commit](https://github.com/seL4/seL4_tools/commit/83c1891fb007c945e61d04887e49dd1a82eb9b69)
  - [util_libs commit](https://github.com/seL4/util_libs/commit/8ed42ff66c586ab2b6dc32f678e00764453754f0)

- Raspberry Pi 4 - Port to new SoC requiring new serial and timer drivers
  - [seL4 commit](https://github.com/seL4/seL4/commit/2a0e5a2a1fbbb6706e79cf12d0efd7f1004b3389)
  - [seL4_tools commit](https://github.com/seL4/seL4_tools/commit/f086b4b818d51519a6d00f6934d97c2a3a834cbe)
  - [util_libs commit](https://github.com/seL4/util_libs/commit/b6f99879e59950f39ee35472ab15eac1efb0f139)

## Testing the Port With seL4Test

To confirm the correct functionality of a port it is recommended that the [seL4Test](https://docs.sel4.systems/projects/sel4test/) test suite is executed.

To build and execute seL4Test for testing purposes against forks of the seL4 git repositories the following procedure can be used. For the purposes of this example it is envisioned that the engineer has forked the [`sel4`](https://github.com/seL4/seL4), [`seL4_tools`](https://github.com/seL4/seL4_tools) and [`util_libs`](https://github.com/seL4/util_libs) git repositories under GitHub account `work_account` and performed the required modifications under branches named `my_port`. When following the instructions the commands will need to be modified to match the name of the engineer's GitHub account and branch name.

1. Create a fork of the [`sel4test-manifest`](https://github.com/seL4/sel4test-manifest) repository through the GitHub web interface and create branch `my_port`.

2. Modify file `default.xml` in the `my_port` branch of the forked `sel4test-manifest` repository to point at the engineer's forks / branches. This will require the following changes.

   - Add a new `remote` to the manifest:

        ```xml
        <remote name="work_account" fetch="https://github.com/work_account"/>
        ```

   - For each modified repository update the associated `project` entry . For example, the entry for the `seL4` repository would be changed from:

        ```xml
        <project name="seL4.git" path="kernel" revision="..." upstream="master" dest-branch="master"/>
        ```

      To the following:

        ```xml
        <project name="seL4.git" path="kernel" remote="work_account" revision="my_port" upstream="my_port" dest-branch="my_port"/>
        ```

3. Check out the newly updated manifest within the build environment:

    ```sh
    mkdir /host/seL4test
    cd /host/seL4test
    repo init -u https://github.com/work_account/sel4test-manifest.git -b my_port
    repo sync
    ```

4. Building and executing seL4Test can be performed by following the instructions previously provided in the [Building Applications](../building_applications.md) and [Execution on Target Platform](../execution_on_target_platform.md) sections with the build commands modified appropriately to target the newly added platform.
