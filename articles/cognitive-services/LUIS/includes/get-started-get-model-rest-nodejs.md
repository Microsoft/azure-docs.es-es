---
title: Obtención de un modelo con una llamada a REST en Node.js
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: diberry
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 06/03/2020
ms.author: diberry
ms.custom: devx-track-javascript
ms.openlocfilehash: d2dd43366068c8b0d5f280a4c5c22ac774824f59
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87407272"
---
[Documentación de referencia](https://westeurope.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c45) | [Ejemplo](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/LUIS/node-model-with-rest/model.js)

## <a name="prerequisites"></a>Requisitos previos

* Lenguaje de programación [Node.js](https://nodejs.org/)
* [Visual Studio Code](https://code.visualstudio.com/)

## <a name="example-utterances-json-file"></a>Archivo JSON de expresiones de ejemplo

[!INCLUDE [Quickstart explanation of example utterance JSON file](get-started-get-model-json-example-utterances.md)]

## <a name="create-the-nodejs-project"></a>Creación del proyecto Node.js

1. Cree una nueva carpeta para almacenar el proyecto de Node. js, como `node-model-with-rest`.

1. Abra un nuevo símbolo del sistema, vaya a la carpeta que ha creado y ejecute el siguiente comando:

    ```console
    npm init
    ```

    Presione Entrar en cada aviso para aceptar los valores predeterminados.

1. Introduzca el siguiente comando para instalar el módulo request-promise:

    ```console
    npm install --save request
    npm install --save request-promise
    npm install --save querystring
    ```

## <a name="change-model-programmatically"></a>Cambio de modelo mediante programación

1. Cree un nuevo archivo llamado `model.js`. Agregue el siguiente código:

    [!code-javascript[Code snippet](~/cognitive-services-quickstart-code/javascript/LUIS/node-model-with-rest/model.js)]

1. Reemplace los valores a partir de `YOUR-` por sus propios valores.

    |Información|Propósito|
    |--|--|
    |`YOUR-APP-ID`| El identificador de la aplicación de LUIS. |
    |`YOUR-AUTHORING-KEY`|La clave de creación de 32 caracteres.|
    |`YOUR-AUTHORING-ENDPOINT`| El punto de conexión de la dirección URL de creación. Por ejemplo, `https://replace-with-your-resource-name.api.cognitive.microsoft.com/`. El nombre del recurso se establece al crear el recurso.|

    Las claves y recursos asignados son visibles en el portal de LUIS, en la sección Administrar de la página **Recursos de Azure**. El identificador de la aplicación está disponible en la misma sección Administrar, en la página **Configuración de la aplicación**.

1. En el símbolo del sistema, escriba el siguiente comando para ejecutar el proyecto:

    ```console
    node model.js
    ```

1. Revise la respuesta de creación:

    ```json
    addUtterance:
    [
      {
        "value": {
          "ExampleId": 1137150691,
          "UtteranceText": "order a pizza"
        },
        "hasError": false
      },
      {
        "value": {
          "ExampleId": 1137150692,
          "UtteranceText": "order a large pepperoni pizza"
        },
        "hasError": false
      },
      {
        "value": {
          "ExampleId": 1137150693,
          "UtteranceText": "i want two large pepperoni pizzas on thin crust"
        },
        "hasError": false
      }
    ]
    train POST:
    {
      "statusId": 9,
      "status": "Queued"
    }
    train GET:
    [
      {
        "modelId": "edb46abf-0000-41ab-beb2-a41a0fe1630f",
        "details": {
          "statusId": 9,
          "status": "Queued",
          "exampleCount": 0
        }
      },
      {
        "modelId": "a5030be2-616c-4648-bf2f-380fa9417d37",
        "details": {
          "statusId": 9,
          "status": "Queued",
          "exampleCount": 0
        }
      },
      {
        "modelId": "3f2b1f31-a3c3-4fbd-8182-e9d9dbc120b9",
        "details": {
          "statusId": 9,
          "status": "Queued",
          "exampleCount": 0
        }
      },
      {
        "modelId": "e4b6704b-1636-474c-9459-fe9ccbeba51c",
        "details": {
          "statusId": 9,
          "status": "Queued",
          "exampleCount": 0
        }
      },
      {
        "modelId": "031d3777-2a00-4a7a-9323-9a3280a30000",
        "details": {
          "statusId": 9,
          "status": "Queued",
          "exampleCount": 0
        }
      },
      {
        "modelId": "9250e7a1-06eb-4413-9432-ae132ed32583",
        "details": {
          "statusId": 9,
          "status": "Queued",
          "exampleCount": 0
        }
      }
    ]
    done
    ```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado con este inicio rápido, elimine la carpeta del proyecto del sistema de archivos.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Procedimientos recomendados para una aplicación](../luis-concept-best-practices.md)