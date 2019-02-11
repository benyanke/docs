---
layout: post
title:  "Aruba Switch Basic Setup and Configuration"
slug:   aruba-switch-basics
date:   2019-02-10 23:58:29
categories: networking
tags: 
 - networking
 - aruba
 - hpe
 - switch
 - basic-configuration
---


TODO: REMOVE ME LATER:
https://www.arubanetworks.com/techdocs/ClearPass/Aruba_DeployGd_HTML/Content/HP%20Switch%20Integration/Switch_configuration.htm


_This guide was written with the Aruba S2500 switch, though it should be mostly applicable to other similar models_

## Factory Reset

When obtaining a new piece of used network equipment, the best method first is to clear the configuration of the device,
to start with a clean slate.

Connect to the serial console port on the switch. This will allow you to manage the switch. On the front LCD menu, 
select "Maintenance" then "Factory Reset". Accept the confirmations which follow.

From here, the connected serial console will begin to show you the steps of the factory reset. Wait until the system 
boots again. It will take roughly 3-5 minutes.


While it's booting, you will see the ArubaOS version number scroll by (example below). This may be useful for future 
debugging and searching, so you may want to write it down.

```
ArubaOS Version 7.4.1.4 (build 54199 / label #54199)
```


## Log In

After the factory reset complets, the serial console will print :

```
User:
```

Log in with the default username and password of `password` / `forgetme!` is used to complete the reset.

```
User: password
Password: *********
```

## Basic Setup

1) Change the username and password
2) TBD

## Initial Provisioning

To begin provisioning, run `enable` with password `enable` to use elevated privileges.

```
(ArubaS2500-48T) >enable
Password:******
(ArubaS2500-48T) #configure terminal 
Enter Configuration commands, one per line. End with CNTL/Z

(ArubaS2500-48T) (config) #quick-setup 
Would you like to use Quick-Setup to help with the basic configuration of the system (y|n)?y
```

Select "y" then "[enter]" to use the quick setup.

Walk through the steps of the setup, answering the questions as they arise.

Note that you will need two passwords: one for login, one for enable.


## Change Username and Password

Set the username and password to a non-default value:

```

```

## Links

Firmware downloads: https://h10145.www1.hpe.com/downloads/ProductsList.aspx
Documentation: http://h17007.www1.hpe.com/us/en/networking/library/index.aspx

