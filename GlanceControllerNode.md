## Instalando Glance no Controller

### Instalando o Glance:
```SH
# yum install -y openstack-glance
```

### Criando Base de dados e Privilégios para Glance
```SH
# mysql -u root -p
MariaDB [(none)]> CREATE DATABASE glance;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'qwe123qwe';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'qwe123qwe';
```

### Criando usuário e serviço para o Glance
```SH
source admin-rc
(admin)# openstack user create --domain default --password qwe123qwe glance
(admin)# openstack role add --project service --user glance admin
(admin)# openstack service create --name glance --description "OpenStack Image" image
```
### Criando Endpoints para o Glance
```SH
(admin)# openstack endpoint create --region RegionOne image public http://controller:9292
(admin)# openstack endpoint create --region RegionOne image internal http://controller:9292
(admin)# openstack endpoint create --region RegionOne image admin http://controller:9292
```
### Arquivos de configuração do Glance:

/etc/glance/glance-api.conf:
```
[database]
connection = mysql+pymysql://glance:qwe123qwe@controller/glance

[keystone_authtoken]
www_authenticate_uri  = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = qwe123qwe

[paste_deploy]
flavor = keystone

[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
```

/etc/glance/glance-registry.conf
```
[database]
connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance

[keystone_authtoken]
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = GLANCE_PASS

[paste_deploy]
flavor = keystone
```

### Populando o banco do Glance:
```SH
# su -s /bin/sh -c "glance-manage db_sync" glance
```

### Garantir serviço ativo e rodando:
```SH
# systemctl enable openstack-glance-api.service openstack-glance-registry.service
# systemctl start openstack-glance-api.service openstack-glance-registry.service
```

### Baixando e Criando Imagens para o OpenStack
Baixando imagem do CirrOS:
```SH
(admin)# curl -O http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img
(admin)# openstack image create "Cirros 0.3.5" --file cirros-0.3.5-x86_64-disk.img --disk-format qcow2 --container-format bare --public
```
**(NÃO RODAR ESSE COMANDO)** Baixando imagem do Ubuntu Trusty 14.04 LTS:
```SH
(admin)# curl -O https://cloud-images.ubuntu.com/trusty/current/trusty-server-cloudimg-amd64-disk1.img
(admin)# openstack image create "Ubuntu Trusty 14.04" --file trusty-server-cloudimg-amd64-disk1.img --disk-format qcow2 --container-format bare --public
```

**(NÃO RODAR ESSE COMANDO)** Baixando image do Ubuntu Bionic 18.04 LTS:
```SH
(admin)# curl -O http://cloud-images.ubuntu.com/minimal/releases/bionic/release/ubuntu-18.04-minimal-cloudimg-amd64.img
(admin)# openstack image create "Ubuntu Bionic 18.04 LTS" --file ubuntu-18.04-minimal-cloudimg-amd64.img -disk-format qcow2 --container-format bare --public
```

### Garantindo serviços na inicialização

```SH
(admin)# systemctl enable openstack-glance-api.service openstack-glance-registry.service
(admin)# systemctl start openstack-glance-api.service openstack-glance-registry.service
```
