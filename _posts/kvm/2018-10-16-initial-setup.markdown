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
is generally simply refered to as "KVM". KVM is the core kernel module, and QEMU is the 
related userland components. They are closely intertwined but distinct software 
projects.

## Installing KVM



TODO - perhaps most of these should be their own guides, but logging here

 - Creating a guest on KVM
 - virtio drivers - research this
 - virt manager
 - virsh basics
 - migrations
 - editing raw XML
 - KVM tuning
