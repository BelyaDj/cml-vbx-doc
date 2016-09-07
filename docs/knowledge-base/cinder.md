# Cinder - Block Storage Service

[TOC]

---

## Install service "Cinder-backup" on MOS 8

1) In controller check cinder services:
```
# cinder service-list
+------------------+---------------------+------+---------+-------+----------------------------+-----------------+

|      Binary      |         Host        | Zone |  Status | State |         Updated_at         | Disabled Reason |

+------------------+---------------------+------+---------+-------+----------------------------+-----------------+

| cinder-scheduler | node-11.virtbox.com | nova | enabled |   up  | 2016-07-28T11:11:47.000000 |        -        |

|  cinder-volume   | node-13.virtbox.com | nova | enabled |   up  | 2016-07-28T11:11:43.000000 |        -        |

+------------------+---------------------+------+---------+-------+----------------------------+-----------------+
```
See that there isn't service “cinder-backup”

2) Go to cinder-volume-node (node-13) and install backup-service and nfs-client:
```sh
# apt-get install cinder-backup nfs-common -y
```

After installation cinder-backup service isn't running.
You should configure it before.

On Controller-node and Cinder-node:
```sh
# vi /etc/cinder/cinder.conf
```

Add in  [DEFAULT]:
```
backup_driver = cinder.backup.drivers.nfs
backup_share = 192.168.31.63:/nfs
```

On Cinder-node:
```sh
# service cinder-backup restart
# service cinder-volume restart
```

On Controller-node:
```sh
# service cinder-scheduler restart
```

Check mount-point on Cinder-node:
(mount-point is creating then “cinder backup-create” is running)
```sh
# df -h
Filesystem          Size  Used Avail Use% Mounted on
udev                2.0G   12K  2.0G   1% /dev
tmpfs               396M  428K  395M   1% /run
/dev/dm-0            20G  1.8G   17G  10% /
none                4.0K     0  4.0K   0% /sys/fs/cgroup
none                5.0M     0  5.0M   0% /run/lock
none                2.0G     0  2.0G   0% /run/shm
none                100M     0  100M   0% /run/user
/dev/vda3           196M   58M  129M  32% /boot
192.168.31.63:/nfs   99G  336M   93G   1% /var/lib/cinder/backup_mount/73ec1f519aac3d83baa8a5cecc00ab21
```

Check permissions on '/var/lib/cinder/backup_mount/73ec1f519aac3d83baa8a5cecc00ab21'

Owner should be 'cinder:cinder'

Make
```sh
# chown -R cinder:cinder /var/lib/cinder/backup_mount/73ec1f519aac3d83baa8a5cecc00ab21 
```

### Bugs

- In Openstack earlier Kilo you can't create incremental volume backup.
- In Openstack earlier Liberty you can't create backup attached volume (no --force attribute).
- In Openstack earlier Mitaka you can delete full backup without incremental backups. Incremental backups are staying without references. In Mitaka deletion full backup delete all incremental backup.


## Working with NFS (on Ubuntu 14.04)

### Install NFS-server

On Server (Ubuntu 14.04)
```sh
# apt-get install nfs-kernel-server
```

Edit '/etc/exports' and add the shares:
```sh
/ubuntu  *(ro,sync,no_root_squash) 
       or
/home    *(rw,sync,no_root_squash)
```

```
# service nfs-kernel-server restart
```

### Install NFS-client

On Client (Ubuntu 14.04)
```sh
# apt-get install nfs-common
```

Mount NFS-share to necessary folder:
```sh
# mount [ip_nfs_server]:/[shared_path_from_exports] /mnt                  (or to another mount point)
```

Check mount-points:
```
# df -h
```