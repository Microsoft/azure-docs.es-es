---
title: Enlace de salida de Azure Blob Storage para Azure Functions
description: Aprenda a proporcionar datos de Azure Blob Storage a una instancia de Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 02/13/2020
ms.author: cshoe
ms.custom: devx-track-python
ms.openlocfilehash: d089cb1844356032b956110a7475d839031ef006
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87849835"
---
# <a name="azure-blob-storage-output-binding-for-azure-functions"></a>Enlace de salida de Azure Blob Storage para Azure Functions

El enlace de salida permite modificar y eliminar datos de Azure Blob Storage en una instancia de Azure Functions.

Para obtener información sobre los detalles de instalación y configuración, vea la [información general](./functions-bindings-storage-blob.md).

## <a name="example"></a>Ejemplo

# <a name="c"></a>[C#](#tab/csharp)

El ejemplo siguiente es una [función de C#](functions-dotnet-class-library.md) que utiliza un desencadenador de blobs y dos enlaces de blobs de salida. La función se activa por la creación de un blob de imágenes en el contenedor *sample-images*. Crea copias pequeñas y medianas del blob de imágenes.

```csharp
using System.Collections.Generic;
using System.IO;
using Microsoft.Azure.WebJobs;
using SixLabors.ImageSharp;
using SixLabors.ImageSharp.Formats;
using SixLabors.ImageSharp.PixelFormats;
using SixLabors.ImageSharp.Processing;

public class ResizeImages
{
    [FunctionName("ResizeImage")]
    public static void Run([BlobTrigger("sample-images/{name}")] Stream image,
        [Blob("sample-images-sm/{name}", FileAccess.Write)] Stream imageSmall,
        [Blob("sample-images-md/{name}", FileAccess.Write)] Stream imageMedium)
    {
        IImageFormat format;

        using (Image<Rgba32> input = Image.Load<Rgba32>(image, out format))
        {
            ResizeImage(input, imageSmall, ImageSize.Small, format);
        }

        image.Position = 0;
        using (Image<Rgba32> input = Image.Load<Rgba32>(image, out format))
        {
            ResizeImage(input, imageMedium, ImageSize.Medium, format);
        }
    }

    public static void ResizeImage(Image<Rgba32> input, Stream output, ImageSize size, IImageFormat format)
    {
        var dimensions = imageDimensionsTable[size];

        input.Mutate(x => x.Resize(dimensions.Item1, dimensions.Item2));
        input.Save(output, format);
    }

    public enum ImageSize { ExtraSmall, Small, Medium }

    private static Dictionary<ImageSize, (int, int)> imageDimensionsTable = new Dictionary<ImageSize, (int, int)>() {
        { ImageSize.ExtraSmall, (320, 200) },
        { ImageSize.Small,      (640, 400) },
        { ImageSize.Medium,     (800, 600) }
    };

}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

<!--Same example for input and output. -->

En el ejemplo siguiente se muestran enlaces de entrada y salida de blobs en un archivo *function.json* y código de [script de C# (.csx)](functions-reference-csharp.md) que usa los enlaces. La función realiza una copia de un blob de texto. La función se activa mediante un mensaje de cola que contiene el nombre del blob que se va a copiar. El nuevo blob se denomina *{nombreoriginaldelblob}-Copy*.

En el archivo *function.json*, la propiedad de metadatos `queueTrigger` se utiliza para especificar el nombre del blob en las propiedades de `path`:

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "myOutputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

En la sección de [configuración](#configuration) se explican estas propiedades.

Este es el código de script de C#:

```cs
public static void Run(string myQueueItem, string myInputBlob, out string myOutputBlob, ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed: {myQueueItem}");
    myOutputBlob = myInputBlob;
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

<!--Same example for input and output. -->

En el ejemplo siguiente se muestran enlaces de entrada y salida de blobs en un archivo *function.json* y código de [script de JavaScript](functions-reference-node.md) que usa los enlaces. La función realiza una copia de un blob. La función se activa mediante un mensaje de cola que contiene el nombre del blob que se va a copiar. El nuevo blob se denomina *{nombreoriginaldelblob}-Copy*.

En el archivo *function.json*, la propiedad de metadatos `queueTrigger` se utiliza para especificar el nombre del blob en las propiedades de `path`:

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "myOutputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

En la sección de [configuración](#configuration) se explican estas propiedades.

Este es el código de JavaScript:

```javascript
module.exports = function(context) {
    context.log('Node.js Queue trigger function processed', context.bindings.myQueueItem);
    context.bindings.myOutputBlob = context.bindings.myInputBlob;
    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

<!--Same example for input and output. -->

En el ejemplo siguiente se muestran enlaces de entrada y salida de blobs en un archivo *function.json* y el [código de Python](functions-reference-python.md) que usa los enlaces. La función realiza una copia de un blob. La función se activa mediante un mensaje de cola que contiene el nombre del blob que se va a copiar. El nuevo blob se denomina *{nombreoriginaldelblob}-Copy*.

En el archivo *function.json*, la propiedad de metadatos `queueTrigger` se utiliza para especificar el nombre del blob en las propiedades de `path`:

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "queuemsg",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "inputblob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "outputblob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false,
  "scriptFile": "__init__.py"
}
```

En la sección de [configuración](#configuration) se explican estas propiedades.

Este es el código de Python:

```python
import logging
import azure.functions as func


def main(queuemsg: func.QueueMessage, inputblob: func.InputStream,
         outputblob: func.Out[func.InputStream]):
    logging.info('Python Queue trigger function processed %s', inputblob.name)
    outputblob.set(inputblob)
```

# <a name="java"></a>[Java](#tab/java)

En esta sección se incluyen los ejemplos siguientes:

* [Desencadenador HTTP, uso de OutputBinding](#http-trigger-using-outputbinding-java)
* [Desencadenador de cola, uso del valor devuelto de la función](#queue-trigger-using-function-return-value-java)

#### <a name="http-trigger-using-outputbinding-java"></a>Desencadenador HTTP, uso de OutputBinding (Java)

 El ejemplo siguiente muestra una función de Java que usa la anotación `HttpTrigger` para recibir un parámetro que contiene el nombre de un archivo en un contenedor de Blob Storage. Posteriormente, la anotación `BlobInput` lee el archivo y pasa el contenido a la función como `byte[]`. La anotación `BlobOutput` enlaza a `OutputBinding outputItem`. La función usa este posteriormente para escribir el contenido del blob de entrada en el contenedor de almacenamiento configurado.

```java
  @FunctionName("copyBlobHttp")
  @StorageAccount("Storage_Account_Connection_String")
  public HttpResponseMessage copyBlobHttp(
    @HttpTrigger(name = "req", 
      methods = {HttpMethod.GET}, 
      authLevel = AuthorizationLevel.ANONYMOUS) 
    HttpRequestMessage<Optional<String>> request,
    @BlobInput(
      name = "file", 
      dataType = "binary", 
      path = "samples-workitems/{Query.file}") 
    byte[] content,
    @BlobOutput(
      name = "target", 
      path = "myblob/{Query.file}-CopyViaHttp")
    OutputBinding<String> outputItem,
    final ExecutionContext context) {
      // Save blob to outputItem
      outputItem.setValue(new String(content, StandardCharsets.UTF_8));

      // build HTTP response with size of requested blob
      return request.createResponseBuilder(HttpStatus.OK)
        .body("The size of \"" + request.getQueryParameters().get("file") + "\" is: " + content.length + " bytes")
        .build();
  }
```

#### <a name="queue-trigger-using-function-return-value-java"></a>Desencadenador de cola, uso del valor devuelto de la función (Java)

 El ejemplo siguiente muestra una función de Java que usa la anotación `QueueTrigger` para recibir un mensaje que contiene el nombre de un archivo en un contenedor de Blob Storage. Posteriormente, la anotación `BlobInput` lee el archivo y pasa el contenido a la función como `byte[]`. La anotación `BlobOutput` enlaza al valor devuelto de la función. El entorno de ejecución usa este valor posteriormente para escribir el contenido del blob de entrada en el contenedor de almacenamiento configurado.

```java
  @FunctionName("copyBlobQueueTrigger")
  @StorageAccount("Storage_Account_Connection_String")
  @BlobOutput(
    name = "target", 
    path = "myblob/{queueTrigger}-Copy")
  public String copyBlobQueue(
    @QueueTrigger(
      name = "filename", 
      dataType = "string",
      queueName = "myqueue-items") 
    String filename,
    @BlobInput(
      name = "file", 
      path = "samples-workitems/{queueTrigger}") 
    String content,
    final ExecutionContext context) {
      context.getLogger().info("The content of \"" + filename + "\" is: " + content);
      return content;
  }
```

 En la [biblioteca en tiempo de ejecución de funciones de Java](/java/api/overview/azure/functions/runtime), utilice la anotación `@BlobOutput` en los parámetros de función cuyo valor se escribiría en un objeto del almacenamiento de blobs.  El parámetro type debe ser `OutputBinding<T>`, donde T es cualquier tipo nativo de Java o un POJO.

---

## <a name="attributes-and-annotations"></a>Atributos y anotaciones

# <a name="c"></a>[C#](#tab/csharp)

En las [bibliotecas de clase C#](functions-dotnet-class-library.md), use [BlobAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/dev/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs/BlobAttribute.cs).

El constructor del atributo toma la ruta de acceso al blob y un parámetro `FileAccess` que indica lectura o escritura, tal como se muestra en el ejemplo siguiente:

```csharp
[FunctionName("ResizeImage")]
public static void Run(
    [BlobTrigger("sample-images/{name}")] Stream image,
    [Blob("sample-images-md/{name}", FileAccess.Write)] Stream imageSmall)
{
    ...
}
```

Puede establecer la propiedad `Connection` para especificar la cuenta de almacenamiento que se usará, tal como se muestra en el ejemplo siguiente:

```csharp
[FunctionName("ResizeImage")]
public static void Run(
    [BlobTrigger("sample-images/{name}")] Stream image,
    [Blob("sample-images-md/{name}", FileAccess.Write, Connection = "StorageConnectionAppSetting")] Stream imageSmall)
{
    ...
}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

El script de C# no admite atributos.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

JavaScript no admite atributos.

# <a name="python"></a>[Python](#tab/python)

Python no admite atributos.

# <a name="java"></a>[Java](#tab/java)

El atributo `@BlobOutput` da acceso al blob que desencadenó la función. Si utiliza una matriz de bytes con el atributo, establezca `dataType` en `binary`. Vea el [ejemplo de salida](#example) para más información.

---

Para obtener un ejemplo completo, vea el [ejemplo de salida](#example).

Puede usar el atributo `StorageAccount` para especificar la cuenta de almacenamiento en el nivel de clase, método o parámetro. Para obtener más información, consulte [Desencadenador: atributos](./functions-bindings-storage-blob-trigger.md#attributes-and-annotations).

## <a name="configuration"></a>Configuración

En la siguiente tabla se explican las propiedades de configuración de enlace que se definen en el archivo *function.json* y el atributo `Blob`.

|Propiedad de function.json | Propiedad de atributo |Descripción|
|---------|---------|----------------------|
|**type** | N/D | Se debe establecer en `blob`. |
|**direction** | N/D | Debe establecerse en `out` para un enlace de salida. Las excepciones se indican en la sección [uso](#usage). |
|**name** | N/D | Nombre de la variable que representa el blob en el código de la función.  Se establece en `$return` para hacer referencia al valor devuelto de la función.|
|**path** |**BlobPath** | Ruta de acceso al contenedor de blobs. |
|**connection** |**Connection**| El nombre de una configuración de aplicación que contiene la cadena de conexión de almacenamiento que se usará para este enlace. Si el nombre de la configuración de aplicación comienza con "AzureWebJobs", puede especificar solo el resto del nombre aquí. Por ejemplo, si establece `connection` en "MyStorage", el entorno en tiempo de ejecución de Functions busca una configuración de aplicación denominada "AzureWebJobsMyStorage". Si deja `connection` vacía, el entorno en tiempo de ejecución de Functions usa la cadena de conexión de almacenamiento predeterminada en la configuración de aplicación que se denomina `AzureWebJobsStorage`.<br><br>La cadena de conexión debe ser para una cuenta de almacenamiento de uso general, no una [cuenta de almacenamiento solo de blob](../storage/common/storage-account-overview.md#types-of-storage-accounts).|
|N/D | **Acceder** | Indica si va a leer o escribir. |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Uso

# <a name="c"></a>[C#](#tab/csharp)

[!INCLUDE [functions-bindings-blob-storage-output-usage.md](../../includes/functions-bindings-blob-storage-output-usage.md)]

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

[!INCLUDE [functions-bindings-blob-storage-output-usage.md](../../includes/functions-bindings-blob-storage-output-usage.md)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

En JavaScript, acceda a los datos de blob mediante `context.bindings.<name from function.json>`.

# <a name="python"></a>[Python](#tab/python)

Puede declarar parámetros de función como los siguientes tipos para escribir en el almacenamiento de blobs:

* Cadenas como `func.Out(str)`
* Secuencias como `func.Out(func.InputStream)`

Vea el [ejemplo de salida](#example) para más información.

# <a name="java"></a>[Java](#tab/java)

El atributo `@BlobOutput` da acceso al blob que desencadenó la función. Si utiliza una matriz de bytes con el atributo, establezca `dataType` en `binary`. Vea el [ejemplo de salida](#example) para más información.

---

## <a name="exceptions-and-return-codes"></a>Excepciones y códigos de retorno

| Enlace |  Referencia |
|---|---|
| Blob | [Códigos de error de blobs](/rest/api/storageservices/fileservices/blob-service-error-codes) |
| Blob, tabla, cola |  [Códigos de error de almacenamiento](/rest/api/storageservices/fileservices/common-rest-api-error-codes) |
| Blob, tabla, cola |  [Solución de problemas](/rest/api/storageservices/fileservices/troubleshooting-api-operations) |

## <a name="next-steps"></a>Pasos siguientes

- [Ejecución de una función cuando cambian los datos de Blob Storage](./functions-bindings-storage-blob-trigger.md)
- [Lectura de datos de Azure Blob Storage cuando se ejecuta una función](./functions-bindings-storage-blob-input.md)
