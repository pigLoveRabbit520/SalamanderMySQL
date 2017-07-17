# MySQL主从配置
以主备都在一台机器上为例,平台为windows,主备库都在D盘,分别名为mysql和mysql-slave

1. 主库上创建复制账号
```
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'repl'@'127.0.0.1' IDENTIFIED BY 'password';
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
Master_Host: 127.0.0.1
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