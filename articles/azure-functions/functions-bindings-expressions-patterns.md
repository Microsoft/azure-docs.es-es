---
title: Patrones y expresiones de enlace de Azure Functions
description: Aprenda a crear diferentes expresiones de enlace de Azure Functions según patrones comunes.
author: craigshoemaker
ms.topic: reference
ms.custom: devx-track-csharp
ms.date: 02/18/2019
ms.author: cshoe
ms.openlocfilehash: b4e2b5afd7742791218394422d00ee8ee46cb23a
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88212609"
---
# <a name="azure-functions-binding-expression-patterns"></a>Patrones de expresiones de enlace de Azure Functions

Una de las características más eficaces de los [desencadenadores y enlaces](./functions-triggers-bindings.md) son las *expresiones de enlace*. En el archivo *function.json*, y en el código y en los parámetros de función, puede usar expresiones que se resuelvan como valores procedentes de diversos orígenes.

La mayoría de las expresiones se identifican encerrándolas entre llaves. Por ejemplo, en una función de desencadenador de cola, `{queueTrigger}` se resuelve como el texto del mensaje de cola. Si la propiedad `path` de un enlace de blob de salida es `container/{queueTrigger}` y la función se desencadena mediante un mensaje de cola `HelloWorld`, se crea un blob denominado `HelloWorld`.

Tipos de expresiones de enlace

* [Configuración de aplicación](#binding-expressions---app-settings)
* [Nombre de archivo de desencadenador](#trigger-file-name)
* [Metadatos de desencadenador](#trigger-metadata)
* [Cargas JSON](#json-payloads)
* [GUID nuevos](#create-guids)
* [Fecha y hora actuales](#current-time)

## <a name="binding-expressions---app-settings"></a>Expresiones de enlace: configuración de aplicación

Como procedimiento recomendado, los secretos y las cadenas de conexión deberían administrarse mediante los ajustes de la aplicación, en lugar de archivos de configuración. De este modo, se limita el acceso a estos secretos y resulta seguro almacenar archivos como *function.json* en repositorios de control de código fuente públicos.

Los ajustes de la aplicación también son útiles cuando se desea cambiar la configuración en función del entorno. Por ejemplo, en un entorno de prueba, es posible que quiera supervisar una cola o un contenedor de almacenamiento de blobs diferente.

Las expresiones de enlace de configuración de aplicación se identifican de forma diferente de otras expresiones de enlace: se encierran entre símbolos de porcentaje en lugar de entre llaves. Por ejemplo, si la ruta de acceso del enlace de salida de blob es `%Environment%/newblob.txt` y el valor de configuración de aplicación `Environment` es `Development`, se creará un blob en el contenedor `Development`.

Cuando una función se ejecuta localmente, los valores de configuración de aplicación proceden del archivo *local.settings.json*.

> [!NOTE]
> La propiedad `connection` de los desencadenadores y enlaces es un caso especial, ya que resuelve automáticamente los valores como configuración de aplicación, sin símbolos de porcentaje. 

En el ejemplo siguiente, aparece un desencadenador de Azure Queue Storage que se sirve del valor de configuración de la aplicación `%input-queue-name%` para definir la cola que se desencadenará.

```json
{
  "bindings": [
    {
      "name": "order",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "%input-queue-name%",
      "connection": "MY_STORAGE_ACCT_APP_SETTING"
    }
  ]
}
```

Puede usar el mismo enfoque con las bibliotecas de clases:

```csharp
[FunctionName("QueueTrigger")]
public static void Run(
    [QueueTrigger("%input-queue-name%")]string myQueueItem, 
    ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed: {myQueueItem}");
}
```

## <a name="trigger-file-name"></a>Nombre de archivo de desencadenador

El patrón `path` de un desencadenador de blob puede ser uno que permite hacer referencia al nombre del blob de desencadenamiento en otros enlaces y el código de función. El patrón también puede incluir criterios de filtrado que especifiquen qué blobs pueden desencadenar una invocación de función.

Por ejemplo, en el siguiente enlace de desencadenador de blob, el patrón `path` es `sample-images/{filename}`, lo que crea una expresión de enlace denominada `filename`:

```json
{
  "bindings": [
    {
      "name": "image",
      "type": "blobTrigger",
      "path": "sample-images/{filename}",
      "direction": "in",
      "connection": "MyStorageConnection"
    },
    ...
```

Luego, la expresión `filename` puede utilizarse en un enlace de salida para especificar el nombre del blob que se va a crear:

```json
    ...
    {
      "name": "imageSmall",
      "type": "blob",
      "path": "sample-images-sm/{filename}",
      "direction": "out",
      "connection": "MyStorageConnection"
    }
  ],
}
```

El código de función tiene acceso a este mismo valor usando `filename` como nombre de parámetro:

```csharp
// C# example of binding to {filename}
public static void Run(Stream image, string filename, Stream imageSmall, ILogger log)  
{
    log.LogInformation($"Blob trigger processing: {filename}");
    // ...
} 
```

<!--TODO: add JavaScript example -->
<!-- Blocked by bug https://github.com/Azure/Azure-Functions/issues/248 -->

La misma capacidad para usar patrones y expresiones de enlace se aplica a los atributos de las bibliotecas de clases. En el ejemplo siguiente, los parámetros del constructor de atributo son los mismos valores `path` que los ejemplos de *function.json* anteriores: 

```csharp
[FunctionName("ResizeImage")]
public static void Run(
    [BlobTrigger("sample-images/{filename}")] Stream image,
    [Blob("sample-images-sm/{filename}", FileAccess.Write)] Stream imageSmall,
    string filename,
    ILogger log)
{
    log.LogInformation($"Blob trigger processing: {filename}");
    // ...
}

```

También puede crear expresiones para partes del nombre de archivo. En el ejemplo siguiente, la función se desencadena solo en los nombres de archivo que coinciden con un patrón: `anyname-anyfile.csv`

```json
{
    "name": "myBlob",
    "type": "blobTrigger",
    "direction": "in",
    "path": "testContainerName/{date}-{filetype}.csv",
    "connection": "OrderStorageConnection"
}
```

Para obtener más información sobre cómo usar expresiones y patrones en la cadena de ruta de acceso de blob, consulte la [referencia de enlace de blob de almacenamiento](functions-bindings-storage-blob.md).

## <a name="trigger-metadata"></a>Metadatos de desencadenador

Además de la carga de datos que proporciona un desencadenador (como el contenido del mensaje de la cola que desencadenó una función), muchos desencadenadores facilitan valores de metadatos adicionales. Estos valores pueden usarse como parámetros de entrada en C# y F# o como propiedades en el objeto `context.bindings` de JavaScript. 

Por ejemplo, un desencadenador de Azure Queue Storage admite las siguientes propiedades:

* QueueTrigger (desencadena el contenido del mensaje si hay una cadena válida)
* DequeueCount
* ExpirationTime
* Identificador
* InsertionTime
* NextVisibleTime
* PopReceipt

A estos valores de metadatos se accede en las propiedades del archivo *function.json*. Por ejemplo, supongamos que usa un desencadenador de cola y el mensaje de la cola contiene el nombre de un blob que desea leer. En el archivo *function.json*, puede usar la propiedad de metadatos `queueTrigger` en la propiedad de blob `path`, como se muestra en el ejemplo siguiente:

```json
  "bindings": [
    {
      "name": "myQueueItem",
      "type": "queueTrigger",
      "queueName": "myqueue-items",
      "connection": "MyStorageConnection",
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "direction": "in",
      "connection": "MyStorageConnection"
    }
  ]
```

Los detalles sobre las propiedades de metadatos de cada desencadenador se describen en el artículo de referencia correspondiente. Para un ejemplo, consulte la sección acerca de los [metadatos de los desencadenadores de cola](functions-bindings-storage-queue-trigger.md#message-metadata). También podrá encontrar documentación en la pestaña **Integrar** del portal, en la sección **Documentación**, debajo del área de configuración de enlaces.  

## <a name="json-payloads"></a>Cargas JSON

Cuando una carga de desencadenador es JSON, puede hacer referencia a sus propiedades en la configuración de otros enlaces de la misma función y del código de función.

El siguiente ejemplo muestra el archivo *function.json* de una función de webhook que recibe el nombre de un blob en JSON: `{"BlobName":"HelloWorld.txt"}`. Un enlace de entrada de blob lee el blob y el enlace de salida HTTP devuelve el contenido del blob en la respuesta HTTP. Tenga en cuenta que el enlace de entrada de blob obtiene el nombre del blob haciendo referencia directamente a la propiedad `BlobName` (`"path": "strings/{BlobName}"`).

```json
{
  "bindings": [
    {
      "name": "info",
      "type": "httpTrigger",
      "direction": "in",
      "webHookType": "genericJson"
    },
    {
      "name": "blobContents",
      "type": "blob",
      "direction": "in",
      "path": "strings/{BlobName}",
      "connection": "AzureWebJobsStorage"
    },
    {
      "name": "res",
      "type": "http",
      "direction": "out"
    }
  ]
}
```

Para que funcione en C# y F#, necesita una clase que defina los campos que se deserializarán, como en el ejemplo siguiente:

```csharp
using System.Net;
using Microsoft.Extensions.Logging;

public class BlobInfo
{
    public string BlobName { get; set; }
}
  
public static HttpResponseMessage Run(HttpRequestMessage req, BlobInfo info, string blobContents, ILogger log)
{
    if (blobContents == null) {
        return req.CreateResponse(HttpStatusCode.NotFound);
    } 

    log.LogInformation($"Processing: {info.BlobName}");

    return req.CreateResponse(HttpStatusCode.OK, new {
        data = $"{blobContents}"
    });
}
```

En JavaScript, la deserialización de JSON se lleva a cabo de manera automática.

```javascript
module.exports = function (context, info) {
    if ('BlobName' in info) {
        context.res = {
            body: { 'data': context.bindings.blobContents }
        }
    }
    else {
        context.res = {
            status: 404
        };
    }
    context.done();
}
```

### <a name="dot-notation"></a>Notación de puntos

Si algunas de las propiedades de la carga útil JSON son objetos con propiedades, puede hacer referencia a ellos directamente mediante la notación de puntos. Por ejemplo, suponga que su carga útil JSON tiene el siguiente aspecto:

```json
{
  "BlobName": {
    "FileName":"HelloWorld",
    "Extension":"txt"
  }
}
```

Puede hacer referencia directamente a `FileName` como `BlobName.FileName`. Con este formato JSON, así sería el aspecto de la propiedad `path` del ejemplo anterior:

```json
"path": "strings/{BlobName.FileName}.{BlobName.Extension}",
```

En C#, necesitaría dos clases:

```csharp
public class BlobInfo
{
    public BlobName BlobName { get; set; }
}
public class BlobName
{
    public string FileName { get; set; }
    public string Extension { get; set; }
}
```

## <a name="create-guids"></a>Creación de los identificadores únicos globales

La expresión de enlace `{rand-guid}` crea un identificador único global. La siguiente ruta de acceso de blob en un archivo `function.json` crea un blob con un nombre como *50710cb5-84b9-4d87-9d83-a03d6976a682.txt*.

```json
{
  "type": "blob",
  "name": "blobOutput",
  "direction": "out",
  "path": "my-output-container/{rand-guid}.txt"
}
```

## <a name="current-time"></a>Hora actual

La expresión de enlace `DateTime` se resuelve como `DateTime.UtcNow`. La siguiente ruta de acceso de blob en un archivo `function.json` crea un blob con un nombre como *2018-02-16T17-59-55Z.txt*.

```json
{
  "type": "blob",
  "name": "blobOutput",
  "direction": "out",
  "path": "my-output-container/{DateTime}.txt"
}
```
## <a name="binding-at-runtime"></a>Enlace en tiempo de ejecución

En C# y otros lenguajes .NET, puede usar un patrón de enlace imperativo, en contraposición a los enlaces declarativos de *function.json* y los atributos. Los enlaces imperativos resultan útiles cuando los parámetros de enlace tienen que calcularse en tiempo de ejecución, en lugar de en el tiempo de diseño. Para obtener más información, consulte la [referencia para desarrolladores de C#](functions-dotnet-class-library.md#binding-at-runtime) o la [referencia para desarrolladores de scripts de C#](functions-reference-csharp.md#binding-at-runtime).

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Uso del valor devuelto de una función de Azure](./functions-bindings-return-value.md)
