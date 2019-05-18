---
title: 'Inicio rápido: Instalación de Ansible en máquinas virtuales Linux en Azure | Microsoft Docs'
description: En este inicio rápido, aprenderá a instalar Ansible y a configurarlo para administrar recursos de Azure en Ubuntu, CentOS y SLES.
keywords: ansible, azure, devops, bash, cloudshell, playbook, bash
ms.topic: quickstart
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: 60cefe24feab9de4262e81eb1bc313aeadc7eb05
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65409258"
---
# <a name="quickstart-install-ansible-on-linux-virtual-machines-in-azure"></a>Inicio rápido: Instalación de Ansible en máquinas virtuales Linux en Azure

Ansible permite automatizar la implementación y la configuración de recursos en un entorno. En este artículo se muestra cómo configurar Ansible para algunas de las distribuciones de Linux más habituales. Para instalar Ansible en otras distribuciones, ajuste los paquetes instalados para adaptarlos a la plataforma en concreto. 

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [open-source-devops-prereqs-azure-sub.md](../../../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [open-source-devops-prereqs-create-sp.md](../../../includes/open-source-devops-prereqs-create-service-principal.md)]
- **Acceso a Linux o a una máquina virtual Linux**: si no tiene una máquina Linux, cree una [máquina virtual Linux](/azure/virtual-network/quick-create-cli).

## <a name="install-ansible-on-an-azure-linux-virtual-machine"></a>Instalación de Ansible en una máquina virtual Linux de Azure

Inicie sesión en su equipo Linux y seleccione una de las distribuciones siguientes para obtener información sobre cómo instalar Ansible:

- [CentOS 7.4](#centos-74)
- Ubuntu 16.04 LTS
- [SLES 12 SP2](#sles-12-sp2)

### <a name="centos-74"></a>CentOS 7.4

En esta sección, configurará CentOS para usar Ansible.

1. Abra una ventana del terminal.

1. Escriba el siguiente comando para instalar los paquetes necesarios para los módulos de SDK de Python de Azure:

    ```bash
    sudo yum check-update; sudo yum install -y gcc libffi-devel python-devel openssl-devel epel-release
    sudo yum install -y python-pip python-wheel
    ```

1. Escriba el siguiente comando para instalar los paquetes necesarios de Ansible:

    ```bash
    sudo pip install ansible[azure]
    ```

1. [Cree las credenciales de Azure](#create-azure-credentials).

### <a name="ubuntu-1604-lts"></a>Ubuntu 16.04 LTS

En esta sección, configurará Ubuntu para usar Ansible.

1. Abra una ventana del terminal.

1. Escriba el siguiente comando para instalar los paquetes necesarios para los módulos de SDK de Python de Azure:

    ```bash
    sudo apt-get update && sudo apt-get install -y libssl-dev libffi-dev python-dev python-pip
    ```

1. Escriba el siguiente comando para instalar los paquetes necesarios de Ansible:

    ```bash
    sudo pip install ansible[azure]
    ```

1. [Cree las credenciales de Azure](#create-azure-credentials).

### <a name="sles-12-sp2"></a>SLES 12 SP2

En esta sección, configurará SLES para usar Ansible.

1. Abra una ventana del terminal.

1. Escriba el siguiente comando para instalar los paquetes necesarios para los módulos de SDK de Python de Azure:

    ```bash
    sudo zypper refresh && sudo zypper --non-interactive install gcc libffi-devel-gcc5 make \
        python-devel libopenssl-devel libtool python-pip python-setuptools
    ```

1. Escriba el siguiente comando para instalar los paquetes necesarios de Ansible:

    ```bash
    sudo pip install ansible[azure]
    ```

1. Escriba el siguiente comando para quitar el paquete de criptografía de Python en conflicto:

    ```bash
    sudo pip uninstall -y cryptography
    ```

1. [Cree las credenciales de Azure](#create-azure-credentials).

## <a name="create-azure-credentials"></a>Creación de credenciales de Azure

Para configurar las credenciales de Ansible, necesita la siguiente información:

* El identificador de suscripción a Azure 
* Valores de la entidad de servicio

Si usa Ansible Tower o Jenkins, declare los valores de la entidad de servicio como variables de entorno.

Configure las credenciales de Ansible con una de las técnicas siguientes:

- [Creación de un archivo de credenciales de Ansible](#file-credentials)
- [Uso de variables de entorno de Ansible](#env-credentials)

### <a name="span-idfile-credentials-create-ansible-credentials-file"></a><span id="file-credentials"/> Creación de un archivo de credenciales de Ansible

En esta sección, creará un archivo de credenciales local para proporcionar credenciales a Ansible. 

Para obtener más información sobre cómo definir las credenciales de Ansible, consulte [Providing Credentials to Azure Modules](https://docs.ansible.com/ansible/guide_azure.html#providing-credentials-to-azure-modules) (Provisión de credenciales a módulos de Azure).

1. Para un entorno de desarrollo, cree un archivo llamado `credentials` en la máquina virtual de host:

    ```bash
    mkdir ~/.azure
    vi ~/.azure/credentials
    ```

1. Inserte las líneas siguientes al archivo. Reemplace los marcadores de posición por los valores de la entidad de servicio.

    ```bash
    [default]
    subscription_id=<your-subscription_id>
    client_id=<security-principal-appid>
    secret=<security-principal-password>
    tenant=<security-principal-tenant>
    ```

1. Guarde y cierre el archivo.

### <a name="span-idenv-credentialsuse-ansible-environment-variables"></a><span id="env-credentials"/> Uso de variables de entorno de Ansible

En esta sección, exportará los valores de la entidad de servicio para configurar las credenciales de Ansible.

1. Abra una ventana del terminal.

1. Exporte los valores de la entidad de servicio:

    ```bash
    export AZURE_SUBSCRIPTION_ID=<your-subscription_id>
    export AZURE_CLIENT_ID=<security-principal-appid>
    export AZURE_SECRET=<security-principal-password>
    export AZURE_TENANT=<security-principal-tenant>
    ```

## <a name="verify-the-configuration"></a>Comprobar la configuración

Para comprobar si la configuración es correcta, use Ansible para crear un grupo de recursos de Azure.

[!INCLUDE [create-resource-group-with-ansible.md](../../../includes/ansible-snippet-create-resource-group.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"] 
> [Inicio rápido: Configuración de una máquina virtual Linux en Azure con Ansible](./ansible-create-vm.md)