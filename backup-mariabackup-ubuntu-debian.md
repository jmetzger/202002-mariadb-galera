# Mariabackup (Debian/Ubuntu)

```
# Needs to fit to installed mariadb - version 


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
