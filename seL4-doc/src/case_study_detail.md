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

From a concurrency perspective the security demo can be summarised as threads of execution on the high-side managing plain-text data running asynchronously to  threads of execution on the 'low-side' managing cipher-text data.

Threads of execution on the high-side perform the following functions:

1. Reading plain-text keypresses from the USB keyboard;
2. Encrypting those keypresses from plain-text to cipher-text; and
3. Placing cipher-text keypresses into the shared circular buffer between the high-side and low-side.

Whilst threads of execution on the low-side perform the following functions:

1. Reading cipher-text keypresses from the shared circular buffer between the high-side and low-side;
2. Periodically writing received cipher-text to a log file; and
3. Outputting cipher-text to a network socket (if connected).

All data is passed from the high-side to the low-side through a shared circular buffer stored within a dataport. It is this circular buffer between the high-side and low-side that permits:

1. Data (the cipher-text) to be passed from the high-side to the low-side; and
2. Threads of execution on the high-side and threads of execution on the low-side to run asynchronously.

## Example of Inter-Component Communication

The circular buffer shared between the Crypto component (hide-side) and the Transmitter component (low-side) is used in this section as a detailed worked example of CAmkES inter-component data flow and control flow.

The buffer has been deliberately designed for the purposes this worked example to use of all three types of component interface listed in the [seL4 CAmkES manual](https://docs.sel4.systems/projects/camkes/manual.html), i.e. *procedure*, *event* and *port*.

### Port

At its core the circular buffer is a simple character array with *head* and *tail* pointers capturing the array indexes associated with the start and end of the used portion of the buffer.

- Further details are documented alongside the definition of the `dataport_buffer_t` type definition in `include/dataport_buffer.h`.
- *Port* interfaces of data type `dataport_buffer_t` are declared in the Crypto and Transmitter component CAmkES files.
- An `seL4SharedData` connection the two *port* interfaces is then declared in the CAmkES assembly (see `security_demo.camkes`).

This results in an instance of the circular buffer type being made available in an area of memory shared by both the Crypto and Transmitter components.

### Procedure

Reading data from, or writing data to, the circular buffer requires the data array, *head* pointer, and *tail* pointer to be modified. Such modifications to the buffer cannot be allowed to occur concurrently by both the Crypto and Transmitter components otherwise corruptions of the buffer may occur. As such access to the buffer across the two components must be protected to avoid concurrent access.

Within the security demo a mutex is used to enforce this critical section; each component must hold the lock on the mutex prior to accessing the circular buffer, and must release the lock when access to the circular buffer has been completed.

The mutex is owned by the Crypto component, see definition of `circular_buffer_mutex` in the Crypto component's CAmkES file. To allow the Transmitter component to access the mutex a *procedure* interface is used providing `lock` and `unlock` routines.

- Declaration of the `lock` and `unlock` *procedure* templates are provided by `interfaces/Lock_RPC.idl4`.
-



### Event

TBC

