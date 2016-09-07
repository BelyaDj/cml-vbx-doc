# Nova

[TOC]

---

## How Nova stores virtual machine disks

**An ephemeral storage includes a root ephemeral volume and an additional ephemeral volume.**

The root disk is associated with an instance, and exists only for the life of this very instance.
Generally, it is used to store an instance's root file system, persists across the guest operating
system reboots, and is removed on an instance deletion. The amount of the root
ephemeral volume is defined by the flavor of an instance. **(By default is '0', which means "No limit")**

In addition to the ephemeral root volume, all default types of flavors, except `m1.tiny`,
which is the smallest one, provide an additional ephemeral block device sized between 20
and 160 GB (a configurable value to suit an environment). **It is represented as a raw block
device with no partition table or file system.** A cloud-aware operating system can discover,
format, and mount such a storage device. OpenStack Compute defines the default file system
for different operating systems as Ext4 for Linux distributions, VFAT for non-Linux and
non-Windows operating systems, and NTFS for Windows. However, it is possible to specify
any other filesystem type by using `virt_mkfs` or `default_ephemeral_format` configuration options.

*OpenStack Cloud Administrator Guide, p. 61, "Ephemeral storage"*

---
