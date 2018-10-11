---
layout: post
title:  "Create Host Bridge Network"
date:   2018-10-10 12:58:29
categories: KVM
tags: 
 - kvm
 - qemu
 - virtual machine
---


By default, KVM provides a NAT network for VMs. This documents the setup of a
bridge interface, to allow VMs on the  host to join the network of the host,
instead of being behind a NAT (which is default).

In 18.04 this is accomplished by using the netplan network configuration tool,
which was not used in the previous LTS.

Most installations will have a `50-cloud-init.yaml` by default, but you may
have to adjust if not.

The bridge interface will replace the main interface, by default.

First, install `bridge-utils`:

```
sudo apt update && sudo apt install -y bridge-utils 
```

Open the file:

```
sudo nano /etc/netplan/50-cloud-init.yaml
```

By default, it will look something like:
```
network:
    ethernets:
        enp1s0:
            addresses: []
            dhcp4: true
        enp2s0:
            addresses: []
            dhcp4: true
            optional: true
    version: 2
```

First, disable DHCP on the interface you wish to bridge. Set `dhcp4: true` to `dhcp4: no`.

Add this to the bottom, noting the interface which
it should be bridged to. To bridge with a static IP:
(NOTE: UNTESTED)
```
    bridges:
        br0:
            interfaces: [enp0s3]
            dhcp4: no
            optional: true
            addresses: [172.168.0.10/24]
            gateway4: 172.168.0.1
            nameservers:
              addresses: [172.168.0.1]
```

Or, to bridge with DHCP:
```
    bridges:
        br0:
            interfaces: [enp1s0]
            dhcp4: true
            optional: true
```


The final file will look like:
```
network:
    ethernets:
        enp1s0:
            addresses: []
            dhcp4: no   
        enp2s0:
            addresses: []
            dhcp4: true
            optional: true
    version: 2

    bridges:
        br0:
            interfaces: [enp1s0]
            dhcp4: true
            optional: true
```

To apply the configuration:
```
sudo netplan apply
```

Or to apply with debugging:
```
sudo netplan --debug  apply
```

This will disable the nic you mention, and enable the new bridge nic. This will
change the new mac address, and the IP address, so you will have to update the DHCP 
reservation if this is being used.


Next, confirm the bridge is set up properly:
```
sudo networkctl status -a
```

`br0` should be in state: `routable (configured)`.

Similarly, check the ip addresses:
```
ip addr
```

br0 should have an ip address associated.

Finally, to connect a VM to the bridge network, when creating the VM in `virt-manager`,
select "Specify shared device name", and enter `br0` in the text field.
