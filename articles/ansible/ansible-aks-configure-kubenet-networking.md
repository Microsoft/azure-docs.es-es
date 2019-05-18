---
title: 'Tutorial: Configuración de redes kubenet en Azure Kubernetes Service (AKS) con Ansible | Microsoft Docs'
description: Obtenga información sobre cómo usar Ansible para configurar redes kubenet en un clúster de Azure Kubernetes Service (AKS).
keywords: ansible, azure, devops, bash, cloudshell, playbook, aks, container, aks, kubernetes
ms.topic: tutorial
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: cd32347f9de87ea6272be922d0359f1cc7f6f758
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65231305"
---
# <a name="tutorial-configure-kubenet-networking-in-azure-kubernetes-service-aks-using-ansible"></a>Tutorial: Configuración de redes kubenet en Azure Kubernetes Service (AKS) con Ansible

[!INCLUDE [ansible-28-note.md](../../includes/ansible-28-note.md)]

[!INCLUDE [open-source-devops-intro-aks.md](../../includes/open-source-devops-intro-aks.md)]

Con AKS, puede implementar un clúster mediante los siguientes modelos de red:

- [Redes kubenet](/azure/aks/configure-kubenet): los recursos de red normalmente se crean y configuran cuando se implementa el clúster de AKS.
- [Red de Azure Container Networking Interface (CNI)](/azure/aks/configure-azure-cni): el clúster de AKS se conecta a los recursos y configuraciones de la red virtual existentes.

Para más información sobre las conexiones de red a las aplicaciones de AKS, vea [Conceptos de redes de aplicaciones en Azure Kubernetes Service (AKS)](/azure/aks/concepts-network).

[!INCLUDE [ansible-tutorial-goals.md](../../includes/ansible-tutorial-goals.md)]

> [!div class="checklist"]
>
> * Creación de un clúster de AKS
> * Configuración de redes kubenet de Azure

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [open-source-devops-prereqs-create-service-principal.md](../../includes/open-source-devops-prereqs-create-service-principal.md)]
[!INCLUDE [ansible-prereqs-cloudshell-use-or-vm-creation2.md](../../includes/ansible-prereqs-cloudshell-use-or-vm-creation2.md)]

## <a name="create-a-virtual-network-and-subnet"></a>Creación de una red virtual y una subred

El código del cuaderno de estrategias de esta sección crea los siguientes recursos de Azure:

- Virtual network
- Subred en la red virtual

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

El código del cuaderno de estrategias de esta sección crea un clúster de AKS dentro de una red virtual. 

Guarde el siguiente cuaderno de estrategias como `aks.yml`:

```yml
- name: List supported kubernetes version from Azure
  azure_rm_aks_version:
      location: "{{ location }}"
  register: versions

- name: Create AKS cluster with vnet
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
          network_plugin: kubenet
          pod_cidr: 192.168.0.0/16
          docker_bridge_cidr: 172.17.0.1/16
          dns_service_ip: 10.0.0.10
          service_cidr: 10.0.0.0/16
  register: aks
```

Estas son algunas notas claves que se deben tener en cuenta al trabajar con el cuaderno de estrategias de ejemplo:

- Use el módulo `azure_rm_aks_version` para buscar la versión compatible.
- `vnet_subnet_id` es la subred que creó en la sección anterior.
- `network_profile` define las propiedades del complemento de red de kubenet.
- `service_cidr` se usa para asignar servicios internos del clúster de AKS a una dirección IP. Este intervalo de direcciones IP debe ser un espacio de direcciones que no se use en ninguna otra parte de la red. 
- La dirección `dns_service_ip` debe ser la dirección ".10" del intervalo de direcciones IP del servicio.
- `pod_cidr` debe ser un espacio de direcciones grande que no se use en ninguna otra parte del entorno de red. El intervalo de direcciones debe ser lo suficientemente grande como para alojar el número de nodos hasta el que pretende escalar verticalmente. No se puede cambiar este intervalo de direcciones una vez implementado el clúster.
- El intervalo de direcciones IP para pods se usa para asignar un espacio de direcciones de /24 a cada nodo del clúster. En el ejemplo siguiente, `pod_cidr` de 192.168.0.0/16 asigna el primer nodo 192.168.0.0/24, el segundo nodo 192.168.1.0/24 y el tercer nodo 192.168.2.0/24.
- A medida que el clúster se escala o actualiza, Azure continúa asignando un intervalo de direcciones IP para pods a cada nuevo nodo.
- El cuaderno de estrategias carga `ssh_key` desde `~/.ssh/id_rsa.pub`. Si lo modifica, utilice el formato de línea única, empezando por "ssh-rsa" (sin las comillas).
- Los valores `client_id` y `client_secret` se cargan desde `~/.azure/credentials`, que es el archivo de credenciales predeterminado. Puede establecer estos valores en su entidad de servicio o cargarlos desde las variables de entorno:

    ```yml
    client_id: "{{ lookup('env', 'AZURE_CLIENT_ID') }}"
    client_secret: "{{ lookup('env', 'AZURE_SECRET') }}"
    ```

## <a name="associate-the-network-resources"></a>Asociación de los recursos de red

Al crear un clúster de AKS, también se crean un grupo de seguridad de red y una tabla de ruta. AKS se encarga de administrar estos recursos, que se actualizan al crear y exponer los servicios. Asocie el grupo de seguridad de red y la tabla de ruta con la subred de red virtual como se indica a continuación. 

Guarde el siguiente cuaderno de estrategias como `associate.yml`.

```yml
- name: Get route table
  azure_rm_routetable_facts:
      resource_group: "{{ node_resource_group }}"
  register: routetable

- name: Get network security group
  azure_rm_securitygroup_facts:
      resource_group: "{{ node_resource_group }}"
  register: nsg

- name: Parse subnet id
  set_fact:
      subnet_name: "{{ vnet_subnet_id | regex_search(subnet_regex, '\\1') }}"
      subnet_rg: "{{ vnet_subnet_id | regex_search(rg_regex, '\\1') }}"
      subnet_vn: "{{ vnet_subnet_id | regex_search(vn_regex, '\\1') }}"
  vars:
      subnet_regex: '/subnets/(.+)'
      rg_regex: '/resourceGroups/(.+?)/'
      vn_regex: '/virtualNetworks/(.+?)/'

- name: Associate network resources with the node subnet
  azure_rm_subnet:
      name: "{{ subnet_name[0] }}"
      resource_group: "{{  subnet_rg[0] }}"
      virtual_network_name: "{{ subnet_vn[0] }}"
      security_group: "{{ nsg.ansible_facts.azure_securitygroups[0].id }}"
      route_table: "{{ routetable.route_tables[0].id }}"
```

Estas son algunas notas claves que se deben tener en cuenta al trabajar con el cuaderno de estrategias de ejemplo:

- `node_resource_group` es el nombre del grupo de recursos en el que se crean los nodos de AKS.
- `vnet_subnet_id` es la subred que creó en la sección anterior.


## <a name="run-the-sample-playbook"></a>Ejecución del cuaderno de estrategias de ejemplo

En esta sección se indica el cuaderno de estrategias de ejemplo completo que llama a las tareas que se crean en este artículo. 

Guarde el siguiente cuaderno de estrategias como `aks-kubenet.yml`:

```yml
---
- hosts: localhost
  vars:
      resource_group: aksansibletest
      name: aksansibletest
      location: eastus
  tasks:
     - name: Ensure resource group exist
       azure_rm_resourcegroup:
           name: "{{ resource_group }}"
           location: "{{ location }}"

     - name: Create vnet
       include_tasks: vnet.yml

     - name: Create AKS
       vars:
           vnet_subnet_id: "{{ subnet.state.id }}"
       include_tasks: aks.yml

     - name: Associate network resources with the node subnet
       vars:
           vnet_subnet_id: "{{ subnet.state.id }}"
           node_resource_group: "{{ aks.node_resource_group }}"
       include_tasks: associate.yml

     - name: Get details of the AKS
       azure_rm_aks_facts:
           name: "{{ name }}"
           resource_group: "{{ resource_group }}"
           show_kubeconfig: user
       register: output

     - name: Show AKS cluster detail
       debug:
           var: output.aks[0]
```

En la sección `vars`, realice los cambios siguientes:

- Para la clave `resource_group`, cambie el valor `aksansibletest` por el nombre del grupo de recursos.
- Para la clave `name`, cambie el valor `aksansibletest` por el nombre de AKS.
- Para la clave `Location`, cambie el valor `eastus` por la ubicación del grupo de recursos.

Ejecute el cuaderno de estrategias completo con el comando `ansible-playbook`:

```bash
ansible-playbook aks-kubenet.yml
```

Al ejecutar el cuaderno de estrategias, se muestran resultados similares a la salida siguiente:

```Output
PLAY [localhost] 

TASK [Gathering Facts] 
ok: [localhost]

TASK [Ensure resource group exist] 
ok: [localhost]

TASK [Create vnet] 
included: /home/devops/aks-kubenet/vnet.yml for localhost

TASK [Create vnet] 
ok: [localhost]

TASK [Create subnet] 
ok: [localhost]

TASK [Create AKS] 
included: /home/devops/aks-kubenet/aks.yml for localhost

TASK [List supported kubernetes version from Azure] 
 [WARNING]: Azure API profile latest does not define an entry for
ContainerServiceClient

ok: [localhost]

TASK [Create AKS cluster with vnet] 
changed: [localhost]

TASK [Associate network resources with the node subnet] 
included: /home/devops/aks-kubenet/associate.yml for localhost

TASK [Get route table] 
ok: [localhost]

TASK [Get network security group] 
ok: [localhost]

TASK [Parse subnet id] 
ok: [localhost]

TASK [Associate network resources with the node subnet] 
changed: [localhost]

TASK [Get details of the AKS] 
ok: [localhost]

TASK [Show AKS cluster detail] 
ok: [localhost] => {
    "output.aks[0]": {
        "id": /subscriptions/BBBBBBBB-BBBB-BBBB-BBBB-BBBBBBBBBBBB/resourcegroups/aksansibletest/providers/Microsoft.ContainerService/managedClusters/aksansibletest",
        "kube_config": "apiVersion: ...",
        "location": "eastus",
        "name": "aksansibletest",
        "properties": {
            "agentPoolProfiles": [
                {
                    "count": 3,
                    "maxPods": 110,
                    "name": "nodepool1",
                    "osDiskSizeGB": 100,
                    "osType": "Linux",
                    "storageProfile": "ManagedDisks",
                    "vmSize": "Standard_D2_v2",
                    "vnetSubnetID": "/subscriptions/BBBBBBBB-BBBB-BBBB-BBBB-BBBBBBBBBBBB/resourceGroups/aksansibletest/providers/Microsoft.Network/virtualNetworks/aksansibletest/subnets/aksansibletest"
                }
            ],
            "dnsPrefix": "aksansibletest",
            "enableRBAC": false,
            "fqdn": "aksansibletest-cda2b56c.hcp.eastus.azmk8s.io",
            "kubernetesVersion": "1.12.6",
            "linuxProfile": {
                "adminUsername": "azureuser",
                "ssh": {
                    "publicKeys": [
                        {
                            "keyData": "ssh-rsa ..."
                        }
                    ]
                }
            },
            "networkProfile": {
                "dnsServiceIP": "10.0.0.10",
                "dockerBridgeCidr": "172.17.0.1/16",
                "networkPlugin": "kubenet",
                "podCidr": "192.168.0.0/16",
                "serviceCidr": "10.0.0.0/16"
            },
            "nodeResourceGroup": "MC_aksansibletest_pcaksansibletest_eastus",
            "provisioningState": "Succeeded",
            "servicePrincipalProfile": {
                "clientId": "AAAAAAAA-AAAA-AAAA-AAAA-AAAAAAAAAAAA"
            }
        },
        "type": "Microsoft.ContainerService/ManagedClusters"
    }
}

PLAY RECAP 
localhost                  : ok=15   changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, elimine los recursos creados en este artículo. 

Guarde el siguiente código como `cleanup.yml`:

```yml
---
- hosts: localhost
  vars:
      resource_group: aksansibletest
  tasks:
      - name: Clean up resource group
        azure_rm_resourcegroup:
            name: "{{ resource_group }}"
            state: absent
            force: yes
```

En la sección `vars`, reemplace el marcador de posición `{{ resource_group_name }}` por el nombre del grupo de recursos.

Ejecute el comando de estrategias con el comando `ansible-playbook`:

```bash
ansible-playbook cleanup.yml
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Configuración de la red de Azure Container Networking Interface (CNI) en AKS con Ansible](./ansible-aks-configure-cni-networking.md)