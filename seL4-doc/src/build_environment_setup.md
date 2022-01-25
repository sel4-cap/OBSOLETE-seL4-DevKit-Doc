# Build Environment Setup

The build environment is the environment within which all binaries for the target environment are built, i.e. it the environment that provides the required compilation toolchain.

To both simplify the requirements on the host environment and to enable rapid setup the build environment is provided as a pre-built Docker image.

## Environment Details

The build environment is a Debian Linux system pre-installed with all of the required development tools. The primary tools supplied are:

- [__Git__](https://git-scm.com) version control system.
- [__Repo__](https://gerrit.googlesource.com/git-repo/+/refs/heads/master/README.md) Git repository management tool.
- [__aarch64-linux-gnu__](https://gcc.gnu.org) cross compiler toolchain targeting the ARM AArch64 instruction set.
- [__arm-linux-gnueabi__](https://gcc.gnu.org) cross compiler toolchain targeting the ARM AArch32 instruction set.
- [__make__](https://www.gnu.org/software/make/), [__cmake__](https://cmake.org), and [__ninja__](https://ninja-build.org) build automation tools.

## Installation

[Host machine setup](host_machine_setup.md) must be completed, specifically installation of Docker, and the Docker tools must be active prior to installation of the build environment.

Installation of the build environment comprises the download of a pre-built Docker image into the Docker tools. This can be performed from the command line as follows:

```
docker pull ghcr.io/sel4devkit/maaxboard:latest
```

During download of the image progress is reported in the following format:

```
% docker pull ghcr.io/sel4devkit/maaxboard:latest
latest: Pulling from sel4devkit/maaxboard
0e29546d541c: Pull complete 
c2db25fafa50: Pull complete 
00af15d3de8e: Downloading [==========>                                ]  241.7MB/1.188GB
c6749363a298: Download complete 
521655c15371: Download complete 
```

On completion of the download the build environment is ready for use.

## Usage

xxx