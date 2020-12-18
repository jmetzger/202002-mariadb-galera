# Monitoring 

## What to monitor ? 

  * Einzelne Nodes
  * wsrep_local_state_comment -> synced ## eher ein Thema und wenn der nur auf nicht synced steht (z.B. ganzen Tag)  
  * wsrep_cluster_size 
  * wsrep_cluster_status # Mitglied des Primär-Knotens -> Primary // Non-Primary is BAD !! 
  * Läuft die node 
  * 3306 port offen ? 4567 offen ? 
  
## pmmdemo (Percona Management und Monitoring Tool) 


  * https://pmmdemo.percona.com/graph/d/pmm-home/home-dashboard?orgId=1&refresh=1m&var-interval=$__auto_interval_interval&var-environment=All&var-node_name=pxc-80-2&var-service_name=All&var-cluster=All&var-replication_set=All&var-database=All&var-node_type=All&var-service_type=All&var-username=All&var-schema=All
  * https://www.percona.com/doc/percona-monitoring-and-management/deploy/server/docker.html




  
