# OpenStack
Passos para aula de OpenStack

# TODAS AS MAQUINAS
```sh
# yum install centos-release-openstack-rocky -y
# yum install python-openstackclient -y
# yum install openstack-selinux -y
```

# Controller Node
#### Instalando MariaDB
```sh
# yum install mariadb mariadb-server python2-PyMySQL -y
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

Agora rode o comando mysql_secure_installation:

```sh
# mysql_secure_installation
```

#### Instalando Message Queue
```sh
# yum install rabbitmq-server -y
# systemctl enable rabbitmq-server.service -y
# systemctl start rabbitmq-server.service -y
```

Agora configure com usuário openstack dentro do rabbitmq:

```sh
rabbitmqctl add_user openstack RABBIT_PASS
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```

#### Instalando o Memcached
```sh
yum install memcached python-memcached -y
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
openstack user create --domain default --password-prompt glance
openstack role add --project service --user glance admin
openstack service create --name glance --description "OpenStack Image" image
```
### Criando Endpoints para o Glance
```SH
openstack endpoint create --region RegionOne image public http://controller:9292
openstack endpoint create --region RegionOne image internal http://controller:9292
openstack endpoint create --region RegionOne image admin http://controller:9292
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
# curl -O ht://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img
# openstack image create "Cirros 0.3.5" --file cirros-0.3.5-x86_64-disk.img --disk-format qcow2 --container-format bare --publico
```
**(NÃO RODAR ESSE COMANDO)** Baixando imagem do Ubuntu Trusty 14.04 LTS:
```SH
curl -O https://cloud-images.ubuntu.com/trusty/current/trusty-server-cloudimg-amd64-disk1.img
openstack image create "Ubuntu Trusty 14.04" --file trusty-server-cloudimg-amd64-disk1.img --disk-format qcow2 --container-format bare --public
```

**(NÃO RODAR ESSE COMANDO)** Baixando image do Ubuntu Bionic 18.04 LTS:
```SH
curl -O http://cloud-images.ubuntu.com/minimal/releases/bionic/release/ubuntu-18.04-minimal-cloudimg-amd64.img
openstack image create "Ubuntu Bionic 18.04 LTS" --file ubuntu-18.04-minimal-cloudimg-amd64.img -disk-format qcow2 --container-format bare --public
```

### Garantindo serviços na inicialização

```SH
systemctl enable openstack-glance-api.service openstack-glance-registry.service
systemctl start openstack-glance-api.service openstack-glance-registry.service
```

## Instalando Nova no Controller
#### Criar a base de dados
```SH
CREATE DATABASE nova_api;
CREATE DATABASE nova;
CREATE DATABASE nova_cell0;
CREATE DATABASE placement;
```

#### Criando o permisionamento da base de dados
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

### Criando usuário Nova:
```SH
openstack user create --domain default --password-prompt nova
openstack role add --project service --user nova admin
openstack service create --name nova --description "OpenStack Compute" compute
```

### Criando Endpoints do serviço Compute:
```SH
openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1
openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1
openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1
```
### Criando usuário Placement:
```SH
openstack user create --domain default --password-prompt placement
openstack role add --project service --user placement admin
openstack service create --name placement --description "Placement API" placement
```
### Criando Endpoints para Placement:
```SH
openstack endpoint create --region RegionOne placement public http://controller:8778
openstack endpoint create --region RegionOne placement internal http://controller:8778
openstack endpoint create --region RegionOne placement admin http://controller:8778
```

### Instalando Nova
```SH
yum install openstack-nova-api openstack-nova-conductor openstack-nova-console openstack-nova-novncproxy  openstack-nova-scheduler openstack-nova-placement-api
```

### Garantir serviços subindo no boot:
```SH
systemctl enable openstack-nova-api openstack-nova-conductor openstack-nova-console openstack-nova-novncproxy  openstack-nova-scheduler openstack-nova-placement-api
```

#### Configurando Nova na Controller
/etc/nova/nova.conf:
```
[DEFAULT]
enabled_apis=osapi_compute,metadata
transport_url=rabbit://openstack:qwe123qwe@controller
my_ip=10.0.10.11
use_neutron=true
firewall_driver=nova.virt.firewall.NoopFirewallDriver

[api]
auth_strategy=keystone

[api_database]
connection=mysql+pymysql://nova:qwe123qwe@controller/nova_api

[database]
connection=mysql+pymysql://nova:qwe123qwe@controller/nova

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
service_metadata_proxy = true
metadata_proxy_shared_secret = qwe123qwe

[placement]
region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:5000/v3
username = placement
password = qwe123qwe

[placement_database]
connection=mysql+pymysql://placement:qwe123qwe@controller/placement

[scheduler]
discover_hosts_in_cells_interval=300

[vnc]
enabled = true
server_listen = $my_ip
server_proxyclient_address = $my_ip
```

### Liberar acesso na API do Placement(bug):
Edite o arquivo /etc/httpd/conf.d/00-nova-placement-api.conf:
```
<Directory /usr/bin>
   <IfVersion >= 2.4>
      Require all granted
   </IfVersion>
   <IfVersion < 2.4>
      Order allow,deny
      Allow from all
   </IfVersion>
</Directory>
```

***Não esquecer de reiniciar o HTTPd:***
```SH
systemctl restart httpd
```

# Compute Node
## Instalando Nova na Compute01

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


