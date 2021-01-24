![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/OpenStack%C2%AE_Logo_2016.svg/1200px-OpenStack%C2%AE_Logo_2016.svg.png)


# ATENÇÃO ESSE PROJETO NÃO ESTÁ ATUALIZADO COM A ÚLTIMA VERSÃO DO OPENSTACK, PORTANTO ALGUNS ELEMENTOS DE ARQUITETURA DO OPENSTACK PODEM NÃO FAZER MAIS SENTIDO.


# Informações sobre o Projeto

Esse passo-a-passo tem como objetivo instruir alguém que queira instalar o OpenStack de forma manual. Apesar de desconsiderar fortemente a realização da implementação do OpenStack de forma manual, a utilização dessa para entendimento de todos os componentes, e como as integrações acontecem, é de extrema valia para evolução de um profissional de cloud. Para uma implementação automatizada, recomendo utilizar ferramentas de gerênciamento de configuração como Ansible, Puppet e Chef


 * Sistema Operacional utilizado: **CENTOS 7 64-bit (7.6.1810)**
 * Versão OpenStack utilizada: **Rocky**
 * Utilização de OpenVSwith: **NÃO**
 * Utilização de Storage com CEPH: **NÃO**
 
 ```SH
 (admin)#  <- Linhas iniciadas com esse prefixo devem ser executadas com conta admin do OpenStack autenticada.
 (user)# <- Linhas iniciadas com esse prefixo devem ser executadas com conta do SEU USUÁRIO do OpenStack autenticada.
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
 
 
## Antes de iniciar a instalação do NOVA e NEUTRON

### Desative o Firewalld nas maquinas
```SH
# systemctl stop firewalld
# systemctl disable firewalld
```
 
## [Instalando Nova no Controller](NovaControllerNode.md)
 - OpenStack Nova API
 - OpenStack Nova Scheduler
 - OpenStack Nova Conductor
 - OpenStack Nova Console
 - OpenStack Nova Placement
 - OpenStack Nova NoVNCProxy
 
## [Instalando Nova na Compute01](NovaComputeNode.md)
 - OpenStack Nova Compute

## [Instalando Neutron no Controller Node](NeutronControllerNode.md)
 - OpenStack Neutron
 - OpenStack Neutron ML2
 - OpenStack Neutron LinuxBridge
 - ebtables

## [Configurando Neutron no Compute Node](NeutronComputeNode.md)
 - OpenStack Neutron LinuxBridge
 - ebtables
 - ipset

## [Instalando OpenStack Dashboard Horizon](HorizonControllerNode.md)
 - OpenStack Dashboard

## [Instalando OpenStack Cinder no Controller Node](CinderControllerNode.md)

## [Instalando OpenStack Cinder no Storage Node](CinderStorageNode.md)

## [Instalando OpenStack Swift no Controller Node](SwiftControllerNode.md)

## [Instalando OpenStack Swift no Object Node](SwiftObjectNode.md)

## [Instalando OpenStack Manila no Controller Node](ManilaControllerNode.md)

## [Instalando OpenStack Manila no Storage Node](ManilaStorageNode.md)
