---
title: 'Tutorial: Configuración de conjuntos de escalado de máquinas virtuales de Azure con Ansible | Microsoft Docs'
description: Aprenda a usar Ansible para crear y configurar conjuntos de escalado de máquinas virtuales de Azure
keywords: ansible, azure, devops, bash, cuaderno de estrategias, máquina virtual, conjunto de escalado de máquinas virtuales, vmss
ms.topic: tutorial
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: 41ef6103a899970142df1a6beee0ad428419f3df
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65230731"
---
# <a name="tutorial-configure-virtual-machine-scale-sets-in-azure-using-ansible"></a>Tutorial: Configuración de conjuntos de escalado de máquinas virtuales de Azure con Ansible

[!INCLUDE [ansible-27-note.md](../../includes/ansible-27-note.md)]

[!INCLUDE [open-source-devops-intro-vmss.md](../../includes/open-source-devops-intro-vmss.md)]

[!INCLUDE [ansible-tutorial-goals.md](../../includes/ansible-tutorial-goals.md)]

> [!div class="checklist"]
>
> * Configuración de los recursos de una máquina virtual
> * Configuración de un conjunto de escalado
> * Escalado del conjunto de escalado mediante el aumento de las instancias de máquina virtual 

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [ansible-prereqs-cloudshell-use-or-vm-creation2.md](../../includes/ansible-prereqs-cloudshell-use-or-vm-creation2.md)]

## <a name="configure-a-scale-set"></a>Configuración de un conjunto de escalado

El código del cuaderno de estrategias de esta sección define los siguientes recursos:

* **Grupo de recursos** en el que se implementarán todos sus recursos.
* **Red virtual** en el espacio de direcciones de 10.0.0.0/16
* **Subred** en la red virtual
* **Dirección IP pública** que permite tener acceso a recursos de Internet
* **Grupo de seguridad de red** que controla el flujo del tráfico de red de entrada y salida del conjunto de escalado.
* **Equilibrador de carga** que distribuye el tráfico a través de un conjunto de máquinas virtuales definidas usando reglas de equilibrador de carga.
* **Conjunto de escalado de máquinas virtuales** que usa todos los recursos creados

Hay dos formas de obtener el cuaderno de estrategias de ejemplo:

* [Descargar el cuaderno de estrategias](https://github.com/Azure-Samples/ansible-playbooks/blob/master/vmss/vmss-create.yml) y guardarlo en `vmss-create.yml`.
* Crear un archivo llamado `vmss-create.yml` y copiar en él el siguiente contenido:

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
    vmss_name: myScaleSet
    vmss_lb_name: myScaleSetLb
    location: eastus
    admin_username: azureuser
    admin_password: "{{ admin_password }}"
  tasks:
    - name: Create a resource group
      azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        location: "{{ location }}"
    - name: Create virtual network
      azure_rm_virtualnetwork:
        resource_group: "{{ resource_group }}"
        name: "{{ vmss_name }}"
        address_prefixes: "10.0.0.0/16"
    - name: Add subnet
      azure_rm_subnet:
        resource_group: "{{ resource_group }}"
        name: "{{ vmss_name }}"
        address_prefix: "10.0.1.0/24"
        virtual_network: "{{ vmss_name }}"
    - name: Create public IP address
      azure_rm_publicipaddress:
        resource_group: "{{ resource_group }}"
        allocation_method: Static
        name: "{{ vmss_name }}"
    - name: Create Network Security Group that allows SSH
      azure_rm_securitygroup:
        resource_group: "{{ resource_group }}"
        name: "{{ vmss_name }}"
        rules:
          - name: SSH
            protocol: Tcp
            destination_port_range: 22
            access: Allow
            priority: 1001
            direction: Inbound

    - name: Create a load balancer
      azure_rm_loadbalancer:
        name: "{{ vmss_lb_name }}"
        location: "{{ location }}"
        resource_group: "{{ resource_group }}"
        public_ip: "{{ vmss_name }}"
        probe_protocol: Tcp
        probe_port: 8080
        probe_interval: 10
        probe_fail_count: 3
        protocol: Tcp
        load_distribution: Default
        frontend_port: 80
        backend_port: 8080
        idle_timeout: 4
        natpool_frontend_port_start: 50000
        natpool_frontend_port_end: 50040
        natpool_backend_port: 22
        natpool_protocol: Tcp

    - name: Create Scale Set
      azure_rm_virtualmachinescaleset:
        resource_group: "{{ resource_group }}"
        name: "{{ vmss_name }}"
        vm_size: Standard_DS1_v2
        admin_username: "{{ admin_username }}"
        admin_password: "{{ admin_password }}"
        ssh_password_enabled: true
        capacity: 2
        virtual_network_name: "{{ vmss_name }}"
        subnet_name: "{{ vmss_name }}"
        upgrade_policy: Manual
        tier: Standard
        managed_disk_type: Standard_LRS
        os_disk_caching: ReadWrite
        image:
          offer: UbuntuServer
          publisher: Canonical
          sku: 16.04-LTS
          version: latest
        load_balancer: "{{ vmss_lb_name }}"
        data_disks:
          - lun: 0
            disk_size_gb: 20
            managed_disk_type: Standard_LRS
            caching: ReadOnly
          - lun: 1
            disk_size_gb: 30
            managed_disk_type: Standard_LRS
            caching: ReadOnly
```

Antes de ejecutar el cuaderno de estrategias, consulte las notas siguientes:

* En la sección `vars`, reemplace el marcador de posición `{{ admin_password }}` por su propia contraseña.

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook vmss-create.yml
```

Tras ejecutar el cuaderno de estrategias, debería ver resultados similares a los siguientes:

```Output
PLAY [localhost] 

TASK [Gathering Facts] 
ok: [localhost]

TASK [Create a resource group] 
changed: [localhost]

TASK [Create virtual network] 
changed: [localhost]

TASK [Add subnet] 
changed: [localhost]

TASK [Create public IP address] 
changed: [localhost]

TASK [Create Network Security Group that allows SSH] 
changed: [localhost]

TASK [Create a load balancer] 
changed: [localhost]

TASK [Create Scale Set] 
changed: [localhost]

PLAY RECAP 
localhost                  : ok=8    changed=7    unreachable=0    failed=0

```

## <a name="view-the-number-of-vm-instances"></a>Visualización del número de instancias de máquina virtual

El [conjunto de escalado configurado](#configure-a-scale-set) actualmente tiene dos instancias. Los pasos siguientes se usan para confirmar dicho valor:

1. Inicie sesión en el [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040).

1. Vaya al conjunto de escalado que ha configurado.

1. Podrá ver el nombre del conjunto de escalado con el número de instancias entre paréntesis: `Standard_DS1_v2 (2 instances)`

1. También puede comprobar el número de instancias con [Azure Cloud Shell](https://shell.azure.com/) mediante la ejecución del siguiente comando:

    ```azurecli-interactive
    az vmss show -n myScaleSet -g myResourceGroup --query '{"capacity":sku.capacity}' 
    ```

    Los resultados de la ejecución del comando de la CLI de Azure en Cloud Shell muestran que ahora existen tres instancias: 

    ```bash
    {
      "capacity": 3,
    }
    ```

## <a name="scale-out-a-scale-set"></a>Escalado horizontal de un conjunto de escalado

El código del cuaderno de estrategias de esta sección recupera información sobre el conjunto de escalado y cambia su capacidad de dos a tres.

Hay dos formas de obtener el cuaderno de estrategias de ejemplo:

* [Descargar el cuaderno de estrategias](https://github.com/Azure-Samples/ansible-playbooks/blob/master/vmss/vmss-scale-out.yml) y guardarlo en `vmss-scale-out.yml`.
* Crear un archivo llamado `vmss-scale-out.yml` y copiar en él el siguiente contenido:

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
    vmss_name: myScaleSet
  tasks: 
    - name: Get scaleset info
      azure_rm_virtualmachine_scaleset_facts:
        resource_group: "{{ resource_group }}"
        name: "{{ vmss_name }}"
        format: curated
      register: output_scaleset

    - name: Dump scaleset info
      debug:
        var: output_scaleset

    - name: Modify scaleset (change the capacity to 3)
      set_fact:
        body: "{{ output_scaleset.ansible_facts.azure_vmss[0] | combine({'capacity': 3}, recursive=True) }}"

    - name: Update something in that scale set
      azure_rm_virtualmachinescaleset: "{{ body }}"
```

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook vmss-scale-out.yml
```

Tras ejecutar el cuaderno de estrategias, debería ver resultados similares a los siguientes:

```Output
PLAY [localhost] 

TASK [Gathering Facts] 
ok: [localhost]

TASK [Get scaleset info] 
ok: [localhost]

TASK [Dump scaleset info] 
ok: [localhost] => {
    "output_scaleset": {
        "ansible_facts": {
            "azure_vmss": [
                {
                    ......
                }
            ]
        },
        "changed": false,
        "failed": false
    }
}

TASK [Modify scaleset (set upgradePolicy to Automatic and capacity to 3)] 
ok: [localhost]

TASK [Update something in that scale set] 
changed: [localhost]

PLAY RECAP 
localhost                  : ok=5    changed=1    unreachable=0    failed=0
```

## <a name="verify-the-results"></a>Verificación de los resultados

Compruebe los resultados del trabajo mediante Azure Portal:

1. Inicie sesión en el [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040).

1. Vaya al conjunto de escalado que ha configurado.

1. Podrá ver el nombre del conjunto de escalado con el número de instancias entre paréntesis: `Standard_DS1_v2 (3 instances)` 

1. También puede comprobar el cambio con [Azure Cloud Shell](https://shell.azure.com/) ejecutando el siguiente comando:

    ```azurecli-interactive
    az vmss show -n myScaleSet -g myResourceGroup --query '{"capacity":sku.capacity}' 
    ```

    Los resultados de la ejecución del comando de la CLI de Azure en Cloud Shell muestran que ahora existen tres instancias: 

    ```bash
    {
      "capacity": 3,
    }
    ```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"] 
> [Tutorial: Implementación de aplicaciones en conjuntos de escalado de máquinas virtuales de Azure con Ansible](./ansible-deploy-app-vmss.md)