---
title: 'Tutorial: Escalabilidad automática de los conjuntos de escalado de máquinas virtuales de Azure con Ansible | Microsoft Docs'
description: Aprenda a usar Ansible para escalar los conjuntos de escalado de máquinas virtuales con la escalabilidad automática de Azure.
keywords: ansible, azure, devops, bash, playbook, scale, autoscale, virtual machine, virtual machine scale set, vmss
ms.topic: tutorial
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: 4f2cd66b7460fc6fe48cb55f45bf4bc309ae054c
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65231269"
---
# <a name="tutorial-autoscale-virtual-machine-scale-sets-in-azure-using-ansible"></a>Tutorial: Escalabilidad automática de los conjuntos de escalado de máquinas virtuales de Azure mediante Ansible

[!INCLUDE [ansible-27-note.md](../../includes/ansible-27-note.md)]

[!INCLUDE [open-source-devops-intro-vmss.md](../../includes/open-source-devops-intro-vmss.md)]

La característica de ajuste automático del número de instancias de máquina virtual se denomina [escalabilidad automática](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview). La ventaja de la escalabilidad automática es que reduce la sobrecarga de administración para supervisar y optimizar el rendimiento de la aplicación. La escalabilidad automática puede configurarse a petición o según una programación definida. Con Ansible, puede especificar las reglas de escalabilidad automática que definen el rendimiento aceptable para una experiencia positiva del cliente.

[!INCLUDE [ansible-tutorial-goals.md](../../includes/ansible-tutorial-goals.md)]

> [!div class="checklist"]
>
> * Definición de un perfil de escalado automático
> * Escalabilidad automática según una programación periódica
> * Escalabilidad automática basada en el rendimiento de la aplicación
> * Recuperación de la información de configuración de escalabilidad automática 
> * Deshabilitación de una configuración de escalabilidad automática

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [ansible-prereqs-cloudshell-use-or-vm-creation2.md](../../includes/ansible-prereqs-cloudshell-use-or-vm-creation2.md)] 
[!INCLUDE [ansible-prereqs-vm-scale-set.md](../../includes/ansible-prereqs-vm-scale-set.md)]

## <a name="autoscale-based-on-a-schedule"></a>Escalado automático según una programación

Para habilitar el escalado automático en un conjunto de escalado, primero debe definir un perfil de escalado automático. Este perfil define la capacidad predeterminada, mínima y máxima del conjunto de escalado. Estos límites le permiten controlar el costo al no crear continuamente instancias de máquina virtual, y equilibrar un rendimiento aceptable con un número mínimo de instancias que permanecen en un evento de reducción horizontal. 

Ansible permite escalar los conjuntos de escalado en una fecha específica o según una programación periódica.

El código del cuaderno de estrategias de esta sección aumenta el número de instancias de máquina virtual a tres a las 10:00 horas todos los lunes.

Guarde el siguiente cuaderno de estrategias como `vmss-auto-scale.yml`:

```yml
---
- hosts: localhost
  vars:
    resource_group: myResourceGroup
    vmss_name: myScaleSet
    name: autoscalesetting
  tasks: 
    - name: Create autoscaling
      azure_rm_autoscale:
         resource_group: "{{ resource_group }}"
         name: "{{ name }}"
         target:
           namespace: "Microsoft.Compute"
           types: "virtualMachineScaleSets"
           name: "{{ vmss_name }}"
         enabled: true
         profiles:
         - count: '3'
           min_count: '3'
           max_count: '3'
           name: Auto created scale condition
           recurrence_timezone: Pacific Standard Time
           recurrence_frequency: Week
           recurrence_days:
              - Monday
           recurrence_mins:
              - '0'
           recurrence_hours:
              - '10'
```

Ejecute el comando de estrategias con el comando `ansible-playbook`:

```bash
ansible-playbook vmss-auto-scale.yml
```

## <a name="autoscale-based-on-performance-data"></a>Escalabilidad automática basada en datos de rendimiento

Si aumenta la demanda de la aplicación, la carga de las instancias de máquina virtual de los conjuntos de escalado aumenta. Si este aumento de la carga es continuado, en lugar de ser algo puntual, puede configurar reglas de escalado automático para aumentar el número de instancias de máquina virtual en el conjunto de escalado. Cuando se crean estas instancias de máquina virtual y se implementan las aplicaciones, el conjunto de escalado empieza a distribuir el tráfico entre ellas mediante el equilibrador de carga. Ansible permite controlar qué métricas supervisar, por ejemplo, el uso de CPU, el uso de disco y el tiempo de carga de la aplicación. Puede reducir y escalar horizontalmente los conjuntos de escalado en función de los umbrales de las métricas de rendimiento, siguiendo una programación periódica o en una fecha determinada. 

El código del cuaderno de estrategias de esta sección comprueba la carga de trabajo de la CPU de los 10 minutos anteriores, a las 18:00 horas todos los lunes. 

En función de las métricas de porcentaje de la CPU, el cuaderno de estrategias realiza alguna de las siguientes acciones:

- Escala horizontalmente el número de instancias de máquina virtual a cuatro.
- Reduce horizontalmente el número de instancias de máquina virtual a una.

Guarde el siguiente cuaderno de estrategias como `vmss-auto-scale-metrics.yml`:

```yml
---
- hosts: localhost
  vars:
    resource_group: myResourceGroup
    vmss_name: myScaleSet
    name: autoscalesetting
  tasks:
  - name: Get facts of the resource group
    azure_rm_resourcegroup_facts:
      name: "{{ resource_group }}"
    register: rg

  - name: Get scale set resource uri
    set_fact:
      vmss_id: "{{ rg.ansible_facts.azure_resourcegroups[0].id }}/providers/Microsoft.Compute/virtualMachineScaleSets/{{ vmss_name }}"
    
  - name: Create autoscaling
    azure_rm_autoscale:
      resource_group: "{{ resource_group }}"
      name: "{{ name }}"
      target: "{{ vmss_id }}"
      enabled: true
      profiles:
      - count: '1'
        max_count: '1'
        min_count: '1'
        name: 'This scale condition is executed when none of the other scale condition(s) match'
        recurrence_days:
          - Monday
        recurrence_frequency: Week
        recurrence_hours:
          - 18
        recurrence_mins:
          - 0
        recurrence_timezone: Pacific Standard Time
      - count: '1'
        min_count: '1'
        max_count: '4'
        name: Auto created scale condition
        recurrence_days:
          - Monday
        recurrence_frequency: Week
        recurrence_hours:
          - 18
        recurrence_mins:
          - 0
        recurrence_timezone: Pacific Standard Time
        rules:
          - cooldown: 5
            direction: Increase
            metric_name: Percentage CPU
            metric_resource_uri: "{{ vmss_id }}"
            operator: GreaterThan
            statistic: Average
            threshold: 70
            time_aggregation: Average
            time_grain: 1
            time_window: 10
            type: ChangeCount
            value: '1'
          - cooldown: 5
            direction: Decrease
            metric_name: Percentage CPU
            metric_resource_uri: "{{ vmss_id }}"
            operator: LessThan
            statistic: Average
            threshold: 30
            time_aggregation: Average
            time_grain: 1
            time_window: 10
            type: ChangeCount
            value: '1'
```

Ejecute el comando de estrategias con el comando `ansible-playbook`:

```bash
ansible-playbook vmss-auto-scale-metrics.yml
```

## <a name="get-autoscale-settings-information"></a>Obtener información de la configuración de escalabilidad automática 

El código del cuaderno de estrategias de esta sección utiliza el módulo `azure_rm_autoscale_facts` para recuperar los detalles de la configuración de escalabilidad automática.

Guarde el siguiente cuaderno de estrategias como `vmss-auto-scale-get-settings.yml`:

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
    name: autoscalesetting
  tasks: 
    - name: Retrieve autoscale settings information
      azure_rm_autoscale_facts:
        resource_group: "{{ resource_group }}"
        name: "{{ name }}"
      register: autoscale_query
    
    - debug:
        var: autoscale_query.autoscales[0]
```

Ejecute el comando de estrategias con el comando `ansible-playbook`:

```bash
ansible-playbook vmss-auto-scale-get-settings.yml
```

## <a name="disable-autoscale-settings"></a>Deshabilitación de la configuración de escalabilidad automática

Hay dos maneras de deshabilitar la configuración de escalabilidad automática. Una consiste en cambiar la clave `enabled` de `true` a `false`. La segunda se basa en eliminar la configuración.

El código del cuaderno de estrategias de esta sección elimina la configuración de escalabilidad automática. 

Guarde el siguiente cuaderno de estrategias como `vmss-auto-scale-delete-setting.yml`:

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
    name: autoscalesetting
  tasks: 
    - name: Delete autoscaling
      azure_rm_autoscale:
         resource_group: "{{ resource_group }}"
         name: "{{ name }}"
         state: absent
```

Ejecute el comando de estrategias con el comando `ansible-playbook`:

```bash
vmss-auto-scale-delete-setting.yml
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"] 
> [Tutorial: Actualización de la imagen personalizada de los conjuntos de escalado de máquinas virtuales de Azure con Ansible](./ansible-vmss-update-image.md)