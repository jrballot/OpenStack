# OpenStack
Passos para aula de OpenStack

# TODAS AS MAQUINAS
```sh
# yum install centos-release-openstack-rocky
# yum install python-openstackclient
# yum install openstack-selinux
```

# Controller Node
#### Instalando MariaDB
```sh
# yum install mariadb mariadb-server python2-PyMySQL
```

Edite o arquivo /etc/my.cnf com conteúdo que segue abaixo da entrda [mysqld]:
```
bind-address = 10.0.10.11
default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
```
Posteriomente execute os comandos:

```sh
# systemctl enable mariadb.service
# systemctl start mariadb.service
```

Agora rode o comando mysql_secure_installation

```sh
# mysql_secure_installation
```

#### Instalando Message Queue
```sh
# yum install rabbitmq-server
# systemctl enable rabbitmq-server.service
# systemctl start rabbitmq-server.service
```

Agora configure com usuário openstack

```sh
rabbitmqctl add_user openstack RABBIT_PASS
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```

#### Instalando o Memcached
```sh
yum install memcached python-memcached
```

Dentro do arquivo de configuração /etc/sysconfig/memcached adicione:
```
OPTIONS="-l 127.0.0.1,::1,controller"
```
Após essa etapa garanta o serviço ativo e pronto para subir no boot:

```sh
# systemctl enable memcached.service
# systemctl start memcached.service
```


# Demais serviços do Core OpenStack

## Instalando Keystone na Controller

### Configurando o MariaDB
MariaDB
```
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'KEYSTONE_DBPASS';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'KEYSTONE_DBPASS';
```
Instalando Keystone
```
# yum install openstack-keystone httpd mod_wsgi
```

Agora edite o keystone.conf em /etc/keystone, adicionando os seguintes parametros:
```
[database]
connection = mysql+pymysql://keystone:KEYSTONE_DBPASS@controller/keystone
.
.
.
[token]
provider = fernet
```
### Populando a base de dados do MariaDB
Agora conseguimos popular a database com comando keystone-manage db_sync:

```sh
# su -s /bin/sh -c "keystone-manage db_sync" keystone
```

```SH
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
keystone-manage bootstrap --bootstrap-password qwe123qwe --bootstrap-admin-url http://controller:5000/v3/ --bootstrap-internal-url http://controller:5000/v3/ --bootstrap-public-url http://controller:5000/v3/ --bootstrap-region-id RegionOne
```

### Configurando HTTPd para redirecionar o WSGI
Definir o ServerName para controller no HTTPd
```SH
# ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
# systemctl enable httpd.service
# systemctl start httpd.service
```

Variáveis de ambiente para logar no OpenStack

```SH
$ export OS_USERNAME=admin
$ export OS_PASSWORD=ADMIN_PASS
$ export OS_PROJECT_NAME=admin
$ export OS_USER_DOMAIN_NAME=Default
$ export OS_PROJECT_DOMAIN_NAME=Default
$ export OS_AUTH_URL=http://controller:5000/v3
$ export OS_IDENTITY_API_VERSION=3
```

### Configurando Dominios e Projetos
```sh
# openstack project create --domain default --description "Service Project" service
# openstack project create --domain default --description "<SEU_NOME> Project" <SEU_NOME>
$ openstack user create --domain default --password-prompt <SEU_USUARIO>
$ openstack role create <SUA_ROLE>
$ openstack role add --project <SEU_PROJETO> --user <SEU_USUARIO> <SUA_ROLE>
```

### Validando autenticacao no Keystone

Remova a definição das variáveis OS_PASSWORD e OS_AUTH_URL:
Agora podemos testar a autenticação solicitando um token de acesso

```SH
openstack --os-auth-url http://controller:5000/v3 --os-project-domain-name Default --os-user-domain-name Default --os-project-name admin --os-username admin token issue
$ openstack --os-auth-url http://controller:5000/v3 --os-project-domain-name Default --os-user-domain-name Default --os-project-name myproject --os-username myuser token issue
```

### Instalando Glance no Controller

/etc/glance/glance-api.conf:
```
[dataabase]
connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance

[keystone_authtoken]
www_authenticate_uri  = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = GLANCE_PASS

[paste_deploy]
falvor = keystone

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

Para popular o banco do Glance:
```SH
# su -s /bin/sh -c "glance-manage db_sync" glance
```

Garantir serviço ativo e rodando:
```SH
# systemctl enable openstack-glance-api.service openstack-glance-registry.service
# systemctl start openstack-glance-api.service openstack-glance-registry.service
```

Baixando e Criando Imagens para o OpenStack
```SH
# curl -O ht://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img
# openstack image create "Cirros 0.3.5" --file cirros-0.3.5-x86_64-disk.img --disk-format qcow2 --container-format bare --publico

curl -O https://cloud-images.ubuntu.com/trusty/current/trusty-server-cloudimg-amd64-disk1.img
openstack image create "Ubuntu Trusty 14.04" --file trusty-server-cloudimg-amd64-disk1.img --disk-format qcow2 --container-format bare --public
```

### Instalando Nova no Controller
Criar a base de dados:
```SH
CREATE DATABASE nova_api;
CREATE DATABASE nova;
CREATE DATABASE nova_cell0;
CREATE DATABASE placement;
```

Criando o permisionamento da base de dados:
```SH
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'NOVA_DBPASS';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'NOVA_DBPASS';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost'   IDENTIFIED BY 'NOVA_DBPASS';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'localhost' IDENTIFIED BY 'PLACEMENT_DBPASS';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%' IDENTIFIED BY 'PLACEMENT_DBPASS';
```

O que é Cell0 no OpenStack?????

Criando usuário nova no OpenStack:
```SH
openstack user create --domain default --password-prompt nova
openstack role add --project service --user nova admin
openstack service create --name nova --description "OpenStack Compute" compute
```

Criando Endpoints:
```SH
openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1
openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1
openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1
```

openstack user create --domain default --password-prompt placement
openstack role add --project service --user placement admin
openstack service create --name placement --description "Placement API" placement

openstack endpoint create --region RegionOne placement public http://controller:8778
openstack endpoint create --region RegionOne placement internal http://controller:8778
openstack endpoint create --region RegionOne placement admin http://controller:8778

Instalando Nova
```SH
yum install openstack-nova-api openstack-nova-conductor openstack-nova-console openstack-nova-novncproxy  openstack-nova-scheduler openstack-nova-placement-api
```

#### Configurando Nova
/etc/nova/nova.conf:



#### Instalando Nova na Compute01
yum install openstack-nova-compute

