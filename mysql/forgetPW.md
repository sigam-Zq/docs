# mysql忘记密码

数据库

```bash
service mysqld stop


vim /etc/my.cnf.d/mysql-server.cnf

# 添加下面的内容

#在[mysqld] 下面新增 

# 跳过密码验证
skip-grant-tables


service mysqld start 

# 过程如下

[root@iZ8vb4qrjoahmfx2oe3ujvZ crmservice]# mysql -uroot
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 7
Server version: 8.0.26 Source distribution

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> update mysql.user set authentication_string=password('12345678') where user='root';
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '('12345678') where user='root'' at line 1
mysql> ALTER user 'root'@'localhost' IDENTIFIED BY '12345678';
ERROR 1290 (HY000): The MySQL server is running with the --skip-grant-tables option so it cannot execute this statement
mysql> exit
Bye


```


* 这里有一个小坑

mysql 5 版本的可以使用 password 关键字 和update 进行修改密码 但是 版本8 之后不同了

https_blog.csdn.net/?url=https%3A%2F%2Fblog.csdn.net%2Fwolf131721%2Farticle%2Fdetails%2F93004013

* 第二个小坑
这里mysql 登录应该使用 跳过密码登录了 但是没生效
<p>
如下


```bash

[root@iZ8vb4qrjoahmfx2oe3ujvZ crmservice]# mysql -uroot
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 7
Server version: 8.0.26 Source distribution

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
mysql>
mysql>
mysql> ALTER user 'root'@'localhost' IDENTIFIED BY '12345678';
ERROR 1290 (HY000): The MySQL server is running with the --skip-grant-tables option so it cannot execute this statement
mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)

mysql> ALTER user 'root'@'localhost' IDENTIFIED BY '12345678';
Query OK, 0 rows affected (0.00 sec)



```

* 这里 再次去进行登录就需要重新输入密码了就

