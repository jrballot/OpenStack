# Instalando Neutron na Controller Node

![neutron](https://aptira.com/wp-content/uploads/2017/04/openstack_neutron.png)

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
