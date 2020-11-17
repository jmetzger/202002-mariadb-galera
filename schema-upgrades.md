# Schema Updates (TOI/RSU)

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

RSU automatically switched the local node to Donor/Desynced state, to ensure it will not impact
the rest of the cluster
