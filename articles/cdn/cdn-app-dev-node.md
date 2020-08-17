---
title: Introducción al SDK de Azure CDN para Node.js | Microsoft Docs
description: Aprenda a escribir aplicaciones Node.js para administrar Azure CDN.
services: cdn
documentationcenter: nodejs
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: c4bb6a61-de3d-4f0c-9dca-202554c43dfa
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 01/23/2017
ms.author: mazha
ms.custom: devx-track-javascript
ms.openlocfilehash: 96dc7e8d9fb647779fc2188fe36fe9fbced2e53c
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87387496"
---
# <a name="get-started-with-azure-cdn-development"></a>Introducción al desarrollo de Azure CDN
> [!div class="op_single_selector"]
> * [Node.js](cdn-app-dev-node.md)
> * [.NET](cdn-app-dev-net.md)
> 
> 

Puede usar el [SDK de Azure CDN para Node.js](https://www.npmjs.com/package/azure-arm-cdn) para automatizar la creación y la administración de puntos de conexión y perfiles de red CDN.  Este tutorial explica paso a paso la creación de una aplicación de consola Node.js sencilla que muestra algunas de las operaciones disponibles.  No se pretende describir todos los aspectos del SDK de Azure CDN para Node.js en detalle.

Para completar este tutorial, debe tener ya instalado y configurado [Node.js](https://www.nodejs.org) **4.x.x** o superior.  Puede usar cualquier editor de texto que desee para crear la aplicación de Node.js.  Para escribir este tutorial, se ha usado [Visual Studio Code](https://code.visualstudio.com).  

> [!TIP]
> El [proyecto completado en este tutorial](https://code.msdn.microsoft.com/Azure-CDN-SDK-for-Nodejs-c712bc74) está disponible para descargarse en MSDN.
> 
> 

[!INCLUDE [cdn-app-dev-prep](../../includes/cdn-app-dev-prep.md)]

## <a name="create-your-project-and-add-npm-dependencies"></a>Creación del proyecto e incorporación de dependencias NPM
Ahora que hemos creado un grupo de recursos para los perfiles de CDN y concedido permiso a la aplicación de Azure AD para administrar perfiles y puntos de conexión de CDN dentro de ese grupo, podemos comenzar a crear la aplicación.

Cree una carpeta para almacenar la aplicación.  Desde una consola con las herramientas de Node.js en su ruta de acceso actual, establezca su ubicación actual en esta nueva carpeta e inicialice el proyecto ejecutando:

```console
npm init
```

A continuación, se le ofrecerán una serie de preguntas para inicializar el proyecto.  Para **punto de entrada**, este tutorial usa *app.js*.  Puede ver las demás opciones en el ejemplo siguiente.

![Salida de NPM init](./media/cdn-app-dev-node/cdn-npm-init.png)

El proyecto se inicializa ahora con un archivo *packages.json* .  Va a usar algunas bibliotecas de Azure contenidas en paquetes NPM.  Vamos a usar el tiempo de ejecución de cliente de Azure para Node.js (ms-rest-azure) y la biblioteca de cliente de Azure CDN para Node.js (azure-arm-cd).  Vamos a agregar estos elementos al proyecto como dependencias.

```console
npm install --save ms-rest-azure
npm install --save azure-arm-cdn
```

Una vez que los paquetes se han instalado, el archivo *package.json* debe ser similar a este ejemplo (los números de versión pueden variar):

``` json
{
  "name": "cdn_node",
  "version": "1.0.0",
  "description": "Azure CDN Node.js tutorial project",
  "main": "app.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Cam Soper",
  "license": "MIT",
  "dependencies": {
    "azure-arm-cdn": "^0.2.1",
    "ms-rest-azure": "^1.14.4"
  }
}
```

Por último, con el editor de texto, cree un archivo de texto en blanco y guárdelo en la raíz de la carpeta del proyecto como *app.js*.  Ya está todo preparado para empezar a escribir código.

## <a name="requires-constants-authentication-and-structure"></a>"Requieres", constantes, autenticación y estructura
Con *apps.js* abierto en el editor, vamos a escribir la estructura básica del programa.

1. En la parte superior agregue los "requiere" para los paquetes NPM con lo siguiente:
   
    ``` javascript
    var msRestAzure = require('ms-rest-azure');
    var cdnManagementClient = require('azure-arm-cdn');
    ```
2. Necesitamos definir algunas constantes que los métodos van a usar.  Agregue lo siguiente.  Asegúrese de reemplazar los marcadores de posición (incluidos los **&lt;corchetes angulares&gt;** ) con sus propios valores según sea necesario.
   
    ``` javascript
    //Tenant app constants
    const clientId = "<YOUR CLIENT ID>";
    const clientSecret = "<YOUR CLIENT AUTHENTICATION KEY>"; //Only for service principals
    const tenantId = "<YOUR TENANT ID>";
   
    //Application constants
    const subscriptionId = "<YOUR SUBSCRIPTION ID>";
    const resourceGroupName = "CdnConsoleTutorial";
    const resourceLocation = "<YOUR PREFERRED AZURE LOCATION, SUCH AS Central US>";
    ```
3. A continuación, crearemos una instancia del cliente de administración de la red CDN y le asignaremos nuestras credenciales.
   
    ``` javascript
    var credentials = new msRestAzure.ApplicationTokenCredentials(clientId, tenantId, clientSecret);
    var cdnClient = new cdnManagementClient(credentials, subscriptionId);
    ```
   
    Si utiliza la autenticación de usuario individual, estas dos líneas tendrán un aspecto algo distinto.
   
   > [!IMPORTANT]
   > Use este ejemplo de código únicamente si opta por la autenticación de usuario individual en lugar de una entidad de servicio.  Asegúrese de guardar las credenciales de usuario individual y de mantenerlas en secreto.
   > 
   > 
   
    ``` javascript
    var credentials = new msRestAzure.UserTokenCredentials(clientId, 
        tenantId, '<username>', '<password>', '<redirect URI>');
    var cdnClient = new cdnManagementClient(credentials, subscriptionId);
    ```
   
    No olvide reemplazar los elementos en **&lt;corchetes angulares&gt;** por la información correcta.  Para `<redirect URI>`, use el URI de redirección que especificó al registrar la aplicación en Azure AD.
4. Nuestra aplicación de consola Node.js necesita algunos parámetros de línea de comandos.  Vamos a dar por válido que por lo menos un parámetro se pasó.
   
   ```javascript
   //Collect command-line parameters
   var parms = process.argv.slice(2);
   
   //Do we have parameters?
   if(parms == null || parms.length == 0)
   {
       console.log("Not enough parameters!");
       console.log("Valid commands are list, delete, create, and purge.");
       process.exit(1);
   }
   ```
5. Con esto llegamos a la parte principal del programa, donde vamos a derivar a otras funciones basadas en los parámetros que se pasaron.
   
    ```javascript
    switch(parms[0].toLowerCase())
    {
        case "list":
            cdnList();
            break;
   
        case "create":
            cdnCreate();
            break;
   
        case "delete":
            cdnDelete();
            break;
   
        case "purge":
            cdnPurge();
            break;
   
        default:
            console.log("Valid commands are list, delete, create, and purge.");
            process.exit(1);
    }
    ```
6. En varios lugares del programa, es necesario asegurarse de que se ha pasado la cantidad correcta de parámetros, si parece que esto no es así, hay que mostrar ayudas.  Vamos a crear funciones para hacerlo.
   
   ```javascript
   function requireParms(parmCount) {
       if(parms.length < parmCount) {
           usageHelp(parms[0].toLowerCase());
           process.exit(1);
       }
   }
   
   function usageHelp(cmd) {
       console.log("Usage for " + cmd + ":");
       switch(cmd)
       {
           case "list":
               console.log("list profiles");
               console.log("list endpoints <profile name>");
               break;
   
           case "create":
               console.log("create profile <profile name>");
               console.log("create endpoint <profile name> <endpoint name> <origin hostname>");
               break;
   
           case "delete":
               console.log("delete profile <profile name>");
               console.log("delete endpoint <profile name> <endpoint name>");
               break;
   
           case "purge":
               console.log("purge <profile name> <endpoint name> <path>");
               break;
   
           default:
               console.log("Invalid command.");
       }
   }
   ```
7. Por último, las funciones que vamos a usar en el cliente de administración de la red CDN son asincrónicas, por lo que necesitan un método al que llamar cuando hayan terminado.  Vamos a hacer uno que pueda mostrar el resultado desde el cliente de administración de la red CDN (si lo hubiera) y salir del programa sin contratiempos.
   
    ```javascript
    function callback(err, result, request, response) {
        if (err) {
            console.log(err);
            process.exit(1);
        } else {
            console.log((result == null) ? "Done!" : result);
            process.exit(0);
        }
    }
    ```

Ahora que ya hemos escrito la estructura básica del programa, deberíamos crear las funciones a las que se llama en base a los parámetros.

## <a name="list-cdn-profiles-and-endpoints"></a>Lista de perfiles y puntos de conexión de CDN
Comencemos con código para enumera los perfiles y puntos de conexión existentes.  Los comentarios de código proporcionan la sintaxis esperada para saber dónde va cada parámetro.

```javascript
// list profiles
// list endpoints <profile name>
function cdnList(){
    requireParms(2);
    switch(parms[1].toLowerCase())
    {
        case "profiles":
            console.log("Listing profiles...");
            cdnClient.profiles.listByResourceGroup(resourceGroupName, callback);
            break;

        case "endpoints":
            requireParms(3);
            console.log("Listing endpoints...");
            cdnClient.endpoints.listByProfile(parms[2], resourceGroupName, callback);
            break;

        default:
            console.log("Invalid parameter.");
            process.exit(1);
    }
}
```

## <a name="create-cdn-profiles-and-endpoints"></a>Creación de perfiles y puntos de conexión de CDN
A continuación, escribiremos las funciones para crear perfiles y puntos de conexión.

```javascript
function cdnCreate() {
    requireParms(2);
    switch(parms[1].toLowerCase())
    {
        case "profile":
            cdnCreateProfile();
            break;

        case "endpoint":
            cdnCreateEndpoint();
            break;

        default:
            console.log("Invalid parameter.");
            process.exit(1);
    }
}

// create profile <profile name>
function cdnCreateProfile() {
    requireParms(3);
    console.log("Creating profile...");
    var standardCreateParameters = {
        location: resourceLocation,
        sku: {
            name: 'Standard_Verizon'
        }
    };

    cdnClient.profiles.create(parms[2], standardCreateParameters, resourceGroupName, callback);
}

// create endpoint <profile name> <endpoint name> <origin hostname>        
function cdnCreateEndpoint() {
    requireParms(5);
    console.log("Creating endpoint...");
    var endpointProperties = {
        location: resourceLocation,
        origins: [{
            name: parms[4],
            hostName: parms[4]
        }]
    };

    cdnClient.endpoints.create(parms[3], endpointProperties, parms[2], resourceGroupName, callback);
}
```

## <a name="purge-an-endpoint"></a>Purga de un punto de conexión
Suponiendo que se haya creado el punto de conexión, una tarea habitual que podríamos llevar a cabo en el programa es purgar el contenido del punto de conexión.

```javascript
// purge <profile name> <endpoint name> <path>
function cdnPurge() {
    requireParms(4);
    console.log("Purging endpoint...");
    var purgeContentPaths = [ parms[3] ];
    cdnClient.endpoints.purgeContent(parms[2], parms[1], resourceGroupName, purgeContentPaths, callback);
}
```

## <a name="delete-cdn-profiles-and-endpoints"></a>Eliminación de perfiles y puntos de conexión de CDN
La última función que incluiremos elimina puntos de conexión y perfiles.

```javascript
function cdnDelete() {
    requireParms(2);
    switch(parms[1].toLowerCase())
    {
        // delete profile <profile name>
        case "profile":
            requireParms(3);
            console.log("Deleting profile...");
            cdnClient.profiles.deleteIfExists(parms[2], resourceGroupName, callback);
            break;

        // delete endpoint <profile name> <endpoint name>
        case "endpoint":
            requireParms(4);
            console.log("Deleting endpoint...");
            cdnClient.endpoints.deleteIfExists(parms[3], parms[2], resourceGroupName, callback);
            break;

        default:
            console.log("Invalid parameter.");
            process.exit(1);
    }
}
```

## <a name="running-the-program"></a>Ejecución del programa
Ahora ya podemos ejecutar nuestro programa de Node.js usando nuestro depurador preferido o en la consola.

> [!TIP]
> Si usa código de Visual Studio como depurador, tiene que configurar su entorno para pasar los parámetros de línea de comandos.  Visual Studio Code hace esto en el archivo **launch.json**.  Busque una propiedad denominada **args** y agregue una matriz de valores de cadena para los parámetros, de forma que se parezca a esto: `"args": ["list", "profiles"]`.
> 
> 

Vamos a empezar con una lista de nuestros perfiles.

![Listado de perfiles](./media/cdn-app-dev-node/cdn-list-profiles.png)

Recuperamos una matriz vacía.  Esto es normal ya que no tenemos ningún perfil en nuestro grupo de recursos.  Vamos a crear un perfil.

![Creación de perfil](./media/cdn-app-dev-node/cdn-create-profile.png)

Ahora, vamos a agregar un punto de conexión.

![Crear punto de conexión](./media/cdn-app-dev-node/cdn-create-endpoint.png)

Por último, vamos a eliminar el perfil.

![Eliminación de perfil](./media/cdn-app-dev-node/cdn-delete-profile.png)

## <a name="next-steps"></a>Pasos siguientes
Para ver el proyecto de este tutorial terminado, [descargue el ejemplo](https://code.msdn.microsoft.com/Azure-CDN-SDK-for-Nodejs-c712bc74).

Para ver la referencia del SDK de Azure CDN para Node.js, consulte el documento de [referencia](https://azure.github.io/azure-sdk-for-node/azure-arm-cdn/latest/).

Para encontrar más documentación sobre Azure SDK para Node.js, consulte el [material de referencia completo](https://azure.github.io/azure-sdk-for-node/).

Administre sus recursos de red CDN con [PowerShell](cdn-manage-powershell.md).

