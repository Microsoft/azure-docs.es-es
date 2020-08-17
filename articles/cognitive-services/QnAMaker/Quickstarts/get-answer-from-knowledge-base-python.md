---
title: 'Inicio rápido: Inicio rápido: Obtención de respuestas de la base de conocimiento en QnA Maker con REST y Python'
description: Esta guía de inicio rápido basada en REST de Python le guiará a través del proceso para obtener una respuesta de una base de conocimiento mediante programación.
ms.date: 02/08/2020
ROBOTS: NOINDEX,NOFOLLOW
ms.custom: RESTCURL2020FEB27, devx-track-python
ms.topic: how-to
ms.openlocfilehash: 2d693a63360890b9f658baa1a37d251f311d5617
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87875020"
---
# <a name="quickstart-get-answers-to-a-question-from-a-knowledge-base-with-python"></a>Inicio rápido: Obtención de respuestas a una pregunta desde una base de conocimiento con Python

Esta guía de inicio rápido le conduce por la obtención mediante programación de una respuesta de una base de conocimiento de QnA Maker publicada. La base de conocimiento contiene preguntas y respuestas de [orígenes de datos](../Concepts/knowledge-base.md), como preguntas frecuentes. La [pregunta](../how-to/metadata-generateanswer-usage.md#generateanswer-request-configuration) se envía al servicio QnA Maker. La [respuesta](../how-to/metadata-generateanswer-usage.md#generateanswer-response-properties) incluye la respuesta prevista en primer lugar.

[Documentación de referencia](https://docs.microsoft.com/rest/api/cognitiveservices/qnamakerruntime/runtime) | [Ejemplo](https://github.com/Azure-Samples/cognitive-services-qnamaker-python/blob/master/documentation-samples/quickstarts/get-answer/get-answer-3x.py)

## <a name="prerequisites"></a>Prerrequisitos

* [Python 3.6 o superior](https://www.python.org/downloads/)
* [Visual Studio Code](https://code.visualstudio.com/)
* Debe tener un [servicio QnA Maker](../How-To/set-up-qnamaker-service-azure.md). Para recuperar la clave, seleccione **Claves** en **Administración de recursos** en el panel de Azure del recurso de QnA Maker.
* Configuración de la página **Publicar**. Si no tiene una base de conocimiento publicada, cree una vacía, importe una base de conocimiento en la página **Configuración** y, después, publíquela. Puede descargar y usar [esta base de conocimiento básica](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/knowledge-bases/basic-kb.tsv).

    La configuración de la página de publicación incluye el valor de la ruta de POST, el valor de Host y el valor de EndpointKey.

    ![Configuración Publicar](../media/qnamaker-quickstart-get-answer/publish-settings.png)

## <a name="create-a-python-file"></a>Creación de un archivo de Python

Abra VSCode y cree un archivo llamado `get-answer-3x.py`.

## <a name="add-the-required-dependencies"></a>Incorporación de las dependencias necesarias

Al principio del archivo `get-answer-3x.py`, agregue las dependencias necesarias al proyecto:

[!code-python[Add the required dependencies](~/samples-qnamaker-python/documentation-samples/quickstarts/get-answer/get-answer-3x.py?range=1-2 "Add the required dependencies")]

<!--TBD - reword this following paragraph -->

El host y la ruta son diferentes a los que aparecen en la página **Publicar**. Eso se debe a que la biblioteca de python no permite ningún tipo de enrutamiento en el host. La ruta que aparece en la página **Publicar** como parte del host se ha movido a la ruta.

## <a name="add-the-required-constants"></a>Incorporación de las constantes necesarias

Agregue las constantes necesarias para acceder a QnA Maker. Estos valores se encuentran en la página **Publicar** después de publicar la base de conocimiento.

[!code-python[Add the required constants](~/samples-qnamaker-python/documentation-samples/quickstarts/get-answer/get-answer-3x.py?range=5-25 "Add the required constants")]

## <a name="add-a-post-request-to-send-question-and-get-an-answer"></a>Incorporación de una solicitud POST para enviar una pregunta y recibir una respuesta

El código siguiente realiza una solicitud HTTPS a QnA Maker API para enviar la pregunta a la base de conocimiento y recibe la respuesta:

[!code-python[Add a POST request to send question to knowledge base](~/samples-qnamaker-python/documentation-samples/quickstarts/get-answer/get-answer-3x.py?range=27-48 "Add a POST request to send question to knowledge base")]

El valor del encabezado `Authorization` incluye la cadena `EndpointKey`.

## <a name="run-the-program"></a>Ejecución del programa

Ejecute el programa desde la línea de comandos. Este enviará automáticamente la solicitud a QnA Maker API y luego la imprimirá en la ventana de la consola.

Ejecute el archivo:

```bash
python get-answer-3x.py
```

[!INCLUDE [JSON request and response](../../../../includes/cognitive-services-qnamaker-quickstart-get-answer-json.md)]

Más información sobre la [solicitud](../how-to/metadata-generateanswer-usage.md#generateanswer-request) y la [respuesta](../how-to/metadata-generateanswer-usage.md#generateanswer-response).

[!INCLUDE [Clean up files and knowledge base](../../../../includes/cognitive-services-qnamaker-quickstart-cleanup-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Referencia de QnA Maker (V4) REST API](https://go.microsoft.com/fwlink/?linkid=2092179)
