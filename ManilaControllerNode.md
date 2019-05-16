![Manila]()

## Criando base de dados do Manila

```SH
# mysql -u root -p

CREATE DATABASE manila;
GRANT ALL PRIVILEGES ON manila.* TO 'manila'@'localhost' IDENTIFIED BY 'qwe123qwe';
GRANT ALL PRIVILEGES ON manila.* TO 'manila'@'%' IDENTIFIED BY 'qwe123qwe';
```

## Criando usuário e serviço do Manila e adicionando ao projeto service

```SH
$ source admin-rc
$ openstack user create --domain default --password qwe123qwe manila
$ openstack role add --project service --user manila admin
$ openstack service create --name manila --description "OpenStack Shared File Systems" share
$ openstack service create --name manilav2 --description "OpenStack Shared File Systems V2" sharev2
```

## Adicionando Endpoints do Manila 

```SH
$ openstack endpoint create --region RegionOne share public http://controller:8786/v1/%\(tenant_id\)s
$ openstack endpoint create --region RegionOne share internal http://controller:8786/v1/%\(tenant_id\)s
$ openstack endpoint create --region RegionOne share admin http://controller:8786/v1/%\(tenant_id\)s
$ openstack endpoint create --region RegionOne sharev2 public http://controller:8786/v2/%\(tenant_id\)s
$ openstack endpoint create --region RegionOne sharev2 internal http://controller:8786/v2/%\(tenant_id\)s
$ openstack endpoint create --region RegionOne sharev2 admin http://controller:8786/v2/%\(tenant_id\)s
```

## Instalando OpenStack Manila

```SH
# yum install openstack-manila python-manilaclient
```

## Configurando Manila

Edite o arquivo /etc/manila/manila.conf:

```SH
[DEFAULT]
transport_url = rabbit://openstack:qwe123qwe@controller
default_share_type = default_share_type
share_name_template = share-%s
rootwrap_config = /etc/manila/rootwrap.conf
api_paste_config = /etc/manila/api-paste.ini
auth_strategy = keystone
my_ip = 10.0.10.11

[database]
connection = mysql+pymysql://manila:qwe123qwe@controller/manila

[keystone_authtoken]
memcached_servers = controller:11211
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = manila
password = qwe123qwe

[oslo_concurrency]
lock_path = /var/lock/manila
```

## Populando a base de dados do Manila:
```SH
# su -s /bin/sh -c "manila-manage db sync" manila
```
     
## Iniciando Manila e garantindo inicialização no boot do sistema
```SH    
# systemctl enable openstack-manila-api.service openstack-manila-scheduler.service
# systemctl start openstack-manila-api.service openstack-manila-scheduler.service
```
