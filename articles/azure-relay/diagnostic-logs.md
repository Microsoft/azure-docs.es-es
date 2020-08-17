---
title: Registros de diagnóstico para Conexiones híbridas
description: En este artículo encontrará información general de todos los registros de actividad y de diagnóstico que hay disponibles para Azure Relay.
ms.topic: how-to
ms.date: 06/23/2020
ms.openlocfilehash: 980f2f7a737d3f2460c17a84c472cbf56f5eb90f
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87533009"
---
# <a name="enable-diagnostics-logs-for-azure-relay-hybrid-connections"></a>Habilitar registros de diagnóstico para Conexiones híbridas de Azure Relay
Cuando empiece a usar Conexiones híbridas de Azure Relay, es posible que desee supervisar cómo y cuándo se abren y cierran los clientes de escucha y los emisores, y cómo se crean las Conexiones híbridas y se envían los mensajes. En este artículo se proporciona información general de todos los registros de actividad y de diagnóstico que ofrece el servicio Azure Relay. 

Puede ver dos tipos de registros para Azure Relay:

- [Registros de actividad](../azure-monitor/platform/platform-logs-overview.md): estos registros contienen información sobre las operaciones realizadas en el espacio de nombres de Azure Portal o mediante la plantilla de Azure Resource Manager. Estos registros están siempre habilitados. Por ejemplo: "Crear o actualizar el espacio de nombres", "Crear o actualizar la conexión híbrida". 
- [Registros de diagnóstico](../azure-monitor/platform/platform-logs-overview.md): puede configurar registros de diagnóstico para obtener una vista más completa de todo lo que ocurre con las operaciones y las acciones que se llevan a cabo en el espacio de nombres mediante la API o el SDK del lenguaje.

## <a name="view-activity-logs"></a>Visualización de registros de actividad
Para ver los registros de actividad del espacio de nombres de Azure Relay, cambie a la página **Registro de actividad** de Azure Portal.

![Azure Relay: registro de actividad](./media/diagnostic-logs/activity-log.png)

## <a name="enable-diagnostic-logs"></a>Habilitar registros de diagnóstico

> [!NOTE]
> Los registros de diagnóstico solo están disponibles para Conexiones híbridas, no para Windows Communication Foundation (WCF) para Relay.

Para habilitar los registros de diagnóstico, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), vaya a su espacio de nombres de Azure Relay y, en **Supervisión**, seleccione **Configuración de diagnóstico**.
1. En la página **Configuración de diagnóstico**, seleccione **Agregar configuración de diagnóstico**.  

   ![Vínculo "Agregar configuración de diagnóstico"](./media/diagnostic-logs/add-diagnostic-setting.png)

1. Para configurar las opciones de diagnóstico, siga estos pasos:
    1. En el cuadro **Nombre**, escriba un nombre para la configuración de diagnóstico.  
    2. Seleccione **HybridConnectionsEvent** para el tipo de registro. 
    3. Seleccione uno de estos **destinos** para los registros de diagnóstico:  
        1. Si elige **Archivar en una cuenta de almacenamiento**, configure la cuenta de almacenamiento en la que se almacenarán los registros de diagnóstico.  
        2. Si selecciona **Transmitir a un centro de eventos**, configure el centro de eventos al que quiere transmitir los registros de diagnóstico.
        3. Si elige **Enviar a Log Analytics**, tiene que especificar a qué instancia de Log Analytics se enviarán los diagnósticos.  

        ![Ejemplo de configuración de diagnóstico](./media/diagnostic-logs/sample-diagnostic-settings.png)
1. Seleccione **Guardar** en la barra de herramientas para guardar la configuración.

La nueva configuración surte efecto en unos 10 minutos. Los registros aparecen en el destino de archivo configurado, en el panel **Registros de diagnóstico**. Para más información sobre la configuración de diagnóstico, vea la [información general sobre registros de diagnóstico de Azure](../azure-monitor/platform/platform-logs-overview.md).


## <a name="schema-for-hybrid-connections-events"></a>Esquema de eventos de las conexiones híbridas
Las cadenas del archivo JSON de registro de eventos de Conexiones híbridas incluyen los elementos que aparecen en la tabla siguiente:

| Nombre | Descripción |
| ------- | ------- |
| ResourceId | El identificador de recursos de Azure Resource Manager |
| ActivityId | Identificador interno que se usa para identificar la operación especificada. También puede denominarse "TrackingId". |
| Punto de conexión | Dirección del recurso de Azure Relay |
| OperationName | El tipo de operación de Conexiones híbridas que se va a registrar |
| EventTimeString | Marca de tiempo en formato UTC del registro |
| Message | Mensaje detallado del evento |
| Category | Categoría del evento. Actualmente solo hay `HybridConnectionsEvents`. 


## <a name="sample-hybrid-connections-event"></a>Ejemplo de evento de conexiones híbridas
Este es un ejemplo de evento de conexiones híbridas en formato JSON. 

```json
{
    "resourceId": "/SUBSCRIPTIONS/0000000000-0000-0000-0000-0000000000000/RESOURCEGROUPS/MyResourceGroup/PROVIDERS/MICROSOFT.RELAY/NAMESPACES/MyRelayNamespace",
    "ActivityId": "7006a0db-27eb-445c-939b-ce86133014cc",
    "endpoint": "sb://myrelaynamespace.servicebus.windows.net/myhybridconnection/7006a0db-27eb-445c-939b-ce86133014cc_G5",
    "operationName": "Microsoft.Relay/HybridConnections/NewSenderRegistering",
    "EventTimeString": "2020-04-27T20:27:57.3842810Z",
    "message": "A new sender is registering.",
    "category": "HybridConnectionsEvent"
}
```

## <a name="events-and-operations-captured-in-diagnostic-logs"></a>Eventos y operaciones capturados en registros de diagnóstico

| Operación | Descripción | 
| --------- | ----------- | 
| AuthorizationFailed | Error de autorización.|
| InvalidSasToken | Token de SAS no válido. | 
| ListenerAcceptingConnection | El cliente de escucha que acepta la conexión. |
| ListenerAcceptingConnectionTimeout | El cliente de escucha que acepta la conexión ha agotado el tiempo de espera. |
| ListenerAcceptingHttpRequestFailed | Se ha producido un error en el cliente de escucha que acepta la solicitud HTTP debido a una excepción. |
| ListenerAcceptingRequestTimeout | El cliente de escucha que acepta la solicitud ha agotado el tiempo de espera. |  
| ListenerClosingFromExpiredToken | El cliente de escucha se cierra porque el token de seguridad ha expirado. | 
| ListenerRejectedConnection | El cliente de escucha ha rechazado la conexión. |
| ListenerReturningHttpResponse | El cliente de escucha devuelve una respuesta HTTP. |  
| ListenerReturningHttpResponseFailed | El cliente de escucha devuelve una respuesta HTTP con un código de error. | 
 ListenerSentHttpResponse | El servicio Relay ha recibido una respuesta HTTP del cliente de escucha. | 
| ListenerUnregistered | El cliente de escucha no está registrado. | 
| ListenerUnresponsive | El cliente de escucha no responde al devolver una respuesta. | 
| MessageSendingToListener | Se va a enviar el mensaje al cliente de escucha. |
| MessageSentToListener | Se ha enviado el mensaje al cliente de escucha. | 
| NewListenerRegistered | Se ha registrado un nuevo cliente de escucha. |
| NewSenderRegistering | Se va a registrar un nuevo emisor. | 
| ProcessingRequestFailed | Se ha producido un error en el procesamiento de una operación de Conexiones híbridas. | 
| SenderConnectionClosed | Se cierra la conexión del emisor. |
| SenderListenerConnectionEstablished | El remitente y el cliente de escucha han establecido correctamente la conexión. |
| SenderSentHttpRequest | El remitente ha enviado una solicitud HTTP. | 


## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de Azure Relay, consulte:

* [Introducción a Azure Relay](relay-what-is-it.md)
* [Introducción a Conexiones híbridas de Relay](relay-hybrid-connections-dotnet-get-started.md)
