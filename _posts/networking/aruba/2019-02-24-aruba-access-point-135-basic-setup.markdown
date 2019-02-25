---
layout: post
title:  "Aruba Access Point IAP-135 - Basic Setup and Configuration"
slug:   aruba-iap-135-basics
date:   2019-02-23 23:58:29
categories: networking
tags: 
 - networking
 - aruba
 - hpe
 - switch
 - basic-configuration
published: false
---

_This guide was written with the Aruba IAP-135 AP, though it should be mostly applicable to other 
similar models_

## Factory Reset

When obtaining a new piece of used network equipment, the best method first is to clear the 
configuration of the device, to start with a clean slate.

Connect to the serial console port on the switch. This will allow you to manage the access point.

Once the serial console is open, apply power to the access point. The boot screen will provide some 
basic system information, and an APBoot version number.

When you see the screen below counting down to the autoboot stop, press enter to stop autoboot.

```

Hit <Enter> to stop autoboot:  2
apboot>
```

Once the console is open to apboot, run the following commands: 1) `purge`, 2) `save`, 3) `boot`.

```
apboot> purge
preserving os_partition (0)
Erasing flash...Writing to flash...done
Erasing flash...Writing to flash...done
apboot> save
Saving Environment to Flash...
Erasing flash...Writing to flash...done
apboot> boot
```

These commands clear the running configuration, save the newly cleared configuration, and then 
boots into the newly empty config. After issuing "reset" the access point will reboot.

When the boot finishes, you can log in. The default username and password is `admin`/`admin`.

Once you log in, you'll see a prompt with the hostname being the device's mac address.

TODO: continue here

## Initial Configuration

By default, the AP will fetch an IP from the DHCP server on the LAN. Connect it and it will get an IP.


TODO: Roughly follow this link.
https://www.arubanetworks.com/techdocs/InstantWenger_Mobile/Advanced/Content/Instant%20User%20Guide%20-%20volumes/Initial_Setup.htm

## Links
A few of the following links may be helpful for you.

[Firmware downloads for updated os versions](http://support.arubanetworks.com/LifetimeWarrantySoftware/tabid/121/DMXModule/661/EntryId/21998/Default.aspx)
[Documentation for devices](http://h17007.www1.hpe.com/us/en/networking/library/index.aspx)
