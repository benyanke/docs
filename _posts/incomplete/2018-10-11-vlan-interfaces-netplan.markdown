---
layout: post
title:  "VLan Trunking with Netplan"
slug:   netplan-vlan0trunk
date:   2018-10-10 15:47:28
categories: Networking
tags: 
 - networking
 - netplan
 - vlans
 - ubuntu-18-04
---

## VLAN Interfaces
TODO:


## Edit Config Files
Example untested config here

```
network:
    version: 2
    ethernets:
        ens3:
            addresses: 
                - 192.168.122.201/24
            gateway4: 192.168.122.1
            nameservers:
                addresses: [192.168.122.1]
        ens8: {}

    vlans:
        vlan.101:
            id: 101
            link: ens8
            addresses: [192.168.101.1/24]
        vlan.102:
            id: 102
            link: ens8
            addresses: [192.168.102.1/24]
```


## Apply Configuration with Netplan 

TODO: NETPLAN APPLY HERE
