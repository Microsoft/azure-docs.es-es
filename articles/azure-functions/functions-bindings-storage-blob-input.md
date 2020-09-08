---
title: Enlace de entrada de Azure Blob Storage para Azure Functions
description: Aprenda a proporcionar datos de Azure Blob Storage a una instancia de Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 02/13/2020
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: de590af49e39832a52bf733319b86300ecab2a34
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88213292"
---
# <a name="azure-blob-storage-input-binding-for-azure-functions"></a>Enlace de entrada de Azure Blob Storage para Azure Functions

El enlace de entrada permite leer datos de Blob Storage como entrada para una instancia de Azure Functions.

Para obtener información sobre los detalles de instalación y configuración, consulte [Introducción](./functions-bindings-storage-blob.md).

## <a name="example"></a>Ejemplo

# <a name="c"></a>[C#](#tab/csharp)

El ejemplo siguiente es una [función de C#](functions-dotnet-class-library.md) que utiliza un desencadenador de cola y un enlaces de blobs de entrada. El mensaje de cola contiene el nombre del blob y la función registra el tamaño del blob.

```csharp
[FunctionName("BlobInput")]
public static void Run(
    [QueueTrigger("myqueue-items")] string myQueueItem,
    [Blob("samples-workitems/{queueTrigger}", FileAccess.Read)] Stream myBlob,
    ILogger log)
{
    log.LogInformation($"BlobInput processed blob\n Name:{myQueueItem} \n Size: {myBlob.Length} bytes");
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
      "dataType": "binary",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "$return",
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


def main(queuemsg: func.QueueMessage, inputblob: func.InputStream) -> func.InputStream:
    logging.info('Python Queue trigger function processed %s', inputblob.name)
    return inputblob
```

# <a name="java"></a>[Java](#tab/java)

En esta sección se incluyen los ejemplos siguientes:

* [Desencadenador de HTTP, búsqueda de nombre de blob en la cadena de consulta](#http-trigger-look-up-blob-name-from-query-string)
* [Desencadenador de cola, recepción del nombre del blob de la cola de mensajes](#queue-trigger-receive-blob-name-from-queue-message)

#### <a name="http-trigger-look-up-blob-name-from-query-string"></a>Desencadenador de HTTP, búsqueda de nombre de blob en la cadena de consulta

 El ejemplo siguiente muestra una función de Java que usa la anotación `HttpTrigger` para recibir un parámetro que contiene el nombre de un archivo en un contenedor de Blob Storage. Posteriormente, la anotación `BlobInput` lee el archivo y pasa el contenido a la función como `byte[]`.

```java
  @FunctionName("getBlobSizeHttp")
  @StorageAccount("Storage_Account_Connection_String")
  public HttpResponseMessage blobSize(
    @HttpTrigger(name = "req", 
      methods = {HttpMethod.GET}, 
      authLevel = AuthorizationLevel.ANONYMOUS) 
    HttpRequestMessage<Optional<String>> request,
    @BlobInput(
      name = "file", 
      dataType = "binary", 
      path = "samples-workitems/{Query.file}") 
    byte[] content,
    final ExecutionContext context) {
      // build HTTP response with size of requested blob
      return request.createResponseBuilder(HttpStatus.OK)
        .body("The size of \"" + request.getQueryParameters().get("file") + "\" is: " + content.length + " bytes")
        .build();
  }
```

#### <a name="queue-trigger-receive-blob-name-from-queue-message"></a>Desencadenador de cola, recepción del nombre del blob de la cola de mensajes

 El ejemplo siguiente muestra una función de Java que usa la anotación `QueueTrigger` para recibir un mensaje que contiene el nombre de un archivo en un contenedor de Blob Storage. Posteriormente, la anotación `BlobInput` lee el archivo y pasa el contenido a la función como `byte[]`.

```java
  @FunctionName("getBlobSize")
  @StorageAccount("Storage_Account_Connection_String")
  public void blobSize(
    @QueueTrigger(
      name = "filename", 
      queueName = "myqueue-items-sample") 
    String filename,
    @BlobInput(
      name = "file", 
      dataType = "binary", 
      path = "samples-workitems/{queueTrigger}") 
    byte[] content,
    final ExecutionContext context) {
      context.getLogger().info("The size of \"" + filename + "\" is: " + content.length + " bytes");
  }
```

En la [biblioteca en tiempo de ejecución de funciones de Java](/java/api/overview/azure/functions/runtime), utilice la anotación `@BlobInput` en los parámetros cuyo valor provendría de un blob.  Esta anotación se puede usar con tipos nativos de Java, POJO o valores que aceptan valores NULL mediante `Optional<T>`.

---

## <a name="attributes-and-annotations"></a>Atributos y anotaciones

# <a name="c"></a>[C#](#tab/csharp)

En las [bibliotecas de clase C#](functions-dotnet-class-library.md), use [BlobAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/dev/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs/BlobAttribute.cs).

El constructor del atributo toma la ruta de acceso al blob y un parámetro `FileAccess` que indica lectura o escritura, tal como se muestra en el ejemplo siguiente:

```csharp
[FunctionName("BlobInput")]
public static void Run(
    [QueueTrigger("myqueue-items")] string myQueueItem,
    [Blob("samples-workitems/{queueTrigger}", FileAccess.Read)] Stream myBlob,
    ILogger log)
{
    log.LogInformation($"BlobInput processed blob\n Name:{myQueueItem} \n Size: {myBlob.Length} bytes");
}

```

Puede establecer la propiedad `Connection` para especificar la cuenta de almacenamiento que se usará, tal como se muestra en el ejemplo siguiente:

```csharp
[FunctionName("BlobInput")]
public static void Run(
    [QueueTrigger("myqueue-items")] string myQueueItem,
    [Blob("samples-workitems/{queueTrigger}", FileAccess.Read, Connection = "StorageConnectionAppSetting")] Stream myBlob,
    ILogger log)
{
    log.LogInformation($"BlobInput processed blob\n Name:{myQueueItem} \n Size: {myBlob.Length} bytes");
}
```

Puede usar el atributo `StorageAccount` para especificar la cuenta de almacenamiento en el nivel de clase, método o parámetro. Para más información, consulte [Desencadenador: atributos y anotaciones](./functions-bindings-storage-blob-trigger.md#attributes-and-annotations).

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

El script de C# no admite atributos.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

JavaScript no admite atributos.

# <a name="python"></a>[Python](#tab/python)

Python no admite atributos.

# <a name="java"></a>[Java](#tab/java)

El atributo `@BlobInput` da acceso al blob que desencadenó la función. Si utiliza una matriz de bytes con el atributo, establezca `dataType` en `binary`. Vea el [ejemplo de entrada](#example) para más información.

---

## <a name="configuration"></a>Configuración

En la siguiente tabla se explican las propiedades de configuración de enlace que se definen en el archivo *function.json* y el atributo `Blob`.

|Propiedad de function.json | Propiedad de atributo |Descripción|
|---------|---------|----------------------|
|**type** | N/D | Se debe establecer en `blob`. |
|**direction** | N/D | Se debe establecer en `in`. Las excepciones se indican en la sección [uso](#usage). |
|**name** | N/D | Nombre de la variable que representa el blob en el código de la función.|
|**path** |**BlobPath** | Ruta de acceso al blob. |
|**connection** |**Connection**| El nombre de una configuración de aplicación que contiene la [cadena de conexión de almacenamiento](../storage/common/storage-configure-connection-string.md) que se usará para este enlace. Si el nombre de la configuración de aplicación comienza con "AzureWebJobs", puede especificar solo el resto del nombre aquí. Por ejemplo, si establece `connection` en "MyStorage", el entorno en tiempo de ejecución de Functions busca una configuración de aplicación denominada "AzureWebJobsMyStorage". Si deja `connection` vacía, el entorno en tiempo de ejecución de Functions usa la cadena de conexión de almacenamiento predeterminada en la configuración de aplicación que se denomina `AzureWebJobsStorage`.<br><br>La cadena de conexión debe ser para una cuenta de almacenamiento de uso general, no una [cuenta de almacenamiento solo de blob](../storage/common/storage-account-overview.md#types-of-storage-accounts).|
|N/D | **Acceder** | Indica si va a leer o escribir. |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Uso

# <a name="c"></a>[C#](#tab/csharp)

[!INCLUDE [functions-bindings-blob-storage-input-usage.md](../../includes/functions-bindings-blob-storage-input-usage.md)]

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

[!INCLUDE [functions-bindings-blob-storage-input-usage.md](../../includes/functions-bindings-blob-storage-input-usage.md)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Acceda a los datos de BLOB mediante `context.bindings.<NAME>` donde `<NAME>` coincide con el valor definido en *function.json*.

# <a name="python"></a>[Python](#tab/python)

Acceda a los datos de blob a través del parámetro con el tipo [InputStream](/python/api/azure-functions/azure.functions.inputstream?view=azure-python). Vea el [ejemplo de entrada](#example) para más información.

# <a name="java"></a>[Java](#tab/java)

El atributo `@BlobInput` da acceso al blob que desencadenó la función. Si utiliza una matriz de bytes con el atributo, establezca `dataType` en `binary`. Vea el [ejemplo de entrada](#example) para más información.

---

## <a name="next-steps"></a>Pasos siguientes

- [Ejecución de una función cuando cambian los datos de Blob Storage](./functions-bindings-storage-blob-trigger.md)
- [Escritura de datos de Blob Storage desde una función](./functions-bindings-storage-blob-output.md)
