---
layout: default
title: Configuring your development environment
nav_order: 2
---

# Configuring your development environment

The easiest way to work on the computer assignments is to use a virtual image that contains all the tools and dependencies you will need.

## Using the virtual-image

First, you will need to install [Docker](https://docs.docker.com/get-docker/) on your system.

Then, retrieve the course's docker image with
```
$ docker pull pablooliveira/compil
```

To run the image, if you are on a unix-like system run
```
$ docker run -it -v "$(pwd)":/compil pablooliveira:compil /bin/bash
```
If instead you are on a windows system run
```
$ docker run -it -v %cd%:/compil pablooliveira:compil /bin/bash
```

Once inside the image you should move to the `/compil` directory where your host's local directory has been mounted.


## Configuring the development environment from scratch

If you want to configure the development environment from scratch, we recommend that you use a Debian/Ubuntu like distribution. Ensure that you use a 64bit distribution. Please install the following dependencies. Replace `[version]` with the llvm version included in your distribution, the labs in this course have been tested with version going from 3.9 to 9.0.

```
$ sudo apt-get install build-essential flex bison libboost-program-options-dev llvm-[version]-dev clang-[version] llvm-[version]-tools libz-dev autotools-dev automake autoconf libtool gdb git wget
```
