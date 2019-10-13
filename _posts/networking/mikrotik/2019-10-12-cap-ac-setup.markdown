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
 - ansible
draft: true
---

Power device


## Basic Startup
Connect the access point to the POE injector. Connect the `power+data` end to the AP. Connect the `data` end to your existing network. It will get a DHCP address,
and after you check your DHCP server to check it's address, log into the device using it's IP in the browser. For example: http://192.168.1.24.

## Initial Configuration

Even if you are using Ansible, you will need to set the basic configuration to allow Ansible to take over. Open the terminal in the web interface:

[![Annotation of how to find terminal in web interface](2019-10-12-cap-ac-setup/1.png)](2019-10-12-cap-ac-setup/1.png)

### Set a hostname

```bash
[admin@MikroTik] > /system identity set name=cap01
```

### Change password

```bash
[admin@cap01] > /user set [find name=admin] password=YOURNEWPASSWORD
```

### Enable SSH

```bash
[admin@cap01] > /ip service set ssh port=22
```

### Setup SSH Keys

To setup keys, first upload the key file in the web interface. On the left side, select 'Files'.

[img: 2.png]

[img: 3.png]

Then, use the browse button to select your key file. Your key file should be a text file with one line, with the
key formatted in the authorized_keys format. That is, looking like this:

```
ssh-rsa AAAAB3Nc5v[...]B5nP/KnVKJP1fXQp jdoe@example.com
```

[img: 4.png]

Once the file is uploaded (in our case, `by.pub`), drop back to the console to configure.

```bash
[admin@cap01] > /user ssh-keys import public-key-file=by.pub user=admin
```

You can now login to account `admin` with the corresponding private key matching `by.pub`.

Simply login with SSH like any other host:

```bash
$ ssh admin@192.168.1.24 -i [keyfile path]
```

### Apply OS Updates

Before continuing forward, it's important for the device to be updated. Note that the 
second command will, if an update is available, reboot the device.

```bash
[admin@cap01] > /system package update check-for-updates once
            channel: stable
  installed-version: 6.43.12
             status: finding out latest version...
[admin@cap01] > /system package update install
            channel: stable
  installed-version: 6.43.12
     latest-version: 6.45.6
             status: Downloaded, rebooting...

[admin@cap01] > Shared connection to cap01 closed.
```


### Backup Config

Finally, this step is optional, but it's reccomended that before you run a tool like ansible, save the configuration so that
you can easily rollback to a clean state for a reapplication of configuration management, without needing to redo the manual
setup steps.

```bash
[admin@cap01] > /system backup save dont-encrypt=yes name=clean-config-before-ansible
```

## Wireless Setup

The basic setup is now complete.


guides:

capsman
 - https://www.ispsupplies.com/blog/mikrotik-wireless-capsman-howto
 - 



