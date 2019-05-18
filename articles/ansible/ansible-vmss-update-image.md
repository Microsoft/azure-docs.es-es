---
title: 'Tutorial: actualización de la imagen personalizada de los conjuntos de escalado de máquinas virtuales de Azure con Ansible | Microsoft Docs'
description: Aprenda a usar Ansible para actualizar conjuntos de escalado de máquinas virtuales en Azure con una imagen personalizada.
keywords: ansible, azure, devops, bash, cuaderno de estrategias, máquina virtual, conjunto de escalado de máquinas virtuales, vmss
ms.topic: tutorial
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: d3eedc5b83190af46669b9b5df8643f3c80e9bb1
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65230854"
---
# <a name="tutorial-update-the-custom-image-of-azure-virtual-machine-scale-sets-using-ansible"></a>Tutorial: Actualización de la imagen personalizada de los conjuntos de escalado de máquinas virtuales de Azure con Ansible

[!INCLUDE [ansible-27-note.md](../../includes/ansible-28-note.md)]

[!INCLUDE [open-source-devops-intro-vmss.md](../../includes/open-source-devops-intro-vmss.md)]

Después de implementar una máquina virtual, configúrela con el software que necesite la aplicación. En lugar de hacer esta tarea de configuración para cada máquina virtual, puede crear una imagen personalizada. Una imagen personalizada es una instantánea de una máquina virtual existente que incluye todo el software instalado. Cuando [configure un conjunto de escalado](./ansible-create-configure-vmss.md), especifique la imagen que se usará para las máquinas virtuales de ese conjunto de escalado. Mediante el uso de una imagen personalizada, cada instancia de máquina virtual se configura de forma idéntica para la aplicación. En ocasiones, puede que necesite actualizar la imagen personalizada del conjunto de escalado. Esa tarea constituye el punto central de este tutorial.

[!INCLUDE [ansible-tutorial-goals.md](../../includes/ansible-tutorial-goals.md)]

> [!div class="checklist"]
>
> * Configurar dos máquinas virtuales con HTTPD
> * Crear una imagen personalizada a partir de una máquina virtual ya existente
> * Crear un conjunto de escalado a partir de una imagen
> * Actualizar la imagen personalizada

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [ansible-prereqs-cloudshell-use-or-vm-creation2.md](../../includes/ansible-prereqs-cloudshell-use-or-vm-creation2.md)]

## <a name="configure-two-vms"></a>Configuración de dos máquinas virtuales

El código del cuaderno de estrategias de esta sección crea dos máquinas virtuales con HTTPD instalado en ambas. 

La página `index.html` de cada máquina virtual muestra una cadena de prueba:

* La primera máquina virtual muestra el valor `Image A`
* La segunda muestra el valor `Image B`

Esta cadena está pensada para imitar la configuración de cada máquina virtual con software diferente.

Hay dos formas de obtener el cuaderno de estrategias de ejemplo:

* [Descargar el cuaderno de estrategias](https://github.com/Azure-Samples/ansible-playbooks/blob/master/vmss_images/01-create-vms.yml) y guardarlo en `create_vms.yml`.
* Crear un archivo llamado `create_vms.yml` y copiar en él el siguiente contenido:

```yml
- name: Create two VMs (A and B) with HTTPS
  hosts: localhost
  connection: local
  vars:
    vm_name: vmforimage
    admin_username: testuser
    admin_password: Pass123$$$abx!
    location: eastus
  tasks:
  - name: Create a resource group
    azure_rm_resourcegroup:
      name: "{{ resource_group }}"
      location: "{{ location }}"

  - name: Create virtual network
    azure_rm_virtualnetwork:
      resource_group: "{{ resource_group }}"
      name: "{{ vm_name }}"
      address_prefixes: "10.0.0.0/16"

  - name: Create subnets for VM A and B
    azure_rm_subnet:
      resource_group: "{{ resource_group }}"
      virtual_network: "{{ vm_name }}"
      name: "{{ vm_name }}"
      address_prefix: "10.0.1.0/24"

  - name: Create Network Security Group that allows HTTP
    azure_rm_securitygroup:
      resource_group: "{{ resource_group }}"
      name: "{{ vm_name }}"
      rules:
        - name: HTTP
          protocol: Tcp
          destination_port_range: 80
          access: Allow
          priority: 1002
          direction: Inbound

  - name: Create public IP addresses for VM A and B
    azure_rm_publicipaddress:
      resource_group: "{{ resource_group }}"
      allocation_method: Static
      name: "{{ vm_name }}_{{ item }}"
    loop:
      - A
      - B
    register: pip_output

  - name: Create virtual network inteface cards for VM A and B
    azure_rm_networkinterface:
      resource_group: "{{ resource_group }}"
      name: "{{ vm_name }}_{{ item }}"
      virtual_network: "{{ vm_name }}"
      subnet: "{{ vm_name }}"
      public_ip_name: "{{ vm_name }}_{{ item }}"
      security_group: "{{ vm_name }}"
    loop:
      - A
      - B

  - name: Create VM A and B
    azure_rm_virtualmachine:
      resource_group: "{{ resource_group }}"
      name: "{{ vm_name }}{{ item }}"
      admin_username: "{{ admin_username }}"
      admin_password: "{{ admin_password }}"
      vm_size: Standard_B1ms
      network_interfaces: "{{ vm_name }}_{{ item }}"
      image:
        offer: UbuntuServer
        publisher: Canonical
        sku: 16.04-LTS
        version: latest
    loop:
      - A
      - B

  - name: Create VM Extension
    azure_rm_virtualmachineextension:
      resource_group: "{{ resource_group }}"
      name: testVMExtension
      virtual_machine_name: "{{ vm_name }}{{ item }}"
      publisher: Microsoft.Azure.Extensions
      virtual_machine_extension_type: CustomScript
      type_handler_version: 2.0
      auto_upgrade_minor_version: true
      settings: {"commandToExecute": "sudo apt-get -y install apache2"}
    loop:
      - A
      - B

  - name: Create VM Extension
    azure_rm_virtualmachineextension:
      resource_group: "{{ resource_group }}"
      name: testVMExtension
      virtual_machine_name: "{{ vm_name }}{{ item }}"
      publisher: Microsoft.Azure.Extensions
      virtual_machine_extension_type: CustomScript
      type_handler_version: 2.0
      auto_upgrade_minor_version: true
      settings: {"commandToExecute": "printf '<html><body><h1>Image {{ item }}</h1></body></html>' >> index.html; sudo cp index.html /var/www/html/"}
    loop:
      - A
      - B

  - debug:
      msg: "Public IP Address A: {{ pip_output.results[0].state.ip_address }}"

  - debug:
      msg: "Public IP Address B: {{ pip_output.results[1].state.ip_address }}"
```

Ejecute el cuaderno de estrategias mediante el comando `ansible-playbook`, reemplazando `myrg` por el nombre del grupo de recursos:

```bash
ansible-playbook create-vms.yml --extra-vars "resource_group=myrg"
```

Debido a las secciones `debug` del cuaderno de estrategias, el comando `ansible-playbook` imprimirá la dirección IP de cada máquina virtual. Copie estas direcciones IP para su uso posterior.

![Direcciones IP de las máquinas virtuales](media/ansible-vmss-update-image/vmss-update-vms-ip-addresses.png)

## <a name="connect-to-the-two-vms"></a>Conexión a las dos máquinas virtuales

En esta sección, se conectará a cada máquina virtual. Como se mencionó en la sección anterior, las cadenas `Image A` y `Image B` imitan la posesión de dos máquinas virtuales distintas con configuraciones diferentes.

Use las direcciones IP de la sección anterior para conectarse a ambas máquinas virtuales:

![Captura de pantalla de la máquina virtual A](media/ansible-vmss-update-image/vmss-update-browser-vma.png)

![Captura de pantalla de la máquina virtual B](media/ansible-vmss-update-image/vmss-update-browser-vmb.png)

## <a name="create-images-from-each-vm"></a>Creación de imágenes de cada máquina virtual

En este punto, tiene dos máquinas virtuales con configuraciones ligeramente diferentes (sus archivos `index.html`).

El código del cuaderno de estrategias de esta sección crea una imagen personalizada de cada máquina virtual:

* `image_vmforimageA`: imagen personalizada que se crea para la máquina virtual que muestra `Image A` en su página principal.
* `image_vmforimageB`: imagen personalizada que se crea para la máquina virtual que muestra `Image B` en su página principal.

Hay dos formas de obtener el cuaderno de estrategias de ejemplo:

* [Descargar el cuaderno de estrategias](https://github.com/Azure-Samples/ansible-playbooks/blob/master/vmss_images/02-capture-images.yml) y guardarlo en `capture-images.yml`.
* Crear un archivo llamado `capture-images.yml` y copiar en él el siguiente contenido:

```yml
- name: Capture VM Images
  hosts: localhost
  connection: local
  vars:
    vm_name: vmforimage
  tasks:

  - name: Stop and generalize VMs
    azure_rm_virtualmachine:
      resource_group: "{{ resource_group }}"
      name: "{{ vm_name }}{{ item }}"
      generalized: yes
    loop:
      - A
      - B

  - name: Create an images from a VMs
    azure_rm_image:
      resource_group: "{{ resource_group }}"
      name: "image_{{ vm_name }}{{ item }}"
      source: "{{ vm_name }}{{ item }}"
    loop:
      - A
      - B
```

Ejecute el cuaderno de estrategias mediante el comando `ansible-playbook`, reemplazando `myrg` por el nombre del grupo de recursos:

```bash
ansible-playbook capture-images.yml --extra-vars "resource_group=myrg"
```

## <a name="create-scale-set-using-image-a"></a>Creación de un conjunto de escalado mediante la imagen A

En esta sección, se usa un cuaderno de estrategias para configurar los siguientes recursos de Azure:

* Dirección IP pública
* Equilibrador de carga
* Conjunto de escalado que hace referencia a `image_vmforimageA`

Hay dos formas de obtener el cuaderno de estrategias de ejemplo:

* [Descargar el cuaderno de estrategias](https://github.com/Azure-Samples/ansible-playbooks/blob/master/vmss_images/03-create-vmss.yml) y guardarlo en `create-vmss.yml`.
* Crear un archivo llamado `create-vmss.yml` y copiar en él el siguiente contenido:

```yml
---
- hosts: localhost
  vars:
    vmss_name: vmsstest
    location: eastus
    admin_username: vmssadmin
    admin_password: User123!!!abc
    vm_name: vmforimage
    image_name: "image_vmforimageA"

  tasks:

    - name: Create public IP address
      azure_rm_publicipaddress:
        resource_group: "{{ resource_group }}"
        allocation_method: Static
        name: "{{ vmss_name }}"
      register: pip_output

    - name: Create a load balancer
      azure_rm_loadbalancer:
        name: "{{ vmss_name }}lb"
        location: "{{ location }}"
        resource_group: "{{ resource_group }}"
        public_ip: "{{ vmss_name }}"
        probe_protocol: Tcp
        probe_port: 80
        probe_interval: 10
        probe_fail_count: 3
        protocol: Tcp
        load_distribution: Default
        frontend_port: 80
        backend_port: 80
        idle_timeout: 4
        natpool_frontend_port_start: 50000
        natpool_frontend_port_end: 50040
        natpool_backend_port: 22
        natpool_protocol: Tcp

    - name: Create a scale set
      azure_rm_virtualmachinescaleset:
        resource_group: "{{ resource_group }}"
        name: "{{ vmss_name }}"
        vm_size: Standard_DS1_v2
        admin_username: "{{ admin_username }}"
        admin_password: "{{ admin_password }}"
        ssh_password_enabled: true
        capacity: 2
        virtual_network_name: "{{ vm_name }}"
        subnet_name: "{{ vm_name }}"
        upgrade_policy: Manual
        tier: Standard
        managed_disk_type: Standard_LRS
        os_disk_caching: ReadWrite
        image:
          name: "{{ image_name }}"
          resource_group: "{{ resource_group }}"
        load_balancer: "{{ vmss_name }}lb"

    - debug:
        msg: "Scale set public IP address: {{ pip_output.state.ip_address }}"
```

Ejecute el cuaderno de estrategias mediante el comando `ansible-playbook`, reemplazando `myrg` por el nombre del grupo de recursos:

```bash
ansible-playbook create-vmss.yml --extra-vars "resource_group=myrg"
```

Debido a las secciones `debug` del cuaderno de estrategias, el comando `ansible-playbook` imprimirá la dirección IP del conjunto de escalado. Copie esta dirección IP para su uso posterior.

![Dirección IP pública](media/ansible-vmss-update-image/vmss-update-vmss-public-ip.png)

## <a name="connect-to-the-scale-set"></a>Conexión al conjunto de escalado

En esta sección, se va a conectar al conjunto de escalado. 

Use las direcciones IP de la sección anterior para conectarse al conjunto de escalado.

Como se mencionó en la sección anterior, las cadenas `Image A` y `Image B` imitan la posesión de dos máquinas virtuales distintas con configuraciones diferentes.

El conjunto de escalado hace referencia a la imagen personalizada llamada `image_vmforimageA`. La imagen personalizada `image_vmforimageA` se creó a partir de la máquina virtual en cuya página principal aparece `Image A`.

Como resultado, verá una página principal en la que aparece `Image A`:

![El conjunto de escalado está asociado con la primera máquina virtual.](media/ansible-vmss-update-image/vmss-update-browser-initial-vmss.png)

Deje la ventana del explorador abierta mientras continúa con la siguiente sección.

## <a name="change-custom-image-in-scale-set-and-upgrade-instances"></a>Cambio de la imagen personalizada de un conjunto de escalado y actualización de las instancias

El código del cuaderno de estrategias de esta sección cambia la imagen del conjunto de escalado de `image_vmforimageA` a `image_vmforimageB`. Además, se actualizarán todas las máquinas virtuales actuales que implementó el conjunto de escalado.

Hay dos formas de obtener el cuaderno de estrategias de ejemplo:

* [Descargar el cuaderno de estrategias](https://github.com/Azure-Samples/ansible-playbooks/blob/master/vmss_images/04-update-vmss-image.yml) y guardarlo en `update-vmss-image.yml`.
* Crear un archivo llamado `update-vmss-image.yml` y copiar en él el siguiente contenido:

```yml
- name: Update scale set image reference
  hosts: localhost
  connection: local
  vars:
    vmss_name: vmsstest
    image_name: image_vmforimageB
    admin_username: vmssadmin
    admin_password: User123!!!abc
  tasks:

  - name: Update scale set - second image
    azure_rm_virtualmachinescaleset:
      resource_group: "{{ resource_group }}"
      name: "{{ vmss_name }}"
      vm_size: Standard_DS1_v2
      admin_username: "{{ admin_username }}"
      admin_password: "{{ admin_password }}"
      ssh_password_enabled: true
      capacity: 3
      virtual_network_name: "{{ vmss_name }}"
      subnet_name: "{{ vmss_name }}"
      upgrade_policy: Manual
      tier: Standard
      managed_disk_type: Standard_LRS
      os_disk_caching: ReadWrite
      image:
        name: "{{ image_name }}"
        resource_group: "{{ resource_group }}"
      load_balancer: "{{ vmss_name }}lb"

  - name: List all of the instances
    azure_rm_virtualmachinescalesetinstance_facts:
      resource_group: "{{ resource_group }}"
      vmss_name: "{{ vmss_name }}"
    register: instances

  - debug:
      var: instances

  - name: manually upgrade all the instances 
    azure_rm_virtualmachinescalesetinstance:
      resource_group: "{{ resource_group }}"
      vmss_name: "{{ vmss_name }}"
      instance_id: "{{ item.instance_id }}"
      latest_model: yes
    with_items: "{{ instances.instances }}"
```

Ejecute el cuaderno de estrategias mediante el comando `ansible-playbook`, reemplazando `myrg` por el nombre del grupo de recursos:

```bash
ansible-playbook update-vmss-image.yml --extra-vars "resource_group=myrg"
```

Regrese al explorador y actualice la página. 

Verá que se actualiza la imagen personalizada subyacente de la máquina virtual.

![El conjunto de escalado está asociado con la segunda máquina virtual.](media/ansible-vmss-update-image/vmss-update-browser-updated-vmss.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, elimine los recursos creados en este artículo. 

Guarde el siguiente código como `cleanup.yml`:

```yml
- hosts: localhost
  vars:
    resource_group: myrg
  tasks:
    - name: Delete a resource group
      azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        force_delete_nonempty: yes
        state: absent
```

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook cleanup.yml
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"] 
> [Ansible en Azure](/azure/ansible)