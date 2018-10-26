---
layout: post
title:  "Setup Vlan Trunk on CentOS 7"
slug:   centos7-vlan-trunk
date:   2018-10-10 12:58:29
categories: Networking
tags: 
 - networking
 - vlan
 - trunk
 - centos-7
---


VLan trunking can be a useful tool for a server to exist on multiple networks without
the additional overhead and expense of additional physical interfaces.

This guide describes how to simply setup subinterfaces - virtual interfaces which provide 
access to the VLan - in CentOS 7.

First - run `ip link` to find the name of the interface. In our case, we have enp0s25.

Next, take the name of the interface, and setup the subinterface file at the
following path: 

```
/etc/sysconfig/network-scripts/ifcfg-[interface].[vlan]
```

Often, there is a version of this file already in place without the vlan id you can use as a basis.

Update the file to look like this:

```
VLAN=yes
DEVICE=[interface].[vlan]
ONBOOT=yes
BOOTPROTO=static
IPADDR=192.168.10.1
NETMASK=255.255.255.0
```

The above example documents usage of a static IP on the vlan.
