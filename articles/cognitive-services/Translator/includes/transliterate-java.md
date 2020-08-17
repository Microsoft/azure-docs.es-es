---
author: erhopf
ms.service: cognitive-services
ms.topic: include
ms.date: 08/06/2019
ms.custom: devx-track-java
ms.author: erhopf
ms.openlocfilehash: baf74969bdec495fad201a5f489e93edfabf53b6
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87374743"
---
[!INCLUDE [Prerequisites](prerequisites-java.md)]

[!INCLUDE [Set up and use environment variables](setup-env-variables.md)]

## <a name="initialize-a-project-with-gradle"></a>Inicialización de un proyecto con Gradle

Comencemos por crear un directorio de trabajo para este proyecto. Desde la línea de comandos (o terminal), ejecute este comando:

```console
mkdir transliterate-sample
cd transliterate-sample
```

A continuación, va a inicializar un proyecto de Gradle. Este comando creará archivos de compilación esenciales para Gradle, el más importante, el `build.gradle.kts`, que se utiliza en tiempo de ejecución para crear y configurar la aplicación. Ejecute este comando desde el directorio de trabajo:

```console
gradle init --type basic
```

Cuando se le solicite que elija un **DSL**, seleccione **Kotlin**.

## <a name="configure-the-build-file"></a>Configuración del archivo de compilación

Localice `build.gradle.kts` y ábralo con el IDE o editor de texto favorito. A continuación, se copia en esta configuración de compilación:

```
plugins {
    java
    application
}
application {
    mainClassName = "Transliterate"
}
repositories {
    mavenCentral()
}
dependencies {
    compile("com.squareup.okhttp:okhttp:2.5.0")
    compile("com.google.code.gson:gson:2.8.5")
}
```

Tenga en cuenta que este ejemplo tiene dependencias en OkHttp para las solicitudes HTTP, y Gson para administrar y analizar JSON. Si desea obtener más información acerca de las configuraciones de compilación, consulte [Creating New Gradle Builds](https://guides.gradle.org/creating-new-gradle-builds/) (Creación de nuevas compilaciones de Gradle).

## <a name="create-a-java-file"></a>Creación de un archivo Java

Vamos a crear una carpeta para la aplicación de ejemplo. En el directorio de trabajo, ejecute:

```console
mkdir -p src\main\java
```

A continuación, en esta carpeta, cree un archivo denominado `Transliterate.java`.

## <a name="import-required-libraries"></a>Importación de bibliotecas necesarias

Abra `Transliterate.java` y agregue las siguientes instrucciones de importación:

```java
import java.io.*;
import java.net.*;
import java.util.*;
import com.google.gson.*;
import com.squareup.okhttp.*;
```


## <a name="define-variables"></a>Definición de variables

En primer lugar, deberá crear una clase pública para el proyecto:

```java
public class Transliterate {
  // All project code goes here...
}
```

Agregue estas líneas a la clase `Transliterate`. Primero, se lee la clave de suscripción y el punto de conexión de las variables de entorno. A continuación, observe que junto con la `api-version`, se han anexado dos parámetros adicionales a `url`. Estos parámetros se utilizan para establecer el idioma de entrada y los scripts para la transliteración. En este ejemplo, se ha establecido en japonés (`jpan`) y latín (`latn`). 

```java
private static String subscriptionKey = System.getenv("TRANSLATOR_TEXT_SUBSCRIPTION_KEY");
private static String endpoint = System.getenv("TRANSLATOR_TEXT_ENDPOINT");
String url = endpoint + "/transliterate?api-version=3.0&language=ja&fromScript=jpan&toScript=latn";
```
Si usa una suscripción a varios servicios de Cognitive Services, también debe incluir `Ocp-Apim-Subscription-Region` en los parámetros de la solicitud. [Más información sobre la autenticación con la suscripción a varios servicios](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference#authentication).

## <a name="create-a-client-and-build-a-request"></a>Creación de un cliente y compilación de una solución

Agregue esta línea a la clase `Transliterate` para crear una instancia de `OkHttpClient`:

```java
// Instantiates the OkHttpClient.
OkHttpClient client = new OkHttpClient();
```

A continuación, vamos a crear la solicitud POST. No dude en cambiar el texto para la transliteración.

```java
// This function performs a POST request.
public String Post() throws IOException {
    MediaType mediaType = MediaType.parse("application/json");
    RequestBody body = RequestBody.create(mediaType,
            "[{\n\t\"Text\": \"こんにちは\"\n}]");
    Request request = new Request.Builder()
            .url(url).post(body)
            .addHeader("Ocp-Apim-Subscription-Key", subscriptionKey)
            .addHeader("Content-type", "application/json").build();
    Response response = client.newCall(request).execute();
    return response.body().string();
}
```

## <a name="create-a-function-to-parse-the-response"></a>Creación de una función para analizar la respuesta

Esta sencilla función analiza y embellece la respuesta JSON del servicio Traductor.

```java
// This function prettifies the json response.
public static String prettify(String json_text) {
    JsonParser parser = new JsonParser();
    JsonElement json = parser.parse(json_text);
    Gson gson = new GsonBuilder().setPrettyPrinting().create();
    return gson.toJson(json);
}
```

## <a name="put-it-all-together"></a>Colocación de todo junto

El último paso consiste en realizar una solicitud y obtener una respuesta. Agregue estas líneas al proyecto:

```java
public static void main(String[] args) {
    try {
        Transliterate transliterateRequest = new Transliterate();
        String response = transliterateRequest.Post();
        System.out.println(prettify(response));
    } catch (Exception e) {
        System.out.println(e);
    }
}
```

## <a name="run-the-sample-app"></a>Ejecutar la aplicación de ejemplo

Eso es todo, ya está listo para ejecutar la aplicación de ejemplo. Desde la línea de comandos (o sesión de terminal), navegue hasta la raíz del directorio de trabajo y ejecute:

```console
gradle build
```

Cuando la compilación se complete, ejecute lo siguiente:

```console
gradle run
```

## <a name="sample-response"></a>Respuesta de muestra

```json
[
  {
    "text": "konnichiwa",
    "script": "latn"
  }
]
```

## <a name="next-steps"></a>Pasos siguientes

Eche un vistazo a la referencia de API para comprender todo lo que puede hacer con Traductor.

> [!div class="nextstepaction"]
> [Referencia de API](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference)
