# Configurando Cinder Block Storage na Controller Node
```SH

mysql -u root -p
MariaDB [(none)]> CREATE DATABASE cinder;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' IDENTIFIED BY 'qwe123qwe';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' IDENTIFIED BY 'qwe123qwe';

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
