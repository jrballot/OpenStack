# Instalando Neutron no Compute Node

![neutron](https://aptira.com/wp-content/uploads/2017/04/openstack_neutron.png)

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
