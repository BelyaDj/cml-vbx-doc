# Rally - Benchmark OpenStack with Rally via Ansible

[TOC]

---

## Rally Benchmark Overview

Benchmarking pre-production and production OpenStack clouds is not a trivial task. 
From the one side it is important to reach the OpenStack cloud’s limits, from the other side 
the cloud shouldn’t be damaged. Rally aims to make this task as simple as possible. 
Rally is able to generate enough load for any OpenStack cloud. 
Rally has huge benchmark library (wolkload scenarios), that can make load on all parts of OpenStack together (Keystone, 
Nova, Neutron, etc) or load only on one component (Heat, for example) and measures performance of all actions separately. 
Rally measures action's results in time (how long a scenario are executing in seconds) and can present its 
in JSON-format or in HTML-page with graphics.
Rally can benchmark OpenStack with constant or growing load and you can set success criteria (also called SLA - Service-Level Agreement) 
for every benchmark. Rally will automatically check them and can stop benchmarking before Cloud fails.

Links:

* <http://rally.readthedocs.io/en/latest/tutorial/step_1_setting_up_env_and_running_benchmark_from_samples.html>
* <http://rally.readthedocs.io/en/latest/tutorial/step_4_adding_success_criteria_for_benchmarks.html>
* <http://rally.readthedocs.io/en/latest/tutorial/step_2_input_task_format.html>

## Requirements

__Requirements for using Rally__

* Management-server with installed kvm virtualization
* installed Fuel-server as virtual machine in Management-server
* ready OpenStack deployment (OpenStack >= Kilo)
* access from Fuel to specific OpenStack Management LAN
* access from Fuel to internet
* ssh access to Fuel-server with root permissions
* "operator machine"
* installed Rally >= 0.4.0

__Requirements for "operator machine"__

* git
* ansible>=2.0
* downloaded git-repository "cml-vbx" (<https://git.cmlatitude.com/cml-vbx.git>)
* ssh root access to Management-server
* ssh root access to Fuel-server

## Rally Benchmarking OpenStack

1) Change directory to ansible project and copy inventory file from example
```sh
$ cd cml-vbx/ansible-projects/openstack-deployment/
$ cp hosts.example hosts
```

2) Define in inventory-file (for example, hosts) 5 variables in group `[all:vars]`:

 - ip-address of Management-server where Fuel-server is running as virtual machine  
   `
   ansible_ssh_host=some-ip
   `
 - SSH root password for Management-server or ssh-key  
   `
   ansible_ssh_pass=some-seсure-password
   `  
   or  
   `
   ansible_ssh_private_key_file=id_rsa.pub
   `
 - SSH user  
   `
   user=ubuntu
   `
 - prefix of Fuel virtual machine name  
   `
   prefix='some-prefix'
   `  
 - Horizon domain name (public API) for current OpenStack deployment    
   `
   openstack_hostname=public.domain.com
   `    
   
and 1 variable in group `[fuel-env:vars]`
  
 - Openstack deployment name in Rally  
   `
   env_name='some-env-name'
   `

  For example:
  
```
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
```

3) Define variables for ansible role rally-benchmark

__Default role settings:__   
(in file ansible-projects/openstack-deployment/roles/rally-benchmark/defaults/main.yml)

* Benchmark name    
benchmark_name: 'test-network'
* benchmark library source directory on ansible machine      
benchmark_lib_src_dir: '../../../../../rally-benchmarks/'
* benchmark library destination directory on Rally machine  
benchmark_lib_dst_dir: '/root/rally-benchmarks-library'
* File with benchmark list in YAML format   
benchmark_file: 'vm-performance/network/run_iperf3_via_heat.yml'
* All parameters for rally task here    
benchmark_args_file: 'vm-performance/network/run_iperf3_via_heat-args.yml'
* URL for downloading image for benchmarks
  You can use your own image with cloud-init and pre-installed special software for testing   
image_url: 'https://cloud-images.ubuntu.com/trusty/current/trusty-server-cloudimg-amd64-disk1.img'
* timestamp for results file name   
timestamp: '{{ ansible_date_time.date }}_{{ ansible_date_time.hour }}-{{ ansible_date_time.minute }}-{{ ansible_date_time.second }}'
* directory in remote rally machine for results     
rally_dir: '/root/rally/{{ deployment_name }}'
* Directory on local machine (ansible server) for results from remote rally machine  
local_results_dir: '/var/www/rally'


__Define variables in 2 ways:__

1.  Add needed variables in inventory file in part `[rally:vars]` for overriding default settings:

    * name of benchmark    
       `
       benchmark_name
       `
    * file name on ansible-machine  
       `
       benchmark_file
       `
    * file name of arguments for benchmark on ansible-machine  
       `
       benchmark_args_file
       `
    * directory on "operator machine" for results          
      `
      local_results_dir
      `     
    * URL for downloading image for benchmarks 
      `
      image_url
      `
  
    For example:
    
    ```
    benchmark_name='test_disk_io'
    benchmark_file='vm-performance/disk-io/vmtask_run_script_over_ssh.yml'
    benchmark_args_file='vm-performance/disk-io/vmtask_run_script_over_ssh-args.yml'
    local_results_dir='/var/www/rally'
    image_url='http://192.168.32.53/ubuntu-for-benchmarking.img'
    ```

2. In line with command `ansible-playbook...` for overriding default settings

For example:
```sh
# ansible-playbook -i hosts set-env-facts.yml rally-benchmark.yml -e '{ benchmark_name: "test_cpu", benchmark_file: "vm-performance/cpu-benchmark/vmtask_run_script_over_ssh.yaml", benchmark_args_file: "vm-performance/cpu-benchmark/vmtask_run_script_over_ssh-args.yaml" }' -vvv
```
 

4) Define arguments for rally benchmarking task in file cml-vbx/rally-benchmarks/[bencmark_name]/[bencmark_name]-args.yaml

All scenarios run parallel actions in cycle from "begin_cycle_range" (included) till "end_cycle_range" (excluded) 
by "concurrency" executions in one time. Other words, if you set "concurrency"=3, "begin_cycle_range"=1, "end_cycle_range"=5, then
rally executes by cycle iterations:      
1st iter - 1 parallel action        
2nd iter - 2 parallel actions       
3rd iter - 3 parallel actions       
4th iter - 3 parallel actions and after 1 action        

For example, in file "cml-vbx/rally-benchmarks/vm-performance/network/run_iperf3_via_heat-args.yml"
```
concurrency: 3
begin_cycle_range: 1
end_cycle_range: 2
### username for connecting to test VMs
username: "ubuntu"
### use needed flavors for yours purposes
flavor: "m1.small"
### name of downloaded image in Glance
image_name: "trusty-server-cloudimg-amd64-disk1.img"
### SLA parameters in seconds
max_avg_duration: "1150"
max_seconds_per_iteration: "1000"
run_command_over_ssh: "900"
### action name
command_name: "run TCP 10GB forward"
### path on remote rally-machine (Fuel-server); by default all rally-library copying to /root/rally-benchmarks-library/
script: "/root/rally-benchmarks-library/vm-performance/network/extra/iperf3_one_test_10GB.sh"
### dns server with records horizon_ip
dns_nameservers: "172.30.0.11"
```
For more information about rally benchmark tasks and customizing read in cml-vbx/rally-benchmarks/[bencmark_name]/README.rst

5) Run rally benchmarks via ansible:

- with default settings or settings from inventory-file:
```sh
$ ansible-playbook -i hosts rally-benchmark.yml -vvv
```

- with in-line parameters:
```sh
$ ansible-playbook -i hosts rally-benchmark.yml -e '{ benchmark_name: "test_cpu", benchmark_args_file: "vm-performance/cpu-benchmark/vmtask_run_script_over_ssh-args.yaml" }' -vvv
```

For example

* **Test CPU**:          
    Run on VM script that makes synthetic workload on CPU.  
    Rally creates from 1 to 10 simultaneously working VMs
```sh
$ ansible-playbook -i hosts set-env-facts.yml rally-benchmark.yml -e '{ benchmark_name: "test_cpu", image_url: "http://192.168.32.53/ubuntu-cloud.img", benchmark_file: "vm-performance/cpu/vmtask_run_script_over_ssh.yaml", benchmark_args_file: "vm-performance/cpu/vmtask_run_script_over_ssh-args.yaml" }' -vvv
```
__Result__: [report benchmark cpu](/virtbox-docs/rally-reports/cpu/aso_openstack_test_cpu_10vms_results_2016-08-07_14-53-26.html)        
    
* **Test disk input/output performance**:        
    Run on VM script that makes synthetic workload on disk (write and read operations).  
    Rally creates from 1 to 10 simultaneously working VMs
    
```sh
$ ansible-playbook -i hosts rally-benchmark.yml -e '{ benchmark_name: "test_disk", image_url: "http://192.168.32.53/ubuntu-cloud.img", benchmark_file: "vm-performance/disk-io/vmtask_run_script_over_ssh.yaml", benchmark_args_file: "vm-performance/disk-io/vmtask_run_script_over_ssh-args.yaml" }' -vvv
```
__Result__: [report benchmark disk input output](/virtbox-docs/rally-reports/disk/aso_openstack_test_disk_io_results_2016-08-07_18-51-43.html)
    
* **Test network performance**:     
    Rally creates via Heat Stack from 2 VMs and copies 10 GB via network between 2 VMs.       
    Rally creates from 1 to 10 simultaneously working Stacks (2 VMs in each, total 6 VMs)
    
```sh
$ ansible-playbook -i hosts rally-benchmark.yml -e '{ benchmark_name: "test_network", image_url: "https://cloud-images.ubuntu.com/trusty/current/trusty-server-cloudimg-amd64-disk1.img", benchmark_file: "vm-performance/network/run_iperf3_via_heat.yml", benchmark_args_file: "vm-performance/network/run_iperf3_via_heat-args.yml" }' -vvv
```
__Result__: [report benchmark network](/virtbox-docs/rally-reports/network/aso_openstack_test_network_10_stacks_results_2016-08-07_15-52-59.html)