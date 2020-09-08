---
title: Aspectos internos de Azure SignalR Service
description: Obtenga información sobre los aspectos internos de Azure SignalR Service, la arquitectura, las conexiones y cómo se transmiten los datos.
author: sffamily
ms.service: signalr
ms.topic: conceptual
ms.custom: devx-track-dotnet
ms.date: 11/13/2019
ms.author: zhshang
ms.openlocfilehash: 3fc6971c66d06ae9f25584f5be28b051075bfa49
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88921977"
---
# <a name="azure-signalr-service-internals"></a>Aspectos internos de Azure SignalR Service

Azure SignalR Service se basa en el marco de ASP.NET Core SignalR. También admite SignalR para ASP.NET, ya que vuelve a implementar el protocolo de datos de SignalR para ASP.NET sobre el marco de ASP.NET Core.

Puede migrar fácilmente una aplicación de SignalR para ASP.NET Core local o SignalR para ASP.NET para trabajar con SignalR Service, con el cambio de algunas líneas de código.

En el diagrama siguiente se describe la arquitectura típica cuando se usa SignalR Service con el servidor de aplicaciones.

También se habla de las diferencias de la aplicación de SignalR para ASP.NET Core autohospedada.

![Architecture](./media/signalr-concept-internals/arch.png)

## <a name="server-connections"></a>Conexiones con el servidor

El servidor de aplicaciones de SignalR para ASP.NET Core autohospedado escucha a los clientes y conecta a ellos directamente.

Con SignalR Service, el servidor de aplicaciones ya no acepta conexiones de cliente persistentes. En su lugar:

1. El SDK de Azure SignalR Service expone un punto de conexión `negotiate` para cada centro de conectividad.
1. Este punto de conexión responderá a las solicitudes de negociación del cliente y redirigirá a los clientes a SignalR Service.
1. Finalmente, los clientes se conectarán a SignalR Service.

Para más información, consulte [Client connections](#client-connections) (Conexiones con el cliente).

Una vez iniciado el servidor de aplicaciones, 
- Para SignalR de ASP.NET Core, el SDK de Azure SignalR Service abre cinco conexiones WebSocket por centro de conectividad a SignalR Service. 
- Para SignalR para ASP.NET, el SDK de Azure SignalR Service abre cinco conexiones WebSocket por centro de conectividad a SignalR Service y una por cada conexión de WebSocket de aplicación.

5 conexiones WebSocket es el valor predeterminado que se puede cambiar en la [configuración](https://github.com/Azure/azure-signalr/blob/dev/docs/use-signalr-service.md#connectioncount).

Los mensajes que se envían a los clientes y se reciben de estos se multiplexan en estas conexiones.

Estas conexiones permanecerán conectadas a SignalR Service todo el tiempo. Si una conexión de servidor se desconecta por un problema de red,
- todos los clientes a los que preste servicio esta conexión de servidor se desconectan (para más información al respecto, consulte [Data transmit between client and server](#data-transmit-between-client-and-server) [Transmisión de datos entre el cliente y el servidor]);
- la conexión del servidor inicia la reconexión automáticamente.

## <a name="client-connections"></a>Conexiones de cliente

Cuando use SignalR Service, los clientes se conectan a SignalR Service y no al servidor de aplicaciones.
Son dos los pasos necesarios para establecer conexiones persistentes entre el cliente y SignalR Service.

1. El cliente envía una solicitud de negociación al servidor de aplicaciones. Con el SDK de Azure SignalR Service, el servidor de aplicaciones devuelve una respuesta de redirección con la dirección URL de SignalR Service y el token de acceso.

- Para SignalR para ASP.NET Core, una respuesta de redirección típica es similar a lo siguiente:
    ```
    {
        "url":"https://test.service.signalr.net/client/?hub=chat&...",
        "accessToken":"<a typical JWT token>"
    }
    ```
- Para SignalR para ASP.NET, una respuesta de redirección típica es similar a lo siguiente:
    ```
    {
        "ProtocolVersion":"2.0",
        "RedirectUrl":"https://test.service.signalr.net/aspnetclient",
        "AccessToken":"<a typical JWT token>"
    }
    ```

1. Después de recibir la respuesta de redirección, el cliente usa la nueva dirección URL y el token de acceso para iniciar el proceso normal para conectarse a SignalR Service.

Obtenga más información sobre los [protocolos de transporte](https://github.com/aspnet/SignalR/blob/release/2.2/specs/TransportProtocols.md) de SignalR para ASP.NET Core.

## <a name="data-transmit-between-client-and-server"></a>Transmisión de datos entre el cliente y el servidor

Cuando un cliente se conecta a SignalR Service, el entorno de tiempo de ejecución del servicio busca una conexión de servidor para dar servicio a este cliente.
- Este paso se produce solo una vez y es una asignación unívoca entre las conexiones de cliente y servidor.
- La asignación se mantiene en SignalR Service hasta que el cliente o el servidor se desconecta.

En este momento, el servidor de aplicaciones recibe un evento con información del nuevo cliente. Se crea una conexión lógica al con el cliente en el servidor de aplicaciones. Se establece el canal de datos desde el cliente al servidor de aplicaciones a través de SignalR Service.

SignalR Service transmite datos desde el cliente al servidor de aplicaciones de emparejamiento. Y los datos del servidor de aplicaciones se enviarán a los clientes asignados.

SignalR Service no guarda ni almacena los datos del cliente; todos los datos del cliente recibidos se transmiten a los clientes o servidores de destino en tiempo real.

Como puede ver, Azure SignalR Service es esencialmente una capa de transporte lógico entre el servidor de aplicaciones y los clientes. Todas las conexiones persistentes se descargan en SignalR Service.
El servidor de aplicaciones solo necesita controlar la lógica de negocios en la clase del centro de conectividad, sin preocuparse por las conexiones del cliente.