---
layout: post
title:  "Create ZFS Pool"
slug:   create-pool
date:   2018-10-10 12:58:29
categories: ZFS
tags: 
 - zfs
 - zfsonlinux
 - filesystems
 - storage
 - ubuntu-18-04
---

## ZFS Overview
ZFS (fully called OpenZFS) is a software RAID and volume management tool - it replaces both hardware raid controllers, as well as 
volume managers, and snapshot tooling. It originated from Sun Microsystems, now Oracle, and is natively supported and developed
by the FreeBSD team. It was then ported linux under the project "ZFS on Linux".

ZFS provides a number of features, including:
 * Hardware RAID - no more raid cards
 * Datasets - Allowing one pool to be split into smaller flexible pools with their own replication, properties, and quotas.
 * Snapshots - Lightweight snapshots can be created, storing only the data that has been changed
 * ZFS Send / Receive - This allows a pool or dataset snapshot to be sent, on a block-level. This includes incremental sending,
   meaning that only changed data between snapshots must be sent.
 * Scrub / Metadata Integrity - ZFS checksums all data on disk to ensure no physical corruption can cause issues with the data. 
   Checksums are checked on every FS block read, as well as on a manual scrub, which is typically run about once-per-week, or 
   at least multiple times per month.

## Important Terms and Concepts

_Based on a guide from the [FreeBSD handbook](https://www.freebsd.org/doc/handbook/zfs-term.html)_

ZFS has a number of important terms and concepts to be aware of.

### Pool
A storage pool is the most basic building block of ZFS. A pool is made up of one or more vdevs, the underlying devices that store
the data. A pool is then used to create one or more file systems (datasets) or block devices (volumes). These datasets and volumes
share the pool of remaining free space. Each pool is uniquely identified by a name and a GUID. The features available are determined
by the ZFS version number on the pool.

TODO: document pool upgrade procedures

### Vdev
A pool consists of one or more striped vdevs. To use standard raid terminology, a pool is a RAID-0 of vdevs. As such - it is 
important that each vdev be fully redundant, because a loss of a vdev will result in the loss of a pool. Redundancy is provided by
making the vdevs themselves redundant.

In a production system, a vdev must be made up of a redundant set of disks. If a single disk is imported as a vdev, the loss of that 
single disk will cause the loss of the pool.

However, for testing and development, it may be suitable to use single-disk-vdevs, with the understanding that it is similar to
a RAID-0 raid volume, in which a single disk will cause the pool to be lost. Additionally, vdevs can not be removed from a pool.
If a pool needs to be downsized, a new pool needs to be built, then use `zfs send` and `zfs receive` to send the data to the new
pool.

Below is a list of the types of vdevs:

**Mirror**

A mirror is a set of disks containing the same data, and can contain two or more disks.

When creating a mirror, specify the mirror keyword followed by the list of member devices for the mirror. While the devices in the
mirror can be mixed, a mirror vdev will only hold as much data as its smallest member. A mirror vdev can withstand the failure of 
all but one of its members without losing any data. 

A mirror will write at the speed of it's slowest disk, as all devices need to receive each written block, and will read at a sum 
of the devices, because any block can be read from any device in the mirror.

A regular single disk vdev can be upgraded to a mirror vdev at any time with `zpool attach`.

**RAID-Z**

RAID-Z is a software implementation of RAID5 - it performs similarly to standard RAID-5 implementations. The capacity of each disk
can be mixed, but only the lowest-disk capacity is used across all disks. The capacity of a RAID-Z vdev is the sum of the raw disk
capacity, minus one disk for redundancy.

It is recommended to add no more than nine disks in a single vdev. If the configuration has more disks, it is recommended to divide 
them into separate vdevs and the pool data will be striped across them.

RAID-Z technically production safe, but not reccomended, because a single disk failure results in the vdev having no redundancy. Use
RAID-Z2 or 3 if possible.

**RAID-Z2**

RAID-Z2 is a software implementation of RAID6. It is similar to RAID-Z, but offers two redundant disks.

**RAID-Z2**

RAID-Z2 is a software implementation of RAID6. It is similar to RAID-Z, but offers three redundant disks. It will be slower than 
RAID-Z2 and 1, but is also highly resilient - you can loose up to 3 disks from the vdev safely.

**Disk** - not suitable for production

A *disk* vdev is a single disk. The most basic type of vdev is a standard block device. This can be an entire disk (such as `/dev/sda`)
or a partition (`/dev/sda1`). This is not suitable for production because the loss of that single disk will cause the pool to fail.

However, a disk vdev can be upgraded to a mirror by attaching a second disk of equal or larger size.

TODO: document the command to upgrade a disk to a mirror.

**File** - not suitable for production

In addition to disks, ZFS pools can be backed by regular files, this is especially useful for testing and experimentation. Use the
full path to the file as the device path in zpool create. All vdevs must be at least 128 MB in size.


**Spare**

ZFS has a special pseudo-vdev type for keeping track of available hot spares. Note that installed hot spares are not deployed
automatically; they must manually be configured to replace the failed device using `zfs replace`.

**Log**

ZFS Log Devices, also known as ZFS Intent Log (ZIL) move the intent log from the regular pool devices to a dedicated device,
typically an SSD. Having a dedicated log device can significantly improve the performance of applications with a high volume
of synchronous writes, especially databases. Log devices can be mirrored, but RAID-Z is not supported. If multiple log
devices are used, writes will be load balanced across them.

**Cache**

Adding a cache vdev to a pool will add the storage of the cache to the L2ARC. Cache devices cannot be mirrored. Since a cache
device only stores additional copies of existing data, there is no risk of data loss. Note that having a large cache device
can have adverse effects, because files in the cache must be referenced in memory. If the cache is too large, it can reduce 
the ability to cache data in memory, which will be faster than any cache device.


### Others

TODO: Document the following concepts:
 * pool - done
 * vdev - done
 * Copy-on-write
 * Dataset
 * Clone
 * Compression
 * Deduplication
 * Scrub
 * Resilver
 * Degraded pool
 * each of the vdev types (mirror, RAIDz, etc)
 * snapshot
 * clone
 * Special devices
   * ZIL
   * L2ARC
 

## RAID Card Note
Ideally, RAID cards should be replaced with JBOD HBA devices, to allow the ZFS to have full control over the drive. However, if 
this is not possible for whatever reason, setup each individual disk as it's own single-disk RAID 0 volume.

This is sub-par, because this stops ZFS from doing some lower-level drive access, instead providing it a RAID card abstraction,
but can be acceptable if required.


## Install Packages

First, install the needed packages from the `apt` repos.

```
sudo apt-add-repository universe -y &&
sudo apt update &&
sudo apt install -y zfsutils-linux
```

To confirm the ZFS package is installed properly, run:

```
sudo zpool status
```

This should return `no pools available`, which confirms that ZFS is installed,
and no pool is set up.

## Find Your Disks

Find all your block devices with `lsblk` (LiSt BLocK devices)

```
lsblk
```

Output will look like:
```
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
loop0    7:0    0  86.9M  1 loop /snap/core/4917
sda      8:0    0 136.1G  0 disk 
  sda1   8:1    0   512M  0 part /boot/efi
  sda2   8:2    0 135.6G  0 part /
sdb      8:16   0   2.7T  0 disk 
sdc      8:32   0   2.7T  0 disk 
sdd      8:48   0   2.7T  0 disk 
sde      8:64   0   2.7T  0 disk 
sdf      8:80   0 558.4G  0 disk 
sdg      8:96   0 558.4G  0 disk 
sdh      8:112  0 558.4G  0 disk 
sdi      8:128  0 558.4G  0 disk 
sdj      8:144  0 558.4G  0 disk 
sdk      8:160  0 558.4G  0 disk 
```

The first column is the device name in `/dev/*`. That is: `sde` is the
device found at `/dev/sde` on the system.

In this case, we are setting up 5x vdevs, each a mirror of two drives:

 * mirror 1 (3.0t): `sdb`, `sdc`
 * mirror 2 (3.0t): `sdd`, `sde`
 * mirror 3 (0.5t): `sdf`, `sdg`
 * mirror 4 (0.5t): `sdh`, `sdi`
 * mirror 5 (0.5t): `sdj`, `sdk`

These mirrors are added as high-level devices in the pool we will call `tank` for a total usable space of
about 7.5T.

## Pool Setup Basics
With the pool layout and device names found above, we create the pool.

The command syntax is as follows:

 * `zpool` : this is the name of the command line tool
 * `create` : create a new pool
 * `[name of pool]` : a string value naming the pool. Tank is often a default value used.
 * vdev definition block (repeat once per vdev in your pool)
   * `[type of vdev]` : in our case, we're using mirror. Could also be `RAIDZ` (raid5), `RAIDZ2` (raid 6), or `RAIDZ3` 
      (raid7), or some other values.
   * `device names in vdev`

Since this may not be clear, here are a few examples of commands and their output:


```
zpool create tank mirror c1d0 c2d0 mirror c3d0 c4d0
```

Create a pool called `tank`, with two vdevs. First vdev is a mirror 
with devices `/dev/c1d0` and `/dev/c2d0`, second vdev is a mirror 
with devices `/dev/c3d0` and `/dev/c4d0`.



```
zpool create tank raidz2 sdb sdc sdd sde sdf
```

Create a pool called `tank`, with one vdev. vdev is a raidz2 (raid6), 
consisting of `/dev/sdb` `/dev/sdc` `/dev/sdd` `/dev/sde` and `/dev/sdf`.


## Create Pool

In our case, the pool layout found above, we will create a pool called tank with the mirrors 
we found above using the following command:

```
sudo zpool create -n tank mirror sdb sdc mirror sdd sde mirror sdf sdg mirror sdh sdi mirror sdj sdk
```

The `-n` flag makes it a dry-run. This will allow you to see the proposed result of your actions.


The output will look like:

```
would create 'tank' with the following layout:

	tank
	  mirror
	    sdb
	    sdc
	  mirror
	    sdd
	    sde
	  mirror
	    sdf
	    sdg
	  mirror
	    sdh
	    sdi
	  mirror
	    sdj
	    sdk
```

This is what we expect: 5 pairs of mirrors.

Tweak the syntax with dry-runs as many times as is needed. No changes are made to the disks
when run in dry-run mode. 

When you are satisfied, remove `-n`, and the command will actually be executed on the disks.

```
sudo zpool create tank mirror sdb sdc mirror sdd sde mirror sdf sdg mirror sdh sdi mirror sdj sdk
```

If you receive no output, it was likely successful. Confirm by running a few commands to 
view the status of your pool.

 * `zpool status` - detailed pool information, including disk health
 * `zfs list` - list of all pools and datasets, showing free/used space
 * `df -h` - systemwide disk-space monitor - your pool should now be mounted in the root of 
    the disk at your pool name (ie -  a pool name tank is mounted by default at `/tank`).


## Create Datasets

Datasets are ways of dividing up a pool. A dataset can be assigned quotas, replicated, 
configured, mounted, and snapshot seperately from every other dataset.

For example, you might set one dataset for home directories, and another for VM images. Your VM
images might be snapshotted nightly, replicated off the host, and stored with compression 
disabled, whereas your home directories would be snapshotted hourly with a higher 
compression level enabled.

Each dataset is treated as it's own linux filesystem, but the storage capacity of the entire pool
is shared across datasets (unless you apply quotas or reservations to restrict capacity use).

Datasets can also be nested, like standard linux filesystems.

Create some datasets like this:

```
sudo zfs create tank/vms
sudo zfs create tank/vms/installers
sudo zfs create tank/vms/disks
sudo zfs create tank/system
sudo zfs create tank/system/homedirs
```

This creates datasets for VM install images, VM disks, and the home directories.

After creating your datasets, look again using the tools listed above (`df -h`, `zfs list`, etc), 
and you'll see your datasets listed there.

TODO: ADD MORE DETAILS HERE ABOUT HOW DATASETS LOOK LIKE NORMAL SUBDIRS BUT AREN'T
TODO: ADD MORE DETAILS HERE ABOUT DATASET NAMES VS MOUNT PATHS
