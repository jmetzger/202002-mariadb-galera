# Configuration - Centos  

## Step 1: Configuration SELinux (on every node)

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
# yum provides semanage
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

### Step 2: Configure Firewall  (on every node)

```
# Option 1
systemctl disable firewalld
systemctl stop firewalld 
```


```
# Option 2
https://galeracluster.com/library/documentation/firewalld.html

# redundant - next line
# firewall-cmd --zone=public --add-service=mysql --permanent
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --zone=public --add-port=4444/tcp --permanent
firewall-cmd --zone=public --add-port=4567/udp --permanent
firewall-cmd --zone=public --add-port=4567/tcp --permanent
firewall-cmd --zone=public --add-port=4568/tcp --permanent
firewall-cmd --reload
# did this work ? 
firewall-cmd --list-all-zones | grep -A10 "public"
```

### Step 3: galera-configuration on every node 

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
# centos8 (x86_64)
wsrep_provider=/usr/lib64/galera-4/libgalera_smm.so
# Galera Cluster Configuration
wsrep_cluster_name="test_cluster_jm"
wsrep_cluster_address="gcomm://10.10.11.141,10.10.11.166,10.10.11.170"
# Galera Synchronization Configuration
wsrep_sst_method=rsync
```

### Step 4a: Start/add Node 1 

```
# be sure mariadb is stopped 
systemctl stop mariadb 
galera_new_cluster 

# now check for status 
mysql 
# look for cluster_size 
mysql> show status like '%wsrep%'
```

## Step 4b:  Start/add Node 2

```
# like 2.4, but no galera_new_cluster 
# instead
systemctl start mariadb
# see logs of connecting 
journalctl -u mariadb 
```

## Details 4b: Step-by-Step -> Node 1 

```
Schritt 1: z_galera.cnf rüberschieben (von Server 1) 
Schritt 2: plugin -eintrag ändern -4 raus
Schritt 3: galera.cnf umbenennen galera.cnf.NOT (aus Installation 10.3, wenn vorhanden) 
Schritt 4: 
semanage permissive -a mysqld_t

Schritt 5: Firewall - Regeln eintragen
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --zone=public --add-port=4444/tcp --permanent
firewall-cmd --zone=public --add-port=4567/udp --permanent
firewall-cmd --zone=public --add-port=4567/tcp --permanent
firewall-cmd --zone=public --add-port=4568/tcp --permanent
firewall-cmd --reload

Schritt 6: Server mit: systemctl.stop mariadb;  systemctl start mariadb 
Schritt 7: Überprüfung: mysql  show status like ‚wsrep%‘ # guckst cluster_size 
```
## Step 4c: Start/add Node 3 

  * Same as 4b 
