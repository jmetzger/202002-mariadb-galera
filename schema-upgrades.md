# Schema Updates (TOI/RSU)

##

DDL statements (ALTER, CREATE, RENAME, TRUNCATE, DROP)

TOI (Total Order Isolation + Alternative pt-online-schema-change):
https://severalnines.com/blog/online-schema-upgrade-mysql-galera-cluster-using-toi-method

RSU (Rolling Schema Upgrade) 
```
mysql> SET SESSION wsrep_OSU_method=RSU;
Query OK, 0 rows affected (0.00 sec)

mysql> ALTER TABLE sbtest1.sbtest1 ADD INDEX idx_new (k, c); 
Query OK, 0 rows affected (5 min 19.59 sec)
```

RSU automatically switches the local node to Donor/Desynced state, to ensure it will not impact
the rest of the cluster

## TOI - Blocking Transactions after ALTER 

```
Blocks all transactions that have be done after alter statement
They have to wait 

https://galeracluster.com/library/documentation/schema-upgrades.html

In Total Order Isolation, queries that change the schema replicate as statements to all nodes in the cluster. The nodes wait for all preceding transactions to commit simultaneously, then they execute the schema change in isolation. For the duration of the DDL processing, no other transactions can commit.

```
