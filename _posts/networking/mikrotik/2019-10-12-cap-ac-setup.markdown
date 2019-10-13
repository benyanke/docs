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
# draft: true
---

Power device


## Basic Startup
Connect the access point to the POE injector. Connect the `power+data` end to the AP. Connect the `data` end to your existing network. It will get a DHCP address,
and after you check your DHCP server to check it's address, log into the device using it's IP in the browser. For example: http://192.168.1.24.

## Initial Configuration

Even if you are using Ansible, you will need to set the basic configuration to allow Ansible to take over. Open the terminal in the web interface:

[![Annotation of how to find terminal in web interface](2019-10-12-cap-ac-setup/1.png)](2019-10-12-cap-ac-setup/1.png)

First, set a hostname:

```bash
[admin@MikroTik] > /system identity set name=cap01
```

Then setup your SSH keys:
```bash

```


TODO : document the reset button hold instructions.

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



