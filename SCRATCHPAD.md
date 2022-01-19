------ Overview of intended structure and content of each section ------

Introduction / general overview - **STEPHEN**
- Development kit is based around the Avnet MaaxBoard target board, an iMX8 based SBC.
- Guide is expected to be followed on either a macOS or Linux based host (however the instructions could be adapted for use on windows with Cygwin by an experienced user).
- Guide will provide all of the information required up to and including the building and execution of an seL4 application for target board.
- Document that all compilation for the target board will be performed within a Docker instance. This greatly simplifies the requirements on the host system.
- Document that the guide will be talking about three different environments (host, target and docker build environment). Explain this split and how they will be consistently referred to throughout the guide.

Hardware requirements. - **JOSH**
- Some are optional (e.g. a USB memory stick) depending on how a binary is loaded.
- Provide a shopping list from Farnell of all of the required parts (including memory stick. SD card reader/writer if needed).

Host software requirements. - **JOSH**
- Some are optional (e.g. a TFTP server) depending on how a binary is loaded.

Host machine setup
- Installation and setup of required tools
- Installation of Docker.
- Creation and usage of the build environment docker image

Target platform setup **-MARK**
- How to connect up the MaaxBoard including the TTY connections. **-MARK**
- Need good quality pictures in this section. **-MARK**

Bootloader
- Describe why we need U-Boot, i.e. what it does for us.
- Instructions on how to build U-Boot for the MaaxBoard. - **JOSH**
- Need to explain that the application needs to be loaded into RAM and then executed by our bootloader.
- Explain that there are three primary mechanisms for loading the application into RAM by U-Boot (additional mechanisms such as download over the serial console are available but are very slow so have not been documented. Also download from on-board flash memory is possible but is not applicable to our target board). Need to discuss what are the advantages and disadvantages of each of the mechanisms we document, but that it’s up to the user to decide what works best for them.
    - From SD Card
    - From USB storage device
    - From TFTP - Best for a d=an application development environment as no need to keep plugging / unplugging anything from the board.
- Need to document the key U-Boot commands that are required.
- Talk about how U-Boot operation can be configured by uEnv.txt file and provide our file to support this.

SD card preparation
- Partitioning of the card (commands / instructions for both macOS and Linux) - **JOSH**
- Writing U-Boot boot loader. - **JOSH**
    - Need some warnings here about ‘dd’. Incorrect use can destroy your data / system.
- Placement of uEnv.txt. - **JOSH**

First boot
- U-Boot only.
- Demonstrate access to the U-Boot console.

Building applications and executing on the target platform
- Use seL4-test as a worked example.
- Build instructions.
- Refer back to the section on U-Boot to show how the binary can be loaded into RAM and executed.

Appendices
- Rough guide on getting seL4 building for a new iMX8 board.
