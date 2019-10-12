---
layout: post
title:  "Mikrotik cAP ac Setup Guide"
slug:   cap-setup-guide
date:   2019-10-12 08:47:21
categories: Networking
tags: 
 - mikrotik
 - cap
 - cap-ac
 - wifi
 - access-point
draft: true
---

Power device

Connect the access point to the POE injector. Connect the `power+data` end to the AP, and the `data` end directly to your computer. The access point has a static IP:
192.168.88.1. Give the computer a static IP in the 192.168.88.0/24 subnet, and connect to the access point in the browser `http://192.168.88.1`.

TODO : document the reset button hold instructions

computer
will receive a DHCP address from the `cAP ac`.


======
old
=====

On the ap:

Connect to main network

on another device, run:

/tool mac-telnet [new device mac]

default login is admin, no password.

This will grant telnet to device.

First change hostname for clarity:

/system identity set name=HOSTNAME

Then switch to CAPs mode.






guides:

capsman
 - https://www.ispsupplies.com/blog/mikrotik-wireless-capsman-howto
 - 



