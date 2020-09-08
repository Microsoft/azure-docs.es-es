---
title: Obtención del modelo con la llamada de REST en C#
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: diberry
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 06/03/2020
ms.author: diberry
ms.custom: devx-track-csharp
ms.openlocfilehash: 6c03e403db7157a0b009278f0e343799e179e34c
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88931407"
---
[Documentación de referencia](https://westeurope.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c45) | [Ejemplo](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/LanguageUnderstanding/csharp-model-with-rest/Program.cs)

## <a name="prerequisites"></a>Requisitos previos

* [.NET Core 3.1](https://dotnet.microsoft.com/download)
* [Visual Studio Code](https://code.visualstudio.com/)

## <a name="example-utterances-json-file"></a>Archivo JSON de expresiones de ejemplo

[!INCLUDE [Quickstart explanation of example utterance JSON file](get-started-get-model-json-example-utterances.md)]

## <a name="create-pizza-app"></a>Creación de una aplicación de pizza

[!INCLUDE [Create pizza app](get-started-get-model-create-pizza-app.md)]

## <a name="change-model-programmatically"></a>Cambio de modelo mediante programación

1. Cree una aplicación de consola con C# como lenguaje de destino, con `csharp-model-with-rest` como nombre del proyecto y de la carpeta.

    ```console
    dotnet new console -lang C# -n csharp-model-with-rest
    ```

1. Cambie al directorio `csharp-model-with-rest` que ha creado e instale las dependencias requeridas con estos comandos:

    ```console
    cd csharp-model-with-rest
    dotnet add package System.Net.Http
    dotnet add package JsonFormatterPlus
    ```

1. Sobrescriba Program.cs con el código siguiente:

    [!code-csharp[Code snippet](~/cognitive-services-quickstart-code/dotnet/LanguageUnderstanding/csharp-model-with-rest/Program.cs)]

1. Reemplace los valores a partir de `YOUR-` por sus propios valores.

    |Información|Propósito|
    |--|--|
    |`YOUR-APP-ID`| El identificador de la aplicación de LUIS. |
    |`YOUR-AUTHORING-KEY`|La clave de creación de 32 caracteres.|
    |`YOUR-AUTHORING-ENDPOINT`| El punto de conexión de la dirección URL de creación. Por ejemplo, `https://replace-with-your-resource-name.api.cognitive.microsoft.com/`. El nombre del recurso se establece al crear el recurso.|

    Las claves y recursos asignados son visibles en el portal de LUIS, en la sección Administrar de la página **Recursos de Azure**. El identificador de la aplicación está disponible en la misma sección Administrar, en la página **Configuración de la aplicación**.

1. Compile la aplicación de la consola.

    ```console
    dotnet build
    ```

1. Ejecución de la aplicación de consola.

    ```console
    dotnet run
    ```

1. Revise la respuesta de creación:

    ```console
    Added utterances.
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
    Sent training request.
    {
        "statusId": 9,
        "status": "Queued"
    }
    Requested training status.
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
    ```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado con este inicio rápido, elimine la carpeta del proyecto del sistema de archivos.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Procedimientos recomendados para una aplicación](../luis-concept-best-practices.md)
