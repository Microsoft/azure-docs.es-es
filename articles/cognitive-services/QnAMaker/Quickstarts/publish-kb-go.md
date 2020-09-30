---
title: 'Inicio rápido: Publicación de la base de conocimiento en QnA Maker con REST y Go'
description: Esta guía de inicio rápido basada en REST de Go publica la base de conocimiento y crea un punto de conexión al que se puede llamar en su aplicación o bot de chat.
ms.date: 02/08/2020
ROBOTS: NOINDEX,NOFOLLOW
ms.custom: RESTCURL2020FEB27
ms.topic: how-to
ms.openlocfilehash: b94b09fcb3bfff2eeacabaaa49eb5e4c751ec79d
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89267758"
---
# <a name="quickstart-publish-a-knowledge-base-in-qna-maker-using-go"></a>Inicio rápido: Publicación de una base de conocimiento en QnA Maker con Go

En esta guía de inicio rápido basada en REST se describe la publicación mediante programación de una base de conocimiento (KB). La publicación inserta la versión más reciente de la base de conocimiento en un índice de Azure Cognitive Search dedicado y crea un punto de conexión que se puede llamar en su aplicación o bot de chat.

En esta guía de inicio rápido se llama a las siguientes API de QnA Maker:
* [Publicar](https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/knowledgebase/publish): esta API no requiere ninguna información en el cuerpo de la solicitud.

## <a name="prerequisites"></a>Prerrequisitos

* [Go 1.10.1](https://golang.org/dl/)
* Debe tener un [servicio QnA Maker](../How-To/set-up-qnamaker-service-azure.md). Para recuperar la clave y el punto de conexión (que incluye el nombre del recurso), seleccione **Inicio rápido** en el recurso en Azure Portal.

* Identificador de la base de conocimiento (KB) de QnA Maker encontrado en la dirección URL en el parámetro de cadena de consulta de `kbid`, como se muestra a continuación.

    ![Identificador de base de conocimiento de QnA Maker](../media/qnamaker-quickstart-kb/qna-maker-id.png)

    Si aún no tiene una base de conocimiento, puede crear una de ejemplo para usar en este inicio rápido: [Creación de una base de conocimiento](create-new-kb-csharp.md)

> [!NOTE]
> Los archivos de la solución completa están disponibles en el repositorio [**Azure-Samples/cognitive-services-qnamaker-go** de GitHub](https://github.com/Azure-Samples/cognitive-services-qnamaker-go/tree/master/documentation-samples/quickstarts/publish-knowledge-base).

## <a name="create-a-go-file"></a>Creación de un archivo de Go

Abra VSCode y cree un archivo llamado `publish-kb.go`.

## <a name="add-the-required-dependencies"></a>Incorporación de las dependencias necesarias

En la parte superior de `publish-kb.go`, agregue las líneas siguientes para agregar las dependencias necesarias al proyecto:

:::code language="go" source="~/cognitive-services-quickstart-code/go/QnAMaker/rest/publish-kb.go" id="dependencies":::

## <a name="create-the-main-function"></a>Creación de la función main

Después de las dependencias necesarias, agregue la siguiente clase:

```Go
package main

func main() {

}
```

## <a name="add-post-request-to-publish-kb"></a>Incorporación de la solicitud POST para publicar la base de conocimiento

Agregue el siguiente código, que realiza una solicitud HTTPS a QnA Maker API para publicar una base de conocimiento y recibe la respuesta:

:::code language="go" source="~/cognitive-services-quickstart-code/go/QnAMaker/rest/publish-kb.go" id="main":::

La llamada API devuelve un estado 204 en caso de publicación correcta sin ningún contenido en el cuerpo de la respuesta. El código agrega el contenido para las respuestas 204.

Para cualquier otra respuesta, esa respuesta se devuelve sin modificar.

## <a name="build-and-run-the-program"></a>Compilar y ejecutar el programa

Escriba el siguiente comando para compilar el archivo. Si la compilación es correcta, el símbolo del sistema no devuelve ninguna información.

```bash
go build publish-kb.go
```

Escriba el siguiente comando en una línea de comandos para ejecutar el programa. Este enviará la solicitud a QnA Maker API para publicar la base de conocimiento y, después, imprimirá 204 en el caso de aciertos o errores.

```bash
./publish-kb
```

[!INCLUDE [Clean up files and knowledge base](../../../../includes/cognitive-services-qnamaker-quickstart-cleanup-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

Después de publicar la base de conocimiento, es preciso que la dirección URL del punto de conexión [ genere una respuesta](./get-answer-from-knowledge-base-go.md).

> [!div class="nextstepaction"]
> [Referencia de QnA Maker (V4) REST API](https://go.microsoft.com/fwlink/?linkid=2092179)
