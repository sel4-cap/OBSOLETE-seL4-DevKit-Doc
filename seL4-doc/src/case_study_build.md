# Building the Case Study Application

This section builds and runs the `security_demo` demonstration application described in [Case Study Introduction](case_study_introduction.md). All the host machine and target platform requirements described previously in this developer kit documentation are assumed.

## Building the Application

As usual, this assumes that the user is already running a Docker container within the [build environment](build_environment_setup.md), where we can create a directory and clone the code and dependencies.

```text
mkdir /host/security_demo
cd /host/security_demo
```

_Temporary-note: below is currently on maaxboard-usb branch; it will ultimately change_

```bash
repo init -u https://github.com/sel4devkit/camkes-manifest.git -b maaxboard-usb
```

```bash
repo sync
```

The application requires an IP address to be allocated to PicoServer. This should be substituted for `xxx.xxx.xxx.xxx` below. From the `/host/security_demo` directory, we execute the following commands:

```text
mkdir build
cd build
```

```bash
../init-build.sh -DCAMKES_APP=security_demo -DPLATFORM=maaxboard -DPICOSERVER_IP_ADDR=xxx.xxx.xxx.xxx
```

```bash
ninja
```

#### Implementation note

If there is a `fatal error: pico_device.h: No such file or directory`, simply re-run the `init_build` command above, followed by `ninja`. There is a known race condition with the CMake configuration of picoserver (see [here](https://lists.sel4.systems/hyperkitty/list/devel@sel4.systems/thread/O5B42BFF4FZ2WSCPUK6C6QUAJHD6DETN/)).


## Running the Application

To be continued...