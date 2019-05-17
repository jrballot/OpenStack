
# Instalando OpenStack Murano

## Criando base de dados

```
# mysql -u root -p

CREATE DATABASE murano;

GRANT ALL PRIVILEGES ON murano.* TO 'murano'@'localhost' IDENTIFIED BY 'qwe123qwe';
GRANT ALL PRIVILEGES ON murano.* TO 'murano'@'%' IDENTIFIED BY 'qwe123qwe';
```

## Criando usuário Murano
```
# . admin-openrc

(admin)# openstack user create --domain default --password-prompt murano
(admin)# openstack role add --project service --user murano admin
(admin)# openstack service create --name murano --description "Application Catalog" application-catalog
```

## Criando Endpoints do Murano
```
(admin)# openstack endpoint create --region RegionOne application-catalog public http://controller:8082
(admin)# openstack endpoint create --region RegionOne application-catalog internal http://controller:8082
(admin)# openstack endpoint create --region RegionOne application-catalog admin http://controller:8082
```
     
## Instalando pacotes do Murano:
```
yum install openstack-murano-api openstack-murano-engine openstack-murano-dashboard -y
```

## Configurando

Altere o conteúdo do arquivo /etc/murano/murano.conf:

```
    [DEFAULT]
    debug = true
    verbose = true

    [database]
    connection = mysql+pymysql://murano:qwe123qwe@controller/murano


    [keystone]
    auth_url = 'http://controller:5000/v2.0'

    [keystone_authtoken]
    www_authenticate_uri = 'http://controller:5000/v2.0'
    auth_host = 'controller'
    auth_port = 5000
    auth_protocol = http
    admin_tenant_name = admin
    admin_user = admin
    admin_password = qwe123qwe

    [murano]
    url = http://controller:8082
```
    
## Populando Base de Dados

```SH
# su -s /bin/sh -c "murano-db-manage upgrade" murano
```


# Finalizando e iniciando OpenStack Murano

```SH
# systemctl enable murano-api.service murano-engine.service
# systemctl start murano-api.service murano-engine.service
```
