# Configurando Cinder Block Storage na Controller Node

## Configurando Banco de Dados para Cinder

```SH

mysql -u root -p
MariaDB [(none)]> CREATE DATABASE cinder;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' IDENTIFIED BY 'qwe123qwe';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' IDENTIFIED BY 'qwe123qwe';
```
## Configurando Usuario e Endpooint para o Cinder
```SH
source admin-rc 
{admin}> openstack user create  --domain default --password-prompt cinder
{admin}> openstack role add --project service --user cinder admin
{admin}> openstack service create --name cinderv2 --description "OpenStack Block Storage" volumev2
{admin}> openstack service create --name cinderv3 --description "OpenStack Block Storage" volumev3
{admin}> openstack endpoint create --region RegionOne   volumev2 public http://controller:8776/v2/%\(project_id\)s
{admin}> openstack endpoint create --region RegionOne   volumev2 internal http://controller:8776/v2/%\(project_id\)s
{admin}> openstack endpoint create --region RegionOne   volumev2 admin http://controller:8776/v2/%\(project_id\)s
{admin}> openstack endpoint create --region RegionOne   volumev3 public http://controller:8776/v3/%\(project_id\)s
{admin}> openstack endpoint create --region RegionOne   volumev3 internal http://controller:8776/v3/%\(project_id\)s
{admin}> openstack endpoint create --region RegionOne   volumev3 admin http://controller:8776/v3/%\(project_id\)s
```

## Instalando o Cinder na Controller Node
```SH
yum install -y openstack-cinder
```

## Confiurando Cinder
vim /etc/cinder/cinder.conf:
```SH

[DEFAULT]
transport_url = rabbit://openstack:qwe123qwe@controller
auth_strategy = keystone
my_ip = 10.0.10.11

[database]
connection = mysql+pymysql://cinder:qwe123qwe@controller/cinder

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_id = default
user_domain_id = default
project_name = service
username = cinder
password = qwe123qwe

[oslo_concurrency]
lock_path = /var/lib/cinder/tmp
```
## Populando banco do Cinder
```SH
su -s /bin/sh -c "cinder-manage db sync" cinder
```
**===== Configuração que Segue deve ser feita na Compute Node =====**
## Configurando Nova na Compute Node

vim /etc/nova/nova.conf:
```SH
[cinder]
os_region_name = RegionOne
```
**=====+=====**


## Reiniciando serviços na Controller Node

```SH
systemctl restart openstack-nova-api
systemctl enable openstack-cinder-api.service openstack-cinder-scheduler.service
systemctl start openstack-cinder-api.service openstack-cinder-scheduler.service
```
