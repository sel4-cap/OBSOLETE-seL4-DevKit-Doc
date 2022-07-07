# Case Study Design Details

Following on the [case study introduction](case_study_intro.md) this section provides further details on the design and functionality of the 'security domain' demonstrator application.

It is expected that the reader is familiar with the [seL4 CAmkES manual](https://docs.sel4.systems/projects/camkes/manual.html).

## Code Structure

The application is held within the following structure with key folders and files shown:

```text
security_demo
│
├───components
│   ├───<component_1>
│   │   ├───src
│   │   │   └───<component_source_file>.c
│   │   └───<component_camkes_file>.camkes
│   ...
│   └───<component_n>
│       ├───src
│       │   └───<component_source_file>.c
│       └───<component_camkes_file>.camkes
│
├───include
│   └───plat
│       ├───<platform>
│       │   ├───eth_platform_devices.h
│       │   ├───mmc_platform_devices.h
│       │   └───usb_platform_devices.h
│       └───dataport_buffer.h
│
├───interfaces
│   ├───Character_RPC.idl4
│   └───Lock_RPC.idl4
│
├───CMakeLists.txt
└───security_demo.camkes
```

- `CMakeLists.txt`: Application build fie.
- `security_demo.camkes`: Top-level CAmkES file for the project, declaring the application's CAmkES 'assembly' including declaration of all inter-component connections.
- `components/<component>/<component_camkes_file>.camkes`: A component's CAmkES file, declaring the component's attributes including declaration of all interfaces (i.e. exposed inter-component interaction points).
- `interfaces/Character_RPC.idl4`: Declaration of a remote procedure call method used to communicate a character between components.
- `interfaces/Lock_RPC.idl4`: Declaration of remote procedure call methods used to access a mutex held by another component.
- `include/dataport_buffer.h`: Declaration of a C data type for a circular buffer to be stored in a dataport.
- `include/<platform>/<eth|mmc|usb>_platform_devices.h`: Declaration of (platform specific) CAmkES attributes to grant a component with capabilities to access a hardware device.

** Talk about key files and what they do.

** The CAmkES files for each component exposes its interfaces (e.g. dataports, procedureal interfaces and notifications).
** The 'assembly' defines the connections between the component interfaces.

## Concurrency Model

** Talk about threads.
** Communication between them, locking, etc.
** Mutex, critical region.

## Example of Inter-Component Communication

** Introduce three core inter-component comms mechanisms (with reference to the CAmkES manual).

** Talk about the Crypto -> Transmitter link. All three methods used.