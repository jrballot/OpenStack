![heat](http://blogs.vmware.com/openstack/files/2017/12/images.png)
## Instalando Heat
```
CREATE DATABASE heat;
GRANT ALL PRIVILEGES ON heat.* TO 'heat'@'localhost' IDENTIFIED BY 'qwe123qwe';
GRANT ALL PRIVILEGES ON heat.* TO 'heat'@'%' IDENTIFIED BY 'qwe123qwe';
```
## Criando usuário Heatno OpenStack
```
{admin}> openstack user create --domain default --password-prompt heat
```

## Adicionando ao projeto service
```
{admin}> openstack role add --project service --user heat admin
```
## Autenticando com usuário admin
```
source admin-rc 
```

## Criando usuário de Orquestração e CloudFormation
```
{admin}> openstack user create --domain default --password-prompt heat
{admin}> openstack role add --project service --user heat admin
{admin}> openstack service create --name heat   --description "Orchestration" orchestration
{admin}> openstack service create --name heat-cfn   --description "Orchestration"  cloudformation
```
## Adicionando Endpoint
```
{admin}> openstack endpoint create --region RegionOne   orchestration public http://controller:8004/v1/%\(tenant_id\)s
{admin}> openstack endpoint create --region RegionOne   orchestration internal http://controller:8004/v1/%\(tenant_id\)s
{admin}> openstack endpoint create --region RegionOne   orchestration admin http://controller:8004/v1/%\(tenant_id\)s
{admin}> openstack endpoint create --region RegionOne   cloudformation public http://controller:8000/v1
{admin}> openstack endpoint create --region RegionOne   cloudformation internal http://controller:8000/v1
{admin}> openstack endpoint create --region RegionOne   cloudformation admin http://controller:8000/v1
```

## Criando domínio, usuários e adicionando roles para Heat
```
{admin}> openstack domain create --description "Stack projects and users" heat
{admin}> openstack user create --domain heat --password-prompt heat_domain_admin
{admin}> openstack role add --domain heat --user-domain heat --user heat_domain_admin admin
{admin}> openstack role create heat_stack_owner
{admin}> openstack role add --project ballot --user julio heat_stack_owner
{admin}> openstack role create heat_stack_user
```
## Instalando HEAT-API e HEAT-API-CFN e HEAT-ENGINE
```
yum install -y openstack-heat-api openstack-heat-api-cfn   openstack-heat-engine
```
## Configurando HEAT
vim /etc/heat/heat.conf:
```
[database]
connection = mysql+pymysql://heat:qwe123qwe@controller/heat

[DEFAULT]
transport_url = rabbit://openstack:qwe123qwe@controller
heat_metadata_server_url = http://controller:8000
heat_waitcondition_server_url = http://controller:8000/v1/waitcondition
stack_domain_admin = heat_domain_admin
stack_domain_admin_password = qwe123qwe
stack_user_domain_name = heat


[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = heat
password = qwe123qwe

[trustee]
auth_type = password
auth_url = http://controller:35357
username = heat
password = qwe123qwe
user_domain_name = default

[clients_keystone]
auth_uri = http://controller:5000

```
## Populando Database do HEAT:
```
su -s /bin/sh -c "heat-manage db_sync" heat
```

## Iniciando e Garantindo serviços ativos no boot 
```
systemctl enable openstack-heat-api.service   openstack-heat-api-cfn.service openstack-heat-engine.service
systemctl start openstack-heat-api.service   openstack-heat-api-cfn.service openstack-heat-engine.service
systemctl status openstack-heat-api.service   openstack-heat-api-cfn.service openstack-heat-engine.service
```

## Verificando Instalação:
```
openstack orchestration service list
```


