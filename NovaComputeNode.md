![nova](https://blog.yarwood.me.uk/img/nova.png)

# Configurando Nova no Compute Node

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
