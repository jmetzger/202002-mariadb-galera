# MySQL - Administration / Galera - Training (codership)  

```
We are working with Centos 7 here 
```

## Agenda 

  * Architecture of the MySQL - Server
 
  * Installation

  * Konfiguration
 
  * Security and User-Rights
 
  * Locking 

  * Database-Objects 

  * Monitoring 

  * Performance Optimization 

  * Backups 

  * MySQL - Galera - Cluster 

## 1. Centos 7 - Repo - Configuration  

  * https://galeracluster.com/library/documentation/install-mysql.html

```
# /etc/yum.repos.d/galera.repo
[galera]
name = Galera
baseurl = https://releases.galeracluster.com/galera-4/centos/7/x86_64/
gpgkey = https://releases.galeracluster.com/GPG-KEY-galeracluster.com
gpgcheck = 1

[mysql-wsrep]
name = MySQL-wsrep
baseurl =  https://releases.galeracluster.com/mysql-wsrep-8.0/centos/7/x86_64/
gpgkey = https://releases.galeracluster.com/GPG-KEY-galeracluster.com
gpgcheck = 1

```

## 2. Installation 


```
yum install galera-4 and mysql-wsrep-8.0
# get the temporary password 
cat /var/log/mysqld.log | grep "password.*generated"
```

## 3. Configuration SELinux  

```
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

## 4. Firewall 

```
# redundant - next line
# firewall-cmd --zone=public --add-service=mysql --permanent
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --zone=public --add-port=4444/tcp --permanent
firewall-cmd --zone=public --add-port=4567/tcp --permanent
firewall-cmd --zone=public --add-port=4567/udp --permanent
firewall-cmd --zone=public --add-port=4568/tcp --permanent
firewall-cmd --reload
```
