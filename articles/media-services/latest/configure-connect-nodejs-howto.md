---
title: 'Conexión a la API de Azure Media Services v3: Node.js'
description: En este artículo se muestra cómo conectarse a Media Services v3 API con Node.js.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-javascript
ms.openlocfilehash: 4f6b24b17ac0e12c286a11abe8732bb86e2deb5c
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89297355"
---
# <a name="connect-to-media-services-v3-api---nodejs"></a>Conexión a la API de Media Services v3: Node.js

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

En este artículo se muestra cómo conectar con el SDK de node.js de Azure Media Services v3 con el método de inicio de sesión de la entidad de servicio.

## <a name="prerequisites"></a>Prerrequisitos

- [Instale Node.js](https://nodejs.org/en/download/).
- [Cree una cuenta de Media Services](./create-account-howto.md). Asegúrese de recordar el nombre del grupo de recursos y el nombre de la cuenta de Media Services.

> [!IMPORTANT]
> Revise las [convenciones de nomenclatura](media-services-apis-overview.md#naming-conventions).

## <a name="create-packagejson"></a>Creación de package.json

1. Cree un archivo package.json con el editor que prefiera.
1. Abra el archivo y pegue el siguiente código:

```json
{
  "name": "media-services-node-sample",
  "version": "0.1.0",
  "description": "",
  "main": "./index.js",
  "dependencies": {
    "azure-arm-mediaservices": "^4.1.0",
    "azure-storage": "^2.8.0",
    "ms-rest": "^2.3.3",
    "ms-rest-azure": "^2.5.5"
  }
}
```

Se deben especificar los siguientes paquetes:

|Paquete|Descripción|
|---|---|
|`azure-arm-mediaservices`|SDK de Azure Media Services. <br/>Para asegurarse de que usa el paquete más reciente de Azure Media Services, consulte [NPM install azure-arm-mediaservices](https://www.npmjs.com/package/azure-arm-mediaservices/) (Instalación de azure-arm-mediaservices para NPM).|
|`azure-storage`|SDK de Storage. Se utiliza cuando se cargan los archivos en los recursos.|
|`ms-rest-azure`| Se usa para iniciar sesión.|

Puede ejecutar el comando siguiente para asegurarse de que usa el paquete más reciente:

```
npm install azure-arm-mediaservices
```

## <a name="connect-to-nodejs-client"></a>Conexión con el cliente de Node.js

1. Cree un archivo .js con el editor que prefiera.
1. Abra el archivo y pegue el siguiente código.
1. Establezca los valores en la sección "endpoint config" en los valores que se obtuvo de [Acceso a la API](./access-api-howto.md).

```js
'use strict';

const MediaServices = require('azure-arm-mediaservices');
const msRestAzure = require('ms-rest-azure');
const msRest = require('ms-rest');
const azureStorage = require('azure-storage');

// endpoint config
// make sure your URL values end with '/'
const armAadAudience = "";
const aadEndpoint = "";
const armEndpoint = "";
const subscriptionId = "";
const accountName = "";
const region = "";
const aadClientId = "";
const aadSecret = "";
const aadTenantId = "";
const resourceGroup = "";

let azureMediaServicesClient;

///////////////////////////////////////////
//     Entrypoint for sample script      //
///////////////////////////////////////////

msRestAzure.loginWithServicePrincipalSecret(aadClientId, aadSecret, aadTenantId, {
  environment: {
    activeDirectoryResourceId: armAadAudience,
    resourceManagerEndpointUrl: armEndpoint,
    activeDirectoryEndpointUrl: aadEndpoint
  }
}, async function(err, credentials, subscriptions) {
    if (err) return console.log(err);
    azureMediaServicesClient = new MediaServices(credentials, subscriptionId, armEndpoint, { noRetryPolicy: true });
    
    console.log("connected");

});
```

## <a name="run-your-app"></a>Ejecutar la aplicación

Abra un símbolo del sistema. Busque el directorio del ejemplo y ejecute los comandos siguientes:

```
npm install 
node index.js
```

## <a name="see-also"></a>Consulte también

- [Conceptos de Azure Media Services](concepts-overview.md)
- [Instalación de azure-arm-mediaservices para NPM](https://www.npmjs.com/package/azure-arm-mediaservices/)

## <a name="next-steps"></a>Pasos siguientes

Explore la documentación de [referencia de Node.js](/javascript/api/overview/azure/mediaservices/management) y los [ejemplos](https://github.com/Azure-Samples/media-services-v3-node-tutorials) que muestran cómo usar Media Services API con node.js.
