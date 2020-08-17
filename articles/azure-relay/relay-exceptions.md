---
title: Excepciones de Azure Relay y cómo resolverlas | Microsoft Docs
description: Lista de excepciones de Azure Relay y las acciones sugeridas que puede seguir para ayudar a resolverlas.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: a644dfe80255c64980400866a5e3d197f75375bd
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87532975"
---
# <a name="azure-relay-exceptions"></a>Excepciones de Azure Relay

En este artículo se enumeran algunas excepciones generadas por las API de Azure Relay. Esta referencia está sujeta a cambios, de modo que compruebe las actualizaciones.

## <a name="exception-categories"></a>Categorías de excepciones

Las API de Relay generan excepciones que podrían pertenecer a las siguientes categorías. También se presentan las acciones sugeridas que puede llevar a cabo para resolver las excepciones.

*   **Error de codificación de usuario**: [System.ArgumentException](/dotnet/api/system.argumentexception?view=netcore-3.1), [System.InvalidOperationException](/dotnet/api/system.invalidoperationexception?view=netcore-3.1), [System.OperationCanceledException](/dotnet/api/system.operationcanceledexception?view=netcore-3.1), [System.Runtime.Serialization.SerializationException](/dotnet/api/system.runtime.serialization.serializationexception?view=netcore-3.1). 

    **Acción general**: intente corregir el código antes de continuar.
*   **Error de instalación o configuración**: [System.UnauthorizedAccessException](/dotnet/api/system.unauthorizedaccessexception?view=netcore-3.1). 

    **Acción general**: revise la configuración. Si es necesario, cambie la configuración.
*   **Excepciones transitorias**: [Microsoft.ServiceBus.Messaging.MessagingException](/dotnet/api/microsoft.servicebus.messaging.messagingexception), [Microsoft.ServiceBus.Messaging.ServerBusyException](/dotnet/api/microsoft.servicebus.messaging.serverbusyexception), [Microsoft.ServiceBus.Messaging.MessagingCommunicationException](/dotnet/api/microsoft.servicebus.messaging.messagingcommunicationexception). 

    **Acción general**: intente realizar de nuevo la operación o informe a los usuarios.
*   **Otras excepciones**: [System.Transactions.TransactionException](/dotnet/api/system.transactions.transactionexception?view=netcore-3.1), [System.TimeoutException](/dotnet/api/system.timeoutexception?view=netcore-3.1). 

    **Acción general**: específica del tipo de excepción. Consulte la tabla en la sección siguiente. 

## <a name="exception-types"></a>Tipos de excepciones

En la tabla siguiente se enumeran los tipos de excepción de mensajería y sus causas. También incluye acciones sugeridas que puede seguir para resolver las excepciones.

| **Tipo de excepción** | **Descripción** | **Acción sugerida** | **Nota sobre el reintento automático o inmediato** |
| --- | --- | --- | --- |
| [Tiempo de espera](/dotnet/api/system.timeoutexception?view=netcore-3.1) |El servidor no respondió a la operación solicitada en el tiempo especificado que está controlado por [OperationTimeout](/dotnet/api/microsoft.servicebus.messaging.messagingfactorysettings.operationtimeout). Puede que el servidor haya completado la operación solicitada. Esto se puede producir debido a un retraso de red o a otros retrasos de infraestructura. |Compruebe la coherencia del estado del sistema y vuelva a intentarlo si es necesario. Consulte [TimeoutException](#timeoutexception). |El reintento podría resultar útil en algunos casos; agregue lógica de reintento al código. |
| [Operación no válida](/dotnet/api/system.invalidoperationexception?view=netcore-3.1) |La operación del usuario solicitada no está permitida en el servidor o servicio. Consulte el mensaje de excepción para obtener detalles. |Compruebe el código y la documentación. Asegúrese de que la operación solicitada sea válida. |El reintento no le será de ayuda. |
| [Operación cancelada](/dotnet/api/system.operationcanceledexception?view=netcore-3.1) |Se realiza un intento para invocar una operación en un objeto que ya se ha cerrado, anulado o eliminado. En raras ocasiones, la transacción de ambiente ya se ha eliminado. |Compruebe el código y asegúrese de que no invoca operaciones de un objeto desechado. |El reintento no le será de ayuda. |
| [Acceso no autorizado](/dotnet/api/system.unauthorizedaccessexception?view=netcore-3.1) |El objeto [TokenProvider](/dotnet/api/microsoft.servicebus.tokenprovider) no pudo adquirir un token, el token no es válido o no contiene las notificaciones necesarias para realizar la operación. |Asegúrese de que el proveedor de tokens se haya creado con los valores correctos. Compruebe la configuración de Access Control Service. |El reintento podría resultar útil en algunos casos; agregue lógica de reintento al código. |
| [Excepción de argumento](/dotnet/api/system.argumentexception?view=netcore-3.1),<br /> [Argumento Null](/dotnet/api/system.argumentnullexception?view=netcore-3.1),<br />[Argumento fuera del intervalo](/dotnet/api/system.argumentoutofrangeexception?view=netcore-3.1) |Se ha producido una o varias de las siguientes situaciones:<br />Uno o varios de los argumentos proporcionados para el método no son válidos.<br /> El identificador URI proporcionado a [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) o [Create](/dotnet/api/microsoft.servicebus.messaging.messagingfactory.create) contiene uno o más segmentos de ruta de acceso.<br />El esquema de URI proporcionado a [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) o [Create](/dotnet/api/microsoft.servicebus.messaging.messagingfactory.create) no es válido. <br />El valor de la propiedad es mayor de 32 KB. |Compruebe el código de llamada y asegúrese de que los argumentos sean correctos. |El reintento no le será de ayuda. |
| [Servidor ocupado](/dotnet/api/microsoft.servicebus.messaging.serverbusyexception) |El servicio no puede procesar la solicitud en este momento. |El cliente puede esperar durante un período de tiempo y volver a intentar realizar la operación. |El cliente puede reintentar después de un intervalo específico. Si el reintento genera otra excepción, compruebe el comportamiento de reintento de esa excepción. |
| [Cuota superada](/dotnet/api/microsoft.servicebus.messaging.quotaexceededexception) |La entidad de mensajería alcanzó su tamaño máximo permitido. |Cree espacio en la entidad recibiendo mensajes de esta o de sus subcolas. Consulte [QuotaExceededException](#quotaexceededexception). |El reintento podría resultar útil si los mensajes se eliminan mientras este se lleva a cabo. |
| [Tamaño de mensaje superado](/dotnet/api/microsoft.servicebus.messaging.messagesizeexceededexception) |Una carga de mensaje supera el límite de 256 KB. Tenga en cuenta que el límite de 256 KB es el tamaño total del mensaje. El tamaño total del mensaje puede incluir propiedades del sistema y cualquier sobrecarga .NET. |Reduzca el tamaño de la carga del mensaje y vuelva a intentar la operación. |El reintento no le será de ayuda. |

## <a name="quotaexceededexception"></a>QuotaExceededException

[QuotaExceededException](/dotnet/api/microsoft.servicebus.messaging.quotaexceededexception) indica que se ha superado una cuota de una entidad específica.

Para Relay, esta excepción encapsula [System.ServiceModel.QuotaExceededException](/dotnet/api/system.servicemodel.quotaexceededexception?view=dotnet-plat-ext-3.1), lo que indica que se ha superado el número máximo de agentes de escucha para este punto de conexión. Esto se indica en el valor **MaximumListenersPerEndpoint** del mensaje de excepción.

## <a name="timeoutexception"></a>TimeoutException
[TimeoutException](/dotnet/api/system.timeoutexception?view=netcore-3.1) indica que la operación iniciada por el usuario está superando el tiempo de espera. 

Compruebe el valor de la propiedad [ServicePointManager.DefaultConnectionLimit](/dotnet/api/system.net.servicepointmanager.defaultconnectionlimit?view=netcore-3.1#System_Net_ServicePointManager_DefaultConnectionLimit). Alcanzar este límite puede provocar una [TimeoutException](/dotnet/api/system.timeoutexception?view=netcore-3.1).

Para Relay, puede que reciba excepciones de tiempo de espera al abrir por primera vez una conexión de remitente de la retransmisión. Hay dos causas comunes de esta excepción:

*   Puede que el valor de [OpenTimeout](/previous-versions/) sea demasiado pequeño (incluso por una fracción de un segundo).
* Es posible que los agentes de escucha de retransmisión locales no respondan (o puede que encuentren problemas relativos a las reglas de firewall que prohíban a los agentes de escucha aceptar nuevas conexiones de cliente) y que el valor de [OpenTimeout](/previous-versions/) sea inferior a unos 20 segundos.

Ejemplo:

```
'System.TimeoutException’: The operation did not complete within the allotted timeout of 00:00:10.
The time allotted to this operation may have been a portion of a longer timeout.
```

### <a name="common-causes"></a>Causas comunes
Hay dos causas comunes de este error:

*   **Configuración incorrecta**
    
    El tiempo de espera de la operación podría ser demasiado pequeño para la condición operativa. El valor predeterminado para el tiempo de espera de la operación en el SDK de cliente es 60 segundos. Compruebe si su código tiene el valor establecido en algo demasiado pequeño. Tenga en cuenta que el uso de CPU y el estado de la red pueden afectar al tiempo necesario para que se complete una operación. Se recomienda no establecer el tiempo de espera de la operación en un valor muy bajo.
*   **Error temporal del servicio**

    En ocasiones, el servicio Relay podría experimentar retrasos en el procesamiento de solicitudes. Esto puede ocurrir, por ejemplo, durante los períodos de mucho tráfico. Si esto ocurre, vuelva a intentar la operación después de un retraso hasta que la operación sea correcta. Si la misma operación aún experimenta errores después de varios intentos, visite el [sitio de estado del servicio de Azure](https://azure.microsoft.com/status/) para ver si hay interrupciones del servicio conocidas.

## <a name="next-steps"></a>Pasos siguientes
* [Preguntas frecuentes sobre Azure Relay](relay-faq.md)
* [Crear un espacio de nombres de Relay](relay-create-namespace-portal.md)
* [Introducción a Azure Relay y .NET](relay-hybrid-connections-dotnet-get-started.md)
* [Introducción a Azure Relay y Node](relay-hybrid-connections-node-get-started.md)
