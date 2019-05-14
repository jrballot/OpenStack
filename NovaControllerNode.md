# Instalando Nova no Controller
#### Criar a base de dados
```SH
MariaDB [(none)]> CREATE DATABASE nova_api;
MariaDB [(none)]> CREATE DATABASE nova;
MariaDB [(none)]> CREATE DATABASE nova_cell0;
MariaDB [(none)]> CREATE DATABASE placement;
```

#### Criando o permisionamento da base de dados
```SH
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'qwe123qwe';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'qwe123qwe';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'qwe123qwe';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'qwe123qwe';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost'   IDENTIFIED BY 'qwe123qwe';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY 'qwe123qwe';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'localhost' IDENTIFIED BY 'qwe123qwe';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%' IDENTIFIED BY 'qwe123qwe';
```

### Criando usuário Nova:
```SH
openstack user create --domain default --password-prompt nova
openstack role add --project service --user nova admin
openstack service create --name nova --description "OpenStack Compute" compute
```

### Criando Endpoints do serviço Compute:
```SH
openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1
openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1
openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1
```

### Criando usuário Placement:
```SH
openstack user create --domain default --password-prompt placement
openstack role add --project service --user placement admin
openstack service create --name placement --description "Placement API" placement
```

### Criando Endpoints para Placement:
```SH
openstack endpoint create --region RegionOne placement public http://controller:8778
openstack endpoint create --region RegionOne placement internal http://controller:8778
openstack endpoint create --region RegionOne placement admin http://controller:8778
```

### Instalando Nova
```SH
yum install openstack-nova-api openstack-nova-conductor openstack-nova-console openstack-nova-novncproxy  openstack-nova-scheduler openstack-nova-placement-api -y
```

### Garantir serviços subindo no boot:
```SH
systemctl enable openstack-nova-api openstack-nova-conductor openstack-nova-console openstack-nova-novncproxy  openstack-nova-scheduler
```

#### Configurando Nova na Controller
/etc/nova/nova.conf:
```
[DEFAULT]
enabled_apis=osapi_compute,metadata
transport_url=rabbit://openstack:qwe123qwe@controller
my_ip=10.0.10.11
use_neutron=true
firewall_driver=nova.virt.firewall.NoopFirewallDriver

[api]
auth_strategy=keystone

[api_database]
connection=mysql+pymysql://nova:qwe123qwe@controller/nova_api

[database]
connection=mysql+pymysql://nova:qwe123qwe@controller/nova

[glance]
api_servers = http://controller:9292

[keystone_authtoken]
auth_url = http://controller:5000/v3
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = qwe123qwe

[neutron]
url = http://controller:9696
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = qwe123qwe
service_metadata_proxy = true
metadata_proxy_shared_secret = qwe123qwe

[placement]
region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:5000/v3
username = placement
password = qwe123qwe

[placement_database]
connection=mysql+pymysql://placement:qwe123qwe@controller/placement

[scheduler]
discover_hosts_in_cells_interval=300

[vnc]
enabled = true
server_listen = $my_ip
server_proxyclient_address = $my_ip
```

### Liberar acesso na API do Placement(bug):
Edite o arquivo /etc/httpd/conf.d/00-nova-placement-api.conf:
```
<Directory /usr/bin>
   <IfVersion >= 2.4>
      Require all granted
   </IfVersion>
   <IfVersion < 2.4>
      Order allow,deny
      Allow from all
   </IfVersion>
</Directory>
```

***Não esquecer de reiniciar o HTTPd:***
```SH
systemctl restart httpd
```

### Populando base de dados do Nova-API e Placement
```SH
# su -s /bin/sh -c "nova-manage api_db sync" nova
# su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
# su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
# su -s /bin/sh -c "nova-manage db sync" nova
```

Validando configuração:
```SH
# su -s /bin/sh -c "nova-manage cell_v2 list_cells" nova
```

### Finalizando instanalação no Controller Node

```SH
# systemctl enable openstack-nova-api.service \
  openstack-nova-consoleauth openstack-nova-scheduler.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service
# systemctl start openstack-nova-api.service \
  openstack-nova-consoleauth openstack-nova-scheduler.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service
```
