# Rally - Verify OpenStack with Rally via Ansible

[TOC]

---

## Rally Verify Overview

Rally can verify OpenStack cloud with Tempest.
Tempest - The OpenStack Integration Test Suite. This is a set of integration tests to be run against a live OpenStack cluster. 
Tempest has batteries of tests for OpenStack API validation, Scenarios, and other specific tests useful in validating an OpenStack deployment.

Links:

* <http://rally.readthedocs.io/en/latest/tutorial/step_10_verifying_cloud_via_tempest.html>
* <http://docs.openstack.org/developer/tempest/>
* <https://github.com/openstack/tempest>
* <https://refstack.openstack.org/>

---

## Requirements

#### Requirements for using Rally

* Management-server with installed kvm virtualization
* installed Fuel-server as virtual machine in Management-server
* ready OpenStack deployment (OpenStack >= Kilo)
* access from Fuel to specific OpenStack Management LAN
* access from Fuel to internet
* ssh access to Fuel-server with root permissions
* "operator machine"
* installed Rally >= 0.4.0

#### Requirements for "operator machine"

* git
* ansible>=2.0
* downloaded git-repository "cml-vbx" (<https://git.cmlatitude.com/cml-vbx.git>)
* ssh root access to Management-server
* ssh root access to Fuel-server

---

## Rally Verification OpenStack

1. Change directory to ansible project

        $ cd cml-vbx/ansible-projects/openstack-deployment/

2. Define in inventory-file (for example, hosts) 4 variables in group `[all:vars]`:

    * ip-address of Management-server where Fuel-server is running as virtual machine

            ansible_ssh_host=some-ip

    * SSH root password for Management-server or ssh-key  

            ansible_ssh_pass=some-seсure-password

        or  

            ansible_ssh_private_key_file=id_rsa.pub

    * SSH user  

            user=ubuntu

    * prefix of Fuel virtual machine name  

            prefix='some-prefix'

    * Horizon domain name (public API) for current OpenStack deployment    

            openstack_hostname=public.domain.com
      
    and 1 variable in group `[fuel-env:vars]`:  
 
    * Openstack deployment name in Rally  

            env_name='some-env-name'

    For example:

            [all:vars]
            ansible_ssh_host=192.168.0.100
            ansible_ssh_private_key_file=/home/user/.ssh/id_rsa.pub
            user=root
            prefix=demo-fuel
            openstack_hostname=public.company.com
            [fuel-env]
            fuel-env
            [fuel-env:vars]
            env_name='demo-openstack'

3. Define variables for Rally verification

    __Default role settings:__   
    _(in file ansible-projects/openstack-deployment/roles/rally-verify/defaults/main.yml)_

    * URL to git repository with Tempest

            tempest_source: 'https://github.com/openstack/tempest.git'

    * Tempest test name

            tests_name: '2015.07.required'

    * Tempest tests URL

            tempest_tests_url: 'https://refstack.openstack.org/api/v1/guidelines/2015.07/tests?target=platform&type=required&alias=true&flag=false'

    * Directory on local machine (ansible server) for results from remote rally machine

            local_result_file: /tmp/

    * Directory in remote rally machine for results 

            rally_dir: '/root/rally/{{ deployment_name }}'

    * Timestamp for results file name   

            timestamp: '{{ ansible_date_time.date }}_{{ ansible_date_time.hour }}-{{ ansible_date_time.minute }}-{{ ansible_date_time.second }}

    * __Define variables in 2 ways:__

        * add needed variables in inventory file in part `[rally:vars]` for overriding default settings

            * Tempest test name    
                `tests_name`

            * Tempest tests URL  
                `tempest_tests_url`

            * directory on "operator machine" for results     
                `local_result_file`
  
        For example:

            tests_name='2015.07.required'
            tempest_tests_url='https://refstack.openstack.org/api/v1/guidelines/2015.07/tests?target=platform&type=required&alias=true&flag=false'
            local_results_file='/var/www/rally/'

    * in line with command `ansible-playbook ...` for overriding default settings

        For example:
    
            $ ansible-playbook -i hosts set-env-facts.yml rally-verify.yml -e '{ tests_name: '2015.07.required', tempest_tests_url: 'https://refstack.openstack.org/api/v1/guidelines/2015.07/tests?target=platform&type=required&alias=true&flag=false', local_results_file: '/var/www/rally/' }' -vvv

__Result__: [report benchmark network](/virtbox-docs/rally-reports/openstack_2015.07.required_results.html)

---
