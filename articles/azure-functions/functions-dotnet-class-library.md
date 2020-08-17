---
title: Referencia para desarrolladores de C# de Azure Functions
description: Cómo desarrollar funciones de Azure con C#.
ms.topic: conceptual
ms.date: 07/24/2020
ms.openlocfilehash: 7c00dcd3648ef175d4e834fceef3444f7943e70b
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87288202"
---
# <a name="azure-functions-c-developer-reference"></a>Referencia para desarrolladores de C# de Azure Functions

<!-- When updating this article, make corresponding changes to any duplicate content in functions-reference-csharp.md -->

Este artículo es una introducción al desarrollo de Azure Functions mediante el uso de C# en las bibliotecas de clases .NET.

Azure Functions es compatible con C# y con los lenguajes de programación de scripts de C#. Si busca orientación sobre cómo [usar C# en Azure Portal](functions-create-function-app-portal.md), vea la [referencia para desarrolladores de scripts de C#](functions-reference-csharp.md).

En este artículo se supone que ya ha leído los siguientes artículos:

* [Guía para desarrolladores de Azure Functions](functions-reference.md)
* [Herramientas de Visual Studio 2019 de Azure Functions](functions-develop-vs.md)

## <a name="supported-versions"></a>Versiones compatibles

Las versiones del entorno en tiempo de ejecución de Functions funcionan con versiones específicas de .NET. En la tabla siguiente se muestra el nivel más alto de .NET Core y .NET Framework que se puede usar con una versión específica de Functions en el proyecto. 

| Versiones del entorno en tiempo de ejecución de Functions | Versión máxima de .NET |
| ---- | ---- |
| Functions 3.x | .NET Core 3.1 |
| Functions 2.x | .NET Core 2.2 |
| Functions 1.x | .NET Framework 4.7 |

Para obtener más información, consulte [Introducción a las versiones de tiempo de ejecución de Azure Functions](functions-versions.md).

## <a name="functions-class-library-project"></a>Proyecto de biblioteca de clases de Functions

En Visual Studio, la plantilla de proyecto de **Azure Functions** crea un proyecto de biblioteca de clases de C# que contiene los archivos siguientes:

* [host.json](functions-host-json.md): almacena los valores de configuración que afectan a todas las funciones del proyecto cuando se ejecuta localmente o en Azure.
* [local.settings.json](functions-run-local.md#local-settings-file): almacena la configuración de la aplicación y las cadenas de conexión que se utilizan cuando se ejecuta localmente. Este archivo contiene información confidencial que no se publicará en la aplicación de función de Azure. En su lugar, [agregue opciones de configuración de la aplicación a la aplicación de función](functions-develop-vs.md#function-app-settings).

Al compilar el proyecto, se genera una estructura de carpetas que se parece al siguiente ejemplo del directorio de salida de la compilación:

```
<framework.version>
 | - bin
 | - MyFirstFunction
 | | - function.json
 | - MySecondFunction
 | | - function.json
 | - host.json
```

Este directorio es lo que se implementa en la aplicación de función en Azure. Las extensiones de enlace necesarias en la [versión 2.x](functions-versions.md) del sistema en tiempo de ejecución de Functions se [agregan al proyecto como paquetes de NuGet](./functions-bindings-register.md#vs).

> [!IMPORTANT]
> El proceso de compilación crea un archivo *function.json* para cada función. Este archivo *function.json* no está pensado para que se pueda modificar directamente. No se puede cambiar la configuración del enlace ni deshabilitar la función mediante la edición de este archivo. Para obtener información sobre cómo deshabilitar una función, consulte [How to disable functions (Cómo deshabilitar funciones)](disable-function.md).


## <a name="methods-recognized-as-functions"></a>Métodos reconocidos como funciones

En una biblioteca de clases, una función es un método estático con un atributo `FunctionName` y un atributo desencadenador, tal como se muestra en el ejemplo siguiente:

```csharp
public static class SimpleExample
{
    [FunctionName("QueueTrigger")]
    public static void Run(
        [QueueTrigger("myqueue-items")] string myQueueItem, 
        ILogger log)
    {
        log.LogInformation($"C# function processed: {myQueueItem}");
    }
} 
```

El atributo `FunctionName` marca el método como punto de entrada de una función. El nombre debe ser único dentro de un proyecto, debe empezar por una letra y solo puede incluir letras, números, `_` y `-`, y hasta 127 caracteres. Las plantillas de proyecto suelen crear un método denominado `Run`, pero el nombre de método puede ser cualquier nombre de método de C# válido.

El atributo desencadenador especifica el tipo de desencadenador y enlaza los datos de entrada a un parámetro del método. La función de ejemplo se desencadena mediante un mensaje de cola, y este último se pasa al método en el `myQueueItem` parámetro.

## <a name="method-signature-parameters"></a>Parámetros de la firma del método

La firma del método puede contener parámetros distintos del usado con el atributo desencadenador. Estos son algunos de los parámetros adicionales que puede incluir:

* [Enlaces de entrada y salida](functions-triggers-bindings.md) marcados como tales con atributos.  
* Un parámetro `ILogger` o `TraceWriter` ([versión 1.x](functions-versions.md#creating-1x-apps) exclusivamente) para el [registro](#logging).
* Un parámetro `CancellationToken` para el [cierre estable](#cancellation-tokens).
* Parámetros de [expresiones de enlace](./functions-bindings-expressions-patterns.md) para obtener metadatos de desencadenador.

El orden de los parámetros en la signatura de función no es importante. Por ejemplo, puede colocar los parámetros de desencadenador antes o después de otros enlaces y puede colocar el parámetro de registrador antes o después de los parámetros de desencadenador o enlace.

### <a name="output-binding-example"></a>Ejemplo de enlace de salida

En el ejemplo siguiente se modifica el ejemplo anterior mediante la adición de un enlace de la cola de salida. La función escribe el mensaje de la cola que desencadena la función en un nuevo mensaje de cola de una cola distinta.

```csharp
public static class SimpleExampleWithOutput
{
    [FunctionName("CopyQueueMessage")]
    public static void Run(
        [QueueTrigger("myqueue-items-source")] string myQueueItem, 
        [Queue("myqueue-items-destination")] out string myQueueItemCopy,
        ILogger log)
    {
        log.LogInformation($"CopyQueueMessage function processed: {myQueueItem}");
        myQueueItemCopy = myQueueItem;
    }
}
```

En los artículos de referencia de enlace ([colas de almacenamiento](functions-bindings-storage-queue.md), por ejemplo) se explica qué tipos de parámetros puede usar con los atributos de enlace de entrada o salida y desencadenador.

### <a name="binding-expressions-example"></a>Ejemplo de expresiones de enlace

El código siguiente recibe el nombre de la cola que se va a supervisar desde una configuración de aplicación y recibe la hora de creación del mensaje de cola en el parámetro `insertionTime`.

```csharp
public static class BindingExpressionsExample
{
    [FunctionName("LogQueueMessage")]
    public static void Run(
        [QueueTrigger("%queueappsetting%")] string myQueueItem,
        DateTimeOffset insertionTime,
        ILogger log)
    {
        log.LogInformation($"Message content: {myQueueItem}");
        log.LogInformation($"Created at: {insertionTime}");
    }
}
```

## <a name="autogenerated-functionjson"></a>Archivo function.json generado automáticamente

El proceso de compilación crea un archivo *function.json* en una carpeta de la función en la carpeta de compilación. Como se indicó anteriormente, este archivo no está pensado para que se pueda modificar directamente. No se puede cambiar la configuración del enlace ni deshabilitar la función mediante la edición de este archivo. 

El propósito de este archivo es proporcionar información al controlador de escala que se usará para [escalar decisiones en el plan de consumo](functions-scale.md#how-the-consumption-and-premium-plans-work). Por esta razón, el archivo solo tiene información del desencadenador, no sobre los enlaces de entrada o salida.

El archivo *function.json* generado incluye una propiedad `configurationSource` que indica el tiempo de ejecución para utilizar atributos de .NET para los enlaces, en lugar de la configuración de *function.json*. Este es un ejemplo:

```json
{
  "generatedBy": "Microsoft.NET.Sdk.Functions-1.0.0.0",
  "configurationSource": "attributes",
  "bindings": [
    {
      "type": "queueTrigger",
      "queueName": "%input-queue-name%",
      "name": "myQueueItem"
    }
  ],
  "disabled": false,
  "scriptFile": "..\\bin\\FunctionApp1.dll",
  "entryPoint": "FunctionApp1.QueueTrigger.Run"
}
```

## <a name="microsoftnetsdkfunctions"></a>Microsoft.NET.Sdk.Functions

El archivo *function.json* se genera mediante el paquete NuGet [Microsoft\.NET\.Sdk\.Functions](https://www.nuget.org/packages/Microsoft.NET.Sdk.Functions). 

Se usa el mismo paquete para las versiones 1.x y 2.x de Functions Runtime. La plataforma de destino es lo que diferencia un proyecto 1.x de un proyecto 2.x. Estas son las partes relevantes de los archivos *.csproj* que muestran distintas plataformas de destino y el mismo paquete `Sdk`:

**Functions 1.x**

```xml
<PropertyGroup>
  <TargetFramework>net461</TargetFramework>
</PropertyGroup>
<ItemGroup>
  <PackageReference Include="Microsoft.NET.Sdk.Functions" Version="1.0.8" />
</ItemGroup>
```

**Functions 2.x**

```xml
<PropertyGroup>
  <TargetFramework>netcoreapp2.1</TargetFramework>
  <AzureFunctionsVersion>v2</AzureFunctionsVersion>
</PropertyGroup>
<ItemGroup>
  <PackageReference Include="Microsoft.NET.Sdk.Functions" Version="1.0.8" />
</ItemGroup>
```

Entre las dependencias del paquete `Sdk` se encuentran los desencadenadores y los enlaces. Un proyecto 1.x hace referencia a los enlaces y desencadenadores 1.x porque tienen como destino .NET Framework, mientras que los enlaces y desencadenadores 2.x tienen como destino .NET Core.

El `Sdk` paquete también depende de [Newtonsoft.Json](https://www.nuget.org/packages/Newtonsoft.Json) e, indirectamente, de [WindowsAzure.Storage](https://www.nuget.org/packages/WindowsAzure.Storage). Estas dependencias garantizan que el proyecto utiliza las versiones de los paquetes que funcionan con la versión de Functions Runtime que el proyecto tiene como destino. Por ejemplo, `Newtonsoft.Json` tiene la versión 11 para .NET Framework 4.6.1, pero la instancia de Functions Runtime que tiene como destino .NET Framework 4.6.1 solo es compatible con `Newtonsoft.Json` 9.0.1. Por tanto, el código de función de ese proyecto también tiene que usar `Newtonsoft.Json` 9.0.1.

El código fuente de `Microsoft.NET.Sdk.Functions` está disponible en el repositorio de GitHub [azure\-functions\-vs\-build\-sdk](https://github.com/Azure/azure-functions-vs-build-sdk).

## <a name="runtime-version"></a>Versión en tiempo de ejecución

Visual Studio usa [Azure Functions Core Tools](functions-run-local.md#install-the-azure-functions-core-tools) para ejecutar proyectos de Functions. Core Tools es una interfaz de línea de comandos para Functions Runtime.

Core Tools se instala mediante npm, que no afecta a la versión de Core Tools que Visual Studio utiliza. Para la versión 1.x de Functions Runtime, Visual Studio almacena Core Tools en *%USERPROFILE%\AppData\Local\Azure.Functions.Cli* y utiliza la versión más reciente almacenada allí. Para Functions 2.x, Core Tools se incluye en la extensión **Azure Functions and Web Jobs Tools**. Para 1.x y 2.x, puede ver qué versión se usa en la salida de la consola cuando se ejecuta un proyecto de Functions:

```terminal
[3/1/2018 9:59:53 AM] Starting Host (HostId=contoso2-1518597420, Version=2.0.11353.0, ProcessId=22020, Debug=False, Attempt=0, FunctionsExtensionVersion=)
```

## <a name="readytorun"></a>ReadyToRun

La aplicación de funciones se puede compilar como [archivos binarios de ReadyToRun](/dotnet/core/whats-new/dotnet-core-3-0#readytorun-images). ReadyToRun es una forma de compilación Ahead Of Time que puede mejorar el rendimiento de inicio para ayudar a reducir el impacto del [inicio en frío](functions-scale.md#cold-start) cuando se ejecuta en un [plan de consumo](functions-scale.md#consumption-plan).

ReadyToRun está disponible en .NET 3.0 y requiere la [versión 3.0 del runtime de Azure Functions](functions-versions.md).

Para compilar un proyecto como ReadyToRun, actualice el archivo del proyecto agregando los elementos `<PublishReadyToRun>` y `<RuntimeIdentifier>`. A continuación se encuentra la configuración para publicar en una aplicación de funciones de Windows de 32 bits.

```xml
<PropertyGroup>
  <TargetFramework>netcoreapp3.1</TargetFramework>
  <AzureFunctionsVersion>v3</AzureFunctionsVersion>
  <PublishReadyToRun>true</PublishReadyToRun>
  <RuntimeIdentifier>win-x86</RuntimeIdentifier>
</PropertyGroup>
```

> [!IMPORTANT]
> Actualmente, ReadyToRun no admite la compilación cruzada. Debe compilar la aplicación en la misma plataforma que el destino de implementación. Preste también atención al "valor de bits" que está configurado en la aplicación de funciones. Por ejemplo, si la aplicación de funciones de Azure es Windows de 64 bits, debe compilarla en Windows con `win-x64` como [identificador de runtime](/dotnet/core/rid-catalog).

La aplicación también se puede compilar con ReadyToRun desde la línea de comandos. Para más información, consulte la opción `-p:PublishReadyToRun=true` en [`dotnet publish`](/dotnet/core/tools/dotnet-publish).

## <a name="supported-types-for-bindings"></a>Tipos compatibles para los enlaces

Cada enlace tiene sus propios tipos compatibles; por ejemplo, un atributo desencadenador de blobs puede aplicarse a un parámetro de cadena, un parámetro POCO, un parámetro `CloudBlockBlob` o cualquiera de los demás tipos compatibles. En el [artículo de referencia sobre los enlaces de blobs](functions-bindings-storage-blob-trigger.md#usage) se enumeran todos los tipos de parámetros compatibles. Para obtener más información, vea el artículo sobre [desencadenadores y enlaces](functions-triggers-bindings.md) y los [documentos de referencia sobre enlaces para cada tipo de enlace](functions-triggers-bindings.md#next-steps).

[!INCLUDE [HTTP client best practices](../../includes/functions-http-client-best-practices.md)]

## <a name="binding-to-method-return-value"></a>Enlace al valor devuelto del método

Puede usar un valor devuelto de método para un enlace de salida mediante la aplicación del atributo al valor devuelto de método. Para ver ejemplos, consulte [Desencadenadores y enlaces](./functions-bindings-return-value.md). 

Utilice el valor devuelto solo si una ejecución de función correcta siempre da como resultado un valor devuelto para pasar al enlace de salida. En caso contrario, use `ICollector` o `IAsyncCollector`, como se muestra en la sección siguiente.

## <a name="writing-multiple-output-values"></a>Escribir varios valores de salida

Para escribir varios valores en un enlace de salida, o si una invocación de función correcta podría no dar nada como resultado para pasar al enlace de salida, use los tipos [`ICollector`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/ICollector.cs) o [`IAsyncCollector`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/IAsyncCollector.cs). Estos tipos son colecciones de solo escritura que se escriben en el enlace de salida cuando se completa el método.

En este ejemplo se escriben varios mensajes en cola en la misma cola mediante `ICollector`:

```csharp
public static class ICollectorExample
{
    [FunctionName("CopyQueueMessageICollector")]
    public static void Run(
        [QueueTrigger("myqueue-items-source-3")] string myQueueItem,
        [Queue("myqueue-items-destination")] ICollector<string> myDestinationQueue,
        ILogger log)
    {
        log.LogInformation($"C# function processed: {myQueueItem}");
        myDestinationQueue.Add($"Copy 1: {myQueueItem}");
        myDestinationQueue.Add($"Copy 2: {myQueueItem}");
    }
}
```

## <a name="logging"></a>Registro

Para grabar la salida en los registros de streaming de C#, incluya un argumento de tipo [ILogger](/dotnet/api/microsoft.extensions.logging.ilogger). Le recomendamos que lo llame `log`, como en el ejemplo siguiente:  

```csharp
public static class SimpleExample
{
    [FunctionName("QueueTrigger")]
    public static void Run(
        [QueueTrigger("myqueue-items")] string myQueueItem, 
        ILogger log)
    {
        log.LogInformation($"C# function processed: {myQueueItem}");
    }
} 
```

Evite el uso de `Console.Write` en Azure Functions. Para más información, consulte [Escribir registros en funciones de C#](functions-monitoring.md#write-logs-in-c-functions) en el artículo **Supervisión de Azure Functions**.

## <a name="async"></a>Async

Para convertir una función en [asincrónica](/dotnet/csharp/programming-guide/concepts/async/), use la palabra clave `async` y devuelva un objeto `Task`.

```csharp
public static class AsyncExample
{
    [FunctionName("BlobCopy")]
    public static async Task RunAsync(
        [BlobTrigger("sample-images/{blobName}")] Stream blobInput,
        [Blob("sample-images-copies/{blobName}", FileAccess.Write)] Stream blobOutput,
        CancellationToken token,
        ILogger log)
    {
        log.LogInformation($"BlobCopy function processed.");
        await blobInput.CopyToAsync(blobOutput, 4096, token);
    }
}
```

No puede usar parámetros `out` en funciones asincrónicas. Para los enlaces de salida, utilice el [valor devuelto de función](#binding-to-method-return-value) o un [objeto recopilador](#writing-multiple-output-values) en su lugar.

## <a name="cancellation-tokens"></a>Tokens de cancelación

Una función puede aceptar un parámetro [CancellationToken](/dotnet/api/system.threading.cancellationtoken) que permite que el sistema operativo notifique al código cuando la función esté a punto de finalizar. Puede utilizar esta notificación para asegurarse de que la función no se termina inesperadamente en una forma que deje los datos en un estado incoherente.

En el ejemplo siguiente se muestra cómo comprobar la finalización inminente de la función.

```csharp
public static class CancellationTokenExample
{
    public static void Run(
        [QueueTrigger("inputqueue")] string inputText,
        TextWriter logger,
        CancellationToken token)
    {
        for (int i = 0; i < 100; i++)
        {
            if (token.IsCancellationRequested)
            {
                logger.WriteLine("Function was cancelled at iteration {0}", i);
                break;
            }
            Thread.Sleep(5000);
            logger.WriteLine("Normal processing for queue message={0}", inputText);
        }
    }
}
```

## <a name="environment-variables"></a>Variables de entorno

Para obtener una variable de entorno o un valor de configuración de aplicación, use `System.Environment.GetEnvironmentVariable`, como se muestra en el ejemplo de código siguiente:

```csharp
public static class EnvironmentVariablesExample
{
    [FunctionName("GetEnvironmentVariables")]
    public static void Run([TimerTrigger("0 */5 * * * *")]TimerInfo myTimer, ILogger log)
    {
        log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
        log.LogInformation(GetEnvironmentVariable("AzureWebJobsStorage"));
        log.LogInformation(GetEnvironmentVariable("WEBSITE_SITE_NAME"));
    }

    public static string GetEnvironmentVariable(string name)
    {
        return name + ": " +
            System.Environment.GetEnvironmentVariable(name, EnvironmentVariableTarget.Process);
    }
}
```

La configuración de la aplicación se puede leer de las variables de entorno tanto cuando se desarrolla de manera local como cuando se ejecuta en Azure. Cuando se desarrolla de manera local, la configuración de la aplicación procede de la colección `Values` en el archivo *local.settings.json*. En ambos entornos, tanto local como Azure, `GetEnvironmentVariable("<app setting name>")` recupera el valor de la configuración de aplicación con nombre. Por ejemplo, cuando se ejecuta de manera local, se devolvería "My Site Name" si el archivo *local.settings.json* contiene `{ "Values": { "WEBSITE_SITE_NAME": "My Site Name" } }`.

La propiedad [System.Configuration.ConfigurationManager.AppSettings](/dotnet/api/system.configuration.configurationmanager.appsettings) es una API alternativa para obtener los valores de configuración de la aplicación, pero se recomienda que use `GetEnvironmentVariable` como se muestra aquí.

## <a name="binding-at-runtime"></a>Enlace en tiempo de ejecución

En C# y otros lenguajes .NET, puede usar un patrón de enlace [imperativo](https://en.wikipedia.org/wiki/Imperative_programming), en contraposición a los enlaces [*declarativos*](https://en.wikipedia.org/wiki/Declarative_programming) de los atributos. Los enlaces imperativos resultan útiles cuando los parámetros de enlace tienen que calcularse en tiempo de ejecución, en lugar de en el tiempo de diseño. Con este patrón, se pueden establecer enlaces compatibles de entrada y salida sobre la marcha en el código de función.

Defina un enlace imperativo como se indica a continuación:

- **No** incluya un atributo en la signatura de función para los enlaces imperativos deseados.
- Pase un parámetro de entrada [`Binder binder`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Host/Bindings/Runtime/Binder.cs) o [`IBinder binder`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/IBinder.cs).
- Utilice el siguiente patrón de C# para realizar el enlace de datos.

  ```cs
  using (var output = await binder.BindAsync<T>(new BindingTypeAttribute(...)))
  {
      ...
  }
  ```

  `BindingTypeAttribute` es el atributo de .NET que define el enlace y `T` es un tipo de entrada o de salida compatible con ese tipo de enlace. `T` no puede ser un tipo de parámetro `out` (como `out JObject`). Por ejemplo, el enlace de salida de la tabla de Mobile Apps admite [seis tipos de salida](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.MobileApps/MobileTableAttribute.cs#L17-L22), pero solo se puede utilizar [ICollector\<T>](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/ICollector.cs) o [IAsyncCollector\<T>](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/IAsyncCollector.cs) con el enlace imperativo.

### <a name="single-attribute-example"></a>Ejemplo de un único atributo

El ejemplo de código siguiente crea un [enlace de salida al blob de almacenamiento](functions-bindings-storage-blob-output.md) con la ruta de acceso al blob definida en tiempo de ejecución y, a continuación, escribe una cadena en el blob.

```cs
public static class IBinderExample
{
    [FunctionName("CreateBlobUsingBinder")]
    public static void Run(
        [QueueTrigger("myqueue-items-source-4")] string myQueueItem,
        IBinder binder,
        ILogger log)
    {
        log.LogInformation($"CreateBlobUsingBinder function processed: {myQueueItem}");
        using (var writer = binder.Bind<TextWriter>(new BlobAttribute(
                    $"samples-output/{myQueueItem}", FileAccess.Write)))
        {
            writer.Write("Hello World!");
        };
    }
}
```

[BlobAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs/BlobAttribute.cs) define el enlace de entrada o salida del [blob de almacenamiento](functions-bindings-storage-blob.md), y [TextWriter](/dotnet/api/system.io.textwriter) es un tipo de enlace de salida admitido.

### <a name="multiple-attribute-example"></a>Ejemplo de varios atributos

En el ejemplo anterior se obtiene el valor de la aplicación para la cadena de conexión en la cuenta de almacenamiento principal de la aplicación de función (que es `AzureWebJobsStorage`). Se puede especificar una configuración personalizada de la aplicación para utilizarla para la cuenta de almacenamiento agregando el atributo [StorageAccountAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs) y pasando la matriz de atributos a `BindAsync<T>()`. Use un parámetro `Binder`, no `IBinder`.  Por ejemplo:

```cs
public static class IBinderExampleMultipleAttributes
{
    [FunctionName("CreateBlobInDifferentStorageAccount")]
    public async static Task RunAsync(
            [QueueTrigger("myqueue-items-source-binder2")] string myQueueItem,
            Binder binder,
            ILogger log)
    {
        log.LogInformation($"CreateBlobInDifferentStorageAccount function processed: {myQueueItem}");
        var attributes = new Attribute[]
        {
        new BlobAttribute($"samples-output/{myQueueItem}", FileAccess.Write),
        new StorageAccountAttribute("MyStorageAccount")
        };
        using (var writer = await binder.BindAsync<TextWriter>(attributes))
        {
            await writer.WriteAsync("Hello World!!");
        }
    }
}
```

## <a name="triggers-and-bindings"></a>Desencadenadores y enlaces 

[!INCLUDE [Supported triggers and bindings](../../includes/functions-bindings.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre los desencadenadores y los enlaces](functions-triggers-bindings.md)

> [!div class="nextstepaction"]
> [Más información sobre procedimientos recomendados para Azure Functions](functions-best-practices.md)
