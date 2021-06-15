---
layout: default
title: Configuring your development environment
nav_order: 2
---

# Configuring your development environment

The easiest way to work on the computer assignments is to use a virtual image that contains all the tools and dependencies you will need.

## Using the virtual-image

TODO: Include a link to the virtual-image and instructions for using it locally.

## Configuring the development environment from scratch

If you want to configure the development environment from scratch, we recommend that you use a Debian/Ubuntu like distribution. Ensure that you use a 64bit distribution. Please install the following dependencies. Replace `[version]` with the llvm version included in your distribution, the labs in this course have been tested with version going from 3.9 to 9.0.

```$ sudo apt-get install flex bison gdb git libboost-all-dev llvm-[version]-dev clang-[version] llvm-[version]-tools libz-dev autoconf```
