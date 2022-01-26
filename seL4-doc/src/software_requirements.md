# Software Requirements

This developer kit is intended to have minimal host machine software requirements by using Docker containment as much as possible.

Installation instructions are not reproduced here; please follow the instructions given by the application providers.

Configuration of the applications is considered in the [Host Machine Setup](host_machine_setup.md) section.

This guide assumes that the user has the required host environment permissions to install and configure the applications, for example on macOS and Linux it is assumed that the user has `sudo` privileges.

## Mandatory Requirements

The following mandatory applications are assumed on the host machine:

- **Container toolset**: Used to provide the build environment. This guide assumes use of [Docker Desktop](https://www.docker.com/products/docker-desktop) for macOS or Windows host environments or [Docker Engine](https://hub.docker.com/search?offering=community&operating_system=linux&q=&type=edition) for Linux based host environments.

- **SD card writer application**: Used to write the SD card from which the MaaXBoard boots. [Balena Etcher](https://www.balena.io/etcher/) has been selected due to its availability on macOS, Windows and Linux environments. Alternative means of writing SD cards are available but not supported by this document.

- **Serial terminal application**: Used for serial communication with the target platform. [CoolTerm](https://freeware.the-meiers.org/) has been selected due to its availability on macOS, Windows and Linux environments. Alternative applications are available but not supported by this document.

## Optional Requirements

**TBD**
