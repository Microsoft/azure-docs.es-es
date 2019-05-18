---
title: 'Tutorial: Administración del tráfico web con Azure Application Gateway mediante Ansible | Microsoft Docs'
description: Aprenda a usar Ansible para crear y configurar una instancia de Azure Application Gateway para administrar el tráfico web
keywords: ansible, azure, devops, bash, cuaderno de estrategias, azure application gateway, equilibrador de carga, tráfico web
ms.topic: tutorial
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: 9f8ed3e1da72db3e1b13d5d2aef1cce8fc3922a2
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65231261"
---
# <a name="tutorial-manage-web-traffic-with-azure-application-gateway-using-ansible"></a>Tutorial: Administración del tráfico web con Azure Application Gateway mediante Ansible

[!INCLUDE [ansible-27-note.md](../../includes/ansible-27-note.md)]

[Azure Application Gateway](/azure/application-gateway/overview) es un equilibrador de carga de tráfico web que permite administrar el tráfico a las aplicaciones web. En función del puerto y la dirección IP de origen, los equilibradores de carga tradicionales enrutan el tráfico a un puerto y una dirección IP de destino. Application Gateway proporciona un mayor nivel de control donde se puede enrutar el tráfico según la dirección URL. Por ejemplo, podría definir que, si `images` es la ruta de acceso a la dirección URL, el tráfico se enrute a un conjunto de servidores específico (conocido como grupo) configurado para las imágenes.

[!INCLUDE [ansible-tutorial-goals.md](../../includes/ansible-tutorial-goals.md)]

> [!div class="checklist"]
>
> * Configurar una red
> * Creación de dos instancias de contenedor de Azure con imágenes HTTPD
> * Creación de una puerta de enlace de aplicación que funciona con las instancias de contenedor de Azure del clúster de servidores

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [ansible-prereqs-cloudshell-use-or-vm-creation2.md](../../includes/ansible-prereqs-cloudshell-use-or-vm-creation2.md)]

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

El código del cuaderno de estrategias de esta sección crea un grupo de recursos de Azure. Un grupo de recursos es un contenedor lógico en el que se configuran los recursos de Azure.  

Guarde el siguiente cuaderno de estrategias como `rg.yml`:

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
    location: eastus 
  tasks:
    - name: Create a resource group
      azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        location: "{{ location }}"
```

Antes de ejecutar el cuaderno de estrategias, vea las notas siguientes:

- El nombre del grupo de recursos es `myResourceGroup`. Este valor se utiliza a lo largo del tutorial.
- El grupo de recursos se crea en la ubicación `eastus`.

Ejecute el comando de estrategias con el comando `ansible-playbook`:

```bash
ansible-playbook rg.yml
```

## <a name="create-network-resources"></a>Crear recursos de red

El código del cuaderno de estrategias de esta sección crea una red virtual para permitir que la puerta de enlace de aplicación se comunique con otros recursos.

Guarde el siguiente cuaderno de estrategias como `vnet_create.yml`:

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
    location: eastus 
    vnet_name: myVNet 
    subnet_name: myAGSubnet 
    publicip_name: myAGPublicIPAddress
    publicip_domain: mydomain
  tasks:
    - name: Create a virtual network
      azure_rm_virtualnetwork:
        name: "{{ vnet_name }}"
        resource_group: "{{ resource_group }}"
        address_prefixes_cidr:
            - 10.1.0.0/16
            - 172.100.0.0/16
        dns_servers:
            - 127.0.0.1
            - 127.0.0.2

    - name: Create a subnet
      azure_rm_subnet:
        name: "{{ subnet_name }}"
        virtual_network_name: "{{ vnet_name }}"
        resource_group: "{{ resource_group }}"
        address_prefix_cidr: 10.1.0.0/24

    - name: Create a public IP address
      azure_rm_publicipaddress:
        resource_group: "{{ resource_group }}" 
        allocation_method: Dynamic
        name: "{{ publicip_name }}"
        domain_name_label: "{{ publicip_domain }}"
```

Antes de ejecutar el cuaderno de estrategias, vea las notas siguientes:

* La sección `vars` contiene los valores que se usan para crear los recursos d red. 
* Deberá cambiar estos valores para su entorno específico.

Ejecute el comando de estrategias con el comando `ansible-playbook`:

```bash
ansible-playbook vnet_create.yml
```

## <a name="create-servers"></a>Creación de servidores

El código del cuaderno de estrategias de esta sección crea dos instancias de contenedor de Azure con imágenes HTTPD para usarlos como servidores web para la puerta de enlace de aplicación.  

Guarde el siguiente cuaderno de estrategias como `aci_create.yml`:

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
    location: eastus 
    aci_1_name: myACI1
    aci_2_name: myACI2
  tasks:
    - name: Create a container with httpd image 
      azure_rm_containerinstance:
        resource_group: "{{ resource_group }}"
        name: "{{ aci_1_name }}"
        os_type: linux
        ip_address: public
        location: "{{ location }}"
        ports:
          - 80
        containers:
          - name: mycontainer
            image: httpd
            memory: 1.5
            ports:
              - 80

    - name: Create another container with httpd image 
      azure_rm_containerinstance:
        resource_group: "{{ resource_group }}"
        name: "{{ aci_2_name }}"
        os_type: linux
        ip_address: public
        location: "{{ location }}"
        ports:
          - 80
        containers:
          - name: mycontainer
            image: httpd
            memory: 1.5
            ports:
              - 80
```

Ejecute el comando de estrategias con el comando `ansible-playbook`:

```bash
ansible-playbook aci_create.yml
```

## <a name="create-the-application-gateway"></a>Creación de la puerta de enlace de aplicaciones

El código del cuaderno de estrategias de esta sección crea una puerta de enlace de aplicación denominada `myAppGateway`.  

Guarde el siguiente cuaderno de estrategias como `appgw_create.yml`:

```yml
- hosts: localhost
  connection: local
  vars:
    resource_group: myResourceGroup
    vnet_name: myVNet
    subnet_name: myAGSubnet
    location: eastus
    publicip_name: myAGPublicIPAddress
    appgw_name: myAppGateway
    aci_1_name: myACI1
    aci_2_name: myACI2
  tasks:
    - name: Get info of Subnet
      azure_rm_resource_facts:
        api_version: '2018-08-01'
        resource_group: "{{ resource_group }}"
        provider: network
        resource_type: virtualnetworks
        resource_name: "{{ vnet_name }}"
        subresource:
          - type: subnets
            name: "{{ subnet_name }}"
      register: subnet

    - name: Get info of backend server 2
      azure_rm_resource_facts:
        api_version: '2018-04-01'
        resource_group: "{{ resource_group }}"
        provider: containerinstance
        resource_type: containergroups
        resource_name: "{{ aci_1_name }}"
      register: aci_1_output
    - name: Get info of backend server 2
      azure_rm_resource_facts:
        api_version: '2018-04-01'
        resource_group: "{{ resource_group }}"
        provider: containerinstance
        resource_type: containergroups
        resource_name: "{{ aci_2_name }}"
      register: aci_2_output

    - name: Create instance of Application Gateway
      azure_rm_appgateway:
        resource_group: "{{ resource_group }}"
        name: "{{ appgw_name }}"
        sku:
          name: standard_small
          tier: standard
          capacity: 2
        gateway_ip_configurations:
          - subnet:
              id: "{{ subnet.response[0].id }}"
            name: appGatewayIP
        frontend_ip_configurations:
          - public_ip_address: "{{ publicip_name }}"
            name: appGatewayFrontendIP
        frontend_ports:
          - port: 80
            name: appGatewayFrontendPort
        backend_address_pools:
          - backend_addresses:
              - ip_address: "{{ aci_1_output.response[0].properties.ipAddress.ip }}"
              - ip_address: "{{ aci_2_output.response[0].properties.ipAddress.ip }}"
            name: appGatewayBackendPool
        backend_http_settings_collection:
          - port: 80
            protocol: http
            cookie_based_affinity: enabled
            name: appGatewayBackendHttpSettings
        http_listeners:
          - frontend_ip_configuration: appGatewayFrontendIP
            frontend_port: appGatewayFrontendPort
            name: appGatewayHttpListener
        request_routing_rules:
          - rule_type: Basic
            backend_address_pool: appGatewayBackendPool
            backend_http_settings: appGatewayBackendHttpSettings
            http_listener: appGatewayHttpListener
            name: rule1
```

Antes de ejecutar el cuaderno de estrategias, vea las notas siguientes:

* `appGatewayIP` se define en el bloque `gateway_ip_configurations`. Se necesita una referencia de subred para la configuración IP de la puerta de enlace.
* `appGatewayBackendPool` se define en el bloque `backend_address_pools`. Una puerta de enlace de aplicaciones debe tener al menos un grupo de direcciones de servidores back-end.
* `appGatewayBackendHttpSettings` se define en el bloque `backend_http_settings_collection`. Especifica que se use el puerto 80 y un protocolo HTTP para la comunicación.
* `appGatewayHttpListener` se define en el bloque `backend_http_settings_collection`. Agente de escucha predeterminado asociado con appGatewayBackendPool.
* `appGatewayFrontendIP` se define en el bloque `frontend_ip_configurations`. Asigna myAGPublicIPAddress a appGatewayHttpListener.
* `rule1` se define en el bloque `request_routing_rules`. La regla de enrutamiento predeterminada asociada a appGatewayHttpListener.

Ejecute el comando de estrategias con el comando `ansible-playbook`:

```bash
ansible-playbook appgw_create.yml
```

La puerta de enlace de aplicaciones puede tardar varios minutos en crearse.

## <a name="test-the-application-gateway"></a>Prueba de la puerta de enlace de aplicaciones

1. En la sección [Creación de un grupo de recursos](#create-a-resource-group), debe especificar una ubicación. Tenga en cuenta su valor.

1. En la sección [Creación de recursos de red](#create-network-resources), debe especificar el dominio. Tenga en cuenta su valor.

1. Para la dirección URL de prueba, reemplace el siguiente patrón por la ubicación y el dominio: `http://<domain>.<location>.cloudapp.azure.com`.

1. Vaya a la dirección URL de prueba.

1. Si ve la página siguiente significa que la puerta de enlace de aplicación funciona según lo previsto.

    ![Prueba correcta de una puerta de enlace de aplicación que funciona](media/ansible-application-gateway-configure/application-gateway.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, elimine los recursos creados en este artículo. 

Guarde el siguiente código como `cleanup.yml`:

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
  tasks:
    - name: Delete a resource group
      azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        state: absent
```

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook cleanup.yml
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Ansible en Azure](/azure/ansible/)