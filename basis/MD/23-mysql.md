# MySQL

## 基本环境

| 主机名称     | 主机IP地址        | 备注        |
| ------------ | ----------------- | ----------- |
| mysql-master | 192.168.200.10/24 | MySQL主节点 |
| mysql-slave  | 192.168.200.11/24 | MySQL从节点 |

MySQL版本：5.7.35

## 下载MySQL

## 安装MySQL

## 部署主从MySQL

### 修改my.cnf文件

master节点使用vim打开my.cnf配置文件在`[mysqld]`下添加一下内容

```shell
honya@mysql-master:~$ sudo vim /etc/my.cnf
[mysqld]
# 开启二进制日志【必须开启】
log-bin=mysql-bin
# 服务器唯一ID，默认是1，一般取IP最后一段【必须设置，相对于唯一标识符】
server-id=111

local-infile=1
# 数据库数据存放位置
datadir=/data/mysql-5.7.35/data
# 数据库日志存储位置
log_error=/data/logs/mysql/mysqld.log

socket=/tmp/mysql.sock

character-set-server=utf8

sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

innodb_file_per_table=1

event_scheduler=ON

max_connections=3000

concurrent_insert = 2

open_files_limit = 65535

innodb_open_files=65535

binlog_cache_size = 1M

thread_cache_size = 256

innodb_open_files = 500

innodb_buffer_pool_size = 2048M

innodb_thread_concurrency = 0

innodb_purge_threads = 1

innodb_flush_log_at_trx_commit = 2

innodb_log_buffer_size = 8M

innodb_log_file_size = 64M

innodb_log_files_in_group = 3

innodb_max_dirty_pages_pct = 90

innodb_lock_wait_timeout = 120

innodb_write_io_threads = 6

innodb_read_io_threads = 4

bulk_insert_buffer_size = 32M

myisam_sort_buffer_size = 1024M

myisam_max_sort_file_size = 10G

myisam_repair_threads = 1

interactive_timeout = 28800

wait_timeout = 28800

key_buffer_size = 1024M
default_password_lifetime=0

long_query_time = 1
slow_query_log=/data/logs/mysql/slow_query.log

[client]

socket=/tmp/mysql.sock

```

slave节点使用vim打开my.cnf配置文件在`[mysqld]`下添加一下内容

```shell
honya@mysql-slave:~$ sudo vim /etc/my.cnf
[mysqld]
# 开启二进制日志【slave节点可不必开启】
log-bin=mysql-bin
# 服务器唯一ID，默认是1，一般取IP最后一段【必须设置，相对于唯一标识符】
server-id=222

local-infile=1
# 数据库数据存放位置
datadir=/data/mysql-5.7.35/data
# 数据库日志存储位置
log_error=/data/logs/mysql/mysqld.log

socket=/tmp/mysql.sock

character-set-server=utf8

sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

innodb_file_per_table=1

event_scheduler=ON

max_connections=3000

concurrent_insert = 2

open_files_limit = 65535

innodb_open_files=65535

binlog_cache_size = 1M

thread_cache_size = 256

innodb_open_files = 500

innodb_buffer_pool_size = 2048M

innodb_thread_concurrency = 0

innodb_purge_threads = 1

innodb_flush_log_at_trx_commit = 2

innodb_log_buffer_size = 8M

innodb_log_file_size = 64M

innodb_log_files_in_group = 3

innodb_max_dirty_pages_pct = 90

innodb_lock_wait_timeout = 120

innodb_write_io_threads = 6

innodb_read_io_threads = 4

bulk_insert_buffer_size = 32M

myisam_sort_buffer_size = 1024M

myisam_max_sort_file_size = 10G

myisam_repair_threads = 1

interactive_timeout = 28800

wait_timeout = 28800

key_buffer_size = 1024M
default_password_lifetime=0

long_query_time = 1
slow_query_log=/data/logs/mysql/slow_query.log

[client]

socket=/tmp/mysql.sock
```

### 重启服务

```shell
honya@master:~$ sudo service mysql restart
honya@slave:~$ sudo service mysql restart
```

### master节点建立帐户并授权slave

进入master节点MySQL_Server并创建同步用户以及授权

```mysql
honya@master:~$ mysql -uroot -p123456a
# 一般不用root帐号，如果将IP地址替换成“%”表示所有客户端都可能连，只要帐号，密码正确，此处可用具体客户端IP代替，如192.168.200.11，加强安全。
mysql> GRANT REPLICATION SLAVE ON *.* to 'slaveuse'@'192.168.200.11' identified by '123456a';
mysq> 
```

### 查看master的状态

```mysql
mysql> SHOW master STATUS;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000002 |      368 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

### 配置slave节点

```mysql
mysql> change master to master_host='192.168.200.10',master_user='mysync',master_password='q123456',master_log_file='mysql-bin.000002',master_log_pos=368;   //注意不要断开，308数字前后无单引号。
Mysql> start slave;    //启动从服务器复制功能
```

### 检查从服务器复制功能状态

```mysql
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.200.10		# master节点IP地址
                  Master_User: slaveuse				# 授权帐户名，尽量避免使用root
                  Master_Port: 3306					# 数据库端口，部分版本没有此行
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 368					#同步读取二进制日志的位置，大于等于Exec_Master_Log_Pos
               Relay_Log_File: server02-relay-bin.000003
                Relay_Log_Pos: 581					# 主服务器日志偏移量
        Relay_Master_Log_File: mysql-bin.000002		# 主服务器日志文件
             Slave_IO_Running: Yes					# IO线程此状态必须YES
            Slave_SQL_Running: Yes					# SQL线程此状态必须YES
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 368
              Relay_Log_Space: 957
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 111
                  Master_UUID: 4b4c54a4-bf9f-11ec-bdc2-00155d00f104
             Master_Info_File: /data/mysql-5.7.35/data/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)
```

### 测试主从是否同步

在master上创建一个新的数据库且创建数据库表的话slaver节点也会自动进行同步

### 关于解决Slave_SQL_Running: No

#### 问题原因

1. 程序可能在slave上进行了写操作。

2. 也可能是slave机器重起后，事务回滚所造成。

##### 解决方案一

一般是事务回滚所造成解决如下：

```mysql
# 停止slave服务
mysql> stop slave ;
# 尝试使用sql_slave_skip_counter跳过错误(实际遇到1062写入key冲突，我们应该根据 Duplicate entry 删除从库对应记录)
mysql> set GLOBAL SQL_SLAVE_SKIP_COUNTER=1;
# 再次启动slave
mysql> start slave ;
```

##### 解决方案二（推荐）

```mysql
首先停掉Slave服务：
mysql> slave stop
到master节点上查看主机状态：
记录File和Position对应的值
进入master节点执行

mysql> show master status;
+----------------------+----------+--------------+------------------+
| File                 | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+----------------------+----------+--------------+------------------+
| localhost-bin.000094 | 33622483 |              |                  | 
+----------------------+----------+--------------+------------------+
1 row in set (0.00 sec)

# 回到slave节点上执行手动同步、
# 注意master_log_file的值对应master节点输入show master status;所显示的File列的值要一致
# 注意master_log_pos的值对应master节点输入show master status;所显示的Position列的值要一致
mysql> change master to master_host='master_ip',master_user='user',master_password='pwd',master_port=3306,master_log_file='localhost-bin.000094',master_log_pos=33622483;
1 row in set (0.00 sec)
mysql> start slave ;
1 row in set (0.00 sec)
# 此时再次查看在slave节点上查看slvae的一个状态
mysql> show slave status\G
*************************** 1. row ***************************
            Master_Log_File: localhost-bin.000094
        Read_Master_Log_Pos: 33768775
             Relay_Log_File: localhost-relay-bin.000537
              Relay_Log_Pos: 1094034
      Relay_Master_Log_File: localhost-bin.000094
           Slave_IO_Running: Yes
          Slave_SQL_Running: Yes
            Replicate_Do_DB:
```

