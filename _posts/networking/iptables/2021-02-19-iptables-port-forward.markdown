---
layout: post
title:  "NAT Port Forwards with Iptables
slug:   nat-port-forward-with-iptables
date:   2021-02-19 22:53:12
categories: Networking
tags: 
 - networking
 - iptables
 - nat
---

## Enable Kernel Packet Forwarding
To enable immediately, open a root shell and run:

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

And to persist after reboot, ensure the following line is set in `/etc/sysctl.conf`:

```
net.ipv4.ip_forward = 1
```

## Setup Iptables Rules

Run the snippet below, replacing the environment variables with the proper values. `PUBLIC_IP` should
be the IP bound to a local NIC which you want to listen on to forward traffic.

```bash
PUBLIC_IP="203.0.113.23"
BACKEND_IP="10.10.10.24"
FRONTEND_PORT="80"
BACKEND_PORT="8080"

# Run once per rule to create
iptables -t nat -A PREROUTING -p tcp --dport ${FRONTEND_PORT} -j DNAT --to-destination ${BACKEND_IP}:${BACKEND_PORT}

# Only run once
iptables -t nat -A POSTROUTING -j MASQUERADE
```

After running the commands, the rules are in place in memory, but not persistent.

Once done, you can list the rules that are currently running to confirm.

```bash
iptables -t nat --list
```

## Persist and Restore Rules

Next, we must dump the current rules to a file so it can be loaded on every boot.

First, dump to a file.

```bash
iptables-save > /etc/iptables.rules
```

Then, open `/etc/systemd/system/restore-iptables-rules.service` in your text editor of
choice, and put the following content in the file:

```
[Unit]
Description = Apply iptables rules

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'iptables-restore < /etc/iptables.rules'

[Install]
WantedBy=network-pre.target
```

Then, run:

```bash
sudo systemctl enable restore-iptables-rules.service
```


## Final Test
At this point, your rules are running in memory, and will be put into place
on each boot. Reboot your system to confirm functionality.
