<h1 align="center">Cài đặt OpenStack Manual trên CentOS 7</h1>

# Phần I. Chuẩn bị

## 1. Danh sách IP, host
| Hostname | hardware | Interface |
|--------------|-------|------|
| Controller01| <ul><li>2 CPU - 2GB RAM</li><li>20GB Disk OS</li></ul>| <ul><li>eth0: 192.168.33.21  (MNGT)</li><li>eth1: 192.168.34.21 (CEPH_COM)</li><li>eth2: 192.168.35.21 (DATA_VM)</li><li>eth3: 192.168.36.21 (Provider)</li></ul>|
| Compute01 | <ul><li>6 CPU - 8GB RAM</li><li>20GB Disk OS</li></ul>| <ul><li>eth0: 192.168.33.22  (MNGT)</li><li>eth1: 192.168.34.22 (CEPH_COM)</li><li>eth2: 192.168.35.22 (DATA_VM)</li><li>eth3: 192.168.36.22 (Provider)</li></ul>|


## 2. Bật chế độ ảo hóa trên KVM

- Ở đây triển khai trên môi trường ảo hóa KVM nên ta phải bật mode ảo hóa đối với máy ảo được tạo ra.

**` Thực hiện trên node KVM vật lý `**
- Kiểm tra xem ảo hóa đã được enable hay chưa:
```sh
- Đối với Server sử dụng chip Intel:
cat /sys/module/kvm_intel/parameters/nested

- Đối với Server sử dụng chip AMD:
cat /sys/module/kvm_amd/parameters/nested
```
- Kết quả thường sẽ trả về là N:

```sh
[root@masterkvm7101 ~]# cat /sys/module/kvm_intel/parameters/nested
N
```

> Lưu ý: Tiến hành `shutdown` tất cả các VM trên Server **`KVM`**.

- Để enable ảo hóa trên KVM, tạo một file có đường dẫn /etc/modprobe.d/kvm-nested.conf và thêm cấu hình như sau:
```sh
[root@masterkvm7101 ~]# vi /etc/modprobe.d/kvm-nested.conf
options kvm-intel nested=1
options kvm-intel enable_shadow_vmcs=1
options kvm-intel enable_apicv=1
options kvm-intel ept=1
```
- Tiến hành Remove **`kvm_intel`** module sau đó thêm lại:
```sh
[root@masterkvm7101 ~]# modprobe -r kvm_intel
[root@masterkvm7101 ~]# modprobe -a kvm_intel
```
- Khởi động các VM lên và tiến hành kiểm tra:
```sh
[root@Controller01 ~]# cat /proc/cpuinfo | egrep -c "vmx|svm"
2
[root@Controller01 ~]# lscpu
```
Kết quả trả về **`Virtualization type: full`** là thành công.
<h3 align="center"><img src="..\..\03-Images\Lab\57.png"></h3>

# Phần II. Cấu hình Node Controller

## 1. Cấu hình cơ bản

- Disable firewalld
```sh
sudo systemctl disable firewalld
sudo systemctl stop firewalld
sudo systemctl disable NetworkManager
sudo systemctl stop NetworkManager
sudo systemctl enable network
sudo systemctl start network
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```
- Cấu hình các mode sysctl
```sh
echo 'net.ipv4.conf.all.arp_ignore = 1'  >> /etc/sysctl.conf
echo 'net.ipv4.conf.all.arp_announce = 2'  >> /etc/sysctl.conf
echo 'net.ipv4.conf.all.rp_filter = 2'  >> /etc/sysctl.conf
echo 'net.netfilter.nf_conntrack_tcp_be_liberal = 1'  >> /etc/sysctl.conf
cat << EOF >> /etc/sysctl.conf
net.ipv4.ip_nonlocal_bind = 1
net.ipv4.tcp_keepalive_time = 6
net.ipv4.tcp_keepalive_intvl = 3
net.ipv4.tcp_keepalive_probes = 6
net.ipv4.ip_forward = 1
net.ipv4.conf.all.rp_filter = 0
net.ipv4.conf.default.rp_filter = 0
EOF
```
- kiểm tra lại cấu hình
```sh
sysctl -p
```

- Khai báo file hosts:
```sh
echo "192.168.33.21 Controller01" >> /etc/hosts
echo "192.168.33.22 COMPUTE01" >> /etc/hosts
```
- Khai báo repo mariadb và update
```sh
echo '[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.5/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1' >> /etc/yum.repos.d/MariaDB.repo
yum -y update
```

- Tạo SSH key và coppy sang các node compute

```sh
ssh-keygen -t rsa -f /root/.ssh/id_rsa -q -P ""
ssh-copy-id -o StrictHostKeyChecking=no -i /root/.ssh/id_rsa.pub root@Controller01
ssh-copy-id -o StrictHostKeyChecking=no -i /root/.ssh/id_rsa.pub root@COMPUTE01
scp /root/.ssh/id_rsa root@COMPUTE01:/root/.ssh/
```

- Đứng từ controller tiến hành SSH đến node compute
```sh
[root@Controller01 ~]# ssh root@COMPUTE01
Last login: Tue Mar  1 00:31:01 2022 from 192.168.33.133
[root@COMPUTE01 ~]#
```

- Cài đặt các gói cần thiết
```sh
yum -y install centos-release-openstack-queens
yum -y install crudini wget vim
yum -y install python-openstackclient openstack-selinux python2-PyMySQL
```

## 2. Cài đặt và cấu hình NTP
```sh
yum -y install chrony
sed -i 's/server 0.centos.pool.ntp.org iburst/ \
server 1.vn.pool.ntp.org iburst \
server 0.asia.pool.ntp.org iburst \
server 3.asia.pool.ntp.org iburst/g' /etc/chrony.conf
sed -i 's/server 1.centos.pool.ntp.org iburst/#/g' /etc/chrony.conf
sed -i 's/server 2.centos.pool.ntp.org iburst/#/g' /etc/chrony.conf
sed -i 's/server 3.centos.pool.ntp.org iburst/#/g' /etc/chrony.conf
sed -i 's/#allow 192.168.0.0\/16/allow 10.10.10.0\/24/g' /etc/chrony.conf
```

```sh
systemctl enable chronyd.service
systemctl start chronyd.service
chronyc sources
```

## 3. Cài đặt và cấu hình memcache

- Cài đặt memcached
```sh
yum install -y memcached
```
- Cấu hình connect local
```sh
vi /etc/sysconfig/memcached
cập nhật thông số:

PORT="11211"
USER="memcached"
MAXCONN="10240"
CACHESIZE="1024"
OPTIONS="-l 127.0.0.1"
```

- khởi động dịch vụ
```sh
systemctl enable memcached.service
systemctl restart memcached.service
```

## 4. Cài đặt và cấu hình MariaDB

- Cài đặt
```sh
yum install -y mariadb mariadb-server python2-PyMySQL
```
- Copy lại file cấu hình gốc:
```sh
cp /etc/my.cnf.d/server.cnf /etc/my.cnf.d/server.cnf.orig
rm -rf /etc/my.cnf.d/server.cnf
```
- Config file cấu hình mới:
```sh
cat << EOF > /etc/my.cnf.d/openstack.cnf
[mysqld]
bind-address = 192.168.33.21
default-storage-engine = innodb
innodb_file_per_table
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
EOF
```
- Khởi động lại service
```sh
systemctl enable mariadb.service
systemctl restart mariadb.service
```
> đặt lại mật khẩu MariaDB: mysql_secure_installation, tránh các ký tự #,@ ở cuối

## 5. Cài đặt và cấu hình RabbitMQ
- Cài đặt Erlang, các gói phụ trợ
```sh
yum -y install epel-release
yum update -y
yum -y install socat wget
cd ~ && wget https://packages.erlang-solutions.com/erlang/rpm/centos/7/x86_64/esl-erlang_24.0.2-1~centos~7_amd64.rpm
sudo yum -y install esl-erlang_24.0.2-1~centos~7_amd64.rpm
```
- Cài đặt
```sh
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.9.7/rabbitmq-server-3.9.7-1.el7.noarch.rpm
rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
sudo yum -y install rabbitmq-server-3.9.7-1.el7.noarch.rpm
```
- Cấu hình rabbitmq
```sh
systemctl enable rabbitmq-server.service
systemctl start rabbitmq-server.service
rabbitmq-plugins enable rabbitmq_management
systemctl restart rabbitmq-server
curl -O http://localhost:15672/cli/rabbitmqadmin
chmod a+x rabbitmqadmin
mv rabbitmqadmin /usr/sbin/
rabbitmqadmin list users
```
- Cấu hình trên node controller:
```sh
rabbitmqctl add_user openstack 0962012918tT#
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
rabbitmqctl set_user_tags openstack administrator
```

## 6. Cài đặt keystone
- Tạo Database:
```sh
mysql -u root -p
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY '0962012918tT';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY '0962012918tT';
exit
```
- Cài đặt package
```sh
yum install openstack-keystone httpd mod_wsgi -y
```

- Cấu hình bind Port:
```sh
cp /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
sed -i -e 's/VirtualHost \*/VirtualHost 192.168.33.21/g' /etc/httpd/conf.d/wsgi-keystone.conf
sed -i -e 's/Listen 5000/Listen 192.168.33.21:5000/g' /etc/httpd/conf.d/wsgi-keystone.conf
sed -i -e 's/Listen 35357/Listen 192.168.33.21:35357/g' /etc/httpd/conf.d/wsgi-keystone.conf
sed -i -e 's/^Listen.*/Listen 192.168.33.21:80/g' /etc/httpd/conf/httpd.conf
```





cat << EOF >> /etc/keystone/keystone.conf
[DEFAULT]
[assignment]
[auth]
[cache]
[catalog]
[cors]
[credential]
[database]
connection = mysql+pymysql://keystone:0962012918tT@192.168.33.21/keystone
[domain_config]
[endpoint_filter]
[endpoint_policy]
[eventlet_server]
[federation]
[fernet_tokens]
[healthcheck]
[identity]
[identity_mapping]
[ldap]
[matchmaker_redis]
[memcache]
[oauth1]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[paste_deploy]
[policy]
[profiler]
[resource]
[revoke]
[role]
[saml]
[security_compliance]
[shadow_users]
[signing]
[token]
provider = fernet
[tokenless_auth]
[trust]
EOF

keystone-manage bootstrap --bootstrap-password 0962012918tT \
--bootstrap-admin-url http://192.168.33.21:5000/v3/ \
--bootstrap-internal-url http://192.168.33.21:5000/v3/ \
--bootstrap-public-url http://192.168.33.21:5000/v3/ \
--bootstrap-region-id RegionOne

export OS_USERNAME=admin
export OS_PASSWORD=0962012918tT
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://192.168.33.21:35357/v3
export OS_IDENTITY_API_VERSION=3

openstack --os-auth-url http://192.168.33.99:35357/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name admin --os-username admin token issue



cat << EOF >> admin-openrc
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=0962012918tT
export OS_AUTH_URL=http://192.168.33.21:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF


cat << EOF >> demo-openrc
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=0962012918tT
export OS_AUTH_URL=http://192.168.33.21:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF

mysql -u root -p0962012918tT
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY '0962012918tT';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY '0962012918tT';
exit

openstack user create --domain default --password 0962012918tT glance
openstack role add --project service --user glance admin
openstack service create --name glance --description "OpenStack Image" image



openstack endpoint create --region RegionOne image public http://192.168.33.21:9292
openstack endpoint create --region RegionOne image admin http://192.168.33.21:9292
openstack endpoint create --region RegionOne image internal http://192.168.33.21:9292

cat << EOF >> /etc/glance/glance-api.conf
[DEFAULT]
bind_host = 192.168.33.21
registry_host = 192.168.33.21
[cors]
[database]
connection = mysql+pymysql://glance:0962012918tT@192.168.33.21/glance
[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
[image_format]
[keystone_authtoken]
auth_uri = http://192.168.33.21:5000
auth_url = http://192.168.33.21:5000
memcached_servers = 192.168.33.21:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = 0962012918tT
[matchmaker_redis]
[oslo_concurrency]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[paste_deploy]
flavor = keystone
[profiler]
[store_type_location_strategy]
[task]
[taskflow_executor]
EOF

cat << EOF >> /etc/glance/glance-registry.conf
[DEFAULT]
bind_host = 192.168.33.21
[database]
connection = mysql+pymysql://glance:0962012918tT@192.168.33.21/glance
[keystone_authtoken]
auth_uri = http://192.168.33.21:5000
auth_url = http://192.168.33.21:5000
memcached_servers = 192.168.33.21
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = 0962012918tT
[matchmaker_redis]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_policy]
[paste_deploy]
flavor = keystone
[profiler]
EOF

mysql -u root -p0962012918tT
CREATE DATABASE nova_api;
CREATE DATABASE nova;
CREATE DATABASE nova_cell0;
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY '0962012918tT';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY '0962012918tT';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY '0962012918tT';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY '0962012918tT';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY '0962012918tT';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY '0962012918tT';
exit


openstack user create --domain default --password 0962012918tT nova
openstack role add --project service --user nova admin
openstack service create --name nova --description "OpenStack Compute" compute

openstack endpoint create --region RegionOne compute public http://192.168.33.21:8774/v2.1
openstack endpoint create --region RegionOne compute admin http://192.168.33.21:8774/v2.1
openstack endpoint create --region RegionOne compute internal http://192.168.33.21:8774/v2.1


openstack user create --domain default --password 0962012918tT placement
openstack role add --project service --user placement admin
openstack service create --name placement --description "Placement API" placement



openstack endpoint create --region RegionOne placement public http://192.168.33.21:8778
openstack endpoint create --region RegionOne placement admin http://192.168.33.21:8778
openstack endpoint create --region RegionOne placement internal http://192.168.33.21:8778

yum install -y openstack-nova-api openstack-nova-conductor openstack-nova-console openstack-nova-novncproxy openstack-nova-scheduler openstack-nova-placement-api



cat << EOF >> /etc/nova/nova.conf
[DEFAULT]
my_ip = 192.168.33.21
enabled_apis = osapi_compute,metadata
use_neutron = True
osapi_compute_listen=192.168.33.21
metadata_host=192.168.33.21
metadata_listen=192.168.33.21
metadata_listen_port=8775
firewall_driver = nova.virt.firewall.NoopFirewallDriver
transport_url = rabbit://openstack:0962012918tT@192.168.33.21:5672
[api]
auth_strategy = keystone
[api_database]
connection = mysql+pymysql://nova:0962012918tT@192.168.33.21/nova_api
[barbican]
[cache]
backend = oslo_cache.memcache_pool
enabled = true
memcache_servers = 192.168.33.21:11211
[cells]
[cinder]
[compute]
[conductor]
[console]
[consoleauth]
[cors]
[crypto]
[database]
connection = mysql+pymysql://nova:0962012918tT@192.168.33.21/nova
[devices]
[ephemeral_storage_encryption]
[filter_scheduler]
[glance]
api_servers = http://192.168.33.21:9292
[guestfs]
[healthcheck]
[hyperv]
[ironic]
[key_manager]
[keystone]
[keystone_authtoken]
auth_url = http://192.168.33.21:5000/v3
memcached_servers = 192.168.33.21:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = 0962012918tT
[libvirt]
[matchmaker_redis]
[metrics]
[mks]
[neutron]
[notifications]
[osapi_v21]
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
rabbit_ha_queues = true
rabbit_retry_interval = 1
rabbit_retry_backoff = 2
amqp_durable_queues= true
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[pci]
[placement]
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://192.168.33.21:5000/v3
username = placement
password = 0962012918tT
[quota]
[rdp]
[remote_debug]
[scheduler]
discover_hosts_in_cells_interval = 300
[serial_console]
[service_user]
[spice]
[upgrade_levels]
[vault]
[vendordata_dynamic_auth]
[vmware]
[vnc]
novncproxy_host=192.168.33.21
enabled = true
vncserver_listen = 192.168.33.21
vncserver_proxyclient_address = 192.168.33.21
novncproxy_base_url = http://192.168.33.21:6080/vnc_auto.html
[workarounds]
[wsgi]
[xenserver]
[xvp]
EOF



sed -i -e 's/VirtualHost \*/VirtualHost 192.168.33.21/g' /etc/httpd/conf.d/00-nova-placement-api.conf
sed -i -e 's/Listen 8778/Listen 192.168.33.21:8778/g' /etc/httpd/conf.d/00-nova-placement-api.conf




mysql -u root -p0962012918tT
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY '0962012918tT';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY '0962012918tT';
exit

openstack user create --domain default --password 0962012918tT neutron
openstack role add --project service --user neutron admin
openstack service create --name neutron --description "OpenStack Networking" network

cat << EOF >> /etc/neutron/neutron.conf
[DEFAULT]
bind_host = 192.168.33.21
core_plugin = ml2
service_plugins = router
transport_url = rabbit://openstack:0962012918tT@192.168.33.21:5672
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
allow_overlapping_ips = True
dhcp_agents_per_network = 2
[agent]
[cors]
[database]
connection = mysql+pymysql://neutron:0962012918tT@192.168.33.21/neutron
[keystone_authtoken]
auth_uri = http://192.168.33.21:5000
auth_url = http://192.168.33.21:35357
memcached_servers = 192.168.33.21:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = 0962012918tT
[matchmaker_redis]
[nova]
auth_url = http://192.168.33.21:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = 0962012918tT
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
rabbit_retry_interval = 1
rabbit_retry_backoff = 2
amqp_durable_queues = true
rabbit_ha_queues = true
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[quotas]
[ssl]
EOF



cat << EOF >> /etc/neutron/plugins/ml2/linuxbridge_agent.ini
[DEFAULT]
[agent]
[linux_bridge]
physical_interface_mappings = provider:eth3
[network_log]
[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
[vxlan]
enable_vxlan = true
local_ip = 192.168.35.21
l2_population = true
EOF



[neutron]
url = http://192.168.33.21:9696
auth_url = http://192.168.33.21:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = 0962012918tT
service_metadata_proxy = true
metadata_proxy_shared_secret = 0962012918tT


filehtml=/var/www/html/index.html
touch $filehtml
cat << EOF >> $filehtml
<html>
<head>
<META HTTP-EQUIV="Refresh" Content="0.5; URL=http://192.168.33.21/dashboard">
</head>
<body>
<center> <h1>Redirecting to OpenStack Dashboard</h1> </center>
</body>
</html>
EOF








- Chỉnh sua file **`/etc/yum.repos.d/CentOS-QEMU-EV.repo`**

```sh
sed -i 's|baseurl=http:\/\/mirror.centos.org\/$contentdir\/$releasever\/virt\/$basearch\/kvm-common\/|baseurl=http:\/\/mirror.centos.org\/centos\/7\/virt\/x86_64\/kvm-common\/|g' /etc/yum.repos.d/CentOS-QEMU-EV.repo
```

## 2. Cài đặt NOVA

- Cài đặt Packages

```sh
yum install openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables -y
```

- Cấu hình neutron
```sh
cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.org 
rm -rf /etc/neutron/neutron.conf
```
thay thế bằng
```sh
cat << EOF >> /etc/neutron/neutron.conf
[DEFAULT]
transport_url = rabbit://openstack:0962012918tT#@192.168.33.21:5672
auth_strategy = keystone
[agent]
[cors]
[database]
[keystone_authtoken]
auth_uri = http://192.168.33.21:5000
auth_url = http://192.168.33.21:35357
memcached_servers = 192.168.33.21:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = 0962012918tT#
[matchmaker_redis]
[nova]
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
rabbit_ha_queues = true
rabbit_retry_interval = 1
rabbit_retry_backoff = 2
amqp_durable_queues= true
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[quotas]
[ssl]
EOF
```
- Cấu hình file LB agent:
Lưu ý khi chạy đoạn ở dưới chú ý 2 tham số:
physical_interface_mappings = provider:eth1 (interface name provider)

local_ip = 192.168.35.21 (IP dải DATAVM compute01) - đổi sang IP tương ứng với compute02
```sh
cat << EOF >> /etc/neutron/plugins/ml2/linuxbridge_agent.ini
[DEFAULT]
[agent]
[linux_bridge]
physical_interface_mappings = provider:eth3
[network_log]
[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
[vxlan]
enable_vxlan = true
local_ip = 192.168.35.21
l2_population = true
EOF
```