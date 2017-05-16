---
title: 允许外网访问 MySQL
date: 2017-05-16 17:16:26
tags: DB
---

1. user 表中 User **root** 的 Host 为 **%**:

  ```sql
  USE mysql;
  SELECT user, host FROM user;
  UPDATE user SET host='%' WHERE user='root';
  FLUSH PRIVILEGES;
  ```

2. 授权用户

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

     ​

