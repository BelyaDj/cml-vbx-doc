# Neutron

[TOC]

---

## Problem with Virtual Router

##### Interfaces(ports) always in status "DOWN"

You need to restart a `neutron-l3-agent` on controller node
```sh
service neutron-l3-agent restart
```

---
