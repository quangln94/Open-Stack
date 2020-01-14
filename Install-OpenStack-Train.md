# Install OpenStack

|Service|Code Name|Description|
|-------|---------|-----------|
|Identity Service|Keystone|User Management|
|Compute Service|Nova|Virtual Machine Management|
|Image Service|Glance|Manages Virtual image like kernel image or disk image|
|Dashboard|Horizon|Provides GUI console via Web browser|
|Object Storage|Swift|Provides Cloud Storage|
|Block Storage|Cinder|Storage Management for Virtual Machine|
|Network Service|Neutron|Virtual Networking Management|
|Orchestration Service|Heat|Provides Orchestration function for Virtual Machine|
|Metering Service|Ceilometer|Provides the function of Usage measurement for accounting|
|Database Service|Trove|Database resource Management|
|Data Processing Service|Sahara|Provides Data Processing function|
|Bare Metal Provisioning|Ironic|Provides Bare Metal Provisioning function|
|Messaging Service|Zaqar|Provides Messaging Service function|
|Shared File System|Manila|Provides File Sharing Service|
|DNS Service|Designate|Provides DNS Server Service|
|Key Manager Service|Barbican|Provides Key Management Service|

## Chuẩn bị

Add Repository of Openstack Train.
```sh
$ yum -y install centos-release-openstack-train
$ sed -i -e "s/enabled=1/enabled=0/g" /etc/yum.repos.d/CentOS-OpenStack-train.repo
```
Install RabbitMQ, Memcached.
```sh
# install from Openstack Train
[root@dlp ~]# yum --enablerepo=centos-openstack-train -y install rabbitmq-server memcached
[root@dlp ~]# vi /etc/my.cnf.d/mariadb-server.cnf
# add into [mysqld] section
[mysqld]
.....
.....
# default value 151 is not enough on Openstack Env
max_connections=500
# because sometimes it happens errors with utf8mb4 on Openstack DB
character-set-server=utf8 

[root@dlp ~]# vi /etc/sysconfig/memcached
# line 5: change (listen all)
OPTIONS="-l 0.0.0.0,::"
[root@dlp ~]# systemctl restart mariadb rabbitmq-server memcached
[root@dlp ~]# systemctl enable mariadb rabbitmq-server memcached
# add openstack user (set any password you like for "password")
[root@dlp ~]# rabbitmqctl add_user openstack password
Creating user "openstack"
[root@dlp ~]# rabbitmqctl set_permissions openstack ".*" ".*" ".*"
Setting permissions for user "openstack" in vhost "/"
```

