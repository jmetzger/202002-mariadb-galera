# Performance Metrics - Galera 

## wsrep - status 

````
# How many times, communicated, he had to pause
# He/She had to sent flow_control (others should slow donw 
| wsrep_flow_control_paused     | 0                                                                                                                                              |
| wsrep_flow_control_sent       | 0      
```

```
# Hinkt evtl. bei empfangenen writeset transaktionen hinterher. 
wsrep_local_recv_queue_avg | 0.0434783 |

wsrep_local_send_queue_avg | 0    

```

## wsrep  // adjust threads for better performance 

```
 show status like '%wsrep_cert_deps_distance%';
+--------------------------+---------+
| Variable_name            | Value   |
+--------------------------+---------+
| wsrep_cert_deps_distance | 4.19298 |
+--------------------------+---------+
1 row in set (0.001 sec)

### wsrep_slave_threads --> should not be set higher that wsrep_cert_deps_distance 

show variables like 'wsrep_slave_threads';


MariaDB [training]> show status like '%wsrep_thread_%';
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_thread_count | 2     |
+--------------------+-------+
1 row in set (0.001 sec)


```
