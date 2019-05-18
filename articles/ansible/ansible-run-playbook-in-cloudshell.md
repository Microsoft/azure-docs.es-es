---
title: 'Inicio rápido: Ejecución de cuadernos de estrategias de Ansible a través de Bash en Azure Cloud Shell | Microsoft Docs'
description: En este inicio rápido aprenderá a realizar diversas tareas de Ansible con Bash en Azure Cloud Shell
keywords: ansible, azure, devops, bash, cloudshell, playbook, bash
ms.topic: quickstart
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: 083d10de94c39ab134b8e475b66ebf2df30088bc
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65407648"
---
# <a name="quickstart-run-ansible-playbooks-via-bash-in-azure-cloud-shell"></a>Inicio rápido: Ejecución de cuadernos de estrategias de Ansible a través de Bash en Azure Cloud Shell

Azure Cloud Shell es un shell interactivo, accesible desde el explorador, para administrar recursos de Azure. Cloud Shell le permite usar la línea de comandos de Bash o Powershell. En este artículo, va a usar Bash en Azure Cloud Shell para ejecutar un cuaderno de estrategias de Ansible.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../../includes/open-source-devops-prereqs-azure-subscription.md)]
- **Configuración de Azure Cloud Shell**: si no está familiarizado con Azure Cloud Shell, consulte [Inicio rápido para Bash en Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/quickstart).
[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="automatic-credential-configuration"></a>Configuración automática de credenciales

Cuando se inicia sesión en Cloud Shell, Ansible se autentica con Azure para administrar la infraestructura sin ninguna configuración adicional. 

Si trabaja con varias suscripciones, especifique la suscripción que usa Ansible exportando la variable de entorno `AZURE_SUBSCRIPTION_ID`. 

Para obtener una lista de las suscripciones a Azure, ejecute el siguiente comando:

```azurecli-interactive
az account list
```

Mediante el identificador de suscripción de Azure, establezca `AZURE_SUBSCRIPTION_ID` como sigue:

```azurecli-interactive
export AZURE_SUBSCRIPTION_ID=<your-subscription-id>
```

## <a name="verify-the-configuration"></a>Comprobar la configuración
Para comprobar si la configuración es correcta, use Ansible para crear un grupo de recursos de Azure.

[!INCLUDE [create-resource-group-with-ansible.md](../../includes/ansible-snippet-create-resource-group.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"] 
> [Inicio rápido: Configuración de una máquina virtual en Azure con Ansible](/azure/virtual-machines/linux/ansible-create-vm)