# Swift

[TOC]

---

## How Swift stores Glance images

Get password for glance user
```
# grep glance /etc/astute.yaml -A 5 | grep user_password | awk '{ print $2 }'
TUpmdpgbj9e2laXL5JQn16TS
```

List all swift objects stored in container named glance
```
# swift --os-auth-url http://10.0.0.2:5000/v2.0/ --auth-version 2 --os-project-name services --os-username glance --os-password TUpmdpgbj9e2laXL5JQn16TS list glance
0c808e53-bfce-468e-9670-ccd3203af4f5
236ddb85-0717-43f5-8040-8e9795ce258d
```

---
