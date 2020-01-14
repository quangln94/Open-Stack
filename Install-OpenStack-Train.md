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
$ yum --enablerepo=centos-openstack-train -y install rabbitmq-server memcached
```
Vào file `/etc/my.cnf.d/mariadb-server.cnf` và `/etc/sysconfig/memcached` add thêm vào các phần như sau:
```sh
$ vim /etc/my.cnf.d/mariadb-server.cnf
# add into [mysqld] section
[mysqld]
# default value 151 is not enough on Openstack Env
max_connections=500
# because sometimes it happens errors with utf8mb4 on Openstack DB
character-set-server=utf8 
```
```sh
$ vi /etc/sysconfig/memcached
# line 5: change (listen all)
OPTIONS="-l 0.0.0.0,::"
```
Enable dịch vụ
```sh
$ systemctl restart mariadb rabbitmq-server memcached
$ systemctl enable mariadb rabbitmq-server memcached
```
Add user và password cho `openstack`
```sh
$ rabbitmqctl add_user openstack password
Creating user "openstack"
$ rabbitmqctl set_permissions openstack ".*" ".*" ".*"
Setting permissions for user "openstack" in vhost "/"
```
Cấu hình firewall allow port cho service services.
```sh
$ firewall-cmd --add-service=mysql --permanent
$ firewall-cmd --add-port={11211/tcp,5672/tcp} --permanent
$ firewall-cmd --reload
```
## 1. Cài đặt và cấu hình keystone

**Thêm User và Database trên MariaDB cho Keystone.**

```sh
$ mysql -u root -p
MariaDB [(none)]> create database keystone;
MariaDB [(none)]> grant all privileges on keystone.* to keystone@'localhost' identified by 'password';
MariaDB [(none)]> grant all privileges on keystone.* to keystone@'%' identified by 'password';
MariaDB [(none)]> flush privileges;
MariaDB [(none)]> exit
```

**Cài đặt Keystone**
```sh
$ yum --enablerepo=centos-openstack-train,epel -y install openstack-keystone openstack-utils python-openstackclient httpd mod_wsgi
```

**Cấu hình Keystone**
```sh
$ vim /etc/keystone/keystone.conf
# line 431: add the line (specify Memcache server)
memcache_servers = 10.0.0.30:11211
# line 571: add the line (MariaDB connection info)
connection = mysql+pymysql://keystone:password@10.0.0.30/keystone
[token]
# line 2435: uncomment
provider = fernet
```
```sh
$ su -s /bin/bash keystone -c "keystone-manage db_sync"
```
initialize keys
```sh
$ keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
$ keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```
define own host (controller host)
```sh
$ export controller=10.0.0.30
```
bootstrap keystone (replace any password you like for "adminpassword" section)
```sh
$ keystone-manage bootstrap --bootstrap-password adminpassword \
--bootstrap-admin-url http://$controller:5000/v3/ \
--bootstrap-internal-url http://$controller:5000/v3/ \
--bootstrap-public-url http://$controller:5000/v3/ \
--bootstrap-region-id RegionOne
```
Cấu hình SELinux và Firewalld
```sh
setsebool -P httpd_use_openstack on
setsebool -P httpd_can_network_connect on
setsebool -P httpd_can_network_connect_db on
firewall-cmd --add-port=5000/tcp --permanent
firewall-cmd --reload
```
Enable config cho Keystone và start Apache httpd.
```sh
$ ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
$ systemctl start httpd
$ systemctl enable httpd
```
Tạo và Load file environment variables.

The password for [OS_PASSWORD] is the one you set it on bootstrapping keystone.

The URL for [OS_AUTH_URL] is the Keystone server's hostname or IP address.

```sh
$ vim ~/keystonerc
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=adminpassword
export OS_AUTH_URL=http://10.0.0.30:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
export PS1='[\u@\h \W(keystone)]\$ '
```
```sh
$ chmod 600 ~/keystonerc
$ source ~/keystonerc
[root@dlp ~(keystone)]# echo "source ~/keystonerc " >> ~/.bash_profile
```
Tạo project
```sh
[root@dlp ~(keystone)]# openstack project create --domain default --description "Service Project" service
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Service Project                  |
| domain_id   | default                          |
| enabled     | True                             |
| id          | 2b9c4eda462e40a4a3d03b0a38f12bf1 |
| is_domain   | False                            |
| name        | service                          |
| options     | {}                               |
| parent_id   | default                          |
| tags        | []                               |
+-------------+----------------------------------+

# confirm settings
[root@dlp ~(keystone)]# openstack project list
+----------------------------------+---------+
| ID                               | Name    |
+----------------------------------+---------+
| 2b9c4eda462e40a4a3d03b0a38f12bf1 | service |
| 4f93fd3e6e184c859641b017115016a4 | admin   |
+----------------------------------+---------+
```
## 2. Cài đặt và cấu hình Glance
