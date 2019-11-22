---
layout: post
title:  "Basic VyOS Configuration"
slug:   basic-setup
date:   2019-11-22 00:02:04
categories: vyos
tags:
 - networking
 - vyos
 - router
 - firewall
---

VyOS is a linux-based CLI-only router distribution. It is designed to act as a router 
appliance, offering complex enterprise-grade routing and switching features to any user 
for free.

It is based on a modifed Debian base, so can run anywhere Debian would run. It is 
suitable for both physical installations - prefered for production border firewalls - 
as well as virtual installs - good for testing and VM-only networking.

This guide will document the steps required to take a clean VyOS install and set up the 
basic functions required to use it as a router at the edge of a test or residential 
network, including NAT, port forwarding, DHCP client function for the external IP, DHCP 
server functions for internal networking. With these functions, you can replace any 
standard consumer edge router with a far more powerful and secure device.


Further information can be found in the [VyOS user's guide](https://wiki.vyos.net/wiki/User_Guide)


## Install

For the initial install, boot from the bootable media, and login with username and 
password of "vyos."

From there, run command `install image` and follow the prompts.

```bash
vyos@vyos:~$ install image
```

When complete, reboot. 


## Configure Mode

As with other unix systems and networking equipment, there are multiple security contexts 
with greater or lesser privilege. 

To enter "edit" mode on vyos, which allows configuration to be edited, not just read, 
type: `configure`.

```
vyos@vyos:~$ configure 
[edit]
vyos@vyos#
```

When in `configure` mode, the `[edit]` tag will be displayed on each shell prompt,
as well as the `$` turning into a `#`, which is used to seperate a root shell from
a nonroot shell.

When you are complete with changes, type `commit`. This will take the configuration 
changes you have made and make them live.

## Workflow with Save command

IMPORTANT: when you finish your configuration session, it is critical to run the `save` command
from a configuration shell context.

This stores the current committed configuration to the startup config. Running `commit` writes the
current config to memory, while running `save` writes it to the startup.

The general workflow of a configuration session should look like:

```bash
vyos@vyos:~$ configure
vyos@vyos# [your configuration commands]
vyos@vyos# [your configuration commands]
vyos@vyos# commit
vyos@vyos# save
vyos@vyos# exit
```

Similarly, you can specify a paramater on where to save the file, if you don't want to store to the 
startup config. This can also be used for backups of the current running config. Try any one of 
these commands below.

```bash
vyos@vyos# save /local/file/path/config.conf
vyos@vyos# save scp://user@remote.host/path/to/file.config
vyos@vyos# save ftp://user@remote.host/path/to/file.config
vyos@vyos# save tftp://remote.host/router.config
```

[More documentation about the save command](https://wiki.vyos.net/wiki/Save_(command))


## Hostname

The first step in any system build is setting the hostname.

```bash
vyos@vyos# set system host-name [fw01]
vyos@vyos# set system domain-name [corp.example.com]
vyos@vyos# commit
```

## Set Timezone

Now, we'll set the system's timezone. After typing the command, press `[TAB]`
multiple times to see the options, and select yours by continuing to type.

```bash
vyos@vyos# set system time-zone [TAB]
```

## User Accounts

During installation, you set a password for the `vyos` user. You can 
add additional users using the following two commands. The user 
is being created with the following details:

 * username: jsmith
 * Full name: John Smith
 * Password: mypassword


```bash
vyos@vyos# set system login user jsmith full-name "John Smith"
vyos@vyos# set system login user jsmith authentication plaintext-password mypassword
```

Additionally, you can delete users (such as the default `vyos` user, which is 
recommended) by running the following command. Note that you can not delete 
the currently logged in user. Below, we'll delete the default user after
adding our own.

Note: you will need to log out and login as your new user `jsmith` before
you can delete the default user.

```bash
vyos@vyos# delete system login user vyos
```

## SSH Keys

Passwords are insecure for SSH login - the best practice is to use SSH public/
private key pairs.

To specify a key for a user, use command `loadkey [user] [keypath]`.

Possible protocols for keypath include:

 - `/path/to/file` (Path on local router filesystem)
 - `scp://<user>@<host>/<file>`
 - `http://<host>/<file>`
 - `scp://<user>@<host>/<file>`
 - `tftp://<host>/<file>`

For example, load a key from a public URL using the command below.

```
[edit]
vyos@vyos# loadkey jsmith https://example.com/users/jsmith/ssh/publickey
```

## Interface Setup

Most router configurations feature a singlular outside/wan
interface, and one or more inside interfaces for various
features.

Begin by enumeration the interfaces on the system.

```bash
vyos@vyos# show interfaces
```

The system then lists all the interfaces and their mac addresses.

Then, we will set up the interfaces we found. We'll set up one outside
interface with DHCP, and one inner interface with an IP, a
configuration suitable for a standard residential or small 
business network.

Setup the external DHCP WAN interface: 

```bash
vyos@vyos# set interfaces ethernet eth0 address dhcp
vyos@vyos# set interfaces ethernet eth0 description 'OUTSIDE'
```

Setup the static inside LAN interface:

```bash
vyos@vyos# set interfaces ethernet eth1 address '192.168.0.1/24'
vyos@vyos# set interfaces ethernet eth1 description 'INSIDE'
```

Finally, set the DNS server for the router.

```bash
vyos@vyos# set system name-server 9.9.9.9
vyos@vyos# set system name-server 149.112.112.112
```

## Enable SSH

Next, you can enable SSH. The command below starts SSH listening
on all the router's IPs.

```bash
vyos@vyos# set service ssh port 22
```

NOTE: this also exposes SSH access on the public OUTSIDE interface.
Use with caution. Adding firewall rules to limit to internal use
only is recommended.


## Internal NAT (src nat)

By default, VyOS only routes packets. Additional setup is required to allow 
multiple devices to share a public IP on the WAN connection (`src nat`).

Set the following configuration. Replace `eth0` with your OUTSIDE interface,
and the `source address` IP with your device's INSIDE IP.

```bash
vyos@vyos# set nat source rule 100 outbound-interface 'eth0'
vyos@vyos# set nat source rule 100 source address '192.168.0.0/24'
vyos@vyos# set nat source rule 100 translation address masquerade
```

## DHCP Server for Internal Clients

VyOS can act as a DHCP server if you wish.

Set the basic config for the DHCP clients:

```bash
vyos@vyos# set service dhcp-server shared-network-name LAN subnet 192.168.0.0/24 default-router '192.168.0.1'
vyos@vyos# set service dhcp-server shared-network-name LAN subnet 192.168.0.0/24 dns-server '192.168.0.1'
vyos@vyos# set service dhcp-server shared-network-name LAN subnet 192.168.0.0/24 domain-name 'internal-network'
vyos@vyos# set service dhcp-server shared-network-name LAN subnet 192.168.0.0/24 lease '86400'
```

Finally, set your IP range for DHCP to hand out IPs:

```bash
vyos@vyos# set service dhcp-server shared-network-name LAN subnet 192.168.0.0/24 range 0 start 192.168.0.9
vyos@vyos# set service dhcp-server shared-network-name LAN subnet 192.168.0.0/24 range 0 stop '192.168.0.254'
```

## Setup DNS forwarder

VyOS can be set up to be a DNS forwarder, including caching. This allows
clients to query the VyOS device for DNS, and it can pass on requests to
public DNS servers.

Replace the interface with your INSIDE interface.

```bash
vyos@vyos# set service dns forwarding system
vyos@vyos# set service dns forwarding listen-on 'eth4'
```

## Switch Networks

You're now configured! Switch your network interfaces and test to ensure it works as expected!
