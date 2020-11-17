# Server konfigurieren 

## Mit Restart in /etc/my.cnf.d/server.cnf 

```
Step 1: config - eintrag unterhalb von [mysqld] schreiben 
Step 2: systemctl restart mariadb 
```

## Während der Laufzeit 

```
# mit mysql-client connecten 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 13
Server version: 10.4.17-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> set session log_output=NONE;
ERROR 1229 (HY000): Variable 'log_output' is a GLOBAL variable and should be set with SET GLOBAL
MariaDB [(none)]> set global log_output=NONE;
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]>
```
