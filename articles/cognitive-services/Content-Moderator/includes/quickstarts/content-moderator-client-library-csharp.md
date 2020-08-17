---
title: Inicio rápido de la biblioteca cliente de .NET para Content Moderator
titleSuffix: Azure Cognitive Services
description: Empiece a trabajar con la biblioteca cliente de Content Moderator para .NET con este inicio rápido.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: quickstart
ms.date: 05/27/2020
ms.author: pafarley
ms.openlocfilehash: aa2663706d052619dd1f40c0154222b0dac8c790
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87298880"
---
Introducción a la biblioteca cliente de Content Moderator para .NET. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas. Content Moderator es un servicio cognitivo que comprueba el contenido de texto, imágenes y vídeo en busca de material que sea potencialmente ofensivo, peligroso o no deseable por algún otro motivo. Cuando se encuentra dicho material, el servicio aplica las etiquetas adecuadas (marcas) al contenido. Luego, la aplicación puede controlar el contenido marcado para cumplir las normativas o mantener el entorno adecuado para los usuarios.

Use la biblioteca cliente de Content Moderator para .NET para:

* [Moderar texto](#moderate-text)
* [Moderar imágenes](#moderate-images)
* [Crear una revisión](#create-a-review)

[Documentación de referencia](https://docs.microsoft.com/dotnet/api/overview/azure/cognitiveservices/client/contentmoderator?view=azure-dotnet) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Vision.ContentModerator) | [Paquete (NuGet)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) | [Ejemplos](https://docs.microsoft.com/azure/cognitive-services/content-moderator/samples-dotnet)

## <a name="prerequisites"></a>Prerrequisitos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/)
* La versión actual de [.NET Core](https://dotnet.microsoft.com/download/dotnet-core).

## <a name="setting-up"></a>Instalación

### <a name="create-a-content-moderator-azure-resource"></a>Creación de un recurso de Azure para Content Moderator

Los servicios de Azure Cognitive Services se representan por medio de recursos de Azure a los que se suscribe. Cree un recurso para Content Moderator con [Azure Portal](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account) o la [CLI de Azure](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account-cli) en la máquina local. También puede:

* Ver este recurso en [Azure Portal](https://portal.azure.com/).

Después de obtener una clave del recurso, [cree las variables de entorno](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account#configure-an-environment-variable-for-authentication) para la clave y la dirección URL del punto de conexión, llamadas `CONTENT_MODERATOR_SUBSCRIPTION_KEY` y `CONTENT_MODERATOR_ENDPOINT`, respectivamente.

### <a name="create-a-new-c-application"></a>Creación de una aplicación de C#

Cree una nueva aplicación de consola de .NET Core en el IDE o editor de texto que prefiera. 

En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `dotnet new` para crear una nueva aplicación de consola con el nombre `content-moderator-quickstart`. Este comando crea un sencillo proyecto "Hola mundo" de C# con un solo archivo de origen: *Program.cs*.

```console
dotnet new console -n content-moderator-quickstart
```

Cambie el directorio a la carpeta de aplicaciones recién creada. Para compilar la aplicación:

```console
dotnet build
```

La salida de la compilación no debe contener advertencias ni errores. 

```console
...
Build succeeded.
 0 Warning(s)
 0 Error(s)
...
```

En el directorio del proyecto, abra el archivo *Program.cs* en el editor o IDE que prefiera. Agregue las instrucciones siguientes `using` :

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_using)]

En la clase **Program**, cree variables para la ubicación del punto de conexión del recurso y claves como variables de entorno.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_creds)]

> [!NOTE]
> Si ha creado las variables de entorno después de haber iniciado la aplicación, deberá cerrar y volver a abrir el editor, el IDE o el shell que lo ejecuta para acceder a ellas.

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Dentro del directorio de aplicaciones, instale la biblioteca cliente de Content Moderator para .NET con el siguiente comando:

```console
dotnet add package Microsoft.Azure.CognitiveServices.ContentModerator --version 2.0.0
```

Si usa el IDE de Visual Studio, la biblioteca cliente estará disponible como un paquete de NuGet descargable.

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases controlan algunas de las características principales de la biblioteca cliente de .NET para Content Moderator.

|Nombre|Descripción|
|---|---|
|[ContentModeratorClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.contentmoderator.contentmoderatorclient?view=azure-dotnet)|Esta clase es necesaria para todas las funciones de Content Moderator. Cree una instancia de ella con la información de suscripción y úsela para generar instancias de otras clases.|
|[ImageModeration](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.contentmoderator.imagemoderation?view=azure-dotnet)|Esta clase proporciona la funcionalidad para analizar imágenes de contenido para adultos, información personal o caras humanas.|
|[TextModeration](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.contentmoderator.textmoderation?view=azure-dotnet)|Esta clase proporciona la funcionalidad para analizar el texto y detectar el idioma, palabras obscenas, errores e información personal.|
|[Revisiones](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.contentmoderator.reviews?view=azure-dotnet)|Esta clase proporciona la funcionalidad de las API de revisión, como los métodos para crear trabajos, flujos de trabajo personalizados y revisiones humanas.|

## <a name="code-examples"></a>Ejemplos de código


En estos fragmentos de código se muestra cómo realizar las siguientes tareas con la biblioteca cliente de Content Moderator para .NET:

* [Autenticar el cliente](#authenticate-the-client)
* [Moderar texto](#moderate-text)
* [Moderar imágenes](#moderate-images)
* [Crear una revisión](#create-a-review)

## <a name="authenticate-the-client"></a>Autenticar el cliente

En un nuevo método, cree una instancia de objetos de cliente con la clave y el punto de conexión. No necesita un cliente diferente para cada escenario, aunque esto puede resultar útil para mantener el código organizado.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_client)]

## <a name="moderate-text"></a>Moderar texto

En el código siguiente se usa un cliente de Content Moderator para analizar un cuerpo de texto e imprimir los resultados en la consola. En la raíz de su clase **Program**, defina los archivos de entrada y salida:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_text_vars)]

A continuación, en la raíz del proyecto agregue un archivo *TextFile.txt*. Agregue su propio texto a este archivo o use el siguiente texto de ejemplo:

```
Is this a grabage email abcdef@abcd.com, phone: 4255550111, IP: 255.255.255.255, 1234 Main Boulevard, Panapolis WA 96555.
Crap is the profanity here. Is this information PII? phone 4255550111
```

Agregue la siguiente llamada de método al método `Main`.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_textmod_call)]

A continuación, defina el método de moderación de texto en algún lugar de la clase **Program**:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_textmod)]

## <a name="moderate-images"></a>Moderar imágenes

En el código siguiente se usa un cliente de Content Moderator, junto con un objeto [ImageModeration](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.contentmoderator.imagemoderation?view=azure-dotnet), para analizar imágenes remotas en busca de contenido para adultos y de tipo explícito.

> [!NOTE]
> También puede analizar el contenido de una imagen local. Consulte en la [documentación de referencia](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.contentmoderator.imagemoderation.evaluatefileinputwithhttpmessagesasync?view=azure-dotnet#Microsoft_Azure_CognitiveServices_ContentModerator_ImageModeration_EvaluateFileInputWithHttpMessagesAsync_System_IO_Stream_System_Nullable_System_Boolean__System_Collections_Generic_Dictionary_System_String_System_Collections_Generic_List_System_String___System_Threading_CancellationToken_) los métodos y las operaciones que funcionan con imágenes locales.

### <a name="get-sample-images"></a>Obtener imágenes de ejemplo

Defina los archivos de entrada y salida:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_image_vars)]

A continuación, cree el archivo de entrada, *ImageFiles.txt*, en la raíz del proyecto. En este archivo, se agregan las direcciones URL de las imágenes que se van a analizar, una en cada línea. Puede utilizar las siguientes imágenes de ejemplo:

```
https://moderatorsampleimages.blob.core.windows.net/samples/sample2.jpg
https://moderatorsampleimages.blob.core.windows.net/samples/sample5.png
```

Pase los archivos de entrada y salida a la siguiente llamada de método en el método `Main`. Definirá este método en un paso posterior.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_imagemod_call)]

### <a name="define-helper-class"></a>Definir la clase auxiliar

Agregue la definición de clase siguiente en la clase **Program**. Esta clase interna controlará los resultados de moderación de las imágenes.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_dataclass)]

### <a name="define-the-image-moderation-method"></a>Definición del método de moderación de la imagen

El método siguiente recorre en iteración las direcciones URL de imágenes de un archivo de texto, crea una instancia de **EvaluationData** y analiza la imagen para detectar contenido para adultos o explícito, texto y caras humanas. A continuación, agrega la instancia final de **EvaluationData** a una lista y escribe la lista completa de datos devueltos en la consola.

#### <a name="iterate-through-image-urls"></a>Recorrer en iteración las direcciones URL de imágenes

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_imagemod_iterate)]

#### <a name="analyze-content"></a>Análisis de contenido

Para más información sobre los atributos de imagen que Content Moderator analiza, consulte la guía [Conceptos de moderación de imágenes](../../image-moderation-api.md).

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_imagemod_analyze)]

#### <a name="write-moderation-results-to-file"></a>Escribir resultados de moderación en un archivo

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_imagemod_save)]

## <a name="create-a-review"></a>Creación de una revisión

Puede usar la biblioteca cliente de .NET para Content Moderator para insertar contenido en la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com) y que los moderadores humanos puedan revisarlo. Para más información sobre la herramienta de revisión, consulte la [guía conceptual de la herramienta de revisión](../../review-tool-user-guide/human-in-the-loop.md).

El método de esta sección usa la clase [Reviews](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.contentmoderator.reviews?view=azure-dotnet) para crear una revisión, recuperar su identificador y comprobar sus detalles después de recibir la entrada humana a través del portal web de la herramienta de revisión. Registra toda esta información en un archivo de texto de salida. Llame al método desde el método `Main`:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_review_call)]

### <a name="get-sample-images"></a>Obtener imágenes de ejemplo

Declare la siguiente matriz en la raíz de su clase **Program**. Esta variable hace referencia a una imagen de ejemplo que se usa para crear la revisión.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_review_urls)]

### <a name="get-review-credentials"></a>Obtener las credenciales de revisión

Inicie sesión en la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com) y recupere el nombre de su equipo. A continuación, asígnelo a la variable adecuada en la clase **Program**. Opcionalmente, puede configurar un punto de conexión de devolución de llamada para recibir actualizaciones sobre la actividad de la revisión.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_review_vars)]

### <a name="define-helper-class"></a>Definir la clase auxiliar

Agregue la definición de clase siguiente en la clase **Program**. Esta clase se usará para representar una única instancia de revisión que se envía a la herramienta de revisión.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_review_item)]

### <a name="define-helper-method"></a>Definir el método auxiliar

Agregue el método siguiente a la clase **Program**. Este método escribirá los resultados de las consultas de revisión en el archivo de texto de salida.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_writeline)]

### <a name="define-the-review-creation-method"></a>Definir el método de creación de la revisión

Ahora está listo para definir el método que controlará la creación y consulta de la revisión. Agregue un nuevo método, **CreateReviews**, y defina las siguientes variables locales.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_createreview_fields)]

#### <a name="post-reviews-to-the-review-tool"></a>Publicar revisiones en la herramienta de revisión

A continuación, agregue el código siguiente para recorrer en iteración las imágenes de ejemplo determinadas, agregar metadatos y enviarlo todo a la herramienta de revisión en un solo lote. 

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_createreview_create)]

El objeto devuelto de la llamada a la API contendrá valores de identificador únicos para cada imagen cargada. En el código siguiente se analizan estos identificadores y, a continuación, se usan para consultar en Content Moderator el estado de cada imagen del lote.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_createreview_ids)]

### <a name="get-review-details"></a>Obtención de detalles de la revisión

El código siguiente hace que el programa espere la entrada del usuario. Cuando llegue a este paso en tiempo de ejecución, puede ir a la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com) personalmente, comprobar que se ha cargado la imagen de ejemplo e interactuar con ella. Para obtener información sobre cómo interactuar con una revisión, consulte la [guía de procedimientos de revisión](https://docs.microsoft.com/azure/cognitive-services/content-moderator/review-tool-user-guide/review-moderated-images). Cuando haya terminado, puede presionar cualquier tecla para continuar con el programa y recuperar los resultados del proceso de revisión.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_createreview_results)]

Si ha usado un punto de conexión de devolución de llamada en este escenario, debería recibir un evento en este formato:

```console
{'callback_endpoint': 'https://requestb.in/qmsakwqm',
 'content': '',
 'content_id': '3ebe16cb-31ed-4292-8b71-1dfe9b0e821f',
 'created_by': 'cspythonsdk',
 'metadata': [{'key': 'sc', 'value': 'True'}],
 'review_id': '201901i14682e2afe624fee95ebb248643139e7',
 'reviewer_result_tags': [{'key': 'a', 'value': 'True'},
                          {'key': 'r', 'value': 'True'}],
 'status': 'Complete',
 'sub_team': 'public',
 'type': 'Image'}
```

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación desde el directorio de la aplicación con el comando `dotnet run`.

```dotnet
dotnet run 
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a usar la biblioteca de .NET para Content Moderator para realizar tareas de moderación. A continuación, lea una guía conceptual para aprender más sobre la moderación de imágenes u otro contenido multimedia.

> [!div class="nextstepaction"]
> [Conceptos de moderación de imágenes](https://docs.microsoft.com/azure/cognitive-services/content-moderator/image-moderation-api)

* [¿Qué es Azure Content Moderator?](../../overview.md)
* El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/ContentModerator/Program.cs).
