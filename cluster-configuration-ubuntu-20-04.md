# Cluster Configuration (ubuntu 20.04 - MariaDB 10.4 from mariadb.org) 

## Node 1 

```
# cat /etc/mysql/mariadb.conf.d/z_galera.cnf
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
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Galera Cluster Configuration

wsrep_cluster_name="test_cluster"
wsrep_cluster_address="gcomm://192.168.56.101,192.168.56.102,192.168.56.103"

# Galera Synchronization Configuration

wsrep_sst_method=rsync
```

```
# start 1st cluster node 
systemctl stop mariadb
galera_new_cluster # always use this to bootstrap first cluster node 

systemctl status mariadb 
journalctl -u mariadb 

```

