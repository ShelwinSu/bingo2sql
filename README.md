# bingo2sql
MySQL Binlog 解析工具

从MySQL binlog解析出原始SQL，对应的回滚SQL等。

#### 功能说明

- 本地离线解析：指定本地binlog文件和要解析的表结构即可
- 远程在线解析：指定远程数据库地址，起止时间范围或binlog范围，可指定库/表和操作类型，GTID/线程号等
- 解析服务API：提供HTTP协议方式的解析接口，支持解析和打包下载

#### 限制和要求

- MySQL必须开启binlog
- binlog_format = row
- binlog_row_image = full

#### 测试对比

测试步骤及结束详见 [效率测试](docs/test.md)


### 支持模式

#### 1. 本地解析
  经过测试，可以恢复最新的MySQL8.0.22 版本下的数据误删除恢复。
```sh
bingo2sql --start-file=~/db_cmdb/blog/mysql-bin.000001 -t table.sql
```
其中`-t`参数指定的是建表语句文件,内容类似:
```sql
-- 需要解析哪个表,提供哪个表的建表语句
CREATE TABLE `tt` (
  id int auto_increment primary key,
  `TABLE_NAME` varchar(64) NOT NULL DEFAULT ''
) ;
```


#### 2. 远程解析

远程解析的参数及使用均与binlog2sql类似

```
bingo2sql -h=127.0.0.1 -P 3306 -u test -p test -d db1_3306_test_inc \
  --start-time="2006-01-02 15:04:05" -t t1 -B
```

#### 3. 解析服务

bingo2sql 支持以服务方式运行,提供解析的HTTP接口支持

```sh
bingo2sql --server --config=config.ini
```


### 支持选项

**解析模式**

- --stop-never 持续解析binlog。可选。默认false，同步至执行命令时最新的binlog位置。

- -K, --no-primary-key 对INSERT语句去除主键。可选。默认false

- -B, --flashback 生成回滚SQL，可解析大文件，不受内存限制。可选。默认false。与stop-never或no-primary-key不能同时添加。

- -M, --minimal-update 最小化update语句. 可选. (default true)

- -I, --minimal-insert 使用包含多个VALUES列表的多行语法编写INSERT语句. (default true)

**解析范围控制**

- --start-file 起始解析文件，只需文件名，无需全路径 。可选，如果指定了起止时间，可以忽略该参数。

- --start-pos 起始解析位置。可选。默认为start-file的起始位置。

- --stop-file 终止解析文件。可选。若解析模式为stop-never，此选项失效。

- --stop-pos 终止解析位置。可选。默认为stop-file的最末位置；若解析模式为stop-never，此选项失效。

- --start-time 起始解析时间，格式'%Y-%m-%d %H:%M:%S'等。可选。默认不过滤。

- --stop-time 终止解析时间，格式'%Y-%m-%d %H:%M:%S'等。可选。默认不过滤。

- -C, --connection-id 线程号，可解析指定线程的SQL。

- -g, --gtid GTID范围.格式为uuid:编号[-编号],多个时以逗号分隔，例如：6573bb29-9d94-11e9-9e0c-0242ac130002:1-100

- --max 解析的最大行数,设置为0则不限制，以避免解析范围过大 (default 100000)

**对象过滤**

-d, --databases 只解析目标db的sql，多个库用逗号隔开，如-d db1,db2。可选。默认为空。

-t, --tables 只解析目标table的sql，多张表用逗号隔开，如-t tbl1,tbl2。可选。默认为空。

--ddl 解析ddl，仅支持正向解析。可选。默认false。

--sql-type 只解析指定类型，支持 insert,update,delete。多个类型用逗号隔开，如--sql-type=insert,delete。可选。默认为增删改都解析。


**附加信息**

- --show-gtid            显示gtid (default true)

- --show-time            显示执行时间,同一时间仅显示首次 (default true)

- --show-all-time        显示每条SQL的执行时间 (default false)

- --show-thread          显示线程号,便于区别同一进程操作 (default false)

- -o, --output          本地或远程解析时，可输出到指定文件(置空则输出到控制台，可通过 > file重定向)

**mysql连接配置** (仅远程解析需要)

```
 -h host
 -P port
 -u user
 -p password
```

#### 致谢
    bingo2sql借鉴和学习了很多业界知名的开源项目，在此表示感谢！
- [go-mysql](https://github.com/siddontang/go-mysql) handle MySQL network protocol and replication
- [binlog2sql](https://github.com/danfengcao/binlog2sql) Parse MySQL binlog to SQL you want
- [binlog_rollback](https://github.com/GoDannyLai/binlog_rollback) mysql binlog rollback | flashback | redo | dml report | ddl info
