---
layout: post
title:  "Mount a QCOW2 File as a Filesystem"
slug:   mount-qcow2-as-filesystem
date:   2021-11-18 21:53:29
categories: KVM
tags: 
 - kvm
 - qcow2
 - qemu
 - virsh
 - virtual machine
 - hypervisor
 - ubuntu-20-04
---

Typically, your VM disks will exist on the host as `qcow2` files. However, if you ever need to do something on the VM disk 
without booting the VM, you can mount the `qcow2` file directly on the host. Note that you can only do this either 1) while the 
guest is shut down or 2) on a clone of the guest's disk, not on a running guest.

This can be useful for automating tasks in the guest from the host, or pulling a file from an old snapshot.

This technique can be particularly useful in combination with ZFS snapshots on the host, where you use the read-only 
snapshot of the VM disk and mount it to fetch a file from within the guest, at the time of the snapshot, reducing the 
need for in-guest backup agents.

## Which One?

If you want to change files within the guest's disk, you'll need to mount read-write. However, if you're simply trying 
to read a file from the guest, especially from something like a ZFS snapshot, you'll need to take the additional steps 
of mounting read-only, which doesn't work by default. Pick one of the two below.

## Mount Read Write

First, enable `qemu-nbd` on the host, which is the daemon that allows the qcow2 file to be mounted as a filesystem.

```
sudo modprobe nbd max_part=8
```

Next, create the mount with `qemu-nbd`.

```
sudo qemu-nbd --connect=/dev/nbd0 /var/lib/libvirt/images/app01.qcow2
```

Then, use `fdisk` to view the partitions that can be mounted.

```
root@host:~# fdisk /dev/nbd0 -l

Disk /dev/nbd0: 50 GiB, 53687091200 bytes, 104857600 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 0A8E7183-678B-4618-84F5-8A5F44F7FEC3

Device      Start       End   Sectors Size Type
/dev/nbd0p1  2048      4095      2048   1M BIOS boot
/dev/nbd0p2  4096 104855167 104851072  50G Linux filesystem
root@host:~#
```

From `fdisk`, it appears that `/dev/nbd0p2` is the the VM's root volume.

Now, make a spot to mount it, and run the mount.

```
sudo mkdir /mnt/app01/
sudo mount /dev/nbd0p2 /mnt/app01
```

Now, check your mount point - you should have your VM root mounted to your mountpoint.

```
$ df -h
Filesystem     Size  Used Avail Use% Mounted on
/dev/nbd0p2    50G   43G  4.6G  91% /mnt/app01
```

You may now copy out the files you need.

When done, you need to tear down the mount, and disable `qemu-nbd`.

```
sudo umount /mnt/app01
sudo qemu-nbd --disconnect /dev/nbd0
sudo rmmod nbd
```

## Mount Read Only

First, enable `qemu-nbd` on the host, which is the daemon that allows the qcow2 file to be mounted as a filesystem.

```
sudo modprobe nbd max_part=8
```

Next, create the mount with `qemu-nbd`.

```
sudo qemu-nbd --read-only --connect=/dev/nbd0 /var/lib/libvirt/images/app01.qcow2
```

Then, use `fdisk` to view the partitions that can be mounted.

```
root@host:~# fdisk /dev/nbd0 -l

Disk /dev/nbd0: 50 GiB, 53687091200 bytes, 104857600 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 0A8E7183-678B-4618-84F5-8A5F44F7FEC3

Device      Start       End   Sectors Size Type
/dev/nbd0p1  2048      4095      2048   1M BIOS boot
/dev/nbd0p2  4096 104855167 104851072  50G Linux filesystem
root@host:~#
```

From `fdisk`, it appears that `/dev/nbd0p2` is the the VM's root volume.

Now, make a spot to mount it, and run the mount.

Note: the `norecovery` is required to properly mount read-only. This is not an issue, 
because by nature of mounting read-only, we aren't writing any data to the disk, and so 
recovery of failed writes is not a problem.


```
sudo mkdir /mnt/app01/
sudo mount -o ro,norecovery /dev/nbd0p2 /mnt/app01
```

Now, check your mount point - you should have your VM root mounted to your mountpoint.

```
$ df -h
Filesystem     Size  Used Avail Use% Mounted on
/dev/nbd0p2    50G   43G  4.6G  91% /mnt/app01
```

You may now copy out the files you need.

When done, you need to tear down the mount, and disable `qemu-nbd`.

```
sudo umount /mnt/app01
sudo qemu-nbd --disconnect /dev/nbd0
sudo rmmod nbd
```
