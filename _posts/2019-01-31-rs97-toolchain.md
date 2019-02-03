---
layout: post
title: "Using docker to cross-compile for the RS97"
description: "How to set up and use an RS97 toolchain on docker"
category: articles
tags: [RS97, UselessRS97, Docker, cross-compile]
---

### Introduction
I've recently picked up an RS-97 handheld, it's a neat little device that uses the old Revo K101 clone GBA shell. In the box it comes preinstalled with some emulators (and an absolute tonne of roms, being from China where copyright doesn't seem to matter). 

Dingux was ported to the device and I've been using the UselessRS97 firmware. Although gameblabla, the developer, has recently stopped updating the firmware due to differences in hardware revisions - I've found it to be an excellent piece of software.

The toolchain for the device found at <https://github.com/gameblabla/buildroot-rs97> requires debian, however me using Mac OS and not really being keen to dual boot linux on my macbook air I decided to use Docker.

Using docker, a light-weight version of debian can be run that has only the packages required to get the toolchain working. It also means I can develop on OSX without switching over to a virtual machine, the same method can also be used to cross develop on Windows and Linux as well.

### Docker
We'll be using Docker to host the debian image that will contain the toolchain

* Install Docker
    * For Windows use <https://docs.docker.com/docker-for-windows/install/>
    * For Mac OS use <https://docs.docker.com/docker-for-mac/install/>
    * For Linux <https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-using-the-repository>

I've already created the docker image, so you can just download the image and get started by:

~~~ bash
docker pull biomood/rs97:latest
~~~

Next, you'll need to run the image using the command below. This will start running debian and put you straight into the shell.

~~~ bash
docker run --rm -ti -v [path]:/code biomood/rs97:latest
~~~

where
: _run_ - tell Docker to run the image
: _--rm_ - removes the container's file system when the image is finished
: _-ti_ - allocates a virtual terminal session and keeps STDIN open
: _-v_ - mounts a volume, the _[path]_ option should be some location on your host machine, this will then get mapped to the _/code_ directory in the docker image

If all goes well debian should've started and you'll see something like this in your terminal:

~~~ bash
root@8ac34c1be2d5:/#
~~~

### The Toolchain
The toolchain is installed in _/opt/rs97-toolchain_ and
[here](https://gist.github.com/davidatroberts/8ade17be770e08172ca49f6f5ba474ec) you can find a basic makefile that I've been using to compile my projects. 
This assumes that the directory structure is:

root
: src
: lib
: bin
: obj

The output file will be _output.dge_. You can then copy this over onto the SD card in the RS97 and all being well your program will run.