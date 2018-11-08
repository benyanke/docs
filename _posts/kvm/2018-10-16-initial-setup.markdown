---
layout: post
title:  "Setup a KVM/QEMU Hypervisor"
slug:   create-host
date:   2018-10-16 12:58:29
categories: KVM
tags: 
 - kvm
 - qemu
 - virtual machine
 - hypervisor
 - ubuntu-18-04
---

KVM stands for Kernel Virtual Machine, a hypervisor which runs on Linux servers. This 
hypervisor is used by many in production, including multiple cloud providers.

KVM is actually made up two pieces: KVM and QEMU, though colloquially, the entire stack 
is generally simply refered to as "KVM". KVM is the core kernel hypervisor module, and 
QEMU is an emulator along with the related userland components. They are closely 
intertwined but distinct software projects. Strictly speaking, QEMU could be run on it's 
own, as an emulator, but KVM increases speed via hardware hypervisor functionality.

## Pre-Checks

First, you must run a few pre-checks to ensure that your host properly supports all of 
the features to fully take advantage of VM hosting.

This ensures that your processor supports VT-x (Intel) or AMD-V (AMD), the hardware 
virtualization extensions. This outputs an integer: 0 means zero supported cores found, 
` or more means your CPU supports these virtualization extensions.

```
egrep -c '(vmx|svm)' /proc/cpuinfo
```

## Installing KVM

Next, install the 
```
sudo apt-get install -y qemu-kvm libvirt-bin virtinst bridge-utils cpu-checker
```

Additionally, you will probably want to install the following package, which provides 
virt-* tools, which allow you to access guest disk images without booting the VM 
(`virt-ls`, `virt-df`, etc). These are not strictly required, however.

```
sudo apt-get install -y libguestfs-tools
```

Run the following command to ensure KVM is installed:

```
kvm-ok
```

This should output `KVM acceleration can be used`. If it does, you've successfully 
installed KVM.

## Group Permissions 

By default, only root has permission to modify VMs. However, when using tools like 
`virt-manager` remotely, you do not want to SSH as root, as this is insecure.

Instead, add any users needing to modify VMs to the `libvirt` group. 

```
sudo usermod -aG libvirt [username]
```

This will not take affect until after the user's session is restarted.

The user can check their own groups by simply running `group`, which outputs the groups 
that the user is part of.

Additionally, you can run `group [username]` to check the group membership of another 
user.


## Complete!

You're done and ready to run VMs. There are many management tools available for KVM, 
including `virsh` (terminal), `virt-manager` (GUI), and a host of web based tools.

## Networking

By default, VMs will exist on a NAT network on the host - that is - they have internet 
access, but are not easily accessable from outside the host.

Most configurations use a bridge network - Checkout the next guide to set this up:

_[Install KVM Bridge Network]({{ site.baseurl }}{% post_url kvm/2018-10-10-bridge-network %})_

