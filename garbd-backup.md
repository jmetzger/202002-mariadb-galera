# Back-Ups with Galera Arbitrator

Although you can make a back-up fairly easily with mysqldump by removing a node from the cluster, and you can also make a back-up with rsync by shutting down mysqld on a node, it can be done more gracefully and with minimal interference with the Galera Cluster by using Galera Arbitrator.

Galera Arbitrator can receive a request to make a back-up, manually from the command-line using the garbd daemon, or it can be initiated automatically by a scheduling tool like cron.

The Arbitrator chooses a node to be the donor—unless we tell it which to use. That node is then desynced. Incidentally, the back-up can be requested from any node in the cluster, and any node can be used to make the back-up.

The donor node will then execute the back-up script. We’ll have to create such a script to use whatever tools we prefer and back-up how and where we want. Once the back-up is completed, the node will be re-synchronized.

Let’s take a look at how to configure Galera Arbitrator. Then we’ll look at how to create a back-up script.

## Configure Galera Arbitrator

Galera Arbitrator is included in the Galera Cluster software. The Galera Arbitrator daemon may be called upon from the command-line for single functions, like making a back-up. To do this, we’ll have to construct a simple script to run whichever tool we prefer, such as mysqldump.

First, using a plain text editor, we’ll create a configuration file for the Arbitrator daemon. Its file name and location aren’t important, but /etc/garbd.cnf is a good choice. Below is an example:

```
group='galera-training'
address="gcomm://172.31.30.39:4567,172.31.18.53:4567,172.31.26.106:4567"
options="gmcast.listen_addr=tcp://0.0.0.0:4444"
donor=“galera-3"
log='/var/log/garbd.log'
sst='backup_mysqldump'
```

The group parameter is to give the name of the cluster. This is equivalent to the parameter wsrep_cluster_name. The address parameter is equivalent to the wsrep_cluster_address parameter, for listing the IP addresses of the nodes in the cluster and the ports to use.

The options parameter is equivalent to wsrep_provider_options, but we have to specify gmcast.listen_addr and the address and port on which garbd daemon will listen for connections from other nodes.

The donor parameter is used to specify which node will perform the backup. The log parameter is used to specify the path and name of the log file.

The sst option provides the suffix of the back-up script. 

```
It must be in the /usr/bin directory and start with the name wsrep_sst_. 
So in the example here, the script’s name is wsrep_sst_backup_mysqldump. 
```

Those are all of the settings we need for Galera Arbitrator. We can actually set any or all of them from the command-line, but a configuration file is more convenient. Now we need a back-up script.

## Back-Up Script

Below is a very simple back-up script which uses bash and mysqldump. Ht’s meant to be simple. 

```
#!/bin/bash

# SET VARIABLES
db_user='admin_backup'
db_passwd='Rover123!'

backup_dir='/backup'
backup_sub_dir='temp'

today=`date +"%Y%m%d"`
backup_today="galera-mysqldump-$today.sql"
gtid_file="gtid-$today.dat"


# LOAD COMMON SCRIPT
. /usr/bin/wsrep_sst_common


# COPY CONFIGURATION FILES & SAVE GTID
cp /etc/my.cnf $backup_dir/$backup_sub_dir/
cp /etc/garb.cnf $backup_dir/$backup_sub_dir/

echo "GTID: ${WSREP_SST_OPT_GTID}" > $backup_dir/$backup_sub_dir/$gtid_file


# SAVE DATABASE TO DUMP FILE
mysqldump --user="$db_user" --password="$db_passwd" \
          --flush-logs --all-databases \
          > $backup_dir/$backup_sub_dir/$backup_today

# ARCHIVE BACK-UP FILES
cd $backup_sub_dir
tar -czf $backup_dir/$backup_today.tgz * --transform "s,^,$backup_today/,"
```

## Explanation 

The first section defines some variables: the user name and password it will use with mysqldump. There are more secure ways to do this, but we’re trying to keep this script very straightforward. The next pair of variables contain the paths for storing the back-ups. Then there’s a variables that will store today’s date to be part of the name of the back-up file.

The next stanza assembles the names of files: the dump file (e.g., galera-mysqldump-20191025.sql), the data file for the GTID (e.g., gtid-20191025.dat)

The second section is small, but it’s important. It loads the common SST script that Galera Arbitrator uses. Among other things, it will contain some variables we can use. In particular, it has the variable which contains the GTID.

In the third section, the script makes copies of the database configuration files (e.g., my.cnf) and the Galera Arbitrator configuration file (e.g., garb.cnf). The last line in this section gets the GTID variable from the common script, and writes it to a text file.

The next section uses mysqldump to generate a dump file containing all of the databases on the node. We looked at the options already, so we’ll move on.

In the last section, the script will use the tar command to create an archive file that will contain all of the files in the back-up sub-directory, and then zip that file (e.g., galera-mysqldump-20191025.tgz). The --transform option is so that when it’s extracted, it will put everything in a directory named based on today’s date.

That’s everything: it’s everything we need. Below is how we would execute this script from the command line, in conjunction with Galera Arbitrator:

garbd --cfg /etc/garb.cnf
This one option, --cfg is to give the path and name of the Galera Arbitrator configuration file. The daemon will read it before doing anything.

