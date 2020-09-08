---
title: 'Inicio rápido: Biblioteca cliente de Administración de Azure para Node.js'
description: En este inicio rápido, comience con la biblioteca cliente de Administración de Azure para Node.js.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 08/05/2020
ms.author: pafarley
ms.openlocfilehash: fcfefb2b6cd5c23e9b77d7d27413d1ff6d775bfe
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89321626"
---
[Documentación de referencia](https://docs.microsoft.com/javascript/api/@azure/arm-cognitiveservices/?view=azure-node-latest) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/arm-cognitiveservices) | [Paquete (NPM)](https://www.npmjs.com/package/@azure/arm-cognitiveservices) | [Ejemplos](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/arm-cognitiveservices#sample-code)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure válida: [cree una de manera gratuita](https://azure.microsoft.com/free/).
* La versión actual de [Node.js](https://nodejs.org/)

[!INCLUDE [Create a service principal](./create-service-principal.md)]

[!INCLUDE [Create a resource group](./create-resource-group.md)]

## <a name="create-a-new-nodejs-application"></a>Creación de una aplicación Node.js

En una ventana de la consola (como cmd, PowerShell o Bash), cree un directorio para la aplicación y vaya a él. 

```console
mkdir myapp && cd myapp
```

Ejecute el comando `npm init` para crear una aplicación de nodo con un archivo `package.json`. 

```console
npm init
```

Cree un archivo denominado _index.js_ antes de continuar.

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Instale los siguientes paquetes NPM:

```console
npm install @azure/arm-cognitiveservices
npm install @azure/ms-rest-js
npm install @azure/ms-rest-nodeauth
```

el archivo `package.json` de la aplicación se actualizará con las dependencias.

### <a name="import-libraries"></a>Importación de bibliotecas

Abra el script _index.js_ e importe las siguientes bibliotecas.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/azure_management_service/create_delete_resource.js?name=snippet_imports)]

## <a name="authenticate-the-client"></a>Autenticar el cliente

Agregue los campos siguientes a la raíz del script y rellene sus valores, para lo que debe usar la entidad de servicio que creó y la información de su cuenta de Azure.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/azure_management_service/create_delete_resource.js?name=snippet_constants)]

A continuación, agregue la siguiente función `quickstart` para controlar el trabajo principal del programa. El primer bloque de código construye un objeto **CognitiveServicesManagementClient**, para lo que usa las variables de credenciales que especificó anteriormente. Este objeto es necesario para todas las operaciones de Administración de Azure.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/azure_management_service/create_delete_resource.js?name=snippet_main_auth)]

## <a name="call-management-functions"></a>Funciones de administración de llamadas

Agregue el código siguiente al final de la función `quickstart` para enumerar los recursos disponibles, crear un recurso de ejemplo, enumerar los recursos que posee y, después, eliminar el recurso de ejemplo. Estas funciones las definirá en los siguientes pasos.

## <a name="create-a-cognitive-services-resource"></a>Creación de un recurso de Cognitive Services

### <a name="choose-a-service-and-pricing-tier"></a>Elección de un servicio y un plan de tarifa

Al crear un recurso, necesitará conocer el "tipo" de servicio que desea usar y el [plan de tarifa](https://azure.microsoft.com/pricing/details/cognitive-services/) (o la SKU) que desee. Usará esta y otra información como parámetros al crear el recurso. La siguiente función enumera los "tipos" de Cognitive Services disponibles.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/azure_management_service/create_delete_resource.js?name=snippet_list_avail)]

[!INCLUDE [cognitive-services-subscription-types](../../../../includes/cognitive-services-subscription-types.md)]

[!INCLUDE [SKUs and pricing](./sku-pricing.md)]

## <a name="create-a-cognitive-services-resource"></a>Creación de un recurso de Cognitive Services

Para crear un recurso de Cognitive Services y suscribirse a él, use la función **Create**. Esta función agrega un nuevo recurso facturable al grupo de recursos que se pasa. Cuando cree el recurso, deberá conocer el "tipo" de servicio que desea usar, el plan de tarifa (o SKU) y la ubicación de Azure. La siguiente función usa todos estos valores como argumentos y crea un recurso.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/azure_management_service/create_delete_resource.js?name=snippet_create)]

## <a name="view-your-resources"></a>Visualización de los recursos

Para ver todos los recursos de su cuenta de Azure (de todos los grupos de recursos), use la siguiente función:

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/azure_management_service/create_delete_resource.js?name=snippet_list)]

## <a name="delete-a-resource"></a>Eliminar un recurso

La función siguiente elimina el recurso especificado del grupo de recursos dado.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/azure_management_service/create_delete_resource.js?name=snippet_delete)]

## <a name="run-the-application"></a>Ejecución de la aplicación

Agregue el siguiente código al final del script para llamar a la función `quickstart` principal con control de errores.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/azure_management_service/create_delete_resource.js?name=snippet_main)]

Luego, en la ventana de consola, ejecute la aplicación con el comando `node`.

```console
node index.js
```

## <a name="see-also"></a>Consulte también

* [Documentación de referencia del SDK de Administración de Azure](https://docs.microsoft.com/javascript/api/@azure/arm-cognitiveservices/?view=azure-node-latest)
* [¿Qué es Azure Cognitive Services?](../../Welcome.md)
* [Autenticación de solicitudes en Azure Cognitive Services](../../authentication.md)
* [Creación de un recurso mediante Azure Portal](../../cognitive-services-apis-create-account.md)