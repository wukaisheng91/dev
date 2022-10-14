
# MYSQL 主从

### 主
``` bash
create user 'copy'@'%' IDENTIFIED BY 'copy123456';
grant replication slave on *.* to 'copy'@'10.10.10.%';
flush PRIVILEGES;
```
* 需要测试是否可以连接得上 *
### 查看同步的bin_log 文件与 同步的位置

show master status;

### 从
#### 配置文件

``` my.cnf
[mysqld]
server-id = 2
replicate-ignore-db=mysql
replicate-ignore-db=sys

```

#### 防止大SQL 报错
```
show VARIABLES like '%max_allowed_packet%';
set gloabeglobal  max_allowed_packet = 1000 * 原来的;
```

#### 设置主机地址密码

``` mysql
change master to
master_host='10.10.10.151',
master_port=3306,
master_user='copy',
master_password='copy123456',
master_log_file='mysql-bin.000010',
master_log_pos=1955;
```
#### 开启同步
start slave;
show slave status \G

#### 停止同步
stop slave;