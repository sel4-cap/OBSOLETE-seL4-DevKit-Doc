# Case Study Design Detail

Following on from the [case study introduction](case_study_intro.md), this section provides further details on the design and functionality of the 'security domain' demonstrator application.

It is expected that the reader is familiar with the [seL4 CAmkES manual](https://docs.sel4.systems/projects/camkes/manual.html).

## Code Structure

The application is held within the following structure (all within the `camkes/apps` folder), with key folders and files shown:

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

- `CMakeLists.txt`: Application build file.
- `security_demo.camkes`: Top-level CAmkES file for the project, declaring the application's CAmkES 'assembly' including declaration of all inter-component connections.
- `components/<component>/<component_camkes_file>.camkes`: A component's CAmkES file, declaring the component's attributes including declaration of all interfaces (i.e. exposed inter-component interaction points).
- `interfaces/Character_RPC.idl4`: Declaration of a remote procedure call method used to communicate a character between components.
- `interfaces/Lock_RPC.idl4`: Declaration of remote procedure call methods used to access a mutex held by another component.
- `include/dataport_buffer.h`: Declaration of a C data type for a circular buffer to be stored in a dataport.
- `include/<platform>/<eth|mmc|usb>_platform_devices.h`: Declaration of (platform specific) CAmkES attributes to grant a component with capabilities access to a hardware device.

## Concurrency Model

From a concurrency perspective, the security demonstrator can be summarised as threads of execution on the 'high-side' managing plaintext data running asynchronously to threads of execution on the 'low-side' managing ciphertext data.

Threads of execution on the high-side perform the following functions:

1. Reading plaintext keypresses from the USB keyboard;
2. Encrypting those keypresses from plaintext to ciphertext; and
3. Placing ciphertext keypresses into a shared circular buffer between the high-side and low-side.

Asynchronously, threads of execution on the low-side perform the following functions:

1. Reading ciphertext keypresses from a shared circular buffer between the high-side and low-side;
2. Periodically writing received ciphertext to a log file; and
3. Outputting ciphertext to a network socket (if connected).

All data is passed from the high-side to the low-side through a shared circular buffer stored within a dataport. It is this circular buffer between the high-side and low-side that permits:

1. Data transfer from the high-side to the low-side; and
2. Threads of execution on the high-side and threads of execution on the low-side to run asynchronously.

## Example of Inter-Component Communication

The circular buffer shared between the Crypto component (high-side) and the Transmitter component (low-side) is used in this section as a detailed worked example of CAmkES inter-component data flow and control flow.

The buffer has been deliberately designed for the purpose of this worked example to use of all three types of component interface listed in the [seL4 CAmkES manual](https://docs.sel4.systems/projects/camkes/manual.html), i.e. *procedure*, *event* and *port*.

### Port

At its core the circular buffer is a simple character array with *head* and *tail* holding indexes associated with the start and end of the used portion of the array.

- Further details are documented alongside the definition of the `dataport_buffer_t` type definition in `include/dataport_buffer.h`.
- *Port* interfaces of data type `dataport_buffer_t` are declared in the Crypto and Transmitter component CAmkES files.
- An `seL4SharedData` connection between the two *port* interfaces is then declared in the CAmkES assembly (see `security_demo.camkes`).

This results in an instance of the circular buffer type being made available in an area of memory shared by both the Crypto and Transmitter components.

### Procedure

Reading data from, or writing data to, the circular buffer requires the data array, *head* index, and *tail* index to be modified. Such modification of the buffer cannot be allowed to occur concurrently by both the Crypto and Transmitter components, otherwise corruption of the buffer may occur. Access to the buffer by the two components must therefore be protected to avoid concurrent access.

Within the security demonstrator, a mutex is used to enforce this critical section; each component must hold the lock on the mutex prior to accessing the circular buffer, and must release the lock when access to the circular buffer has been completed.

The mutex is owned by the Crypto component; see definition of `circular_buffer_mutex` in the Crypto component's CAmkES file. To allow the Transmitter component to access the mutex, a *procedure* interface is used providing `lock` and `unlock` routines.

- Declaration of the `lock` and `unlock` *procedure* templates are provided by `interfaces/Lock_RPC.idl4`.
- *Procedure* interfaces using the `lock` and `unlock` templates are declared in the Crypto and Transmitter component CAmkES files.
- An `seL4RPCCall` connection between the two *procedure* interfaces is then declared in the CAmkES assembly (see `security_demo.camkes`).

This results in the Crypto component being supplied with a mutex and remote procedure call interfaces being supplied to the Transmitter component to allow it to access the mutex.

### Event

Whilst a functional system could be produced with just the *port* and *procedure* interfaces described above, there would be no mechanism for the Transmitter component to determine whether the circular buffer contains any data other than to periodically poll the contents of the circular buffer; this would be needlessly inefficient.

Instead, an *event* interface is used to allow the Crypto component to notify Transmitter when a character has been written into the circular buffer. This allows the Transmitter component to wait or poll for such a notification before locking the mutex and accessing the circular buffer.

- *Event* interfaces, with the Crypto component as the emitter and Transmitter as the consumer, are declared in the Crypto and Transmitter component CAmkES files.
- An `seL4Notification` connection between the two *event* interfaces is then declared in the CAmkES assembly (see `security_demo.camkes`).

This results in the creation of a notification that can be emitted from the Crypto component and can be either waited on, or polled, by the Transmitter component.

### Summary

Putting all three interface types (*port*, *procedure*, and *event*) together results in:

- A shared circular buffer;
- A mutex which can be accessed via remote procedure calls to protect the buffer against concurrent access; and
- A notification mechanism to allow the producer to notify the consumer when new data is available.

This thereby allows asynchronous data transfer and buffering between components such that the producer and consumer can work concurrently.

The example source code for the producer component demonstrating use of these inter-component communication mechanisms can be found in `components/Crypto/src/crypto.c`. The source code for the consumer component can be found in `components/Transmitter/src/transmitter.c`.
