---
title: 'Inicio rápido: Obtención de respuestas de la base de conocimiento en QnA Maker con REST y C#'
description: Esta guía de inicio rápido basada en REST de C# le guiará a través del proceso para obtener una respuesta de una base de conocimiento mediante programación.
ms.date: 02/08/2020
ROBOTS: NOINDEX,NOFOLLOW
ms.custom: RESTCURL2020FEB27, devx-track-csharp
ms.topic: how-to
ms.openlocfilehash: 886a4643210efa95c46cd997eb231601641c5b43
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89259785"
---
# <a name="quickstart-get-answers-to-a-question-from-a-knowledge-base-with-c"></a>Inicio rápido: Obtención de respuestas a una pregunta desde una base de conocimiento con C#

Esta guía de inicio rápido le conduce por la obtención mediante programación de una respuesta de una base de conocimiento de QnA Maker publicada. La base de conocimiento contiene preguntas y respuestas de [orígenes de datos](../Concepts/knowledge-base.md), como preguntas frecuentes. La [pregunta](../how-to/metadata-generateanswer-usage.md#generateanswer-request-configuration) se envía al servicio QnA Maker. La [respuesta](../how-to/metadata-generateanswer-usage.md#generateanswer-response-properties) incluye la respuesta prevista en primer lugar.

[Documentación de referencia](https://docs.microsoft.com/rest/api/cognitiveservices/qnamakerruntime/runtime) | [Ejemplo](https://github.com/Azure-Samples/cognitive-services-qnamaker-csharp/blob/master/documentation-samples/quickstarts/get-answer/QnAMakerAnswerQuestion/Program.cs)

## <a name="prerequisites"></a>Prerrequisitos

* Versión de [**Visual Studio Community Edition**](https://www.visualstudio.com/downloads/) más reciente.
* Debe tener un [servicio QnA Maker](../How-To/set-up-qnamaker-service-azure.md). Para recuperar la clave, seleccione **Claves** en **Administración de recursos** en el panel de Azure del recurso de QnA Maker.
* Configuración de la página **Publicar**. Si no tiene una base de conocimiento publicada, cree una vacía, importe una base de conocimiento en la página **Configuración** y, después, publíquela. Puede descargar y usar [esta base de conocimiento básica](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/knowledge-bases/basic-kb.tsv).

    La configuración de la página de publicación incluye el valor de la ruta de POST, el valor de Host y el valor de EndpointKey.

    ![Configuración Publicar](../media/qnamaker-quickstart-get-answer/publish-settings.png)

## <a name="create-a-knowledge-base-project"></a>Creación de un proyecto de base de conocimiento

1. Abra Visual Studio 2019 Community Edition.
1. Cree un nuevo proyecto de aplicación de consola (.Net Core) y asígnele el nombre QnaMakerQuickstart. Acepte los valores predeterminados del resto de la configuración.

## <a name="add-the-required-dependencies"></a>Incorporación de las dependencias necesarias

En la parte superior del archivo program.cs, reemplace la instrucción using por las líneas siguientes para agregar las dependencias necesarias al proyecto:

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/QnAMaker/rest/query-kb.cs" id="dependencies":::

## <a name="add-the-required-constants"></a>Incorporación de las constantes necesarias

Al principio de la clase `Program`, en `Main`, agregue las constantes necesarias para acceder a QnA Maker. Estos valores se encuentran en la página **Publicar** después de publicar la base de conocimiento.

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/QnAMaker/rest/query-kb.cs" id="constants":::

## <a name="add-a-post-request-to-send-question-and-get-answer"></a>Incorporación de una solicitud POST para enviar una pregunta y recibir una respuesta

El código siguiente realiza una solicitud HTTPS a QnA Maker API para enviar la pregunta a la base de conocimiento y recibe la respuesta:

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/QnAMaker/rest/query-kb.cs" id="post":::

El valor del encabezado `Authorization` incluye la cadena `EndpointKey`.

Más información sobre la [solicitud](../how-to/metadata-generateanswer-usage.md#generateanswer-request) y la [respuesta](../how-to/metadata-generateanswer-usage.md#generateanswer-response).

## <a name="build-and-run-the-program"></a>Compilar y ejecutar el programa

Compile y ejecute el programa desde Visual Studio. Este enviará automáticamente la solicitud a QnA Maker API y luego la imprimirá en la ventana de la consola.

[!INCLUDE [JSON request and response](../../../../includes/cognitive-services-qnamaker-quickstart-get-answer-json.md)]

[!INCLUDE [Clean up files and knowledge base](../../../../includes/cognitive-services-qnamaker-quickstart-cleanup-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Referencia de QnA Maker (V4) REST API](https://go.microsoft.com/fwlink/?linkid=2092179)
