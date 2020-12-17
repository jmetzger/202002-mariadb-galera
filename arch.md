# Architecture 
 
## 1.2. Ports used

```
Port 3306 :: mysql default port + mysqldump for SST 
Port 4567 is reserved for Galera Cluster replication traffic. Multicast replication uses both TCP and UDP transport on this port.
Port 4568 :: Used for Incremental State Transfer.
Port 4444 :: Used for all other SSt's: mariabackup, rsync 
```

* Port 3306 is the default port for MySQL client connection 
  * + used for State Snapshort Transfers using mysqldump 
* Port 4444 is used for all State Transfer Backups (rsync, mariabackup) 
