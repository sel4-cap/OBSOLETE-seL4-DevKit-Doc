# Software Requirements

This developer kit is intended to have minimal software requirements on the host machine by using Docker containment as much as possible. The applications that follow are either needed or assumed on the host machine.

Installation instructions are not reproduced here; please follow the instructions given by the application providers. Configuration of the applications is considered in the [Host Machine Setup](host_machine_setup.md) section.

This guide assumes that the user has sufficiently elevated privileges on the host machine to install and configure the applications; for example on macOS and Linux it is assumed that the user has `sudo` privileges.

## Mandatory Requirements

The following mandatory applications are assumed on the host machine:

- **Container toolset**: Used to provide the build environment. This guide assumes use of [Docker Desktop](https://www.docker.com/products/docker-desktop) for macOS or Windows host environments or [Docker Engine](https://hub.docker.com/search?offering=community&operating_system=linux&q=&type=edition) for Linux-based host environments.

- **SD card writer application**: Used to write the SD card from which the MaaXBoard boots. [Balena Etcher](https://www.balena.io/etcher/) has been selected due to its availability on macOS, Windows and Linux environments. Alternative means of writing SD cards are available but not supported by this document.

- **Serial terminal application**: Used for serial communication with the target platform. [CoolTerm](https://freeware.the-meiers.org/) has been selected due to its availability on macOS, Windows and Linux environments. Alternative applications are available but not supported by this document.

## Optional Requirements

If the user wishes to perform TFTP transfer of executables from the host environment to the target platform (see the [Bootloader](bootloader.md) section) then installation of a TFTP server is required.

Full details of the installation and configuration of a TFTP server is considered to be outside the scope of this developer kit, however the following guidance is offered:

- **macOS**: A TFTP server is included in macOS by default. The [TftpServer](https://www.macupdate.com/app/mac/11116/tftpserver) application provides a graphical user interface to simplify use of the in-built server.

- **Linux**: TFTP servers are available for all major Linux distributions. On Fedora and CentOS a server can be installed via `yum -y install tftp-server`; on Debian and Ubuntu a server can be installed via `apt-get install tftpd-hpa`.

- **Windows**: Multiple free TFTP server applications are available, e.g. [here](https://www.solarwinds.com/free-tools/free-tftp-server) and [here](https://pjo2.github.io/tftpd64).
