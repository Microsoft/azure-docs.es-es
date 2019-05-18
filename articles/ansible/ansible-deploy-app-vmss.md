---
title: 'Tutorial: Implementación de aplicaciones en conjuntos de escalado de máquinas virtuales de Azure con Ansible | Microsoft Docs'
description: Obtenga información sobre cómo usar Ansible para configurar conjuntos de escalado de máquinas virtuales de Azure e implementar una aplicación en el conjunto de escalado.
keywords: ansible, azure, devops, bash, cuaderno de estrategias, máquina virtual, conjunto de escalado de máquinas virtuales, vmss
ms.topic: tutorial
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: a44fd06ace9b21122f5f4253ac7d9601b54e6b62
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65231052"
---
# <a name="tutorial-deploy-apps-to-virtual-machine-scale-sets-in-azure-using-ansible"></a>Tutorial: Implementación de aplicaciones en conjuntos de escalado de máquinas virtuales de Azure con Ansible

[!INCLUDE [ansible-27-note.md](../../includes/ansible-27-note.md)]

[!INCLUDE [open-source-devops-intro-vmss.md](../../includes/open-source-devops-intro-vmss.md)]

[!INCLUDE [ansible-tutorial-goals.md](../../includes/ansible-tutorial-goals.md)]

> [!div class="checklist"]
>
> * Recuperar la información del host para un grupo de máquinas virtuales de Azure
> * Clonar y compilar la aplicación de ejemplo
> * Instalar el JRE (Java Runtime Environment) en un conjunto de escalado
> * Implementar la aplicación Java en un conjunto de escalado

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [ansible-prereqs-cloudshell-use-or-vm-creation2.md](../../includes/ansible-prereqs-cloudshell-use-or-vm-creation2.md)] 
[!INCLUDE [ansible-prereqs-vm-scale-set.md](../../includes/ansible-prereqs-vm-scale-set.md)]
- **git** - [git](https://git-scm.com) se usa para descargar un ejemplo de Java utilizado en este tutorial.
- **Java SE Development Kit (JDK)**: el [JDK](https://aka.ms/azure-jdks) se usa para compilar el proyecto de Java de ejemplo.
- **Apache Maven** - [Apache Maven](https://maven.apache.org/download.cgi) se usa para compilar el proyecto de Java de ejemplo.

## <a name="get-host-information"></a>Obtención de información sobre host

El código del cuaderno de estrategias de esta sección recupera la información del host para un grupo de máquinas virtuales. El código obtiene las direcciones IP públicas y el equilibrador de carga de un grupo de recursos especificado y crea un grupo host denominado `scalesethosts` en el inventario.

Guarde el siguiente cuaderno de estrategias de ejemplo como `get-hosts-tasks.yml`:

  ```yml
  - name: Get facts for all Public IPs within a resource groups
    azure_rm_publicipaddress_facts:
      resource_group: "{{ resource_group }}"
    register: output_ip_address

  - name: Get loadbalancer info
    azure_rm_loadbalancer_facts:
      resource_group: "{{ resource_group }}"
      name: "{{ loadbalancer_name }}"
    register: output

  - name: Add all hosts
    add_host:
      groups: scalesethosts
      hostname: "{{ output_ip_address.ansible_facts.azure_publicipaddresses[0].properties.ipAddress }}_{{ item.properties.frontendPort }}"
      ansible_host: "{{ output_ip_address.ansible_facts.azure_publicipaddresses[0].properties.ipAddress }}"
      ansible_port: "{{ item.properties.frontendPort }}"
      ansible_ssh_user: "{{ admin_username }}"
      ansible_ssh_pass: "{{ admin_password }}"
    with_items:
      - "{{ output.ansible_facts.azure_loadbalancers[0].properties.inboundNatRules }}"
  ```

## <a name="prepare-an-application-for-deployment"></a>Preparación de una aplicación para la implementación

El código del cuaderno de estrategias de esta sección utiliza `git` para clonar un proyecto de ejemplo de Java desde GitHub y compila el proyecto. 

Guarde el siguiente cuaderno de estrategias como `app.yml`:

  ```yml
  - hosts: localhost
    vars:
      repo_url: https://github.com/spring-guides/gs-spring-boot.git
      workspace: ~/src/helloworld

    tasks:
    - name: Git Clone sample app
      git:
        repo: "{{ repo_url }}"
        dest: "{{ workspace }}"

    - name: Build sample app
      shell: mvn package chdir="{{ workspace }}/complete"
  ```

Ejecute el cuaderno de estrategias de Ansible de ejemplo con el siguiente comando:

  ```bash
  ansible-playbook app.yml
  ```

Tras ejecutar el cuaderno de estrategias, debería ver resultados similares a los siguientes:

  ```Output
  PLAY [localhost] 

  TASK [Gathering Facts] 
  ok: [localhost]

  TASK [Git Clone sample app] 
  changed: [localhost]

  TASK [Build sample app] 
  changed: [localhost]

  PLAY RECAP 
  localhost                  : ok=3    changed=2    unreachable=0    failed=0

  ```

## <a name="deploy-the-application-to-a-scale-set"></a>Implementación de la aplicación en un conjunto de escalado

El código del cuaderno de estrategias de esta sección se usa para:

* Instalar el JRE en un grupo host llamado `saclesethosts`
* Implementar la aplicación de Java en un grupo host denominado `saclesethosts`

Hay dos formas de obtener el cuaderno de estrategias de ejemplo:

* [Descargar el cuaderno de estrategias](https://github.com/Azure-Samples/ansible-playbooks/blob/master/vmss/vmss-setup-deploy.yml) y guardarlo en `vmss-setup-deploy.yml`.
* Crear un archivo llamado `vmss-setup-deploy.yml` y copiar en él el siguiente contenido:

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
    scaleset_name: myScaleSet
    loadbalancer_name: myScaleSetLb
    admin_username: azureuser
    admin_password: "{{ admin_password }}"
  tasks:
  - include: get-hosts-tasks.yml

- name: Install JRE on a scale set
  hosts: scalesethosts
  become: yes
  vars:
    workspace: ~/src/helloworld
    admin_username: azureuser

  tasks:
  - name: Install JRE
    apt:
      name: default-jre
      update_cache: yes

  - name: Copy app to Azure VM
    copy:
      src: "{{ workspace }}/complete/target/gs-spring-boot-0.1.0.jar"
      dest: "/home/{{ admin_username }}/helloworld.jar"
      force: yes
      mode: 0755

  - name: Start the application
    shell: java -jar "/home/{{ admin_username }}/helloworld.jar" >/dev/null 2>&1 &
    async: 5000
    poll: 0
```

Antes de ejecutar el cuaderno de estrategias, vea las notas siguientes:

* En la sección `vars`, reemplace el marcador de posición `{{ admin_password }}` por su propia contraseña.
* Para usar el tipo de conexión ssh con contraseñas, instale el programa sshpass:

    Ubuntu:

    ```bash
    apt-get install sshpass
    ```

    CentOS:

    ```bash
    yum install sshpass
    ```

* En algunos entornos, puede que vea un error sobre el uso de una contraseña SSH en lugar de una clave. Si recibe ese error, puede deshabilitar la comprobación de la clave de host con la adición de la línea siguiente a `/etc/ansible/ansible.cfg` o `~/.ansible.cfg`:

    ```bash
    [defaults]
    host_key_checking = False
    ```

Ejecute el cuaderno de estrategias con el siguiente comando:

  ```bash
  ansible-playbook vmss-setup-deploy.yml
  ```

El resultado de la ejecución del comando ansible-playbook indica que la aplicación Java de ejemplo se ha instalado en el grupo host del conjunto de escalado:

  ```Output
  PLAY [localhost]

  TASK [Gathering Facts]
  ok: [localhost]

  TASK [Get facts for all Public IPs within a resource groups]
  ok: [localhost]

  TASK [Get loadbalancer info]
  ok: [localhost]

  TASK [Add all hosts]
  changed: [localhost] ...

  PLAY [Install JRE on scale set]

  TASK [Gathering Facts]
  ok: [40.114.30.145_50000]
  ok: [40.114.30.145_50003]

  TASK [Copy app to Azure VM]
  changed: [40.114.30.145_50003]
  changed: [40.114.30.145_50000]

  TASK [Start the application]
  changed: [40.114.30.145_50000]
  changed: [40.114.30.145_50003]

  PLAY RECAP
  40.114.30.145_50000        : ok=4    changed=3    unreachable=0    failed=0
  40.114.30.145_50003        : ok=4    changed=3    unreachable=0    failed=0
  localhost                  : ok=4    changed=1    unreachable=0    failed=0
  ```

## <a name="verify-the-results"></a>Verificación de los resultados

Verifique los resultados de su trabajo en la dirección URL del equilibrador de carga del conjunto de escalado:

![Aplicación Java en ejecución en un conjunto de escalado de Azure.](media/ansible-vmss-deploy/ansible-deploy-app-vmss.png)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Escalabilidad automática de los conjuntos de escalado de máquinas virtuales de Azure mediante Ansible](./ansible-auto-scale-vmss.md)