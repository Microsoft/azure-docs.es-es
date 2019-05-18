---
title: 'Inicio rápido: Configuración de máquinas virtuales Linux en Azure con Ansible | Microsoft Docs'
description: En este inicio rápido, aprenderá a crear una máquina virtual Linux en Azure mediante Ansible.
keywords: ansible, azure, devops, máquina virtual
ms.topic: tutorial
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: ce99b537dd5958c2bec43759c58a9c182dd05142
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65237025"
---
# <a name="quickstart-configure-linux-virtual-machines-in-azure-using-ansible"></a>Inicio rápido: Configuración de máquinas virtuales Linux en Azure con Ansible

Mediante un lenguaje declarativo, Ansible permite automatizar la creación, configuración e implementación de recursos de Azure con los *cuadernos* de Ansible. En este artículo se presenta un cuaderno de estrategias de Ansible de ejemplo para configurar las máquinas virtuales Linux. El [cuaderno de Ansible completo](#complete-sample-ansible-playbook) aparece al final de este artículo.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [open-source-devops-prereqs-azure-sub.md](../../../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [ansible-prereqs-cloudshell-use-or-vm-creation1.md](../../../includes/ansible-prereqs-cloudshell-use-or-vm-creation1.md)]

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Ansible necesita un grupo de recursos en el que implementará todos los recursos. En la siguiente sección, el cuaderno de Ansible de ejemplo crea un grupo de recursos llamado `myResourceGroup` en la ubicación `eastus`:

```yaml
- name: Create resource group
  azure_rm_resourcegroup:
    name: myResourceGroup
    location: eastus
```

## <a name="create-a-virtual-network"></a>Creación de una red virtual

Cuando se crea una máquina virtual de Azure, es preciso crear una [red virtual](/azure/virtual-network/virtual-networks-overview) o usar una red virtual existente. También es preciso decidir la forma en que pretende que se acceda a las máquinas virtuales de la red virtual. En la siguiente sección, el cuaderno de Ansible crea una red virtual llamada `myVnet` en el espacio de direcciones `10.0.0.0/16`:

```yaml
- name: Create virtual network
  azure_rm_virtualnetwork:
    resource_group: myResourceGroup
    name: myVnet
    address_prefixes: "10.0.0.0/16"
```

Todos los recursos de Azure implementados en una red virtual se implementan en una [subred](/azure/virtual-network/virtual-network-manage-subnet) de esa red virtual. 

En la siguiente sección, el cuaderno de Ansible de ejemplo crea una subred llamada `mySubnet` en la red virtual `myVnet`:

```yaml
- name: Add subnet
  azure_rm_subnet:
    resource_group: myResourceGroup
    name: mySubnet
    address_prefix: "10.0.1.0/24"
    virtual_network: myVnet
```

## <a name="create-a-public-ip-address"></a>Crear una dirección IP pública





Las [direcciones IP públicas](/azure/virtual-network/virtual-network-ip-addresses-overview-arm) permiten a los recursos de Internet la comunicación entrante a los recursos de Azure. También habilitan los recursos de Azure para la comunicación saliente a los servicios de Azure orientados al público. En ambos escenarios, se asigna una dirección IP al recurso al que se tiene acceso. Hasta que cancele la asignación, la dirección estará dedicada al recurso. Si el recurso no tiene asignada una dirección IP pública, el recurso puede seguir comunicándose hacia Internet. La conexión se realiza mediante Azure asignando dinámicamente una dirección IP disponible. La dirección asignada dinámicamente no está dedicada a al recurso.

En la siguiente sección, el cuaderno de Ansible de ejemplo crea una dirección IP pública llamada `myPublicIP`:

```yaml
- name: Create public IP address
  azure_rm_publicipaddress:
    resource_group: myResourceGroup
    allocation_method: Static
    name: myPublicIP
```

## <a name="create-a-network-security-group"></a>Crear un grupo de seguridad de red

Los [grupos de seguridad de red](/azure/virtual-network/security-overview) filtran el tráfico de red entre los recursos de Azure en una red virtual. Se definen las reglas de seguridad que controlan el tráfico entrante y saliente hacia y desde los recursos de Azure. Para obtener más información acerca de los recursos de Azure y los grupos de seguridad de red, consulte [Integración de red virtual para los servicios de Azure](/azure/virtual-network/virtual-network-for-azure-services).

En el cuaderno de estrategias siguiente, se crea un grupo de seguridad de red denominado `myNetworkSecurityGroup`. El grupo de seguridad de red incluye una regla que permite el tráfico en el puerto TCP 22.

```yaml
- name: Create Network Security Group that allows SSH
  azure_rm_securitygroup:
    resource_group: myResourceGroup
    name: myNetworkSecurityGroup
    rules:
      - name: SSH
        protocol: Tcp
        destination_port_range: 22
        access: Allow
        priority: 1001
        direction: Inbound
```

## <a name="create-a-virtual-network-interface-card"></a>Creación de una tarjeta de interfaz de red virtual

Una tarjeta de interfaz de red virtual conecta la máquina virtual a una red virtual determinada, una dirección IP pública y un grupo de seguridad de red. 

En la siguiente sección, el cuaderno de Ansible de ejemplo crea una tarjeta de interfaz de red virtual llamada `myNIC` conectada a los recursos de red virtual que ha creado:

```yaml
- name: Create virtual network inteface card
  azure_rm_networkinterface:
    resource_group: myResourceGroup
    name: myNIC
    virtual_network: myVnet
    subnet: mySubnet
    public_ip_name: myPublicIP
    security_group: myNetworkSecurityGroup
```

## <a name="create-a-virtual-machine"></a>de una máquina virtual

El último paso es crear una máquina virtual que usa todos los recursos que ha creado en las secciones anteriores de este artículo. 

El cuaderno de Ansible de ejemplo presentado en esta sección crea una máquina virtual llamada `myVM` y conecta la tarjeta de interfaz de red virtual llamada `myNIC`. Reemplace el marcador de posición &lt;datos-de-clave> por sus propios datos de clave pública completos.

```yaml
- name: Create VM
  azure_rm_virtualmachine:
    resource_group: myResourceGroup
    name: myVM
    vm_size: Standard_DS1_v2
    admin_username: azureuser
    ssh_password_enabled: false
    ssh_public_keys:
      - path: /home/azureuser/.ssh/authorized_keys
        key_data: <your-key-data>
    network_interfaces: myNIC
    image:
      offer: CentOS
      publisher: OpenLogic
      sku: '7.5'
      version: latest
```

## <a name="complete-sample-ansible-playbook"></a>Cuaderno de Ansible de ejemplo completo

Esta sección enumera el cuaderno de Ansible de ejemplo completo que creó en el transcurso de este artículo. 

```yaml
- name: Create Azure VM
  hosts: localhost
  connection: local
  tasks:
  - name: Create resource group
    azure_rm_resourcegroup:
      name: myResourceGroup
      location: eastus
  - name: Create virtual network
    azure_rm_virtualnetwork:
      resource_group: myResourceGroup
      name: myVnet
      address_prefixes: "10.0.0.0/16"
  - name: Add subnet
    azure_rm_subnet:
      resource_group: myResourceGroup
      name: mySubnet
      address_prefix: "10.0.1.0/24"
      virtual_network: myVnet
  - name: Create public IP address
    azure_rm_publicipaddress:
      resource_group: myResourceGroup
      allocation_method: Static
      name: myPublicIP
    register: output_ip_address
  - name: Dump public IP for VM which will be created
    debug:
      msg: "The public IP is {{ output_ip_address.state.ip_address }}."
  - name: Create Network Security Group that allows SSH
    azure_rm_securitygroup:
      resource_group: myResourceGroup
      name: myNetworkSecurityGroup
      rules:
        - name: SSH
          protocol: Tcp
          destination_port_range: 22
          access: Allow
          priority: 1001
          direction: Inbound
  - name: Create virtual network inteface card
    azure_rm_networkinterface:
      resource_group: myResourceGroup
      name: myNIC
      virtual_network: myVnet
      subnet: mySubnet
      public_ip_name: myPublicIP
      security_group: myNetworkSecurityGroup
  - name: Create VM
    azure_rm_virtualmachine:
      resource_group: myResourceGroup
      name: myVM
      vm_size: Standard_DS1_v2
      admin_username: azureuser
      ssh_password_enabled: false
      ssh_public_keys:
        - path: /home/azureuser/.ssh/authorized_keys
          key_data: <your-key-data>
      network_interfaces: myNIC
      image:
        offer: CentOS
        publisher: OpenLogic
        sku: '7.5'
        version: latest
```

## <a name="run-the-sample-ansible-playbook"></a>Ejecución del cuaderno de Ansible de ejemplo

Esta sección le guía en la ejecución del cuaderno de Ansible de ejemplo presentada en este artículo.

1. Inicie sesión en el [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040).

1. Abra [Cloud Shell](/azure/cloud-shell/overview).

1. Cree un archivo (para que contenga el cuaderno) llamado `azure_create_complete_vm.yml` y ábralo en el editor VI como se indica a continuación:

   ```azurecli-interactive
   vi azure_create_complete_vm.yml
   ```

1. Seleccione la tecla **I** para acceder al modo de inserción.

1. Pegue el [cuaderno de Ansible de ejemplo completo](#complete-sample-ansible-playbook) en el editor.

1. Seleccione la tecla **Esc** para acceder al modo de inserción.

1. Guarde el archivo y salga del editor vi, para lo que debe escribir el siguiente comando:

    ```bash
    :wq
    ```

1. Ejecute el cuaderno de Ansible de ejemplo.

   ```bash
   ansible-playbook azure_create_complete_vm.yml
   ```

1. La salida es similar a la siguiente, en la que se muestra que la máquina virtual se ha creado correctamente:

   ```bash
   PLAY [Create Azure VM] ****************************************************

   TASK [Gathering Facts] ****************************************************
   ok: [localhost]

   TASK [Create resource group] *********************************************
   changed: [localhost]

   TASK [Create virtual network] *********************************************
   changed: [localhost]

   TASK [Add subnet] *********************************************************
   changed: [localhost]

   TASK [Create public IP address] *******************************************
   changed: [localhost]

   TASK [Dump public IP for VM which will be created] ********************************************************************
   ok: [localhost] => {
      "msg": "The public IP is <ip-address>."
   }

   TASK [Create Network Security Group that allows SSH] **********************
   changed: [localhost]

   TASK [Create virtual network inteface card] *******************************
   changed: [localhost]

   TASK [Create VM] **********************************************************
   changed: [localhost]

   PLAY RECAP ****************************************************************
   localhost                  : ok=8    changed=7    unreachable=0    failed=0
   ```

1. El comando SSH se usa para acceder a la máquina virtual Linux. Reemplace el marcador de posición &lt;dirección-ip> por la dirección IP del paso anterior.

    ```bash
    ssh azureuser@<ip-address>
    ```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"] 
> [Inicio rápido: Manage a Linux virtual machine in Azure using Ansible](./ansible-manage-linux-vm.md) (Inicio rápido: Administración de una máquina virtual Linux en Azure mediante Ansible)