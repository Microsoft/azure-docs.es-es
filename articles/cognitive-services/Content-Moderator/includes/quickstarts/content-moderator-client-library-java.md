---
title: Inicio rápido de la biblioteca cliente de Java para Content Moderator
titleSuffix: Azure Cognitive Services
description: En este inicio rápido empezará a usar la biblioteca cliente de Content Moderator de Azure Cognitive Services para Java.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: include
ms.date: 08/17/2020
ms.custom: devx-track-java
ms.author: pafarley
ms.openlocfilehash: 2fb18503600d48775c2d5cb981841f5ae627e7eb
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88510138"
---
Introducción a la biblioteca cliente de Content Moderator para Java. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas. Content Moderator es un servicio cognitivo que comprueba el contenido de texto, imágenes y vídeo en busca de material que sea potencialmente ofensivo, peligroso o no deseable por algún otro motivo. Cuando se encuentra dicho material, el servicio aplica las etiquetas adecuadas (marcas) al contenido. Luego, la aplicación puede controlar el contenido marcado para cumplir las normativas o mantener el entorno adecuado para los usuarios.

Use la biblioteca cliente de Content Moderator para Java para:

* Moderar las imágenes de contenido para adultos o de tipo explícito, texto o caras humanas.

[Documentación de referencia](https://docs.microsoft.com/java/api/overview/azure/cognitiveservices/client/contentmoderator?view=azure-java-stable) | [Artefacto (Maven)](https://mvnrepository.com/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-contentmoderator) | [Ejemplos](https://docs.microsoft.com/samples/browse/?products=azure&term=content-moderator)

## <a name="prerequisites"></a>Prerrequisitos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/)
* La última versión de [Java Development Kit (JDK)](https://www.oracle.com/technetwork/java/javase/downloads/index.html)
* La [herramienta de compilación de Gradle](https://gradle.org/install/) u otro administrador de dependencias.

## <a name="setting-up"></a>Instalación

### <a name="create-a-content-moderator-azure-resource"></a>Creación de un recurso de Azure para Content Moderator

Los servicios de Azure Cognitive Services se representan por medio de recursos de Azure a los que se suscribe. Cree un recurso para Content Moderator con [Azure Portal](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account) o la [CLI de Azure](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account-cli) en la máquina local. También puede:

* Ver el recurso en [Azure Portal](https://portal.azure.com/)

Después de obtener una clave del recurso, [cree una variable de entorno](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account#configure-an-environment-variable-for-authentication) para ella, denominada `AZURE_CONTENTMODERATOR_KEY`.

### <a name="create-a-new-gradle-project"></a>Creación de un proyecto de Gradle

En una ventana de la consola (como cmd, PowerShell o Bash), cree un directorio para la aplicación y vaya a él. 

```console
mkdir myapp && cd myapp
```
Ejecute `gradle init`. Este comando creará archivos de compilación esenciales para Gradle, como *build.gradle.kts*, que se usa en tiempo de ejecución para crear y configurar la aplicación. Ejecute este comando desde el directorio de trabajo:

```console
gradle init --type basic
```

Cuando se le pida que elija un lenguaje DSL para el script de compilación, seleccione **Kotlin**.

Busque *build.gradle.kts* y ábralo con el IDE o el editor de texto que prefiera. A continuación, cópielo en la siguiente configuración de compilación. Esta configuración define el proyecto como una aplicación Java cuyo punto de entrada es la clase **ContentModeratorQuickstart**. Se importa la biblioteca cliente de Content Moderator, así como el SDK de Gson para la serialización JSON.

```kotlin
plugins {
    java
    application
}

application{ 
    mainClassName = "ContentModeratorQuickstart"
}

repositories{
    mavenCentral()
}

dependencies{
    compile(group = "com.microsoft.azure.cognitiveservices", name = "azure-cognitiveservices-contentmoderator", version = "1.0.2-beta")
    compile(group = "com.google.code.gson", name = "gson", version = "2.8.5")
}
```

En el directorio de trabajo, ejecute el siguiente comando para crear una carpeta de origen del proyecto.

```console
mkdir -p src/main/java
```

Luego, cree un archivo llamado *ContentModeratorQuickstart.java* en la nueva carpeta. Abra el archivo en el editor o el IDE que prefiera e importe las siguientes bibliotecas encima:

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imports)]

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases controlan algunas de las características principales de la biblioteca cliente de Content Moderator.

|Nombre|Descripción|
|---|---|
|[ContentModeratorClient](https://docs.microsoft.com/java/api/com.microsoft.azure.cognitiveservices.vision.contentmoderator.contentmoderatorclient?view=azure-java-stable)|Esta clase es necesaria para todas las funciones de Content Moderator. Cree una instancia de ella con la información de suscripción y úsela para generar instancias de otras clases.|
|[ImageModeration](https://docs.microsoft.com/java/api/com.microsoft.azure.cognitiveservices.vision.contentmoderator.imagemoderations?view=azure-java-stable)|Esta clase proporciona la funcionalidad para analizar imágenes de contenido para adultos, información personal o caras humanas.|
|[TextModerations](https://docs.microsoft.com/java/api/com.microsoft.azure.cognitiveservices.vision.contentmoderator.textmoderations?view=azure-java-stable)|Esta clase proporciona la funcionalidad para analizar el texto y detectar el idioma, palabras obscenas, errores e información personal.|
|[Revisiones](https://docs.microsoft.com/java/api/com.microsoft.azure.cognitiveservices.vision.contentmoderator.reviews?view=azure-java-stable)|Esta clase proporciona la funcionalidad de las API de revisión, como los métodos para crear trabajos, flujos de trabajo personalizados y revisiones humanas.|


## <a name="code-examples"></a>Ejemplos de código

En estos fragmentos de código se muestra cómo realizar las siguientes tareas con la biblioteca cliente de Content Moderator para Java:

* [Autenticar el cliente](#authenticate-the-client)
* [Moderar imágenes](#moderate-images)

## <a name="authenticate-the-client"></a>Autenticar el cliente

> [!NOTE]
> En este paso se supone que ha [creado una variable de entorno](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account#configure-an-environment-variable-for-authentication) para la clave de Content Moderator, llamada `AZURE_CONTENTMODERATOR_KEY`.

En el método `main` de la aplicación, cree un objeto [ContentModeratorClient](https://docs.microsoft.com/java/api/com.microsoft.azure.cognitiveservices.vision.contentmoderator.contentmoderatorclient?view=azure-java-stable) con el valor del punto de conexión de la suscripción y la variable de entorno de la clave de suscripción. 

> [!NOTE]
> Si ha creado la variable de entorno después de haber iniciado la aplicación, deberá cerrar y volver a abrir el editor, el IDE o el shell que lo ejecuta para acceder a la variable.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_client)]

## <a name="moderate-images"></a>Moderar imágenes

### <a name="get-images"></a>Obtener imágenes

En la carpeta **src/main/** del proyecto, cree una carpeta **resources** y vaya a ella. Luego, cree el archivo *ImageFiles.txt*. En este archivo, se agregan las direcciones URL de las imágenes que se van a analizar, una en cada línea. Puede utilizar las siguientes imágenes de ejemplo:

```
https://moderatorsampleimages.blob.core.windows.net/samples/sample2.jpg
https://moderatorsampleimages.blob.core.windows.net/samples/sample5.png
```

### <a name="define-helper-class"></a>Definir la clase auxiliar

Luego, en el archivo *ContentModeratorQuickstart.java*, agregue la siguiente definición de clase dentro de la clase **ContentModeratorQuickstart**. Esta clase interna se usará más adelante en el proceso de moderación de imágenes.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_evaluationdata)]

### <a name="iterate-through-images"></a>Recorrer en iteración las imágenes

A continuación, agregue el siguiente código al final del método `main`. O bien, puede agregarlo a otro método que se llama desde `main`. Este código recorre cada línea del archivo _ImageFiles.txt_.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_iterate)]

### <a name="check-for-adultracy-content"></a>Buscar contenido para adultos o de tipo explícito
Esta línea de código comprueba la imagen de la dirección URL especificada en busca de contenido para adultos o de tipo explícito. Consulte la guía conceptual de moderación de imágenes para información sobre estos términos.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_ar)]

### <a name="check-for-text"></a>Buscar texto
Esta línea de código busca en la imagen texto visible.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_text)]

### <a name="check-for-faces"></a>Buscar caras
Esta línea de código busca en la imagen caras humanas.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_faces)]

Por último, almacene la información devuelta en la lista `EvaluationData`.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_storedata)]

### <a name="print-results"></a>Imprimir resultados

Después del bucle `while`, agregue el siguiente código, que imprime los resultados en la consola y en un archivo de salida *src/main/resources/ModerationOutput.json*.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_printdata)]

Cierre la instrucción `try` y agregue una instrucción `catch` para completar el método.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_catch)]

## <a name="run-the-application"></a>Ejecución de la aplicación

Puede compilar la aplicación con:

```console
gradle build
```

Ejecute la aplicación con el comando `gradle run`:

```console
gradle run
```

Luego, vaya al archivo *src/main/resources/ModerationOutput.json* y vea los resultados de la moderación de contenido.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a usar la biblioteca Java de Content Moderator para realizar tareas de moderación. A continuación, lea una guía conceptual para aprender más sobre la moderación de imágenes u otro contenido multimedia.

> [!div class="nextstepaction"]
>[Conceptos de moderación de imágenes](https://docs.microsoft.com/azure/cognitive-services/content-moderator/image-moderation-api)

* [¿Qué es Azure Content Moderator?](../../overview.md)
* El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java).