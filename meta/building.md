# Build Guide

## Table of content

- [Build Guide](#build-guide)
  - [Table of content](#table-of-content)
  - [Supported environments](#supported-environments)
  - [Building Seva](#building-Seva)
    - [1. Building Source code](#1-building-source-code)
    - [2. Setting up](#2-setting-up)
    - [3. Building the OS](#3-building-the-os)
    - [4. Running in a virtual machine](#4-running-in-a-virtual-machine)

## Supported environments

Building The Seva Kernel requires

- A [Linux Distribution](issues.md) of your choice (Arch/Debian/Red Hat)

### Unfortunately, macOS is not currently supported. However, it is in our roadmap.

To test and debug Seva, these are required.

- qemu
- gdb

```sh
# On Debian-based distributions
sudo apt install qemu-kvm git qemu grub-pc-bin xorriso gcc mtools make binutils nasm
```

```sh
# On Arch-based distributions
sudo pacman -S gcc nasm binutils make grub qemu libisoburn git mtools qemu
```

```sh
# On Red Hat-based distributions
sudo dnf install gcc nasm gcc-g++ make binutils mtools xorriso ImageMagick git patch qemu-kvm qemu
```

## Building Seva

### 1. Building source code

Clone our repository with all of its submodules.

```sh
$ git clone --recursive https://github.com/Uquinix/seva

$ cd seva
```

If you have already cloned this repo without `--recursive` do:

```sh
$ cd seva

$ git submodule init
```

### 2. Setting up

These dependencies must be installed to build the toolchain:

- bison
- build-essential
- libgmp3-dev
- flex
- texinfo
- libmpfr-dev
- libmpc-dev

To install these dependencies, you can run the commands corresponding to your distribution below:

```sh
# On Debian-based distributions
$ sudo apt install build-essential bison flex libgmp3-dev libmpc-dev libmpfr-dev texinfo
```

```sh
# On Arch-based distributions
$ sudo pacman -S base-devel bison flex mpc mpfr texinfo
```

```sh
# For Red Hat Red Hat-based distributions
$ sudo dnf install bison flex mpc-devel mpfr-devel gmp-devel texinfo patch
```

Then for building the toolchain run the `build.sh` script

```sh
## Build the tool chain
$ toolchain/build.sh

## Then wait for completion
```

This script does the following (without the requirement of root access):

- Download `gcc` and `binutils`
- Patch them using binutils.patch and gcc.patch
- Configures and builds them

### 3. Building the OS

To build the OS, run this command from the root of the project:

```sh
$ make all
```

This command should build the neccassary components and setup an ISO file which you can boot into via QEMU or Virtualbox.

> Compatibility with Virtual Box is flakey, we reccomend to use QEMU primarly for debugging and testing.

### 4. Running in a virtual machine

The make file also allows the user to automatically run the iso file in their desired virtualisation software:

```sh
$ make run CONFIG_VMACHINE=qemu # for QEMU
# or
$ make run CONFIG_VMACHINE=vbox # for Virtual Box
```