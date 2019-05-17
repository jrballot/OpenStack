```
heat_template_version: 2015-10-15

description: Servidor com Nova Rede Interna + Rede Externa + Security Group

resources:

  dexter_security_group:
    type: OS::Neutron::SecurityGroup
    properties:
      name: dexter_security_group
      rules:
        - protocol: icmp
        - protocol: tcp
          port_range_min: 22
          port_range_max: 22

  infra:
    type: OS::Neutron::Net
    properties:
      name: infra

  sub_infra:
    type: OS::Neutron::Subnet
    properties:
      network_id: { get_resource: infra }
      cidr: 10.10.10.0/24
      dns_nameservers:
        - 8.8.8.8

  router:
    type: OS::Neutron::Router
    properties:
      external_gateway_info:
        network: provider

  router-interface:
    type: OS::Neutron::RouterInterface
    properties:
      router_id: { get_resource: router }
      subnet: { get_resource: sub_infra }

  instancia_port:
    type: OS::Neutron::Port
    properties:
      network: { get_resource: infra }
      security_groups:
        - { get_resource: dexter_security_group }
      fixed_ips:
        - subnet_id: { get_resource: sub_infra }

  instancia_analista:
    type: OS::Nova::Server
    properties:
      image: Cirros 0.3.5
      flavor: m1.nano
      key_name: chave-analista
      admin_user: cirros
      networks:
        - port: { get_resource: instancia_port }

  instancia_floating_ip:
    type: OS::Neutron::FloatingIP
    properties:
      floating_network: provider

  instancia_floating_ip_assoc:
    type: OS::Neutron::FloatingIPAssociation
    properties:
      floatingip_id: { get_resource: instancia_floating_ip }
      port_id: { get_resource: instancia_port }

outputs:
  instancia_ip:
    description: Endereco IP da instancia.
    value: { get_attr: [instancia_analista, first_address]}
  instancia_public_ip:
    description: Endereco publico da instancia.
    value: { get_attr: [ instancia_floating_ip, floating_ip_address ] }
```
