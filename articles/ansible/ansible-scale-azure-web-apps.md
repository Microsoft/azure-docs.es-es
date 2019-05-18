---
title: 'Tutorial: Escalado de aplicaciones en Azure App Service con Ansible | Microsoft Docs'
description: Aprenda a escalar verticalmente una aplicación en Azure App Service.
keywords: ansible, azure, devops, bash, playbook, Azure App Service, Web App, escala, Java
ms.topic: tutorial
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: d63708cd87afa426f2712da6d0fcb11c84590798
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65230949"
---
# <a name="tutorial-scale-apps-in-azure-app-service-using-ansible"></a>Tutorial: Escalado de aplicaciones en Azure App Service con Ansible

[!INCLUDE [ansible-27-note.md](../../includes/ansible-27-note.md)]

[!INCLUDE [open-source-devops-intro-app-service.md](../../includes/open-source-devops-intro-app-service.md)]

[!INCLUDE [ansible-tutorial-goals.md](../../includes/ansible-tutorial-goals.md)]

> [!div class="checklist"]
>
> * Obtener datos de un plan de App Service existente
> * Escalado vertical del plan de App Service a S2 con tres roles de trabajo

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [ansible-prereqs-cloudshell-use-or-vm-creation2.md](../../includes/ansible-prereqs-cloudshell-use-or-vm-creation2.md)]
- Una **aplicación de Azure App Service**: si no tiene una aplicación de Azure App Service, [configure una aplicación en Azure App Service con Ansible](ansible-create-configure-azure-web-apps.md).

## <a name="scale-up-an-app"></a>Escalado vertical de una aplicación

Existen dos flujos de trabajo de escalado: *escalado vertical* y *escalado horizontal*.

**Escalado vertical:** Escalar verticalmente significa adquirir más recursos. Estos recursos incluyen CPU, memoria, espacio en disco, máquinas virtuales, etc. Para escalar verticalmente una aplicación, cambie el plan de tarifa del plan de App Service al que pertenece la aplicación. 
**Escalado horizontal**: Significa aumentar el número de instancias de máquina virtual que ejecutan la aplicación. Según el plan de tarifa de App Service, puede escalar horizontalmente a un máximo de 20 instancias. El [escalado automático](/azure/azure-monitor/platform/autoscale-get-started) le permite escalar el total de instancias automáticamente en función de las programaciones y reglas predefinidas.

El código del cuaderno de estrategias de esta sección define la siguiente operación:

* Obtener datos de un plan de App Service existente
* Actualizar el plan de App Service a S2 con tres roles de trabajo

Guarde el siguiente cuaderno de estrategias como `webapp_scaleup.yml`:

```yml
- hosts: localhost
  connection: local
  vars:
    resource_group: myResourceGroup
    plan_name: myAppServicePlan
    location: eastus

  tasks:
  - name: Get facts of existing App service plan
    azure_rm_appserviceplan_facts:
      resource_group: "{{ resource_group }}"
      name: "{{ plan_name }}"
    register: facts

  - debug: 
      var: facts.appserviceplans[0].sku

  - name: Scale up the App service plan
    azure_rm_appserviceplan:
      resource_group: "{{ resource_group }}"
      name: "{{ plan_name }}"
      is_linux: true
      sku: S2
      number_of_workers: 3
      
  - name: Get facts
    azure_rm_appserviceplan_facts:
      resource_group: "{{ resource_group }}"
      name: "{{ plan_name }}"
    register: facts

  - debug: 
      var: facts.appserviceplans[0].sku
```

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook webapp_scaleup.yml
```

Tras ejecutar el cuaderno de estrategias, debería ver resultados similares a los siguientes:

```Output
PLAY [localhost] 

TASK [Gathering Facts] 
ok: [localhost]

TASK [Get facts of existing App service plan] 
 [WARNING]: Azure API profile latest does not define an entry for WebSiteManagementClient

ok: [localhost]

TASK [debug] 
ok: [localhost] => {
    "facts.appserviceplans[0].sku": {
        "capacity": 1,
        "family": "S",
        "name": "S1",
        "size": "S1",
        "tier": "Standard"
    }
}

TASK [Scale up the App service plan] 
changed: [localhost]

TASK [Get facts] 
ok: [localhost]

TASK [debug] 
ok: [localhost] => {
    "facts.appserviceplans[0].sku": {
        "capacity": 3,
        "family": "S",
        "name": "S2",
        "size": "S2",
        "tier": "Standard"
    }
}

PLAY RECAP 
localhost                  : ok=6    changed=1    unreachable=0    failed=0 
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"] 
> [Ansible en Azure](/azure/ansible/)