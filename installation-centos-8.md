# Installation (Centos 8)

## Repo - Configuration (Centos 8)  

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

### Installation (centos 8) 

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
