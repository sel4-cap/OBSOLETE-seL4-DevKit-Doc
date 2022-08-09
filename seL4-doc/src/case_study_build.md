# Case Study Building and Running

This section builds and runs the `security_demo` demonstration application described in [Case Study Introduction](case_study_intro.md). All the host machine and target platform requirements described previously in this developer kit documentation are assumed.

## Building the Application

As usual, this assumes that the user is already running a Docker container within the [build environment](build_environment_setup.md), where we can create a directory and clone the code and dependencies.

```text
mkdir /host/security_demo
cd /host/security_demo
```

```bash
repo init -u https://github.com/sel4devkit/camkes-manifest.git
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

This command should be repeated as a workaround - see the [implementation note](#implementation-note) below:

```bash
../init-build.sh -DCAMKES_APP=security_demo -DPLATFORM=maaxboard -DPICOSERVER_IP_ADDR=xxx.xxx.xxx.xxx
```

Then run `ninja` as usual:

```bash
ninja
```

### Implementation note

There is a known race condition with the CMake configuration of picoserver, which, if only run once, results in `fatal error: pico_device.h: No such file or directory`. A workaround is to run the command twice, or run CMake again, by typing `cmake .` (see [here](https://lists.sel4.systems/hyperkitty/list/devel@sel4.systems/thread/O5B42BFF4FZ2WSCPUK6C6QUAJHD6DETN/)).

## Preparing to Run

A successful build will result in an executable file called `capdl-loader-image-arm-maaxboard` in the `images` subdirectory. This should be copied to a file named `sel4_image` and then made available to the preferred loading mechanism, such as TFTP, as per [Execution on Target Platform](execution_on_target_platform.md).

Running the `security_demo` application requires the following:

- Connect a keyboard to the USB socket[^1] of the MaaXBoard;
- Establish an Ethernet connection between the MaaXBoard and the host machine, which can be direct or via a network, as outlined in [an earlier section](bootloader.md#loading-via-tftp) (e.g. it will already be in place if TFTP is being used to transfer executables).

[^1]: Note: Currently, only the upper USB port on the Avnet MaaXBoard is active (i.e. the port furthest away from the PCB); the lower USB port does not function. This is a feature of the power domains on the board, not the USB driver.

If the user has experience of running the [`picoserver_uboot` test application](uboot_driver_usage.md#test-application-picoserver_uboot), then elements of the `security_demo` application will be familiar. For example, from a terminal window on the host machine, we will use the `netcat` (`nc`) command (native to Linux or macOS, or available as a [download](https://nmap.org/ncat/) for Windows) to connect to the MaaXBoard, so this should be prepared.

## Running the Application

The application invokes three instances of the [U-Boot Driver Library](uboot_driver_library.md), so various sets of diagnostic messages are repeated on the CoolTerm display as the application starts. We should not be unduly concerned with some of the individual messages, such as `No ethernet found`, since in this case only one of the library instances is configured to use Ethernet (i.e. the library invoked by the EthDriverUboot component), and amongst the other messages there should be confirmation that it was successful, e.g. `Assigned ipv4 xxx.xxx.xxx.xxx to device eth0`. There are also some `clk_register: failed ... (parent ...)` messages, which are harmless (a fault in U-Boot's clock driver for the MaaXBoard).

When the application's initialisation has completed, we should see:

```text
run_uboot_command@uboot_wrapper.c:176 --- running command 'fatrm mmc 0:2 transmitter_log.txt' ---
Net:   transmitter_log.txt: doesn't exist
run_uboot_command@uboot_wrapper.c:181 --- command 'fatrm mmc 0:2 transmitter_log.txt' completed with return code 1 ---
```

Note that on subsequent runs, the log file will exist (unless the user has intentionally deleted it), and the output will instead read:

```text
run_uboot_command@uboot_wrapper.c:176 --- running command 'fatrm mmc 0:2 transmitter_log.txt' ---
run_uboot_command@uboot_wrapper.c:181 --- command 'fatrm mmc 0:2 transmitter_log.txt' completed with return code 0 ---
```

In either scenario, this is housekeeping by the application to delete any previous Transmitter logfile from the SD card, before it starts writing new log data. The logfile is named `transmitter_log.txt` and is expected on the third partition of the SD card - see the FAT partition `FILESYS` established during the [Partitioning the SD Card appendix](appendices/partitioning_sd_card.md).

Just as with the [`picoserver_uboot` test application](uboot_driver_usage.md#test-application-picoserver_uboot), the application may sporadically display `No such port ....` messages as it monitors traffic on the network. This is expected diagnostic behaviour that may be ignored; indeed, the lack of any such messages may indicate for example that the Ethernet driver has not initialised properly.

The application is now ready to perform various actions concurrently:

1. If a key is pressed, the plaintext character will be encrypted into a ciphertext character;
2. If a client requests an Ethernet connection on port 1234, the application will establish the connection and transmit ciphertext to the client, continuing to do so until the client closes the connection;
3. Every 30 seconds, if there are any ciphertext characters that it has not yet logged to file, the application will append them to the logfile on the SD card.

For item (2), from a terminal window on the host machine, start `netcat` with the command:

```bash
nc xxx.xxx.xxx.xxx 1234
```

where `xxx.xxx.xxx.xxx` is the IP address of the MaaXBoard, as previously specified in the `init_build` call, and 1234 is the port number that has been programmed into the application.

In the CoolTerm window, a message such as the following will signify a successful connection:

```text
transmitter: Connection established with yyy.yyy.yyy.yyy on socket 1
```

where `yyy.yyy.yyy.yyy` is the IP address of the host machine.

If this connection is not established promptly, please refer to the previous [implementation note](uboot_driver_usage.md#implementation-note) relating to network connection delays.

If characters have been typed at the USB keyboard before making the `netcat` connection, then (ciphertext) characters will appear straightaway on the host machine, since the application stores characters in a buffer while they cannot be sent.

Plaintext messages may be typed on the keyboard (note that these are not displayed anywhere), for example:

```text
Hello world, testing 123. We love seL4!
```

Within the host machine's `netcat` session, the corresponding ciphertext message should appear:

```text
Uryyb jbeyq, grfgvat 123. Jr ybir frY4!
```

Periodically, the CoolTerm window will show diagnostic messages concerning the logfile, such as:

```text
run_uboot_command@uboot_wrapper.c:176 --- running command 'fatwrite mmc 0:2 0x55b010 transmitter_log.txt 26 1' ---
38 bytes written in 5 ms (6.8 KiB/s)
run_uboot_command@uboot_wrapper.c:181 --- command 'fatwrite mmc 0:2 0x55b010 transmitter_log.txt 26 1' completed with return code 0 ---
```

The application will continue indefinitely. `netcat` sessions on the host machine may be terminated (Ctrl-C) and restarted, whereupon the application will establish a new connection (buffering output in the meantime).

If the MaaXBoard is powered off and its SD card removed and transferred to the host machine, the `transmitter_log.txt` can be accessed from the `FILESYS` partition as one would expect.

To reduce the risk of corrupting the SD card, it is advisable to avoid powering off the MaaXBoard during a write operation of the logfile. As this operation lasts in the order of 10 ms and only occurs every 30 seconds (and only then if there are any ciphertext characters that have not yet been logged to file), the risk is both remote and avoidable (e.g. power off soon after a `fatwrite` command message, or wait for >30 seconds after the last `fatwrite` command message).
