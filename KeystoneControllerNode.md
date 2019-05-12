![](https://vexxhost.com/wp-content/uploads/2017/05/OpenStack_Project_Keystone_vertical-1-300x250.png)


## Instalando Keystone na Controller

### Configurando o MariaDB
MariaDB
```
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'qwe123qwe';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'qwe123qwe';
```
Instalando Keystone
```
# yum install openstack-keystone httpd mod_wsgi -y
```

Agora edite o keystone.conf em /etc/keystone, adicionando os seguintes parametros:
```
[database]
connection = mysql+pymysql://keystone:qwe123qwe@controller/keystone
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
# keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
# keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
# keystone-manage bootstrap --bootstrap-password qwe123qwe --bootstrap-admin-url http://controller:5000/v3/ --bootstrap-internal-url http://controller:5000/v3/ --bootstrap-public-url http://controller:5000/v3/ --bootstrap-region-id RegionOne
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
export OS_USERNAME=admin                                                                                        
export OS_PASSWORD=qwe123qwe                                                                                      
export OS_PROJECT_NAME=admin                                                                                        
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3

export PS1='[\u@\h \W]($OS_USERNAME)\$ '

```

Salve essas configurações no arquivo admin-rc no diretório local, para reutilizarmos para autenticação.



### Configurando Dominios e Projetos
```sh
# openstack project create --domain default --description "Service Project" service
# openstack project create --domain default --description "<SEU_NOME> Project" <SEU_NOME>
# openstack user create --domain default --password-prompt <SEU_USUARIO>
# openstack role create <SUA_ROLE>
# openstack role add --project <SEU_PROJETO> --user <SEU_USUARIO> <SUA_ROLE>
```

### Validando autenticacao no Keystone

Remova a definição das variáveis OS_PASSWORD e OS_AUTH_URL:
Agora podemos testar a autenticação solicitando um token de acesso

```SH
openstack --os-auth-url http://controller:5000/v3 --os-project-domain-name Default --os-user-domain-name Default --os-project-name admin --os-username admin token issue
$ openstack --os-auth-url http://controller:5000/v3 --os-project-domain-name Default --os-user-domain-name Default --os-project-name myproject --os-username myuser token issue
```
