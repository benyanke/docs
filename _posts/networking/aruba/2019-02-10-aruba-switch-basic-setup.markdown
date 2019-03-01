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

## Set Username and Password

To begin setup, run `enable` with password `enable` to use elevated privileges.

```
(ArubaS2500-48T) >enable
Password:******

(ArubaS2500-48T) #configure terminal 
Enter Configuration commands, one per line. End with CNTL/Z
```

Now, set the username and password with `mgmt-user admin root`
```
(ArubaS2500-48T) (config) #mgmt-user admin root
Password:
```

Type "exit[enter]" three times to exit configure mode, enable mode, and the main shell. This will return 
you to the login prompt.

Now, log in with your new account: "admin" -> your password you selected.

```
(ArubaS2500-48T) 
User: admin
Password: ***************
(ArubaS2500-48T) >
```

If you see the `(...) >` prompt, you are now logged in with your new login account.

Next, we need to set the 'enable' password.


Log in with command `enable` and password `enable, and run "enable secret".

```
(ArubaS2500-48T) >enable
Password:******
(ArubaS2500-48T) #configure terminal
Enter Configuration commands, one per line. End with CNTL/Z

(ArubaS2500-48T) (config) #enable secret
Password:
```

Finally, write the terminal config with `write memory` to store the running config
to the memory to be used on boot.

Your user and enable password are now configured.


## Initial Provisioning

Next, set up your basic options:

```
(ArubaS2500-48T) #configure terminal 
Enter Configuration commands, one per line. End with CNTL/Z
```

Once in configuration mode, enter the following commands (adjusting values as needed)
to provide the switch basic configuration.

```
hostname "switch.hostname.example.com"
ip name-server 1.2.3.4
clock timezone "CST" -6 0
write memory
```

## Back up config

Finally, you want to backup your configuration - otherwise if there are any issues, 
you'll need to rebuild the entire config.

Minimally, you can print the configuration to the console using `show running-config`.

Or, to back up the entire configuration, run `backup flash`. This will write the current
configuration to "flashbackup.tar.gz." When the write finishes, confirm the timestamp 
looks correct with command `dir`.

Once the file is there, you can use tftp to copy the config off the device:

```
copy flash: flashbackup.tar.gz tftp: <tftp-server-ip-address> flashbackup.tar.gz
```


## Links

A few of the following links may be helpful for you.

[Firmware downloads for updated os versions](https://h10145.www1.hpe.com/downloads/ProductsList.aspx)

[Documentation for devices](http://h17007.www1.hpe.com/us/en/networking/library/index.aspx)

