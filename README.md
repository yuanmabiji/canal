**注意**：经过初步测试，deployer服务挂了，那么binlog无法同步;deployer服务没挂，canalAdapterApplication挂了，那么重启后还是能同步之前的binlog。

canal源码环境搭建步骤：
# 准备
对于自建 MySQL , 需要先开启 Binlog 写入功能，配置 binlog-format 为 ROW 模式，my.cnf 中配置如下

[mysqld]
log-bin=mysql-bin # 开启 binlog
binlog-format=ROW # 选择 ROW 模式
server_id=1 # 配置 MySQL replaction 需要定义，不要和 canal 的 slaveId 重复
注意：针对阿里云 RDS for MySQL , 默认打开了 binlog , 并且账号默认具有 binlog dump 权限 , 不需要任何权限或者 binlog 设置,可以直接跳过这一步
授权 canal 链接 MySQL 账号具有作为 MySQL slave 的权限, 如果已有账户可直接 grant

CREATE USER canal IDENTIFIED BY 'canal';  
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
-- GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%' ;
FLUSH PRIVILEGES;

# 启动
## 1 修改E:\canal\deployer\src\main\resources\example\instance.properties配置文件下面的配置

canal.instance.mysql.slaveId=123 #伪装成从库id

canal.instance.master.address=x127.0.0.1:3306 #要监听的Mysql主库（可以远程监听，即canal服务器不跟mysql主库部署在同一台机器）

canal.instance.dbUsername=canal #username/password（跟Mysql授权的用户一致）

canal.instance.dbPassword=canal
## 2 启动CanalLauncher

# 同步源mysql库到目标mysql库
1. 修改E:\canal\client-adapter\launcher\src\main\resources\application.yml
2. 增加E:\canal\client-adapter\launcher\src\main\resources\rdb\mytest_user.yml配置文件
3. 启动CanalAdapterApplication即可。

# client example调试
启动SimpleCanalClientTest

# 部署admin客户端
1. 执行脚本：E:\canal\admin\admin-web\src\main\resources\canal_manager.sql
2. 修改mysql连接配置，安装Ebean Enhancement插件并重启，并右键admin-web项目开启
3. 启动CanalAdminApplication




[![build status](https://travis-ci.com/alibaba/canal.svg?branch=master)](https://travis-ci.com/alibaba/canal)
[![codecov](https://codecov.io/gh/alibaba/canal/branch/master/graph/badge.svg)](https://codecov.io/gh/alibaba/canal)
![maven](https://img.shields.io/maven-central/v/com.alibaba.otter/canal.svg)
![license](https://img.shields.io/github/license/alibaba/canal.svg)
[![average time to resolve an issue](http://isitmaintained.com/badge/resolution/alibaba/canal.svg)](http://isitmaintained.com/project/alibaba/canal "average time to resolve an issue")
[![percentage of issues still open](http://isitmaintained.com/badge/open/alibaba/canal.svg)](http://isitmaintained.com/project/alibaba/canal "percentage of issues still open")

## 简介

![](https://img-blog.csdnimg.cn/20191104101735947.png)

**canal [kə'næl]**，译意为水道/管道/沟渠，主要用途是基于 MySQL 数据库增量日志解析，提供增量数据订阅和消费

早期阿里巴巴因为杭州和美国双机房部署，存在跨机房同步的业务需求，实现方式主要是基于业务 trigger 获取增量变更。从 2010 年开始，业务逐步尝试数据库日志解析获取增量变更进行同步，由此衍生出了大量的数据库增量订阅和消费业务。

基于日志增量订阅和消费的业务包括
- 数据库镜像
- 数据库实时备份
- 索引构建和实时维护(拆分异构索引、倒排索引等)
- 业务 cache 刷新
- 带业务逻辑的增量数据处理

当前的 canal 支持源端 MySQL 版本包括 5.1.x , 5.5.x , 5.6.x , 5.7.x , 8.0.x

## 工作原理

#### MySQL主备复制原理
![](http://dl.iteye.com/upload/attachment/0080/3086/468c1a14-e7ad-3290-9d3d-44ac501a7227.jpg)

- MySQL master 将数据变更写入二进制日志( binary log, 其中记录叫做二进制日志事件binary log events，可以通过 show binlog events 进行查看)
- MySQL slave 将 master 的 binary log events 拷贝到它的中继日志(relay log)
- MySQL slave 重放 relay log 中事件，将数据变更反映它自己的数据

#### canal 工作原理

- canal 模拟 MySQL slave 的交互协议，伪装自己为 MySQL slave ，向 MySQL master 发送dump 协议
- MySQL master 收到 dump 请求，开始推送 binary log 给 slave (即 canal )
- canal 解析 binary log 对象(原始为 byte 流)

## 重要版本更新说明

1. canal 1.1.x 版本（[release_note](https://github.com/alibaba/canal/releases)）,性能与功能层面有较大的突破,重要提升包括:

- 整体性能测试&优化,提升了150%. #726 参考: [Performance](https://github.com/alibaba/canal/wiki/Performance)
- 原生支持prometheus监控 #765 [Prometheus QuickStart](https://github.com/alibaba/canal/wiki/Prometheus-QuickStart)
- 原生支持kafka消息投递 #695 [Canal Kafka/RocketMQ QuickStart](https://github.com/alibaba/canal/wiki/Canal-Kafka-RocketMQ-QuickStart)
- 原生支持aliyun rds的binlog订阅 (解决自动主备切换/oss binlog离线解析) 参考: [Aliyun RDS QuickStart](https://github.com/alibaba/canal/wiki/aliyun-RDS-QuickStart)
- 原生支持docker镜像 #801 参考: [Docker QuickStart](https://github.com/alibaba/canal/wiki/Docker-QuickStart)

2.  canal 1.1.4版本，迎来最重要的WebUI能力，引入canal-admin工程，支持面向WebUI的canal动态管理能力，支持配置、任务、日志等在线白屏运维能力，具体文档：[Canal Admin Guide](https://github.com/alibaba/canal/wiki/Canal-Admin-Guide)

## 文档

- [Home](https://github.com/alibaba/canal/wiki/Home)
- [Introduction](https://github.com/alibaba/canal/wiki/Introduction)
- [QuickStart](https://github.com/alibaba/canal/wiki/QuickStart)
  - [Docker QuickStart](https://github.com/alibaba/canal/wiki/Docker-QuickStart)
  - [Canal Kafka/RocketMQ QuickStart](https://github.com/alibaba/canal/wiki/Canal-Kafka-RocketMQ-QuickStart")
  - [Aliyun RDS for MySQL QuickStart](https://github.com/alibaba/canal/wiki/aliyun-RDS-QuickStart)
  - [Prometheus QuickStart](https://github.com/alibaba/canal/wiki/Prometheus-QuickStart)
- Canal Admin
  - [Canal Admin QuickStart](https://github.com/alibaba/canal/wiki/Canal-Admin-QuickStart)
  - [Canal Admin Guide](https://github.com/alibaba/canal/wiki/Canal-Admin-Guide)
  - [Canal Admin ServerGuide](https://github.com/alibaba/canal/wiki/Canal-Admin-ServerGuide)
  - [Canal Admin Docker](https://github.com/alibaba/canal/wiki/Canal-Admin-Docker)
- [AdminGuide](https://github.com/alibaba/canal/wiki/AdminGuide)
- [ClientExample](https://github.com/alibaba/canal/wiki/ClientExample)
- [ClientAPI](https://github.com/alibaba/canal/wiki/ClientAPI)
- [Performance](https://github.com/alibaba/canal/wiki/Performance)
- [DevGuide](https://github.com/alibaba/canal/wiki/DevGuide)
- [BinlogChange(MySQL 5.6)](https://github.com/alibaba/canal/wiki/BinlogChange%28mysql5.6%29)
- [BinlogChange(MariaDB)](https://github.com/alibaba/canal/wiki/BinlogChange%28MariaDB%29)
- [TableMetaTSDB](https://github.com/alibaba/canal/wiki/TableMetaTSDB)
- [ReleaseNotes](http://alibaba.github.com/canal/release.html)
- [Download](https://github.com/alibaba/canal/releases)
- [FAQ](https://github.com/alibaba/canal/wiki/FAQ)

## 多语言

canal 特别设计了 client-server 模式，交互协议使用 protobuf 3.0 , client 端可采用不同语言实现不同的消费逻辑，欢迎大家提交 pull request 
  
- canal java 客户端: [https://github.com/alibaba/canal/wiki/ClientExample](https://github.com/alibaba/canal/wiki/ClientExample)
- canal c# 客户端: [https://github.com/dotnetcore/CanalSharp](https://github.com/dotnetcore/CanalSharp)
- canal go客户端: [https://github.com/CanalClient/canal-go](https://github.com/CanalClient/canal-go)
- canal php客户端: [https://github.com/xingwenge/canal-php](https://github.com/xingwenge/canal-php)
- canal Python客户端：[https://github.com/haozi3156666/canal-python](https://github.com/haozi3156666/canal-python)
- canal Rust客户端：[https://github.com/laohanlinux/canal-rs](https://github.com/laohanlinux/canal-rs)

canal 作为 MySQL binlog 增量获取和解析工具，可将变更记录投递到 MQ 系统中，比如 Kafka/RocketMQ，可以借助于 MQ 的多语言能力 

- 参考文档: [Canal Kafka/RocketMQ QuickStart](https://github.com/alibaba/canal/wiki/Canal-Kafka-RocketMQ-QuickStart)

## 相关开源&产品

- [canal 消费端开源项目: Otter](http://github.com/alibaba/otter)
- [阿里巴巴去 Oracle 数据迁移同步工具: yugong](http://github.com/alibaba/yugong)
- [阿里巴巴离线同步开源项目 DataX](https://github.com/alibaba/datax)
- [阿里巴巴数据库连接池开源项目 Druid](https://github.com/alibaba/druid)
- [阿里巴巴实时数据同步工具 DTS](https://www.aliyun.com/product/dts)

## 问题反馈
- 报告 issue: [github issues](https://github.com/alibaba/canal/issues)
