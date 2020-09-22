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
# Show service mysql
systemctl list-unit-files -t service | grep mysql
systemctl start mysqld.service 
systemctl status mysqld.service 
# Nicht beim Starten vom Server starten 
systemctl disable mysqld.service 
systemctl enable mysqld.service 

# get the temporary password 
cat /var/log/mysqld.log | grep "password.*generated"
```

```
# findout if mysql listen to the outside
# look for *:mysql
lsof -i 
```

### 2.3 Configuration SELinux  

```
# Option 1
# Disable selinux at runtime 
sestatus
# switch from enforcing to permissive (runtime) 
setenforce 0
getenforce
sestatus

```


```
# Option 2
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
# Option 1
systemctl disable firewalld
systemctl stop firewalld 
```


```
# Option 2
# redundant - next line
# firewall-cmd --zone=public --add-service=mysql --permanent
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --zone=public --add-port=4444/tcp --permanent
firewall-cmd --zone=public --add-port=4567/tcp --permanent
firewall-cmd --zone=public --add-port=4567/udp --permanent
firewall-cmd --zone=public --add-port=4568/tcp --permanent
firewall-cmd --reload
```

## 3 Configuration 

### Best source 

https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html

## MySQL Galera Cluster 

###  MySQL Galera Cluster Architecture

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

*  A node only receives the missing write sets.

*  Is only done, when
    * The missing write-sets are in the gcache of the donor
    * Otherwice SST is done

##### Global Transaction Number


*  Global Transaction ID

*  In order to keep the state identical across the cluster, \\ the wsrep API uses a Global Transaction ID, or GTID. \\ This allows it to identify state changes and to identify the current state in relation to the last state change.

*  Example: 45eec521-2f34-11e0-0800-2a36050b826b:94530586304

#####  gcache(=write-set-cache) ==


*  Cache to save writesets

*  Used to perform IST

#### How do out-of-sync Nodes resynchronize ?

#### When does a cluster make sense and when not ?

##### PRE-Requisite


*  The primary focus of Galera Cluster is data consistency across the nodes.

#####  DOES NOT: Application not ready



*  If application has a lot of hotspots, writing to all nodes will be critical

*  e.g. update counter set counter=counter+1; # Table counter having just one row

*  This will produce a lot of "DEADLOCKS".

##### DOES NOT: InnoDB is not used as engine


*  For MariaDB Cluster to work properly InnoDB-Engine is needed

*  MyISAM is currently experimental.. does not support DML's (e.g. INSERT into ....)

#####  DOES NOT: Windows is used


*  MariaDB Cluster does not work on Windows

##### DOES NOT: Application heavily uses Query_Cache


*  If you heavily rely on Query Cache you should first \\ fix the queries and redesign your application. \\ There is experimental Query Cache support since Galera v3.1. \\ But the Galera documentation clearly states: Do not use query cache..

#####  Refs


*  https://www.fromdual.com/limitations-of-galera-cluster

#### Limitations

```"Transaction size. While Galera does not explicitly limit the transaction size, a writeset is processed as a single memory-resident buffer and as a result, extremely large transactions (e.g. LOAD DATA) may adversely affect node performance. To avoid that, the wsrep_max_ws_rows and wsrep_max_ws_size system variables limit transaction rows to 128K and the transaction size to 1Gb by default. If necessary, users may want to increase those limits. Future versions will add support for transaction fragmentation."
```

#### Structure of the Configuration


*  Discuss the layers.

#### Basic Configuration of the galera cluster


```
# Ubuntu/Debian

# /etc/mysql/conf.d/galera.cnf
# Centos/Redhat

# /etc/my.cnf.d/99_galera.cnf
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0

# Set to 1 sec instead of per transaction

# for better performance // Attention: You might loose data on power outage
innodb_flush_log_at_trx_commit=0

# Galera Provider Configuration

wsrep_on=ON
# ubuntu

wsrep_provider=/usr/lib/galera/libgalera_smm.so
# centos7 (x86_64)

wsrep_provider=/usr/lib64/galera/libgalera_smm.so

# Galera Cluster Configuration

wsrep_cluster_name="test_cluster"
wsrep_cluster_address="gcomm://first_ip,second_ip,third_ip"

# Galera Synchronization Configuration

wsrep_sst_method=rsync
```

##### Notes on: innodb_autoinc_lock_mode


*  Value 0 and 1 can cause a lot of table-locks

*  Value 2 (=interleaved) allows the system to insert in parallel (better performance)
    * !! Works only with ROW-Based replication

*  `<code>`From galera manual
innodb_autoinc_lock_mode=2
Do not change this value.

Other modes may cause INSERT statements on tables
with AUTO_INCREMENT columns to fail.

Note Warning: When innodb_autoinc_lock_mode is set
to traditional lock mode, indicated by 0, or to consecutive lock mode,
indicated by 1, in Galera Cluster it can cause unresolved deadlocks
and make the system unresponsive.

`</code>`

#### First going live ;o)

##### Node 1


        The first server needs to be started without searching for the other servers
        # on systemd - systems  it is easy
        # execute this script
        # in case mysql is running
        systemctl stop mysql
        galera_new_cluster
        # after that check, if it is running
        systemctl status mysql


##### Node 1: Check cluster size

     mysql -uroot -p
        show status like 'wsrep_cluster_size';
        +--------------------+-------+
        | Variable_name      | Value |
        +--------------------+-------+
        | wsrep_cluster_size | 1     |
        +--------------------+-------+
        1 row in set (0.001 sec)


##### Node 2


        # mariadb 10.3 starts server directly after installation
        # so stop it first
        systemctl stop mysql

        # galera.cnf configuration must be in place
        systemctl start mysql


##### Node 2: Check status


        # Is it part of the cluster now ?
        # mysql -uroot -p
        show status like 'wsrep_cluster_size'
        # is it synced (so it has all the data from the other node
        show status like 'wsrep_local_state_comment'


#####  Node 3

