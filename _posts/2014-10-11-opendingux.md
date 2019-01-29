---
layout: post
title: "OpenDingux Dev Environment for OS X (sort of)"
description: "A quick tutorial for setting up a dev environment for OpenDinux with Dingoo and OS X"
category: articles
tags: [Dingoo, OpenDingux]
---

### Introduction
A couple of years ago I picked up a Dingoo A320, it was awesome especially for commuting back and forth to Sky where I was working at the time. Anyway things change and inevitably it ended up in the back of a draw. Having moved house again, I found it, though there were a few scratches on the screen that were quickly taken care with good ol'Brasso. I decided that it was time for me to make something for it and learn SDL during the process. So this post is a quick tutorial to setting up GCC to compile on OS X. (I say OS X, I'm actually running the toolchain under a virtual machine, I'll explain as I go on)

This tutorial does assume that you've got OpenDingux installed and running on your Dingoo, the latest version can be found [here](http://www.treewalker.org/opendingux/)

### The Toolchain
There is a toolchain that is provided for OpenDingux that includes GCC, SDL and a number of other useful libraries that can be found at the following [link](http://www.treewalker.org/opendingux/). You'll notice that it's for Linux and it's 32-bit only. Now it is possible to build a cross-compiler for OS X that can be used to build directly, however I'm going to show you a different (and in my view easier method) using a virtual machine.

### Installing the Virtual Machine
To run the virtual machine that we'll install the toolchain under I'm using Virtual Box. You could of course use Parallels or VMware, but the following instructions will assume you're using Virtual Box. 

First things first, install Virtual Box available [here](https://www.virtualbox.org/). Now we need to install a 32-bit Linux OS, personally I'm using Xubuntu. Make sure you download the 32-bit version.

After you've downloaded Xubuntu, run Virtual Box and create a new virtual machine, call it Xubuntu32 and for type select Linux, and for version choose Ubuntu 32-bit. Click 'Continue' and select the amount of RAM to use, I've just set it to 512MB, just enough. Next create a virtual hard drive with whatever size you need. After you've created the virtual machine, run it and follow the 'First Start Wizard'. Choose the Xubuntu iso you downloaded earlier, then just follow the Xubuntu installation instructions.  

### Installing the Toolchain
Once Xubuntu has been installed and it's running, open Firefox, it should already be installed, then download the toolchain from the link previously mentioned in this post. After it's been downloaded copy it to '/opt/' and then add '/opt/opendingux-toolchain/usr/bin' to your PATH.

That's pretty much it for installing the toolchain, and if you want to use Xubuntu to develop the code you can just stop here. If you carry on reading I'll tell you how to run Xubuntu headless and just access it with SSH.

### Installing SSH server
We'll be running Xubuntu in headless mode and we'll access it via SSH. To do this you'll need to install an SSH server, so in Xubuntu open a terminal and type in 'sudo apt-get install openssh-server'. 

Now to access the guest OS (Xubuntu) from the Host (OS X) we need to set up the Networks properly in Virtual Box. First we need to create a 'Host-only Network' so that the host and the guest can see each other. So, in the Virtual Box Manager go to Preferences, click the Network icon and then click on the 'Host-only Networks' tab. On the right-hand side there'll be a little icon that looks like a network card with a plus symbol, click it. It'll create a a new network with the name of 'vboxnet0'.

Now we need to make Xubuntu use this network. Do this by shutting down Xubuntu then in the Virtual Box Manager make sure Xubuntu is highlighted in the left-hand side where all the virtual machines are listed and then click on Settings. Next, click on the Network icon at the top. There should be a number of adapter tabs, click the second one. Select the checkbox labeled 'Enable Network Adapter'. On the 'Attached to' dropdownbox select 'Host-only Adapter' and in the 'Name' dropdownbox below select 'vboxnet0'.

This is a good time to check that we can SSH into Xubuntu. Start the SSH server in Xubuntu by entering a terminal and typing 'sudo service ssh start'. We then need the IP address of the guest OS, in the terminal type in 'ifconfig' and copy the IP address, for me it was '192.168.56.101'. In OS X open Terminal and enter 'ssh user@192.168.56.101' where 'user' is the username you chose when you installed Xubuntu, then enter your password. If all goes well you've now successfully got access to Xubuntu.

### Running Xubuntu Headless
The one thing left to do is to run Xubuntu headless, this means that we don't have to have another window open. We'll have to install the Virtual Box Extension Packs, I suggest you follow the instructions on this [page](http://www.thomas-krenn.com/en/wiki/Headless_Mode_for_Virtual_Machines_of_VirtualBox#Installing_Extension_Packs). Just below the instructions for installation are instructions for running and connecting to the virtual machine (instead of 'ubuntu-server', use 'xubuntu32').

You've now got a Xubuntu virtual machine running headless with the OpenDingux toolchain installed!

For the next post I'll go through a simple hello world with SDL and uploading it onto the Dingoo
