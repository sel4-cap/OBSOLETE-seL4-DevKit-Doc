# Software Requirements

This developer kit is intended to have minimal software requirements on the host machine by using Docker containment as much as possible. The following applications are either needed or assumed on the host machine:

- [Docker](https://www.docker.com/get-started)
    - a Docker container provides the primary development environment
- [balenaEtcher](https://www.balena.io/etcher/) (often known simply as Etcher)
    - used for programming the SD card
- [CoolTerm](https://freeware.the-meiers.org/)
    - used for serial communication with the MaaXBoard

Each of these applications is available for Linux, macOS, and Windows machines. Whilst Docker is essential, alternative means of programming an SD card and serial communications are available, although the examples given in this documentation (configuration, screenshots etc.) will assume the applications named above.

Installation instructions are not reproduced here; please follow the instructions given by the application providers (follow the links above). Elevated user privileges that allow the installation of applications on the host machine will be needed.

Configuration of the applications is considered in the [Host Machine Setup](host_machine_setup.md) section.