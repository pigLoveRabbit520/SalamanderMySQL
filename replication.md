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
       MASTER_LOG_POS=0;  
```
MASTER_LOG_FILE和MASTER_LOG_POS可以通过show master status查看
