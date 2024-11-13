---
title: Lab 0 - Git crash course
layout: default
parent: Introduction
mathjax: true
nav_order: 3
---

# Lab 0 - Git crash course

## Setup

For all the compilation labs we will use [git](http://git-scm.com). A
git repository tracks a project. Each repository may be cloned on different
local or remote computers.

This document is under [CC
by-nc-sa](https://creativecommons.org/licenses/by-nc-sa/2.0/) licence, inspired
by the lecture _Gerez vos codes souces avec git_ from M. Nebra.

## Introduction to Git

Git is able to follow the changes inside a text file for each
line of code. It can record all the intermediate versions and
changes in your project. But Git is not only a backup tool,
it offers many other features such as:

* Git records who made each change to each file and why.

* If two people work on the same file simultaneously, Git is
  capable of merging their modifications and avoids loosing any contributions.

Git is a "distributed" version management software. Each of the developers
participating in the same project has its own _git_ repository. Developers
can exchange versions and commits (changes) in a peer-to-peer fashion.
Often a central _repository_ is created on a server. The central repository
serves as a meeting point and facilitates exchanges between developers.

It is in this last mode that we will work in this course.

## Creating an account on gitlab-chps.ens.uvsq.fr

To save your work between lab sessions we will
use the server at [gitlab-chps.ens.uvsq.fr](http://gitlab-chps.ens.uvsq.fr).

▶ An account with your university email `first.last@ens.uvsq.fr` has been created. The initial password is everything before the `@` in your email. For example, if your email is `first.last@ens.uvsq.fr`, you will use the password `first.last`.

▶ Immediatly after logging change your password to secure your account. Click on the drop-down menu on the top-right, then go to `Preferences > Password`.

▶ When you log on the server, you will see that a project (`coa/firstname-lastname`) has been created for you.

▶ (optional) To clone and push on your repository without typing your password each time, create and add an ssh key to your account. First look how to create an ssh-key on your system. Then click on the drop-down menu, then choose `Settings > SSH Keys` to upload your public key pair.

## Cloning the repository

▶ To start working on your git repository, you'll have to clone it. Log into
  [gitlab-chps.ens.uvsq.fr](http://gitlab-chps.ens.uvsq.fr) and copy the address of
  the repository. Select the protocol (HTTP or SSH) on the drop-down menu at
  the center of the interface (do not choose SSH unless you have added an SSH
  key on your profile). Then copy the address that is located nearby.

▶ Now go to the terminal and type the following commands:

```bash
    # WARNING: please replace pablo-oliveira by your own name

    # clone the repository 
    $ git clone http://gitlab-chps.ens.uvsq.fr/coa/pablo-oliveira.git 
    warning: You appear to have cloned an empty repository.
    Checking connectivity... done.

    # enter the repository 
    $ cd pablo-oliveira/

    # Now create a lab0 folder and enter it 
    $ mkdir lab0
    $ cd lab0
```

If you want to clone this repository on other machines; the procedure is the
same. 

In the rest of the lab, you will work inside the `lab0` directory that you
just created. 

## Adding and committing files 

First configure git with your personal information,

```bash
$ git config --global user.name "Pablo Oliveira"
$ git config --global user.email pablo.oliveira@uvsq.fr
```

Now write a `test.cc` program that displays "Hello World!"
on the standard output and a Makefile file to compile it with `clang++-3.9`.
Write your program in C++.

To commit the files first add them in git with

```bash
$ git add Makefile test.cc
$ git status
On branch main

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

	new file:   Makefile
	new file:   test.cc
```

The `git status` command allows you to see what's new from the
last save (or commit) in the repository. Here `git status` shows
that both files are ready to be committed.

The `git commit` command opens an editor that allows you to type a
a message explaining the changes.

For commit messages use the following convention,

* the first line must contain a short and informative message that
   summarize your changes

* the second line must be empty

* next you can describe in detail your changes

For example:
```
    lab0 (exo1): write a hello world program

    test.cc prints "Hello World!" to the standard
    output. Also added a makefile.
```

## Using the log

Change your program again. This time change the program so that it segfaults.
You can be creative ☺

Then type `git diff` to see your changes.
Then commit your changes with `git add test.cc` followed by `git commit`.

You can view the journal of changes with `git log`

```bash
$ git log
commit 66ea53ab301045fc5a990fac3672f842dc75b2e8
Author: Pablo Oliveira <pablo.oliveira@uvsq.fr>
Date:   Thu Sep 14 22:59:15 2023 +0200

    Harmless modification that makes the program segfault

commit b35c2b6551d4b3738170637634702ed4d82aa501
Author: Pablo Oliveira <pablo.oliveira@uvsq.fr>
Date:   Thu Sep 14 22:55:05 2023 +0200

    lab0 (exo1): write a hello world program

    test.cc prints "Hello World!" to the standard
    output. Also added a makefile.
```


Each commit is followed by a unique hexadecimal code that identifies it. You
can view changes between two versions with `git diff b35c2b6 66ea53a`, there
is no need to type all the numbers, the first are sufficient (except in case of
conflict). Try !

The different versions form a **branch** of versions.
In git the main branch is usually called `main`.

Git allows you to work on several branches at the same time, go back to
previous versions of code, and many more things but that goes beyond the scope
of this introduction. 

## Sharing code with others 

To send your changes to another repository, you will need to use two commands:

* `git pull` retrieves changes from another repository

* `git push` sends your changes to another repository 

Since you already clone your repository from gitlab-chps.ens.uvsq.fr; this server is already configure as the default push server.

To send your changes do,

```bash
$ git push origin main

Counting objects: 3, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 271 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To http://pablo-oliveira@gitlab-chps.ens.uvsq.fr/coa/pablo-oliveira.git
* [new branch]      main -> main
```
