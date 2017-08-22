# MySQL主从配置(主库无写入情况)
以主备都在一台机器上为例,平台为windows,主备库都在D盘,分别名为mysql和mysql-slave

1. 主库上创建复制账号
```
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'repl'@'192.168.1.151' IDENTIFIED BY 'password';
```

2. 修改主库和备库配置文件
master的my.ini(linux为my.cnf)
```
log-bin=D:/mysql/mysql-bin
server_id=488 #server_id不要和slave重复即可
sync_binlog=1 #MySQL提交事务前会将二进制日志同步到磁盘上
```

slave的my.ini
```
log-bin=D:/mysql-slave/mysql-bin
server_id=489
relay_log=D:/mysql-slave/mysql-relay-bin #中继日志位置
read_only=1
log_slave_updates=1
```

3. 启动复制
```
CHANGE MASTER TO MASTER_HOST='192.168.1.152',   
       MASTER_USER='repl',   
       MASTER_PASSWORD='password',   
       MASTER_LOG_FILE='mysql-bin.000001',   
       MASTER_LOG_POS=8;  
```
MASTER_LOG_FILE和MASTER_LOG_POS可以通过show master status查看  
开始复制 
```
START SLAVE
```

4. 查看复制状态
```
SHOW SLAVE STATUS\G
```
可以看到(举例)
```
*************************** 1. row ***************************

Slave_IO_State: Waiting for master to send event
Master_Host: 192.168.1.152
Master_User: repl
Master_Port: 3306
Connect_Retry: 60
Master_Log_File: mysql-bin.000001
Read_Master_Log_Pos: 8s     //#同步读取二进制日志的位置，大于等于Exec_Master_Log_Pos
Relay_Log_File: mysql-relay-bin.000001
Relay_Log_Pos: 251
Relay_Master_Log_File: mysql-bin.000001
Slave_IO_Running: Yes 
Slave_SQL_Running: Yes 
```
当Slave_IO_Running和Slave_SQL_Running都是Yes时,表示复制正在运行

# 配置主从(有写入数据时，即线上的运行的db)
1. FLUSH TABLES WITH READ LOCK(FTWRL);
主数据库锁定所有表操作, 不让数据再进行写入操作。该命令主要用于备份工具获取一致性备份(数据与binlog位点匹配)。由于FTWRL总共需要持有两把全局的MDL锁，并且还需要关闭所有表对象，因此这个命令的杀伤性很大，执行命令时容易导致库hang住。  FTWRL主要包括3个步骤：

    * 上全局读锁(lock_global_read_lock)
    * 清理表缓存(close_cached_tables)
    * 上全局COMMIT锁(make_global_read_lock_block_commit)

2. 查看主数据库的状态
```
mysql> show master status;
```
这里跟第一种情况一样，查看日志File和Position的值, 以备从服务器使用。

3. 把主服务器的数据文件复制到从服务器
```
mysql -u root -p db_dev < db_dev.sql
```

4. 取消主数据库锁定
```
mysql> UNLOCK TABLES;
```