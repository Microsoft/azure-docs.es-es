---
title: Preguntas más frecuentes sobre Azure SignalR Service
description: Acceda rápidamente a las preguntas frecuentes sobre Azure SignalR Service, acerca de la solución de problemas y de los escenarios de uso típico.
author: sffamily
ms.service: signalr
ms.topic: overview
ms.custom: devx-track-dotnet
ms.date: 11/13/2019
ms.author: zhshang
ms.openlocfilehash: 9aa510a055cb76b30508cb98a25cd9c919eb117d
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88928316"
---
# <a name="azure-signalr-service-faq"></a>Preguntas más frecuentes sobre Azure SignalR Service

## <a name="is-azure-signalr-service-ready-for-production-use"></a>¿Azure SignalR Service está listo para usarlo en el entorno de producción?

Sí.
Para ver el anuncio de disponibilidad general, consulte [Azure SignalR ya está disponible con carácter general](https://azure.microsoft.com/blog/azure-signalr-service-now-generally-available/). 

[ASP.NET Core SignalR](https://docs.microsoft.com/aspnet/core/signalr/introduction) es totalmente compatible.

La compatibilidad con ASP.NET SignalR todavía está en *versión preliminar pública*. Este es un [ejemplo de código](https://github.com/aspnet/AzureSignalR-samples/tree/master/aspnet-samples/ChatRoom).

## <a name="the-client-connection-closes-with-the-error-message-no-server-available-what-does-it-mean"></a>La conexión de cliente se cierra con el mensaje de error "No hay ningún servidor disponible". ¿Qué significa?

Este error solo se produce cuando los clientes envían mensajes a SignalR Service.

Si no tiene ningún servidor de aplicaciones y solo usa la API REST de SignalR Service, este comportamiento es así **por naturaleza**.
En la arquitectura sin servidor, las conexiones cliente están en modo de **ESCUCHA** y no enviarán ningún mensaje a SignalR Service.
Lea más información en [API REST](./signalr-quickstart-rest-api.md).

Si tiene servidores de aplicaciones, este mensaje de error significa que no hay ningún servidor de aplicaciones conectado a la instancia de SignalR Service.

Las causas posibles son:
- No hay ningún servidor de aplicaciones conectado con SignalR Service. Compruebe los registros del servidor de aplicaciones para ver los posibles errores de conexión. Este caso es poco común en la configuración de alta disponibilidad con más de un servidor de aplicaciones.
- Hay problemas de conectividad con las instancias de SignalR Service. Este problema es transitorio y se recuperará automáticamente.
Si persiste durante más de una hora, [abra una incidencia en GitHub](https://github.com/Azure/azure-signalr/issues/new) o [cree una solicitud de soporte técnico en Azure](https://docs.microsoft.com/azure/azure-portal/supportability/how-to-create-azure-support-request).

## <a name="when-there-are-multiple-application-servers-are-client-messages-sent-to-all-servers-or-just-one-of-them"></a>Cuando hay varios servidores de aplicaciones, ¿los mensajes del cliente se envían a todos los servidores o solo a uno de ellos?

Es una asignación uno a uno entre el cliente y el servidor de aplicaciones. Los mensajes de un cliente siempre se envían al mismo servidor de aplicaciones.

La asignación entre el cliente y el servidor de aplicaciones se mantendrá hasta que el cliente o el servidor de aplicaciones se desconecte.

## <a name="if-one-of-my-application-servers-is-down-how-can-i-find-it-and-get-notified"></a>Si uno de mis servidores de aplicación no funciona, ¿cómo puedo saberlo y recibir notificaciones?

SignalR Service supervisa los latidos de los servidores de aplicaciones.
Si no se reciben latidos durante un período de tiempo determinado, se considera que el servidor de aplicaciones está sin conexión. Se desconectarán todas las conexiones de cliente asignadas a este servidor de aplicaciones.

## <a name="why-does-my-custom-iuseridprovider-throw-exception-when-switching-from-aspnet-core-signalr--sdk-to-azure-signalr-service-sdk"></a>¿Por qué el `IUserIdProvider` personalizado genera una excepción al cambiar del SDK de ASP.NET Core SignalR al SDK de Azure SignalR Service?

El parámetro `HubConnectionContext context` es distinto entre el SDK de ASP.NET Core SignalR y el SDK de Azure SignalR Service cuando se llama a `IUserIdProvider`.

En ASP.NET Core SignalR, `HubConnectionContext context` es el contexto proveniente de la conexión de cliente física con valores válidos para todas las propiedades.

En el SDK de Azure SignalR Service, `HubConnectionContext context` es el contexto proveniente de la conexión de cliente lógica. La conexión de cliente física se conecta a la instancia de SignalR Service, por lo que solo se proporciona un número limitado de propiedades.

Por ahora, solo `HubConnectionContext.GetHttpContext()` y `HubConnectionContext.User` están disponibles para el acceso.
Puede revisar el código de origen [aquí](https://github.com/Azure/azure-signalr/blob/dev/src/Microsoft.Azure.SignalR/HubHost/ServiceHubConnectionContext.cs).

## <a name="can-i-configure-the-transports-available-in-signalr-service-as-configuring-it-on-server-side-with-aspnet-core-signalr-for-example-disable-websocket-transport"></a>¿Puedo configurar los transportes disponibles en SignalR Service como si lo configurará en el lado servidor con ASP.NET Core SignalR? Por ejemplo, ¿puedo deshabilitar el transporte de WebSocket?

No.

Azure SignalR Service proporciona los tres transportes que ASP.NET Core SignalR Service admite de manera predeterminada. No se puede configurar. SignalR Service controlará las conexiones y los transportes para todas las conexiones de cliente.

Puede configurar los transportes del lado cliente como se documenta [aquí](https://docs.microsoft.com/aspnet/core/signalr/configuration?view=aspnetcore-2.1&tabs=dotnet#configure-allowed-transports-2).

## <a name="what-is-the-meaning-of-metrics-like-message-count-or-connection-count-showed-in-azure-portal-which-kind-of-aggregation-type-should-i-choose"></a>¿Qué significan las métricas, como el número de mensajes o el número de conexiones, que se muestran en Azure Portal? ¿Qué tipo de agregación debo elegir?

[Aquí](signalr-concept-messages-and-connections.md) puede encontrar los detalles sobre cómo se calculan estas métricas.

En la hoja de información general de los recursos de Azure SignalR Service, ya hemos elegido el tipo de agregación adecuado. Y si va a la hoja Métricas, puede tomar el tipo de agregación que encontrará [aquí](../azure-monitor/platform/metrics-supported.md#microsoftsignalrservicesignalr) como referencia.

## <a name="what-is-the-meaning-of-service-mode-defaultserverlessclassic-how-can-i-choose"></a>¿Qué significa el modo de servicio `Default`/`Serverless`/`Classic`? ¿Cómo lo elijo?

Modos:
* El modo `Default` *requiere* un servidor de centro de conectividad. En este modo, Azure SignalR enruta el tráfico de cliente a las conexiones del servidor de centro del conectividad conectado. Azure SignalR comprueba si hay un servidor del centro de conectividad conectado. Si no se encuentra un servidor del centro de conectividad conectado, Azure SignalR rechaza las conexiones de cliente entrantes. También puede usar **Management API** en este modo para administrar los clientes conectados directamente mediante Azure SignalR.
* El modo `Serverless` *no*  permite ninguna conexión de servidor, es decir, rechazará todas las conexiones de servidor. Todos los clientes deben estar en modo sin servidor. Los clientes se conectan a Azure SignalR y los usuarios suelen usar tecnologías sin servidor, como **Azure Function** para administrar las lógicas del centro de conectividad. Consulte un [ejemplo sencillo](https://docs.microsoft.com/azure/azure-signalr/signalr-quickstart-azure-functions-javascript?WT.mc_id=signalrquickstart-github-antchu) que usa el modo sin servidor de Azure SignalR.
* El modo `Classic` es un estado mixto. Si un centro de conectividad tiene conexión de servidor, el nuevo cliente se enrutará al servidor del centro de conectividad; de lo contrario, el cliente entrará en modo sin servidor.

  Esto puede provocar un problema, por ejemplo, que todas las conexiones del servidor se pierdan durante un momento y algunos clientes entren en modo sin servidor, en lugar de realizar el enrutamiento al servidor del centro.

Opciones:
1. Si no hay servidor de centro de conectividad, elija `Serverless`.
1. Si todos los centros tienen servidores de centro de conectividad, elija `Default`.
1. Si algunos centros tienen servidores y otros no, elija `Classic`, pero tenga en cuenta que esta opción puede provocar problemas; lo mejor es crear dos instancias, una es `Serverless` y, la otra, `Default`.

## <a name="any-feature-differences-when-using-azure-signalr-for-aspnet-signalr"></a>¿Hay diferencias en cuanto a características cuando se usa Azure SignalR para ASP.NET SignalR?
Si se usa Azure SignalR, algunas API y características de ASP.NET SignalR dejan de admitirse:
- La capacidad de pasar a un estado arbitrario entre los clientes y el centro de conectividad (también llamado `HubState`) no se admite cuando se usa Azure SignalR.
- La clase `PersistentConnection` no se admite cuando se usa Azure SignalR.
- Si se usa Azure SignalR, no se admite el **transporte Forever Frame**.
- Azure SignalR no reproduce los mensajes enviados al cliente cuando el cliente está sin conexión.
- Cuando se usa Azure SignalR, el tráfico para una conexión de cliente siempre se enruta a una instancia del servidor de aplicaciones mientras dure la conexión.

ASP.NET SignalR se admite por motivos de compatibilidad, por lo que no se admiten todas las características nuevas de ASP.NET Core SignalR. Por ejemplo, **MessagePack**, **Streaming**, etc. solo están disponibles para las aplicaciones de ASP.NET Core SignalR.

SignalR Service puede configurarse para distintos modos de servicio: `Classic`/`Default`/`Serverles`s. En esta compatibilidad con ASP.NET, no se admite el modo `Serverless`. Tampoco se admite la API REST del plano de datos.

## <a name="where-do-my-data-reside"></a>¿Dónde se encuentran mis datos?

Azure SignalR Service funciona como un servicio de procesador de datos. No almacenará ningún contenido del cliente y la residencia de datos se promete por diseño. Si usa Azure SignalR Service junto con otros servicios de Azure, como Azure Storage para diagnósticos, [aquí](https://azure.microsoft.com/resources/achieving-compliant-data-residency-and-security-with-azure/) encontrará instrucciones sobre cómo mantener la residencia de los datos en las regiones de Azure.
