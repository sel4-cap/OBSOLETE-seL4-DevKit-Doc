# Background

The [seL4 Microkernel](https://sel4.systems) is a technology of increasing interest and has potential to deliver significant benefit across a range of applications that require strong domain separation and software enforced trusted execution environments.

seL4 is formally verified software and therefore offers significant benefits over other software based implementations in terms of the assurance and trust that can be gained. Given it is open source COTS technology the potential for reduced cost for high assurance products and reduced time to market are significant.

Developing on top of the seL4 microkernel is currently complex and time consuming, and the expertise is currently limited to a small number of individuals and organisations. This developer kit is targeted at reducing the barrier to entry for using (i.e. developing on top of) the seL4 technology and increasing the number of companies that can cost-effectively adopt seL4 in their products.

# Introduction

This document presents an entry-level developer kit to help users gain familiarity with using the seL4 microkernel and ultimately to encourage more use of seL4 within projects and products. It does this by:
- selecting a readily available, low-cost single board computer (the [Avnet MaaXBoard](https://www.avnet.com/wps/portal/us/products/avnet-boards/avnet-board-families/maaxboard/maaxboard/));
- detailing the minimum hardware and software requirements necessary to follow the guide;
- supplying the required development tools to be able to generate seL4 executable images;
- demonstrating how an seL4 executable image can be loaded and executed on the chosen development board; and
- documenting the specific steps needed along with more general guidance.

This developer kit does not cover or replace any of the extensive documentation and tutorials of the [seL4 website](https://sel4.systems). 

## Audience

The intended audience for this developer kit is an application developer who:
- has experience programming within a Linux/macOS environment;
- is already familiar with seL4's concepts and benefits; and
- is looking for more practical guidance to get started with seL4.

Whilst the developer kit documentation aims to cover all the salient points and commands, it assumes a reasonable level of developer knowledge and does not necessarily cover every detail of 'standard' Linux/macOS environments (for example, it covers seL4-related installation, but does not cover, say, installation of macOS Xcode command line tools).

## Development Environments

Throughout this developer kit activities will be performed across three different environments:

1. **Host environment**: xxx. 

2. **Build environment**: xxx.

3. **Target environment**: The [Avnet MaaXBoard](https://www.avnet.com/wps/portal/us/products/avnet-boards/avnet-board-families/maaxboard/maaxboard/) single board computer. This is the environment upon which seL4 binaries are executed. User interaction with this environment is performed via a serial console from the host environment.