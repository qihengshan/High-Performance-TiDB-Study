# TiDB压测
## 压测工具
* sysbench
* go-ycsb
* go-tcp

## 机器配置
aliyun机器 9台

## 集群拓扑信息

|实例 |个数 | 物理机配置 |IP | 配置|
|:----| :----|:----|:----|:----|
|TiDB<br>PD | 3 |12 vCPU 48 GiB * 1 300GB (ssd) |172.31.235.200 <br> 172.31.235.201 <br> 172.31.235.202 | 默认端口,全局目录配置 |
|TiKV  |3 |16 VCore 32GB 2TB (nvme ssd) * 1 | 172.31.235.203 <br> 172.31.235.204 <br> 172.31.235.205 | 默认端口,全局目录配置 |
|TiFlash | 1 | 12 vCPU 48 GiB 2TB (nvme ssd) * 1 | 172.31.235.199 | 默认端口, 全局目录配置 |
|Monitoring & Grafana | 1 | 4 VCore 8GB * 1 100GB (ssd) | 172.31.235.198 | 默认端口,全局目录配置|
|压测机 | 1 | 12 VCore 24GB * 1 100GB (ssd) | 172.31.235.197 | 无 |


## 集群部署
### 部署配置文件
```yaml
# # Global variables are applied to all deployments and used as the default value of
# # the deployments if a specific deployment value is missing.
global:
  user: "work"
  ssh_port: 22
  deploy_dir: "/home/work/tidb-deploy"
  data_dir: "/home/work/tidb-data"

# # Monitored variables are applied to all the machines.
monitored:
  node_exporter_port: 9100
  blackbox_exporter_port: 9115
  # deploy_dir: "/tidb-deploy/monitored-9100"
  # data_dir: "/tidb-data/monitored-9100"
  # log_dir: "/tidb-deploy/monitored-9100/log"

# # Server configs are used to specify the runtime configuration of TiDB components.
# # All configuration items can be found in TiDB docs:
# # - TiDB: https://pingcap.com/docs/stable/reference/configuration/tidb-server/configuration-file/
# # - TiKV: https://pingcap.com/docs/stable/reference/configuration/tikv-server/configuration-file/
# # - PD: https://pingcap.com/docs/stable/reference/configuration/pd-server/configuration-file/
# # All configuration items use points to represent the hierarchy, e.g:
# #   readpool.storage.use-unified-pool
# #
# # You can overwrite this configuration via the instance-level `config` field.

server_configs:
  tidb:
    log.slow-threshold: 300
  tikv:
    # server.grpc-concurrency: 4
    # raftstore.apply-pool-size: 2
    # raftstore.store-pool-size: 2
    # rocksdb.max-sub-compactions: 1
    # storage.block-cache.capacity: "16GB"
    # readpool.unified.max-thread-count: 12
    readpool.storage.use-unified-pool: false
    readpool.coprocessor.use-unified-pool: true
  pd:
    schedule.leader-schedule-limit: 4
    schedule.region-schedule-limit: 2048
    schedule.replica-schedule-limit: 64

pd_servers:
  - host: 172.31.235.200
    # ssh_port: 22
    # name: "pd-1"
    # client_port: 2379
    # peer_port: 2380
    # deploy_dir: "/tidb-deploy/pd-2379"
    # data_dir: "/tidb-data/pd-2379"
    # log_dir: "/tidb-deploy/pd-2379/log"
    # numa_node: "0,1"
    # # The following configs are used to overwrite the `server_configs.pd` values.
    # config:
    #   schedule.max-merge-region-size: 20
    #   schedule.max-merge-region-keys: 200000
  - host: 172.31.235.201
  - host: 172.31.235.202

tidb_servers:
  - host: 172.31.235.200
    # ssh_port: 22
    # port: 4000
    # status_port: 10080
    # deploy_dir: "/tidb-deploy/tidb-4000"
    # log_dir: "/tidb-deploy/tidb-4000/log"
    # numa_node: "0,1"
    # # The following configs are used to overwrite the `server_configs.tidb` values.
    # config:
    #   log.slow-query-file: tidb-slow-overwrited.log
  - host: 172.31.235.201
  - host: 172.31.235.202

tikv_servers:
  - host: 172.31.235.203
    # ssh_port: 22
    # port: 20160
    # status_port: 20180
    # deploy_dir: "/tidb-deploy/tikv-20160"
    # data_dir: "/tidb-data/tikv-20160"
    # log_dir: "/tidb-deploy/tikv-20160/log"
    # numa_node: "0,1"
    # # The following configs are used to overwrite the `server_configs.tikv` values.
    # config:
    #   server.grpc-concurrency: 4
    #   server.labels: { zone: "zone1", dc: "dc1", host: "host1" }

  - host: 172.31.235.204
  - host: 172.31.235.205

tiflash_servers:
  - host: 172.31.235.199
    # data_dir: /home/work/tidb-data/tiflash-9000
    # deploy_dir: /home/work/tidb-deploy/tiflash-9000
    # ssh_port: 22
    # tcp_port: 9000
    # http_port: 8123
    # flash_service_port: 3930
    # flash_proxy_port: 20170
    # flash_proxy_status_port: 20292
    # metrics_port: 8234
    # deploy_dir: /tidb-deploy/tiflash-9000
    # numa_node: "0,1"
    # # The following configs are used to overwrite the `server_configs.tiflash` values.
    # config:
    #   logger.level: "info"
    # learner_config:
    #   log-level: "info"
  # - host: 10.0.1.12
  # - host: 10.0.1.13

monitoring_servers:
  - host: 172.31.235.198
    # ssh_port: 22
    # port: 9090
    # deploy_dir: "/tidb-deploy/prometheus-8249"
    # data_dir: "/tidb-data/prometheus-8249"
    # log_dir: "/tidb-deploy/prometheus-8249/log"

grafana_servers:
  - host: 172.31.235.198
    # port: 3000
    # deploy_dir: /tidb-deploy/grafana-3000

alertmanager_servers:
  - host: 172.31.235.198
    # ssh_port: 22
    # web_port: 9093
    # cluster_port: 9094
    # deploy_dir: "/tidb-deploy/alertmanager-9093"
    # data_dir: "/tidb-data/alertmanager-9093"
    # log_dir: "/tidb-deploy/alertmanager-9093/log"
```

### Tiup安装集群

`tiup cluster deploy tidb-stress v4.0.4 ./topology-tidb-cluster.ymal --user root -i /root/.ssh/id_rsa`

*{pd-ip}:{pd-port}/dashboard*

*{grafana_servers-ip}:{grafana_servers-port}/*

#### 部署完毕后, 拓扑信息
```shell script
[root@tidb-stress workspace]# tiup cluster list
Starting component `cluster`: /root/.tiup/components/cluster/v1.0.9/tiup-cluster list
Name         User  Version  Path                                              PrivateKey
----         ----  -------  ----                                              ----------
tidb-stress  work  v4.0.4   /root/.tiup/storage/cluster/clusters/tidb-stress  /root/.tiup/storage/cluster/clusters/tidb-stress/ssh/id_rsa
[root@tidb-stress workspace]#
[root@tidb-stress workspace]#
[root@tidb-stress workspace]# tiup cluster display tidb-stress
Starting component `cluster`: /root/.tiup/components/cluster/v1.0.9/tiup-cluster display tidb-stress
tidb Cluster: tidb-stress
tidb Version: v4.0.4
ID                    Role          Host            Ports                            OS/Arch       Status  Data Dir                                Deploy Dir
--                    ----          ----            -----                            -------       ------  --------                                ----------
172.31.235.198:9093   alertmanager  172.31.235.198  9093/9094                        linux/x86_64  Up      /home/work/tidb-data/alertmanager-9093  /home/work/tidb-deploy/alertmanager-9093
172.31.235.198:3000   grafana       172.31.235.198  3000                             linux/x86_64  Up      -                                       /home/work/tidb-deploy/grafana-3000
172.31.235.200:2379   pd            172.31.235.200  2379/2380                        linux/x86_64  Up      /home/work/tidb-data/pd-2379            /home/work/tidb-deploy/pd-2379
172.31.235.201:2379   pd            172.31.235.201  2379/2380                        linux/x86_64  Up|UI   /home/work/tidb-data/pd-2379            /home/work/tidb-deploy/pd-2379
172.31.235.202:2379   pd            172.31.235.202  2379/2380                        linux/x86_64  Up|L    /home/work/tidb-data/pd-2379            /home/work/tidb-deploy/pd-2379
172.31.235.198:9090   prometheus    172.31.235.198  9090                             linux/x86_64  Up      /home/work/tidb-data/prometheus-9090    /home/work/tidb-deploy/prometheus-9090
172.31.235.200:4000   tidb          172.31.235.200  4000/10080                       linux/x86_64  Up      -                                       /home/work/tidb-deploy/tidb-4000
172.31.235.201:4000   tidb          172.31.235.201  4000/10080                       linux/x86_64  Up      -                                       /home/work/tidb-deploy/tidb-4000
172.31.235.202:4000   tidb          172.31.235.202  4000/10080                       linux/x86_64  Up      -                                       /home/work/tidb-deploy/tidb-4000
172.31.235.199:9000   tiflash       172.31.235.199  9000/8123/3930/20170/20292/8234  linux/x86_64  Up      /home/work/tidb-data/tiflash-9000       /home/work/tidb-deploy/tiflash-9000
172.31.235.203:20160  tikv          172.31.235.203  20160/20180                      linux/x86_64  Up      /home/work/tidb-data/tikv-20160         /home/work/tidb-deploy/tikv-20160
172.31.235.204:20160  tikv          172.31.235.204  20160/20180                      linux/x86_64  Up      /home/work/tidb-data/tikv-20160         /home/work/tidb-deploy/tikv-20160
172.31.235.205:20160  tikv          172.31.235.205  20160/20180                      linux/x86_64  Up      /home/work/tidb-data/tikv-20160         /home/work/tidb-deploy/tikv-20160
[root@tidb-stress workspace]# 
```
 
集群信息查看
```text
tiup cluster list
tiup cluster display tidb-stress
```

## Haproxy配置
```shell script
wget https://www.haproxy.org/download/2.2/src/haproxy-2.2.2.tar.gz
tar -zxvf haproxy-2.2.2.tar.gz && mv haproxy-2.2.2 haproxy
cd haproxy && mkdir etc && mkdir -p var/run
make TARGET=linux-glibc PREFIX=/home/work/haproxy
make install PREFIX=/home/work/haproxy
```
```text
# cat /home/work/haproxy/etc/haproxy.cfg
global
        log 127.0.0.1 local0 info
        maxconn 65535
        daemon
        stats socket /home/work/haproxy/var/run/stats mode 700 level admin
        #debug
        #quiet

defaults
#        log     global
        log    127.0.0.1  local0 info
        mode    tcp
        option dontlognull
        option tcpka
        option tcplog
        retries 3
        maxconn 65535
        balance roundrobin
        timeout queue   1m 
        timeout connect 100s
        timeout client  86400s
        timeout server  86400s
        timeout check   100s


listen  SLAVE
#        log     global
        mode tcp
        bind 0.0.0.0:4000
        log    127.0.0.1  local0 info
        maxconn 4096
        balance leastconn
        server tidb-node1:4000 172.31.235.200:4000 weight 100 check inter 5s rise 2 fall 3
        server tidb-node2:4000 172.31.235.201:4000 weight 100 check inter 5s rise 2 fall 3
        server tidb-node3:1000 172.31.235.202:4000 weight 100 check inter 5s rise 2 fall 3
        # ##option mysql-check user haproxy 

listen  admin_status
        mode  http
        bind 0.0.0.0:8899
        option httplog
        log global
        stats enable
        stats refresh 300s
        stats hide-version
        stats realm Haproxy\ Statistics
        stats uri  /admin-status
        stats auth  admin:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
        stats admin if TRUE
```
启动
`/home/work/haproxy/sbin/haproxy -D -f /home/work/haproxy/etc/haproxy.cfg -p /home/work/haproxy/var/run/haproxy.pid`


## sysbench
sysbench是lua编写的简单的数据库压测工具
包括：随机点查、简单更新(update where id=?)、范围查询
### 安装
```shell script
curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.rpm.sh | sudo bash 
yum install -y sysbench
```

*配置文件*

sysbench-config.conf
```text
mysql-host=172.31.235.197
mysql-port=4000
mysql-user=root
mysql-db=sbtest
report-interval=10
db-driver=mysql
```
*准备*
```sql
# 创建db
create database sbtest;
# 导入数据之前先设置为乐观事务模式, 导入数据结束在设置为悲观模式
set global tidb_disable_txn_auto_retry = off;
set global tidb_txn_mode = "optimistic";
```
*压测命令*
stress-command.txt
```text
# 数据导入
sysbench --config-file=sysbench-config.conf oltp_point_select --threads=100 --tables=100 --table_size=1000000 prepare
# point select
sysbench --config-file=sysbench-config.conf oltp_point_select --threads=128 --tables=100 --table_size=1000000 --time=120 run
# update index
sysbench --config-file=sysbench-config.conf oltp_update_index --threads=128 --tables=100 --table_size=1000000 --time=120 run
# read only
sysbench --config-file=sysbench-config.conf oltp_read_write --threads=128 --tables=100 --table_size=1000000 --time=120 run
```

### 压测结果
oltp_point_select

| Threads | QPS       |95% latency(ms)|
| :----   | :----     | :----|
|128      |187246     |1.08  |
|256      |250835     |2.00  |
|512      |267957     |4.10  |

oltp_read_write

| Threads | QPS       |95% latency(ms)|
| :----   | :----     | :---- |
|128      |43747      |167.44 |
|256      |59683      |211.60 |
|512      |74236      |272.27 |

## go-ycsb
golang版本的ycsb数据库性能测试工具
负载类型包括：workloada ~ workloadf 6中类型
可以调整wordloads下的配置文件来调整如下配置:
1. 调整 requestdistribution 来更改随机分布的方式
2. 调整 fieldcount 来修改 count 数量
更多配置参考: [https://github.com/brianfrankcooper/YCSB/wiki/Core-Properties](https://github.com/brianfrankcooper/YCSB/wiki/Core-Properties)

### 安装
`git clone https://github.com/pingcap/go-ycsb && cd go-ycsb && make`


### 压测
load & run
```shell script
./bin/go-ycsb load mysql -P workloads/workloada -p recordcount=1000000 -p fieldcount=100 -p mysql.host={{host}} -p mysql.port={{port}} -p mysql.db={{db}}
./bin/go-ycsb run mysql -P workloads/workloada -p operationcount=2000000 -p filecount=100 -p mysql.host={{host}} -p mysql.port={{port}} -p mysql.db={{db}} --threads=128
```

| Threads | read      | update |
| :----   | :----     | :----  |
|128      |1682.8     | 1687.9 |
|256      |2177.0     | 2170.1 |
|512      |2232.1     | 2218.1 |
|1024     |2579.3     | 2509.7 |

## go-tcp 压测工具

### 安装
`git clone https://github.com/pingcap/go-tpc.git && cd go-tpc && make build`

### 压测 tpcc

```shell script
# TPC-C
./bin/go-tpc tpcc -H {{host}} -P {{port}} -D tpcc --warehouses 1000 --threads=256 prepare
./bin/go-tpc tpcc -H {{host}} -P {{port}} -D tpcc --warehouses 1000 --threads=256 --time 120s run
```
大概数据量
```text
+--------------------+---------+
| table_schema       | size(G) |
+--------------------+---------+
| METRICS_SCHEMA     |    0.00 |
| tpcc               |   87.57 |
| mysql              |    0.00 |
| INFORMATION_SCHEMA |    0.00 |
| PERFORMANCE_SCHEMA |    0.00 |
+--------------------+---------+
```

#### 压测结果
time 120s

| Threads | tpmC      |
| :----   | :----     |
|128      |6692.3     |
|256      |8628.3     |
|512      |10009.4    |
|1024     |10614.6    |


### 压测 tpch
```shell script
# TPC-H 针对分析性数据库压测
./bin/go-tpc tpch prepare -H {{host}} -P {{port}} -D tpch -sf 15 --analyze
./bin/go-tpc tpch run -H {{host}} -P {{port}} -D tpch -sf 15
```
