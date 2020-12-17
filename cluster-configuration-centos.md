# Configuration - Centos  

```
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0
# Set to 1 sec instead of per transaction
# for better performance // Attention: You might loose data on power
outage
innodb_flush_log_at_trx_commit=0
# Galera Provider Configuration
wsrep_on=ON
# centos8 (x86_64)
wsrep_provider=/usr/lib64/galera-4/libgalera_smm.so
# Galera Cluster Configuration
wsrep_cluster_name="test_cluster_jm"
wsrep_cluster_address="gcomm://10.10.11.141,10.10.11.166,10.10.11.170"
# Galera Synchronization Configuration
wsrep_sst_method=rsync
```
