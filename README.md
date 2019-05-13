![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/OpenStack%C2%AE_Logo_2016.svg/1200px-OpenStack%C2%AE_Logo_2016.svg.png)

# Informações sobre o Projeto

Esse passo-a-passo tem como objetivo instruir alguém que queira instalar o OpenStack de forma manual. Apesar de desconsiderar fortemente a realização da implementação do OpenStack de forma manual, a utilização dessa para entendimento de todos os componentes, e como as integrações acontecem, é de extrema valia para evolução de um profissional de cloud. Para uma implementação automatizada, recomendo utilizar ferramentas de gerênciamento de configuração como Ansible, Puppet e Chef


 * Sistema Operacional utilizado: **CENTOS 7 64-bit (7.6.1810)**
 * Versão OpenStack utilizada: **Rocky**
 * Utilização de OpenVSwith: **NÃO**
 * Utilização de Storage com CEPH: **NÃO**
 
 ```SH
 (admin)#  <- Linhas iniciadas com esse prefixo devem ser executadas com conta admin do OpenStack autenticada.
 (user)# <- Linhas iniciadas com esse prefixo devem ser executadas com conta de USUARIO do OpenStack autenticada.
```
# TODAS AS MAQUINAS

As instruções que seguem devem ser realizadas em **TODAS** as maquinas antes de começar a instalação dos serviços do OpenStack

```sh
# yum install centos-release-openstack-rocky -y
# yum install python-openstackclient -y
# yum install openstack-selinux -y
# yum install vim chrony -y
```

# Controller Node [aqui](ControllerNode.md)
 - MariaDB
 - Memcached
 - RabbitMQ

# Demais serviços do Core OpenStack

## [Instalando Keystone na Controller](KeystoneControllerNode.md) 
 - Keystone
 - HTTPd
 - HTTPd mod_wsgi
 
## [Instalando Glance no Controller](GlanceControllerNode.md)
 - OpenStack Glance (Registry e API) 
 
## [Instalando Nova no Controller](NovaControllerNode.md)
 - OpenStack Nova API
 - OpenStack Nova Scheduler
 - OpenStack Nova Conductor
 - OpenStack Nova Console
 - OpenStack Nova Placement
 - OpenStack Nova NoVNCProxy
 
## [Instalando Nova na Compute01](NovaComputeNode.md)


## [Instalando Neutron no Controller Node](NeutronControllerNode.md)


## [Configurando Neutron no Compute Node](NeutronComputeNode.md)


## [Instalando OpenStack Dashboard Horizon](HorizonControllerNode.md))

## [Instalando OpenStack Cinder no Controller Node](CinderControllerNode.md)

## [Instalando OpenStack Cinder no Storage Node](CinderStorageNode.md)
