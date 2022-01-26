# Introduction

## Background

The [seL4 Microkernel](https://sel4.systems) is a technology of increasing interest and has potential to deliver significant benefit across a range of applications that require strong domain separation and software enforced trusted execution environments.

seL4 is formally verified software and therefore offers significant benefits over other software based implementations in terms of the assurance and trust that can be gained. Given it is open source COTS technology the potential for reduced cost for high assurance products and reduced time to market are significant.

Developing on top of the seL4 microkernel is currently complex and time consuming, and the expertise is currently limited to a small number of individuals and organisations. This developer kit is targeted at reducing the barrier to entry for using (i.e. developing on top of) the seL4 technology and increasing the number of companies that can cost-effectively adopt seL4 in their products.

## Overview

This document presents an entry-level developer kit to help users gain familiarity with using the seL4 microkernel and ultimately to encourage more use of seL4 within projects and products. It does this by:

- selecting a readily available, low-cost single board computer (the [Avnet MaaXBoard](https://www.avnet.com/wps/portal/us/products/avnet-boards/avnet-board-families/maaxboard/maaxboard/)) as the target platform;

- detailing the minimum hardware and software requirements necessary to follow the guide;

- supplying a preconfigured build environment with all tools required to generate seL4 executable images;

- demonstrating how an seL4 executable image can be loaded and executed on the chosen target platform; and

- documenting the specific steps needed along with more general guidance.

This developer kit does not cover or replace any of the extensive documentation and tutorials of the [seL4 website](https://sel4.systems).

## Audience

The intended audience for this developer kit is an application developer who:

- has experience programming within a Linux, macOS or Windows environment;

- is already familiar with seL4's concepts and benefits; and

- is looking for more practical guidance to get started with seL4.

Whilst the developer kit documentation aims to cover all the salient points and commands, it assumes a reasonable level of developer knowledge and does not necessarily cover every detail of 'standard' Linux/macOS/Windows environments (for example, it covers seL4-related installation, but does not cover, say, installation of Docker).

## Development Environments

Throughout this developer kit activities will be performed across three different environments:

1. **Host Machine**: The engineer's host machine. This can be either macOS, Linux or Windows based.

2. **Build environment**: The environment within which all binaries for the target environment are built. This is a Linux based development environment running within a [Docker](https://www.docker.com) container on the host machine. The use of a Docker container to provide all of the tooling required to build seL4 applications for the target platform greatly simplifies the requirements of the host machine.

3. **Target Platform**: The [Avnet MaaXBoard](https://www.avnet.com/wps/portal/us/products/avnet-boards/avnet-board-families/maaxboard/maaxboard), a single board computer based around the [NXP i.MX 8M SoC (system on chip)](https://www.nxp.com/products/processors-and-microcontrollers/arm-processors/i-mx-applications-processors/i-mx-8-processors/i-mx-8m-family-armcortex-a53-cortex-m4-audio-voice-video:i.MX8M). This is the environment upon which seL4 binaries are executed. User interaction with the target platform is performed via a serial console from the host machine.

The required configuration of the host and target environments are detailed in the [Host Machine Setup](host_machine_setup.md) and [Target Platform Setup](target_platform_setup.md) sections. A preconfigured build environment is supplied as part of this developer kit as detailed in the [Build Environment Setup](build_environment_setup.md) section.

## Structure

The document is structured into the following sections providing a step-by-step guide:

- **Basic Requirements**: Defines the hardware requirements and the requirements on the user's host environment that need to be met to follow this guide. Completion of this section is expected to require the purchase of some equipment, e.g. the Avnet MaaXBoard.
  
- **Development Environment Setup**: Guides the user through setup of all development environments such as unboxing and setup of the Avnet MaaXBoard as well us installation and configuration of all required tools.

- **First Boot**: Provides an introduction and overview of the bootloader, guidance on how to prepare an SD card with the bootloader, through to the first power-up of the Avnet MaaXBoard and interaction with the bootloader running on the board.

- **seL4 Application Development**: Guides the user through the compilation of an seL4 binary followed by execution of the binary on the Avnet MaaXBoard.

- **Appendices**: Whilst the main sections of this document seek to simplify processes, e.g. through the use of pre-built assets and environments, it is accepted that this may result in a lack of flexibility. The appendices provide deeper technical details to enable greater flexibility. For example where a pre-built asset or environment has been used the appendices detail how they were built and provide guidance of how their configuration could be changed.
