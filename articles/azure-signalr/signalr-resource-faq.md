---
title: Preguntas más frecuentes sobre Azure SignalR Service
description: Obtenga respuestas a las preguntas frecuentes sobre Azure SignalR Service, incluida la solución de problemas y los escenarios de uso típico.
author: sffamily
ms.service: signalr
ms.topic: overview
ms.custom: devx-track-dotnet
ms.date: 11/13/2019
ms.author: zhshang
ms.openlocfilehash: d5dd765dd9b174ffbfec35b63ad5e55ce84193ad
ms.sourcegitcommit: de2750163a601aae0c28506ba32be067e0068c0c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89489568"
---
# <a name="azure-signalr-service-faq"></a>Preguntas más frecuentes sobre Azure SignalR Service

## <a name="is-azure-signalr-service-ready-for-production-use"></a>¿Azure SignalR Service está listo para usarlo en el entorno de producción?

Sí.
Para ver el anuncio de disponibilidad general, consulte la entrada de blob [Azure SignalR ya está disponible con carácter general](https://azure.microsoft.com/blog/azure-signalr-service-now-generally-available/). 

[ASP.NET Core SignalR](https://docs.microsoft.com/aspnet/core/signalr/introduction) es totalmente compatible.

La compatibilidad con ASP.NET SignalR todavía está en *versión preliminar pública*. [Este es un ejemplo de código](https://github.com/aspnet/AzureSignalR-samples/tree/master/aspnet-samples/ChatRoom).

## <a name="the-client-connection-closes-with-the-error-message-no-server-available-what-does-it-mean"></a>La conexión de cliente se cierra con el mensaje de error "No hay ningún servidor disponible". ¿Qué significa?

Este error solo se produce cuando los clientes envían mensajes a Azure SignalR Service.

Si no tiene ningún servidor de aplicaciones y solo usa la API REST de Azure SignalR Service, este comportamiento es así *intencionadamente*.
En la arquitectura sin servidor, las conexiones cliente están en modo de *escucha* y no enviarán ningún mensaje a SignalR Service.
Consulte [más información sobre la API REST](./signalr-quickstart-rest-api.md).

Si tiene servidores de aplicaciones, este mensaje de error significa que no hay ningún servidor de aplicaciones conectado a la instancia de Azure SignalR Service.

Las causas posibles son:
- No hay ningún servidor de aplicaciones conectado con Azure SignalR Service. Compruebe los registros del servidor de aplicaciones para ver los posibles errores de conexión. Este caso es poco común en la configuración de alta disponibilidad con más de un servidor de aplicaciones.
- Hay problemas de conectividad con las instancias de Azure SignalR Service. Este problema es transitorio y las instancias se recuperarán automáticamente.
Si persiste durante más de una hora, [abra una incidencia en GitHub](https://github.com/Azure/azure-signalr/issues/new) o [cree una solicitud de soporte técnico en Azure](https://docs.microsoft.com/azure/azure-portal/supportability/how-to-create-azure-support-request).

## <a name="when-there-are-multiple-application-servers-are-client-messages-sent-to-all-servers-or-just-one-of-them"></a>Cuando hay varios servidores de aplicaciones, ¿los mensajes del cliente se envían a todos los servidores o solo a uno de ellos?

Hay una asignación uno a uno entre el cliente y el servidor de aplicaciones. Los mensajes de un cliente siempre se envían al mismo servidor de aplicaciones.

La asignación se mantiene hasta que el cliente o el servidor de aplicaciones se desconecta.

## <a name="if-one-of-my-application-servers-is-down-how-can-i-find-it-and-get-notified"></a>Si uno de mis servidores de aplicación no funciona, ¿cómo puedo saberlo y recibir notificaciones?

Azure SignalR Service supervisa los latidos de los servidores de aplicaciones.
Si no se reciben latidos durante un período de tiempo determinado, se considera que el servidor de aplicaciones está sin conexión. Se desconectarán todas las conexiones de cliente asignadas a este servidor de aplicaciones.

## <a name="why-does-my-custom-iuseridprovider-throw-an-exception-when-im-switching-from-aspnet-core-signalr-sdk-to-azure-signalr-service-sdk"></a>¿Por qué el `IUserIdProvider` personalizado genera una excepción al cambiar del SDK de ASP.NET Core SignalR al SDK de Azure SignalR Service?

El parámetro `HubConnectionContext context` es distinto entre el SDK de ASP.NET Core SignalR y el SDK de Azure SignalR Service cuando se llama a `IUserIdProvider`.

En ASP.NET Core SignalR, `HubConnectionContext context` es el contexto proveniente de la conexión de cliente física con valores válidos para todas las propiedades.

En el SDK de Azure SignalR Service, `HubConnectionContext context` es el contexto proveniente de la conexión de cliente lógica. La conexión de cliente física se conecta a la instancia de Azure SignalR Service, por lo que solo se proporciona un número limitado de propiedades.

Por ahora, solo `HubConnectionContext.GetHttpContext()` y `HubConnectionContext.User` están disponibles para el acceso.
Puede [revisar el código de origen](https://github.com/Azure/azure-signalr/blob/dev/src/Microsoft.Azure.SignalR/HubHost/ServiceHubConnectionContext.cs).

## <a name="can-i-configure-the-transports-available-in-azure-signalr-service-on-the-server-side-with-aspnet-core-signalr-for-example-can-i-disable-websocket-transport"></a>¿Puedo configurar los transportes disponibles en Azure SignalR Service en el lado servidor con ASP.NET Core SignalR? Por ejemplo, ¿puedo deshabilitar el transporte de WebSocket?

No.

Azure SignalR Service proporciona los tres transportes que ASP.NET Core SignalR Service admite de manera predeterminada. No se puede configurar. Azure SignalR Service controlará las conexiones y los transportes para todas las conexiones de cliente.

Los transportes del lado cliente se pueden establecer como se documenta en la [configuración de ASP.NET Core SignalR](https://docs.microsoft.com/aspnet/core/signalr/configuration?view=aspnetcore-2.1&tabs=dotnet#configure-allowed-transports-2).

## <a name="what-is-the-meaning-of-metrics-like-message-count-or-connection-count-shown-in-the-azure-portal-which-kind-of-aggregation-type-should-i-choose"></a>¿Qué significan las métricas, como el número de mensajes o el número de conexiones, que se muestran en Azure Portal? ¿Qué tipo de agregación debo elegir?

Puede encontrar detalles sobre cómo calcular estas métricas en [Mensajes y conexiones de Azure SignalR Service](signalr-concept-messages-and-connections.md).

En el panel de información general de los recursos de Azure SignalR Service, ya hemos elegido el tipo de agregación adecuado. Si va al panel de métricas, puede usar el tipo de agregación para [Mensajes y conexiones de Azure SignalR Service](../azure-monitor/platform/metrics-supported.md#microsoftsignalrservicesignalr) como referencia.

## <a name="what-is-the-meaning-of-the-default-serverless-and-classic-service-modes-how-can-i-choose"></a>¿Cuál es el significado de los modos de servicio `Default`, `Serverless` y `Classic`? ¿Cómo lo elijo?

A continuación se muestra información sobre los modos:
* El modo `Default` *requiere* un servidor concentrador. En este modo, Azure SignalR Service enruta el tráfico de cliente a las conexiones del servidor concentrador conectado. Azure SignalR Service comprueba si hay un servidor concentrador conectado. Si el servicio no encuentra un servidor concentrador conectado, rechaza las conexiones de cliente entrantes. También puede usar *Management API* en este modo para administrar los clientes conectados directamente mediante Azure SignalR Service.
* El modo `Serverless` *no* permite ninguna conexión de servidor. Es decir, rechazará todas las conexiones del servidor. Todos los clientes deben estar en modo sin servidor. Los clientes se conectan a Azure SignalR Service y los usuarios suelen usar tecnologías sin servidor, como *Azure Functions* para administrar las lógicas del centro de conectividad. [Consulte un ejemplo sencillo](https://docs.microsoft.com/azure/azure-signalr/signalr-quickstart-azure-functions-javascript?WT.mc_id=signalrquickstart-github-antchu) que usa el modo sin servidor de Azure SignalR Service.
* El modo `Classic` es un estado mixto. Cuando un centro de conectividad tiene una conexión de servidor, el nuevo cliente se enrutará a un servidor concentrador. De lo contrario, el cliente entrará en modo sin servidor. 

  Esto podría causar un problema. Por ejemplo, si todas las conexiones del servidor se pierdan durante un momento, algunos clientes entren en modo sin servidor, en lugar de realizar el enrutamiento al servidor concentrador.

Estas son algunas directrices para elegir un modo:
- Si no hay ningún servidor concentrador, elija `Serverless`.
- Si todos los centros tienen servidores concentradores, elija `Default`.
- Si algunos centros de conectividad tienen servidores concentradores pero otros no, puede elegir `Classic`, pero esto podría causar un problema. La mejor manera es crear dos instancias: una es `Serverless` y la otra es `Default`.

## <a name="are-there-any-feature-differences-in-using-azure-signalr-service-with-aspnet-signalr"></a>¿Hay diferencias de características en el uso de Azure SignalR Service con ASP.NET SignalR?
Cuando se usa Azure SignalR Service, no se admiten algunas API y características de ASP.NET SignalR:
- No se admite la capacidad de pasar a un estado arbitrario entre los clientes y el centro de conectividad (también llamado `HubState`).
- No se admite la clase `PersistentConnection`.
- No se admite el *transporte de Forever Frame*.
- Azure SignalR Service ya no reproduce los mensajes enviados al cliente cuando el cliente está sin conexión.
- Cuando se usa Azure SignalR Service, el tráfico de una conexión de cliente siempre se enruta (también denominado *persistente*) a una instancia del servidor de aplicaciones mientras dure la conexión.

ASP.NET SignalR se admite por motivos de compatibilidad, por lo que no se admiten todas las características nuevas de ASP.NET Core SignalR. Por ejemplo, *MessagePack* y *Streaming* solo están disponibles para las aplicaciones de ASP.NET Core SignalR.

Puede configurar Azure SignalR Service para diferentes modos de servicio: `Classic`, `Default` y `Serverless`. El modo `Serverless` no se admite en ASP.NET. Tampoco se admite la API REST del plano de datos.

## <a name="where-does-my-data-reside"></a>¿Dónde residen mis datos?

Azure SignalR Service funciona como un servicio de procesador de datos. No almacenará ningún contenido del cliente y la residencia de datos se incluye intencionadamente. Si usa Azure SignalR Service junto con otros servicios de Azure, como Azure Storage para diagnósticos, consulte [estas notas del producto](https://azure.microsoft.com/resources/achieving-compliant-data-residency-and-security-with-azure/) para obtener instrucciones sobre cómo mantener la residencia de los datos en las regiones de Azure.
