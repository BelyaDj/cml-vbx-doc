# LVM

[TOC]

---

!!! important
    Before doing any operations with LVM, VM must be powered off 

---

## Create snapshot

```
lvcreate -L40G -s -n demo_host_clear /dev/libvirt_hdd/demo_host
```
---

## Restore from snaphot

```
lvconvert --merge libvirt_hdd/demo_host_clear
```

---
