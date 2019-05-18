---
title: 'Tutorial: Configuración de tablas de rutas de Azure con Ansible | Microsoft Docs'
description: Aprenda a crear, cambiar y eliminar tablas de rutas de Azure con Ansible
keywords: ansible, azure, devops, bash, cuaderno de estrategias, redes, ruta, tabla de rutas
ms.topic: tutorial
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: 846ff510603c0ed0888ec92ece8b86fad0354c19
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65230889"
---
# <a name="tutorial-configure-azure-route-tables-using-ansible"></a>Tutorial: Configuración de tablas de rutas de Azure con Ansible

[!INCLUDE [ansible-27-note.md](../../includes/ansible-28-note.md)]

Azure enruta automáticamente el tráfico entre redes locales, las redes virtuales y las subredes de Azure. Si necesita más control sobre el enrutamiento de su entorno, puede crear una [tabla de rutas](/azure/virtual-network/virtual-networks-udr-overview). 

[!INCLUDE [ansible-tutorial-goals.md](../../includes/ansible-tutorial-goals.md)]

> [!div class="checklist"]
>
> Creación de una tabla de rutas Creación de una red virtual y una subred Asociación de una tabla de rutas con una subred Desasociación de una tabla de rutas de una subred Creación y eliminación de rutas Consulta de una tabla de rutas Eliminación de una tabla de rutas

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [ansible-prereqs-cloudshell-use-or-vm-creation2.md](../../includes/ansible-prereqs-cloudshell-use-or-vm-creation2.md)]

## <a name="create-a-route-table"></a>Creación de una tabla de rutas

El código del cuaderno de estrategias de esta sección crea una tabla de rutas. Para más información sobre los límites de la tabla de rutas, consulte [límites de Azure](/azure/azure-subscription-service-limits#azure-resource-manager-virtual-networking-limits). 

Guarde el siguiente cuaderno de estrategias como `route_table_create.yml`:

```yml
- hosts: localhost
  vars:
    route_table_name: myRouteTable
    resource_group: myResourceGroup
  tasks:
    - name: Create a route table
      azure_rm_routetable:
        name: "{{ route_table_name }}"
        resource_group: "{{ resource_group }}"
```

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook route_table_create.yml
```

## <a name="associate-a-route-table-to-a-subnet"></a>Asociación de una tabla de rutas a una subred

El código del cuaderno de estrategias de esta sección:

* Crea una red virtual
* Crea una subred dentro de la red virtual
* Asocia una tabla de rutas a la subred

Las tablas de rutas no se asocian a las redes virtuales. En su lugar, las tablas de rutas se asocian a la subred de una red virtual.

La red virtual y la tabla de rutas deben coexistir en la misma suscripción y ubicación de Azure.

Las subredes y las tablas de rutas tienen una relación de uno a varios. Se puede definir una subred sin ninguna tabla de rutas asociada o con una tabla de rutas. Las tablas de rutas se pueden asociar con una subred, con muchas o con ninguna. 

El tráfico de la subred se enruta según:

- las rutas definidas dentro de las tablas de rutas
- las [rutas predeterminadas](/azure/virtual-network/virtual-networks-udr-overview#default)
- las rutas propagadas desde una red local

La red virtual debe estar conectada a una puerta de enlace de red virtual de Azure. La puerta de enlace puede ser ExpressRoute, o VPN si se usa BGP con una puerta de enlace de VPN.

Guarde el siguiente cuaderno de estrategias como `route_table_associate.yml`:

```yml
- hosts: localhost
  vars:
    subnet_name: mySubnet
    virtual_network_name: myVirtualNetwork 
    route_table_name: myRouteTable
    resource_group: myResourceGroup
  tasks:
    - name: Create virtual network
      azure_rm_virtualnetwork:
        name: "{{ virtual_network_name }}"
        resource_group: "{{ resource_group }}"
        address_prefixes_cidr:
        - 10.1.0.0/16
        - 172.100.0.0/16
        dns_servers:
        - 127.0.0.1
        - 127.0.0.3
    - name: Create a subnet with route table
      azure_rm_subnet:
        name: "{{ subnet_name }}"
        virtual_network_name: "{{ virtual_network_name }}"
        resource_group: "{{ resource_group }}"
        address_prefix_cidr: "10.1.0.0/24"
        route_table: "{ route_table_name }"
```

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook route_table_associate.yml
```

## <a name="dissociate-a-route-table-from-a-subnet"></a>Desasociación de una tabla de rutas de una subred

El código del cuaderno de estrategias de esta sección permite desasociar una tabla de rutas de una subred.

Al desasociar una tabla de rutas de una subred, establezca la `route_table` de la subred en `None`. 

Guarde el siguiente cuaderno de estrategias como `route_table_dissociate.yml`:

```yml
- hosts: localhost
  vars:
    subnet_name: mySubnet
    virtual_network_name: myVirtualNetwork 
    resource_group: myResourceGroup
  tasks:
    - name: Dissociate a route table
      azure_rm_subnet:
        name: "{{ subnet_name }}"
        virtual_network_name: "{{ virtual_network_name }}"
        resource_group: "{{ resource_group }}"
        address_prefix_cidr: "10.1.0.0/24"
```

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook route_table_dissociate.yml
```

## <a name="create-a-route"></a>Creación de una ruta

El código del cuaderno de estrategias de esta sección crea una ruta en la tabla de rutas. 

Guarde el siguiente cuaderno de estrategias como `route_create.yml`:

```yml
- hosts: localhost
  vars:
    route_name: myRoute
    route_table_name: myRouteTable
    resource_group: myResourceGroup
  tasks:
    - name: Create route
      azure_rm_route:
        name: "{{ route_name }}"
        resource_group: "{{ resource_group }}"
        next_hop_type: virtual_network_gateway
        address_prefix: "10.1.0.0/16"
        route_table_name: "{{ route_table_name }}"
```

Antes de ejecutar el cuaderno de estrategias, consulte las notas siguientes:

* `virtual_network_gateway` se define como `next_hop_type`. Para más información sobre cómo Azure selecciona las rutas, consulte [Introducción al enrutamiento](/azure/virtual-network/virtual-networks-udr-overview).
* `address_prefix` se define como `10.1.0.0/16`. No se puede duplicar el prefijo dentro de la tabla de rutas.

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook route_create.yml
```

## <a name="delete-a-route"></a>Eliminación de una ruta

El código del cuaderno de estrategias de esta sección permite eliminar una ruta de una tabla de rutas.

Guarde el siguiente cuaderno de estrategias como `route_delete.yml`:

```yml
- hosts: localhost
  vars:
    route_name: myRoute
    route_table_name: myRouteTable
    resource_group: myResourceGroup
  tasks:
    - name: Remove route
      azure_rm_route:
        name: "{{ route_name }}"
        resource_group: "{{ resource_group }}"
        route_table_name: "{{ route_table_name }}"
        state: absent
```

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook route_delete.yml
```

## <a name="get-route-table-information"></a>Obtención de información de la tabla de rutas

El código del cuaderno de estrategias de esta sección usa el módulo de Ansible `azure_rm_routetable_facts` para recuperar la información de la tabla de rutas.

Guarde el siguiente cuaderno de estrategias como `route_table_facts.yml`:

```yml
- hosts: localhost
  vars:
    route_table_name: myRouteTable
    resource_group: myResourceGroup
  tasks: 
    - name: Get route table information
      azure_rm_routetable_facts:
         resource_group: "{{ resource_group }}"
         name: "{{ route_table_name }}"
      register: query
    
    - debug:
         var: query.route_tables[0]
```

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook route_table_facts.yml
```

## <a name="delete-a-route-table"></a>Eliminación de una tabla de rutas

El código del cuaderno de estrategias de esta sección elimina una tabla de rutas.

Cuando se elimina una tabla de rutas, se eliminan también todas sus rutas.

Una tabla de rutas no se puede eliminar si está asociada con una subred. [Desasocie la tabla de rutas de todas las subredes](#dissociate-a-route-table-from-a-subnet) antes de intentar eliminarla. 

Guarde el siguiente cuaderno de estrategias como `route_table_delete.yml`:

```yml
- hosts: localhost
  vars:
    route_table_name: myRouteTable
    resource_group: myResourceGroup
  tasks:
    - name: Create a route table
      azure_rm_routetable:
        name: "{{ route_table_name }}"
        resource_group: "{{ resource_group }}"
        state: absent
```

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook route_table_delete.yml
```

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"] 
> [Ansible en Azure](/azure/ansible/)