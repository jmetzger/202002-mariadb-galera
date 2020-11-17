# Determine gcache size 

## How long can we be down ? 

```
set @start := (select sum(VARIABLE_VALUE/1024/1024) from information_schema.global_status where VARIABLE_NAME like 'WSREP%bytes'); do sleep(60); set @end := (select sum(VARIABLE_VALUE/1024/1024) from information_schema.global_status where VARIABLE_NAME like 'WSREP%bytes'); set @gcache := (select SUBSTRING_INDEX(SUBSTRING_INDEX(@@GLOBAL.wsrep_provider_options,'gcache.size = ',-1), 'M', 1)); select round((@end - @start),2) as `MB/min`, round((@end - @start),2) * 60 as `MB/hour`, @gcache as `gcache Size(MB)`, round(@gcache/round((@end - @start),2),2) as `Time to full(minutes)`;
```

## Explanation ? 

Ja, dieser läßt sich umgekehrt berechnen,
d.h. wieviel Downtime kann ich mir leisten ? 

Referenz: 
https://fromdual.com/gcache_size_in_galera_cluster

```
#my.cnf
[mysqld]
wsrep_provider_options="gcache.size=256M"
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
