```
heat_template_version: 2015-10-15

description: Servidor com Rede Interna + Rede Externa + Security Group + Volume

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

  instancia_port:
    type: OS::Neutron::Port
    properties:
      network: analista
      security_groups:
        - { get_resource: dexter_security_group }

  instancia_analista:
    type: OS::Nova::Server
    properties:
      image: cirros
      flavor: m1.nano
      key_name: chave-analista
      admin_user: cirros
      networks:
        - port: { get_resource: instancia_port }

  volume_analista:
    type: OS::Cinder::Volume
    properties:
      size: 2
  volume_analista_data_attachment:
    type: OS::Cinder::VolumeAttachment
    properties:
      instance_uuid: { get_resource: instancia_analista }
      volume_id: { get_resource: volume_analista }

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
