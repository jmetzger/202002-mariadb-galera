# FAQ's  

## Frage 1:  Gibt es eine Formel für den gcache ? 

Ja, dieser läßt sich umgekehrt berechnen,
d.h. wieviel Downtime kann ich mir leisten ? 

Referenz: 
https://fromdual.com/gcache_size_in_galera_cluster

```
#my.cnf
[mysqld]
wsrep_provider_options="gcache.size=256M"
```

```
Hold time = GCache size / Replication Rate.
Where:
		Replication Rate = Amount of replicated data / time. (ime in seconds)
		Amount of replicated data = (wsrep_replicated_bytes + wsrep_received_bytes) after the maintenance window - (wsrep_replicated_bytes + wsrep_received_bytes) before the maintenance window.
The amount of replicated data for the customer's case = 7200MB.
Now, we can find out how much GCache (default 128M) can handle for the customer's case:
Hold time = 128MB / (7200MB / 4h) = 128MB / 0.5 MB = 256s.
Then, we can calculate the right GCache size value to handle the maintenance window by the following formula: GCache = Maintenance window * Replication Rate = 14400s * 0.5 MB. GCache = 7200MB.
In other words, the right GCache size should be equivalent to (or not less than) the amount of replicated data.
```


## Frage 2: What does OSU-Method: TOI and OSU-Method: RSU mean ?

SHOW VARIABLES LIKE 'wsrep_OSU_method';

OSU= Online Schema Updates.
Defines how Online Schema Updates are done. 
a) TOI: Default: TOI - Total Order Isolation 
(Update are done simultaneously on all nodes, blocks all traffics for write AND READ in the meantime - to all nodes). So while doing does, cluster is „none functional“ 
Online Schema Updates on MariaDB are expensive: 
- Temporary Table is created with new structure .
- Original table is completely locked 
- Temporary Table is populated, 
- Temporary Table is renamed to table-name 

Good: Consistency 
Bad: Blocks complete system 

ALTERNATIV: pt-online-schema-change 
https://www.percona.com/doc/percona-toolkit/LATEST/pt-online-schema-change.html


B) RSU: Rolling Schema Upgrade 
- DDL will only be processed locally on the node. 
- Node is desynchronized from the cluster for the duration of the DDL processing 
- In this time: doesn’t block the other nodes 
- complete? Then -> the node applies the delayed replication events and synchronizes with the cluster.
For more information, see Rolling Schema Upgrade.

Good: Non Blocking

Bad: danger of loosing consistency (application / admin has to handle this himself):
- Table defnitions have to be backwards compatible:
  1) adding new fields at the end 
  2) new fields need to have defaults. 

Practically:
SET GLOBAL wsrep_OSU_method='RSU';
// Update is only done on the local node 
// you have to do the schema upgrade on every node 


select @@wsrep_OSU_method,@@wsrep_on,@@wsrep_desync;
+--------------------+------------+----------------+
| @@wsrep_OSU_method | @@wsrep_on | @@wsrep_desync |
+--------------------+------------+----------------+
| RSU                |          1 |              0 |
+--------------------+------------+----------------+
1 row in set (0.00 sec)

Desyncing (so setting wsrep_desync = 1) is done automatically, when a DDL starts and the node is set  back to wsrep_desync=0 after then node has finished. 

Only use when:
ones that cannot break ROW replication when the writer node and applier nodes have different metadata

Refer to: 
https://www.percona.com/blog/2019/03/25/how-to-perform-compatible-schema-changes-in-percona-xtradb-cluster-advanced-alternative/

Alternaive: Manual rsu:
SET GLOBAL wsrep_desync=1;
SET wsrep_on=0;
ALTER ...  /* compatible schema change only! */
SET wsrep_on=1;
SET GLOBAL wsrep_desync=0;

## Frage 3: Was ist besser mariabackup oder rsync.
Aus meiner Sicht ist mariabackup besser weil es non-blocking. 
rsync ist im Gegensatz dazu blockierend, d.h. während des SST
steht der Donor nicht für Anfragen zur Verfügung.

Ref: https://www.claudiokuenzler.com/blog/887/comparing-galera-wsrep-sst-methods-rsync-vs-mariabackup

## Frage 4: Welche Werte sind wichtig beim Monitoring. Wie kann ich sehen, ob ein Nodes hinterherhinkt. 
https://severalnines.com/database-blog/monitoring-galera-cluster-mysql-or-mariadb-understanding-metrics-updated

```
wsrep_local_send_queue_avg
wsrep_local_recv_queue_avg
```

## Frage 5: Keine Frage aber ein sehr guter Read:
https://www.percona.com/blog/2014/11/17/typical-misconceptions-on-galera-for-mysql/

## Frage 6: How fast is a cluster (Referring to: Callaghan’s law: “In a Galera cluster a given row can’t be modified more than once per RTT”.)

### RTT - Round-Trip-Time lässt sich mit einem Ping messen
ping -c 4 10.10.11.141
PING 10.10.11.141 (10.10.11.141) 56(84) bytes of data.
64 bytes from 10.10.11.141: icmp_seq=1 ttl=64 time=0.270 ms
64 bytes from 10.10.11.141: icmp_seq=2 ttl=64 time=1.05 ms
64 bytes from 10.10.11.141: icmp_seq=3 ttl=64 time=1.04 ms
64 bytes from 10.10.11.141: icmp_seq=4 ttl=64 time=1.01 ms

--- 10.10.11.141 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 97ms
rtt min/avg/max/mdev = 0.270/0.841/1.047/0.331 ms

=> hier: durchschnittlich: rtt avg = 0.84 ms 

```
Q: The network RTT between my servers is 100ms. I know that each commit takes at least as much time as a network RTT so can I only hope to run 10 writes/s?
This needs some clarification: commit is synchronous because the full transaction is replicated to all nodes when it is committed. However you pay the same price whether the transaction has a single statements or several statements.
So if all your writes are auto-committed transactions, each write will trigger a commit that will need at least one RTT to complete. If RTT is 100ms, that means 10 writes/s.
But if your transactions have 10 writes, you will only need a commit every 10 writes: with 10 commits/s, you can now run 100 writes/s.
And finally several threads can commit at the same time, increasing the write throughput. With 10 concurrent threads executing transactions with 10 statements, you get 1000 writes/s.
Of course, this is only theory. In the real world, you will probably not get so perfectly aligned numbers. You can look at this post (Comparing Percona XtraDB Cluster with Semi-Sync replication Cross-WAN) to see real numbers with 1 thread and 32 threads.
Conclusion: network latency is a limiting factor for write throughput, that’s true. But that may not be as bad as you can think. And remember Callaghan’s law: “In a Galera cluster a given row can’t be modified more than once per RTT”.
```

## Frage 7: Wie funktioniert flow control:

* unable to match the apply speed with the replicated write-set speed
* If queue crosses some threshold (dictated by gcs.fc_limit), then flow control kicks in.
* Flow control causes members of the cluster to temporary halt/slow-down so that the slower node can catch up.

## Frage 8: tunen von gc.fc_limit 

Flow control causes members of the cluster to temporary halt/slow-down so that the slower node can catch up.

```# set global wsrep_provider_options="gcs.fc_limit=16";```

If you are settings this value really high, you loose freshness of data.
(e.g. like 1600, so it can keep 1600 write sets in the cache) 

-> please also activate:
set global wsrep_sync_wait=„gcs.wsrep_sync_wait=1“ 

wsrep_sync_wait

```
Defines whether the node enforces strict cluster-wide causality checks.
Command-line Format
--wsrep-sync-wait
System Variable
wsrep_sync_wait
Variable Scope
Session
Dynamic Variable
Yes
Permitted Values
Bitmask
Default Value
0
Initial Version
Version 3.6
```

When you enable this parameter, the node triggers causality checks in response to certain types of queries. During the check, the node blocks new queries while the database server catches up with all updates made in the cluster to the point where the check was begun. Once it reaches this point, the node executes the original query.

Ref: https://www.percona.com/blog/2017/04/22/beware-increasing-fc_limit-can-affect-select-latency/

## Frage 9: Richtige Werte für Performance Optimierung: 

```
# unschädlich, gleiches Verhalten wie bei single_node, wenn nicht alle cluster-
# Nodes Gleichzeitig Stromausfall haben  
innodb_flush_log_at_trx_commit = 2
sync_bin_log = 0 
Auf keinen Fall > 0 
```

## Frage 10: Wie hoch sollten die Parallel-Threads sein ? 
```
SET GLOBAL wsrep_slave_threads = 48;
```
To get the best out of this, we need to know two things:
	•	The number of cores the server has.
	•	The value of wsrep_cert_deps_distance status. (this is our max) - potential degrees of parallelization:
	•	       https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-cert-deps-distance

wsrep_slave_threads
 A recommended value is anywhere between 16 to 48:
(Should be minimum 4x the cpu core, but not more than wsrep_cert_deps_distance) 


## Frage 11. Wie erstelle ich am besten backups ? 

Wichtig: Die node muss desynchrnoisiert werden, bevor ich Backups mache:

```
Variante 1:
mysql -p -u root --execute "SET GLOBAL wsrep_desync = ON; SET GLOBAL wsrep_on='off'"
Dann:
mysqldump -p -u admin_backup \
          --flush-logs --all-databases \
          > /backups/db-backup-20191025.sql

mysql -p -u root --execute "SET GLOBAL wsrep_on='on'; SET GLOBAL wsrep_desync = OFF"

```

```
Variante 2:
mysql -p -u root -e "STOP SLAVE"

mysqldump -p -u admin_backup --flush-logs --all-databases \
          > /backups/temp/backup-20191025.sql

cp -r /etc/my.cnf* /backups/temp/
```

```
Variante 3: 
Using garbd
Refer to: 
https://galeracluster.com/library/training/tutorials/galera-backup.html
```



## Frage 12: How does mariadb galera cluster work with mysql_upgrade and RSU 

```
 On Fri, Jun 19, 2015 at 9:55 AM, Tom Worster <fsb@xxxxxxxxxx> wrote:
> Can I upgrade a production cluster from 10.0.15 to 10.0.19 while the cluster
> remains in service, upgrading one node at a time?
> 
> 
> Yes, you can do this. If you plan to run mysql_upgrade on each node
> after upgrading, you may want to set wsrep_OSU_method to RSU before
> doing so. I think this is especially important for this upgrade, since
> mysql_upgrade in 10.0.19 calls REPAIR VIEW. That statement doesn't
> exist in 10.0.15.
> 
> http://galeracluster.com/documentation-webpages/mysqlwsrepoptions.html#wsrep-osu-method
> 
> https://mariadb.com/kb/en/mariadb/repair-view/
> 
> e.g.:
> 
> service mysql stop
> yum update MariaDB-Galera-server MariaDB-client MariaDB-devel
> MariaDB-shared galera-3
> service mysql start
> mysql --execute="SET GLOBAL wsrep_OSU_method=RSU;"
> mysql_upgrade
> mysql --execute="SET GLOBAL wsrep_OSU_method=TOI;"
> 
> Geoff
```

Refer to: https://lists.launchpad.net/maria-discuss/msg02698.html


## Frage 14: problem with wsrep has not yet prepare 

https://stackoverflow.com/questions/40653238/mariadb-galera-error-when-a-node-shutdown-error-1047-wsrep-has-not-yet-prepare
https://dba.stackexchange.com/questions/255100/single-node-stuck-in-wsrep-has-not-yet-prepared-node-for-application-use
