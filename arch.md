# Architecture 
 
## 1.2. Ports used

```
3306 :: mysql default port + mysqldump for SST 
4567 is reserved for Galera Cluster replication traffic. Multicast replication uses both TCP and UDP transport on this port.
4568 is the port for Incremental State Transfer.
4444 is used for all other State Snapshot Transfer.
```

* Port 3306 is the default port for MySQL client connection 
  * + used for State Snapshort Transfers using mysqldump 

