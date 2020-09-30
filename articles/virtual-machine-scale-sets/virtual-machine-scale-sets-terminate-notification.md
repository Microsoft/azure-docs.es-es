---
title: Notificación de finalización para instancias de conjunto de escalado de máquinas virtuales de Azure
description: Aprenda a habilitar la notificación de finalización para las instancias de conjunto de escalado de máquinas virtuales de Azure.
author: avirishuv
ms.author: avverma
ms.topic: conceptual
ms.service: virtual-machine-scale-sets
ms.subservice: management
ms.date: 02/26/2020
ms.reviewer: jushiman
ms.custom: avverma
ms.openlocfilehash: d4b31eb59ed0bae2afe408546ece66eacade9ddb
ms.sourcegitcommit: 80b9c8ef63cc75b226db5513ad81368b8ab28a28
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2020
ms.locfileid: "90603839"
---
# <a name="terminate-notification-for-azure-virtual-machine-scale-set-instances"></a>Notificación de finalización para instancias de conjunto de escalado de máquinas virtuales de Azure
Las instancias de conjunto de escalado pueden optar por recibir notificaciones de finalización de instancias y establecer un tiempo de espera de retraso predefinido para la operación de finalización. La notificación de finalización se envía mediante Azure Metadata Service – [Scheduled Events](../virtual-machines/windows/scheduled-events.md), que proporciona notificaciones y retrasa las operaciones importantes, como reinicios o reimplementaciones. La solución agrega otro evento, Terminate, a la lista de Scheduled Events, y el retraso asociado de este evento dependerá del límite del retraso especificado por los usuarios en su configuración del modelo del conjunto de escalado.

Una vez inscritas en la característica, las instancias de conjunto de escalado no tienen que esperar a que expire el tiempo de espera especificado antes de que se elimine la instancia. Después de recibir una notificación de finalización, se puede optar por eliminar la instancia en cualquier momento antes de que expire el tiempo de espera de finalización.

## <a name="enable-terminate-notifications"></a>Habilitar notificaciones de finalización
Hay varias maneras de habilitar las notificaciones de terminación en las instancias de conjunto de escalado, tal como se describe en los siguientes ejemplos.

### <a name="azure-portal"></a>Azure portal

Los pasos siguientes habilitan la notificación de finalización al crear un nuevo conjunto de escalado. 

1. Vaya a **Conjuntos de escalado de máquinas virtuales**.
1. Seleccione **+ Agregar** para crear un nuevo conjunto de escalado.
1. Vaya a la pestaña **Administración** (Administración). 
1. Busque la sección **Finalización de la instancia**.
1. En **Instance termination notification** (Notificación de finalización de la instancia), seleccione **Activado**.
1. En **Retraso de la finalización (minutos)** , establezca el tiempo de espera predeterminado deseado.
1. Cuando haya terminado de crear el nuevo conjunto de escalado, seleccione el botón **Revisar y crear**. 

> [!NOTE]
> No puede establecer las notificaciones de finalización en conjuntos de escalado existentes en Azure Portal

### <a name="rest-api"></a>API DE REST

En el ejemplo siguiente se habilita la notificación de finalización en el modelo de conjunto de escalado.

```
PUT on `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}?api-version=2019-03-01`
```

```json
{
  "properties": {
    "virtualMachineProfile": {
            "scheduledEventsProfile": {
                "terminateNotificationProfile": {
                    "notBeforeTimeout":"PT5M",
                    "enable":true
                }
            }
        }
    }        
}

```

En el bloque anterior se especifica un tiempo de espera de 5 minutos (como se indica en *PT5M*) para cualquier operación de finalización en todas las instancias del conjunto de escalado. El campo *notBeforeTimeout* puede tener cualquier valor entre 5 y 15 minutos en formato ISO 8601. Puede cambiar el tiempo de espera predeterminado de la operación de finalización modificando la propiedad *notBeforeTimeout* en *terminateNotificationProfile* como se ha descrito anteriormente.

Después de habilitar *scheduledEventsProfile* en el modelo del conjunto de escalado y establecer el valor de *notBeforeTimeout*, actualice las instancias individuales al [modelo más reciente](virtual-machine-scale-sets-upgrade-scale-set.md#how-to-bring-vms-up-to-date-with-the-latest-scale-set-model) para que se reflejen los cambios.

> [!NOTE]
>Las notificaciones de finalización de las instancias del conjunto de escalado solo se pueden habilitar con la versión 2019-03-01 o posteriores de la API.

### <a name="azure-powershell"></a>Azure PowerShell
Al crear un nuevo conjunto de escalado, puede habilitar las notificaciones de finalización en el conjunto de escalado mediante el cmdlet [New-AzVmssConfig](/powershell/module/az.compute/new-azvmssconfig).

Este script de ejemplo le guía por la creación de un conjunto de escalado y los recursos asociados mediante el archivo de configuración: [Creación de un conjunto de escalado de máquinas virtuales completo](./scripts/powershell-sample-create-complete-scale-set.md) Puede proporcionar la notificación de finalización de configuración mediante la adición de los parámetros *TerminateScheduledEvents* y *TerminateScheduledEventNotBeforeTimeoutInMinutes* al objeto de configuración para crear el conjunto de escalado. En el ejemplo siguiente se habilita la característica con tiempo de espera de retraso de 10 minutos.

```azurepowershell-interactive
New-AzVmssConfig `
  -Location "VMSSLocation" `
  -SkuCapacity 2 `
  -SkuName "Standard_DS2" `
  -UpgradePolicyMode "Automatic" `
  -TerminateScheduledEvents $true `
  -TerminateScheduledEventNotBeforeTimeoutInMinutes 10
```

Use el cmdlet [Update-AzVmss](/powershell/module/az.compute/update-azvmss) para habilitar las notificaciones de finalización en un conjunto de escalado existente.

```azurepowershell-interactive
Update-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -VMScaleSetName "myScaleSet" `
  -TerminateScheduledEvents $true `
  -TerminateScheduledEventNotBeforeTimeoutInMinutes 15
```
En el ejemplo anterior se habilitan las notificaciones de finalización en un conjunto de escalado existente y se establece un tiempo de espera de 15 minutos para el evento de finalización.

Después de habilitar los eventos programados en el modelo del conjunto de escalado y establecer el valor de tiempo de espera, actualice las instancias individuales al [modelo más reciente](virtual-machine-scale-sets-upgrade-scale-set.md#how-to-bring-vms-up-to-date-with-the-latest-scale-set-model) para que se reflejen los cambios.

### <a name="azure-cli-20"></a>CLI de Azure 2.0

En el ejemplo siguiente se habilita la notificación de finalización al crear un nuevo conjunto de escalado.

```azurecli-interactive
az group create --name <myResourceGroup> --location <VMSSLocation>
az vmss create \
  --resource-group <myResourceGroup> \
  --name <myVMScaleSet> \
  --image UbuntuLTS \
  --admin-username <azureuser> \
  --generate-ssh-keys \
  --terminate-notification-time 10
```

En el ejemplo anterior se crea primero un grupo de recursos y, después, se crea un nuevo conjunto de escalado con las notificaciones de finalización habilitadas para un tiempo de espera predeterminado de 10 minutos.

En el ejemplo siguiente se habilita la notificación de finalización en un conjunto de escalado existente.

```azurecli-interactive
az vmss update \  
  --resource-group <myResourceGroup> \
  --name <myVMScaleSet> \
  --enable-terminate-notification true \
  --terminate-notification-time 10
```

## <a name="get-terminate-notifications"></a>Obtención de notificaciones de finalización

Las notificaciones de finalización se entregan mediante [Scheduled Events](../virtual-machines/windows/scheduled-events.md), que pertenece a Azure Metadata Service. Azure Metadata Service expone información sobre la ejecución de máquinas virtuales mediante un punto de conexión de REST accesible desde la propia máquina virtual. La información se encuentra disponible a través de una dirección IP no enrutable, de modo que no se expone fuera de la máquina virtual.

Scheduled Events se habilita para un conjunto de escalado la primera vez que se realiza una solicitud de eventos. Puede esperar una demora de hasta dos minutos en la respuesta en la primera llamada. Consulte el punto de conexión periódicamente para detectar los próximos eventos de mantenimiento, así como el estado de las actividades de mantenimiento en curso.

Scheduled Events se deshabilitará para el conjunto de escalado si las instancias de este no realizan ninguna solicitud durante 24 horas.

### <a name="endpoint-discovery"></a>Detección de punto de conexión
En el caso de las máquinas virtuales con red virtual habilitada, el servicio de metadatos está disponible desde una dirección IP no enrutable estática, 169.254.169.254.

El punto de conexión completo de la versión más reciente de Scheduled Events es:
> "http://169.254.169.254/metadata/scheduledevents?api-version=2019-01-01"

### <a name="query-response"></a>Respuesta de la consulta
Una respuesta contiene una matriz de eventos programados. Una matriz vacía significa que actualmente no hay eventos programados.

En caso de que haya eventos programados, la respuesta contiene una matriz de eventos. Para un evento "Terminate", la respuesta tendrá el siguiente aspecto:
```
{
    "DocumentIncarnation": {IncarnationID},
    "Events": [
        {
            "EventId": {eventID},
            "EventType": "Terminate",
            "ResourceType": "VirtualMachine",
            "Resources": [{resourceName}],
            "EventStatus": "Scheduled",
            "NotBefore": {timeInUTC},
        }
    ]
}
```
*DocumentIncarnation* es una etiqueta de entidad y proporciona una manera fácil de inspeccionar si la carga de los eventos ha cambiado desde la última consulta.

Para más información sobre cada uno de los campos anteriores, consulte la documentación de Scheduled Events para [Windows](../virtual-machines/windows/scheduled-events.md#event-properties) y [Linux](../virtual-machines/linux/scheduled-events.md#event-properties).

### <a name="respond-to-events"></a>Respuesta a eventos
Una vez que se haya enterado de un evento próximo y que haya completado la lógica para un apagado estable, puede aprobar el evento pendiente mediante una llamada POST a Metadata Service con EventId. La llamada POST indica a Azure que puede continuar con la eliminación de la máquina virtual.

A continuación se muestra el JSON esperado en el cuerpo de la solicitud POST. La solicitud debe contener una lista de StartRequests. Cada StartRequest contiene el EventId para el evento que desea acelerar:
```
{
    "StartRequests" : [
        {
            "EventId": {EventId}
        }
    ]
}
```

Asegúrese de que todas las máquinas virtuales del conjunto de escalado solo aprueban el EventID pertinente para esa máquina virtual. Una máquina virtual puede obtener su propio nombre de máquina virtual [mediante Instance Metadata Service](virtual-machine-scale-sets-instance-ids.md#instance-metadata-vm-name). Este nombre toma la forma "{nombreDelConjuntoDeEscalado}_{ID-instancia}" y se mostrará en la sección "Resources" de la respuesta de consulta descrita anteriormente.

También puede hacer referencia a los scripts de ejemplo para consultar y responder a eventos usando [Python](../virtual-machines/linux/scheduled-events.md#python-sample).

## <a name="tips-and-best-practices"></a>Sugerencias y prácticas recomendadas
-   Notificaciones de finalización solo en operaciones "de eliminación": todas las operaciones de eliminación (eliminación manual o reducción horizontal iniciada mediante escalabilidad automática) generará eventos de finalización si el conjunto de escalado tiene *scheduledEventsProfile* habilitado. Otras operaciones como reiniciar, restablecer la imagen inicial, volver a implementar y detener/desasignar no generan eventos de finalización. Las notificaciones de finalización no se pueden habilitar para máquinas virtuales de prioridad baja.
-   No hay esperas obligatorias para el tiempo de espera: puede iniciar la operación de finalización en cualquier momento después de que se haya recibido el evento y antes de que expire el tiempo especificado en *NotBefore* en la configuración de este.
-   Eliminación obligatoria en tiempo de espera: la versión preliminar no proporciona ninguna funcionalidad para ampliar el valor del tiempo de espera una vez que se ha generado un evento. Una vez que expire el tiempo de espera, se procesará el evento de finalización pendiente y se eliminará la máquina virtual.
-   Valor de tiempo de espera modificable: puede modificar el valor del tiempo de espera en cualquier momento antes de que se elimine una instancia. Para ello, modifique la propiedad *notBeforeTimeout* en el modelo del conjunto de escalado y actualice las instancias de máquina virtual al modelo más reciente.
-   Aprobación de todas las eliminaciones pendientes: si hay una eliminación pendiente en VM_1 que no está aprobada y ha aprobado otro evento de finalización en VM_2, la máquina virtual VM_2 no se eliminará hasta que se apruebe el evento de finalización de VM_1 o se agote el tiempo de espera. Una vez aprobado el evento de finalización para VM_1, se eliminarán tanto VM_1 como VM_2.
-   Aprobación de todas las eliminaciones simultáneas: ampliando el ejemplo anterior, si VM_1 y VM_2 tienen el mismo tiempo especificado en *NotBefore*, se deben aprobar los dos eventos de finalización o no se eliminarán las máquinas virtuales antes de que expire el tiempo de espera.

## <a name="troubleshoot"></a>Solución de problemas
### <a name="failure-to-enable-scheduledeventsprofile"></a>Error al habilitar scheduledEventsProfile
Si recibe un error "BadRequest" con un mensaje de error que indica que "no se pudo encontrar el miembro "scheduledEventsProfile" en el objeto de tipo VirtualMachineProfile", compruebe la versión de API usada para las operaciones del conjunto de escalado. Se requiere Compute API versión **2019-03-01** o superior. 

### <a name="failure-to-get-terminate-events"></a>Error al obtener los eventos de finalización
Si no obtiene ningún evento de **finalización** a través de Scheduled Events, compruebe la versión de API que se usa para obtener los eventos. Se requiere Metadata Service API versión **2019-01-01** u otra posterior para los eventos de finalización.
>"http://169.254.169.254/metadata/scheduledevents?api-version=2019-01-01"

### <a name="getting-terminate-event-with-incorrect-notbefore-time"></a>Obtención del evento de finalización con una hora de NotBefore incorrecta  
Después de habilitar *scheduledEventsProfile* en el modelo del conjunto de escalado y establecer el valor de *notBeforeTimeout*, actualice las instancias individuales al [modelo más reciente](virtual-machine-scale-sets-upgrade-scale-set.md#how-to-bring-vms-up-to-date-with-the-latest-scale-set-model) para que se reflejen los cambios.

## <a name="next-steps"></a>Pasos siguientes
Obtenga información sobre cómo [implementar la aplicación](virtual-machine-scale-sets-deploy-app.md) en conjuntos de escalado de máquinas virtuales.
