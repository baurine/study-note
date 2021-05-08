# TiDB Study Note

source: [TiDB 线上课程](https://university.pingcap.com/)

## PCTA-1001 TiDB 架构概述

TiDB 设计思想，周边工具

why: history

数据暴发性增长

解决思想：

1. 分库分表
1. NoSQL - 放弃 SQL，事物... 必须放到上层做事物
1. 2012 F1 & Spanner，前者计算引擎，后者存储引擎

TiDB - F1 & Spanner

- 扩展
- 高可用性
- ACID transaction
- SQL

整体架构：TiDB / TiKV / PD

TiDB 是无状态的，所以可以随便扩展加机器

TiKV 是存储层，底层是 RocksDB，为了消除单点失效，创建副本，使用一致性协议 Raft，实现高可用，默认 3 副本

扩展性问题：scale，将数据切片，分布在不同机器上，分片算法，region，key 是连续区间，region 根据容量自动分裂

MVCC：`key_version: data` 可以查询历史数据

Transaction Model: Google Percolator，乐观锁，取 timestamp 是单点，但不会成为瓶颈

PD：集群管理器，自己也是集群，避免单点失效

集群的配置信息放在 PD 中，PD 对集群进行自动调度，是集群的大脑

SQL Layer: TiDB

- map table to key-value

TiDB 这一层内部挺复杂的，主要是解析 SQL，优化，计算下推

DDL，TiDB 无需锁表

Part 4 - TiSpark = Spark SQL on TiKV

Part 5 - Tooling: syncer, binlog (增量)

全量 mydumper, loader (via tidb), lightning (skip tidb, 直接导入 tikv 中)

## PCTA-1002 TiDB SQL 基础知识

- data type
- SQL Statements: DDL DML
- 使用索引优化

支持 MySQL 除了空间类型外的其它类型，只支持 utf8 bm4

DDL - database/table 创建修改，创建索引

DML - CRUD

having 只能放在 group by 后面

inner join (join 不上的抛弃) / left join, right join (join 不上的不抛弃)

optimize:

- analyse table (收集表的统计信息)
- the explain statements (查看查询计划)
- create and use index
- index hints: use index 强制使用某个 index，ignore index 不使用某个 index

## PCTA-1003 TiKV 集群基础知识

How we build TiKV

B+ Tree / LSM 有写放大的问题 --> Titan 解决写放大的问题

Raft 一致性协议

MySQL 使用 master/slave 但不能保证 slave 的数据和 master 是一致的

分布式事物：Percolator MVCC

PD: scheduler

- leader balance
- region balance
- hot region balance
- ...

## PCTA-4001 分布式数据库概览

数据库的历史

什么是数据库 - wikipedia, data / data model / processing (CRUD)

birth: 1970 IBM paper, 1972 Oracle, 1995 MySQL, 1996 PostgreSQL

- 2003 Memcached key-value
- 2006 no sql, google big table,
- cassandra 2008
- redis 2009

relational database: ...

- Oracle: focus on OLTP
- MySQL: MariaDB
- PostgreSQL: 偏学术，周边生态不如 MySQL，最近几年流行起来

将来是分布式数据库

part 3 - 存储引擎

B-tree / LSM

B-tree: InnoDB / LMDB / WiredTiger (mongoDB?) (firendly to HDD)

LSM-tree: LevelDB / RocksDB / WiredTiger (SSD)

part 4 - NoSQL

某些场景下，可以牺牲强一致性，换取低延迟，高并发，比如 weibo 等...

Google and Amazon's work

Google:

- GFS
- MapReduce
- BigTable， 底层存在 GFS 中 --> 开源版本: hbase

Amazon:

- Dynamo --> 开源版本：Cassandra

hadoop platform:

hbase: 实际底层存在 hdfs 中

mongoDB: 创新在于数据模型，json，不要求每行 schema 一样，但也是坑... 一般用于产品原型或小项目

ElasticSearch: 全文检索引擎 ELK (ES+Logstash+Kibana)

cache system:

- memcached
- redis

单机

codis: redis cluster

part 5 - OLAP & data warehouse

分析场景，报表...

相关的数据库：

Hive: SQL --> 转成 map reduce，缺点：map reduce 很慢，不实时

Impala / Kudu: MPP SQL Engine

GreenPlum:

Apache Kylin: 空间换时间

Apache Spark: 替换 map reduce，比 map reduce 快 100x ... 和 hadoop & hdfs 一起用，也可以和其它底层 db 一起用

part 6 - distribution db / NewSQL

- 扩容 / 缩容，弹性伸缩
- 存储和计算分离，所需资源不同
- 高可用，避免单点，一致性
- 兼容 sql
- ACID

Google Spanner / F1

- spanner: no sql
- F1: sql

Cockroach DB

- PostgreSQL
- Pure Go

TiDB

- HTAP
- TiDB / TiKV / TiSpark

part 7 - cloud native

virtualization: VM --> container

micro-service: SOA --> micro-service --> lambda architecture (千兆万兆网卡)

shared storage

amazon Aurora: shared storage，100% 兼容 MySQL，因为它只是替换了 MySQL 的存储层

cloud TiDB: tidb-operator (上层有 dashboard)

## PCTA-2001 TiDB 安装部署及集群管理

- 机器 os 要求：linux
- 机器硬件配置要求：cpu / memory / hard-disk (SSD) / 网卡 (万兆)
- 端口...

ansible: ansible playbook (yaml)

tidb-ansible

ansible online:

中控机：安装依赖，创建 tidb 用户，其它

inventory.ini

ansible offline: 主要是要自己下载好多安装包

docker compose:

part 2 - TiDB 的日常管理

监控：prometheus / grafana

工具和日志

- tools
  - PD control
  - TiKV control
  - TiDB control
- Logs
  - TiDB logs
  - TiKV logs
  - PD logs
  - ansible logs

扩缩容的操作

大致了解一下

## PCTA-2002 TiDB 基本管理及监控

TiDB 的启停操作： ansible / ssh

TiDB log: `curl -X POST -d "tidb_general_log=1|0" http://ip/settings` // 开启关闭 log

调整参数：ansible yml

DML/DDL/用户权限管理: 兼容 MySQL 语法

统计信息维护

- show stats_healthy
- analyze table table_name [index idx_name]
- ...

TiDB 专用操作：调整 GC 时间

part 2 - 监控

访问监控，配置在 ansible 中 ip:3000 admin/admin

监控指标详解：

告警及处理办法：

## PCTA-2003 备份恢复，数据迁移

skip

## PCTA-2004 高可用原理及扩缩容操作

skip

## PCTA-2005 TiDB 业务开发最佳实践

事物 - TiDB 使用乐观锁 + mvcc 不能克服写偏斜异常

这些场景的解决办法，使用正确的 sql

计数器/秒杀等场景不适合用 TiDB，最好是转移到 redis 中处理

嵌套事物：

大事物：拆分

自增 ID：只保证唯一且自增，不保证连续

唯一性约束，不支持外键

其它的先不看了

---

CockroachDB

- [Install CockroachDB on Mac](https://www.cockroachlabs.com/docs/stable/install-cockroachdb-mac.html)
- [Start a Local Cluster (Insecure)](https://www.cockroachlabs.com/docs/stable/start-a-local-cluster.html)

启动 cluster，start 三个节点：

```sh
$ cockroach start --insecure --store=node1 --listen-addr=localhost:26257 --http-addr=localhost:8080 --join=localhost:26257,localhost:26258,localhost:26259 --background

$ cockroach start --insecure --store=node2 --listen-addr=localhost:26258 --http-addr=localhost:8081 --join=localhost:26257,localhost:26258,localhost:26259 --background

$ cockroach start --insecure --store=node3 --listen-addr=localhost:26259 --http-addr=localhost:8082 --join=localhost:26257,localhost:26258,localhost:26259 --background
```

init：

```sh
cockroach init --insecure --host=localhost:26257
```

使用内置 sql client：

```sh
$ cockroach sql --insecure --host=localhost:26257

> CREATE DATABASE bank;
> CREATE TABLE bank.accounts (id INT PRIMARY KEY, balance DECIMAL);
> INSERT INTO bank.accounts VALUES (1, 1000.50);
> SELECT * FROM bank.accounts;
> \q
```

访问 Admin UI：http://localhost:8080

停止 cluster：

```sh
$ cockroach quit --insecure --host=localhost:26257
$ cockroach quit --insecure --host=localhost:26258
$ cockroach quit --insecure --host=localhost:26259
```
