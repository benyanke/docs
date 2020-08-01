---
layout: post
title:  "Make Snaps Work With A Nonstandard Path"
slug:   different-homedir-path
date:   2020-04-30 11:22:21
categories: snap
tags: 
 - snap
 - snapcraft
 - apparmor
 - ubuntu-18-04
 - ubuntu
---

## Problem

Snapcraft uses apparmor to contain software running within the snap, and to ensure it doesn't write to the 
system outside of expected areas. However, this leads to issues when files are in nonstandard locations, such 
as your home directory residing somewhere like `/home/nethome/[user]` for NFS mounted home directories.

## Solution

The solution is to make apparmor aware of new home directory locations.

This can be done in one of two ways.

Interactively, with this command:

```bash
$ sudo dpkg-reconfigure apparmor
```

Or, alternatively, you can run this one-liner to provide the correct configuration, assuming your
alternate homedir is in `/home/nethome/USER`:

```bash
$ echo "@{HOMEDIRS}+=/home/nethome/" | sudo tee /etc/apparmor.d/tunables/home.d/alt-homedir
```


This should now work, without needing to restart any services, allowing snaps to correctly write
to your home directory not at /home.
