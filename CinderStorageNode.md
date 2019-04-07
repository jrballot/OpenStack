# Configurando o Cinder Block Storage na Storage Node
```SH

yum install lvm2 device-mapper-persistent-data
systemctl enable lvm2-lvmetad.service
systemctl start lvm2-lvmetad.service

pvcreate /dev/sdb
vgcreate cinder-volumes /dev/sdb

vim /etc/lvm/lvm.conf:
devices {
...
filter = [ "a/sda/", "a/sdb/", "r/.*/"]

Faça o mesmo para Compute Node
vim /etc/lvm/lvm.cong:
devices {
...
filter = [ "a/sda/", "r/.*/"]

yum install openstack-cinder targetcli python-keystone

vim /etc/cinder/cinder.conf:
[DEFAULT]
transport_url = rabbit://openstack:qwe123qwe@controller
auth_strategy = keystone
my_ip = 10.0.10.41
enabled_backends = lvm
glance_api_servers = http://controller:9292

[database]
connection = mysql+pymysql://cinder:qwe123qwe@controller/cinder


[keystone_authtoken]
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_id = default
user_domain_id = default
project_name = service
username = cinder
password = qwe123qwe

[lvm]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_group = cinder-volumes
iscsi_protocol = iscsi
iscsi_helper = lioadm

[oslo_concurrency]
lock_path = /var/lib/cinder/tmp

systemctl enable openstack-cinder-volume.service target.service
systemctl start openstack-cinder-volume.service target.service

```
