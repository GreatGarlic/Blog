---
title: 允许其他机器访问 MySQL
date: 2017-05-16 17:16:26
tags: DB
---
A 机器上的 MySQL 默认只能 A 机器上的软件访问，即 localhost，如果 B 机器上的软件想访问 A 机器上的 MySQL，需要 MySQL 对 B 机器的 IP 进行授权。

## 方式一

* 任意主机以用户 **root** 和密码 **root** 连接到 MySQL 服务器

  ```sql
  GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;
  FLUSH PRIVILEGES;
  ```

* 指定 IP 为（如192.168.10.186）的主机以用户 **alice** 和密码 **Passw0rd** 连接到 MySQL 服务器

  ```sql
  GRANT ALL PRIVILEGES ON *.* TO 'alice'@'192.168.10.186' IDENTIFIED BY 'Passw0rd' WITH GRANT OPTION; 
  FLUSH PRIVILEGES;
  ```

## 方式二

网上还看到说直接修改 user 表中 User **root** 的 Host 为 **%**，最好别这么干，不小心会哭的:

```sql
USE mysql;
SELECT user, host FROM user;
UPDATE user SET host='%' WHERE user='root';
FLUSH PRIVILEGES;
```
按照上面的修改 host 为 % 后外网可以访问了，但是本地却访问出错:

```
mysql -uroot -p 
提示
Access denied for user 'root'@'localhost' (using password: YES) when trying
```

可用按下面的方式补救:

```
1. 启动 mysqld_safe
   mysqld_safe --user=mysql --skip-grant-tables --skip-networking &
2. 登陆修改
   mysql -u root mysql
   use mysql
   UPDATE user SET host='localhost' WHERE user='root';
   FLUSH PRIVILEGES;
   quit

这时可以看到 user 中关于 root 的记录会多一条
```



