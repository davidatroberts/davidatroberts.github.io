---
layout: post
title: "Alfred and the Ghost"
description: "Controlling an LED lamp with Alfred"
category: articles
tags: [Ghost, Raspberry Pi, Alfred, Lua]
published: false
---

### Introduction

If anyone's read through the previous posts on this blog you may have seen my post where I described how I used an Arduino to control a Pacman ghost lamp. Well it might take me a while but I always come back to my projects, therefore this post is a bit of a follow-up and a rewrite of the old one. Basically I now have a Raspberry Pi A+ using the Linux Infrared Remote Control (LIRC) package with a couple of Lua libraries wrapped around it that expose some REST endpoints that I then use a workflow to control via Alfred.  

### LIRC
So, what is LIRC? Well it's a package that allows linux to decode and send infra-red signals, basically we record a signal from a remote control, this creates a config file, then using that config file we can send out the IR signals and control whatever we want.
First things first lets deal with how to receive the IR commands from the remote control that comes with the Ghost lamp and then record it. There are a few things we'll need an IR sensor, and about a 200ohm resistor. If I was a better blogger I'd give you all the details about circuit diagrams etc and how to use irrecord that's within the LIRC package, but instead I'm going to be super lazy and just give you the link to the article I followed, with full credit to [ModMyPi](https://www.modmypi.com/blog/raspberry-pis-remotes-ir-receivers)