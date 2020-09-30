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
ms.openlocfilehash: 77b1e9ab245f668ab81741451a5e032f37bc3625
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945755"
---
## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Versiones de [Node.js](https://nodejs.org/), Active LTS y Maintenance LTS (se recomiendan 8.11.1 y 10.14.1).
- Un recurso activo de Communication Services y una cadena de conexión. [Cree un recurso de Communication Services](../create-communication-resource.md).

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-nodejs-application"></a>Creación de una nueva aplicación Node.js

Abra la ventana de comandos o de terminal, cree un nuevo directorio para la aplicación y navegue hasta este.

```console
mkdir user-tokens-quickstart && cd user-tokens-quickstart
```

Ejecute `npm init -y` para crear un archivo **package.json** con la configuración predeterminada.

```console
npm init -y
```

### <a name="install-the-package"></a>Instalar el paquete

Use el comando `npm install` para instalar la biblioteca cliente de administración de Azure Communication Services para JavaScript.

```console

npm install @azure/communication-administration --save

```

La opción `--save` muestra la biblioteca como una dependencia en el archivo **package.json**.

## <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Desde el directorio del proyecto:

1. Abra un nuevo archivo de texto en el editor de código.
1. Agregue una llamada `require` para cargar el `CommunicationIdentityClient`.
1. Cree la estructura del programa, incluido un control de excepciones básico.

Use el código siguiente para empezar:

```javascript
const { CommunicationIdentityClient } = require('@azure/communication-administration');

const main = async () => {
  console.log("Azure Communication Services - User Access Tokens Quickstart")

  // Quickstart code goes here
};

main().catch((error) => {
  console.log("Encountered and error");
  console.log(error);
})
```

1. Guarde el nuevo archivo como **issue-token.js** en el directorio *user-tokens-quickstart*.

[!INCLUDE [User Access Tokens Object Model](user-access-tokens-object-model.md)]

## <a name="authenticate-the-client"></a>Autenticar el cliente

Cree una instancia de un objeto `CommunicationIdentityClient` con su cadena de conexión. El código siguiente recupera la cadena de conexión para el recurso de una variable de entorno denominada `COMMUNICATION_SERVICES_CONNECTION_STRING`. Obtenga información sobre cómo [administrar la cadena de conexión del recurso](../create-communication-resource.md#store-your-connection-string).

Agregue el código siguiente al método `main`:

```javascript
// This code demonstrates how to fetch your connection string
// from an environment variable.
const connectionString = process.env['COMMUNICATION_SERVICES_CONNECTION_STRING'];

// Instantiate the user token client
const identityClient = new CommunicationIdentityClient(connectionString);
```

## <a name="create-a-user"></a>Creación de un usuario

Azure Communication Services mantiene un directorio de identidades ligero. Use el método `createUser` para crear una nueva entrada en el directorio con un `Id` único. Debe mantener una asignación entre los usuarios de la aplicación y las identidades generadas por Communication Services (por ejemplo, se pueden almacenar en la base de datos del servidor de aplicaciones).

```javascript
let userResponse = await identityClient.createUser();
console.log(`\nCreated a user with ID: ${userResponse.communicationUserId}`);
```

## <a name="issue-user-access-tokens"></a>Emisión de tokens de acceso de usuario

Use el método `issueToken` para emitir un token de acceso para un usuario de Communication Services. Si no proporciona el parámetro `user` opcional, se creará un nuevo usuario y se devolverá con el token.

```javascript
// Issue an access token with the "voip" scope for a new user
let tokenResponse = await identityClient.issueToken(userResponse, ["voip"]);
const { token, expiresOn } = tokenResponse;
console.log(`\nIssued a token with 'voip' scope that expires at ${expiresOn}:`);
console.log(token);
```

Los tokens de acceso de usuario son credenciales de corta duración que deben volver a emitirse para evitar que los usuarios experimenten interrupciones del servicio. La propiedad de respuesta `expiresOn` indica la duración del token.

## <a name="revoke-user-access-tokens"></a>Revocación de tokens de acceso de usuario

En algunos casos, puede que necesite revocar explícitamente los tokens de acceso de usuario, por ejemplo, si un usuario cambia la contraseña que usa para autenticarse en el servicio. Se usa el método `revokeTokens` para invalidar todos los tokens de acceso de un usuario.

```javascript  
await identityClient.revokeTokens(userResponse);
console.log(`\nSuccessfully revoked all tokens for user with Id: ${userResponse.communicationUserId}`);
```

## <a name="delete-a-user"></a>Eliminar un usuario

La eliminación de un usuario revoca todos los tokens activos e impide que se emitan tokens posteriores para las identidades. También quita todo el contenido conservado asociado al usuario.

```javascript
await identityClient.deleteUser(userResponse);
console.log(`\nDeleted the user with Id: ${userResponse.communicationUserId}`);
```

## <a name="run-the-code"></a>Ejecución del código

Desde un símbolo del sistema de la consola, navegue hasta el directorio que contiene el archivo *issue-token.py* y, a continuación, ejecute el comando `node` siguiente para ejecutar la aplicación.

```console
node ./issue-token.js
```
