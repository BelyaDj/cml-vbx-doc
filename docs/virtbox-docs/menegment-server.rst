Ansible Role "kvm-host"
=======================

[TOC]

---

##### Synopsis

This role installs and configures hypervisor kvm, pool of lvm-disks and networks
  
##### Software Requirements
 
 - Ubuntu 14.04
 - Ansible >= 2.0

##### Input parameters / Inventory Requirements

+------------------+-----------+----------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------+--------------------+
| parameter        | required  | default                                                                                                  | choices                                                               | comments           |
+==================+===========+==========================================================================================================+=======================================================================+====================+
| bridges          | yes       | "[{'name': 'eth0', 'inet': 'manual'}, {'name': 'eth1', 'inet': 'static', 'address': '192.168.0.20/24'}]" | - "[{'name': 'eth0', 'inet': 'manual'}]"                              |                    |
|                  |           |                                                                                                          | - "[{'name': 'eth1', 'inet': 'static', 'address': '192.168.0.20/24'}]"|                    |
+------------------+-----------+----------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------+--------------------+
| interfaces       | yes       |  bright color                                                                                            | yes                                                                   |  bright color      |
| interfaces       | yes       |  bright color                                                                                            | yes                                                                   |  bright color      |
+------------------+-----------+----------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------+--------------------+
| lvm-pools        | yes       |  "[{'name': 'hdd', 'device_path': '/dev/sda'}, {'name': 'ssd', 'device_path': '/dev/sdd'}]"              | - {'name': 'hdd', 'device_path': '/dev/sda'}                          |   cures scurvy     |
|                  |           |                                                                                                          | - {'name': 'ssd', 'device_path': '/dev/sdd'}                          |                    |
|                  |           |                                                                                                          | - {'name': 'hdd15k', 'device_path': '/dev/sdb'}                       |                    |
|                  |           |                                                                                                          | - {'name': 'hdd15kSGL', 'device_path': '/dev/sdc'}                    |                    |
+------------------+-----------+----------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------+--------------------+
| private_networks | yes       |  tasty                                                                                                   | yes                                                                   |   tasty            |
+------------------+-----------+----------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------+--------------------+


##### Outputs


| Some params   | description   | type  | sample   |
|---            |---            |---    |---       |
|               |               |       |          | 



First Header | Second Header | Third Header
------------ | ------------- | ------------
Content Cell | Content Cell  | Content Cell
Content Cell | Content Cell  | Content Cell


| First Header | Second Header | Third Header |
| ------------ | ------------- | ------------ |
| Content Cell | Content Cell  | Content Cell |
| Content Cell | Content Cell  | Content Cell |


First Header | Second Header | Third Header
:----------- | :-----------: | -----------:
Left         | Center        | Right
Left         | Center        | Right


##### RST Tables

+------------------------+------------+----------+----------+
| Header row, column 1   | Header 2   | Header 3 | Header 4 |
| (header rows optional) |            |          |          |
+========================+============+==========+==========+
| body row 1, column 1   | column 2   | column 3 | column 4 |
+------------------------+------------+----------+----------+
| body row 2             | ...        | ...      |          |
+------------------------+------------+----------+----------+

=====  =====  =======
A      B      A and B
=====  =====  =======
False  False  False
True   False  False
False  True   False
True   True   True
=====  =====  =======

+--------------+----------+-----------+-----------+
| row 1, col 1 | column 2 | column 3  | column 4  |
+--------------+----------+-----------+-----------+
| row 2        |  Use the command ``ls | more``.  |
+--------------+----------+-----------+-----------+
| row 3        |          |           |           |
+--------------+----------+-----------+-----------+


##### Markdown to RST table conversion

<!--table-->
| Tables        | Are           | Cool  |
| ------------- |-------------- | ----- |
| col 3 is      | nifty         | $1600 |
| col 2 is      | awesome       |   $12 |
| zebra stripes | are neat      |    $1 |
<!--endtable-->


##### RST table


```eval_rst
.. list-table::
   :widths: 33 33 33
   :header-rows: 1

   * - Tables
     - Are
     - Cool
   * - col 3 is
     - nifty
     - $1600
   * - col 2 is
     - awesome
     - $12
   * - zebra stripes
     - are neat
     - $1

```

---

``` sidebar:: Line numbers and highlights

     emphasis-lines:
       highlights the lines.
     linenos:
       shows the line numbers as well.
     caption:
       shown at the top of the code block.
     name:
       may be referenced with `:ref:` later.
```

``` code-block:: markdown
     :linenos:
     :emphasize-lines: 3,5
     :caption: An example code-block with everything turned on.
     :name: Full code-block example

     # Comment line
     import System
     System.run_emphasis_line
     # Long lines in code blocks create a auto horizontal scrollbar
     System.exit!
```

---


##### Role Dependencies

 - some-parent-role

##### Impact to roles 

 - kvm-nodes
 - fuel
 - fuel-env
 - some-child-role
 
##### Role's steps

```
playbook: kvm-host.yml

  play #1 (kvm-host): kvm-host  TAGS: []
    tasks:
      kvm-host : Checking required variables    TAGS: [check_vars]
      kvm-host : Verify host distribution   TAGS: []
      kvm-host : Update cache   TAGS: []
      kvm-host : Install KVM and utilities packages TAGS: []
      kvm-host : Create '/etc/network/interfaces.d' directory   TAGS: [network]
      kvm-host : Configure interfaces   TAGS: [network]
      kvm-host : Setup public interface '{{ public_interface }}'    TAGS: [network]
      kvm-host : Setup bridge '{{ public_bridge }}' for public network on interface '{{ public_interface }}'    TAGS: [network]
      kvm-host : Bring up '{{ public_bridge }}' TAGS: [network]
      kvm-host : Configure private networks TAGS: [private-network]
      kvm-host : Configure storage pools    TAGS: [storage]
```

##### Examples

- run play-book
```sh
$ ansible-playbook -i hosts kvm-host.yml -vvv
```

- check variables before running
```sh
$ ansible-playbook -i hosts -t check_vars kvm-host.yml -vvv
```
