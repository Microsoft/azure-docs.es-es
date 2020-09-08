---
title: Ciclo de vida y estados de una máquina virtual en Azure
description: Información general del ciclo de vida de una máquina virtual en Azure, incluidas las descripciones de los distintos estados en los que puede estar una máquina virtual en cualquier momento.
services: virtual-machines
author: shandilvarun
ms.service: virtual-machines
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 08/09/2018
ms.author: vashan
ms.openlocfilehash: cd104a31fdba932481889db198ae3cc4998a4466
ms.sourcegitcommit: 64ad2c8effa70506591b88abaa8836d64621e166
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88261895"
---
# <a name="virtual-machines-lifecycle-and-states"></a>Estados y ciclo de vida de las máquinas virtuales

Las máquinas virtuales de Azure pueden tener diferentes estados que se pueden clasificar en estados de *aprovisionamiento* y estados de *energía*. El propósito de este artículo es describir estos estados y resaltar específicamente cuándo se les factura a los clientes por el uso de la instancia. 

## <a name="power-states"></a>Estados de energía

El estado de energía representa el último estado conocido de la máquina virtual.

![Diagrama de estado de energía de la máquina virtual](./media/virtual-machines-common-states-lifecycle/vm-power-states.png)

<br>
La tabla siguiente proporciona una descripción del estado de cada instancia e indica si se factura por uso de instancia o no.

:::row:::
   :::column span="":::

   **State**
   
   :::column-end:::
   :::column span="":::

   **Descripción**

   :::column-end:::
   :::column span="":::

   **Facturación del uso de la instancia**

   :::column-end:::
:::row-end:::

:::row:::
   :::column span="":::

   **Starting** (iniciándose)

   :::column-end:::
   :::column span="":::

   La máquina virtual se está iniciando.

   ```json
   "statuses": [
    {
    "code": "PowerState/starting",
    "level": "Info",
    "displayStatus": "VM starting"
    }
   ]
   ```
   :::column-end:::
   :::column span="":::

   **No facturado**

   :::column-end:::
:::row-end:::

:::row:::
   :::column span="":::

   **Ejecución**

   :::column-end:::
   :::column span="":::

   Estado de funcionamiento normal para una máquina virtual

   ```json
   "statuses": [
    {
    "code": "PowerState/running",
    "level": "Info",
    "displayStatus": "VM running"
    }
  ]
  ```
   :::column-end:::
   :::column span="":::

   **Facturado**

   :::column-end:::
:::row-end:::

:::row:::
   :::column span="":::

   **Stopping** (Deteniéndose)

   :::column-end:::
   :::column span="":::

   Se trata de un estado transitorio. Cuando haya terminado, aparecerá como **Detenido**.

   ```json
   "statuses": [
    {
    "code": "PowerState/stopping",
    "level": "Info",
    "displayStatus": "VM stopping"
    }
   ]
  ```
   :::column-end:::
   :::column span="":::

   **Facturado**

   :::column-end:::
:::row-end:::

:::row:::
   :::column span="":::

   **Stopped**

   :::column-end:::
   :::column span="":::

   Se ha apagado la máquina virtual desde dentro del sistema operativo invitado o está utilizando las API de PowerOff.

   El hardware aún está asignado a la máquina virtual y permanece en el host.

   ```json
   "statuses": [
    {
    "code": "PowerState/stopped",
    "level": "Info",
    "displayStatus": "VM stopped"
    }
   ]
  ```
   :::column-end:::
   :::column span="":::

   **Facturado***

   :::column-end:::
:::row-end:::

:::row:::
   :::column span="":::

   **Desasignando**

   :::column-end:::
   :::column span="":::

   Estado transitorio. Cuando haya terminado, la máquina virtual aparecerá como **Desasignado**.

   ```json
   "statuses": [
    {
    "code": "PowerState/deallocating",
    "level": "Info",
    "displayStatus": "VM deallocating"
    }
   ]
  ```
   :::column-end:::
   :::column span="":::

   **No facturado***

   :::column-end:::
:::row-end:::

:::row:::
   :::column span="":::

   **Desasignado**

   :::column-end:::
   :::column span="":::

   La máquina virtual se ha detenido correctamente y se ha eliminado del host.

   ```json
   "statuses": [
    {
    "code": "PowerState/deallocated",
    "level": "Info",
    "displayStatus": "VM deallocated"
    }
   ]
  ```
   :::column-end:::
   :::column span="":::

   **No facturado**

   :::column-end:::
:::row-end:::


&#42;Algunos recursos de Azure, como los discos y las redes, generan gastos. Las licencias de software en la instancia no cuestan nada.

## <a name="provisioning-states"></a>Estados de aprovisionamiento

Un estado de aprovisionamiento es el estado de una operación de plano de control iniciada por el usuario en la máquina virtual. Estos estados son independientes del estado de energía de una máquina virtual.

- **Crear**: creación de máquinas virtuales.

- **Actualizar**: actualiza el modelo de una máquina virtual existente. Algunos cambios no relacionados con el modelo que se realizan en las máquinas virtuales, como Iniciar o Reiniciar, también se encuadran bajo la opción Actualizar.

- **Eliminar**: eliminación de máquinas virtuales.

- **Desasignar**: es donde se detiene una máquina virtual y se elimina del host. Desasignar una máquina virtual se considera una actualización, por lo que mostrará los estados de aprovisionamiento relacionados con la actualización.



Estos son los estados de operación transitorios una vez que la plataforma ha aceptado una acción iniciada por el usuario:

:::row:::
   :::column span="":::

   **State**
   
   :::column-end:::
   :::column span="2":::

   **Descripción**

   :::column-end:::

:::row-end:::

:::row:::
   :::column span="":::

   **Creating**

   :::column-end:::
   :::column span="2":::

  ```json
   "statuses": [
    {
    "code": "ProvisioningState/creating",
    "level": "Info",
    "displayStatus": "Creating"
    }
   [
   ```
   :::column-end:::

:::row-end:::

:::row:::
   :::column span="":::

   **Actualizando**

   :::column-end:::
   :::column span="2":::

   ```json
   "statuses": [
    {
    "code": "ProvisioningState/updating",
    "level": "Info",
    "displayStatus": "Updating"
    }
   [
   ```
   :::column-end:::

:::row-end:::

:::row:::
   :::column span="":::

   **Eliminando**

   :::column-end:::
   :::column span="2":::

   ```json
   "statuses": [
    {
    "code": "ProvisioningState/deleting",
    "level": "Info",
    "displayStatus": "Deleting"
    }
   [
   ```
   :::column-end:::

:::row-end:::

:::row:::
   :::column span="":::

   **Estados de aprovisionamiento del sistema operativo**
   
   :::column-end:::
   :::column span="2":::

   **Descripción**

   :::column-end:::

:::row-end:::

:::row:::
   :::column span="":::



   :::column-end:::
   :::column span="2":::

   Si se crea una máquina virtual con una imagen de sistema operativo y no con una imagen especializada, se pueden observar los subestados siguientes:

   :::column-end:::

:::row-end:::

:::row:::
   :::column span="":::

   **OSProvisioningInprogress**

   :::column-end:::
   :::column span="2":::

   La máquina virtual está en ejecución y la instalación del sistema operativo invitado está en curso.
 
   ```json
   "statuses": [
    {
    "code": "ProvisioningState/creating/OSProvisioningInprogress",
    "level": "Info",
    "displayStatus": "OS Provisioning In progress"
    }
   [
   ```
   :::column-end:::

:::row-end:::

:::row:::
   :::column span="":::

   **OSProvisioningComplete**

   :::column-end:::
   :::column span="2":::
   
   Estado de corta duración. La máquina virtual pasa rápidamente a **Correcto** a menos que haya que instalar alguna extensión. La instalación de extensiones puede tardar un tiempo.
   
   ```json
   "statuses": [
    {
    "code": "ProvisioningState/creating/OSProvisioningComplete",
    "level": "Info",
    "displayStatus": "OS Provisioning Complete"
    }
   [
   ```
   
   **Nota**: El aprovisionamiento del sistema operativo puede pasar a **Erróneo** si se produce un error del sistema operativo o este no realiza la instalación a tiempo. A los clientes se les facturará por la máquina virtual implementada en la infraestructura.

   :::column-end:::

:::row-end:::

Una vez completada la operación, la máquina virtual pasará a uno de los siguientes estados:

- **Correcto**: se han completado las acciones iniciadas por el usuario.

    ```
  "statuses": [ 
  {
     "code": "ProvisioningState/succeeded",
     "level": "Info",
     "displayStatus": "Provisioning succeeded",
     "time": "time"
  }
  ]
    ```

 

- **Erróneo**: representa una operación con errores. Consulte los códigos de error para obtener más información y posibles soluciones.

    ```
  "statuses": [
    {
      "code": "ProvisioningState/failed/InternalOperationError",
      "level": "Error",
      "displayStatus": "Provisioning failed",
      "message": "Operation abandoned due to internal error. Please try again later.",
      "time": "time"
    }
    ]
    ```



## <a name="vm-instance-view"></a>Vista de instancia de máquina virtual

La API de vista de instancia proporciona información sobre el estado de ejecución de la máquina virtual. Para más información, consulte la documentación de API de [Máquinas virtuales: Instance View](https://docs.microsoft.com/rest/api/compute/virtualmachines/instanceview).

Azure Resource Explorer proporciona una interfaz de usuario sencilla para ver el estado de ejecución de una máquina virtual: [Resource Explorer](https://resources.azure.com/).

Los estados de aprovisionamiento son visibles en las propiedades y la vista de instancia de la máquina virtual. Los estados de energía están disponibles en la vista de instancia de la máquina virtual.

Para recuperar el estado de energía de todas las máquinas virtuales de la suscripción, use [Virtual Machines - List All API](https://docs.microsoft.com/rest/api/compute/virtualmachines/listall) con el parámetro **statusOnly** establecido en *true*.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre la supervisión de la máquina virtual, consulte [Supervisión de máquinas virtuales en Azure](../azure-monitor/insights/monitor-vm-azure.md).