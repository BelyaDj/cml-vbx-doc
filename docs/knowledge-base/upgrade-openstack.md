# Upgrade MOS 4.1 (Havana) to MOS 5.0 (Icehouse)
<https://www.mirantis.com/blog/yes-can-upgrade-openstack-heres/>

[TOC]

---

The general methodology

Every service has its own idiosyncrasies, but in general upgrades follow this pattern:

* Stop the service.
* Create a database dump for the source cloud.
* Replace the config files on the target cloud with a copy of the config files from the source cloud.
* Replace the target database for the service with the dump from the source cloud.
* If necessary, upgrade the structure of the database to match the target cloud’s release level.  (Fortunately, this can be done easily via provided scripts.)
* Adjust permissions on the relevant folders.
* Restart the service.

Performing the migration
As a means of testing our new process, we updated a cloud running Mirantis OpenStack-4.1 (Havana) to Mirantis OpenStack-5.0 (Icehouse) by completely replacing the cloud controller.  (The process would be the same to move to later versions of OpenStack as well.)

Setting up

Don’t forget to make sure that the credentials for the database and AMQP services on the new controller are the same as those for the old controller so the compute nodes can interact with them correctly.
We started by deploying two clouds, one using OpenStack Havana, and one using OpenStack Icehouse, in the same configuration, using FUEL for deployment. Our clouds have the same settings for data storage, file based storage for OpenStack Glance, and iSCSI over LVM for OpenStack Cinder. We also used shared networks (management, storage, internal) for both clouds.

Before starting the upgrade procedure, we made backups of the Havana configuration files and databases, as well as the openrc file, but we don’t want any of the image files to have their contents changed while we’re copying so we need to disable any activity in the source cloud.

In order to avoid undesirable activity we should turn off Api Service on the Controller Node of source cloud.
```
# service openstack-nova-api stop
```

Next, create directories for the backup config files:
```
# for i in keystone glance nova cinder neutron openstack-dashboard; \
do mkdir $i-havana; \
done
```

Then do the actual backup:
```
# for i in keystone glance nova cinder neutron openstack-dashboard; \
do cp -r /etc/$i/* $i-havana/; \
done
```

Finally, copy the storage data from the Havana installation to the Icehouse installation.  In our case, that consists of Glace image files and Cinder volume data on LVM.
```
# scp -r /var/lib/glance <user_name>@<ip_of_Icehouse_controller>:/var/lib/
```

At this point you’ve finished the preparation phase and you’re ready to do the actual switchover.  

---

#### Upgrading Keystone

Start by creating the Keystone database dump on the source cloud:
```
# mysqldump -u keystone -pfDm3oOLv keystone > havana-keytone-db-backup.sql
```
Copy the output file to the Icehouse controller.

Next you’ll need to replace the keystone configuration files on the Icehouse controller with the configuration from the Havana installation.  Start by stopping the keystone process:
```
# service openstack-keystone stop
```
On the target server, change  /etc/keystone/keystone.conf so that the credentials match what the database on Icehouse will expect.  Once you’ve done that, drop existing keystone database and create the new one with the needed permissions for the keystone user:
```
# mysql> DROP DATABASE keystone;
# mysql> CREATE DATABASE keystone;
```
Import the exported Keystone database:
```
# mysql -uroot keystone < backup-havana/havana-keystone-db-backup.sql
```
Next, we can use the keystone-manage tool function, which enables us to upgrade the database structure to Icehouse:
```
# keystone-manage db_sync
```
Finally, change permissions on the keystone folder:
```
# chown -R keystone:keystone /etc/keystone/
```
Now start the service and make sure everything works:
```
# service openstack-keystone start
# source openrc.havana
```
You should get a list of endpoints:
```
# keystone endpoint-list
.
+----------------------------------+-----------+-------------------------------------------+---------------------------------------+---------------------------------------+----------------------------------+
|                id                |   region  |                 publicurl                 |              internalurl              |                adminurl               |            service_id            |
+----------------------------------+-----------+-------------------------------------------+---------------------------------------+---------------------------------------+----------------------------------+
| 199949f2481242fc96b8e50b95e0053e | RegionOne |          http://172.16.40.43:9696         |          http://10.0.0.4:9696         |          http://10.0.0.4:9696         | c2c6fc9a8b7f4feeaeb970069174dacc |
| 2e2b3f5c1906465f8e269644e73a1268 | RegionOne |          http://172.16.40.43:9292         |          http://10.0.0.4:9292         |          http://10.0.0.4:9292         | eca3144671534bb1be469e9acac22d60 |
| 33a1717ea672454ead5daf2aa3a05df6 | RegionOne |  http://172.16.40.43:8773/services/Cloud  |  http://10.0.0.4:8773/services/Cloud  |  http://10.0.0.4:8773/services/Admin  | c16c6ae8b9e445c197b66aaa4566a039 |
| 5869dee417c94c39865ab44acbbb021d | RegionOne | http://172.16.40.43:8004/v1/%(tenant_id)s | http://10.0.0.4:8004/v1/%(tenant_id)s | http://10.0.0.4:8004/v1/%(tenant_id)s | ca4a8d13471144bcbc7c4920b55890e3 |
| 8cb1c8216990434eb46e8180faf522bd | RegionOne |       http://172.16.40.43:5000/v2.0       |       http://10.0.0.4:5000/v2.0       |       http://10.0.0.4:35357/v2.0      | 43e1d90586df44f399224c67ea6e9b97 |
| 9a8c4808244a48feb6f6df739b1cfcaa | RegionOne | http://172.16.40.43:8776/v1/%(tenant_id)s | http://10.0.0.4:8776/v1/%(tenant_id)s | http://10.0.0.4:8776/v1/%(tenant_id)s | 21bcde17c33347d1bb00bcbd8b84b447 |
| eb4adb395e72482f8447d50103299a6d | RegionOne | http://172.16.40.43:8774/v2/%(tenant_id)s | http://10.0.0.4:8774/v2/%(tenant_id)s | http://10.0.0.4:8774/v2/%(tenant_id)s | 438d65e654af44fb9c047cd661a65a14 |
+----------------------------------+-----------+-------------------------------------------+---------------------------------------+---------------------------------------+----------------------------------+
```

---

#### Upgrading Glance

The approach for other services will be similar to that for Keystone. Configuration files should be changed on the new controller, and databases from the old controller should be converted and imported.

First stop all glance services:
```
for i in /etc/init.d/openstack-glance-*; do $i stop; done
```

Drop the glance database on the new controller, create a new one and import the glance database from the Havana cluster:
```
mysql -uroot glance  < backup-havana/havana-glance-db-backup.sql
```

Make sure to convert the character set for each table to UTF-8:
```
# mysql -u root -p
mysql> SET foreign_key_checks = 0;
mysql> ALTER TABLE glance.image_locations CONVERT TO CHARACTER SET 'utf8';
mysql> ALTER TABLE glance.image_members CONVERT TO CHARACTER SET 'utf8';
mysql> ALTER TABLE glance.image_properties CONVERT TO CHARACTER SET 'utf8';
mysql> ALTER TABLE glance.image_tags CONVERT TO CHARACTER SET 'utf8';
mysql> ALTER TABLE glance.images CONVERT TO CHARACTER SET 'utf8';
mysql> ALTER TABLE glance.migrate_version CONVERT TO CHARACTER SET 'utf8';
mysql> SET foreign_key_checks = 1;
mysql> exit
```

Update the glance database:
```
glance-manage db_sync
```

Replace the glance configuration files on the Icehouse controller with the configuration from the Havana controller:
```
# cp -f backup-havana/glance-havana/* /etc/glance/
```

Change the credentials in /etc/glance/glance-api.conf to the database and RabbitMQ to match the expected values:
```
sql_connection=mysql://glance:1vCYsATB@127.0.0.1/glance?read_timeout=60
rabbit_use_ssl = False
rabbit_userid = nova
rabbit_password = ygrfyaNZ
```
And then in /etc/glance/glance-registry.conf:
```
sql_connection=mysql://glance:1vCYsATB@127.0.0.1/glance?read_timeout=60
```

Change the permissions for the /etc/glance directory:
```
# chown -R glance:glance /etc/glance
```

Copy the glance image-cache and images files from the Havana OpenStack cluster to the Icehouse OpenStack cluster, to the actual directories. In our case we copied the content from /var/lib/glance to the new server in the same directory and changed the permissions:
```
# chown -R glance:glance /var/lib/glance
```

Start the service and make sure it works.
```
service openstack-glance-api start
service openstack-glance-registry start
```
```
# glance image-list
+--------------------------------------+--------+-------------+------------------+----------+--------+
| ID                                   | Name   | Disk Format | Container Format | Size     | Status |
+--------------------------------------+--------+-------------+------------------+----------+--------+
| 5bd625a0-cb69-4018-93c8-cf8bf0348f52 | TestVM | qcow2       | bare             | 14811136 | active |
+--------------------------------------+--------+-------------+------------------+----------+--------+
```

---

#### Upgrade Nova

The upgrade approach for the Nova service will be similar, but we need to remember that in this setup, we also have the Neutron service, which handles metadata transferring from the Nova service, so we can’t run the nova-metadata service.

Stop all nova services on the Icehouse cluster:
```
# for i in /etc/init.d/openstack-nova-*;do $i stop; done
```

Drop the nova database, create the new one and import the nova database from the Havana cluster:
```
# mysql -uroot nova < backup-havana/havana-nova-db-backup.sql
```
Replace the nova configuration files on the Icehouse controller with the configuration from the Havana cluster:
```
# cp -f backup-havana/nova-havana/* /etc/nova/
```
Don’t forget to make sure that the credentials for rabbitmq and mysql on the controller and compute nodes in /etc/nova/nova.conf file are correct for the target cloud.

Change the permissions on the /etc/nova directory:
```
# chown -R nova:nova /etc/nova
```

Update the nova database:
```
nova-manage db_sync
```

!!! note
    Since we’re going to connect old Havana compute nodes to the icehouse cluster, please note, that Nova supports a limited live upgrade model for the compute nodes in Icehouse. To do this, upgrade controller infrastructure (everything except nova-compute) first, but set the [upgrade_levels]/compute=icehouse-compat option on compute nodes. This will enable Icehouse controller services to talk to Havana compute services. Upgrades of individual compute nodes can then proceed normally. When all the computes are upgraded, unset the compute version option to retain the default and restart the controller services. Find the following section and key in /etc/nova/nova.conf and make sure the version is set to “icehouse-compat”:

Start the nova services, restart the openstack-nova-compute service on the compute nodes, and test. (Be careful not to start nova-metadata if you use the neutron-metadata-service.)

To make sure Nova is working correctly, use the nova-manage tool:
```
# nova-manage service list
Binary           Host                                 Zone             Status     State Updated_At
nova-compute     node-3.domain.tld                    nova             enabled    :-)   2014-09-03 15:17:31
nova-cert        node-1.domain.tld                    internal         enabled    :-)   2014-09-03 15:17:25
nova-conductor   node-1.domain.tld                    internal         enabled    :-)   2014-09-03 15:17:25
nova-consoleauth node-1.domain.tld                    internal         enabled    :-)   2014-09-03 15:17:25
nova-console     node-1.domain.tld                    internal         enabled    :-)   2014-09-03 15:17:25
nova-scheduler   node-1.domain.tld                    internal         enabled    :-)   2014-09-03 15:17:25
```

Services that are running properly will show a status of “;-)”. Should there be a problem, you’ll see a status of “XXX”.  You can also check the instances list:
```
# nova list
+--------------------------------------+------------+--------+------------+-------------+---------------------+
| ID                                   | Name       | Status | Task State | Power State | Networks            |
+--------------------------------------+------------+--------+------------+-------------+---------------------+
| 75eee200-5961-4e2e-bc9f-b6c276c68554 | instance-1 | ACTIVE | -          | Running     | net-1=192.168.100.2 |
| bc04ec96-21f6-4390-a968-b4a722fa544f | instance-2 | ACTIVE | -          | Running     | net-1=192.168.100.4 |
+--------------------------------------+------------+--------+------------+-------------+---------------------+
```

---

#### Upgrading Neutron

The upgrade approach for Neutron is also similar to other services, but there is a difference in the upgrade sequence. The database adoption process should be done in two phases: one to migrate the source and destination cloud version assignation, and one to migrate the service configuration and plugins list. Because in this particular example, both clusters use Mirantis OpenStack and the same openvswitch plugin, the upgrade process is less complex, but even in the case of the ML2 plugin, the process should be straightforward.

To perform the upgrade, start by stopping all neutron services on the target controller:
```
# for i in /etc/init.d/neutron-*; do $i stop; done
```

Drop the neutron database, create the new one and import the neutron database from the source cluster:
```
# mysql -uroot neutron < backup-havana/havana-neutron-db-backup.sql
```

Replace the neutron configuration files on the target controller with the configuration from the source cluster:
```
# cp -f backup-havana/neutron-havana/* /etc/neutron/
```

In /etc/neutron/neutron.conf, make sure the credentials for the database and AMPQ match the actual values.

Change the permissions for the  /etc/neutron directory:
```
# chown -R neutron:neutron /etc/neutron
```

Update the neutron database:
```
# neutron-db-manage --config-file /etc/neutron/neutron.conf \
--config-file /etc/neutron/plugins/openvswitch/ovs_neutron_plugin.ini stamp havana
# neutron-db-manage --config-file /etc/neutron/neutron.conf \
--config-file /etc/neutron/plugins/openvswitch/ovs_neutron_plugin.ini upgrade icehouse
```

Start the neutron services and verify:
```
# for i in /etc/init.d/neutron-*; do $i start; done
```
```
# neutron net-list
+--------------------------------------+-----------+-------------------------------------------------------+
| id                                   | name      | subnets                                               |
+--------------------------------------+-----------+-------------------------------------------------------+
| 36560f8d-73ee-4f41-8def-8aecf7516417 | net04     | 46009ba8-426b-40ad-bde8-88ad932fb706 192.168.111.0/24 |
| 50b9e0ed-8dd5-4ea8-b4bf-87ef2ca879a5 | net04_ext | b1252005-89f1-49e4-9443-c289602d18ea 172.16.40.32/27  |
| a325ff94-6860-400f-a261-b91979138d35 | net-1     | d4b25a0e-5393-4227-ad76-da00f1a08017 192.168.100.0/24 |
+--------------------------------------+-----------+-------------------------------------------------------+
```

!!! note
    Please note that occasionally, neutron displays a problem with “flopping” agents.  In this case, the neutron agent shows the wrong status; working agents are shown as down, and vice versa.  The OpenStack community is still working on fixing this issue.

---

#### Upgrade Cinder

In our setup, both clouds have the LVM backend for Cinder on the controller, so we need to perform a few extra steps in order to properly complete the data transfer:

Create a new volume on the target controller in advance. This new volume should be the same size as the volume on the source controller.  Transfer the data from the source controller to the destination controller.

Adopt the cinder database. If the iSCSI target for the compute service stays the same (in other words, it has the same IP address), then further database changes are unnecessary. If the IP address changes, however, then you’ll need to correct the database. In this case we need to fix the iSCSI targets so that the instances can save the connection to their attached volumes. In our test, we also changed volumes from the storage network to the management network.

Now you’re ready to move on with the upgrade as usual.

Stop all cinder services on the target cluster:
```
for i in /etc/init.d/openstack-cinder-*; do $i stop; done
```

Drop the cinder database on the target controller, create a new one and import the cinder database from the source cluster:
```
mysql -u root cinder < backup-havana/havana-cinder-db-backup.sql
```

Replace the cinder configuration files on the starget controller with the configuration from the source cluster:
```
# cp -f backup-havana/cinder-havana/* /etc/cinder/
```

In /etc/cinder/cinder.conf, make sure the credentials for the database and AMPQ match the actual credentials.

Change the permissions for the /etc/cinder directory:
```
# chown -R cinder:cinder /etc/cinder
```

Update the cinder database:
```
# cinder-manage db_sync
```

Start the cinder services and verify:
```
# service openstack-cinder-api start
# service openstack-cinder-volume start
# service openstack-cinder-scheduler start
```

Make sure to check that the new controller node and each compute node are connected through the storage network, because this network is used for the iSCSI protocol.

If you have a different IP address for the iSCSI target but volumes are already attached, you will need to change the iscsi_ip_address parameter in the /etc/cinder/cinder.conf file, as well as the connection information in the nova database on the controller node in the block_device_mapping table.

For example, let’s assume that we have mysql on the destination controller node, and the nova database shows an instance with an attached volume (id=f374f730-4adc-40f3-bfd4-67c4ac14ba76). Before the service upgrade we might see:
```
mysql> select volume_id, connection_info from block_device_mapping where volume_id='f374f730-4adc-40f3-bfd4-67c4ac14ba76';
+--------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| volume_id                            | connection_info                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
+--------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| f374f730-4adc-40f3-bfd4-67c4ac14ba76 | {"driver_volume_type": "iscsi", "serial": "f374f730-4adc-40f3-bfd4-67c4ac14ba76", "data": {"access_mode": "rw", "target_discovered": false, "encrypted": false, "qos_specs": null, "target_iqn": "iqn.2010-10.org.openstack:volume-f374f730-4adc-40f3-bfd4-67c4ac14ba76", "target_portal": "192.168.1.4:3260", "volume_id": "f374f730-4adc-40f3-bfd4-67c4ac14ba76", "target_lun": 1, "device_path": "/dev/disk/by-path/ip-192.168.1.4:3260-iscsi-iqn.2010-10.org.openstack:volume-f374f730-4adc-40f3-bfd4-67c4ac14ba76-lun-1", "auth_password": "eNJy8GZUta39wqEiCr23", "auth_username": "3NRooC58UK3YiYRaPyAt", "auth_method": "CHAP"}} |
+--------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)
mysql> update block_device_mapping set connection_info='{"driver_volume_type": "iscsi", "serial": "f374f730-4adc-40f3-bfd4-67c4ac14ba76", "data": {"access_mode": "rw", "target_discovered": false, "encrypted": false, "qos_specs": null, "target_iqn": "iqn.2010-10.org.openstack:volume-f374f730-4adc-40f3-bfd4-67c4ac14ba76", "target_portal": "10.0.0.4:3260", "volume_id": "f374f730-4adc-40f3-bfd4-67c4ac14ba76", "target_lun": 1, "device_path": "/dev/disk/by-path/ip-10.0.0.4:3260-iscsi-iqn.2010-10.org.openstack:volume-f374f730-4adc-40f3-bfd4-67c4ac14ba76-lun-1", "auth_password": "eNJy8GZUta39wqEiCr23", "auth_username": "3NRooC58UK3YiYRaPyAt", "auth_method": "CHAP"}}' where volume_id='f374f730-4adc-40f3-bfd4-67c4ac14ba76';
Query OK, 1 row affected (0.14 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```
So in this case, the iSCSI IP has been changed from 192.168.1.4 to 10.0.0.4 — in other words, from the storage network to the management network.

After you make these changes, restart cinder and the tgtd daemons on the controller node.

To test the volumes, run the Cinder CLI commands:
```
# cinder list
+--------------------------------------+--------+--------------+------+-------------+----------+--------------------------------------+
|                  ID                  | Status | Display Name | Size | Volume Type | Bootable |             Attached to              |
+--------------------------------------+--------+--------------+------+-------------+----------+--------------------------------------+
| 69717c74-2cdd-4ef7-b7dd-a38ce63baa54 | in-use |    vol-2     |  2   |     None    |  false   | bc04ec96-21f6-4390-a968-b4a722fa544f |
| f374f730-4adc-40f3-bfd4-67c4ac14ba76 | in-use |    vol-1     |  1   |     None    |  false   | 75eee200-5961-4e2e-bc9f-b6c276c68554 |
+--------------------------------------+--------+--------------+------+-------------+----------+--------------------------------------+
```

You might also look for information about a particular volume, as in:
```
# cinder show <volume_id>
Upgrade Dashboard
```

---

#### Upgrade the OpenStack Dashboard UI

To adopt and configure Horizon, the first step is to change default the keystone role directive in the /etc/openstack-dashboard/local_settings file:

Make sure that the controller IP address is set correctly assigned for the “OPENSTACK_HOST” variable. It should be changed to actual address for the controller, such as:

- OPENSTACK_HOST = "10.0.0.4"
Finally, restart the service:
```
service httpd restart
```
After the web server restarts, check the dashboard by accessing it through the browser.

---
