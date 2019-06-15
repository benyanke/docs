---
layout: post
title:  "Edit a VM's XML Configuration with Virsh"
slug:   virsh-edit-xml-configuration
date:   2018-11-8 21:53:29
categories: KVM
tags: 
 - kvm
 - qemu
 - virsh
 - virtual machine
 - hypervisor
 - ubuntu-18-04
---


Typically KVM instances are managed through tools like `virt-manager`, but 
sometimes it's needed to edit the XML configuraiton directly to adjust more 
specific settings.


Start by dumping the XML to a temporary file. You can name this file 
whatever you wish - it is simply used for editing and reimporting.

```
virsh dumpxml name_of_vm > name_of_vm.xml
```

This dumps the configuration to an XML file in the temporary location you 
specified. Now, you can edit your file (in the example, `name_of_vm.xml`), 
and make whatever XML changes are required.

Once you are done, finish by reloading the temporary XML file back into 
virsh.

```
# Undefine the old vm to prevent an error because of an duplicate UUID.
virsh undefine name-of-vm

# Reimport the file
virsh define name_of_vm.xml
```

This just changes the stored configuration for the VM. To put it into 
effect, restart the VM by stopping (destroying, in KVM terms) and starting 
it.

```
virsh destroy name_of_vm;
virsh start name_of_vm;
```

Your new configuration is now in effect. You can now delete the XML file - 
it is only used during the edit and import phases. It is no longer needed 
after the `define` step.
