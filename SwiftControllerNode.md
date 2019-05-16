# Instalando Swift

## Configurando usuário e domínio Controller Node
```
{admin}> openstack user create --domain default --password-prompt swift
{admin}> openstack role add --project service --user swift admin
{admin}> openstack service create --name swift --description "OpenStack Object Storage" object-store
{admin}> openstack endpoint create --region RegionOne object-store public http://controller:8080/v1/AUTH_%\(project_id\)s
{admin}> openstack endpoint create --region RegionOne object-store internal http://controller:8080/v1/AUTH_%\(project_id\)s
{admin}> openstack endpoint create --region RegionOne object-store admin http://controller:8080/v1
```


## Instalando Pacotes na Controller Node
```
yum install openstack-swift-proxy python-swiftclient python-keystoneclient python-keystonemiddleware
curl -L -o /etc/swift/proxy-server.conf https://git.openstack.org/cgit/openstack/swift/plain/etc/proxy-server.conf-sample?h=stable/rocky
```

## Condigurando Swift proxy-server
Adicione as configurações abaixo no arquivo /etc/swift/proxy-server.conf:
```SH
[DEFAULT]
bind_port = 8080
user = swift
swift_dir = /etc/swift

[pipeline:main]
pipeline = catch_errors gatekeeper healthcheck proxy-logging cache container_sync bulk ratelimit authtoken keystoneauth container-quotas account-quotas slo dlo versioned_writes proxy-logging proxy-server

[app:proxy-server]
use = egg:swift#proxy
account_autocreate = True

[filter:keystoneauth]
use = egg:swift#keystoneauth
operator_roles = admin,user


[filter:authtoken]
paste.filter_factory = keystonemiddleware.auth_token:filter_factory
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_id = default
user_domain_id = default
project_name = service
username = swift
password = qwe123qwe
delay_auth_decision = True

[filter:cache]
use = egg:swift#memcache
...
memcache_servers = controller:11211

```
