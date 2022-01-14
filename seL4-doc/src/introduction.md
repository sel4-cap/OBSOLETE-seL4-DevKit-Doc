# Introduction

This project presents an entry-level Developer Kit (DevKit) to help users gain familiarity with using the seL4 microkernel and ultimately to encourage more use of seL4 within projects and products. It does this by:
- selecting a readily available, low-cost single board computer;
- providing software to run on it to demonstrate seL4 boot-up, basic drivers (networking, USB, and file systems), and a security domain demonstration application;
- documenting the specific steps needed along with more general guidance.

# Audience

The intended audience for this seL4 DevKit is an application developer who:
- has experience programming within a Linux/macOS environment;
- is already familiar with seL4's concepts and benefits;
- is looking for more practical guidance to get started with seL4; and
- wishes more quickly to reach the stage of being able to develop applications on top of the basic functionality provided by the seL4 DevKit.

Whilst the DevKit documentation aims to cover all the salient points and commands, it assumes a reasonable level of developer knowledge and does not necessarily cover every detail of 'standard' Linux/macOS environments (for example, it covers seL4-related installation, but does not cover, say, installation of macOS Xcode command line tools).

This DevKit does not cover or replace any of the extensive documentation and tutorials of the [seL4 website](https://sel4.systems).