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
published: false
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

## Checks

```
kvm-ok
```

This should output `KVM acceleration can be used`. If it does, you've successfully 
installed KVM.
