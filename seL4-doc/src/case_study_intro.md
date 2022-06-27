# Case Study Introduction

This section works through an example to show some of our device drivers being used within a larger CAmkES application. It is nominally labelled as a 'security domain' demonstrator (named `security_demo`) but this is mainly in the context of a simple keyboard-based encryption device that was inspired by an Enigma machine! Its main purpose is to show notional data separation and to provide worked examples of inter-component communications using different seL4 mechanisms.

As with earlier sections of this developer kit documentation, we continue to defer to the seL4 Foundation's [documentation of CAmkES](https://docs.sel4.systems/projects/camkes/) for the bulk of its understanding, but this section will cover aspects of the use of CAmkES where appropriate.

## Overview

The architecture of the demonstrator is shown below.

_Temporary_note: This is a working diagram that will probably be updated_

![Demonstrator architecture](figures/encrypter_arch.png)

Arrow directions show an abstracted view of data flow. Arrow labels list seL4 connection types (some concerned with data flow, some with control flow), which are elaborated in the key. Blue blocks show components created specifically for the security demonstrator (or previously created within the developer kit in the case of [Ethdriver](uboot_driver_usage.md#test-application-picoserver_uboot)); grey blocks show global components.

Plaintext characters are typed on a keyboard and read by the KeyReader component. These characters are then encrypted by the Crypto component to transform them into ciphertext. Since this application is more concerned with demonstrating seL4 concepts than crypto-algorithms, the Enigma machine's rotors and plugboard are replaced with a simple [ROT13](https://en.wikipedia.org/wiki/ROT13) algorithm!

As the KeyReader and Crypto components handle plaintext, they are considered as 'high side' and must be kept separate from the downstream 'low side' components that handle ciphertext.

The encrypted characters are transferred to the Transmitter component via a shared circular buffer, where Crypto writes to the Head of the buffer and Transmitter reads from its Tail. If the buffer is full, Crypto discards characters; otherwise, each time Crypto writes a character to the buffer, it sends a notification to Transmitter to signify that there is data to read. Transmitter acts upon a notification by reading all available characters until the buffer is empty.

The shared buffer is protected from concurrent access by use of a [mutex](https://en.wikipedia.org/wiki/Lock_(computer_science)).