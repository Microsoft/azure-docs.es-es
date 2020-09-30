---
title: 'Guía de inicio rápido: incorporación de llamadas VOIP a una aplicación web mediante Azure Communication Services'
description: En este tutorial, aprenderá a usar la biblioteca de cliente de llamadas de Azure Communication Services para JavaScript
author: ddematheu
ms.author: nimag
ms.date: 08/11/2020
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: d58b4d86936c56a08f27bef59edc1d3cc4ce4617
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90946002"
---
En esta guía de inicio rápido, obtendrá información sobre cómo iniciar una llamada con la biblioteca de cliente de llamadas de Azure Communication Services para JavaScript.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Versiones de [Node.js](https://nodejs.org/), Active LTS y Maintenance LTS (se recomiendan 8.11.1 y 10.14.1).
- Recurso activo de Communication Services. [Creación de un recurso de Communication Services](../../create-communication-resource.md).
- Token de acceso de usuario para crear una instancia del cliente de llamada. Aprenda cómo [crear y administrar token de acceso de usuarios](../../access-tokens.md).

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-nodejs-application"></a>Creación de una aplicación Node.js

Abra la ventana de comandos o de terminal, cree un nuevo directorio para la aplicación y navegue hasta este.

```console
mkdir calling-quickstart && cd calling-quickstart
```

Ejecute `npm init -y` para crear un archivo **package.json** con la configuración predeterminada.

```console
npm init -y
```

### <a name="install-the-package"></a>Instalar el paquete

Use el comando `npm install` para instalar la biblioteca cliente de llamadas de Azure Communication Services para JavaScript.

```console
npm install @azure/communication-common --save
npm install @azure/communication-calling --save
```

La opción `--save` muestra la biblioteca como dependencia en el archivo **package.json**.

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Esta guía de inicio rápido usa webpack para agrupar los recursos de la aplicación. Ejecute el siguiente comando para instalar los paquetes de npm webpack, webpack-cli y webpack-dev-server, y mostrarlos como dependencias de desarrollo en el archivo **package.json**:

```console
npm install webpack webpack-cli webpack-dev-server --save-dev
```

Cree un archivo **index.html** en el directorio raíz del proyecto. Usaremos este archivo para configurar un diseño básico que permitirá al usuario realizar una llamada a un bot de comunicaciones de Azure.

Este es el código:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Communication Client - Calling Sample</title>
  </head>
  <body>
    <h4>Azure Communication Services</h4>
    <h1>Calling Quickstart</h1>
    <input 
      id="callee-id-input"
      type="text"
      placeholder="Who would you like to call?"
      style="margin-bottom:1em; width: 200px;"
    />
    <div>
      <button id="call-button" type="button" disabled="true">
        Start Call
      </button>
      &nbsp;
      <button id="hang-up-button" type="button" disabled="true">
        Hang Up
      </button>
    </div>
    <script src="./bundle.js"></script>
  </body>
</html>
```

Cree un archivo en el directorio raíz del proyecto denominado **client.js** que contendrá la lógica de la aplicación para esta guía de inicio rápido. Agregue el siguiente código para importar el cliente que realiza la llamada y obtener referencias a los elementos DOM para que podamos adjuntar la lógica de negocios. 

```javascript
import { CallClient, CallAgent } from "@azure/communication-calling";
import { AzureCommunicationUserCredential } from '@azure/communication-common';

let call;
const calleeInput = document.getElementById("callee-id-input");
const callButton = document.getElementById("call-button");
const hangUpButton = document.getElementById("hang-up-button");

// quickstart code goes here
```

## <a name="object-model"></a>Modelo de objetos

Las clases e interfaces siguientes controlan algunas de las características principales de la biblioteca cliente de llamadas de Azure Communication Services:

| Nombre                             | Descripción                                                                                                                                 |
| ---------------------------------| ------------------------------------------------------------------------------------------------------------------------------------------- |
| CallClient                       | CallClient es el punto de entrada principal a la biblioteca cliente de llamadas.                                                                       |
| CallAgent                        | CallAgent se usa para iniciar y administrar llamadas.                                                                                            |
| AzureCommunicationUserCredential | La clase AzureCommunicationUserCredential implementa la interfaz CommunicationUserCredential interface que se usa para crear una instancia de CallAgent. |


## <a name="authenticate-the-client"></a>Autenticar el cliente

Debe reemplazar `<USER_ACCESS_TOKEN>` por un token de acceso de usuario válido para el recurso. Consulte la documentación relativa al [token de acceso de usuario](../../access-tokens.md) si aún no tiene ningún token disponible. Con el `CallClient`, inicialice una instancia de `CallAgent` con un `CommunicationUserCredential` que nos permita realizar y recibir llamadas. Agregue el código siguiente a **client.js**:

```javascript
const callClient = new CallClient();
const tokenCredential = new AzureCommunicationUserCredential("<USER ACCESS TOKEN>");
let callAgent;

callClient.createCallAgent(tokenCredential).then(agent => {
  callAgent = agent;
  callButton.disabled = false;
});
```

## <a name="start-a-call"></a>Iniciar una llamada

Agregue un controlador de eventos para iniciar una llamada cuando se haga clic en el `callButton`:

```javascript
callButton.addEventListener("click", () => {
    // start a call
    const userToCall = calleeInput.value;
    call = callAgent.call(
        [{ communicationUserId: userToCall }],
        {}
    );
    // toggle button states
    hangUpButton.disabled = false;
    callButton.disabled = true;
});
```

## <a name="end-a-call"></a>Finalizar una llamada

Agregue un cliente de escucha de eventos para finalizar la llamada actual cuando se haga clic en el `hangUpButton`:

```javascript
hangUpButton.addEventListener("click", () => {
  // end the current call
  call.hangUp({ forEveryone: true });

  // toggle button states
  hangUpButton.disabled = true;
  callButton.disabled = false;
});
```

La propiedad `forEveryone` finaliza la llamada para todos los participantes.

## <a name="run-the-code"></a>Ejecución del código

Utilice `webpack-dev-server` para compilar y ejecutar la aplicación. Ejecute el siguiente comando para agrupar el host de aplicación en un servidor web local:

```console
npx webpack-dev-server --entry ./client.js --output bundle.js --debug --devtool inline-source-map
```

Abra el explorador web y vaya a http://localhost:8080/. Verá lo siguiente:

:::image type="content" source="../media/javascript/calling-javascript-app.png" alt-text="Captura de pantalla de la aplicación JavaScript completada.":::

Para hacer una llamada de VOIP saliente, proporcione un identificador de usuario en el campo de texto y haga clic en el botón **Iniciar llamada** . La llamada a `8:echo123` le conecta a un bot de eco; esto es excelente como introducción y para comprobar que los dispositivos de audio funcionan.
