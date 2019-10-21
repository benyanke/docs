---
layout: post
title:  "Mikrotik CAPsMAN v2 Setup Guide"
slug:   capsman-v2-setup-guide
date:   2019-10-13 08:47:21
categories: Networking
tags: 
 - mikrotik
 - cap
 - capsman
 - wifi
 - access-point
 - ansible
draft: true
---

## Introduction

[TODO : write this]


## Server Setup

Login to your CAPsMAN server (probably your main gateway device) via SSH or the web console.

Set up a channel profile: - you may adjust the attributes as needed

```bash
[admin@yourdevice] > /caps-man channel add comment=5g-profile name=5g tx-power=40 band=5ghz-a/n/ac
[admin@yourdevice] > /caps-man channel add comment=2g-profile name=2g tx-power=40 band=2ghz-b/g/n
```

Now we'll setup security profiles:

```bash
[admin@yourdevice] > /caps-man security add authentication-types=wpa2-psk disable-pmkid=no encryption=aes-ccm group-encryption=aes-ccm group-key-update=30m name=secure comment="Main security profile"
```

And the datapath for trunking back to the controller:

```bash
[admin@yourdevice] > /caps-man datapath add bridge=bridge name=secure
```

Now, with the other config in place, we'll add the configuration for your first wifi network. The SSID will be `net1`, with a password
of `password`. Replace `bridge` with the name of your bridge. Note: on a default Mikrotik router config, the bridge is simply 
named `bridge`, so you may not need to change this.

Note: `united states3` refers to the third iteration of the `united states` standard. If the frequency range is ever adjusted
in the future by the FCC, `united states4` will then be created, and the next change will be `united states5`, etc.

```bash
[admin@yourdevice] > /caps-man configuration add name=config-net1-5g mode=ap ssid=net1-5g country="united states3" datapath=secure security=secure security.passphrase=password channel=5g comment="ssid: net1-5g | pw: [password]"
[admin@yourdevice] > /caps-man configuration add name=config-net1-2g mode=ap ssid=net1-2g country="united states3" datapath=secure security=secure security.passphrase=password channel=2g comment="ssid: net1-2g | pw: [password]"
```

Now we indicate to CAPsMAN that we should use the created configuration. Change the `master-configuration` key to match the named configuration above.

```bash
[admin@yourdevice] > /caps-man provisioning add action=create-dynamic-enabled master-configuration=config-net1
```

Then, specify which interface are allowed to accept CAPs. This is generally going to be your internal LAN master interface, and disallowing the 
WAN interface. The second line then disallows all interfaces, to effectively turn it from a blacklist into a whitelist. Now, CAP requests will only
be served on the LAN.

```bash
[admin@yourdevice] > /caps-man manager interface add comment="Allow CAPs From internal Lan" disabled=no forbid=no interface=bridge
[admin@yourdevice] > /caps-man manager interface set [find interface=all] forbid=yes comment="Deny from all except whitelisted"
```


Finally, enable CAPsMAN to listen for CAPs requests:

```bash
[admin@yourdevice] > /caps-man manager set enabled=yes
```

## Client AP Setup

The CAPsMAN server is now set up. We can now set up the APs to connect to it.

Log into the access point, and connect it to the CAPsMAN server. Specify the internal address of your server.
`interfaces` are the wireless radios to be managed by the server, while `discovery-interfaces` are the interfaces
to use to look for the server (typically the trunk back to your main network).

```bash
[admin@ap01] > /interface wireless cap set enabled=yes interfaces=wlan1,wlan2 discovery-interfaces=ether1,ether2 caps-man-addresses=10.10.10.1 bridge=bridge
```

TODO : Add a way to check here on the status

TODO : document multi SSID features and vlan

TODO : document trunking? not sure if needed

server setup guide:
https://wiki.mikrotik.com/wiki/Manual:Simple_CAPsMAN_setup
