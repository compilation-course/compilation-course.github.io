# About This Course

In this course you will learn how to build a compiler. A compiler takes a source language as input and produces an binary that can be directly executed by the computer's hardware. The course is divided into the following five topics:

1.  Introduction to Compilers
2.  Lexical and Syntax Analysis
3.  Front-End
4.  Intermediate Representation
5.  Code Generation

The course is taught in an inverted mode. Every week we will assign a set of videos that should be watched online. You will also be required to answer a multiple-choice test to ensure that you have understood the material in the videos.

During class hours, we will review together the material in the videos. You should prepare questions and feel free to ask them and discuss them with the group. We will also investigate some extra topics that are not covered in the videos.

During lab hours, you will work on computer assignments in which you will develop the different parts of your own compiler.

# Requirements

Students are expected to have good programming skills in C and a basic knowledge of C++. Students should also understand the general concepts of computer architectures and operating systems; in particular they should known how the CPU executes assembly programs and how the memory is organized.

# Configuring your development environment

The easiest way to work on the computer assignments is to use a virtual image that contains all the tools and dependencies you will need.

## Using the virtual-image

TODO: Include a link to the virtual-image and instructions for using it locally.

## Configuring the development environment from scratch

If you want to configure the development environment from scratch, we recommend that you use a Debian/Ubuntu like distribution. Ensure that you use a 64bit distribution. Please install the following dependencies. Replace `[version]` with the llvm version included in your distribution, the labs in this course have been tested with version going from 3.9 to 9.0.

```$ sudo apt-get install flex bison gdb git libboost-all-dev llvm-[version]-dev clang-[version] llvm-[version]-tools libz-dev autoconf```
