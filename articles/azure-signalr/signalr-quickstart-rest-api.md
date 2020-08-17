---
title: 'Guía de inicio rápido: API de REST del servicio Azure SignalR'
description: Aprenda a usar la API REST con los siguientes ejemplos de Azure SignalR Service. Busque detalles de la especificación de la API REST.
author: sffamily
ms.service: signalr
ms.topic: quickstart
ms.date: 11/13/2019
ms.author: zhshang
ms.openlocfilehash: 707547cb39487111e2e9353de7e82c2409638324
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87045003"
---
# <a name="quickstart-broadcast-real-time-messages-from-console-app"></a>Inicio rápido: Difusión de mensajes en tiempo real desde la aplicación de consola

El servicio Azure SignalR proporciona la [API de REST](https://github.com/Azure/azure-signalr/blob/dev/docs/rest-api.md) para admitir el servidor para escenarios de comunicación del cliente, como la difusión. Puede elegir cualquier lenguaje de programación que puede llamar a la API de REST. Puede publicar mensajes en todos los clientes conectados, en un cliente específico por el nombre o en un grupo de clientes.

En esta guía de inicio rápido, obtendrá información sobre cómo enviar mensajes desde una aplicación de línea de comandos a las aplicaciones cliente conectadas en C#.

## <a name="prerequisites"></a>Prerrequisitos

Esta guía de inicio rápido se puede ejecutar en macOS, Windows o Linux.

* [SDK de .NET Core](https://www.microsoft.com/net/download/core)
* El editor de texto o de código que elija.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en <https://portal.azure.com/> con su cuenta de Azure.

[!INCLUDE [Create instance](includes/signalr-quickstart-create-instance.md)]

## <a name="clone-the-sample-application"></a>Clonación de la aplicación de ejemplo

Mientras se implementa el servicio, pasaremos a preparar el código. Clone la [aplicación de ejemplo de GitHub](https://github.com/aspnet/AzureSignalR-samples.git), establezca la cadena de conexión del servicio SignalR y ejecute la aplicación de forma local.

1. Abra una ventana de terminal de GIT. Cambie a la carpeta donde quiere clonar el proyecto de ejemplo.

1. Ejecute el comando siguiente para clonar el repositorio de ejemplo. Este comando crea una copia de la aplicación de ejemplo en el equipo.

    ```bash
    git clone https://github.com/aspnet/AzureSignalR-samples.git
    ```

## <a name="build-and-run-the-sample"></a>Compilación y ejecución del ejemplo

Este ejemplo es una aplicación de consola que muestra el uso del servicio Azure SignalR. Proporciona dos modos:

- Modo de servidor: use comandos simples para llamar a la API de REST del servicio Azure SignalR.
- Modo de cliente: conéctese al servicio Azure SignalR y reciba mensajes del servidor.

También puede obtener información acerca de cómo generar un token de acceso para autenticarse con el servicio Azure SignalR.

### <a name="build-the-executable-file"></a>Compilación del archivo ejecutable

Se usa macOS osx.10.13-x64 como ejemplo. Puede encontrar una [referencia](https://docs.microsoft.com/dotnet/core/rid-catalog) sobre cómo compilar en otras plataformas.

```bash
cd AzureSignalR-samples/samples/Serverless/

dotnet publish -c Release -r osx.10.13-x64
```

### <a name="start-a-client"></a>Inicio de un cliente

```bash
cd bin/Release/netcoreapp2.1/osx.10.13-x64/

Serverless client <ClientName> -c "<ConnectionString>" -h <HubName>
```

### <a name="start-a-server"></a>Inicio de un servidor

```bash
cd bin/Release/netcoreapp2.1/osx.10.13-x64/

Serverless server -c "<ConnectionString>" -h <HubName>
```

## <a name="run-the-sample-without-publishing"></a>Ejecución del ejemplo sin publicar

También puede ejecutar el comando siguiente para iniciar un servidor o cliente.

```bash
# Start a server
dotnet run -- server -c "<ConnectionString>" -h <HubName>

# Start a client
dotnet run -- client <ClientName> -c "<ConnectionString>" -h <HubName>
```

### <a name="use-user-secrets-to-specify-connection-string"></a>Uso de los secretos de usuario para especificar la cadena de conexión

Puede ejecutar `dotnet user-secrets set Azure:SignalR:ConnectionString "<ConnectionString>"` en el directorio raíz del ejemplo. Después de eso, ya no necesitará la opción `-c "<ConnectionString>"`.

## <a name="usage"></a>Uso

Una vez iniciado el servidor, use el comando para enviar el mensaje:

```
send user <User Id>
send users <User List>
send group <Group Name>
send groups <Group List>
broadcast
```

Puede iniciar a varios clientes con nombres de cliente diferentes.

## <a name="integration-with-third-party-services"></a><a name="usage"> </a> Integración con servicios de terceros

El servicio Azure SignalR permite a los servicios de terceros la integración con el sistema.

### <a name="definition-of-technical-specifications"></a>Definición de las especificaciones técnicas

En la siguiente tabla se muestran todas las versiones de las API REST admitidas hasta la fecha. También hay un archivo de definición disponible para cada versión

Versión | Estado de la API | Puerta | Específico
--- | --- | --- | ---
`1.0-preview` | Disponible | 5002 | [Swagger](https://github.com/Azure/azure-signalr/tree/dev/docs/swagger/v1-preview.json)
`1.0` | Disponible | Estándar | [Swagger](https://github.com/Azure/azure-signalr/tree/dev/docs/swagger/v1.json)

La lista de las API disponibles para cada versión específica está disponible en la siguiente lista.

API | 1.0-preview | 1.0
--- | --- | ---
[Difusión a todos](#broadcast) | **&#x2713;** | **&#x2713;**
[Difusión a un grupo](#broadcast-group) | **&#x2713;** | **&#x2713;**
Difusión a varios grupos | **&#x2713;** (en desuso) | `N / A`
[Envío a un usuario](#send-user) | **&#x2713;** | **&#x2713;**
Envío a algunos usuarios | **&#x2713;** (en desuso) | `N / A`
[Incorporación de un usuario a un grupo](#add-user-to-group) | `N / A` | **&#x2713;**
[Eliminación de un usuario de un grupo](#remove-user-from-group) | `N / A` | **&#x2713;**
[Comprobación de la existencia del usuario](#check-user-existence) | `N / A` | **&#x2713;**
[Eliminación de un usuario de todos los grupos](#remove-user-from-all-groups) | `N / A` | **&#x2713;**
[Envío de una conexión](#send-connection) | `N / A` | **&#x2713;**
[Adición de una conexión a un grupo](#add-connection-to-group) | `N / A` | **&#x2713;**
[Eliminación de una conexión de un grupo](#remove-connection-from-group) | `N / A` | **&#x2713;**
[Cierre de una conexión de cliente](#close-connection) | `N / A` | **&#x2713;**
[Service Health](#service-health) | `N / A` | **&#x2713;**

<a name="broadcast"> </a>
### <a name="broadcast-to-everyone"></a>Difusión a todos

Versión | Método HTTP de API | URL de la solicitud | Cuerpo de la solicitud
--- | --- | --- | ---
`1.0-preview` | `POST` | `https://<instance-name>.service.signalr.net:5002/api/v1-preview/hub/<hub-name>` | `{"target": "<method-name>", "arguments": [...]}`
`1.0` | `POST` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>` | Lo mismo que antes.

<a name="broadcast-group"> </a>
### <a name="broadcast-to-a-group"></a>Difusión a un grupo

Versión | Método HTTP de API | URL de la solicitud | Cuerpo de la solicitud
--- | --- | --- | ---
`1.0-preview` | `POST` | `https://<instance-name>.service.signalr.net:5002/api/v1-preview/hub/<hub-name>/group/<group-name>` | `{"target": "<method-name>", "arguments": [...]}`
`1.0` | `POST` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/groups/<group-name>` | Lo mismo que antes.

<a name="send-user"> </a>
### <a name="sending-to-a-user"></a>Envío a un usuario

Versión | Método HTTP de API | URL de la solicitud | Cuerpo de la solicitud
--- | --- | --- | ---
`1.0-preview` | `POST` | `https://<instance-name>.service.signalr.net:5002/api/v1-preview/hub/<hub-name>/user/<user-id>` | `{"target": "<method-name>", "arguments": [...]}`
`1.0` | `POST` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/users/<user-id>` | Lo mismo que antes.

<a name="add-user-to-group"> </a>
### <a name="adding-a-user-to-a-group"></a>Incorporación de un usuario a un grupo

Versión | Método HTTP de API | URL de la solicitud
--- | --- | ---
`1.0` | `PUT` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/groups/<group-name>/users/<user-id>`

<a name="remove-user-from-group"> </a>
### <a name="removing-a-user-from-a-group"></a>Eliminación de un usuario de un grupo

Versión | Método HTTP de API | URL de la solicitud
--- | --- | ---
`1.0` | `DELETE` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/groups/<group-name>/users/<user-id>`

<a name="check-user-existence"> </a>
### <a name="check-user-existence-in-a-group"></a>Comprobación de la existencia de usuarios en un grupo

Versión de API | Método HTTP de API | URL de la solicitud
---|---|---
`1.0` | `GET` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/users/<user-id>/groups/<group-name>`
`1.0` | `GET` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/groups/<group-name>/users/<user-id>` 

Código de estado de respuesta | Descripción
---|---
`200` | El usuario existe
`404` | El usuario no existe

<a name="remove-user-from-all-groups"> </a>
### <a name="remove-a-user-from-all-groups"></a>Eliminación de un usuario de todos los grupos

Versión de API | Método HTTP de API | URL de la solicitud
---|---|---
`1.0` | `DELETE` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/users/<user-id>/groups`

<a name="send-connection"> </a>
### <a name="send-message-to-a-connection"></a>Envío de un mensaje a una conexión

Versión de API | Método HTTP de API | URL de la solicitud | Cuerpo de la solicitud
---|---|---|---
`1.0` | `POST` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/connections/<connection-id>` | `{ "target":"<method-name>", "arguments":[ ... ] }`

<a name="add-connection-to-group"> </a>
### <a name="add-a-connection-to-a-group"></a>Adición de una conexión a un grupo

Versión de API | Método HTTP de API | URL de la solicitud
---|---|---
`1.0` | `PUT` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/groups/<group-name>/connections/<connection-id>`
`1.0` | `PUT` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/connections/<connection-id>/groups/<group-name>`

<a name="remove-connection-from-group"> </a>
### <a name="remove-a-connection-from-a-group"></a>Eliminación de una conexión de un grupo

Versión de API | Método HTTP de API | URL de la solicitud
---|---|---
`1.0` | `DELETE` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/groups/<group-name>/connections/<connection-id>`
`1.0` | `DELETE` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/connections/<connection-id>/groups/<group-name>`

<a name="close-connection"> </a>
### <a name="close-a-client-connection"></a>Cierre de una conexión de cliente

Versión de API | Método HTTP de API | URL de la solicitud
---|---|---
`1.0` | `DELETE` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/connections/<connection-id>`
`1.0` | `DELETE` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/connections/<connection-id>?reason=<close-reason>`

<a name="service-health"> </a>
### <a name="service-health"></a>Service Health

Versión de API | Método HTTP de API | URL de la solicitud
---|---|---                             
`1.0` | `GET` | `https://<instance-name>.service.signalr.net/api/v1/health`

Código de estado de respuesta | Descripción
---|---
`200` | Servicio en buen estado
`5xx` | Error del servicio

[!INCLUDE [Cleanup](includes/signalr-quickstart-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, aprendió a usar la API de REST para difundir mensajes en tiempo real de SignalR Service a los clientes. A continuación, obtendrá información sobre cómo desarrollar e implementar Azure Functions con enlaces de SignalR Service, que se basa en la API de REST.

> [!div class="nextstepaction"]
> [Desarrollo de Azure Functions con enlaces de Azure SignalR Service](signalr-quickstart-azure-functions-csharp.md)
