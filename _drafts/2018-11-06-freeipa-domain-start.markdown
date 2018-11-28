---
layout: post
title:  "Setting up a Linux Directory Server and Domain with FreeIPA"
slug:   start-with-freeipa
date:   2018-11-02 23:58:29
categories: freeipa
tags: 
 - domain
 - ldap
 - authentication
 - free-ipa
 - ubuntu-18-04
---


based on https://www.freeipa.org/page/Quick_Start_Guide
https://computingforgeeks.com/how-to-configure-freeipa-client-on-ubuntu-18-04-ubuntu-16-04-centos-7/


FreeIPA is an alternative to the **Windows Active Directory** system. It provides a 
number of advantages, including centralization of:

 * System authentication
 * User directory
 * Authentication auditing
 * DNS
 * NTP
 * LDAP
 * certificate management

In short - it provides a central hub for the users, servers, and applications on your 
network. Like **Active Directory**, systems are enrolled, user accounts are created, and 
then users can log into multiple systems using their single domain credentials.

## Naming Your Domain

TODO: document the following:
  * use a subdomain of a domain you own
  * don't use a made up domain
  * don't use your root (split dns)

Let's call our domain `corp.example.com`.

## Infrastructure

Since a directory server like FreeIPA is a core of your network authentication, any 
non-test deployment should have at least 2 domain controllers, for redundant services.

Similarly, in a multi-site deployment, any site which could be seperated from the 
primary site should have it's own controller, so that in the event of a network 
partition such as an internet outage or VPN issue, users at this site can still log in.

In our deployment, we will create two domain controllers:

 * dc1.corp.example.com
 * dc2.corp.example.com

## Setting up your Single-Site Domain

For this first guide, we will set up a single-site, dual-server setup.

First, create a servers, which we will create as the FQDN (fully qualified domain names) 
of `dc1.corp.example.com`. Install Ubuntu 18.04 LTS on each host.

**DNS**

FreeIPA, like many directory servers, is picky about DNS - if not set up right, the 
entire install will fail, or not work properly.

Put your FQDN, followed by the short system hostname, in `/etc/hosts` file, as seen below.

```
10.0.0.1       dc1.corp.example.com dc1
```


**FreeIPA**

Start by enabling the universe repo, and then installing the freeipa server packages.

```
$ sudo apt-add-repository universe
$ sudo apt update && sudo apt install -y freeipa-server
```

During the install, you will be prompted for kerberos realm settings.

The kerberos Realm is the same as the domain you're setting up. Specify your domain, 
in all caps, as the realm, and specify your DC as the realm server.

In our example, the realm is `CORP.EXAMPLE.COM`, and the server is `DC1.CORP.EXAMPLE.COM`.

Next, run the ipa configuration script:

```
$ sudo ipa-server-install
```

You'll walk through several options here:

 * Don't do integrated DNS for now.
 * Specify server hostname as found above.
 * 

TODO: continue here



