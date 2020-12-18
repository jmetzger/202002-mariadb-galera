# MaxScale - LoadBalancer // Installation and Configuration (Debian/Ubuntu)

##  Why do Loadbalancing with MaxScale ?


*  Cluster node transparent to application 
    * Application does not see single nodes 

*  If one node fails you will have no downtime 
    * In opposite: To talking to this node directly 

## License Implications since 2.x

*  MariaDB MaxScale >= 2.0 is licensed under MariaDB BSL.

*  maximum of three servers in a commercial context. 
    * Any more, and you’ll need to buy their commercial license.

*  MariaDB MaxScale 2.1.0 will be released under BSL 1.1 from the start

*  Each release transitions in about max 4 years to GPL 


## The MaxScale load-balancer and its components

*  Routers 
*  Listeners 
*  Filters
*  Servers (backend database server)

###  Filters

*  Logging Filters
*  Statement rewriting filters
*  Result set manipulation filters 
*  Firewill filter
*  Pipeline control filters
    * e.g. tee and send to a second server

*  Ref: https://mariadb.com/kb/en/mariadb-maxscale-25-regex-filter/


## Installation and Setup

### Installation
	
```
apt update
apt install apt-transport-https

# Setting up the repos 
curl -sS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash
# Installing maxscale
apt install maxscale
```

### Setup (Part 1: MaxScale db-user)

  * Do this on one of the galera nodes 
  * Adjust IP !! 

```bash
# IP FROM MAXSCALE
# Setup privileges on cluster nodes
# It is sufficient to set it on one node, because 
# it will be synced to all the other nodes
# on node 1 
CREATE USER 'maxscale'@'10.10.11.139' IDENTIFIED BY 'P@ssw0rd';
#
GRANT SELECT ON mysql.db TO 'maxscale'@'10.10.11.139';
GRANT SELECT ON mysql.user TO 'maxscale'@'10.10.11.139';
GRANT SELECT ON mysql.tables_priv TO 'maxscale'@'10.10.11.139';
#
GRANT SELECT ON mysql.columns_priv TO 'maxscale'@'10.10.11.139';
GRANT SELECT ON mysql.proxies_priv TO 'maxscale'@'10.10.11.139';
#
GRANT SHOW DATABASES ON *.* TO 'maxscale'@'10.10.11.139';
```

```
# On maxscale - server 
apt update 
apt install mariadb-client 
# Test the connection 
# Verbindung sollte aufgebaut werden 
mysql -u maxscale -p -h <ip-eines-der-nodes>
mysql>show databases 
```

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
password=P@ssw0rd
monitor_interval=2000
disable_master_failback=1
available_when_donor=1

[RW-Split-Router]
type=service
router=readwritesplit
servers=server1,server2,server3
user=maxscale
password=P@ssw0rd
max_slave_connections=100%
max_slave_replication_lag=30

[RW-Split-Listener]
type=listener
service=RW-Split-Router
protocol=MariaDBClient
port=3306

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
