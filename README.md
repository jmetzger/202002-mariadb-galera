# MySQL - Galera - Training 

## 1. Centos - Repo - Configuration  

  * https://galeracluster.com/library/documentation/install-mysql.html

```
# /etc/yum.repos.d/galera.repo
[galera]
name = Galera
baseurl = https://releases.galeracluster.com/galera-4/centos/7/x86_64/
gpgkey = https://releases.galeracluster.com/GPG-KEY-galeracluster.com
gpgcheck = 1

[mysql-wsrep]
name = MySQL-wsrep
baseurl =  https://releases.galeracluster.com/mysql-wsrep-8.0/centos/7/x86_64/
gpgkey = https://releases.galeracluster.com/GPG-KEY-galeracluster.com
gpgcheck = 1

```

## 2. Installation 


