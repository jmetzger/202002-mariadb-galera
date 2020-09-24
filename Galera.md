## 10 MySQL Galera Cluster 

###  MySQL Galera Cluster Architecture

#### Technical Structure of the cluster

##### Overview

![Overview](images/replicationapi.png)

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
# centos 7 with mysql from codership 
# Seems not be there in last line 
echo "!includedir /etc/my.cnf.d/" >> /etc/my.cnf 

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

# codership / mysql 8 
wsrep_provider=/usr/lib64/galera-4/libgalera_smm.so

# Galera Cluster Configuration

wsrep_cluster_name="test_cluster_"
wsrep_cluster_address="gcomm://first_ip,second_ip,third_ip"

# Galera Synchronization Configuration

wsrep_sst_method=rsync
```

##### Notes on: innodb_autoinc_lock_mode


*  Value 0 and 1 can cause a lot of table-locks

*  Value 2 (=interleaved) allows the system to insert in parallel (better performance)
    * !! Works only with ROW-Based replication

``` From galera manual
innodb_autoinc_lock_mode=2
Do not change this value.

Other modes may cause INSERT statements on tables
with AUTO_INCREMENT columns to fail.

Note Warning: When innodb_autoinc_lock_mode is set
to traditional lock mode, indicated by 0, or to consecutive lock mode,
indicated by 1, in Galera Cluster it can cause unresolved deadlocks
and make the system unresponsive.

```

#### First going live ;o)

##### Node 1

        The first server needs to be started without searching for the other servers
        # on systemd - systems  it is easy
        # execute this script
        # in case mysql is running
        systemctl stop mysql
        # For mariadb galera cluster
	# galera_new_cluster
	
	# Codership MySQL 8 with wsrep path 
	/usr/sbin/mysqld_bootstrap 
	
        # after that check, if it is running
        systemctl status mysql


##### Node 1: Check cluster size

```
mysql -uroot -p
        show status like 'wsrep_cluster_size';
        +--------------------+-------+
        | Variable_name      | Value |
        +--------------------+-------+
        | wsrep_cluster_size | 1     |
        +--------------------+-------+
        1 row in set (0.001 sec)
```

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
	
	# so stop it first 
	systemctl stop mysql 
	
	# galera.cnf configuration must be in place 
	systemctl start mysql

##### Node 3: Check status
	
	# Is it part of the cluster now ? 
	# mysql -uroot -p 
	show status like 'wsrep_cluster_size'
	# is it synced (so it has all the data from the other node
	show status like 'wsrep_local_state_comment' 

### The MaxScale - Loadbalancer

####  Why do Loadbalancing with MaxScale ?


*  Cluster node transparent to application 
    * Application does not see single nodes 

*  If one node fails you will have no downtime 
    * In opposite: To talking to this node directly 

#### License Implications since 2.x

*  MariaDB MaxScale >= 2.0 is licensed under MariaDB BSL.

*  maximum of three servers in a commercial context. 
    * Any more, and you’ll need to buy their commercial license.

*  MariaDB MaxScale 2.1.0 will be released under BSL 1.1 from the start

*  Each release transitions in about max 4 years to GPL 


#### The MaxScale load-balancer and its components

*  Listeners 

*  Filters
    * http://galeracluster.com/documentation-webpages/backingupthecluster.html

*  Servers (backend database server)

#####  Filters

*  Logging Filters

*  Statement rewriting filters

*  Result set manipulation filters 

*  Firewill filter

*  Pipeline control filters
    * e.g. tee and send to a second server

*  Ref: https://mariadb.com/kb/en/mariadb-maxscale-21-filter-tutorial/

#### Die verschiedenen Load-Balancing Szenarien des MaxScale - Clusters

#### Installation and Setup

##### Installation
	
	# Setting up the repos 
	curl -sS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash
	# Installing maxscale
	apt install maxscale

##### Setup (Part 1: MaxScale db-user)

        # IP FROM MAXSCALE
	# Setup privileges on cluster nodes
	# It is sufficient to set it on one node, because 
	# it will be synced to all the other nodes
	# on node 1 
	CREATE USER 'maxscale'@'10.10.11.139' IDENTIFIED BY 'P@ssw0rd';
	GRANT SELECT ON mysql.db TO 'maxscale'@'10.10.11.139';
	GRANT SELECT ON mysql.user TO 'maxscale'@'10.10.11.139';
	GRANT SELECT ON mysql.tables_priv TO 'maxscale'@'10.10.11.139';
	
	GRANT SHOW DATABASES ON *.* TO 'maxscale'@'10.10.11.139';

##### Setup (Part 2: Configuration)


```
# /etc/maxscale.cnf

[maxscale]

threads=auto
syslog=0
maxlog=1
log_warning=1
log_notice=1
log_info=0
log_debug=0

[Galera-Monitor]

type=monitor
module=galeramon
servers=server1,server2,server3
user=maxscale
password=password
monitor_interval=2000
disable_master_failback=1
available_when_donor=1

[RW-Split-Router]
type=service
router=readwritesplit
servers=server1,server2,server3
user=maxscale
password=password
max_slave_connections=100%
max_slave_replication_lag=30

[CLI]
type=service
router=cli

[RW-Split-Listener]
type=listener
service=RW-Split-Router
protocol=MariaDBClient
port=3306

[CLI-Listener]

type=listener
service=CLI
protocol=maxscaled
address=127.0.0.1
port=6603

[server1]
type=server
address=142.93.98.60
port=3306
protocol=MariaDBBackend

[server2]
type=server
address=142.93.103.153
port=3306
protocol=MariaDBBackend

[server3]
type=server
address=142.93.103.246
port=3306
protocol=MariaDBBackend
```

```
# Start

systemctl start maxscale  
```


```
# What does the log say ? 

# /var/log/maxscale/maxscale.log 
```

### The ProxySQL - Proxy / LoadBalancer

#### Szenario

In this case we are going to look into a RW-Split - Szenario.
This is well documented here as well:
https://severalnines.com/database-blog/how-run-and-configure-proxysql-20-mysql-galera-cluster-docker
(This documentation is even better than original proxysql - doc)

In ProxySQL 2.0 galera functionality is well integrated out of the box. 

#### Installation and Setup

##### Installation of MySQL-Client (Centos 7)


```
# Use yum-repo from codership
# just to be sure we have the same version 
sudo yum install mysql-client (?) 
```

### Common Cluster-Administration-Tasks

####  Find out galera wsrep - version


```
SHOW GLOBAL STATUS LIKE 'wsrep_provider_version';

+------------------------+---------------+
 | Variable_name | Value | 
 | ------------- | ----- | 
+------------------------+---------------+
 | wsrep_provider_version | 25.3.5(rXXXX) | 
 | ---------------------- | ------------- | 
+------------------------+---------------+
```

*  Which version belongs to which MariaDB-Version: \\ https://mariadb.com/kb/en/meta/galera-versions/

*  Deciphering Galera Version Numbers: https://mariadb.com/resources/blog/deciphering-galera-version-numbers/
    

#### Configure Logging and Special Log

*  Everything is logged to error-log 

```#You can decide what to log: 
wsrep_log_conflicts # Log when conflicts occur
                    # Like to servers want to write to the same row
cert.log_conflicts  # Log certificate conflicts while replicating  

wsrep_debug         # Show additional debug information on errors 
                    # Attention: Do not use in Production 
                    # BECAUSE: it also shows auth-information (user/pass)
```

```# Example 
# wsrep Log Options

wsrep_log_conflicts=ON
wsrep_provider_options="cert.log_conflicts=ON"
wsrep_debug=ON
```

*  Ref: http://galeracluster.com/documentation-webpages/log.html



#### Removing and adding cluster node

##### Removing a cluster node


##### Adding a cluster node


*  Ref: https://www.globo.tech/learning-center/new-node-galera-replication-cluster-ubuntu-16/

#### Updaten eines Nodes

##### Technical Backgrounds


*  Approach 1: TOI (Total Order Isolation)

*  Approach 2: RSU (Rolling Schema Updates)

##### Advantages/Disadvantages TOI


*  Advantages: 
    * Simplicity and predictability 
      * ==> data consistency 

*  Disadvantages:
    * All nodes will be LOCKED at once, till change is done 
    * Equal to: ALTER TABLE on a single MariaDB - Server

##### Using TOI

```
# Schema changes are processed on all nodes

# All nodes are locked till it is done.
SET GLOBAL wsrep_OSU_method='TOI';
```

#### Starting a cluster after full shutdown


*  Starting node 1


*  Starting node n+1...x (e.g. node 2 -> node 5) 

### Backup & Monitoring

#### Backup und Recovery

##### Backup

    

*  Best practice: 
    * Preparation:

	
	apt search garbd 
	apt install galera-arbitrator-3

    * http://galeracluster.com/documentation-webpages/backingupthecluster.html

#### Monitoring of the MariaDB cluster


*  There are some possiblities to log state on the commandline 

*  Ref: http://galeracluster.com/documentation-webpages/monitoringthecluster.html

#### Monitoring of a cluster with MaxScale


*  Generic configuration for all monitors (there are more than Galera Monitor) 

*  https://mariadb.com/kb/en/mariadb-maxscale-23-common-monitor-parameters/

#####  Galera Monitor


*  Monitor a cluster 

*  Detects if nodes are part of the cluster

*  Find out if cluster nodes are in sync

#####  Example configuration
```	
[Galera-Monitor]
type=monitor
module=galeramon
servers=server1,server2,server3
user=myuser
password=mypwd
```

### Troubleshooting

####  Handling multi-master conflicts


*  Ref: http://galeracluster.com/2015/06/debugging-transaction-conflicts-in-galera-cluster/

#### Succesful recover of a node after failure without data loss



####  Find slow nodes


*  The cluster is so slow as the slowest node ! 

*  Sometimes we need to identify the slowest node

```
# Node sent a pause event to the cluster when it was behind 

SHOW STATUS LIKE 'wsrep_flow_control_sent'; 
# > 0 => Cannot apply writesets as fast as they are received

SHOW STATUS LIKE 'wsrep_local_recv_queue_avg';
+----------------------------+---------+
 | Variable_name | Value | 
 | ------------- | ----- | 
+----------------------------+---------+
 | wsrep_local_recv_queue_avg | 3.34852 | 
 | -------------------------- | ------- | 
+----------------------------+---------+
```

*  Ref: http://galeracluster.com/documentation-webpages/detectingaslownode.html

#### Desynchronise one node for backup and maintenance in the Cluster


#####  Backup

*  Easiest to be done with xtrabackup  

```
# --no-timestamp creates no subfolder with the current date (target-dir) 

# Allow it to desync 
# in mysql-client 

set global wsrep_desync=ON;
show global status like 'wsrep%';
DB_USER=sstuser
DB_USER_PASS=password
xtrabackup --backup  --galera-info --no-timestamp \
   --target-dir=$MYSQL_BACKUP_DIR \
   --user backup_user --password backup_passwd
# in mysql-client

set global wsrep_desync=OFF;
```

*  Ref: https://mariadb.com/kb/en/library/manual-sst-of-galera-cluster-node-with-mariabackup/

*  Background info: https://www.percona.com/blog/2013/10/08/taking-backups-percona-xtradb-cluster-without-stalls-flow-control/

###  Upgrading Cluster

``` Example Mariadb 10.0 -> 10.1
# Same goes for mysql 

https://mariadb.com/kb/en/library/upgrading-from-mariadb-galera-cluster-100-to-mariadb-101/#comment_3101
```

### Exercises

####  Avoid DeadLocks (Hotspots) - Lab


*  Ref: https://severalnines.com/blog/avoiding-deadlocks-galera-set-haproxy-single-node-writes-and-multi-node-reads

####  Stop one node / Master ?


  *  On MaxScale: maxadmin -pmariadb 
```
list servers 
```

  *  Stop one Node 1:
     * on node: systemctl stop mysql

  *  On MaxScale:
```
	
	# Who is the master now ? (Master Stickiness) 
	# 
	maxadmin -pmariadb list servers 
```

####  Restart one node (down) -> Master ?


  *  On MaxScale
```
	
	maxadmin -pmariadb list servers 
	
```

  *  On node 
```
	systemctl start mysql 
```

  *  On MaxScale
  
```
	maxadmin -pmariadb list servers
```

### Misc

####  SSL Encryption - Galera


*  Ref: msutic.blogspot.com/2017/10/enable-ssl-encryption-for-mariadb.html 

```
Datum	Heute 12:02
mkdir -p /etc/mysql/ssl
cd /etc/mysql/ssl/
openssl genrsa 2048 > ca-key.pem
openssl req -new -x509 -nodes -days 3600 -key ca-key.pem -out  ca-cert.pem
openssl req -newkey rsa:2048 -days 3600 -nodes -keyout  server-key.pem -out -server-req.pem
openssl req -newkey rsa:2048 -days 3600 -nodes -keyout  server-key.pem -out server-req.pem
openssl rsa -in server-key.pem -out server-key.pem
openssl x509 -req -in server-req.pem -days 3600 -CA  ca-cert.pem -CAkey ca-key.pem -set_serial 01 -out server-cert.pem
openssl req -newkey rsa:2048 -days 3600 -nodes -keyout  client-key.pem -out client-req.pem
openssl rsa -in client-key.pem -out client-key.pem
openssl x509 -req -in client-req.pem -days 3600 -CA  ca-cert.pem -CAkey ca-key.pem -set_serial 01 -out client-cert.pem
openssl verify -CAfile ca-cert.pem server-cert.pem client-cert.pem
```
