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
[rabbit](https://www.loadbalancer.org/public/images/articles/2018/05/logo-rabbitmq.png)

```sh
# yum install rabbitmq-server -y
# systemctl enable rabbitmq-server.service -y
# systemctl start rabbitmq-server.service -y
```

Agora configure com usuário openstack dentro do rabbitmq:

```sh
rabbitmqctl add_user openstack qwe123qwe
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```

#### Instalando o Memcached
[memcahed](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSOfwdgZXlF2-rE7yeu_dDrMluzvy4NdDLRTp46WjtGaHV5OPh-)


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
