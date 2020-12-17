# MariaDB - Administration / Galera - Training (Redhat/Centos/Ubuntu/Debian)

## Agenda 

  1. [Architecture of the MySQL - Server](arch.md)
 
  1. Installation
     * [Installation (Ubuntu 20.04 LTS)](installation-ubuntu-20-04.md)
     * [Installation (Centos 8)](installation-centos-8.md)
  
  1. Configuration
     * [MariaDB Cluster Configuration (Ubuntu 20.04 LTS / MariaDb 10.04)](cluster-configuration-ubuntu-20-04.md)
     * [MariaDB Cluster Configuration (Centos)](cluster-configuration-centos.md) 
  
  1. Troubleshooting  
     * [Handling bei Stromausfall](galera-stromausfall.md) 
  
  1. Tipps & Tricks 
     * [Find good gcache-size](determine-gcache.md) 
  
  1. MaxScale 
     * [License MaxScale](#License-Maxscale)
  
  1. Monitoring 

  1. Performance Optimization 

  1. Backup und Restore 

  1. [MySQL - Galera - Cluster](Galera.md#10-mysql-galera-cluster)
  
  1. [Misc](#11-misc) 
  
  1. [Documentation](#12-Documentation) 
  
  1. [Schema Upgrades](schema-upgrades.md)
  
  1. [Backup with Arbitrator aka garbd](garbd-backup.md)
  
  1. [Backup und Recover - mysqldump/mysql ](mysqldump-mysql.md) 
  
  
  1. [EasyPeasy - mysqladmin](mysqladmin.md) 

## 2-Installation and Configuration 

### 2.1 Installation

  * [Installation (Ubuntu 20.04)](installation-ubuntu-20-04.md)
  * [Installation (Centos 8)](installation-centos-8.md)

### 2.2.1 Configuration

  * [MariaDB Cluster Configuration (Ubuntu 20.04 LTS / MariaDb 10.04)](cluster-configuration-ubuntu-20-04.md)
  * [MariaDB Cluster Configuration (Centos)](cluster-configuration-centos.md) 
 

## 9 Backup und Restore

### Import csv-data 

https://www.mysqltutorial.org/import-csv-file-mysql-table/

### Xtrabackup 

```
# Install repo 
sudo yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm

yum install xtrabackup 

# problems on mysql 8.0.19 - use no-check-version here 
# https://jira.percona.com/browse/PXB-1827?attachmentViewMode=list

# backup 
mkdir /data
xtrabackup --no-version-check --backup --target-dir=/data/backup-2020092201/
xtrabackup --no-version-check --prepare --target-dir=/data/backup-2020092201/

# recover - cannot be done while running 
systemctl stop mysqld 
cd /var/lib/
mv mysql mysql.bkup 
xtrabackup --no-version-check --copy-back --target-dir=/data/backup-2020092201/
chown -R mysql:mysql mysql

# beware of sestatus otherwice server will not start
sestatus 
# should show either permissive or disabled 
# to disable 
setenforce 0 
# + change in /etc/selinux/config for next reboot 

systemctl start mysqld 

#
# https://www.percona.com/doc/percona-xtrabackup/8.0/backup_scenarios/full_backup.html

```


### Automation with cron 

```
#/root/.my.cnf
[client]
password=here_my_pass_for_root
```

```
# /etc/cron.daily/backup
# chmod u+x 
#!/bin/bash

if [ ! -d /usr/src/backups ]
then
  mkdir /usr/src/backups/
fi

cd /usr/src/backups
TODAY=$(date +"%Y%m%d%H%M%S")
echo $TODAY
mkdir $TODAY
cd $TODAY


mysqldump --all-databases > all-databases.sql

date >> /var/log/dbbackup.log


```

### Alternative with .timers (systemd)

https://kofler.info/systemd-timer-als-cron-alternative/





### 12 Documentation 

#### Server System Variables 

 * https://mariadb.com/kb/en/server-system-variables/

```
show variables like 'wsrep%' \G
show status like 'wsrep%' \G
```

### 13 Performance 

```
innodb_flush_log_at_trx_commit = 2
# up to 75x faster 
https://dba.stackexchange.com/questions/12611/is-it-safe-to-use-innodb-flush-log-at-trx-commit-2/56673#56673

innodb_flush_method = O_DIRECT_NO_FSYNC
# Not suitable for XFS filesystems.

# recommendation from galera 
innodb_autoinc_lock_mode = 2 

```

### 14 Simulations 

  * Scenario, where node goes into Non-Primary mode, because the other
  * node have no connection 
  
```
## Does not work, because established connections to still on requests
for i in  3306 4444 4567 4568 ; do firewall-cmd --remove-port=$i/tcp; done
firewall-cmd --remove-port=4567/udp 

## try this instead + do it on terminal !
## on node 3 (or another node) 
systemctl stop firewalld 
for i in  3306 4444 4567 4568 ; do firewall-cmd --remove-port=$i/tcp; done
iptables -A INPUT -p tcp --destination-port 80 -j DROP
```





