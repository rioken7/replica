# Master-slave replica

* And slave to master(optional)

**Identity-server**

```sql
Master server: 192.168.1.181
Slave server: 192.168.1.182
```

**_Install MySQL on Master Nodes_**

```sql
sudo apt update && sudo apt install mysql-server
```

**_Open the mysql configuration file on the Master node._**

`Edit`

```sql
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

```sql
# Replication Settings
server-id                       = 1
log_bin                         = /var/lib/mysql/binlog/mysql-bin.log
relay-log                       = /var/lib/mysql/binlog/mysql-relay-bin.log
binlog_format                   = ROW
sync_binlog                     = 1
max_binlog_size                 = 100M
binlog_expire_logs_seconds      = 604800

# Apps needs: Enable log_bin_trust_function_creators
log_bin_trust_function_creators = 1
```

**_save the configuration file and restart MySQL service for the changes to take effect on Master node_**

```sql
sudo systemctl restart mysql && sudo systemctl status mysql
```

**_Create a New User for Replication on Master Node_**

_log in to the MySQL master-server as shown_.

```sql
sudo mysql -u root -p
```

pass: xxxxxxx

_Next, proceed and execute the queries below to create a replica user and grant access to the replication slave. Remember to use your IP address._

```sql
CREATE USER 'demo'@'192.168.xxx.xx' IDENTIFIED BY 'Demo@12pass';  #(add slave ip_address here in remote-users)
```

```sql
GRANT REPLICATION SLAVE ON *.* TO 'demo'@'192.168.137.69';  #(add slave ip_address here in remote-users)
```

```sql
FLUSH PRIVILEGES;
```

```sql
SHOW MASTER STATUS;
```

Note the `mysql-bin.000001` value and the Position ID `xxxx`. These values will be crucial when setting up the slave server.

**slave-servers**

**_Install MySQL on Slave Nodes_**

```sql
sudo apt update && sudo apt install mysql-server
```

**_Open the mysql configuration file on the Slave node._**

`Edit`

```sql
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

```sql
# Replication Settings
server-id                       = 2
read_only                       = 1
skip-slave-start                = 1
log_bin                         = /var/lib/mysql/binlog/mysql-bin.log
relay-log                       = /var/lib/mysql/binlog/mysql-relay-bin.log
binlog_format                   = ROW
sync_binlog                     = 1
max_binlog_size                 = 100M
binlog_expire_logs_seconds      = 604800 # 604800 => 7 days
```

**_save the configuration file and restart MySQL service for the changes to take effect on Slave node_**

```sql
sudo systemctl restart mysql && sudo systemctl status mysql
```

_log in to the MySQL server as shown_.

```sql
sudo mysql -u root -p
```

pass: xxxxxxx

**_stop mysql-replication in slave server_**

```sql
STOP SLAVE;
```

**_ADD the cmd lines in Slave node one by one _**

```sql
CHANGE REPLICATION SOURCE TO SOURCE_HOST='192.168.1.181',
SOURCE_USER='redbusr',
SOURCE_PASSWORD='password',
SOURCE_LOG_FILE='mysql-bin.000001',
SOURCE_LOG_POS=157,
GET_MASTER_PUBLIC_KEY=1;
```
**OR**
```sql
CHANGE REPLICATION SOURCE TO SOURCE_HOST='192.168.1.181',
SOURCE_USER='redbusr',
SOURCE_LOG_FILE='mysql-bin.000001',
SOURCE_LOG_POS=157,
GET_MASTER_PUBLIC_KEY=1;
```

**_start the slave replication slave server_**

```sql
START SLAVE;
```

**_show master-slave replica status in slave server_**

```sql
SHOW REPLICA STATUS\G;
```

```sql

*************************** 1. row ***************************
             Replica_IO_State: Waiting for source to send event
                    Source_Host: 192.168.1.182
                   Source_User: demo
                    Source_Port: 3306
                Connect_Retry: 60
             Source_Log_File: mysql-bin.000001
  Read_Source_Log_Pos: 1114
               Relay_Log_File: mysql-relay-bin.000002
               Relay_Log_Pos: 326
       lay_Source_Log_File: mysql-bin.000001
            plica_IO_Running: Yes
         plica_SQL_Running: Yes

```

**Testing the configuration in both master-slave servers**

**_IN Master-server_**

Create Database in master-server

```sql
CREATE DATABASE kendanic;
```

```sql
SHOW DATABASES;
```

**_IN slave-server_**

```cmd
sudo mysql
```

check if database created in master-server appers in slave-server

```sql
SHOW DATABASES;
```



**_promote the slave to master_**

* Stop Replication on the Slave

```sql
STOP REPLICA;
```

* Check the bin-log and position

SHOW MASTER STATUS;

**Reconfigure the Slaves if you have any other slave**

* Add the host_ip of new master ip(slave 1)

```sql
STOP REPLICA;
```
```sql
CHANGE REPLICATION SOURCE TO MASTER_HOST='192.168.1.182',
MASTER_USER='redbusr',
MASTER_PASSWORD='password',
MASTER_LOG_FILE='mysql-bin.000003',
MASTER_LOG_POS=457,
GET_MASTER_PUBLIC_KEY=1;
```
```sql
START REPLICA;
```

**Check slave status in other slaves**

```sql
SHOW REPLICA STATUS\G;
```
