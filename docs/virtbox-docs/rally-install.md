# Rally - Install Rally via Ansible

[TOC]

---

## Rally Overview

Rally is a benchmarking tool (Benchmark-as-a-Service) that automates and unifies OpenStack 
cloud verification, benchmarking & profiling. It can be used as a basic tool for
an OpenStack CI/CD system that would continuously improve its SLA, performance and
stability. One Rally installation can work with multiple OpenStack clouds.

__Typical cases where Rally aims to help are__:
     
1. Automate measuring & profiling focused on how new code changes affect the OS performance;    
2. Using Rally profiler to detect scaling & performance issues;     
3. Investigate how different deployments affect the OS performance:
   >* Find the set of suitable OpenStack deployment architectures;       
   >* Create deployment specifications for different loads (amount of controllers, swift nodes, etc.);       
4. Automate the search for hardware best suited for particular OpenStack cloud;     
5. Automate the production cloud specification generation:
   >* Determine terminal loads for basic cloud operations: VM start & stop, Block Device create/destroy & various OpenStack API methods;     
   >* Check performance of basic cloud operations in case of different loads.        

__Links__:

- Rally documentation: <http://rally.readthedocs.io/en/latest/overview.html>
- Rally source code: <https://github.com/openstack/rally>


## Requirements

__Requirements for using Rally__

* Management-server with installed kvm virtualization
* installed Fuel-server as virtual machine in Management-server
* ready OpenStack deployment (OpenStack >= Kilo)
* access from Fuel to specific OpenStack Management LAN
* access from Fuel to internet
* ssh access to Fuel-server with root permissions
* "operator machine"

__Requirements for "operator machine"__

* git
* ansible>=2.0
* downloaded git-repository "cml-vbx" (<https://git.cmlatitude.com/cml-vbx.git>)
* ssh root access to Management-server
* ssh root access to Fuel-server
* web access to Fuel-server (ports 80 and/or 443 depends on Fuel type installation)


## Rally Installation

Rally installs on Fuel-server (must be in specific Openstack Management LAN)

1) Change directory to ansible project
```sh
$ cd cml-vbx/ansible-projects/openstack-deployment/
```
2) Define in inventory-file (for example, hosts) 4 variables in group `[all:vars]`:

 - ip-address of Management-server where Fuel-server is running as virtual machine  
   ```
   ansible_ssh_host=some-ip
   ```
 - SSH root password for Management-server or ssh-key  
   ```
   ansible_ssh_pass=some-seсure-password
   ```  
   or  
   ```
   ansible_ssh_private_key_file=id_rsa.pub
   ```
 - SSH user  
   ```
   user=ubuntu
   ```
 - prefix of Fuel virtual machine name  
   ```
   prefix='some-prefix'
   ```  
   
and 1 variable in group `[fuel-env:vars]`:
  
 - Openstack deployment name in Rally  
   ```
   env_name='some-env-name'
   ```

  For example:
    ```
    [all:vars]
    ansible_ssh_host=192.168.0.100
    ansible_ssh_private_key_file=/home/user/.ssh/id_rsa.pub
    user=root
    prefix=demo-fuel
    [fuel-env]
    fuel-env
    [fuel-env:vars]
    env_name='demo-openstack'
    ```

3) Run ansible-playbook for installation Rally
 Ansible uses additional role "set-env-facts.yml" which define all needed parameters automatically and sends its
 to role "rally-install.yml".
 Order is important in command: 1sf - "set-env-facts.yml", 2nd - "rally-install.yml" 
```sh
$ ansible-playbook -i hosts set-env-facts.yml rally-install.yml
```
If installation fails run ansible-playbook with debug information:
```sh
$ ansible-playbook -i hosts set-env-facts.yml rally-install.yml -vvv
```

This message means that Rally successfully installed (failed=0):
 > PLAY RECAP *********************************************************************  
 fuel-env                   : ok=6    changed=2    unreachable=0    failed=0   
 rally                      : ok=11   changed=9    unreachable=0    failed=0  

To check Rally installation connect to Fuel-server and run 'rally --version':
```sh
# rally --version
0.5.1~dev41
```