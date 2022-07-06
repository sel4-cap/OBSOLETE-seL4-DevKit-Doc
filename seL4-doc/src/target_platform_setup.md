# Target Platform Setup

The Avnet MaaXBoard is packaged in a small box with listed kit contents of:

- 1 x MaaXBoard; and
- 1 x Heat Sink Kit.

Our boards came with the heat sink already attached; however, sales photographs of the board are often shown without the heat sink and it is conceivable that you may have to attach it yourself.

The following shows a plan view of the MaaXBoard with heat sink attached:

![Avnet MaaXBoard plan view](figures/avnet-maaxboard-plan.png)

For clarity, not all of the components have been labelled, only those to which we are going to refer. Full hardware details are given in the [MaaXBoard Hardware User Manual](https://www.avnet.com/wps/wcm/connect/onesite/1e83cac7-ebe8-4be4-8776-6781e3833d11/MaaXBoard-Hardware_UserManual-V1.2-EN.pdf?MOD=AJPERES&CACHEID=ROOTWORKSPACE.Z18_NA5A1I41L0ICD0ABNDMDDG0000-1e83cac7-ebe8-4be4-8776-6781e3833d11-nVsEcIl), including the full pin-out for the 40-pin GPIO connector, but we only use the three pins that are labelled in red in the photograph. For reference, pin 1 is also labelled in blue and the numbering is as follows:

![GPIO pin numbering](figures/GPIO-pin-out.png)

## Connecting to the MaaXBoard

Communications with the MaaXBoard from the host machine are via the USB-to-TTL serial UART cable. This converts between the voltage levels and protocols of the board's UART and the host machine's USB port. The USB end plugs into the host machine, and the flying leads are connected to the GPIO pins on the MaaXBoard.

The USB-to-TTL serial UART cable that we used has 3 flying lead connectors, but some cables have more connectors. The leads shown below in our photographs adhere to the following colour convention, but this should not necessarily be assumed for your cable and should be confirmed before connecting:

- Black = Ground;
- Yellow = Transmit;
- Orange = Receive.

![UART connector side 1](figures/uart-connector-side1.png)

![UART connector side 2](figures/uart-connector-side2.png)

The final photograph shows the MaaXBoard (anti-clockwise from the top):

- populated with a Micro SD card;
- connected to a power supply (USB-C);
- connected to a USB flash drive (may not be necessary if loading via Ethernet);
- connected to an Ethernet cable (may not be necessary if loading from USB);
- connected to a USB-to-TTL serial UART cable.

![Avnet MaaXBoard populated](figures/maaxboard-populated.png)
