# Worked Example - Adding support for the ODroidC2 Platform

As part of the review of the [New Platform](uboot_library_add_platform.md)
and [New Driver](uboot_library_add_driver.md) sections of this guide, we ported the
UBoot Driver Library to build on the ODroidC2 platform.

This appendix gives details of how this was achieved, following the structure
of those earlier sections, but adding details of exact changes to each file.

Our goal is to get the UBoot [test applications](uboot_driver_usage.md)) running
on the ODroidC2.

## Root Directory, Platform Name, and basic Platform Details

To build the test application, we need to create a new root directory
in which to create a "repo" structure and perform the build.

The root directory name is "c2new". From here on, all directory names given in this section
are relative to that new root directory.

We also know that the ODroidC2 is already supported by seL4. Its device tree
appears in kernel/tools/dts/odroidc2.dts

The seL4 "platform name" is "odroidc2"

The system-on-chip device at the heart of the C2 is Amlogic S905, also known as
a "meson" SoC.

## Repository setup and forks

To work on a new platform build, we need to modify some of the existing repositories.
To do that, we need to "fork" those repositores and do the work in our own local
branches until it's all working. Finally, we would need to open a "Pull Request"
to merge our changes back into the "upstream" repositories.

Additionally, we need to tell the "repo" tool that we want to build from our
own forked repositories (so that the "repo sync" step gets the sources from
our forks, not from the upstream repositories.)

You'll need to your own GitHub account to do this. From here on, we'll be using the
GitHub account name "rod-chapman" for our local forks.

1. Using the GitHub web GUI, create your own forks of the "camkes-manifest", "camkes",
and "projects_libs" repositories.

2. Clone the "camkes-manifest" fork into your local machine, and create
a new branch called "addc2" to make our changes:

```bash
git clone https://github.com/rod-chapman/camkes-manifest.git
cd camkes-manifest
git checkout -b addc2
git push --set-upstream origin addc2
cd ..
```

3. Similarly, clone the "camkes" and "projects_libs" forks, and add a new branch
to each with the same name:

```bash
git clone https://github.com/rod-chapman/camkes.git
cd camkes
git checkout -b addc2
git push --set-upstream origin addc2
cd ..
```

```bash
git clone https://github.com/rod-chapman/projects_libs.git
cd projects_libs
git checkout -b addc2
git push --set-upstream origin addc2
cd ..
```

4. Edit the default.xml file in the "camkes-manifest" repository to add a new remote (in our
case called "rod") pointing at our own GitHub account.

For example, the line to add is

```text
<remote name="rod" fetch="https://github.com/rod-chapman"/>
```

The exact diff can be seen at this [GitHub commit](https://github.com/rod-chapman/camkes-manifest/commit/eae32a5d03064b43bda5c124a794ce2471cf5c5f)

5. Similarly, edit the default.xml file to specify that the "camkes" and "projects_libs" repositories
should come from the "addc2" branches of our own forked repositories. Find the "project" line for each
repository and modify its entry to specify our own remote ("rod") and branch ("addc2").

For our example, see this [GitHub commit](https://github.com/rod-chapman/camkes-manifest/commit/88d83b8357db801d54d4798b58d95e297e2cc112)

6. Commit and push that change:

```bash
cd camkes-manifest
git add default.xml
git commit -m "Add remote and forked repositories for adding ODroidC2 platform."
git push
cd ..
```

## Test Application Build for the MaaxBoard

Having made no other changes at this point, we should be able to build and run
the UBoot Driver Example program for the MaaxBoard from those newly forked
repositories.

The earlier instructions in the [New Platform](uboot_library_add_platform.md)
section should be followed with one significant change: the first "repo init" command
specifies our fork and branch of the "camkes-manifest" repository.

For example:

```bash
repo init -u https://github.com/rod-chapman/camkes-manifest.git -b addc2
```

Assuming that works, then we can start to make modification to support the ODroidC2.
