---
layout: default
title: Configuring your development environment
nav_order: 2
---

# Configuring your development environment

The easiest way to work on the computer assignments is to use a virtual image that contains all the tools and dependencies you will need.

## Using the virtual-image

First, you will need to install [Docker](https://docs.docker.com/get-docker/) or [Podman](https://podman.io/) on your system.

Then, retrieve the course's docker image with
```
$ docker pull pablooliveira/compil
```
*(If using Podman, replace `docker` with `podman`)*.

To run the image, if you are on a unix-like system run
```
$ docker run -it -v "$(pwd)":/compil pablooliveira/compil /bin/bash
```
If instead you are on a windows system run
```
$ docker run -it -v %cd%:/compil pablooliveira/compil /bin/bash
```

Once inside the image you should move to the `/compil` directory where your host's local directory has been mounted.


## Configuring the development environment from scratch

If you want to configure the development environment from scratch, we recommend that you use a Debian/Ubuntu-like distribution (such as Ubuntu 24.04 LTS). Ensure that you use a 64bit distribution. The labs in this course require LLVM 18. Please install the following dependencies:

```
$ sudo apt-get install build-essential flex bison libboost-program-options-dev llvm-18-dev clang-18 llvm-18-tools clang-format-18 python3 python3-yaml libz-dev autotools-dev automake autoconf libtool gdb git wget
```
