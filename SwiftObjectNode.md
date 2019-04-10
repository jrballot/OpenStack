# Object Node

**MODIFICAR O AMBIENTE PARA TER DOIS OBJECT SERVERS CONFIGURADOS**
object01.dexter.com.br 10.0.10.51
object02.dexter.com.br 10.0.10.52

**Adicione dois discos /dev/sdb e /dev/sdc a Maquina Virtual**

## Instalando pacotes
```
yum install xfsprogs rsync
```

## Configurando Rsyncd
vim /etc/rsyncd.conf:
```
uid = swift
gid = swift
log file = /var/log/rsyncd.log
pid file = /var/run/rsyncd.pid
address = IP_DO_OBJECT_NODE

[account]
max connections = 2
path = /srv/node/
read only = False
lock file = /var/lock/account.lock

[container]
max connections = 2
path = /srv/node/
read only = False
lock file = /var/lock/container.lock

[object]
max connections = 2
path = /srv/node/
read only = False
lock file = /var/lock/object.lock
```

Reiniciando Rsyncd
```
systemctl enable rsyncd.service
systemctl start rsyncd.service
```

## Instalando Componentes do Swift no Object Node
```
yum install openstack-swift-account openstack-swift-container openstack-swift-object
```

## Baixando arquivos de configuração para account-server
```
curl -o /etc/swift/account-server.conf https://git.openstack.org/cgit/openstack/swift/plain/etc/account-server.conf-sample?h=stable/rocky
```

## Adicionar os parâmetros abaixo:
vim /etc/swift/account-server.conf:
```

[DEFAULT]
...
bind_ip = IP_DO_OBJECT_NODE
bind_port = 6202
user = swift
swift_dir = /etc/swift
devices = /srv/node
mount_check = True


[pipeline:main]
...
pipeline = healthcheck recon account-server

[filter:recon]
use = egg:swift#recon
...
recon_cache_path = /var/cache/swift

```

# Baixando arquivo de container do Swift
```
curl -o /etc/swift/container-server.conf https://git.openstack.org/cgit/openstack/swift/plain/etc/container-server.conf-sample?h=stable/rocky
```

vim /etc/swift/container-server.conf:
```
[DEFAULT]
bind_ip = IP_DO_OBJECT_NODE
bind_port = 6201
user = swift
swift_dir = /etc/swift
devices = /srv/node
mount_check = True

[pipeline:main]
pipeline = healthcheck recon container-server

[filter:recon]
use = egg:swift#recon
recon_cache_path = /var/cache/swift

```

## Baixando configuração do object server
```
curl -o /etc/swift/object-server.conf https://git.openstack.org/cgit/openstack/swift/plain/etc/object-server.conf-sample?h=stable/rocky
```

## Incluir as alterações
vim /etc/swift/object-server.conf:
```
[DEFAULT]
bind_ip = IP_DO_OBJECT_NODE
bind_port = 6200
user = swift
swift_dir = /etc/swift
devices = /srv/node
mount_check = True

[pipeline:main]
pipeline = healthcheck recon object-server


[filter:recon]
use = egg:swift#recon
recon_cache_path = /var/cache/swift
recon_lock_path = /var/lock

```


