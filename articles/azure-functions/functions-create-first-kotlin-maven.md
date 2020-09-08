---
title: Creación de la primera función en Azure con Kotlin y Maven
description: Cree y publique una función desencadenada por HTTP en Azure con Kotlin y Maven.
author: dglover
ms.service: azure-functions
ms.topic: quickstart
ms.date: 03/25/2020
ms.author: dglover
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 526287feb93052d10428971dfd8f660d4265c831
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89071268"
---
# <a name="quickstart-create-your-first-function-with-kotlin-and-maven"></a>Inicio rápido: Creación de la primera función con Kotlin y Maven

Este artículo le guía por el uso de la herramienta de la línea de comandos de Maven para compilar y publicar un proyecto de función de Kotlin en Azure Functions. Cuando haya terminado, el código de la función se ejecuta en el [Plan de consumo](functions-scale.md#consumption-plan) en Azure y puede activarse mediante una solicitud HTTP.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

Para desarrollar funciones con Kotlin, debe tener instalado lo siguiente:

- [Kit para desarrolladores de Java](https://aka.ms/azure-jdks), versión 8
- [Apache Maven](https://maven.apache.org), versión 3.0 o posterior
- [CLI de Azure](/cli/azure)
- [Azure Functions Core Tools](./functions-run-local.md#v2), versión 2.6.666 u otra posterior

> [!IMPORTANT]
> La variable de entorno JAVA_HOME se debe establecer en la ubicación de instalación del JDK para completar esta guía de inicio rápido.

## <a name="generate-a-new-functions-project"></a>Generación de un nuevo proyecto de Functions

En una carpeta vacía, ejecute el comando siguiente para generar el proyecto de Functions desde un [arquetipo Maven](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html).

# <a name="bash"></a>[bash](#tab/bash)
```bash
mvn archetype:generate \
    -DarchetypeGroupId=com.microsoft.azure \
    -DarchetypeArtifactId=azure-functions-kotlin-archetype
```

> [!NOTE]
> Si experimenta problemas al ejecutar el comando, eche un vistazo a qué versión de `maven-archetype-plugin` se usa. Dado que se ejecuta el comando en un directorio vacío sin ningún archivo `.pom`, podría estar intentando usar un complemento de la versión anterior de `~/.m2/repository/org/apache/maven/plugins/maven-archetype-plugin` si actualizó desde una versión anterior de Maven. Si es así, intente eliminar el directorio `maven-archetype-plugin` y vuelva a ejecutar el comando.

# <a name="powershell"></a>[PowerShell](#tab/powershell)
```powershell
mvn archetype:generate `
    "-DarchetypeGroupId=com.microsoft.azure" `
    "-DarchetypeArtifactId=azure-functions-kotlin-archetype"
```

# <a name="cmd"></a>[Cmd](#tab/cmd)
```cmd
mvn archetype:generate ^
    -DarchetypeGroupId=com.microsoft.azure ^
    -DarchetypeArtifactId=azure-functions-kotlin-archetype
```
---

Maven le pide que especifique los valores necesarios para terminar de generar el proyecto. Para conocer los valores _groupId_, _artifactId_ y _version_, consulte la referencia sobre las [convenciones de nomenclatura de Maven](https://maven.apache.org/guides/mini/guide-naming-conventions.html). El valor _appName_ debe ser único en todo Azure, por lo que Maven genera un nombre de aplicación en función del valor _artifactId_ escrito anteriormente como valor predeterminado. El valor _packageName_ determina el paquete de Kotlin para el código de función generado.

Los identificadores `com.fabrikam.functions` y `fabrikam-functions` siguientes se utilizan como ejemplo y para facilitar la lectura de los pasos posteriores de esta guía de inicio rápido. En este paso, se recomienda especificar los valores propios para Maven.

<pre>
[INFO] Parameter: groupId, Value: com.fabrikam.function
[INFO] Parameter: artifactId, Value: fabrikam-function
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] Parameter: package, Value: com.fabrikam.function
[INFO] Parameter: packageInPathFormat, Value: com/fabrikam/function
[INFO] Parameter: appName, Value: fabrikam-function-20190524171507291
[INFO] Parameter: resourceGroup, Value: java-functions-group
[INFO] Parameter: package, Value: com.fabrikam.function
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] Parameter: groupId, Value: com.fabrikam.function
[INFO] Parameter: appRegion, Value: westus
[INFO] Parameter: artifactId, Value: fabrikam-function
</pre>

Maven crea los archivos del proyecto en una carpeta nueva llamada _artifactId_, en este ejemplo `fabrikam-functions`. El código generado listo para ejecutarse del proyecto es una sencilla función [desencadenada por HTTP](./functions-bindings-http-webhook.md) que devuelve el cuerpo de la solicitud:

```kotlin
class Function {

    /**
     * This function listens at endpoint "/api/HttpTrigger-Java". Two ways to invoke it using "curl" command in bash:
     * 1. curl -d "HTTP Body" {your host}/api/HttpTrigger-Java&code={your function key}
     * 2. curl "{your host}/api/HttpTrigger-Java?name=HTTP%20Query&code={your function key}"
     * Function Key is not needed when running locally, it is used to invoke function deployed to Azure.
     * More details: https://aka.ms/functions_authorization_keys
     */
    @FunctionName("HttpTrigger-Java")
    fun run(
            @HttpTrigger(
                    name = "req",
                    methods = [HttpMethod.GET, HttpMethod.POST],
                    authLevel = AuthorizationLevel.FUNCTION) request: HttpRequestMessage<Optional<String>>,
            context: ExecutionContext): HttpResponseMessage {

        context.logger.info("HTTP trigger processed a ${request.httpMethod.name} request.")

        val query = request.queryParameters["name"]
        val name = request.body.orElse(query)

        name?.let {
            return request
                    .createResponseBuilder(HttpStatus.OK)
                    .body("Hello, $name!")
                    .build()
        }

        return request
                .createResponseBuilder(HttpStatus.BAD_REQUEST)
                .body("Please pass a name on the query string or in the request body")
                .build()
    }
}
```

## <a name="run-the-function-locally"></a>Ejecución local de la función

Cambie de directorio a la carpeta de proyecto que acaba de crear y compile y ejecute la función con Maven:

```
cd fabrikam-function
mvn clean package
mvn azure-functions:run
```

> [!NOTE]
> Si experimenta esta excepción: `javax.xml.bind.JAXBException` con Java 9, consulte la solución alternativa en [GitHub](https://github.com/jOOQ/jOOQ/issues/6477).

Esta salida se ve cuando la función se ejecuta localmente en el sistema y está lista para responder a las solicitudes HTTP:

<pre>
Now listening on: http://0.0.0.0:7071
Application started. Press Ctrl+C to shut down.

Http Functions:

        HttpTrigger-Java: [GET,POST] http://localhost:7071/api/HttpTrigger-Java
</pre>

Desencadene la función desde la línea de comandos, para lo que debe usar curl en una ventana de terminal nueva:

```
curl -w '\n' -d LocalFunction http://localhost:7071/api/HttpTrigger-Java
```

<pre>
Hello LocalFunction!
</pre>

Use `Ctrl-C` en el terminal para detener el código de función.

## <a name="deploy-the-function-to-azure"></a>Implementación de la función en Azure

El proceso de implementación en Azure Functions usa las credenciales de cuenta desde la CLI de Azure. [Inicie sesión con la CLI de Azure](/cli/azure/authenticate-azure-cli?view=azure-cli-latest) antes de continuar.

```azurecli
az login
```

Implemente el código en una aplicación de función nueva `azure-functions:deploy` con el destino de Maven.

> [!NOTE]
> Cuando se usa Visual Studio Code para implementar la aplicación de función, recuerde que debe elegir una suscripción de pago o se producirá un error. Puede ver la suscripción en el lado izquierdo del IDE.

```
mvn azure-functions:deploy
```

Cuando se complete la implementación, verá la dirección URL que puede usar para acceder a la aplicación de función de Azure:

<pre>
[INFO] Successfully deployed Function App with package.
[INFO] Deleting deployment package from Azure Storage...
[INFO] Successfully deleted deployment package fabrikam-function-20170920120101928.20170920143621915.zip
[INFO] Successfully deployed Function App at https://fabrikam-function-20170920120101928.azurewebsites.net
[INFO] ------------------------------------------------------------------------
</pre>

Pruebe la aplicación de función que se ejecuta en Azure mediante `cURL`. Es preciso que cambie dirección URL del ejemplo siguiente para coincida con la implementada para su propia aplicación de función en el paso anterior.

> [!NOTE]
> Asegúrese de establecer los **derechos de acceso** en `Anonymous`. Al elegir el nivel predeterminado de `Function`, tiene que presentar la [tecla de función](functions-bindings-http-webhook-trigger.md#authorization-keys) en las solicitudes para acceder al punto de conexión de la función.

```
curl -w '\n' https://fabrikam-function-20170920120101928.azurewebsites.net/api/HttpTrigger-Java -d AzureFunctions
```

```Output
Hello AzureFunctions!
```

## <a name="make-changes-and-redeploy"></a>Cambios y reimplementación

Edite el archivo de código fuente `src/main.../Function.java` en el proyecto generado para modificar el texto que devuelve la aplicación de función. Cambie esta línea:

```kotlin
return request
        .createResponseBuilder(HttpStatus.OK)
        .body("Hello, $name!")
        .build()
```

Para el código siguiente:

```kotlin
return request
        .createResponseBuilder(HttpStatus.OK)
        .body("Hi, $name!")
        .build()
```

Guarde los cambios y repita la implementación mediante la ejecución de `azure-functions:deploy` desde el terminal como antes. La aplicación de función se actualizará y esta solicitud:

```
curl -w '\n' -d AzureFunctionsTest https://fabrikam-functions-20170920120101928.azurewebsites.net/api/HttpTrigger-Java
```

Verá la salida actualizada:

<pre>
Hi, AzureFunctionsTest
</pre>


## <a name="reference-bindings"></a>Enlaces de referencia

Para trabajar con [desencadenadores y enlaces de Functions](functions-triggers-bindings.md) que no sean los desencadenadores HTTP y de temporizador, debe instalar las extensiones de enlace. Aunque no es necesario en este artículo, deberá saber cómo habilitar las extensiones al trabajar con otros tipos de enlaces.

[!INCLUDE [functions-extension-bundles](../../includes/functions-extension-bundles.md)]

## <a name="next-steps"></a>Pasos siguientes

Ha creado una aplicación de función de Kotlin con un desencadenador HTTP simple y la ha implementado en Azure Functions.

- Revise la [guía para desarrolladores de Functions en Java](functions-reference-java.md) para más información sobre cómo desarrollar funciones de Java y Kotlin.
- Agregue funciones adicionales con desencadenadores distintos en el proyecto con el destino de Maven `azure-functions:add`.
- Escriba y depure localmente las funciones con [Visual Studio Code](https://code.visualstudio.com/docs/java/java-azurefunctions), [IntelliJ](functions-create-maven-intellij.md) y [Eclipse](functions-create-maven-eclipse.md). 
- Depure las funciones implementadas en Azure con Visual Studio Code. Consulte la documentación sobre las [aplicaciones Java sin servidor](https://code.visualstudio.com/docs/java/java-serverless#_remote-debug-functions-running-in-the-cloud) de Visual Studio Code para instrucciones.
