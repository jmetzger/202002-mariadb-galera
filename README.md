# MariaDB - Administration / Galera - Training (Centos 8)

```
We are working with Centos 7 here 
```

## Agenda 

  1. Architecture of the MySQL - Server
 
  1. [Installation](#2-installation)

  1. Configuration
 
  1. Security and User-Rights
 
  1. Locking 

  1. Database objects 

  1. Monitoring 

  1. Performance Optimization 

  1. Backup und Restore 

  1. [MySQL - Galera - Cluster](Galera.md#10-mysql-galera-cluster)
  
  1. [Misc](#11-misc) 


## 2-Installation

### 2.1 Centos 7 - Repo - Configuration  

  * configure repo on: https://mariadb.org/download/#mariadb-repositories
  * write configuration below into /etc/yum.repos.d/mariadb.repo

```
# write into /etc/yum.repos.d/mariadb.repo 
[mariadb]
name = MariaDB
baseurl = http://mirror.23media.de/mariadb/yum/10.4/centos8-amd64
module_hotfixes=1
gpgkey=http://mirror.23media.de/mariadb/yum/RPM-GPG-KEY-MariaDB
gpgcheck=1
```

### 2.2 Installation 

```
# dnf install mariadb # only installs the client 
dnf install mariadb-server 
# see what is installed 
rpm -qa | grep -i mariadb 
systemctl list-unit-files -t service | grep mariadb
systemctl status mariadb 
systemctl start mariadb 
systemctl status mariadb 

# shows port mariadb is listening 

# Nicht beim Starten vom Server starten 
systemctl disable mysqld.service 
systemctl enable mysqld.service 

# is service listening 
lsof -i
cat /etc/services | grep mysql


# get the temporary password 
cat /var/log/mysqld.log | grep "password.*generated"
```

```
# findout if mysql listen to the outside
# look for *:mysql
lsof -i 
```

### 2.2.1 Configuration /etc/my.cnf.d/z_galera.cnf 

```
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0
# Set to 1 sec instead of per transaction
# for better performance // Attention: You might loose data on power
outage
innodb_flush_log_at_trx_commit=0
# Galera Provider Configuration
wsrep_on=ON
# ubuntu
wsrep_provider=/usr/lib/galera/libgalera_smm.so
# centos7 (x86_64)
wsrep_provider=/usr/lib64/galera/libgalera_smm.so
# Galera Cluster Configuration
wsrep_cluster_name="test_cluster"
wsrep_cluster_address="gcomm://first_ip,second_ip,third_ip"
# Galera Synchronization Configuration
wsrep_sst_method=rsync

```

### 2.3 Configuration SELinux  

```
# Option 1
# Disable selinux at runtime 
sestatus
# switch from enforcing to permissive (runtime) 
setenforce 0
getenforce
sestatus

```


```
# Option 2
# Open everything that is needed 
# setools provide command semanage
yum provides semanage
yum install setools 
# Somehow redundant when you open the service altogether
# With mysqld_t 
# semanage port -a -t mysqld_port_t -p tcp 3306
# semanage port -a -t mysqld_port_t -p tcp 4444
# semanage port -a -t mysqld_port_t -p tcp 4567
# semanage port -a -t mysqld_port_t -p udp 4567
# semanage port -a -t mysqld_port_t -p tcp 4568
semanage permissive -a mysqld_t
```

### 2.4 Firewall 

```
# Option 1
systemctl disable firewalld
systemctl stop firewalld 
```


```
# Option 2
# redundant - next line
# firewall-cmd --zone=public --add-service=mysql --permanent
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --zone=public --add-port=4444/tcp --permanent
firewall-cmd --zone=public --add-port=4567/tcp --permanent
firewall-cmd --zone=public --add-port=4567/udp --permanent
firewall-cmd --zone=public --add-port=4568/tcp --permanent
firewall-cmd --reload
```

## 3 Configuration 

### Best source 

https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html

## 4 Security and user rights 

```
# All available rights 
https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html
```

## 8 Performance Optimization 

### Slow query (good read) 

https://mariadb.com/kb/en/slow-query-log-overview/

## Innodb and general optimization 
https://www.percona.com/blog/2016/10/12/mysql-5-7-performance-tuning-immediately-after-installation/
http://schulung.t3isp.de/documents/pdfs/mysql/mysql-performance.pdf

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




### 11 Misc

#### Savepoints 

https://dev.mysql.com/doc/refman/8.0/en/savepoint.html
