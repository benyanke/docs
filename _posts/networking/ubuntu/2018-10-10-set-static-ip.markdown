---
layout: post
title:  "Set Static IP with Netplan"
slug:   netplan-set-static-ip
date:   2018-10-10 12:58:29
categories: Networking
tags: 
 - networking
 - netplan
 - static-ip
 - ubuntu-18-04
---

## What is Netplan?
Ubuntu 18.04 LTS ships with [Netplan](https://netplan.io) as a network management tool 
by default. Netplan allows simpler and declarative configuration of networks and 
interfaces with YML files, which eases both human and machine configuration.. Netplan 
talks to `systemd-networkd` and `Network Manager` to apply the desired configuration.

## Configuration
Netplan configuration files are stored in `/etc/netplan/*.yaml`. The default one created 
by the Ubuntu installer is stored at `/etc/netplan/50-cloud-init.yaml`. Typically, 
editing this file is sufficent, though you can create others.

*TODO: Confirm it's safe to edit 50-cloud-init.yaml, this is not yet clear to me.*

By default, the configuration file looks like this (given two interfaces called `enp1s0` 
and `enp2s0`): 
```
 network:
    ethernets:
        enp1s0:
            addresses: []
            dhcp4: true
        enp2s0:
            addresses: []
            dhcp4: true
            optional: true
    version: 2
```

To set a static IP, set the options as found below, ensuring you set `dhcp4: false`, and 
then specify `addresses`, `gateway4`, and `nameservers : addresses`. There are also 
additional options available under the `nameservers` key.
```
network:
    ethernets:
        enp1s0:
            dhcp4: false
            addresses: [192.168.1.2/24]
            gateway4: 192.168.1.1
            nameservers:
              addresses: [8.8.8.8,8.8.4.4]
        enp2s0:
            addresses: []
            dhcp4: true
            optional: true
    version: 2
```

Finally, to apply, run:
```
sudo netplan apply
```

Or, if you encounter an error and need to debug:
```
sudo netplan --debug generate
```
