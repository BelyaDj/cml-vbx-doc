# FUEL

[TOC]

---

## Building FUEL plugin from source
```sh
yum install git python-pip createrepo dpkg-devel dpkg-dev rpm rpm-build fuel
git clone https://github.com/stackforge/fuel-plugin-ldap
pip install fuel-plugin-builder
fpb --build fuel-plugin-ldap/
# if failed try to upgrade "reqests"
pip install --upgrade requests
```

## Install FUEL plugin
```sh
fuel plugins --install fuel-plugin-ldap.rpm
```

---
