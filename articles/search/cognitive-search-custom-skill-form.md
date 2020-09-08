---
title: Aptitud personalizada de Form Recognizer (C#)
titleSuffix: Azure Cognitive Search
description: Aprenda cómo crear una aptitud personalizada de Form Recognizer mediante C# y Visual Studio.
manager: nitinme
author: PatrickFarley
ms.author: pafarley
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 01/21/2020
ms.openlocfilehash: 58f1c2621165a7074c04752832c6560b2fd3e423
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88935439"
---
# <a name="example-create-a-form-recognizer-custom-skill"></a>Ejemplo: Creación de una aptitud personalizada de Form Recognizer

En este ejemplo de un conjunto de aptitudes de Azure Cognitive Search, aprenderá a crear una aptitud personalizada de Form Recognizer mediante C# y Visual Studio. Form Recognizer analiza documentos y extrae los pares de clave/valor y los datos de tabla. Al encapsular Form Recognizer en la [interfaz de aptitudes personalizadas](cognitive-search-custom-skill-interface.md), puede agregar esta funcionalidad como un paso en una canalización de enriquecimiento integral. Después, la canalización puede cargar los documentos y realizar otras transformaciones.

## <a name="prerequisites"></a>Prerrequisitos

- [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/) (cualquier edición).
- Al menos cinco formularios del mismo tipo. Puede usar los datos de ejemplo proporcionados en esta guía.

## <a name="create-a-form-recognizer-resource"></a>Creación de un recurso de Form Recognizer

[!INCLUDE [create resource](../cognitive-services/form-recognizer/includes/create-resource.md)]

## <a name="train-your-model"></a>Entrenamiento de un modelo

Deberá entrenar un modelo de Form Recognizer con los formularios de entrada antes de usar esta aptitud. Siga los [inicios rápidos de cURL](../cognitive-services/form-recognizer/quickstarts/curl-train-extract.md) para aprender a entrenar un modelo. Puede usar los formularios de ejemplo proporcionados en ese inicio rápido o puede usar sus propios datos. Una vez entrenado el modelo, copie su valor de identificador en una ubicación segura.

## <a name="set-up-the-custom-skill"></a>Configuración de la aptitud personalizada

En este tutorial se usa el proyecto [AnalyzeForm](https://github.com/Azure-Samples/azure-search-power-skills/tree/master/Vision/AnalyzeForm) del repositorio de GitHub [Azure Search Power Skills](https://github.com/Azure-Samples/azure-search-power-skills). Clone este repositorio en el equipo local y vaya a **Vision/AnalyzeForm/** para acceder al proyecto. A continuación, abra _AnalyzeForm.csproj_ en Visual Studio. Este proyecto crea un recurso de Azure Functions que cumple la [interfaz de habilidades personalizadas](cognitive-search-custom-skill-interface.md) y se puede usar para el enriquecimiento de Azure Cognitive Search. Toma los documentos de formulario como entradas y genera (como texto) los pares clave/valor que especifique.

En primer lugar, agregue variables de entorno de nivel de proyecto. Busque el proyecto **AnalyzeForm** en el panel izquierdo, haga clic con el botón derecho en él y seleccione **Propiedades**. En la ventana **Propiedades**, haga clic en la pestaña **Depurar** y busque el campo **Variables de entorno**. Haga clic en **Agregar** para agregar las siguientes variables:
* `FORMS_RECOGNIZER_ENDPOINT_URL` con el valor establecido en la dirección URL del punto de conexión.
* `FORMS_RECOGNIZER_API_KEY` con el valor establecido en la clave de suscripción.
* `FORMS_RECOGNIZER_MODEL_ID` con el valor establecido en el identificador del modelo que entrenó.
* `FORMS_RECOGNIZER_RETRY_DELAY` con el valor establecido en 1000. Este valor es el tiempo, en milisegundos, que el programa esperará antes de volver a intentar la consulta.
* `FORMS_RECOGNIZER_MAX_ATTEMPTS` con el valor establecido en 100. Este valor es el número de veces que el programa consultará el servicio al intentar obtener una respuesta correcta.

Después, abra _AnalyzeForm.cs_ y busque la variable `fieldMappings`, que hace referencia al archivo *field-mappings.json*. Este archivo (y la variable que hace referencia a él) define la lista de claves que desea extraer de los formularios y una etiqueta personalizada para cada clave. Por ejemplo, un valor de `{ "Address:", "address" }, { "Invoice For:", "recipient" }` significa que el script solo guardará los valores de los campos `Address:` y `Invoice For:` detectados y etiquetará esos valores con `"address"` y `"recipient"`, respectivamente.

Por último, tenga en cuenta la variable `contentType`. Este script ejecuta el modelo de Form Recognizer dado en documentos remotos a los que se hace referencia mediante la dirección URL, por lo que el tipo de contenido es `application/json`. Si desea analizar los archivos locales incluyendo sus flujos de bytes en las solicitudes HTTP, deberá cambiar `contentType` al [tipo MIME](https://developer.mozilla.org/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Complete_list_of_MIME_types) adecuado para el archivo.

## <a name="test-the-function-from-visual-studio"></a>Pruebe la función desde Visual Studio

Después de editar el proyecto, guárdelo y establezca el proyecto **AnalyzeForm** como el proyecto de inicio en Visual Studio (si aún no se ha establecido). Después, presione **F5** para ejecutar la función en el entorno local. Use un servicio REST como [Postman](https://www.postman.com/) para llamar a la función.

### <a name="http-request"></a>Solicitud HTTP

Realizará la siguiente solicitud para llamar a la función.

```HTTP
POST https://localhost:7071/api/analyze-form
```

### <a name="request-body"></a>Cuerpo de la solicitud

Comience con la plantilla de cuerpo de la solicitud siguiente.

```json
{
    "values": [
        {
            "recordId": "record1",
            "data": { 
                "formUrl": "<your-form-url>",
                "formSasToken": "<your-sas-token>"
            }
        }
    ]
}
```

Aquí tendrá que proporcionar la dirección URL de un formulario que tenga el mismo tipo que los formularios con los que se entrenó. Con fines de prueba, puede usar uno de los formularios de entrenamiento. Si siguió el inicio rápido de cURL, los formularios se ubicarán en una cuenta de almacenamiento de blobs de Azure. Abra el Explorador de Azure Storage, busque un archivo de formulario, haga clic con el botón secundario en él y seleccione **Obtener firma de acceso compartido**. La siguiente ventana de diálogo proporcionará una dirección URL y un token de SAS. Escriba estas cadenas en los campos `"formUrl"` y `"formSasToken"` del cuerpo de la solicitud, respectivamente.

> [!div class="mx-imgBorder"]
> ![Explorador de Azure Storage; un documento PDF seleccionado](media/cognitive-search-skill-form/form-sas.png)

Si desea analizar un documento remoto que no está en el almacenamiento de blobs de Azure, pegue su dirección URL en el campo `"formUrl"` y deje el campo `"formSasToken"` en blanco.

> [!NOTE]
> Cuando la aptitud esté integrada en un conjunto de aptitudes, Cognitive Search proporcionará la dirección URL y el token.

### <a name="response"></a>Response

Debería ver una respuesta similar a la del siguiente ejemplo:

```json
{
    "values": [
        {
            "recordId": "record1",
            "data": {
                "address": "1111 8th st. Bellevue, WA 99501 ",
                "recipient": "Southridge Video 1060 Main St. Atlanta, GA 65024 "
            },
            "errors": null,
            "warnings": null
        }
    ]
}
```

## <a name="publish-the-function-to-azure"></a>Publicar la función en Azure

Cuando esté satisfecho con el comportamiento de la función, puede publicarla.

1. En el **Explorador de soluciones** en Visual Studio, haga clic con el botón derecho en el proyecto y seleccione **Publicar**. Elija **Crear nueva** > **Publicar**.
1. Si todavía no tiene conectado Visual Studio a la cuenta de Azure, seleccione **Agregar una cuenta...**
1. Siga las instrucciones que aparecen en pantalla. Especifique un nombre único para el servicio de aplicaciones, la suscripción de Azure, el grupo de recursos, el plan de hospedaje y la cuenta de almacenamiento que quiera usar. Puede crear un nuevo grupo de recursos, un nuevo plan de hospedaje y una nueva cuenta de almacenamiento si aún no los tiene. Cuando haya finalizado, seleccione **Crear**.
1. Una vez completada la implementación, fíjese en la dirección URL del sitio. Esta dirección URL es la dirección de la aplicación de funciones en Azure. Guárdela en una ubicación temporal.
1. En [Azure Portal](https://portal.azure.com), vaya al grupo de recursos y busque la función `AnalyzeForm` que ha publicado. En la sección **Administrar**, verá las claves de host. Copie la clave de host *predeterminada* y guárdela en una ubicación temporal.

## <a name="connect-to-your-pipeline"></a>Conéctese a la canalización

Para usar esta aptitud en una canalización de Cognitive Search, necesitará agregar una definición de habilidad al conjunto de habilidades. El siguiente bloque JSON es una definición de aptitud de ejemplo (debe actualizar las entradas y salidas para reflejar su escenario concreto y entorno de conjunto de aptitudes). Reemplace `AzureFunctionEndpointUrl` por la dirección URL de la función y `AzureFunctionDefaultHostKey` por su clave de host.

```json
{ 
  "description":"Skillset that invokes the Form Recognizer custom skill",
  "skills":[ 
    "[... your existing skills go here]",
    { 
      "@odata.type":"#Microsoft.Skills.Custom.WebApiSkill",
      "name":"formrecognizer",
      "description":"Extracts fields from a form using a pre-trained form recognition model",
      "uri":"[AzureFunctionEndpointUrl]/api/analyze-form?code=[AzureFunctionDefaultHostKey]",
      "httpMethod":"POST",
      "timeout":"PT30S",
      "context":"/document",
      "batchSize":1,
      "inputs":[ 
        { 
          "name":"formUrl",
          "source":"/document/metadata_storage_path"
        },
        { 
          "name":"formSasToken",
          "source":"/document/metadata_storage_sas_token"
        }
      ],
      "outputs":[ 
        { 
          "name":"address",
          "targetName":"address"
        },
        { 
          "name":"recipient",
          "targetName":"recipient"
        }
      ]
    }
  ]
}
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía, creó una aptitud personalizada a partir del servicio Azure Form Recognizer. Para más información sobre aptitudes personalizadas, consulte los siguientes recursos. 

* [Azure Search Power Skills: un repositorio de aptitudes personalizadas](https://github.com/Azure-Samples/azure-search-power-skills)
* [Incorporación de una aptitud personalizada a una canalización de enriquecimiento de inteligencia artificial](cognitive-search-custom-skill-interface.md)
* [Define a skillset](cognitive-search-defining-skillset.md) (Definición de un conjunto de aptitudes)
* [Creación de un conjunto de aptitudes (REST)](/rest/api/searchservice/create-skillset)
* [Asignación de campos enriquecidos](cognitive-search-output-field-mapping.md)