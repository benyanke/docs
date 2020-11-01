---
layout: post
title:  "Mikrotik cAP ac Setup Guide"
slug:   cap-setup-guide
date:   2019-10-14 08:47:21
categories: Networking
tags: 
 - mikrotik
 - cap
 - cap-ac
 - wifi
 - access-point
 - ansible
draft: true
---

Note: this is no longer needed, it's covered by the capsman guide

## Basic Startup
Connect the access point to the POE injector. Connect the `power+data` end to the AP. Connect the `data` end to your existing network. It will get a DHCP address,
and after you check your DHCP server to check it's address, log into the device using it's IP in the browser. For example: http://192.168.1.24.


## Initial Configuration

[add link to routeros initial config setup guide]

## Wireless Setup

The basic setup is now complete.


guides:

capsman
 - https://www.ispsupplies.com/blog/mikrotik-wireless-capsman-howto
 - https://help.mikrotik.com/docs/display/UM/cAP+ac#heading-Resetbutton




On CAP, to setup, go to "wireless" -> "cap" -> put in IP.

If it's already there and not working, disable the cap mode, save, then reenable, and 
save. It'll show up.

