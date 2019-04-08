## Instalando Horizon na Controller Node

!(horizon)[https://git.garr.it/uploads/-/system/project/avatar/70/OpenStack_Project_Horizon_vertical.jpg)

## Instalando Dashboard
```SH
yum install openstack-dashboard
```

## Configurando Dashboard
vim /etc/openstack-dashboard/local_settings

```SH
OPENSTACK_HOST = "controller"
ALLOWED_HOSTS = [...,"localhost","127.0.0.1"]
```

## Descomentar 
```SH
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'

CACHES = {
    'default': {
         'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
         'LOCATION': 'controller:11211',
    }
}

OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
OPENSTACK_API_VERSIONS = {
    "identity": 3,
    "image": 2,
    "volume": 2,
}
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "Default"
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
TIME_ZONE = "America/Sao_Paulo"
```

## Atualizando configuração do VHost
vim /etc/httpd/conf.d/openstack-dashboard.conf
```SH
WSGIApplicationGroup %{GLOBAL}
```
