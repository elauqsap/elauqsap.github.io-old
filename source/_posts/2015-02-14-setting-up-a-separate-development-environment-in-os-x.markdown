---
layout: post
title: "Setting Up A Separate Development Environment In OS X"
date: 2015-02-14 01:30:39 -0600
comments: false
categories: [Development, VirtualBox, OS X, Tips & Tricks]
---

I like to keep my development environment separate from my host environment on my personal laptop. A simple solution for keeping them apart is to set up a headless virtual machine with a samba share. I like this method the most because you can set up a lightweight server that can easily load your code base into your favorite IDE or text editor. I will be using VirtualBox in this example but you can use Fusion as well, it just requires a different command.

#### Prerequisites
 1. Download VirtualBox from [here](https://www.virtualbox.org/wiki/Downloads)
 2. Download Ubuntu Server (14.04 LTS in this example) from [here](http://www.ubuntu.com/download/server)

First, I am going to setup my dotfiles to include a few aliases for interacting with the virtual machine. In your shell configuration file (zsh here) add the equivalent aliases so you can quickly start, mount, and remote into the server. Remember to replace $USER with whatever uid name you will be using on the guest server. Also $GUEST needs to be changed to whatever you call the virtual machine.

``` bash .zshrc
# Ubuntu devbox aliases
alias devbox='VBoxManage startvm "$GUEST" --type headless'
alias sshdev='ssh devbox'
alias mountdev='mount_smbfs //devbox/$USER ~/devbox'
alias umountdev='umount ~/devbox'
```

After you have VirtualBox installed go ahead and create a new Virtual Machine. I carved off 2048 MB in RAM and 20 GB in storage for the development box. You should now have a basic virtual machine, but before you can boot it up we need to take care of a few things. First, create a Host-Only network that we will later attach as the second adapter (VirtualBox -> Preferences -> Network -> Host-only Networks). If "vboxnet0" is not already there, click the add button and then edit. Configuration listed below.

	# Adapter
 	IPv4 Address: 10.10.1.1
  	IPv4 Network Mask: 255.255.255.248
  	IPv6 Address:
  	IPv6 Network Mask Length: 0
---
	# DHCP Server
	Uncheck Enable Server

Close out of VirtualBox's settings and highlight the virtual machine we created. We need to attach the ISO we downloaded earlier to the IDE Controller (Settings -> Storage). Once you have added the ISO to empty IDE slot we are going to switch to the Networking tab to setup our interfaces. The first adapter should be set to NAT. If it is not selected go to the drop down for "Attached to:" and add it. Flip into the second adapter and set it to be Host-only. We are done configuring the virtual machine, go ahead and boot the server and follow the basic install instructions. With Ubuntu, you can choose to install Samba and Open SSH Server during the installation process so that kills two birds with one stone for us.

Once the installation is complete and the server reboots, we need to modify two files on the system. The first file I have listed below is your interfaces file. The first adapter "eth0" will be your NAT so the host can communicate outbound to receive packages. Leave this interface configured for DHCP. The second adapter "eth1" is on our Host-Only network and needs a static address. This network allows the host system to connect to any of the public facing services on the headless virtual machine (SSH, Web Server, MongoDB, etc). 


``` bash /etc/network/interfaces
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto eth0
iface eth0 inet dhcp

# Host-only interface
auto eth1
iface eth1 inet static
	address 	10.10.1.2
	netmask		255.255.255.248
	network		10.10.1.0
	broadcast	10.10.1.7
```
The second file is our samba share configuration that will let you mount the home directory of the guest user. This is where you will store your code base and anything else related to the application. Once you mount it on your host system you can open the project up in your favorite IDE or text editor and begin working. Just remember to save often and unmount the share before shutting down the virtual machine.

``` bash /etc/samba/smb.conf
[dev]
	comment = $USER
	path = /home/$USER
	guest ok = no
	browseable = no
	writeable = yes
	create mask = 0600
	directory mask = 0700
```

On your OS X system, edit your hosts file to include an entry for your virtual machine. This way you can address it by a friendly host name rather than an IP address.

``` bash /etc/hosts
127.0.0.1   localhost
255.255.255.255 broadcasthost
::1             localhost
fe80::1%lo0 localhost

# VirtualBox Hosts
10.10.1.2       devbox
```

That's it! Now you can easily start, mount, and control your virtual machine all from your host system. Just enter "devbox" or whatever alias you changed it to and the virtual machine will boot up in headless mode.