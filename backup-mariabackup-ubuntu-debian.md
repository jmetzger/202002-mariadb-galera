# Mariabackup (Debian/Ubuntu)

## What does set global wsrep_desync='on' do 

  * It allows the node to fall behind and by so, having no impact on the rest of the cluster nodes getting slow 
  * It simply does does by disabling flow control 

## Walkthrough 

  * mariabackup has to be the same version as mariadb 

```
# /root/.my.cnf 
[mariabackup]
user=root 

## or 
## simply use: --user=root with command mariabackup 

# Backup 
mysql -e "set global wsrep_desync = on" # Important !! Cluster node needs to be taken out of the cluster
mkdir /data
mariabackup --backup --galera-info --target-dir=/data/backup-2020092201/
mariabackup --prepare --target-dir=/data/backup-2020092201/
mysql -e "set global wsrep_desync = off" # Put cluster back 

# recover - cannot be done while running 
systemctl stop mysqld 
cd /var/lib/
mv mysql mysql.bkup 
mariabackup --copy-back --target-dir=/data/backup-2020092201/
chown -R mysql:mysql mysql
systemctl start mysqld 
```

## Mariabackup xbstream and pipe to gz 

```
mariabackup --user=root --backup --galera-info --stream=xbstream | gzip -9 > /backup/mariadb.gz 
```
