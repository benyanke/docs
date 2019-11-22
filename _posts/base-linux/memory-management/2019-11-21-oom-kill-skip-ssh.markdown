---
layout: post
title:  "Stop OOM from Killing SSH"
slug:   oom-stop-ssh
date:   2019-11-21 22:01:21
categories: Linux memory-management
tags: 
 - linux
 - ram
 - memory
 - ssh
---

The out-of-memory killer (OOM) in linux helps recover a system to a usable state if the memory presure begins
to grow too high, to avoid processes from failing to be able to allocate memory.


Using the OOM means you're more likely for your system to recover, but it also means a random process may get killed.


The first solution is ALWAYS to either reduce memory usage, or increase your memory. However, it's important to have
multiple safeguards, and one of those is ensuring that the SSH server daemon is never killed, so the system can be managed
even after a low memory event. If the SSH daemon is killed and not restarted, and you do not have physical access to 
the system, this could lead to a lockout where you are unable to access it.

To resolve this issue, write `-17` to `/proc/[pid]/oom_adj`, where `[pid]` is the SSH daemon's pid.

This can be done one-time by running:

```bash
echo -17 > /proc/`cat /var/run/sshd.pid`/oom_adj
```

Or alternatively, it can be scheduled in cron every few minutes, to ensure it's always up to date.


Place it in one of the files in `/etc/cron.d/`, like so:

```bash
# Ensure SSH is never killed by disabling the OOM on the SSH process
* * * * * root cat /var/run/sshd.pid &> /dev/null && echo -17 > /proc/`cat /var/run/sshd.pid`/oom_adj
```

The first half ensures this only runs if the file exists, and the second half does the write.

