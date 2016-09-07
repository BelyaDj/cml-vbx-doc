# Logging Monitoring Alerting

[TOC]

[MOS 7.0: Logging, Monitoring, Alerting (LMA) enhancements](https://www.mirantis.com/blog/mos-7-0-logging-monitoring-alerting-lma-enhancements/)

---

![architecture](/images/lma_map.png)

---

## LMA Collector 
[Official documentation](http://fuel-plugin-lma-collector.readthedocs.io/en/latest/index.html)

Heka process, configs

```sh
$ ps ax | grep hekad
2426 ?        Ssl   21:57 /usr/bin/hekad -config=/etc/log_collector
7943 ?        Ssl   17:54 /usr/bin/hekad -config=/etc/metric_collector
```    

Collectd process, configs

```sh
$ ps ax | grep collectd
24858 ?        Ss     0:00 /usr/sbin/collectdmon -P /var/run/collectd.pid -- -C /etc/collectd/collectd.conf
24859 ?        Sl     8:03 collectd -C /etc/collectd/collectd.conf -f
```    

---

## Elasticsearch and Kibana
[Official documentation](http://fuel-plugin-elasticsearch-kibana.readthedocs.io/en/latest/index.html)

Kibana data
```sh
$ whereis kibana
kibana: /opt/kibana/bin/kibana /opt/kibana/bin/kibana.bat
```

Elastic process
```
$ ps ax | grep elasticsearch
 2950 ?        SLl    1:24 /usr/lib/jvm/java-7-openjdk-amd64//bin/java -Xms1g -Xmx1g -Djava.awt.headless=true -XX:+UseParNewGC -XX:+UseConcMarkSweepGC \
    -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -XX:+HeapDumpOnOutOfMemoryError -XX:+DisableExplicitGC -Dfile.encoding=UTF-8 -Djna.nosys=true \
    -Des.path.home=/usr/share/elasticsearch -cp /usr/share/elasticsearch/lib/elasticsearch-2.3.3.jar:/usr/share/elasticsearch/lib/* org.elasticsearch.bootstrap.Elasticsearch start -d -p /var/run/elasticsearch-es-01.pid \
    -Des.default.path.home=/usr/share/elasticsearch \
    -Des.default.path.logs=/var/log/elasticsearch/es-01 \
    -Des.default.path.data=/var/lib/elasticsearch-es-01 \
    -Des.default.path.work=/tmp/elasticsearch-es-01 \
    -Des.default.path.conf=/etc/elasticsearch/es-01

```

Elasticsearch database 
```sh
$ df -h | grep elasticsearch
Filesystem                               Size  Used Avail Use% Mounted on
/dev/mapper/elasticsearch-elasticsearch   63G  3.0G   57G   5% /opt/es-data
```    

Elasticsearch database tree

```sh
$ tree -d /opt/es-data
/opt/es-data/
├── elasticsearch_data
│   └── es-01
│       └── lma
│           └── nodes
│               └── 0
│                   ├── indices
│                   │   ├── log-2016.07.14
│                   │   │   ├── 0
│                   │   │   │   ├── index
│                   │   │   │   ├── _state
│                   │   │   │   └── translog
│                   │   │   ├── 1
│                   │   │   │   ├── index
│                   │   │   │   ├── _state
│                   │   │   │   └── translog
│                   │   │   ├── 2
│                   │   │   │   ├── index
│                   │   │   │   ├── _state
│                   │   │   │   └── translog
│                   │   │   ├── 3
│                   │   │   │   ├── index
│                   │   │   │   ├── _state
│                   │   │   │   └── translog
│                   │   │   ├── 4
│                   │   │   │   ├── index
│                   │   │   │   ├── _state
│                   │   │   │   └── translog
│                   │   │   └── _state
│                   │   ├── log-2016.07.15
│                   │   │   ├── 0
│                   │   │   │   ├── index
│                   │   │   │   ├── _state
│                   │   │   │   └── translog
│                   │   │   ├── 1
│                   │   │   │   ├── index
...

```

---

## InfluxDB and Grafana
[Official documentation](http://fuel-plugin-influxdb-grafana.readthedocs.io/en/latest/index.html)

Grafana data
```sh
$ whereis grafana
grafana: /etc/grafana /usr/share/grafana
```

InfluxDB process 
```sh
$ ps ax | grep influxdb
2922 ?        Sl    66:58 /usr/bin/influxd -pidfile /var/run/influxdb/influxd.pid -config /etc/influxdb/influxdb.conf
```

InfluxDB database 
```sh
$ df -h | grep influxdb
/dev/mapper/influxdb-influxdb  186G  359M  176G   1% /var/lib/influxdb
```

InfluxDB database tree
```sh
$ tree /var/lib/influxdb
/var/lib/influxdb
├── data
│   ├── _internal
│   │   └── monitor
│   │       ├── 1
│   │       │   └── 000000001-000000001.tsm
│   │       ├── 3
│   │       │   └── 000000001-000000001.tsm
│   │       ├── 6
│   │       │   └── 000000001-000000001.tsm
│   │       ├── 7
│   │       │   └── 000000001-000000001.tsm
│   │       └── 9
│   └── lma
│       └── default
│           ├── 10
│           │   ├── 000000004-000000003.tsm
│           │   └── 000000008-000000003.tsm
│           ├── 2
│           │   └── 000000012-000000004.tsm
│           ├── 4
│           │   └── 000000012-000000002.tsm
│           ├── 5
│           │   └── 000000007-000000002.tsm
│           └── 8
│               ├── 000000016-000000004.tsm
│               └── 000000018-000000002.tsm
├── hh
├── lost+found
├── meta
│   ├── node.json
│   ├── raft.db
│   └── snapshots
└── wal
    ├── _internal
    │   └── monitor
    │       ├── 1
    │       │   └── _00003.wal
    │       ├── 3
    │       │   └── _00003.wal
    │       ├── 6
    │       │   └── _00002.wal
    │       ├── 7
    │       │   └── _00004.wal
    │       └── 9
    │           ├── _00001.wal
    │           └── _00002.wal
    └── lma
        └── default
            ├── 10
            │   └── _00033.wal
            ├── 2
            │   └── _00048.wal
            ├── 4
            │   └── _00044.wal
            ├── 5
            │   └── _00026.wal
            └── 8
                └── _00071.wal
```

---

## LMA Infrastructure Alerting, Nagios
[Official documentation](http://fuel-plugin-lma-infrastructure-alerting.readthedocs.io/en/latest/index.html)

Nagios process 
```sh
$ ps ax | grep nagios
5355 ?        Ssl    2:36 /usr/sbin/nagios3 -d /etc/nagios3/nagios.cfg
```

Nagios data
```sh
$ df -h | grep nagios
/dev/mapper/nagios-nagios      20511356   45568  19400828   1% /var/nagios
```

Nagios alerting archives, etc.
```sh
$ tree /var/nagios
 /var/nagios
 ├── archives
 │   ├── nagios-07-15-2016-00.log
 │   ├── nagios-07-17-2016-00.log
 │   └── nagios-07-18-2016-00.log
 ├── cache
 │   ├── objects.cache
 │   └── status.dat
 ├── lost+found
 └── nagios.log
```

---
