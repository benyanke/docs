---
layout: post
title:  "Clear all Caches from RAM in Linux"
slug:   linux-clear-memory-caches
date:   2019-09-02 22:04:10
categories: Linux memory-management
tags: 
 - linux
 - ram
 - memory
 - cache
 - buffers
---

In linux, memory (RAM) is used to cache various things, such as the disks backing the filesystem. This means
that a long-running linux system will be far more efficent. However, this will also lead to high memory usage,
typically in the upper 80 or 90% range. Typically, this is not a problem, as the system will self-adjust.

However, this can be an issue if you need to allocate a large block of memory - for example, starting a KVM
virtual machine.

If you are seeing an error on starting a process such as "can not allocate memory", and your memory usage is 
in the upper 90%, while not being used by other system processes, you may need to clear the memory cache to free
space in your memory.


In a shell, elevate to root:

```bash
 $ sudo su
```

Once you are root, run the following commands:

```bash
 # free && sync && echo 3 > /proc/sys/vm/drop_caches && free
```

This may take 10-15 seconds, but once you're done, you'll notice your memory usage is likely lower. You can now start
your large memory process, and continue with your day! Note that this will clear all your caches, and require any
file reads to go back to disk instead of being cached, so run this knowing that it's likely to slow down your system
for a short time. As such, only use this when required.
