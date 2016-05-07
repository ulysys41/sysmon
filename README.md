# sysmon
## 준비물
1. Source Host: 모니터링 대상 Host
2. Metric Crawler/forwarder : 대상 Host의 Metric을 생성하는 deamon(CollectD)
  - CollectD
3. Metric Aggregator : 수집되는 Metric을 Metric Repository에 저장하는 Queue(Deffered, Async) 또는 Metirc의 데이터 정제 역할
  - CollectD
4. Metric Repository : 시간과 Host를 기준으로 Metirc을 저장해 주는 시계열DB
  - InfluxDB
5. Metric Dashboard : Metric Repositoy를 DataSource로 사용하여 시계열 그래프로 Metric정보를 표시해 주는 역할
  - Grafana


## common
```
ulimit -n 65536
```
* change limits(`/etc/security/limits.conf`)
```
* 		hard	nofile		65536
* 		soft	nofile		65536
```

## 1. collectD
* [install](https://www.digitalocean.com/community/tutorials/how-to-configure-collectd-to-gather-system-metrics-for-graphite-on-ubuntu-14-04)
```
sudo apt-get update
sudo apt-get install collectd collectd-utils
```
* add FQDN
```
127.0.1.1       metric-aggregator.insator.org   metric-aggregator
127.0.1.1       metric-crawler.insator.org      metric-crawler
127.0.1.1       metric-repository.insator.org   metric-repository
```
* Configuration
  - vim `/etc/collectd/collectd.conf`
```
#Hostname "localhost"
FQDNLookup true
BaseDir "/var/lib/collectd"
PluginDir "/usr/lib/collectd"
TypesDB "/usr/share/collectd/types.db"
...
Interval 10
...
```
```
LoadPlugin syslog
LoadPlugin cpu
LoadPlugin df
LoadPlugin disk
LoadPlugin entropy
LoadPlugin interface
LoadPlugin irq
LoadPlugin load
LoadPlugin memory
LoadPlugin processes
LoadPlugin rrdtool
LoadPlugin swap
LoadPlugin users
```
* syslog
```
LoadPlugin syslog
#LoadPlugin log_logstash

#<Plugin logfile>
#       LogLevel "info"
#       File STDOUT
#       Timestamp true
#       PrintSeverity false
#</Plugin>
<Plugin syslog>
        LogLevel info
</Plugin>
```
* cpu
```
<Plugin cpu>
        ReportByCpu true
        ReportByState true
        ValuesPercentage false
</Plugin>
```
* df
```
<Plugin df>
        Device "/dev/sda1"
#       Device "192.168.0.2:/mnt/nfs"
        MountPoint "/home"
        FSType "ext4"

        # ignore rootfs; else, the root file-system would appear twice, causing
        # one of the updates to fail and spam the log
        FSType rootfs
        # ignore the usual virtual / temporary file-systems
        FSType sysfs
        FSType proc
        FSType devtmpfs
        FSType devpts
        FSType tmpfs
        FSType fusectl
        FSType cgroup
        IgnoreSelected true

#       ReportByDevice false
#       ReportInodes false

#       ValuesAbsolute true
#       ValuesPercentage false
</Plugin>
```
* interface
```
<Plugin interface>
        Interface "eth0"
        IgnoreSelected false
</Plugin>
```
* load
```
<Plugin load>
        ReportRelative true
</Plugin>
```
* memory
```
<Plugin memory>
        ValuesAbsolute true
        ValuesPercentage false
</Plugin>
```
* process
```
<Plugin processes>
        Process "name"
#       ProcessMatch "foobar" "/usr/bin/perl foobar\\.pl.*"
</Plugin>
```
* rrdtool
```
<Plugin rrdtool>
        DataDir "/var/lib/collectd/rrd"
#       CacheTimeout 120
#       CacheFlush 900
#       WritesPerSecond 30
#       CreateFilesAsync false
#       RandomTimeout 0
#
# The following settings are rather advanced
# and should usually not be touched:
#       StepSize 10
#       HeartBeat 20
#       RRARows 1200
#       RRATimespan 158112000
#       XFF 0.1
</Plugin>
```
* swap
```
<Plugin swap>
        ReportByDevice false
        ReportBytes true
</Plugin>
```


## InfluxDB
* Install
```
wget https://dl.influxdata.com/influxdb/releases/influxdb_0.12.2-1_amd64.deb
sudo dpkg -i influxdb_0.12.2-1_amd64.deb
```
* [config](https://github.com/influxdata/influxdb/blob/master/services/collectd/README.md)
```
[[collectd]]
  enabled = true
  bind-address = ":25826" # the bind address
  database = "collectd" # Name of the database that will be written to
  retention-policy = ""
  batch-size = 5000 # will flush if this many points get buffered
  batch-pending = 10 # number of batches that may be pending in memory
  batch-timeout = "10s"
  read-buffer = 0 # UDP read buffer size, 0 means to use OS default
  typesdb = "/usr/share/collectd/types.db"
```
* [best practice1](http://jessesnet.com/development-notes/2015/using-collectd-with-influxdb/)
* [best practice2](https://sonnguyen.ws/monitor-server-with-collectd-influxdb-and-grafana/)
