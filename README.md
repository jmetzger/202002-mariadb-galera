# MySQL - Administration / Galera - Training (codership)  

```
We are working with Centos 7 here 
```

## Agenda 

  1. Architecture of the MySQL - Server
 
  1. Installation

  1. Configuration
 
  1. Security and User-Rights
 
  1. Locking 

  1. Database objects 

  1. Monitoring 

  1. Performance Optimization 

  1. Backups 

  1. MySQL - Galera - Cluster 


## Installation

### 2.1 Centos 7 - Repo - Configuration  

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

### 2.2 Installation 

```
yum install galera-4 and mysql-wsrep-8.0
# get the temporary password 
cat /var/log/mysqld.log | grep "password.*generated"
```

### 2.3 Configuration SELinux  

```
# Open everything that is needed 
# setools provide command semanage
yum provides semanage
yum install setools 
# Somehow redundant when you open the service altogether
# With mysqld_t 
# semanage port -a -t mysqld_port_t -p tcp 3306
# semanage port -a -t mysqld_port_t -p tcp 4444
# semanage port -a -t mysqld_port_t -p tcp 4567
# semanage port -a -t mysqld_port_t -p udp 4567
# semanage port -a -t mysqld_port_t -p tcp 4568
semanage permissive -a mysqld_t
```

### 2.4 Firewall 

```
# redundant - next line
# firewall-cmd --zone=public --add-service=mysql --permanent
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --zone=public --add-port=4444/tcp --permanent
firewall-cmd --zone=public --add-port=4567/tcp --permanent
firewall-cmd --zone=public --add-port=4567/udp --permanent
firewall-cmd --zone=public --add-port=4568/tcp --permanent
firewall-cmd --reload
```

## MySQL Galera Cluster 

###  MariaDB Galera Cluster Architecture

#### Technical Structure of the cluster

##### Overview

{{ ::replicationapi.png?nolink&300 |}}

#####  State Changes (e.g. Update of data)

 1.  On one node in the cluster, a state change occurs on the database.
 2.  In the database, the wsrep hooks translate the changes to the write-set.
 3.  dlopen() makes the wsrep provider functions available to the wsrep hooks.
 4.  The Galera Replication plugin handles write-set certification and replication to the cluster

##### SST / IST

###### State Snaphost Transfer (SST)


*  Full Transfer is done

*  Methods: rsync, xtrabackup, mariadback, mysqldump, rsync

*  Some are blocking, some are not (let's see later)

###### Incremental State Transfer (IST)
