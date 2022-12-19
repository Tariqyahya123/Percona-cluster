## Percona-XtraDB-Cluster
### Install instructions for Ubuntu 18

### Assumptions
|Role|machine name|IP address|Memory|Operating System|
|-|-|-|-|-|
|master node 1|first-node|10.10.0.10|1G|Ubuntu 18|
|master node 2|second-node|10.10.0.11|1G|Ubuntu 18|
|master node 2|third-node|10.10.0.12|1G|Ubuntu 18|

### On First node
##### Add Percona Repository
```
wget https://repo.percona.com/apt/percona-release_0.1-6.$(lsb_release -sc)_all.deb
dpkg -i percona-release_0.1-6.$(lsb_release -sc)_all.deb
```
##### Install Percona-XtraDB-Cluster
```
apt-get update
apt-get install -y percona-xtradb-cluster-57
systemctl stop mysql
```
##### Configure Replication Settings
```
cat >>/etc/mysql/my.cnf<<EOF
[mysqld]
wsrep_provider=/usr/lib/libgalera_smm.so
wsrep_cluster_name=testcluster
wsrep_cluster_address=gcomm://
wsrep_node_name=first-node
wsrep_node_address=10.10.0.10
wsrep_sst_method=xtrabackup-v2
wsrep_sst_auth=repuser:reppassword
pxc_strict_mode=ENFORCING
binlog_format=ROW
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
EOF
```
##### Bootstrap/Initialize the Cluster
```
systemctl start mysql
```
##### Create Replication User
```
mysql -uroot -p -e "create user repuser@localhost identified by 'reppassword'"
mysql -uroot -p -e "grant reload, replication client, process, lock tables on *.* to repuser@localhost"
mysql -uroot -p -e "flush privileges"
```
##### Update Replication configuration
```
sed -i 's/^wsrep_cluster_address=.*/wsrep_cluster_address=gcomm:\/\/10.10.0.10,10.10.0.11,10.10.0.12/' /etc/mysql/my.cnf
```

### On Second node
##### Add Percona Repository
```
wget https://repo.percona.com/apt/percona-release_0.1-6.$(lsb_release -sc)_all.deb
dpkg -i percona-release_0.1-6.$(lsb_release -sc)_all.deb
```
##### Install Percona-XtraDB-Cluster
```
apt-get update
apt-get install -y percona-xtradb-cluster-57
systemctl stop mysql
```
##### Configure Replication Settings
```
cat >>/etc/mysql/my.cnf<<EOF
[mysqld]
wsrep_provider=/usr/lib/libgalera_smm.so
wsrep_cluster_name=testcluster
wsrep_cluster_address=gcomm://10.10.0.10,10.10.0.11,10.10.0.12
wsrep_node_name=second-node
wsrep_node_address=10.10.0.11
wsrep_sst_method=xtrabackup-v2
wsrep_sst_auth=repuser:reppassword
pxc_strict_mode=ENFORCING
binlog_format=ROW
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
EOF
```
##### Start mysql to join the cluster
```
systemctl start mysql
```

### On Third node
##### Add Percona Repository
```
wget https://repo.percona.com/apt/percona-release_0.1-6.$(lsb_release -sc)_all.deb
dpkg -i percona-release_0.1-6.$(lsb_release -sc)_all.deb
```
##### Install Percona-XtraDB-Cluster
```
apt-get update
apt-get install -y percona-xtradb-cluster-57
systemctl stop mysql
```
##### Configure Replication Settings
```
cat >>/etc/mysql/my.cnf<<EOF
[mysqld]
wsrep_provider=/usr/lib/libgalera_smm.so
wsrep_cluster_name=testcluster
wsrep_cluster_address=gcomm://10.10.0.10,10.10.0.11,10.10.0.12
wsrep_node_name=third-node
wsrep_node_address=10.10.0.12
wsrep_sst_method=xtrabackup-v2
wsrep_sst_auth=repuser:reppassword
pxc_strict_mode=ENFORCING
binlog_format=ROW
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
EOF
```
##### Start mysql to join the cluster
```
systemctl start mysql
```
```
