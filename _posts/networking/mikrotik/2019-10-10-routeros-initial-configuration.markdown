---
layout: post
title:  "Mikrotik RouterOS Initial Configuration"
slug:   routeros-initial-configuration
date:   2019-10-10 08:47:21
categories: Networking/mikrotik
tags: 
 - mikrotik
 - routeros
 - ansible
 - configuration-management
---

## Basic Startup
One of the modern best practices when administering networking equipment and servers is using configuration management to define the system state and automate
it's application to the devices. But before you can use configuration management, you must do a basic level of configuration such as setting it's IP, enabling 
SSH, to allow the configuration management tool to access the device.

However, whether or not you are using configuration management or not, this guide can be useful to provide the initial configuration for a RouterOS device.

## Initial Configuration

To begin setup, login to the device via it's web interface. By default, the username is `admin` and password is blank. Open the terminal in the web 
interface using the button on the top right corner:

![Annotation of how to find terminal in web interface](/assets/2019/10/routeros-initial-configuration-001.png)


### Set a hostname

In this example 'cap01' is the hostname we're selecting.

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


![Key Upload](/assets/2019/10/routeros-initial-configuration-002.png)

![Key Upload](/assets/2019/10/routeros-initial-configuration-003.png)

Then, use the browse button to select your key file. Your key file should be a text file with one line, with the
key formatted in the authorized_keys format. That is, looking like this:

```
ssh-rsa AAAAB3Nc5v[...]B5nP/KnVKJP1fXQp jdoe@example.com
```

![Key Upload](/assets/2019/10/routeros-initial-configuration-004.png)

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

## Complete

Your RouterOS device is now ready to configure with it's role-specific configuration, whether you are using the device as a router,
access point, switch, or something else.
