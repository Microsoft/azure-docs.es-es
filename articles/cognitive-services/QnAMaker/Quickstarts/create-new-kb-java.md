---
title: 'Guía de inicio rápido: Creación de la base de conocimiento en QnA Maker con REST y Java'
description: Esta guía de inicio rápido basada en REST para Java le lleva por la creación de una base de conocimiento de QnA Maker de ejemplo mediante programación, que aparecerá en el panel de Azure de su cuenta de la API de Cognitive Services.
ms.date: 12/16/2019
ROBOTS: NOINDEX,NOFOLLOW
ms.custom: RESTCURL2020FEB27, devx-track-java
ms.topic: how-to
ms.openlocfilehash: 3a20198e1fce7b72befb0963a4f1eb7a5e7e3f08
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89259802"
---
# <a name="quickstart-create-a-knowledge-base-in-qna-maker-using-java"></a>Guía de inicio rápido: Creación de una base de conocimiento en QnA Maker mediante Java

Esta guía de inicio rápido describe la creación mediante programación de una base de conocimiento de QnA Maker de ejemplo. QnA Maker extrae automáticamente preguntas y respuestas a partir de contenido semiestructurado, como las preguntas frecuentes, y de [orígenes de datos](../Concepts/knowledge-base.md). El modelo para la base de conocimiento se define en el JSON que se envía en el cuerpo de la solicitud de API.

En esta guía de inicio rápido se llama a las siguientes API de QnA Maker:
* [Creación de una base de conocimiento](https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/knowledgebase/create)
* [Obtención de los detalles de la operación](https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/operations/getdetails)

[Documentación de referencia](https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/knowledgebase) | [Ejemplo de Java](https://github.com/Azure-Samples/cognitive-services-qnamaker-java/blob/master/documentation-samples/quickstarts/create-knowledge-base/CreateKB.java)

[!INCLUDE [Custom subdomains notice](../../../../includes/cognitive-services-custom-subdomains-note.md)]

## <a name="prerequisites"></a>Prerequisites

* [Go 1.10.1](https://golang.org/dl/)
* Debe tener un [servicio QnA Maker](../How-To/set-up-qnamaker-service-azure.md). Para recuperar la clave y el punto de conexión (que incluye el nombre del recurso), seleccione **Inicio rápido** en el recurso en Azure Portal.

El [código de ejemplo](https://github.com/Azure-Samples/cognitive-services-qnamaker-java/blob/master/documentation-samples/quickstarts/create-knowledge-base/CreateKB.java) está disponible en el repositorio de GitHub para QnA Maker con Java.

## <a name="create-a-knowledge-base-file"></a>Creación de un archivo de base de conocimiento

Cree un archivo llamado `CreateKB.java`.

## <a name="add-the-required-dependencies"></a>Incorporación de las dependencias necesarias

En la parte superior de `CreateKB.java`, agregue las líneas siguientes para agregar las dependencias necesarias al proyecto:

:::code language="java" source="~/cognitive-services-quickstart-code/java/QnAMaker/rest/CreateKB.java" id="dependencies":::

## <a name="add-the-required-constants"></a>Incorporación de las constantes necesarias
Después de agregar las dependencias necesarias anteriores, agregue las constantes necesarias a la clase `CreateKB` para acceder a QnA Maker.

Debe tener un [servicio QnA Maker](../How-To/set-up-qnamaker-service-azure.md). Para recuperar la clave y el nombre del recurso, seleccione **Inicio rápido** en Azure Portal en el recurso de QnA Maker.

Establezca los valores siguientes:

* `<your-qna-maker-subscription-key>`: la **clave** es una cadena de 32 caracteres y está disponible en Azure Portal, en el recurso de QnA Maker, en la página de inicio rápido. Esta clave no es la misma que la clave del punto de conexión de predicción.
* `<your-resource-name>`: el **nombre del recurso** se usa para construir la dirección URL del punto de conexión de creación con el formato `https://YOUR-RESOURCE-NAME.cognitiveservices.azure.com`. Este nombre del recurso no es el mismo que el que se utiliza para consultar el punto de conexión de predicción.

No es necesario agregar la última llave para finalizar la clase; se encuentra en el fragmento de código final al final de esta guía de inicio rápido.

:::code language="java" source="~/cognitive-services-quickstart-code/java/QnAMaker/rest/CreateKB.java" id="constants":::


## <a name="add-the-kb-model-definition-classes"></a>Incorporación de las clases de definición de modelo de base de conocimiento
Después de agregar las constantes, agregue las siguientes clases y funciones dentro de la clase `CreateKB` para serializar el objeto de definición de modelo en JSON.

:::code language="java" source="~/cognitive-services-quickstart-code/java/QnAMaker/rest/CreateKB.java" id="model":::

## <a name="add-supporting-functions"></a>Incorporación de funciones auxiliares

A continuación, agregue las siguientes funciones auxiliares dentro de la clase `CreateKB`.

1. Agregue la siguiente función para imprimir el código JSON en un formato legible:

    :::code language="java" source="~/cognitive-services-quickstart-code/java/QnAMaker/rest/CreateKB.java" id="pretty":::

2. Agregue la siguiente clase para administrar la respuesta HTTP:

    :::code language="java" source="~/cognitive-services-quickstart-code/java/QnAMaker/rest/CreateKB.java" id="response":::

3. Agregue el siguiente método para realizar una solicitud POST a las API de QnA Maker. `Ocp-Apim-Subscription-Key` es la clave del servicio QnA Maker, usada para la autenticación.

    :::code language="java" source="~/cognitive-services-quickstart-code/java/QnAMaker/rest/CreateKB.java" id="post":::

4. Agregue el siguiente método para realizar una solicitud GET a las API de QnA Maker.

    :::code language="java" source="~/cognitive-services-quickstart-code/java/QnAMaker/rest/CreateKB.java" id="get":::

## <a name="add-a-method-to-create-the-kb"></a>Incorporación de un método para crear la base de conocimiento
Agregue el método siguiente para crear la base de conocimiento mediante una llamada al método Post.

:::code language="java" source="~/cognitive-services-quickstart-code/java/QnAMaker/rest/CreateKB.java" id="create_kb":::

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

## <a name="add-a-method-to-get-status"></a>Incorporación de un método para obtener el estado
Agregue el método siguiente para comprobar el estado de creación.

:::code language="java" source="~/cognitive-services-quickstart-code/java/QnAMaker/rest/CreateKB.java" id="get_status":::

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

## <a name="add-a-main-method"></a>Incorporación de un método main
El método main crea la base de conocimiento y sondea el estado. El identificador de la operación se devuelve en el campo **Location** (Ubicación) del encabezado de respuesta POST y luego se usa como parte de la ruta en la solicitud GET. El bucle `while` reintenta el estado si no se ha completado.

:::code language="java" source="~/cognitive-services-quickstart-code/java/QnAMaker/rest/CreateKB.java" id="main":::

## <a name="compile-and-run-the-program"></a>Compilación y ejecución del programa

1. Asegúrese de que la biblioteca gson se encuentre en el directorio `./libs`. En la línea de comandos, compile el archivo `CreateKB.java`:

    ```bash
    javac -cp ".;libs/*" CreateKB.java
    ```

2. Escriba el siguiente comando en una línea de comandos para ejecutar el programa. La solicitud se envía a QnA Maker API para crear la base de conocimiento y, luego, se sondean los resultados cada 30 segundos. Cada respuesta se imprime en la ventana de consola.

    ```bash
    java -cp ",;libs/*" CreateKB
    ```

Una vez creada la base de conocimiento, puede verla en el portal de QnA Maker en la página [My knowledge bases](https://www.qnamaker.ai/Home/MyServices) (Mis bases de conocimiento).

[!INCLUDE [Clean up files and KB](../../../../includes/cognitive-services-qnamaker-quickstart-cleanup-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Referencia de QnA Maker (V4) REST API](https://go.microsoft.com/fwlink/?linkid=2092179)
