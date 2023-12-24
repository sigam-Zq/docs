# mysql


小tips


* 在数据库中执行脚本


```bash

[root@iZ8vb4qrjoahmfx2oe3ujvZ crm]# mysql -uroot -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.26 Source distribution

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> source /root/crm/crminit.sql
Query OK, 1 row affected (0.01 sec)

Database changed
Query OK, 0 rows affected, 1 warning (0.04 sec)

Query OK, 0 rows affected (0.03 sec)
Records: 0  Duplicates: 0  Warnings: 0

Query OK, 0 rows affected, 4 warnings (0.04 sec)

Query OK, 0 rows affected, 6 warnings (0.05 sec)

Query OK, 0 rows affected, 1 warning (0.03 sec)

Query OK, 0 rows affected, 4 warnings (0.02 sec)

Query OK, 0 rows affected, 2 warnings (0.04 sec)

Query OK, 0 rows affected, 1 warning (0.05 sec)

Query OK, 0 rows affected, 1 warning (0.04 sec)

Query OK, 0 rows affected, 2 warnings (0.03 sec)

Query OK, 0 rows affected, 1 warning (0.03 sec)

mysql>
mysql>
mysql>
mysql> exit
Bye
```
