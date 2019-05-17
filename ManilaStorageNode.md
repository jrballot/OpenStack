## Instalando Manila na Storage Node¶
```
# yum install openstack-manila-share python2-PyMySQL -y
```
## Install LVM and NFS server packages:
```
# yum install lvm2 nfs-utils nfs4-acl-tools portmap targetcli -y
```
## Iniciando serviço LVM
```
# systemctl enable lvm2-lvmetad.service target.service
# systemctl start lvm2-lvmetad.service target.service
```
## Criando LVM no /dev/sdc:

Definindo /dev/sdc como disco gerenciâvel pelo LVM.
```
devices {
...
filter = [ "a/sda/", "a/sdb/", "a/sdc", "r/.*/"]
...
}
```

```SH
# pvcreate /dev/sdc
# vgcreate manila-volumes /dev/sdc
```

Edit the /etc/manila/manila.conf file and complete the following actions:


```
[DEFAULT]
transport_url = rabbit://openstack:qwe123qwe@controller
default_share_type = default_share_type
rootwrap_config = /etc/manila/rootwrap.conf
auth_strategy = keystone
my_ip = 10.0.10.41
enabled_share_backends = lvm
enabled_share_protocols = NFS

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
lock_path = /var/lib/manila/tmp


 [lvm]
 share_backend_name = LVM
 share_driver = manila.share.drivers.lvm.LVMShareDriver
 driver_handles_share_servers = False
 lvm_share_volume_group = manila-volumes
 lvm_share_export_ip = 10.0.10.41
```

## Diretórios e permissões

Garantir que os diretórios abaixo estejam criados e com permissão de usuário e group para o manila.

```
[root@storage ~]# ls -ld /var/lib/manila/*
drwxr-xr-x. 2 manila manila    6 May 16 02:45 /var/lib/manila/groups
drwxr-xr-x. 9 manila manila 4096 May 17 07:12 /var/lib/manila/mnt
drwxr-xr-x. 2 manila manila 4096 May 17 07:14 /var/lib/manila/tmp
```
      
## Finalizando e iniciando OpenStack Manila na Storage Node

```
# systemctl enable openstack-manila-share.service
# systemctl start openstack-manila-share.service
```
