---
layout: post
title:  "Increase Virtual Disk Size"
date:   2018-10-10 12:58:29
categories: KVM
tags: 
 - kvm
 - virtual disks
 - qemu
 - virtual machine
---


KVM disks can be resized easily to provide more space.


First, find your disk image. Next, stop the virtual machine.

Finally, resize the image with the following command (example: increase size by 10 gb).

```
qemu-img resize [disk image] +10G
```





TODO : Research online VM resizing


Based on SE https://serverfault.com/a/724156/313980

Online Method (using qemu, libvirt, and virtio-block)

For better or worse, the commands below will run even if the target virtual disk is mounted. This can be useful in environments where the disk cannot be unmounted (such as a root partition), the VM must stay on, and the system owner is willing to assume the risk of data corruption. To remove that risk, you would need to log into the VM and unmount the target disk first, something that isn't always possible.

Perform the following from the KVM hypervisor.

    Increase the size of the disk image file itself (specify the amount to increase):

    qemu-img resize <my_vm>.img +10G

    Get the name of the virtio device, via the libvirt shell (drive-virtio-disk0 in this example):

    virsh qemu-monitor-command <my_vm> info block --hmp
      drive-virtio-disk0: removable=0 io-status=ok file=/var/lib/libvirt/images/<my_vm>.img ro=0 drv=raw encrypted=0
      drive-ide0-1-0: removable=1 locked=0 tray-open=0 io-status=ok [not inserted]

    Signal the virtio driver to detect the new size (specify the total new capacity):

    virsh qemu-monitor-command <my_vm> block_resize drive-virtio-disk0 20G --hmp

Then log into the VM. Running dmesg should report that the virtio disk detected a capacity change. At this point, go ahead and resize your partitions and LVM structure as needed.
