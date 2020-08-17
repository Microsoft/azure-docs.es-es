---
title: Registros de diagnóstico de Azure Service Bus | Microsoft Docs
description: En este artículo encontrará información general de todos los registros operativos y de diagnóstico que hay disponibles para Azure Service Bus.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: f227f5a988ccd51425b4f43e87b4ed4d9af74e1f
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88064441"
---
# <a name="enable-diagnostics-logs-for-service-bus"></a>Habilitar registros de diagnóstico para Service Bus

Cuando empiece a usar el espacio de nombres de Azure Service Bus, puede que quiera supervisar cómo y cuándo se crea, se elimina o se accede al espacio de nombres. En este artículo encontrará información general de todos los registros operativos y de diagnóstico que hay disponibles.

Actualmente, Azure Service Bus admite registros de actividades y registros operativos que capturan *operaciones de administración* realizadas en el espacio de nombres de Azure Service Bus. En concreto, estos registros capturan el tipo de operación, incluida la creación de colas, los recursos usados y el estado de la operación.

## <a name="operational-logs-schema"></a>Esquema de registros operativos

Todos los registros se almacenan en el formato de notación de objetos JavaScript (JSON) en estas dos ubicaciones:

- **AzureActivity**: muestra registros de operaciones y acciones realizadas en el espacio de nombres de Azure Portal o mediante implementaciones de plantillas de Azure Resource Manager.
- **AzureDiagnostics**: muestra registros de operaciones y acciones realizadas en el espacio de nombres mediante la API o mediante clientes de administración en el SDK de lenguaje.

Las cadenas JSON de registros operativos incluyen elementos enumerados en esta tabla:

| Nombre | Descripción |
| ------- | ------- |
| ActivityId | Identificador interno que se usa para identificar la actividad especificada. |
| EventName | Nombre de la operación |
| ResourceId | El identificador de recursos de Azure Resource Manager |
| SubscriptionId | Id. de suscripción |
| EventTimeString | Hora de la operación |
| EventProperties | Propiedades de la operación |
| Status | Estado de la operación |
| Autor de llamada | Autor de la llamada de la operación (Azure Portal o Management Client) |
| Category | OperationalLogs |

Este es un ejemplo de una cadena JSON de registro operativo:

```json
{
  "ActivityId": "6aa994ac-b56e-4292-8448-0767a5657cc7",
  "EventName": "Create Queue",
  "resourceId": "/SUBSCRIPTIONS/1A2109E3-9DA0-455B-B937-E35E36C1163C/RESOURCEGROUPS/DEFAULT-SERVICEBUS-CENTRALUS/PROVIDERS/MICROSOFT.SERVICEBUS/NAMESPACES/SHOEBOXEHNS-CY4001",
  "SubscriptionId": "1a2109e3-9da0-455b-b937-e35e36c1163c",
  "EventTimeString": "9/28/2016 8:40:06 PM +00:00",
  "EventProperties": "{\"SubscriptionId\":\"1a2109e3-9da0-455b-b937-e35e36c1163c\",\"Namespace\":\"shoeboxehns-cy4001\",\"Via\":\"https://shoeboxehns-cy4001.servicebus.windows.net/f8096791adb448579ee83d30e006a13e/?api-version=2016-07\",\"TrackingId\":\"5ee74c9e-72b5-4e98-97c4-08a62e56e221_G1\"}",
  "Status": "Succeeded",
  "Caller": "ServiceBus Client",
  "category": "OperationalLogs"
}
```

## <a name="events-and-operations-captured-in-operational-logs"></a>Eventos y operaciones capturados en registros operativos

Los registros operativos capturan todas las operaciones de administración que se realizan en el espacio de nombres de Azure Service Bus. Las operaciones de datos no se capturan debido al elevado volumen de este tipo de operaciones que se realizan en Azure Service Bus.

> [!NOTE]
> Para que sea más fácil hacer un seguimiento de las operaciones de datos, se recomienda usar el seguimiento del lado cliente.

Estas operaciones de administración se capturan en los registros operativos: 

| Ámbito | Operación|
|-------| -------- |
| Espacio de nombres | <ul> <li> Crear espacio de nombres</li> <li> Actualizar espacio de nombres </li> <li> Eliminar espacio de nombres </li> <li> Actualizar la directiva SharedAccess del espacio de nombres </li> </ul> | 
| Cola | <ul> <li> Crear cola</li> <li> Actualizar cola</li> <li> Eliminar cola </li> <li> Eliminar cola automáticamente (AutoDelete) </li> </ul> | 
| Tema | <ul> <li> Crear tema </li> <li> Actualizar tema </li> <li> Eliminar tema </li> <li> Eliminar tema automáticamente (AutoDelete) </li> </ul> |
| Subscription | <ul> <li> Crear suscripción </li> <li> Actualizar suscripción </li> <li> Eliminar suscripción </li> <li> Eliminar suscripción automáticamente (AutoDelete) </li> </ul> |

> [!NOTE]
> Actualmente, no se realiza un seguimiento de las operaciones de *lectura* en los registros operativos.

## <a name="enable-operational-logs"></a>Habilitar registros operativos

Los registros operativos están deshabilitados de forma predeterminada. Para habilitar los registros de diagnóstico, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), vaya a su espacio de nombres de Azure Service Bus y, en **Supervisión**, seleccione **Configuración de diagnóstico**.

   ![Vínculo "Configuración de diagnóstico"](./media/service-bus-diagnostic-logs/image1.png)

1. En el panel **Configuración de diagnóstico**, seleccione **Agregar configuración de diagnóstico**.  

   ![Vínculo "Agregar configuración de diagnóstico"](./media/service-bus-diagnostic-logs/image2.png)

1. Para configurar las opciones de diagnóstico, siga estos pasos:

   a. En el cuadro **Nombre**, escriba un nombre para la configuración de diagnóstico.  

   b. Seleccione uno de estos destinos para los registros de diagnóstico:  
   - Si elige **Archivar en una cuenta de almacenamiento**, tiene que configurar la cuenta de almacenamiento donde se almacenarán los registros de diagnóstico.  
   - Si selecciona **Transmitir a un centro de eventos**, tiene que configurar el centro de eventos al que quiere transmitir los registros de diagnóstico.
   - Si elige **Enviar a Log Analytics**, tiene que especificar a qué instancia de Log Analytics se enviarán los diagnósticos.  

   c. Active la casilla **OperationalLogs**.

    ![Panel "Configuración de diagnóstico"](./media/service-bus-diagnostic-logs/image3.png)

1. Seleccione **Guardar**.

La nueva configuración surte efecto en unos 10 minutos. Los registros aparecen en el destino de archivo configurado, en el panel **Registros de diagnóstico**.

Para más información sobre la configuración de diagnóstico, vea la [información general sobre registros de diagnóstico de Azure](../azure-monitor/platform/platform-logs-overview.md).

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Service Bus, vea:

* [Introducción a Service Bus](service-bus-messaging-overview.md)
* [Introducción a Service Bus](service-bus-dotnet-get-started-with-queues.md)