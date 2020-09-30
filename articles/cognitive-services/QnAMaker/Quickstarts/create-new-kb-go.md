---
title: 'Guía de inicio rápido: Creación una la base de conocimiento en QnA Maker con REST y Go'
description: Esta guía de inicio rápido basada en REST para Go le lleva por la creación de una base de conocimiento de QnA Maker de ejemplo mediante programación, que aparecerá en el panel de Azure de su cuenta de la API de Cognitive Services.
ms.date: 12/16/2019
ROBOTS: NOINDEX,NOFOLLOW
ms.custom: RESTCURL2020FEB27
ms.topic: how-to
ms.openlocfilehash: 38bfada87701533002a2dcf304af573da4c87766
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89259904"
---
# <a name="quickstart-create-a-knowledge-base-in-qna-maker-using-go"></a>Guía de inicio rápido: Creación de una base de conocimiento en QnA Maker mediante Go

Esta guía de inicio rápido describe la creación mediante programación de una base de conocimiento de QnA Maker de ejemplo. QnA Maker extrae automáticamente preguntas y respuestas a partir de contenido semiestructurado, como las preguntas frecuentes, y de [orígenes de datos](../Concepts/knowledge-base.md). El modelo para la base de conocimiento se define en el JSON que se envía en el cuerpo de la solicitud de API.

En esta guía de inicio rápido se llama a las siguientes API de QnA Maker:
* [Creación de una base de conocimiento](https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/knowledgebase/create)
* [Obtención de los detalles de la operación](https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/operations/getdetails)

[Documentación de referencia](https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/knowledgebase) | [Ejemplo de Go](https://github.com/Azure-Samples/cognitive-services-qnamaker-go/blob/master/documentation-samples/quickstarts/create-knowledge-base/create-new-knowledge-base.go)

[!INCLUDE [Custom subdomains notice](../../../../includes/cognitive-services-custom-subdomains-note.md)]

## <a name="prerequisites"></a>Prerequisites

* [Go 1.10.1](https://golang.org/dl/)
* Debe tener un [servicio QnA Maker](../How-To/set-up-qnamaker-service-azure.md). Para recuperar la clave y el punto de conexión (que incluye el nombre del recurso), seleccione **Inicio rápido** en el recurso en Azure Portal.

## <a name="create-a-knowledge-base-go-file"></a>Creación de un archivo de Go de base de conocimiento

Cree un archivo llamado `create-new-knowledge-base.go`.

## <a name="add-the-required-dependencies"></a>Incorporación de las dependencias necesarias

En la parte superior de `create-new-knowledge-base.go`, agregue las líneas siguientes para agregar las dependencias necesarias al proyecto:

:::code language="go" source="~/cognitive-services-quickstart-code/go/QnAMaker/rest/create-kb.go" id="dependencies":::

## <a name="add-the-kb-model-definition"></a>Incorporación de la definición del modelo de la base de conocimiento
Después de las constantes, agregue la siguiente definición de la base de conocimiento. El modelo se convertirá en una cadena después de la definición.

:::code language="go" source="~/cognitive-services-quickstart-code/go/QnAMaker/rest/create-kb.go" id="model":::

## <a name="add-supporting-structures-and-functions"></a>Incorporación de estructuras y funciones auxiliares

A continuación, agregue las siguientes funciones auxiliares.

1. Agregue la estructura de una respuesta HTTP:

    :::code language="go" source="~/cognitive-services-quickstart-code/go/QnAMaker/rest/create-kb.go" id="response":::

1. Agregue el siguiente método para controlar una solicitud POST a las API de QnA Maker. En esta guía de inicio rápido, POST se usa para enviar la definición de base de conocimiento a QnA Maker.

    :::code language="go" source="~/cognitive-services-quickstart-code/go/QnAMaker/rest/create-kb.go" id="post":::

1. Agregue el siguiente método para controlar una solicitud GET a las API de QnA Maker. En esta guía de inicio rápido, la solicitud GET se usa para comprobar el estado de la operación de creación.

    :::code language="go" source="~/cognitive-services-quickstart-code/go/QnAMaker/rest/create-kb.go" id="get":::

## <a name="add-function-to-create-kb"></a>Incorporación de la función para crear una base de conocimiento

Agregue las siguientes funciones para realizar una solicitud HTTP POST a fin de crear la base de conocimiento. El **identificador de la operación** _create_ se devuelve en el campo **Location** del encabezado de respuesta POST y luego se usa como parte de la ruta en la solicitud GET. `Ocp-Apim-Subscription-Key` es la clave del servicio QnA Maker, usada para la autenticación.

:::code language="go" source="~/cognitive-services-quickstart-code/go/QnAMaker/rest/create-kb.go" id="create_kb":::

Esta llamada API devuelve una respuesta JSON que incluye el identificador de operación. Use el identificador de operación para determinar si se ha creado correctamente la base de conocimiento.

```JSON
{
  "operationState": "NotStarted",
  "createdTimestamp": "2018-09-26T05:19:01Z",
  "lastActionTimestamp": "2018-09-26T05:19:01Z",
  "userId": "XXX9549466094e1cb4fd063b646e1ad6",
  "operationId": "8dfb6a82-ae58-4bcb-95b7-d1239ae25681"
}
```

## <a name="add-function-to-get-status"></a>Incorporación de una función para obtener el estado

Agregue la siguiente función para realizar una solicitud HTTP GET para comprobar el estado de la operación. `Ocp-Apim-Subscription-Key` es la clave del servicio QnA Maker, usada para la autenticación.

:::code language="go" source="~/cognitive-services-quickstart-code/go/QnAMaker/rest/create-kb.go" id="get_status":::

Repita la llamada hasta obtener éxito o error:

```JSON
{
  "operationState": "Succeeded",
  "createdTimestamp": "2018-09-26T05:22:53Z",
  "lastActionTimestamp": "2018-09-26T05:23:08Z",
  "resourceLocation": "/knowledgebases/XXX7892b-10cf-47e2-a3ae-e40683adb714",
  "userId": "XXX9549466094e1cb4fd063b646e1ad6",
  "operationId": "177e12ff-5d04-4b73-b594-8575f9787963"
}
```
## <a name="add-main-function"></a>Incorporación de una función principal

La función siguiente es la función principal y permite crear la base de conocimiento y repetir las comprobaciones del estado. Dado que la creación de la base de conocimiento puede tardar algún tiempo, deberá repetir las llamadas para comprobar el estado hasta que sea correcto o se produzca un error.

:::code language="go" source="~/cognitive-services-quickstart-code/go/QnAMaker/rest/create-kb.go" id="main":::


## <a name="compile-the-program"></a>Compilación del programa
Escriba el siguiente comando para compilar el archivo. Si la compilación es correcta, el símbolo del sistema no devuelve ninguna información.

```bash
go build create-new-knowledge-base.go
```

## <a name="run-the-program"></a>Ejecutar el programa

Escriba el siguiente comando en una línea de comandos para ejecutar el programa. La solicitud se envía a QnA Maker API para crear la base de conocimiento y, luego, se sondean los resultados cada 30 segundos. Cada respuesta se imprime en la ventana de consola.

```bash
go run create-new-knowledge-base
```

Una vez creada la base de conocimiento, puede verla en el portal de QnA Maker en la página [My knowledge bases](https://www.qnamaker.ai/Home/MyServices) (Mis bases de conocimiento).

[!INCLUDE [Clean up files and KB](../../../../includes/cognitive-services-qnamaker-quickstart-cleanup-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Referencia de QnA Maker (V4) REST API](https://go.microsoft.com/fwlink/?linkid=2092179)
