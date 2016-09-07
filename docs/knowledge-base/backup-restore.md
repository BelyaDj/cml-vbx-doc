# Backup and Restore

[TOC]

---

## Backup and Restore Nova instance with ephemeral root disk without Cinder Volume

Tested on MOS 8 (OpenStack Liberty)

---

### Backup Virtual Machine with ephemeral root disk via snapshot to Glance (stores as image)
 
On Controller node:

1) set OS variables

```sh
# source openrc
```

2) List instances:

```sh
# nova list
```

3.1) Create snapshot:

```sh
nova image-create [--show] [--poll] <server> <name>

# nova image-create --show --poll c3688d9a-3130-43aa-9454-dc572c3d01cf vm1-backup
```

as result:
```
Server snapshotting... 100% complete
Finished
+-------------------------+--------------------------------------+
| Property                | Value                                |
+-------------------------+--------------------------------------+
| OS-EXT-IMG-SIZE:size    | 22085632                             |
| created                 | 2016-08-17T12:50:29Z                 |
| id                      | eae4f09e-c5d8-4aa7-a118-d0b470946abd |
| metadata base_image_ref | a4c37608-9cbd-4f31-9b2e-d1d328671f0e |
| metadata image_location | snapshot                             |
| metadata image_state    | available                            |
| metadata image_type     | snapshot                             |
| metadata instance_uuid  | c3688d9a-3130-43aa-9454-dc572c3d01cf |
| metadata owner_id       | 3b61fd30ecf349389d259114607e7926     |
| metadata user_id        | d6a1978d22bf498389b48e8ee7da6a5b     |
| minDisk                 | 10                                   |
| minRam                  | 0                                    |
| name                    | vm1-backup                           |
| progress                | 100                                  |
| server                  | c3688d9a-3130-43aa-9454-dc572c3d01cf |
| status                  | ACTIVE                               |
| updated                 | 2016-08-17T12:50:38Z                 |
+-------------------------+--------------------------------------+
```

For more information `nova help image-create`

3.2) Or make nova backup

```
nova backup <server> <name> <backup-type> <rotation>

# nova backup c3688d9a-3130-43aa-9454-dc572c3d01cf vm1-backup2 daily 7
```
For more information `nova help backup`

4) List images

```sh
# glance image-list
```

result:
```
+--------------------------------------+--------------------------------------------------+
| ID                                   | Name                                             |
+--------------------------------------+--------------------------------------------------+
| eae4f09e-c5d8-4aa7-a118-d0b470946abd | vm1-backup                                       |
| 9fc5c2fd-c7ef-4f2a-b57d-52f06ccb7964 | vm1-backup2                                      |
+--------------------------------------+--------------------------------------------------+
```

Shows that this is snapshot (field 'image_type'): `glance image-show eae4f09e-c5d8-4aa7-a118-d0b470946abd`

5) Download snapshot (for migrating or storing in NFS)

```sh
glance image-download --file <file_name> <snapshot_id>

# glance image-download --file vm1-bak.raw eae4f09e-c5d8-4aa7-a118-d0b470946abd
```

---

### Restore Virtual Machine with ephemeral root disk from snapshot (Glance image)

On Controller node

1) Import snapshot from file (if it doesn't exist in Glance)

```
# glance image-create --file ./vm1-bak.raw --name vm1-restored --container-format bare --disk-format qcow2 --visibility public
+------------------+--------------------------------------+
| Property         | Value                                |
+------------------+--------------------------------------+
| checksum         | 8e69ea2c0533dff5c8e5ef1122199ec8     |
| container_format | bare                                 |
| created_at       | 2016-08-17T13:09:35Z                 |
| disk_format      | qcow2                                |
| id               | 7f101566-4e9f-4bfc-b5fb-a48cd9ba99a3 |
| min_disk         | 0                                    |
| min_ram          | 0                                    |
| name             | vm1-restored                         |
| owner            | 3b61fd30ecf349389d259114607e7926     |
| protected        | False                                |
| size             | 22085632                             |
| status           | active                               |
| tags             | []                                   |
| updated_at       | 2016-08-17T13:09:37Z                 |
| virtual_size     | None                                 |
| visibility       | public                               |
+------------------+--------------------------------------+
```

For more information `glance help image-create`

2) List images:

```sh
# glance image-list
```

3) Restore VM from image

```
nova boot --flavor <flavor> <image> --nic <net-id=net-uuid,v4-fixed-ip=ip-addr,v6-fixed-ip=ip-addr,port-id=port-uuid> <vm_name>
```
* <flavor\> should be equal or greater then "backuped" instance, can be obtained `nova flavor-list`
* <net-id\> can be obtained `neutron net-list`

For all arguments try `nova help boot`

Restore:
```
# nova boot --flavor m1.small --image 7f101566-4e9f-4bfc-b5fb-a48cd9ba99a3 --nic net-id=f1adfde0-de6b-4642-aeec-7380544d2ca7 --poll vm1-restored
+--------------------------------------+-----------------------------------------------------+
| Property                             | Value                                               |
+--------------------------------------+-----------------------------------------------------+
| OS-DCF:diskConfig                    | MANUAL                                              |
| OS-EXT-AZ:availability_zone          |                                                     |
| OS-EXT-SRV-ATTR:host                 | -                                                   |
| OS-EXT-SRV-ATTR:hypervisor_hostname  | -                                                   |
| OS-EXT-SRV-ATTR:instance_name        | instance-000003e8                                   |
| OS-EXT-STS:power_state               | 0                                                   |
| OS-EXT-STS:task_state                | scheduling                                          |
| OS-EXT-STS:vm_state                  | building                                            |
| OS-SRV-USG:launched_at               | -                                                   |
| OS-SRV-USG:terminated_at             | -                                                   |
| accessIPv4                           |                                                     |
| accessIPv6                           |                                                     |
| adminPass                            | a6xhzSqoFKe4                                        |
| config_drive                         |                                                     |
| created                              | 2016-08-17T13:45:27Z                                |
| flavor                               | m1.small (2)                                        |
| hostId                               |                                                     |
| id                                   | b8c5f660-070f-4289-bbe1-9f4c70573d7e                |
| image                                | vm1-restored (7f101566-4e9f-4bfc-b5fb-a48cd9ba99a3) |
| key_name                             | -                                                   |
| metadata                             | {}                                                  |
| name                                 | vm1-restored                                        |
| os-extended-volumes:volumes_attached | []                                                  |
| progress                             | 0                                                   |
| security_groups                      | default                                             |
| status                               | BUILD                                               |
| tenant_id                            | 3b61fd30ecf349389d259114607e7926                    |
| updated                              | 2016-08-17T13:45:27Z                                |
| user_id                              | d6a1978d22bf498389b48e8ee7da6a5b                    |
+--------------------------------------+-----------------------------------------------------+

Server building... 100% complete
Finished
```

4) List VMs:

```
# nova list
+--------------------------------------+--------------+--------+------------+-------------+-------------------------------+
| ID                                   | Name         | Status | Task State | Power State | Networks                      |
+--------------------------------------+--------------+--------+------------+-------------+-------------------------------+
| c3688d9a-3130-43aa-9454-dc572c3d01cf | nova-1       | ACTIVE | -          | Running     | admin_internal_net=10.0.111.4 |
| b8c5f660-070f-4289-bbe1-9f4c70573d7e | vm1-restored | ACTIVE | -          | Running     | admin_internal_net=10.0.111.5 |
+--------------------------------------+--------------+--------+------------+-------------+-------------------------------+
```

---

## Backup and Restore Nova instance with Cinder Volume as root-disk

Tested on MOS 8 (OpenStack Liberty)

---

### Backup Virtual Machine with Cinder Volume as root-disk

---

#### Backup Virtual Machine with Cinder Volume as root-disk via Cinder-backup

On Controller node:

1) Check that Cinder-backup service exists
```
# cinder service-list
+------------------+--------------------+------+---------+-------+----------------------------+-----------------+
|      Binary      |        Host        | Zone |  Status | State |         Updated_at         | Disabled Reason |
+------------------+--------------------+------+---------+-------+----------------------------+-----------------+
|  cinder-backup   | node-2.virtbox.com | nova | enabled |   up  | 2016-08-17T15:18:04.000000 |        -        |
| cinder-scheduler | node-5.virtbox.com | nova | enabled |   up  | 2016-08-17T15:17:58.000000 |        -        |
|  cinder-volume   | node-2.virtbox.com | nova | enabled |   up  | 2016-08-17T15:18:04.000000 |        -        |
+------------------+--------------------+------+---------+-------+----------------------------+-----------------+
```

If Cinder-backup service doesn't exist look how to install and configure it here [Install and configure Cinder-backup service](/knowledge-base/cinder/#install-service-cinder-backup-on-mos-8) 

2) List Cinder volumes
```sh
# cinder list
+--------------------------------------+-----------+------------------+---------------+------+----------------------+----------+-------------+-------------+
|                  ID                  |   Status  | Migration Status |      Name     | Size |     Volume Type      | Bootable | Multiattach | Attached to |
+--------------------------------------+-----------+------------------+---------------+------+----------------------+----------+-------------+-------------+
| 5c414913-e0c1-48f3-80c0-7b7fc3a9aef4 | available |        -         | test-1        |  10  |          -           |   true   |    False    |             |
+--------------------------------------+-----------+------------------+---------------+------+----------------------+----------+-------------+-------------+
```

3.1) Stop instance and detach volume or create some non-empty volume (in any way: from Web or CLI), list it (on Controller) 
and create backup:
 
```sh
# cinder backup-create 5c414913-e0c1-48f3-80c0-7b7fc3a9aef4
+-----------+--------------------------------------+
|  Property |                Value                 |
+-----------+--------------------------------------+
|     id    | 92407f10-26e3-4ffb-a55a-35671bbe5a25 |
|    name   |                 None                 |
| volume_id | 5c414913-e0c1-48f3-80c0-7b7fc3a9aef4 |
+-----------+--------------------------------------+
```

or

3.2) Create backup "on-line" (without detaching and stopping instance) with '--force' attribute (new in Liberty):

```sh
# cinder backup-create 5c414913-e0c1-48f3-80c0-7b7fc3a9aef4 --force
+-----------+--------------------------------------+
|  Property |                Value                 |
+-----------+--------------------------------------+
|     id    | 92407f10-26e3-4ffb-a55a-35671bbe5a25 |
|    name   |                 None                 |
| volume_id | 5c414913-e0c1-48f3-80c0-7b7fc3a9aef4 |
+-----------+--------------------------------------+
``` 

4) List Cinder backups

```sh
# cinder backup-list
+--------------------------------------+--------------------------------------+-----------+------+------+--------------+--------------------------------------------+
|                  ID                  |              Volume ID               |   Status  | Name | Size | Object Count |                 Container                  |
+--------------------------------------+--------------------------------------+-----------+------+------+--------------+--------------------------------------------+
| 080db4a8-cd97-4009-b5f1-4667ae45d2cf | 5c414913-e0c1-48f3-80c0-7b7fc3a9aef4 | available |  -   |  10  |      7       | 08/0d/080db4a8-cd97-4009-b5f1-4667ae45d2cf |
+--------------------------------------+--------------------------------------+-----------+------+------+--------------+--------------------------------------------+
```

If you see in column Container “volumebackups” (as below) and not some id then something is wrong and cinder makes backup to swift (default store). Look to logs on cinder-volume-node (/var/log/cinder-all.log)
```
# cinder backup-list
+--------------------------------------+--------------------------------------+----------+------+------+--------------+---------------+
|                  ID                  |              Volume ID               |  Status  | Name | Size | Object Count |   Container   |
+--------------------------------------+--------------------------------------+----------+------+------+--------------+---------------+
| 92407f10-26e3-4ffb-a55a-35671bbe5a25 | 5c414913-e0c1-48f3-80c0-7b7fc3a9aef4 | creating |  -   |  10  |      0       | volumebackups |
+--------------------------------------+--------------------------------------+----------+------+------+--------------+---------------+
```

5) On Cinder-node in mounted NFS-folder or on NFS-server in shared NFS-folder check files:

```sh
# ls -al

total 282084
drwxrwx--- 2 cinder cinder      4096 Jul 28 12:30 .
drwxr-xr-x 3 cinder cinder      4096 Jul 28 12:26 ..
-rw-rw---- 1 cinder cinder 240314074 Jul 28 12:27 backup-00001
-rw-rw---- 1 cinder cinder  13549209 Jul 28 12:28 backup-00002
-rw-rw---- 1 cinder cinder   1952899 Jul 28 12:28 backup-00003
-rw-rw---- 1 cinder cinder   6404295 Jul 28 12:29 backup-00004
-rw-rw---- 1 cinder cinder   1974148 Jul 28 12:30 backup-00005
-rw-rw---- 1 cinder cinder    716775 Jul 28 12:30 backup-00006
-rw-rw---- 1 cinder cinder      2925 Jul 28 12:30 backup_metadata
-rw-rw---- 1 cinder cinder  23920920 Jul 28 12:30 backup_sha256file
```

6) Make incremental backup

On Controller-node:
```sh
# cinder backup-create 5c414913-e0c1-48f3-80c0-7b7fc3a9aef4 --incremental --force
```

On Cinder-node in mounted NFS-folder or on NFS-server in shared NFS-folder check files of increment backup:
```sh
# ls -al
total 23376
drwxrwx--- 2 cinder cinder     4096 Jul 28 13:50 .
drwxr-xr-x 3 cinder cinder     4096 Jul 28 13:49 ..
-rw-rw---- 1 cinder cinder     1828 Jul 28 13:50 backup_metadata
-rw-rw---- 1 cinder cinder 23920920 Jul 28 13:50 backup_sha256file
```

See that this backup smaller than full backup in 4th step. 

You can't restore incremental backup without full backup and intermediate incremental backups.


---

#### Backup Virtual Machine with Cinder Volume as root-disk via cinder-snapshot  

1) List volumes
```sh
# cinder list
```

2) Make snapshot with –force
```sh
# cinder snapshot-create 5c414913-e0c1-48f3-80c0-7b7fc3a9aef4 --force

+-------------+--------------------------------------+
|   Property  |                Value                 |
+-------------+--------------------------------------+
|  created_at |      2016-07-29T08:57:28.270428      |
| description |                 None                 |
|      id     | efafabf6-4bf8-4107-9593-a8c3b910bb4e |
|   metadata  |                  {}                  |
|     name    |                 None                 |
|     size    |                  10                  |
|    status   |               creating               |
|  volume_id  | 5c414913-e0c1-48f3-80c0-7b7fc3a9aef4 |
+-------------+--------------------------------------+
```

Snapshot-create (takes seconds) more faster backup-create (takes minutes)

3) List volume snapshots
```sh
# cinder snapshot-list                                               

+--------------------------------------+--------------------------------------+-----------+------+------+
|                  ID                  |              Volume ID               |   Status  | Name | Size |
+--------------------------------------+--------------------------------------+-----------+------+------+
| efafabf6-4bf8-4107-9593-a8c3b910bb4e | 5c414913-e0c1-48f3-80c0-7b7fc3a9aef4 | available |  -   |  10  |
+--------------------------------------+--------------------------------------+-----------+------+------+
```

4) Create the temporary volume using the snapshot. (For backup purpose)
```sh
# cinder create --snapshot-id efafabf6-4bf8-4107-9593-a8c3b910bb4e  --display-name backup_vol_from_snapshot

+---------------------------------------+--------------------------------------+
|                Property               |                Value                 |
+---------------------------------------+--------------------------------------+
|              attachments              |                  []                  |
|           availability_zone           |                 nova                 |
|                bootable               |                false                 |
|          consistencygroup_id          |                 None                 |
|               created_at              |      2016-07-29T09:03:11.000000      |
|              description              |                 None                 |
|               encrypted               |                False                 |
|                   id                  | 3baa8909-ac28-404f-bfca-26a982415772 |
|                metadata               |                  {}                  |
|            migration_status           |                 None                 |
|              multiattach              |                False                 |
|                  name                 |       backup_vol_from_snapshot       |
|         os-vol-host-attr:host         |   node-13.virtbox.com#LVM-backend    |
|     os-vol-mig-status-attr:migstat    |                 None                 |
|     os-vol-mig-status-attr:name_id    |                 None                 |
|      os-vol-tenant-attr:tenant_id     |   76ce6a4fe6174c3e9f40a61d04b7936e   |
|   os-volume-replication:driver_data   |                 None                 |
| os-volume-replication:extended_status |                 None                 |
|           replication_status          |               disabled               |
|                  size                 |                  10                  |
|              snapshot_id              | efafabf6-4bf8-4107-9593-a8c3b910bb4e |
|              source_volid             |                 None                 |
|                 status                |               creating               |
|                user_id                |   44b09d31aa5d419cb42e9fb0d8befed7   |
|              volume_type              |                 None                 |
+---------------------------------------+--------------------------------------+
```

5) List the cinder volume. You should be able to see the new volume here.
```sh
# cinder list

+--------------------------------------+------------------+------------------+--------------------------------------+------+-------------+----------+--------------+
|                  ID                  |      Status      | Migration Status |                    Name              | Size | Volume Type | Bootable | Multiattach  |       Attached to      
+--------------------------------------+------------------+------------------+--------------------------------------+------+-------------+----------+--------------+
| 3baa8909-ac28-404f-bfca-26a982415772 |     creating     |        -         |   backup_vol_from_snapshot           |  10  |      -      |  false   |    False     |        
+--------------------------------------+------------------+------------------+--------------------------------------+------+-------------+----------+--------------+
```

6) Initiate the cinder backup for newly created volume
```sh
# cinder backup-create 3baa8909-ac28-404f-bfca-26a982415772

+-----------+--------------------------------------+
|  Property |                Value                 |
+-----------+--------------------------------------+
|     id    | 258dfb6a-2e50-4015-87ed-6ce4fb0c028d |
|    name   |                 None                 |
| volume_id | 3baa8909-ac28-404f-bfca-26a982415772 |
+-----------+--------------------------------------+
```

7) List the backup files
```sh
# cinder backup-list  
| 258dfb6a-2e56...ce4fb0c028d          | 3baa8909-ac...26a982415772 | available |  -   |  10  |      7       | 25/8d/258dfb6a-2e50-4015-87ed-6ce4fb0c028d |
```

8) Delete temporary volume (created from snapshot)
```sh
# cinder delete 3baa8909-ac28-404f-bfca-26a982415772
```

---

### Restore Virtual Machine with Cinder Volume as root-disk

On Controller-node:

1) List backups
```sh
# cinder backup-list
```

2.1) Restore to new volume
```sh
# cinder backup-restore 166e2f4b-09ff-4f85-ac2f-37964a8c3fee
+-------------+-----------------------------------------------------+
|   Property  |                        Value                        |
+-------------+-----------------------------------------------------+
|  backup_id  |         166e2f4b-09ff-4f85-ac2f-37964a8c3fee        |
|  volume_id  |         aa414299-22aa-4243-b0f1-7111e11adc47        |
| volume_name | restore_backup_166e2f4b-09ff-4f85-ac2f-37964a8c3fee |
+-------------+-----------------------------------------------------+
```

2.2) Restores to existing volume:
```sh
# cinder backup-restore 6023e6c7-735c-45a4-a6bf-70c31dcf5b74 --volume 5c414913-e0c1-48f3-80c0-7b7fc3a9aef4
```

3) List volumes
```sh
# cinder list
```

For more information about volume
```sh
cinder show <vol-id>

# cinder show 988ffaad-40f3-4638-9db6-417eb96b54f2
```

4) Create the instance using the restored volume.
```sh
# nova boot --flavor m1.small  --block-device source=volume,id="75fe95d4-9b96-4e56-9267-56145bb0edde",dest=volume,shutdown=preserve,bootindex=0 --nic net-id="dd2c74c0-efc8-47f3-8366-5b951a8a28dc" restored_from_snapshot_vol

+--------------------------------------+--------------------------------------------------+
| Property                             | Value                                            |
+--------------------------------------+--------------------------------------------------+
| OS-DCF:diskConfig                    | MANUAL                                           |
| OS-EXT-AZ:availability_zone          |                                                  |
| OS-EXT-SRV-ATTR:host                 | -                                                |
| OS-EXT-SRV-ATTR:hypervisor_hostname  | -                                                |
| OS-EXT-SRV-ATTR:instance_name        | instance-00000005                                |
| OS-EXT-STS:power_state               | 0                                                |
| OS-EXT-STS:task_state                | scheduling                                       |
| OS-EXT-STS:vm_state                  | building                                         |
| OS-SRV-USG:launched_at               | -                                                |
| OS-SRV-USG:terminated_at             | -                                                |
| accessIPv4                           |                                                  |
| accessIPv6                           |                                                  |
| adminPass                            | ZNaTB5N6zGqE                                     |
| config_drive                         |                                                  |
| created                              | 2016-07-29T09:23:58Z                             |
| flavor                               | m1.small (2)                                     |
| hostId                               |                                                  |
| id                                   | d0d775bf-7751-4670-9152-d9bd31247333             |
| image                                | Attempt to boot from volume - no image supplied  |
| key_name                             | -                                                |
| metadata                             | {}                                               |
| name                                 | restored_from_snapshot_vol                       |
| os-extended-volumes:volumes_attached | [{"id": "75fe95d4-9b96-4e56-9267-56145bb0edde"}] |
| progress                             | 0                                                |
| security_groups                      | default                                          |
| status                               | BUILD                                            |
| tenant_id                            | 76ce6a4fe6174c3e9f40a61d04b7936e                 |
| updated                              | 2016-07-29T09:23:58Z                             |
| user_id                              | 44b09d31aa5d419cb42e9fb0d8befed7                 |
+--------------------------------------+--------------------------------------------------+
```

5) Check nova
```sh
# nova list

+--------------------------------------+----------------------------+--------+------------+-------------+----------------------------------------------+
| ID                                   | Name                       | Status | Task State | Power State | Networks                                     |
+--------------------------------------+----------------------------+--------+------------+-------------+----------------------------------------------+
| d0d775bf-7751-4670-9152-d9bd31247333 | restored_from_snapshot_vol | ACTIVE | -          | Running     | admin_internal_net=10.0.111.5, 172.30.88.128 |
+--------------------------------------+----------------------------+--------+------------+-------------+----------------------------------------------+
```

---

## Backup and Restore Cinder volume as attached disk

Tested on MOS 8 (OpenStack Liberty)

---

### Backup Cinder volume as attached disk

Do same actions as at [Backup VM with cinder volume as root disk](/knowledge-base/backup-restore/#backup-virtual-machine-with-cinder-volume-as-root-disk)

---

### Restore Cinder volume as attached disk

Do same actions as at [Restore VM with cinder volume as root disk](/knowledge-base/backup-restore/#restore-virtual-machine-with-cinder-volume-as-root-disk)

One difference: you shouldn't create new instance from volume, only attach new restored volume.  

---

## Move volume to another Openstack

---

### Move volume to another OpenStack via one shared NFS

1) Configure both Openstacks to one NFS-share.

[See how to setup NFS](/knowledge-base/cinder/#cinder-block-storage-service)

2) In 1st Openstack on Controller:
```sh
# cinder backup-list

+--------------------------------------+--------------------------------------+-----------+------+------+--------------+--------------------------------------------+
|                  ID                  |         Volume ID                    |   Status  | Name | Size | Object Count |                 Container                  |
+--------------------------------------+--------------------------------------+-----------+------+------+--------------+--------------------------------------------+
| 60661002-...62f8cdbec0               | 75fe95d4-9b9...6145bb0edde           | available |  -   |  10  |      7       | 60/66/60661002-1668-49dc-97ac-7d62f8cdbec0 |
+--------------------------------------+--------------------------------------+-----------+------+------+--------------+--------------------------------------------+
```

3) Export meta-data of volume's backup:
```sh
# cinder backup-export 60661002-1668-49dc-97ac-7d62f8cdbec0

+----------------+------------------------------------------------------------------------------+
|    Property    |                                    Value                                     |
+----------------+------------------------------------------------------------------------------+
| backup_service |                          cinder.backup.drivers.nfs                           |
|   backup_url   | eyJzdGF0dXMiOiAiYXZhaWxhYmxlIiwgImRpc3BsYXlfbmFtZSI6IG51bGwsICJhdmFpbGFiaWxp |
|                | dHlfem9uZSI6ICJub3ZhIiwgImRlbGV0ZWQiOiBmYWxzZSwgInVwZGF0ZWRfYXQiOiAiMjAxNi0w |
|             	 | Ny0yOVQxMDowNDoxOVoiLCAiaG9zdCI6ICJub2RlLTEzLnZpcnRib3guY29tIiwgInZvbHVtZV9p |
|                | ZCI6ICI3NWZlOTVkNC05Yjk2LTRlNTYtOTI2Ny01NjE0NWJiMGVkZGUiLCAiY29udGFpbmVyIjog |
|	             | IjYwLzY2LzYwNjYxMDAyLTE2NjgtNDlkYy05N2FjLTdkNjJmOGNkYmVjMCIsICJzZXJ2aWNlX21l |
|                | dGFkYXRhIjogImJhY2t1cCIsICJpZCI6ICI2MDY2MTAwMi0xNjY4LTQ5ZGMtOTdhYy03ZDYyZjhj |
|                | ZGJlYzAiLCAic2l6ZSI6IDEwLCAib2JqZWN0X2NvdW50IjogNywgInByb2plY3RfaWQiOiAiNzZj |
|                | ZTZhNGZlNjE3NGMzZTlmNDBhNjFkMDRiNzkzNmUiLCAiZGVsZXRlZF9hdCI6IG51bGwsICJ1c2Vy |
|                | X2lkIjogIjQ0YjA5ZDMxYWE1ZDQxOWNiNDJlOWZiMGQ4YmVmZWQ3IiwgInNlcnZpY2UiOiAiY2lu |
|                | ZGVyLmJhY2t1cC5kcml2ZXJzLm5mcyIsICJkcml2ZXJfaW5mbyI6IHt9LCAiY3JlYXRlZF9hdCI6 |
|                | ICIyMDE2LTA3LTI5VDA5OjQ5OjIxWiIsICJkaXNwbGF5X2Rlc2NyaXB0aW9uIjogbnVsbCwgInBh |
|                | cmVudF9pZCI6IG51bGwsICJudW1fZGVwZW5kZW50X2JhY2t1cHMiOiAwLCAiZmFpbF9yZWFzb24i |
|                | OiBudWxsLCAidGVtcF9zbmFwc2hvdF9pZCI6IG51bGwsICJ0ZW1wX3ZvbHVtZV9pZCI6IG51bGx9 |
|                |                                                                              |
+----------------+------------------------------------------------------------------------------+
```

or
 
```sh
# cinder backup-export 60661002-1668-49dc-97ac-7d62f8cdbec0 | sed -n '/backup_url/,$ s/|.*|  *\(.*\) |/\1/p'
```

and get only  backup_url in one line.

That giant block of text labeled backup_url is not, in fact, a URL. In this case, the actual content is a base64 encoded JSON string. You will need to copy the base64 data to your target OpenStack environment.

Decode the string:
{"status": "available", "display_name": null, "availability_zone": "nova", "deleted": false, "updated_at": "2016-07-29T10:04:19Z", "host": "node-13.virtbox.com", "volume_id": "75fe95d4-9b96-4e56-9267-56145bb0edde", "container": "60/66/60661002-1668-49dc-97ac-7d62f8cdbec0", "service_metadata": "backup", "id": "60661002-1668-49dc-97ac-7d62f8cdbec0", "size": 10, "object_count": 7, "project_id": "76ce6a4fe6174c3e9f40a61d04b7936e", "deleted_at": null, "user_id": "44b09d31aa5d419cb42e9fb0d8befed7", "service": "cinder.backup.drivers.nfs", "driver_info": {}, "created_at": "2016-07-29T09:49:21Z", "display_description": null, "parent_id": null, "num_dependent_backups": 0, "fail_reason": null, "temp_snapshot_id": null, "temp_volume_id": null}

4) In 2nd Openstack

On Controller-node

Import meta-data of volume's backup:
```sh
# cinder backup-import cinder.backup.drivers.nfs eyJzdGF0dXMiOiAiYXZhaWxhYmxlIiwgImRpc3BsYXlfbmFtZSI6IG51bGwsICJhdmFpbGFiaWxpdHlfem9uZSI6ICJub3ZhIiwgImRlbGV0ZWQiOiBmYWxzZSwgInVwZGF0ZWRfYXQiOiAiMjAxNi0wNy0yOVQxMDowNDoxOVoiLCAiaG9zdCI6ICJub2RlLTEzLnZpcnRib3guY29tIiwgInZvbHVtZV9pZCI6ICI3NWZlOTVkNC05Yjk2LTRlNTYtOTI2Ny01NjE0NWJiMGVkZGUiLCAiY29udGFpbmVyIjogIjYwLzY2LzYwNjYxMDAyLTE2NjgtNDlkYy05N2FjLTdkNjJmOGNkYmVjMCIsICJzZXJ2aWNlX21ldGFkYXRhIjogImJhY2t1cCIsICJpZCI6ICI2MDY2MTAwMi0xNjY4LTQ5ZGMtOTdhYy03ZDYyZjhjZGJlYzAiLCAic2l6ZSI6IDEwLCAib2JqZWN0X2NvdW50IjogNywgInByb2plY3RfaWQiOiAiNzZjZTZhNGZlNjE3NGMzZTlmNDBhNjFkMDRiNzkzNmUiLCAiZGVsZXRlZF9hdCI6IG51bGwsICJ1c2VyX2lkIjogIjQ0YjA5ZDMxYWE1ZDQxOWNiNDJlOWZiMGQ4YmVmZWQ3IiwgInNlcnZpY2UiOiAiY2luZGVyLmJhY2t1cC5kcml2ZXJzLm5mcyIsICJkcml2ZXJfaW5mbyI6IHt9LCAiY3JlYXRlZF9hdCI6ICIyMDE2LTA3LTI5VDA5OjQ5OjIxWiIsICJkaXNwbGF5X2Rlc2NyaXB0aW9uIjogbnVsbCwgInBhcmVudF9pZCI6IG51bGwsICJudW1fZGVwZW5kZW50X2JhY2t1cHMiOiAwLCAiZmFpbF9yZWFzb24iOiBudWxsLCAidGVtcF9zbmFwc2hvdF9pZCI6IG51bGwsICJ0ZW1wX3ZvbHVtZV9pZCI6IG51bGx9

+----------+--------------------------------------+
| Property |                Value                 |
+----------+--------------------------------------+
|    id    | 60661002-1668-49dc-97ac-7d62f8cdbec0 |
|   name   |                 None                 |
+----------+--------------------------------------+
```

5) Check backups:
```sh
# cinder backup-list

+--------------------------------------+--------------------------------------+-----------+------+------+--------------+--------------------------------------------+
|                  ID                  |              Volume ID               |   Status  | Name | Size | Object Count |                 Container                  |
+--------------------------------------+--------------------------------------+-----------+------+------+--------------+--------------------------------------------+
| 60661002-166...7d62f8cdbec0          | 75fe95d4-9...5bb0edde                | available |  -   |  10  |      7       | 60/66/60661002-1668-49dc-97ac-7d62f8cdbec0 |
+--------------------------------------+--------------------------------------+-----------+------+------+--------------+--------------------------------------------+
```

We see backup id from 1st Openstack.

6) Restore this backup
```sh
# cinder backup-restore 60661002-1668-49dc-97ac-7d62f8cdbec0
```

7) Create VM and check it. Success!

---

### Move volume to another OpenStack via Glance images

1) Create image from volume (on Controller host):
```
# cinder upload-to-image 3baa8909-ac28-404f-bfca-26a982415772 omg-from-vol

+---------------------+--------------------------------------+
|       Property      |                Value                 |
+---------------------+--------------------------------------+
|   container_format  |                 bare                 |
|     disk_format     |                 raw                  |
| display_description |                 None                 |
|          id         | 3baa8909-ac28-404f-bfca-26a982415772 |
|       image_id      | b975e19f-6f68-4a36-9511-51658c9617d1 |
|      image_name     |             omg-from-vol             |
|         size        |                  10                  |
|        status       |              uploading               |
|      updated_at     |      2016-07-29T09:13:03.000000      |
|     volume_type     |                 None                 |
+---------------------+--------------------------------------+
```

2) List images
```sh
# glance image-list

+--------------------------------------+--------------+
| ID                                   | Name         |
+--------------------------------------+--------------+
| b975e19f-6f68-4a36-9511-51658c9617d1 | omg-from-vol |
```

3) Export image (as a file)
```sh
# glance image-download --file img-from-vol.raw b975e19f-6f68-4a36-9511-51658c9617d1
```

4) Import in another Openstack
```sh
# glance image-create --file img-from-vol.raw --name img-from-vol --container-format bare --disk-format qcow2 --is-public True
```
