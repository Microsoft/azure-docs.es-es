---
title: 'Tutorial: Configuración de la red de Azure CNI en Azure Kubernetes Service (AKS) con Ansible | Microsoft Docs'
description: Obtenga información sobre cómo usar Ansible para configurar redes kubenet en un clúster de Azure Kubernetes Service (AKS).
keywords: ansible, azure, devops, bash, cloudshell, playbook, aks, container, aks, kubernetes
ms.topic: tutorial
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: 2d43b1ffbb7910b16c81df2ff5b21e67dbcb0193
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65231365"
---
# <a name="tutorial-configure-azure-cni-networking-in-azure-kubernetes-service-aks-using-ansible"></a>Tutorial: Configuración de la red de Azure CNI en Azure Kubernetes Service (AKS) con Ansible

[!INCLUDE [ansible-28-note.md](../../includes/ansible-28-note.md)]

[!INCLUDE [open-source-devops-intro-aks.md](../../includes/open-source-devops-intro-aks.md)]

Con AKS, puede implementar un clúster mediante los siguientes modelos de red:

- [Redes kubenet](/azure/aks/configure-kubenet): los recursos de red normalmente se crean y configuran cuando se implementa el clúster de AKS.
- [Red de Azure CNI](/azure/aks/configure-azure-cni): el clúster de AKS se conecta a los recursos y las configuraciones de la red virtual existente.

Para más información sobre las conexiones de red a las aplicaciones de AKS, vea [Conceptos de redes de aplicaciones en Azure Kubernetes Service (AKS)](/azure/aks/concepts-network).

[!INCLUDE [ansible-tutorial-goals.md](../../includes/ansible-tutorial-goals.md)]

> [!div class="checklist"]
>
> * Creación de un clúster de AKS
> * Configuración de redes de Azure CNI

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [open-source-devops-prereqs-create-service-principal.md](../../includes/open-source-devops-prereqs-create-service-principal.md)]
[!INCLUDE [ansible-prereqs-cloudshell-use-or-vm-creation2.md](../../includes/ansible-prereqs-cloudshell-use-or-vm-creation2.md)] 

## <a name="create-a-virtual-network-and-subnet"></a>Creación de una red virtual y una subred

El código del cuaderno de estrategias de ejemplo de esta sección se utiliza para:

- Creación de una red virtual
- Crear una subred dentro de la red virtual

Guarde el siguiente cuaderno de estrategias como `vnet.yml`:

```yml
- name: Create vnet
  azure_rm_virtualnetwork:
      resource_group: "{{ resource_group }}"
      name: "{{ name }}"
      address_prefixes_cidr:
          - 10.0.0.0/8

- name: Create subnet
  azure_rm_subnet:
      resource_group: "{{ resource_group }}"
      name: "{{ name }}"
      address_prefix_cidr: 10.240.0.0/16
      virtual_network_name: "{{ name }}"
  register: subnet
```

## <a name="create-an-aks-cluster-in-the-virtual-network"></a>Creación de un clúster de AKS en la red virtual

El código del cuaderno de estrategias de ejemplo de esta sección se utiliza para:

- Crear un clúster de AKS en una red virtual

Guarde el siguiente cuaderno de estrategias como `aks.yml`:

```yml
- name: List supported kubernetes version from Azure
  azure_rm_aks_version:
      location: "{{ location }}"
  register: versions

- name: Create AKS cluster within a VNet
  azure_rm_aks:
      resource_group: "{{ resource_group }}"
      name: "{{ name }}"
      dns_prefix: "{{ name }}"
      kubernetes_version: "{{ versions.azure_aks_versions[-1] }}"
      agent_pool_profiles:
        - count: 3
          name: nodepool1
          vm_size: Standard_D2_v2
          vnet_subnet_id: "{{ vnet_subnet_id }}"
      linux_profile:
          admin_username: azureuser
          ssh_key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
      service_principal:
          client_id: "{{ lookup('ini', 'client_id section=default file=~/.azure/credentials') }}"
          client_secret: "{{ lookup('ini', 'secret section=default file=~/.azure/credentials') }}"
      network_profile:
          network_plugin: azure
          docker_bridge_cidr: 172.17.0.1/16
          dns_service_ip: 10.2.0.10
          service_cidr: 10.2.0.0/24
  register: aks
```

Estas son algunas notas claves que se deben tener en cuenta al trabajar con el cuaderno de estrategias de ejemplo:

- Use el módulo `azure_rm_aks_version` para buscar la versión compatible.
- `vnet_subnet_id` es la subred que creó en la sección anterior.
- El cuaderno de estrategias carga `ssh_key` desde `~/.ssh/id_rsa.pub`. Si lo modifica, utilice el formato de línea única, empezando por "ssh-rsa" (sin las comillas).
- Los valores `client_id` y `client_secret` se cargan desde `~/.azure/credentials`, que es el archivo de credenciales predeterminado. Puede establecer estos valores en su entidad de servicio o cargarlos desde las variables de entorno:

    ```yml
    client_id: "{{ lookup('env', 'AZURE_CLIENT_ID') }}"
    client_secret: "{{ lookup('env', 'AZURE_SECRET') }}"
    ```

## <a name="run-the-sample-playbook"></a>Ejecución del cuaderno de estrategias de ejemplo

El código del cuaderno de estrategias de ejemplo de esta sección se usa para probar varias características que se muestran a lo largo de este tutorial.

Guarde el siguiente cuaderno de estrategias como `aks-azure-cni.yml`:

```yml
---
- hosts: localhost
  vars:
      resource_group: aksansibletest
      name: aksansibletest
      location: eastus
  tasks:
     - name: Ensure resource group exists
       azure_rm_resourcegroup:
           name: "{{ resource_group }}"
           location: "{{ location }}"

     - name: Create vnet
       include_tasks: vnet.yml

     - name: Create AKS
       vars:
           vnet_subnet_id: "{{ subnet.state.id }}"
       include_tasks: aks.yml

     - name: Show AKS cluster detail
       debug:
           var: aks
```

Estas son algunas notas claves que se deben tener en cuenta al trabajar con el cuaderno de estrategias de ejemplo:

- Cambie el valor `aksansibletest` por el nombre del grupo de recursos.
- Cambie el valor `aksansibletest` por el nombre de AKS.
- Cambie el valor `eastus` por la ubicación del grupo de recursos.

Para ejecutar el cuaderno de estrategias, use el comando ansible-playbook:

```bash
ansible-playbook aks-azure-cni.yml
```

Tras ejecutar el cuaderno de estrategias, debería ver resultados similares a los siguientes:

```Output
PLAY [localhost] 

TASK [Gathering Facts] 
ok: [localhost]

TASK [Ensure resource group exists] 
changed: [localhost]

TASK [Create vnet] 
included: /home/devops/aks-cni/vnet.yml for localhost

TASK [Create vnet] 
changed: [localhost]

TASK [Create subnet] 
changed: [localhost]

TASK [Create AKS] 
included: /home/devops/aks-cni/aks.yml for localhost

TASK [List supported kubernetes version from Azure] 
 [WARNING]: Azure API profile latest does not define an entry for
ContainerServiceClient

ok: [localhost]

TASK [Create AKS cluster with vnet] 
changed: [localhost]

TASK [Show AKS cluster detail] 
ok: [localhost] => {
    "aks": {
        "aad_profile": {},
        "addon": {},
        "agent_pool_profiles": [
            {
                "count": 3,
                "name": "nodepool1",
                "os_disk_size_gb": 100,
                "os_type": "Linux",
                "storage_profile": "ManagedDisks",
                "vm_size": "Standard_D2_v2",
                "vnet_subnet_id": "/subscriptions/BBBBBBBB-BBBB-BBBB-BBBB-BBBBBBBBBBBB/resourceGroups/aksansibletest/providers/Microsoft.Network/virtualNetworks/aksansibletest/subnets/aksansibletest"
            }
        ],
        "changed": true,
        "dns_prefix": "aksansibletest",
        "enable_rbac": false,
        "failed": false,
        "fqdn": "aksansibletest-0272707d.hcp.eastus.azmk8s.io",
        "id": "/subscriptions/BBBBBBBB-BBBB-BBBB-BBBB-BBBBBBBBBBBB/resourcegroups/aksansibletest/providers/Microsoft.ContainerService/managedClusters/aksansibletest",
        "kube_config": "..."
        },
        "location": "eastus",
        "name": "aksansibletest",
        "network_profile": {
            "dns_service_ip": "10.2.0.10",
            "docker_bridge_cidr": "172.17.0.1/16",
            "network_plugin": "azure",
            "network_policy": null,
            "pod_cidr": null,
            "service_cidr": "10.2.0.0/24"
        },
        "node_resource_group": "MC_aksansibletest_aksansibletest_eastus",
        "provisioning_state": "Succeeded",
        "service_principal_profile": {
            "client_id": "AAAAAAAA-AAAA-AAAA-AAAA-AAAAAAAAAAAA"
        },
        "tags": null,
        "type": "Microsoft.ContainerService/ManagedClusters",
        "warnings": [
            "Azure API profile latest does not define an entry for ContainerServiceClient",
            "Azure API profile latest does not define an entry for ContainerServiceClient"
        ]
    }
}

PLAY RECAP 
localhost                  : ok=9    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, elimine los recursos creados en este artículo. 

El código del cuaderno de estrategias de ejemplo de esta sección se utiliza para:

- Eliminar un grupo de recursos al que se hace referencia en la sección `vars`.

Guarde el siguiente cuaderno de estrategias como `cleanup.yml`:

```yml
---
- hosts: localhost
  vars:
      resource_group: {{ resource_group_name }}
  tasks:
      - name: Clean up resource group
        azure_rm_resourcegroup:
            name: "{{ resource_group }}"
            state: absent
            force: yes
```

Estas son algunas notas claves que se deben tener en cuenta al trabajar con el cuaderno de estrategias de ejemplo:

- Reemplace el marcador de posición `{{ resource_group_name }}` por el nombre del grupo de recursos.
- Se eliminarán todos los recursos del grupo de recursos especificado.

Para ejecutar el cuaderno de estrategias, use el comando ansible-playbook:

```bash
ansible-playbook cleanup.yml
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Configuración de Azure Active Directory en AKS con Ansible](./ansible-aks-configure-rbac.md)