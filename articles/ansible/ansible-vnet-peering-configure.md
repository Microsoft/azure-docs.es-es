---
title: 'Tutorial: Configuración del emparejamiento de redes virtuales de Azure con Ansible | Microsoft Docs'
description: Obtenga información sobre cómo usar Ansible para conectar redes virtuales con el emparejamiento de redes virtuales.
keywords: ansible, azure, devops, bash, playbook, networking, peering
ms.topic: tutorial
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: f51e7c857a22a362a3d295fbe087c54b25f85780
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65230764"
---
# <a name="tutorial-configure-azure-virtual-network-peering-using-ansible"></a>Tutorial: Configuración del emparejamiento de redes virtuales de Azure con Ansible

[!INCLUDE [ansible-28-note.md](../../includes/ansible-28-note.md)]

El [emparejamiento de redes virtuales](/azure/virtual-network/virtual-network-peering-overview) permite conectar sin problemas dos redes virtuales de Azure. Una vez emparejadas, las dos redes virtuales aparecen como una sola a efectos de conectividad. 

El tráfico entre máquinas virtuales de la misma red virtual se enruta mediante direcciones IP privadas. De forma similar, el tráfico entre máquinas virtuales de una red virtual emparejada se enruta mediante la infraestructura de red troncal de Microsoft. Como resultado, las máquinas virtuales de diferentes redes virtuales pueden comunicarse entre sí.

[!INCLUDE [ansible-tutorial-goals.md](../../includes/ansible-tutorial-goals.md)]

> [!div class="checklist"]
>
> * Crear dos redes virtuales
> * Emparejar las dos redes virtuales.
> * Eliminación del emparejamiento entre las dos redes

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [ansible-prereqs-cloudshell-use-or-vm-creation2.md](../../includes/ansible-prereqs-cloudshell-use-or-vm-creation2.md)]

## <a name="create-two-resource-groups"></a>Crear dos grupos de recursos.

Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

El código del cuaderno de estrategias de ejemplo de esta sección se utiliza para lo siguiente:

- Crear dos grupos de recursos. 

```yml
  - name: Create a resource group
    azure_rm_resourcegroup:
      name: "{{ resource_group }}"
      location: "{{ location }}"
  - name: Create secondary resource group
    azure_rm_resourcegroup:
      name: "{{ resource_group_secondary }}"
      location: "{{ location }}"
```

## <a name="create-the-first-virtual-network"></a>Crear la primera red virtual.

El código del cuaderno de estrategias de ejemplo de esta sección se utiliza para lo siguiente:

- Creación de una red virtual
- Crear una subred dentro de la red virtual.

```yml
  - name: Create first virtual network
    azure_rm_virtualnetwork:
      resource_group: "{{ resource_group }}"
      name: "{{ vnet_name1 }}"
      address_prefixes: "10.0.0.0/16"
  - name: Add subnet
    azure_rm_subnet:
      resource_group: "{{ resource_group }}"
      name: "{{ vnet_name1 }}"
      address_prefix: "10.0.0.0/24"
      virtual_network: "{{ vnet_name1 }}"
```

## <a name="create-the-second-virtual-network"></a>Crear la segunda red virtual.

El código del cuaderno de estrategias de ejemplo de esta sección se utiliza para lo siguiente:

- Creación de una red virtual
- Crear una subred dentro de la red virtual.

```yml
  - name: Ceate second virtual network
    azure_rm_virtualnetwork:
      resource_group: "{{ resource_group_secondary }}"
      name: "{{ vnet_name2 }}"
      address_prefixes: "10.1.0.0/16"
  - name: Add subnet
    azure_rm_subnet:
      resource_group: "{{ resource_group }}"
      name: "{{ vnet_name2 }}"
      address_prefix: "10.1.0.0/24"
      virtual_network: "{{ vnet_name2 }}"
```

## <a name="peer-the-two-virtual-networks"></a>Emparejar las dos redes virtuales.

El código del cuaderno de estrategias de ejemplo de esta sección se utiliza para lo siguiente:

- Inicializar el emparejamiento de las redes virtuales.
- Emparejar las dos redes virtuales creadas previamente.

```yml
  - name: Initial vnet peering
    azure_rm_virtualnetworkpeering:
      resource_group: "{{ resource_group }}"
      name: "{{ peering_name }}"
      virtual_network: "{{ vnet_name1 }}"
      remote_virtual_network:
        resource_group: "{{ resource_group_secondary }}"
        name: "{{ vnet_name2 }}"
      allow_virtual_network_access: true
      allow_forwarded_traffic: true

  - name: Connect vnet peering
    azure_rm_virtualnetworkpeering:
      resource_group: "{{ resource_group_secondary }}"
      name: "{{ peering_name }}"
      virtual_network: "{{ vnet_name2 }}"
      remote_virtual_network:
        resource_group: "{{ resource_group }}"
        name: "{{ vnet_name1 }}"
      allow_virtual_network_access: true
      allow_forwarded_traffic: true
```

## <a name="delete-the-virtual-network-peering"></a>Eliminar el emparejamiento de las redes virtuales.

El código del cuaderno de estrategias de ejemplo de esta sección se utiliza para lo siguiente:

- Eliminar el emparejamiento entre las dos redes virtuales creadas previamente.

```yml
  - name: Delete vnet peering
    azure_rm_virtualnetworkpeering:
      resource_group: "{{ resource_group }}"
      name: "{{ peering_name }}"
      virtual_network: "{{ vnet_name1 }}"
      state: absent
```

## <a name="get-the-sample-playbook"></a>Obtención del cuaderno de estrategias de ejemplo

Hay dos formas de obtener el cuaderno de estrategias de ejemplo completo:

- [Descargar el cuaderno de estrategias](https://github.com/Azure-Samples/ansible-playbooks/blob/master/vnet_peering.yml) y guardarlo en `vnet_peering.yml`.
- Crear un archivo llamado `vnet_peering.yml` y copiar en él el siguiente contenido:

```yml
- hosts: localhost
  tasks:
    - name: Prepare random postfix
      set_fact:
        rpfx: "{{ 1000 | random }}"
      run_once: yes

- name: Connect virtual networks with virtual network peering
  hosts: localhost
  connection: local
  vars:
    resource_group: "{{ resource_group_name }}"
    resource_group_secondary: "{{ resource_group_name }}2"
    vnet_name1: "myVnet{{ rpfx }}"
    vnet_name2: "myVnet{{ rpfx }}2"
    peering_name: peer1
    location: eastus2
  tasks:
  - name: Create a resource group
    azure_rm_resourcegroup:
      name: "{{ resource_group }}"
      location: "{{ location }}"
  - name: Create secondary resource group
    azure_rm_resourcegroup:
      name: "{{ resource_group_secondary }}"
      location: "{{ location }}"
  - name: Create first virtual network
    azure_rm_virtualnetwork:
      resource_group: "{{ resource_group }}"
      name: "{{ vnet_name1 }}"
      address_prefixes: "10.0.0.0/16"
  - name: Add subnet
    azure_rm_subnet:
      resource_group: "{{ resource_group }}"
      name: "{{ vnet_name1 }}"
      address_prefix: "10.0.0.0/24"
      virtual_network: "{{ vnet_name1 }}"
  - name: Ceate second virtual network
    azure_rm_virtualnetwork:
      resource_group: "{{ resource_group_secondary }}"
      name: "{{ vnet_name2 }}"
      address_prefixes: "10.1.0.0/16"
  - name: Add subnet
    azure_rm_subnet:
      resource_group: "{{ resource_group }}"
      name: "{{ vnet_name2 }}"
      address_prefix: "10.1.0.0/24"
      virtual_network: "{{ vnet_name2 }}"
  - name: Initial vnet peering
    azure_rm_virtualnetworkpeering:
      resource_group: "{{ resource_group }}"
      name: "{{ peering_name }}"
      virtual_network: "{{ vnet_name1 }}"
      remote_virtual_network:
        resource_group: "{{ resource_group_secondary }}"
        name: "{{ vnet_name2 }}"
      allow_virtual_network_access: true
      allow_forwarded_traffic: true

  - name: Connect vnet peering
    azure_rm_virtualnetworkpeering:
      resource_group: "{{ resource_group_secondary }}"
      name: "{{ peering_name }}"
      virtual_network: "{{ vnet_name2 }}"
      remote_virtual_network:
        resource_group: "{{ resource_group }}"
        name: "{{ vnet_name1 }}"
      allow_virtual_network_access: true
      allow_forwarded_traffic: true

  - name: Delete vnet peering
    azure_rm_virtualnetworkpeering:
      resource_group: "{{ resource_group }}"
      name: "{{ peering_name }}"
      virtual_network: "{{ vnet_name1 }}"
      state: absent
```

## <a name="run-the-sample-playbook"></a>Ejecución del cuaderno de estrategias de ejemplo

El código del cuaderno de estrategias de ejemplo de esta sección se usa para probar varias características que se muestran a lo largo de este tutorial.

Estas son algunas notas importantes que hay que tener en cuenta al trabajar con el cuaderno de estrategias de ejemplo:

- En la sección `vars`, reemplace el marcador de posición `{{ resource_group_name }}` por el nombre del grupo de recursos.

Para ejecutar el cuaderno de estrategias, use el comando ansible-playbook:

```bash
ansible-playbook vnet_peering.yml
```

Tras ejecutar el cuaderno de estrategias, debería ver resultados similares a los siguientes:

```Output
PLAY [localhost] 

TASK [Gathering Facts] 
ok: [localhost]

TASK [Prepare random postfix] 
ok: [localhost]

PLAY [Connect virtual networks with virtual network peering] 

TASK [Gathering Facts] 
ok: [localhost]

TASK [Create a resource group] 
changed: [localhost]

TASK [Create secondary resource group] 
changed: [localhost]

TASK [Create first virtual network] 
changed: [localhost]

TASK [Add subnet] 
changed: [localhost]

TASK [Ceate second virtual network] 
changed: [localhost]

TASK [Add subnet] 
changed: [localhost]

TASK [Initial vnet peering] 
changed: [localhost]

TASK [Connect vnet peering] 
changed: [localhost]

TASK [Delete vnet peering] 
changed: [localhost]

PLAY RECAP 
localhost                  : ok=12   changed=9    unreachable=0    failed=0    skipped=0   rescued=0    ignored=0
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, elimine los recursos creados en este artículo. 

El código del cuaderno de estrategias de ejemplo de esta sección se utiliza para lo siguiente:

- Eliminar los dos grupos de recursos creados anteriormente.

Guarde el siguiente cuaderno de estrategias como `cleanup.yml`:

```bash
- hosts: localhost
  vars:
    resource_group: "{{ resource_group_name-1 }}"
    resource_group_secondary: "{{ resource_group_name-2 }}"
  tasks:
    - name: Delete a resource group
      azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        force_delete_nonempty: yes
        state: absent

    - name: Delete a resource group
      azure_rm_resourcegroup:
        name: "{{ resource_group_secondary }}"
        force_delete_nonempty: yes
        state: absent
```

Estas son algunas notas importantes que hay que tener en cuenta al trabajar con el cuaderno de estrategias de ejemplo:

- Reemplace el marcador de posición `{{ resource_group_name-1 }}` por el nombre del primer grupo de recursos que se creó.
- Reemplace el marcador de posición `{{ resource_group_name-2 }}` por el nombre del segundo grupo de recursos que se creó.
- Se eliminarán todos los recursos de los dos grupos de recursos especificados.

Para ejecutar el cuaderno de estrategias, use el comando ansible-playbook:

```bash
ansible-playbook cleanup.yml
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"] 
> [Ansible en Azure](/azure/ansible/)