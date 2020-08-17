---
title: Inicio rápido de la biblioteca cliente para Java de Bing Custom Search
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 02/27/2020
ms.custom: devx-track-java
ms.author: aahi
ms.openlocfilehash: e871edf1f16c1c73a3c3b16649e5aeb622397c8d
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87375245"
---
Introducción a la biblioteca de cliente de Bing Custom Search para Java. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas. Bing Custom Search API le permite crear experiencias de búsqueda a medida y sin anuncios para los temas que le interesan. El código fuente correspondiente a este ejemplo se encuentra disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples/tree/master/Search/BingCustomSearch).

Use la biblioteca de cliente de Bing Custom Search para Java para:

* Buscar los resultados de la búsqueda en la Web, desde la instancia de Bing Custom Search.

[Documentación de referencia](https://docs.microsoft.com/java/api/overview/azure/cognitiveservices/client/bingcustomsearch?view=azure-java-stable) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Search.BingCustomSearch) | [Artifact (Maven)](https://search.maven.org/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-customsearch/) | [Ejemplos](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/).
* La versión actual del [kit de desarrollo de Java (JDK)](https://www.oracle.com/technetwork/java/javase/downloads/index.html).
* La [herramienta de compilación de Gradle](https://gradle.org/install/) u otro administrador de dependencias.
* Una instancia de Bing Custom Search. Consulte [Quickstart: Creación de la primera instancia de Bing Custom Search](../../quick-start.md) para obtener más información.

[!INCLUDE [cognitive-services-bing-custom-search-prerequisites](~/includes/cognitive-services-bing-custom-search-signup-requirements.md)]

Después de obtener una clave del recurso, [cree una variable de entorno](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account#configure-an-environment-variable-for-authentication) para la clave, llamada `AZURE_BING_CUSTOM_SEARCH_API_KEY`.

### <a name="create-a-new-gradle-project"></a>Creación de un proyecto de Gradle

> [!TIP]
> Si no usa Gradle, puede encontrar la biblioteca de cliente y la información de otros administradores de dependencias en el [repositorio central de Maven](https://search.maven.org/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-textanalytics/).

En una ventana de la consola (como cmd, PowerShell o Bash), cree un directorio para la aplicación y vaya a él.

```console
mkdir myapp && cd myapp
```

Ejecute el comando `gradle init` desde el directorio de trabajo. Este comando crea archivos de compilación esenciales para Gradle, incluido un archivo *build.gradle.kts* que se utiliza en tiempo de ejecución para configurar la aplicación.

```console
gradle init --type basic
```

Cuando se le solicite que elija un **DSL**, seleccione **Kotlin**.

## <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Busque *build.gradle.kts* y ábralo con el IDE o el editor de texto que prefiera. A continuación, se copia en esta configuración de compilación. Asegúrese de incluir la biblioteca de cliente en `dependencies`:

```kotlin
plugins {
    java
    application
}
application {
    mainClassName = "main.java.BingCustomSearchSample"
}
repositories {
    mavenCentral()
}
dependencies {
    compile("org.slf4j:slf4j-simple:1.7.25")
    compile("com.microsoft.azure.cognitiveservices:azure-cognitiveservices-customsearch:1.0.2")
}
```

Cree una carpeta para la aplicación de ejemplo. En el directorio de trabajo, ejecute el siguiente comando:

```console
mkdir src/main/java
```

Vaya a la nueva carpeta y cree un archivo llamado *BingCustomSearchSample.java*. Ábralo y agregue las siguientes instrucciones `import`:


[!code-java[import statements](~/cognitive-services-java-sdk-samples/Search/BingCustomSearch/src/main/java/BingCustomSearchSample.java?name=imports)]

Cree una clase llamada `BingCustomSearchSample`.

```java
public class BingCustomSearchSample {
}
```

En la clase, cree un método `main` y una variable para la clave del recurso. Si ha creado la variable de entorno después de haber iniciado la aplicación, cierre y vuelva a abrir el editor, el IDE o el shell que lo ejecuta para acceder a la variable. Los métodos se definirán más adelante.

[!code-java[main method](~/cognitive-services-java-sdk-samples/Search/BingCustomSearch/src/main/java/BingCustomSearchSample.java?name=main)]

## <a name="object-model"></a>Modelo de objetos

El cliente de Bing Custom Search es un objeto [BingCustomSearchAPI](https://docs.microsoft.com/java/api/com.microsoft.azure.cognitiveservices.search.customsearch.bingcustomsearchapi?view=azure-java-stable) que se crea a partir del método [authenticate()](https://docs.microsoft.com/java/api/com.microsoft.azure.cognitiveservices.search.customsearch.bingcustomsearchmanager.authenticate) del objeto [BingCustomSearchManager](https://docs.microsoft.com/java/api/com.microsoft.azure.cognitiveservices.search.customsearch.bingcustomsearchmanager?view=azure-java-stable). Puede enviar una solicitud de búsqueda usando el método [BingCustomInstances.search()](https://docs.microsoft.com/java/api/com.microsoft.azure.cognitiveservices.search.customsearch.bingcustominstances.search?view=azure-java-stable#com_microsoft_azure_cognitiveservices_search_customsearch_BingCustomInstances_search__) del cliente.

La respuesta de la API es un objeto [SearchResponse](https://docs.microsoft.com/java/api/com.microsoft.azure.cognitiveservices.search.customsearch.models.searchresponse?view=azure-java-stable) que contiene información sobre la consulta de búsqueda y los resultados de la búsqueda.

## <a name="code-examples"></a>Ejemplos de código

En estos fragmentos de código se muestra cómo realizar las siguientes tareas con la biblioteca de cliente de Bing Custom Search para Java:

* [Autenticar el cliente](#authenticate-the-client)
* [Obtener los resultados de la búsqueda de la instancia de búsqueda personalizada](#get-search-results-from-your-custom-search-instance)

## <a name="authenticate-the-client"></a>Autenticar el cliente

El método Main debe incluir un objeto [BingCustomSearchManager](https://docs.microsoft.com/java/api/com.microsoft.azure.cognitiveservices.search.customsearch.bingcustomsearchapi?view=azure-java-stable) que toma la clave y llama a su `authenticate()`.

```java
BingCustomSearchAPI client = BingCustomSearchManager.authenticate(subscriptionKey);
```

## <a name="get-search-results-from-your-custom-search-instance"></a>Obtener los resultados de la búsqueda de la instancia de búsqueda personalizada

Use la función [BingCustomInstances.Search()](https://docs.microsoft.com/java/api/com.microsoft.azure.cognitiveservices.search.customsearch.bingcustominstances.search?view=azure-java-stable#com_microsoft_azure_cognitiveservices_search_customsearch_BingCustomInstances_search__) del cliente para enviar una consulta de búsqueda a la instancia personalizada. Establezca el valor de `withCustomConfig` en el identificador de configuración personalizado, o establezca `1` como valor predeterminado. Después de obtener una respuesta de la API, compruebe si se encontraron resultados de la búsqueda. Si es así, obtenga el primer resultado de la búsqueda; para ello, llame a la función `webPages().value().get()` de la respuesta e imprima el nombre del resultado y la dirección URL.

[!code-java[call the custom search API](~/cognitive-services-java-sdk-samples/Search/BingCustomSearch/src/main/java/BingCustomSearchSample.java?name=runSample)]

## <a name="run-the-application"></a>Ejecución de la aplicación

Compile la aplicación con el siguiente comando desde el directorio principal del proyecto:

```console
gradle build
```

Ejecute la aplicación con el objetivo `run`:

```console
gradle run
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una página web de Custom Search](../../tutorials/custom-search-web-page.md)
