# WEEK1-LESSON1-Hello-Transaction

## 要求
evernotecid://B0FF5B04-D5A4-4E17-9F0C-79B9BF8F0D46/appyinxiangcom/3130324/ENResource/p2397

## rust安装

```shell script
# rust 安装
curl https://sh.rustup.rs -sSf | sh
```

## 下载&编译
```shell script
git clone https://github.com/qihengshan/tikv.git
make
git clone https://github.com/qihengshan/tidb.git
make
git clone https://github.com/qihengshan/pd.git
make
```
## 启动

### pd-server部署

```shell script
./bin/pd-server --name=pd1 \
--data-dir=pd1 \
--client-urls="http://127.0.0.1:2379" \
--peer-urls="http://127.0.0.1:2380" \
--initial-cluster="pd1=http://127.0.0.1:2380" -L "info" \
--log-file=./logs/pd1.log
```

### tikv部署
```shell script
./bin/tikv-server --pd="127.0.0.1:2379" --addr="127.0.0.1:20160" --data-dir=tikv1 --log-file=./logs/tikv1.log &
./bin/tikv-server --pd="127.0.0.1:2379" --addr="127.0.0.1:20161" --data-dir=tikv2 --log-file=./logs/tikv2.log &
./bin/tikv-server --pd="127.0.0.1:2379" --addr="127.0.0.1:20162" --data-dir=tikv3 --log-file=./logs/tikv3.log &
```

### tidb部署
```shell script
./bin/tidb-server --store=tikv --path="127.0.0.1:2379" --log-file=./logs/tidb1.log &
```

## 修改源码：启动事务打印 hello transaction
**tidb->kv->txn.go**
```go
// RunInNewTxn will run the f in a new transaction environment.
func RunInNewTxn(store Storage, retryable bool, f func(txn Transaction) error) error {
	var (
		err           error
		originalTxnTS uint64
		txn           Transaction
	)
	logutil.BgLogger().Info("Hello Transaction")
```

## 测试
```sql
mysql> create table test01(id bigint auto_increment, name varchar(50),primary key(id) ) engine=InnoDB charset utf8mb4; 
Query OK, 0 rows affected (0.09 sec)

mysql> show create table test01\G
*************************** 1. row ***************************
       Table: test01
Create Table: CREATE TABLE `test01` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin
1 row in set (0.00 sec)

mysql> insert into test01(name) values('name001'),('name002'),('name003');
Query OK, 3 rows affected (0.01 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> begin ; insert into test01(name) values('name004'),('name005'); commit;
Query OK, 0 rows affected (0.00 sec)

Query OK, 2 rows affected (0.01 sec)
Records: 2  Duplicates: 0  Warnings: 0

Query OK, 0 rows affected (0.00 sec)
```

### 日志输出
```text
[2020/08/18 22:11:29.795 +08:00] [INFO] [txn.go:35] ["Hello Transaction"]
[2020/08/18 22:11:30.121 +08:00] [INFO] [txn.go:35] ["Hello Transaction"]
[2020/08/18 22:11:30.125 +08:00] [WARN] [2pc.go:1006] ["schemaLeaseChecker is not set for this transaction, schema check skipped"] [connID=0] [startTS=418843168630112259] [commitTS=418843168630112260]
[2020/08/18 22:11:30.795 +08:00] [INFO] [txn.go:35] ["Hello Transaction"]
[2020/08/18 22:11:30.795 +08:00] [INFO] [txn.go:35] ["Hello Transaction"]
```