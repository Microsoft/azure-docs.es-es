---
title: Inicio rápido de la biblioteca cliente para Go de Bing Autosuggest
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 05/06/2020
ms.author: aahi
ms.openlocfilehash: c0129ff25f1df492ab6eba9f49add18d5321a3e8
ms.sourcegitcommit: c293217e2d829b752771dab52b96529a5442a190
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/15/2020
ms.locfileid: "88246065"
---
Introducción a la biblioteca cliente de Bing Autosuggest para Go. Siga estos pasos para instalar la biblioteca y probar los ejemplos para realizar tareas básicas.

Use la biblioteca cliente de Bing Autosuggest para Go para obtener sugerencias de búsqueda basadas en cadenas de consulta parciales.

[Documentación de referencia](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/autosuggest) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-go/tree/master/services/cognitiveservices/v2.1/textanalytics) | [Ejemplo de código](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/go/BingAutoSuggest/BingAutoSuggestQuickstart.go)

## <a name="prerequisites"></a>Requisitos previos

* Suscripción a Azure. Si aún no tiene una suscripción a Azure, [puede crear una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services).
* La versión más reciente de [Go](https://golang.org/dl/).

Comience a usar la biblioteca de cliente de Bing Autosuggest mediante la creación de un recurso de Azure. Elija el tipo de recurso que le resulte más adecuado:

[!INCLUDE [cognitive-services-bing-autosuggest-signup-requirements](~/includes/cognitive-services-bing-autosuggest-signup-requirements.md)]

## <a name="create-environment-variables"></a>Creación de variables de entorno

>[!NOTE]
> Los puntos de conexión de los recursos creados después del 1 de julio de 2019 usan el formato de subdominio personalizado que se muestra a continuación. Para más información y para obtener una lista completa de los puntos de conexión regionales, consulte [Nombres de subdominios personalizados para Cognitive Services.](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-custom-subdomains)

Con la clave y el punto de conexión del recurso que ha creado, cree dos variables de entorno para la autenticación:
<!-- replace the below variable names with the names expected in the code sample.-->
* `AUTOSUGGEST_SUBSCRIPTION_KEY`: la clave de recurso para autenticar las solicitudes.
* `AUTOSUGGEST_ENDPOINT`: el punto de conexión del recurso para enviar solicitudes de API. Debería ser parecido a este: `https://<your-custom-subdomain>.api.cognitive.microsoft.com`.

Siga las instrucciones adecuadas para su sistema operativo.
<!-- replace the below endpoint and key examples -->
### <a name="windows"></a>[Windows](#tab/windows)

```console
setx BING_AUTOSUGGEST_SUBSCRIPTION_KEY <replace-with-your-autosuggest-api-key>
setx BING_AUTOSUGGEST_ENDPOINT <replace-with-your-autosuggest-api-endpoint>
```

Después de agregar la variable de entorno, reinicie la ventana de la consola.

### <a name="linux"></a>[Linux](#tab/linux)

```bash
export AUTOSUGGEST_SUBSCRIPTION_KEY=<replace-with-your-autosuggest-api-key>
export AUTOSUGGEST_ENDPOINT=<replace-with-your-autosuggest-api-endpoint>
```

Después de agregar la variable de entorno, ejecute `source ~/.bashrc` desde la ventana de consola para que los cambios surtan efecto.

### <a name="macos"></a>[macOS](#tab/unix)

Edite `.bash_profile` y agregue la variable de entorno:

```bash
export AUTOSUGGEST_SUBSCRIPTION_KEY=<replace-with-your-autosuggest-api-key>
export AUTOSUGGEST_ENDPOINT=<replace-with-your-autosuggest-api-endpoint>
```

Después de agregar la variable de entorno, ejecute `source .bash_profile` desde la ventana de consola para que los cambios surtan efecto.
***

## <a name="create-a-new-go-project"></a>Creación de un nuevo proyecto de Go

En una ventana de consola (cmd, PowerShell, Terminal, Bash), cree una nueva área de trabajo para el proyecto de Go y vaya hasta ella. El área de trabajo contendrá tres carpetas:

* **src**: este directorio contiene código fuente y paquetes. Todos los paquetes instalados con el comando `go get` residirán aquí.
* **pkg**: este directorio contiene objetos de paquete de Go compilados. Todos estos archivos tienen la extensión `.a`.
* **bin**: este directorio contiene los archivos ejecutables binarios que se crean al ejecutar `go install`.

> [!TIP]
> Obtenga más información sobre la estructura de [un área de trabajo de Go](https://golang.org/doc/code.html#Workspaces). En esta guía se incluye información para establecer `$GOPATH` y `$GOROOT`.

Vamos a crear un área de trabajo llamada `my-app` y los subdirectorios necesarios para `src`, `pkg` y `bin`:

```
$ mkdir -p my-app/{src, bin, pkg}  
$ cd my-app
```

## <a name="install-the-client-library-for-go"></a>Instalación de la biblioteca cliente para Go

Instalemos a continuación la biblioteca cliente para Go:

```bash
$ go get -u <library-location-or-url>
```

O, si usa dep, dentro de su repositorio ejecute:

```bash
$ dep ensure -add <library-location-or-url>
```

## <a name="create-your-go-application"></a>Creación de la aplicación de Go

A continuación, vamos a crear un archivo denominado `src/sample-app.go`:

```bash
$ cd src
$ touch sample-app.go
```

Abra `sample-app.go`, agregue el nombre del paquete e importe las siguientes bibliotecas:

```Go
package main

import (
    "context"
    "fmt"
    "github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/autosuggest"
    "github.com/Azure/go-autorest/autorest"
    "log"
    "os"
)
```

Cree una función denominada `main`. A continuación, cree las variables de entorno para la clave y el punto de conexión de Bing Autosuggest:

```go
func main() {
    // Add your Bing Autosuggest subscription key to your environment variables.
    if "" == os.Getenv("BING_AUTOSUGGEST_SUBSCRIPTION_KEY") {
        log.Fatal("Please set/export the environment variable BING_AUTOSUGGEST_SUBSCRIPTION_KEY.")
    }
    // Add your Bing Autosuggest endpoint to your environment variables.
    var subscription_key string = os.Getenv("BING_AUTOSUGGEST_ENDPOINT")
    if "" == os.Getenv("BING_AUTOSUGGEST_ENDPOINT") {
        log.Fatal("Please set/export the environment variable BING_AUTOSUGGEST_ENDPOINT.")
    }
    var endpoint string = os.Getenv("BING_AUTOSUGGEST_ENDPOINT")
}
```

## <a name="code-examples"></a>Ejemplos de código

En estos ejemplos de código se muestra cómo realizar tareas básicas con la biblioteca cliente de Bing Autosuggest para Go:

* [Autenticar el cliente](#authenticate-the-client)
* [Envío de una solicitud de API](#send-an-api-request)

### <a name="authenticate-the-client"></a>Autenticar el cliente

> [!NOTE]
> En este inicio rápido se da por supuesto que ha [creado una variable de entorno](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account#configure-an-environment-variable-for-authentication) para la clave de Bing Autosuggest, denominada `BING_AUTOSUGGEST_SUBSCRIPTION_KEY`, y una para el punto de conexión denominada `BING_AUTOSUGGEST_ENDPOINT`.

En la función `main()`, cree una instancia de un cliente con la clave y el punto de conexión.

```go
// Get the context, which is required by the SDK methods.
ctx := context.Background()

client := autosuggest.New()
// Set the subscription key on the client.
client.Authorizer = autorest.NewCognitiveServicesAuthorizer(subscription_key)
client.Endpoint = endpoint
```

### <a name="send-an-api-request"></a>Envío de una solicitud de API

En el mismo método, utilice el método [AutoSuggestMethodAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.search.autosuggest.autosuggestclientextensions.autosuggestmethodasync?view=azure-dotnet#Microsoft_Azure_CognitiveServices_Search_AutoSuggest_AutoSuggestClientExtensions_AutoSuggestMethodAsync_Microsoft_Azure_CognitiveServices_Search_AutoSuggest_IAutoSuggestClient_System_String_System_String_System_String_System_String_System_String_System_String_System_String_System_String_System_String_System_String_System_String_System_Collections_Generic_IList_System_String__System_Threading_CancellationToken_) para enviar una consulta a Bing. Itere sobre la respuesta de [Suggestions](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.search.autosuggest.models.suggestions?view=azure-dotnet) e imprima la primera sugerencia.

```Go
// This should return the query suggestion "xbox."
result, err := client.AutoSuggest (ctx, "xb", "", "", "", "", "", "", "", "", "", "", []autosuggest.ResponseFormat{"Json"})
if nil != err {
    log.Fatal(err)
}

groups := *result.SuggestionGroups
if len(groups) > 0 {
    group, _ := groups[0].AsSuggestionsSuggestionGroup()
    fmt.Printf ("First suggestion group: %s\n", (*group).Name)
    suggestions := *(*group).SearchSuggestions
    if len(suggestions) > 0 {
        fmt.Println("First suggestion:")
        fmt.Printf("Query: %s\n", *suggestions[0].Query)
        fmt.Printf("Display text: %s\n", *suggestions[0].DisplayText)
    } else {
        fmt.Println("No suggestions found in this group.")
    }
} else {
    fmt.Println("No suggestions found.")
}
```

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación Go con el comando `go run [arguments]` desde el directorio de la aplicación.

```Go
go run sample-app.go
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Eliminación de un grupo de recursos en Azure Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources).
* [Eliminación de un grupo de recursos en la CLI de Azure](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial de Bing Autosuggest](../../tutorials/autosuggest.md)

## <a name="see-also"></a>Consulte también

- [¿Qué es Bing Autosuggest?](../../get-suggested-search-terms.md)
- [Referencia de Bing Autosuggest API v7](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-autosuggest-api-v7-reference)
