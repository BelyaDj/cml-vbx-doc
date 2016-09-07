# Glance

[TOC]

---

## How Glance stores images

By default, **MOS** Glance stores images in Swift
```
# grep -h '^default_store' /etc/glance/glance-api.conf
default_store = swift
```

Look at `direct_url`
```
# glance --os-image-api-version 2 image-show d8ac6b41-6c01-4000-bb91-ef338a2df91c
+------------------+--------------------------------------------------------------------------------------------------------+
| Property         | Value                                                                                                  |
+------------------+--------------------------------------------------------------------------------------------------------+
| checksum         | ee1eca47dc88f4879d8a229cc70a07c6                                                                       |
| container_format | bare                                                                                                   |
| created_at       | 2016-04-25T09:47:22Z                                                                                   |
| direct_url       | swift+http://services%3Aglance:b7svRbjT@10.0.0.2:5000/v2.0/glance/d8ac6b41-6c01-4000-bb91-ef338a2df91c |
| disk_format      | qcow2                                                                                                  |
| id               | d8ac6b41-6c01-4000-bb91-ef338a2df91c                                                                   |
| min_disk         | 0                                                                                                      |
| min_ram          | 64                                                                                                     |
| name             | TestVM                                                                                                 |
| owner            | ee7f1117e87e4fb4bdfda2bcd446b214                                                                       |
| protected        | False                                                                                                  |
| size             | 13287936                                                                                               |
| status           | active                                                                                                 |
| tags             | []                                                                                                     |
| updated_at       | 2016-04-25T09:47:23Z                                                                                   |
| virtual_size     | None                                                                                                   |
| visibility       | public                                                                                                 |
+------------------+--------------------------------------------------------------------------------------------------------+
```

---

## Additional information

By default, OpenStack Glance saves images and OpenStack Instance snapshots on the local filesystem in `/var/lib/glance/images/`.
`/var/lib/glance/` images is a GlusterFS native mount to a Gluster volume off the storage layer.
**The Image Service is run on all controller nodes**, ensuring at least one instance will be available in
case of node failure. It also sits behind HAProxy, which detects when the software fails and routes
requests around the failing instance.

The OpenStack Image Service consists of two parts: glance-api and glance-registry. The former is responsible for the delivery of images;
the compute node uses it to download images from the backend. The latter maintains the metadata information associated with virtual machine
images and requires a database. The glance-api part is an abstraction layer that allows a choice of backend.
Currently, it supports:

* **OpenStack Object Storage**
    * Allows you to store images as objects.
* **File system**
    * Uses any traditional file system to store the images as files.
* **S3**
    * Allows you to fetch images from Amazon S3.
* **HTTP**
    * Allows you to fetch images from a web server. You cannot write images by using this mode.

---
