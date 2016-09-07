# Ansible

[TOC]

---

## Install latest Ansible version

Add Ansible repository
```sh    
$ sudo apt-get -y install software-properties-common && sudo apt-add-repository -y ppa:ansible/ansible
```

Install
```sh
$ sudo apt-get update && sudo apt-get -y install ansible
```

---

## Downgrade Ansible to previous version

```sh
# check available version
$ sudo apt-cache policy ansible

ansible:
  Installed: (none)
  Candidate: 2.1.0.0-1ppa~trusty
  Version table:
     2.1.0.0-1ppa~trusty 0
        500 http://ppa.launchpad.net/ansible/ansible/ubuntu/ trusty/main amd64 Packages
     1.7.2+dfsg-1~ubuntu14.04.1 0
        100 http://archive.ubuntu.com/ubuntu/ trusty-backports/universe amd64 Packages
     1.5.4+dfsg-1 0
        500 http://archive.ubuntu.com/ubuntu/ trusty/universe amd64 Packages
```

```sh
# downgrade
$ sudo apt-get install ansible=1.7.2+dfsg-1~ubuntu14.04.1
```

---

## Building Ansible using source code on Ubuntu 14.04

```sh
$ sudo apt-get remove ansible
$ sudo apt-get install asciidoc make debhelper cdbs devscripts
$ git clone --recursive --branch v2.0.2.0-1 https://github.com/ansible/ansible.git
$ cd ansible && make deb
$ sudo dpkg -i deb-build/unstable/*.deb
```

---

## Building Ansible using source code on CentOS 7

```sh
# We need to install EPEL repository and using "yum localinstall" in order to avoid broken dependencies 
$ wget http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-6.noarch.rpm
$ sudo rpm -ivh epel-release-7-6.noarch.rpm
$ sudo yum remove ansible
$ git clone --recursive --branch v2.0.2.0-1 https://github.com/ansible/ansible.git
$ cd ansible && make rpm
$ sudo yum localinstall rpm-build/ansible*centos.noarch.rpm
```

---

## Boolean in "--exrta-vars"

This is some behaviour examples of boolean values in `--extra-vars`

```
$ ansible-playbook -i hosts.example kvm-host.yml
...
TASK [debug] *******************************************************************
fatal: [kvm-host]: FAILED! => {"failed": true, "msg": "'test' is undefined"}
...
```

```
$ ansible-playbook -i hosts.example kvm-host.yml -e 'test: True'
...
TASK [debug] *******************************************************************
fatal: [kvm-host]: FAILED! => {"failed": true, "msg": "'test' is undefined"}
...
```

```
$ ansible-playbook -i hosts.example kvm-host.yml -e test=True
...
TASK [debug] *******************************************************************
ok: [kvm-host] => {
        "msg": "var = True, type = <type 'unicode'>"
}
...
```
```
$ ansible-playbook -i hosts.example kvm-host.yml -e 'test=True'
...
TASK [debug] *******************************************************************
ok: [kvm-host] => {
        "msg": "var = True, type = <type 'unicode'>"
}
...
```

```
$ ansible-playbook -i hosts.example kvm-host.yml -e '{ test: True }'
...
TASK [debug] *******************************************************************
ok: [kvm-host] => {
        "msg": "var = True, type = <type 'bool'>"
}
...
```
---
