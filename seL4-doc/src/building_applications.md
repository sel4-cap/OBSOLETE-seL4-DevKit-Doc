# Building Applications

For consistency with the methods employed by the [seL4 Foundation](https://sel4.systems), a [CMake](https://cmake.org) based build system is used. Details on the seL4 build system are documented [here](https://docs.sel4.systems/projects/buildsystem/).

As a worked example throughout this section the [seL4Test](https://docs.sel4.systems/projects/sel4test) project will be used. seL4Test is a test suite for seL4 developed and maintained by the seL4 Foundation.

This section of the document assumes the [build environment setup](build_environment_setup.md) has been completed. All commands provided within this section are to be executed within the build environment.

## Getting the Code

Within the build environment a directory named `seL4Test` will be created and all of the code and dependencies (e.g. the seL4 kernel, libraries and requited tools) will be cloned:

```bash
# Create a directory to hold the code
mkdir /host/seL4Test
cd /host/seL4Test
# Clone the code and dependencies
repo init -u https://github.com/seL4/sel4test-manifest.git
repo sync
```

**NOTE TO ROD**: Until MaaXBoard support has been merged back to the seL4 repos the `repo init` line in the above command block needs to be `repo init -u https://github.com/sel4devkit/sel4test-manifest.git`.

## Building

To support building with multiple different configurations within a single project, a directory is created for each configuration to contain the build artefacts. Different configurations are used for different platforms, architectures or other build options.

For the purposes of this example two configurations are used targeted the MaaXBoard, one for AArch32 and one for AArch64.

The executable files produced can be executed on the target platform by following the guidance in the [Execution on Target Platform](execution_on_target_platform.md) section.

### AArch32 Build Instructions

```bash
# Create directory for compilation artefacts
mkdir /host/seL4Test/build-MaaXBoard-AArch32
cd /host/seL4Test/build-MaaXBoard-AArch32
# Configure build directory
../init-build.sh -DPLATFORM=maaxboard -DAARCH32=TRUE
# Compile
ninja
```

On completion of the compilation the resulting executable is available at `/host/seL4Test/build-MaaXBoard-AArch32/images/sel4test-driver-image-arm-maaxboard`.

### AArch64 Build Instructions

```bash
# Create directory for compilation artefacts
mkdir /host/seL4Test/build-MaaXBoard-AArch64
cd /host/seL4Test/build-MaaXBoard-AArch64
# Configure build directory
../init-build.sh -DPLATFORM=maaxboard -DAARCH64=TRUE
# Compile
ninja
```

On completion of the compilation the resulting executable is available at `/host/seL4Test/build-MaaXBoard-AArch64/images/sel4test-driver-image-arm-maaxboard`.
