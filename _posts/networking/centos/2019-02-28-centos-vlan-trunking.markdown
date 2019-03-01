---
layout: post
title:  "Setup Vlan Trunk on CentOS 7"
slug:   centos7-vlan-trunk
date:   2019-02-28 19:58:29
categories: Networking
tags: 
 - networking
 - vlan
 - trunk
 - centos-7
---

A vlan is an OSI layer 2 method to provide multiple logical networks to exist on a 
single physical network. Vlan trunking can be a useful tool for a server to exist on 
multiple networks without the additional overhead and expense of additional physical 
interfaces.

This guide describes how to simply setup subinterfaces - virtual interfaces which provide 
access to the VLan - in CentOS 7.

## Method 1 - ifcfg files

First - run `ip link` to find the name of the interface. In our case, we have enp0s25.

Next, take the name of the interface, and setup the subinterface file at the
following path: 

```
/etc/sysconfig/network-scripts/ifcfg-[interface].[vlan]
```

Often, there is a version of this file already in place without the vlan id you can use 
as a basis.

Update the file to look like this:

```
VLAN=yes
DEVICE=[interface].[vlan]
ONBOOT=yes
BOOTPROTO=static
IPADDR=192.168.10.1
NETMASK=255.255.255.0
```

Finally, restart networking:


```
$ sudo systemctl restart network
```

## Method 2 - ip cli

This same task can also be accomplished using the `ip` command. To add `vlan 43` to 
interface `eth0`, you will create the interface `eth0.43`.

```
$ sudo ip link add link eth0 name eth0.43 type vlan id 43
$ ip link set dev eth0.43 up
```

And to delete the vlan interface:

```
$ sudo ip link delete eth0.43
```
