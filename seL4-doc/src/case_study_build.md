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

If there is a `fatal error: pico_device.h: No such file or directory`, simply re-run the `init_build` command above, followed by `ninja`. There is a known race condition with the CMake configuration of picoserver, with a workaround of running the command twice, or running CMake again, by typing `cmake .` (see [here](https://lists.sel4.systems/hyperkitty/list/devel@sel4.systems/thread/O5B42BFF4FZ2WSCPUK6C6QUAJHD6DETN/)).

## Preparing to Run

A successful build from the will result in an executable file called `capdl-loader-image-arm-maaxboard` in the `images` subdirectory. This should be copied to a file named `sel4_image` and then made available to the preferred loading mechanism, such as TFTP, as per [Execution on Target Platform](execution_on_target_platform.md).

Running the `security_demo` application requires the following:

- Connect a keyboard to the USB[^1] socket of the MaaXBoard;
- Establish an Ethernet connection between the MaaXBoard and the host machine, which can be direct or via a network, as outlined in [an earlier section](bootloader.md#loading-via-tftp) (e.g. it will already be in place if TFTP is being used to transfer executables).

[^1]: Note: Currently, only the top USB port on the Avnet MaaXBoard is active; the bottom USB port does not function. This is a feature of the power domains on the board, not the USB driver.

If the user has experience of running the [`picoserver_uboot` test application](uboot_driver_usage.md#test-application-picoserver_uboot), then elements of the `security_demo` application will be familiar. For example, from a terminal window on the host machine, we will use the `netcat` (`nc`) command (native to Linux or macOS, or available as a [download](https://nmap.org/ncat/) for Windows) to connect to the MaaXBoard, so this should be readied. The following command will be used:

```bash
nc xxx.xxx.xxx.xxx 1234
```

where `xxx.xxx.xxx.xxx` is the IP address of the MaaXBoard, as previously specified in the `init_build` call, and 1234 is the port number that has been programmed into the application.

## Running the Application

The application invokes three instances of the [U-Boot Driver Library](uboot_driver_library.md), so various sets of diagnostic messages are repeated on the CoolTerm display as the application starts. We should not be unduly concerned with some of the individual messages, such as `No ethernet found`, since only one of the library instances is configured to use Ethernet (the library invoked by the EthDriverUboot component), and amongst the other messages should be confirmation that it was successful, e.g. `Assigned ipv4 xxx.xxx.xxx.xxx to device eth0`.

When the application's initialisation has completed, we should see:

```text
run_uboot_command@uboot_wrapper.c:176 --- running command 'fatrm mmc 0:2 transmitter_log.txt' ---
run_uboot_command@uboot_wrapper.c:181 --- command 'fatrm mmc 0:2 transmitter_log.txt' completed with return code 0 ---
```

This is housekeeping by the application to delete any previous Transmitter logfile from the SD card, before it starts writing new log data.

Just as with the [`picoserver_uboot` test application](uboot_driver_usage.md#test-application-picoserver_uboot), the application may sporadically display `No such port ....` messages as it monitors traffic on the network; this is expected behaviour that may be ignored.

The application is now ready to perform various actions concurrently:

- if a key is pressed, the plaintext character will be encrypted into a ciphertext character;
- if it receives an Ethernet connection from a client on port 1234, it will send its ciphertext to the client (all ciphertext characters are buffered until an Ethernet client has connected);
- every 30 seconds, if it has any ciphertext characters that it has not yet logged to file, it will append them to the logfile on the SD card (these are buffered separately to the Ethernet data).

Plaintext messages may be typed on the keyboard, e.g.

```text
Hello world, testing 123. We love seL4!
```

Nothing will be seen, except ... to be continued 