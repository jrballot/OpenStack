
Script HOT para criação de servidor com parametros

```
heat_template_version: 2015-10-15

description: Servidor com Rede Interna + Parametros

parameters:

  image:
    type: string
    description: Imagem usada pela instancia de computacao.
    default: cirros
  flavor:
    type: string
    description: Tipo do flavor utilizado pela instancia.
    default: m1.nano
  key:
    type: string
    description: Nome da chave usada pela instancia de computacao.
    default: chave-analista
  private_network:
    type: string
    description: Rede Interna que a instancia ira utilizar.
    default: analista

resources:

  instancia_analista:
    type: OS::Nova::Server
    properties:
      image: { get_param: image }
      flavor: { get_param: flavor }
      key_name: { get_param: key }
      networks:
        - network: { get_param: private_network }

outputs:
  instancia_ip:
    description: Endereço IP da instancia.
    value: { get_attr: [instancia_analista, first_address]}
```
