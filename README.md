![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/OpenStack%C2%AE_Logo_2016.svg/1200px-OpenStack%C2%AE_Logo_2016.svg.png)

# Informações sobre o Projeto

Esse passo-a-passo tem como objetivo instruir alguém que queira instalar o OpenStack de forma manual. Apesar de desconsiderar fortemente a realização da implementação do OpenStack de forma manual, a utilização dessa para entendimento de todos os componentes, e como as integrações acontecem, é de extrema valia para evolução de um profissional de cloud. Para uma implementação automatizada, recomendo utilizar ferramentas de gerênciamento de configuração como Ansible, Puppet e Chef


 * Sistema Operacional utilizado: **CENTOS 7 64-bit (7.6.1810)**
 * Versão OpenStack utilizada: **Rocky**
 * Utilização de OpenVSwith: **NÃO**
 * Utilização de Storage com CEPH: **NÃO**
 
 ```SH
 (admin)#  <- Linhas iniciadas com esse prefixo devem ser executadas com conta admin do OpenStack autenticada.
 (user)# <- Linhas iniciadas com esse prefixo devem ser executadas com conta de USUARIO do OpenStack autenticada.
```
# TODAS AS MAQUINAS

As instruções que seguem devem ser realizadas em **TODAS** as maquinas antes de começar a instalação dos serviços do OpenStack

```sh
# yum install centos-release-openstack-rocky -y
# yum install python-openstackclient -y
# yum install openstack-selinux -y
```

# Controller Node [aqui](ControllerNode.md)
 - MariaDB
 - Memcached
 - RabbitMQ

# Demais serviços do Core OpenStack

## [Instalando Keystone na Controller](KeystoneControllerNode.md) 
 - Keystone
 - HTTPd
 - HTTPd mod_wsgi
 

## [Instalando Glance no Controller](GlanceControllerNode.md)
 - OpenStack Glance (Registry e API)
 
 
## Instalando Nova no Controller
#### Criar a base de dados
```SH
CREATE DATABASE nova_api;
CREATE DATABASE nova;
CREATE DATABASE nova_cell0;
CREATE DATABASE placement;
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
yum install openstack-nova-api openstack-nova-conductor openstack-nova-console openstack-nova-novncproxy  openstack-nova-scheduler openstack-nova-placement-api
```

### Garantir serviços subindo no boot:
```SH
systemctl enable openstack-nova-api openstack-nova-conductor openstack-nova-console openstack-nova-novncproxy  openstack-nova-scheduler openstack-nova-placement-api
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

## Compute Node
## Instalando Nova na Compute01 [AQUI](NovaComputeNode.md)

#### Instalando pacote nova-compute
```SH
yum install openstack-nova-compute
```
#### Configurando Nova na Compute
/etc/nova/nova.conf:
```
[DEFAULT]
user_neutron = true
firewall_driver = nova.virt.firewall.NoopFirewallDriver
my_ip = 10.0.10.21
enabled_apis = osapi_compute,metadata
transport_url = rabbit://openstack:qwe123qwe@controller
debug=true

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

[libvirt]
virt_type=qemu

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

[oslo_concurrency]
lock_path = /var/lib/nova/tmp

[placement]
region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:5000/v3
username = placement
password = qwe123qwe

[vnc]
enabled = true
server_listen = 0.0.0.0
server_proxyclient_address = $my_ip
novncproxy_base_url = http://controller:6080/vnc_auto.html
```

### Garantindo serviços do Libvirt ativos: 
```SH
# systemctl enable libvirtd.service openstack-nova-compute.service
# systemctl start libvirtd.service openstack-nova-compute.service
```


## Instalando Neutron no Controller Node []()
### Criando Banco e Privilégios

```SH

MariaDB [(none)] CREATE DATABASE neutron;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'qwe123qwe';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'qwe123qwe';
```

### Adicionando usuário Neutron ao projeto Service
```SH
source admin-rc
{admin}# openstack user create --domain default --password-prompt neutron
{admin}# openstack role add --project service --user neutron admin
{admin}# openstack service create --name neutron --description "OpenStack Networking" network
{admin}# openstack endpoint create --region RegionOne network public http://controller:9696
{admin}# openstack endpoint create --region RegionOne network internal http://controller:9696
{admin}# openstack endpoint create --region RegionOne network admin http://controller:9696
```

## Configurando Self-Service Network
### Instalando Neutron
```SH
# yum install -y openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables
```

### Configurando Neutron
 /etc/neutron/neutron.conf:
 ```
[DEFAULT]
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = true
transport_url = rabbit://openstack:qwe123qwe@controller
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true

[database]
connection = mysql+pymysql://neutron:qwe123qwe@controller/neutron

[keystone_authtoken]
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = qwe123qwe

[nova]
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = qwe123qwe

[oslo_concurrency]
lock_path = /var/lib/neutron/tmp

 ```

### Configurando Plugin Modular Layer 2 (bridge e switching)
/etc/neutron/plugins/ml2/ml2_conf.ini:
```
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security

[ml2_type_flat]
flat_networks = provider

[ml2_type_vxlan]
vni_ranges = 1:1000

[securitygroup]
enable_ipset = true
```

/etc/neutron/plugins/ml2/linuxbridge_agent.ini:
```
[linux_bridge]
physical_interface_mappings = provider:enp0s8

[securitygroup]
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
enable_security_group = true

[vxlan]
enable_vxlan = true
local_ip = 10.0.10.11

```

### Garantindo modulos de bridge carregados no Kernel Linux da Controller Node
```SH
modprobe br_netfilter
sysctl -w net.bridge.bridge-nf-call-iptables=1
sysctl -w net.bridge.bridge-nf-call-ip6tables=1
```

### Configurando Agent Layer 3
/etc/neutron/l3_agent.ini:
```
[DEFAULT]
interface_driver = linuxbridge
```
### Configurando Agent DHCP
/etc/neutron/dhcp_agent.ini:
```
[DEFAULT]
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true
verbose = True
dnsmasq_config_file = /etc/neutron/dnsmasq-neutron.conf
```

### Configurando Agent Metadata
/etc/neutron/metadata_agent.ini:
```SH
[DEFAULT]
nova_metadata_host = controller
metadata_proxy_shared_secret = qwe123qwe
```
### Criando link simbolico do plugin
```
# ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```

### Populando Base de Dados do Neutron
```
# su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```

### Garantindo serviços ativos do Neutron no Controller Node
```SH
# systemctl enable neutron-server.service neutron-linuxbridge-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service neutron-l3-agent.service
# systemctl start neutron-server.service neutron-linuxbridge-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service neutron-l3-agent.service
```

### Reiniciar Nova-API para se comunicar com Neutron
```SH
# systemctl restart openstack-nova-api.service
```

## Configurando Neutron no Compute Node [aqui](NeutronComputeNode.md)
### Instalando Neutron no Compute Node
```SH
# yum install openstack-neutron-linuxbridge ebtables ipset
```
### Configurando Neutron no Compute Node
/etc/neutron/neutron.conf:
```SH
[DEFAULT]
transport_url = rabbit://openstack:qwe123qwe@controller
auth_strategy = keystone

[keystone_authtoken]
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = qwe123qwe

[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
```
### Configurando Linux Bridge no Compute Node
/etc/neutron/plugins/ml2/linuxbridge_agent.ini:
```
[linux_bridge]
physical_interface_mappings = provider:enp0s8
bridge_mappings = provider:enp0s8

[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver

[vxlan]
enable_vxlan = true
local_ip = 10.0.10.21
l2_population = true

```

### Garantindo modulos de bridge carregados no Kernel Linux da Compute Node
```SH
modprobe br_netfilter
sysctl -w net.bridge.bridge-nf-call-iptables=1
sysctl -w net.bridge.bridge-nf-call-ip6tables=1
```

### Garantindo serviços ativos do Neutron no Compute Node
```SH
# systemctl enable neutron-linuxbridge-agent.service
# systemctl start neutron-linuxbridge-agent.service
```

### Reiniciar o serviço do Nova para que ele se comunique com Neutron
```SH
# systemctl restart openstack-nova-compute.service
```

### ======= ***REALIZAR OS COMANDOS QUE SEGUEM NA CONTROLLER*** ======= 
### Confirmando Compute Node no database da Controller:
```SH
# openstack compute service list --service nova-compute
```
### Realizando discover na Controller:
```SH
# su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova
```

# Instalando HORIZON ([OpenStack Dashboard](HorizonControllerNode.md))
