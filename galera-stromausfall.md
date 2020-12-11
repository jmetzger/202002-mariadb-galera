# Cluster hochziehen nach stromausfall 

## Basis 

  * mariadb war leider nicht für Neustart enabled
  * Prüfung dazu: systemctl is-enabled mariadb 
 
```
systemctl is-enabled mariadb
disabled
```

## Schritte:

  * Auf jedem Host folgendes Kommando ausführen (also root) 
  
```
sudo -u mysql mysqld --wsrep-recover
# in der letzten Zeil steht, dann die position: z.B. 21 
2020-11-17 11:30:31 0 [Note] WSREP: Recovered position: 6dcdc984-2808-11eb-abeb-f799ea8dadff:21

## achtung: mariadb server darf nicht laufen 
systemctl is-active mariadb 
systemctl status mariadb 
galera_recovery 
```

