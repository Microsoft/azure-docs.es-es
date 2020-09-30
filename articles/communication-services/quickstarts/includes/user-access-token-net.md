---
title: archivo de inclusión
description: archivo de inclusión
services: azure-communication-services
author: matthewrobertson
manager: nimag
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 08/20/2020
ms.topic: include
ms.custom: include file
ms.author: marobert
ms.openlocfilehash: 12a8bccbf9c177c92160d52f4506618d03c791e6
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945746"
---
## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- La versión más reciente de la [biblioteca cliente de .NET Core](https://dotnet.microsoft.com/download/dotnet-core) para su sistema operativo.
- Un recurso activo de Communication Services y una cadena de conexión. [Cree un recurso de Communication Services](../create-communication-resource.md).

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-c-application"></a>Creación de una aplicación de C#

En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `dotnet new` para crear una nueva aplicación de consola con el nombre `UserAccessTokensQuickstart`. Este comando crea un sencillo proyecto "Hola mundo" de C# con un solo archivo de origen: **Program.cs**.

```console
dotnet new console -o UserAccessTokensQuickstart
```

Cambie el directorio a la carpeta de la aplicación recién creada y use el comando `dotnet build` para compilar la aplicación.

```console
cd UserAccessTokensQuickstart
dotnet build
```

### <a name="install-the-package"></a>Instalar el paquete

Mientras sigue en el directorio de aplicaciones, instale la biblioteca de administración de Azure Communication Services para el paquete de .NET mediante el comando `dotnet add package`.

```console
dotnet add package Azure.Communication.Administration
```

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Desde el directorio del proyecto:

1. Abra el archivo **Program.cs** en un editor de texto.
1. Agregue una directiva `using` para incluir el espacio de nombres `Azure.Communication.Administration`.
1. Actualice la declaración del método `Main` para admitir código asincrónico.

Use el código siguiente para empezar:

```csharp
using System;
using Azure.Communication.Administration;

namespace UserAccessTokensQuickstart
{
    class Program
    {
        static async System.Threading.Tasks.Task Main(string[] args)
        {
            Console.WriteLine("Azure Communication Services - User Access Tokens Quickstart");

            // Quickstart code goes here
        }
    }
}
```

[!INCLUDE [User Access Tokens Object Model](user-access-tokens-object-model.md)]

## <a name="authenticate-the-client"></a>Autenticar el cliente

Inicialice un objeto `CommunicationIdentityClient` con su cadena de conexión. El código siguiente recupera la cadena de conexión para el recurso de una variable de entorno denominada `COMMUNICATION_SERVICES_CONNECTION_STRING`. Obtenga información sobre cómo [administrar la cadena de conexión del recurso](../create-communication-resource.md#store-your-connection-string).

Agregue el código siguiente al método `Main`:

```csharp
// This code demonstrates how to fetch your connection string
// from an environment variable.
string ConnectionString = Environment.GetEnvironmentVariable("COMMUNICATION_SERVICES_CONNECTION_STRING");
var client = new CommunicationIdentityClient(ConnectionString);
```

## <a name="create-a-user"></a>Creación de un usuario

Azure Communication Services mantiene un directorio de identidades ligero. Use el método `createUser` para crear una nueva entrada en el directorio con un `Id` único. Debe mantener una asignación entre los usuarios de la aplicación y las identidades generadas por Communication Services (por ejemplo, se pueden almacenar en la base de datos del servidor de aplicaciones).

```csharp
var userResponse = await client.CreateUserAsync();
var user = userResponse.Value;
Console.WriteLine($"\nCreated a user with ID: {user.Id}");
```

## <a name="issue-user-access-tokens"></a>Emisión de tokens de acceso de usuario

Use el método `issueToken` para emitir un token de acceso para un usuario de Communication Services. Si no proporciona el parámetro `user` opcional, se creará un nuevo usuario y se devolverá con el token.

```csharp
// Issue an access token with the "voip" scope for a new user
var tokenResponse = await client.IssueTokenAsync(user, scopes: new [] { CommunicationTokenScope.VoIP });
var token =  tokenResponse.Value.Token;
var expiresOn = tokenResponse.Value.ExpiresOn;
Console.WriteLine($"\nIssued a token with 'voip' scope that expires at {expiresOn}:");
Console.WriteLine(token);
```

Los tokens de acceso de usuario son credenciales de corta duración que deben volver a emitirse para evitar que los usuarios experimenten interrupciones del servicio. La propiedad de respuesta `expiresOn` indica la duración del token.

## <a name="revoke-user-access-tokens"></a>Revocación de tokens de acceso de usuario

En algunos casos, puede que necesite revocar explícitamente los tokens de acceso de usuario, por ejemplo, si un usuario cambia la contraseña que usa para autenticarse en el servicio. Esta funcionalidad está disponible a través de la biblioteca cliente de administración de Azure Communication Services.

```csharp  
await client.RevokeTokensAsync(user);
Console.WriteLine($"\nSuccessfully revoked all tokens for user with ID: {user.Id}");
```

## <a name="delete-a-user"></a>Eliminar un usuario

La eliminación de una identidad revoca todos los tokens activos e impide que se emitan tokens posteriores para las identidades. También quita todo el contenido conservado asociado al usuario.

```csharp
await client.DeleteUserAsync(user);
Console.WriteLine($"\nDeleted the user with ID: {user.Id}");
```

## <a name="run-the-code"></a>Ejecución del código

Ejecute la aplicación desde el directorio de la aplicación con el comando `dotnet run`.

```console
dotnet run
```
