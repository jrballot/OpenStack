
```
heat_template_version: 2015-10-15

description: Servidor com Rede Interna + Rede Externa + Security Group + User Data

resources:

  web_server_security_group:
    type: OS::Neutron::SecurityGroup
    properties:
      name: dexter_web_server_security_group
      rules:
        - protocol: icmp
        - protocol: tcp
          port_range_min: 22
          port_range_max: 22
        - protocol: tcp
          port_range_min: 80
          port_range_max: 80

  instancia_port:
    type: OS::Neutron::Port
    properties:
      network: analista
      security_groups:
        - { get_resource: web_server_security_group }

  instancia_analista:
    type: OS::Nova::Server
    properties:
      image: ubuntu-14.04
      flavor: m2.nano
      key_name: chave-analista
      admin_user: ubuntu
      networks:
        - port: { get_resource: instancia_port }
      user_data: |
        #!/bin/bash
        sudo apt-get update
        sudo apt-get install apache2 -y
        sudo echo "<center><h1>WEBSERVER CLOUD" > /var/www/html/index.html
        sudo echo "<p>" >> /var/www/html/index.html
        sudo echo "Infraestrutura de Nuvens com OpenStack" >> /var/www/html/index.html
      user_data_format: RAW

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
