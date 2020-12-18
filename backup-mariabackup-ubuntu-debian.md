# Mariabackup (Debian/Ubuntu)

  * mariabackup has to be the same version as mariadb 

```
# Backup 
mysql -e "set global wsrep_desync = on" # Important !! Cluster node needs to be taken out of the cluster
mkdir /data
mariabackup --backup --target-dir=/data/backup-2020092201/
mariabackup --prepare --target-dir=/data/backup-2020092201/
mysql -e "set global wsrep_desync = off" # Put cluster back 

# recover - cannot be done while running 
systemctl stop mysqld 
cd /var/lib/
mv mysql mysql.bkup 
xtrabackup --no-version-check --copy-back --target-dir=/data/backup-2020092201/
chown -R mysql:mysql mysql
systemctl start mysqld 
```
