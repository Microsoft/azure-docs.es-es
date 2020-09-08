---
title: Referencia para desarrolladores de scripts de C# de Azure Functions
description: Obtenga información sobre cómo desarrollar Azure Functions mediante scripts de C#.
author: craigshoemaker
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 12/12/2017
ms.author: cshoe
ms.openlocfilehash: 4d5388f850f47323f6ad79f9f91e617e506546bf
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88205448"
---
# <a name="azure-functions-c-script-csx-developer-reference"></a>Referencia para desarrolladores de scripts de C# de Azure Functions (.csx)

<!-- When updating this article, make corresponding changes to any duplicate content in functions-dotnet-class-library.md -->

Este artículo es una introducción al desarrollo de Azure Functions mediante el uso de scripts de C# ( *.csx*).

Azure Functions es compatible con C# y con los lenguajes de programación de scripts de C#. Si busca orientación sobre cómo [usar C# en un proyecto de biblioteca de clases de Visual Studio](functions-develop-vs.md), consulte la [referencia para desarrolladores de C#](functions-dotnet-class-library.md).

En este artículo se supone que ya ha leído la [guía para desarrolladores de Azure Functions](functions-reference.md).

## <a name="how-csx-works"></a>Funcionamiento de .csx

La experiencia de scripts de C# para Azure Functions se basa en el [SDK de Azure WebJobs](https://github.com/Azure/azure-webjobs-sdk/wiki/Introduction). Los datos fluyen en la función de C# a través de los argumentos de método. Los nombres de los argumentos se especifican en un archivo `function.json`, y hay nombres predefinidos para acceder a cosas como el registrador de funciones y los tokens de cancelación.

El formato *.csx* permite escribir menos "texto reutilizable" y centrarse en escribir solo una función de C#. En lugar de encapsular todo en un espacio de nombres y una clase, defina solamente un método `Run`. Incluya las referencias a ensamblados y los espacios de nombres al principio del archivo como de costumbre.

Los archivos *.csx* de una aplicación de función se compilan cuando se inicializa una instancia. Este paso de compilación puede conllevar, por ejemplo, que un arranque en frío pueda tardar más para las funciones de script de C# en comparación con las bibliotecas de clases de C#. En este paso de compilación también se plantea la pregunta de por qué las funciones de script de C# se pueden editar en Azure Portal y las bibliotecas de clases de C# no.

## <a name="folder-structure"></a>Estructura de carpetas

La estructura de carpetas para un proyecto de script de C# tiene el siguiente aspecto:

```
FunctionsProject
 | - MyFirstFunction
 | | - run.csx
 | | - function.json
 | | - function.proj
 | - MySecondFunction
 | | - run.csx
 | | - function.json
 | | - function.proj
 | - host.json
 | - extensions.csproj
 | - bin
```

Hay un archivo [host.json](functions-host-json.md) compartido que se puede usar para configurar la aplicación de función. Cada función tiene su propio archivo de código (.csx) y archivo de configuración de enlace (function.json).

Las extensiones de enlace necesarias en la [versión 2.x y posteriores](functions-versions.md) del entorno en tiempo de ejecución de Functions se definen en el archivo `extensions.csproj`, con los archivos de biblioteca reales de la carpeta `bin`. Al desarrollar de forma local, debe [registrar las extensiones de enlace](./functions-bindings-register.md#extension-bundles). Al desarrollar funciones en Azure Portal, este registro se realiza automáticamente.

## <a name="binding-to-arguments"></a>Enlace a argumentos

Los datos de entrada o salida está enlazados a un parámetro de función de script de C# mediante la propiedad `name` en el archivo de configuración *function.json*. En el ejemplo siguiente se muestra un archivo *function.json* y un archivo *run.csx* para una función queue-triggered. El parámetro que recibe los datos del mensaje de cola se llama `myQueueItem` porque ese es el valor de la propiedad `name`.

```json
{
    "disabled": false,
    "bindings": [
        {
            "type": "queueTrigger",
            "direction": "in",
            "name": "myQueueItem",
            "queueName": "myqueue-items",
            "connection":"MyStorageConnectionAppSetting"
        }
    ]
}
```

```csharp
#r "Microsoft.WindowsAzure.Storage"

using Microsoft.Extensions.Logging;
using Microsoft.WindowsAzure.Storage.Queue;
using System;

public static void Run(CloudQueueMessage myQueueItem, ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed: {myQueueItem.AsString}");
}
```

La instrucción `#r` se explica [más adelante en este artículo](#referencing-external-assemblies).

## <a name="supported-types-for-bindings"></a>Tipos compatibles para los enlaces

Cada enlace tiene sus propios tipos compatibles; por ejemplo, se puede utilizar un desencadenador de blobs con un parámetro de cadena, un parámetro POCO, un parámetro `CloudBlockBlob` o cualquiera de los demás tipos compatibles. En el [artículo de referencia sobre los enlaces de blobs](functions-bindings-storage-blob-trigger.md#usage) se enumeran todos los tipos de parámetros compatibles para los desencadenadores de blobs. Para obtener más información, vea el artículo sobre [desencadenadores y enlaces](functions-triggers-bindings.md) y los [documentos de referencia sobre enlaces para cada tipo de enlace](functions-triggers-bindings.md#next-steps).

[!INCLUDE [HTTP client best practices](../../includes/functions-http-client-best-practices.md)]

## <a name="referencing-custom-classes"></a>Referencia a clases personalizadas

Si tiene que utilizar una clase objeto CRL estándar (POCO) personalizada, puede incluir la definición de clase en el mismo archivo o colocarla en un archivo independiente.

En el ejemplo siguiente se muestra un ejemplo de *run.csx* que incluye una definición de clase POCO.

```csharp
public static void Run(string myBlob, out MyClass myQueueItem)
{
    log.Verbose($"C# Blob trigger function processed: {myBlob}");
    myQueueItem = new MyClass() { Id = "myid" };
}

public class MyClass
{
    public string Id { get; set; }
}
```

Una clase POCO debe tener un captador y un establecedor definidos para cada propiedad.

## <a name="reusing-csx-code"></a>Reutilización del código .csx

Puede usar las clases y los métodos definidos en otros archivos *.csx* con el archivo *run.csx*. Para ello, utilice directivas `#load` en el archivo *run.csx*. En el siguiente ejemplo, una rutina de registro denominada `MyLogger` se comparte en *myLogger.csx* y se carga en *run.csx* mediante la directiva `#load`:

Archivo *run.csx*de ejemplo:

```csharp
#load "mylogger.csx"

using Microsoft.Extensions.Logging;

public static void Run(TimerInfo myTimer, ILogger log)
{
    log.LogInformation($"Log by run.csx: {DateTime.Now}");
    MyLogger(log, $"Log by MyLogger: {DateTime.Now}");
}
```

Archivo *mylogger.csx*de ejemplo:

```csharp
public static void MyLogger(ILogger log, string logtext)
{
    log.LogInformation(logtext);
}
```

El uso de un archivo *.csx* compartido es un patrón común para asignar rigurosamente los datos transferidos entre funciones mediante un objeto POCO. En el siguiente ejemplo simplificado, un desencadenador HTTP y un desencadenador de cola comparten un objeto POCO denominado `Order` para tipar fuertemente los datos del pedido:

Por ejemplo, *run.csx* para el desencadenador HTTP:

```cs
#load "..\shared\order.csx"

using System.Net;
using Microsoft.Extensions.Logging;

public static async Task<HttpResponseMessage> Run(Order req, IAsyncCollector<Order> outputQueueItem, ILogger log)
{
    log.LogInformation("C# HTTP trigger function received an order.");
    log.LogInformation(req.ToString());
    log.LogInformation("Submitting to processing queue.");

    if (req.orderId == null)
    {
        return new HttpResponseMessage(HttpStatusCode.BadRequest);
    }
    else
    {
        await outputQueueItem.AddAsync(req);
        return new HttpResponseMessage(HttpStatusCode.OK);
    }
}
```

Por ejemplo, *run.csx* para el desencadenador de cola:

```cs
#load "..\shared\order.csx"

using System;
using Microsoft.Extensions.Logging;

public static void Run(Order myQueueItem, out Order outputQueueItem, ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed order...");
    log.LogInformation(myQueueItem.ToString());

    outputQueueItem = myQueueItem;
}
```

Por ejemplo, *order.csx*:

```cs
public class Order
{
    public string orderId {get; set; }
    public string custName {get; set;}
    public string custAddress {get; set;}
    public string custEmail {get; set;}
    public string cartId {get; set; }

    public override String ToString()
    {
        return "\n{\n\torderId : " + orderId +
                  "\n\tcustName : " + custName +
                  "\n\tcustAddress : " + custAddress +
                  "\n\tcustEmail : " + custEmail +
                  "\n\tcartId : " + cartId + "\n}";
    }
}
```

Puede usar una ruta de acceso relativa con la directiva `#load` :

* `#load "mylogger.csx"` carga un archivo que se encuentra en la carpeta de la función.
* `#load "loadedfiles\mylogger.csx"` carga un archivo ubicado en una carpeta dentro de la carpeta de la función.
* `#load "..\shared\mylogger.csx"` carga un archivo ubicado en una carpeta del mismo nivel que la carpeta de la función, es decir, directamente en *wwwroot*.

La directiva `#load` solo funciona con archivos *.csx*, no con archivos *.cs*.

## <a name="binding-to-method-return-value"></a>Enlace al valor devuelto del método

Puede usar el valor devuelto de un método para un enlace de salida, mediante el nombre `$return` en *function.json*. Para ver ejemplos, consulte [Desencadenadores y enlaces](./functions-bindings-return-value.md).

Utilice el valor devuelto solo si una ejecución de función correcta siempre da como resultado un valor devuelto para pasar al enlace de salida. En caso contrario, use `ICollector` o `IAsyncCollector`, como se muestra en la sección siguiente.

## <a name="writing-multiple-output-values"></a>Escribir varios valores de salida

Para escribir varios valores en un enlace de salida, o si una invocación de función correcta podría no dar nada como resultado para pasar al enlace de salida, use los tipos [`ICollector`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/ICollector.cs) o [`IAsyncCollector`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/IAsyncCollector.cs). Estos tipos son colecciones de solo escritura que se escriben en el enlace de salida cuando se completa el método.

En este ejemplo se escriben varios mensajes en cola en la misma cola mediante `ICollector`:

```csharp
public static void Run(ICollector<string> myQueue, ILogger log)
{
    myQueue.Add("Hello");
    myQueue.Add("World!");
}
```

## <a name="logging"></a>Registro

Para grabar la salida en los registros de streaming de C#, incluya un argumento de tipo [ILogger](/dotnet/api/microsoft.extensions.logging.ilogger). Es recomendable que lo denomine `log`. Evite el uso de `Console.Write` en Azure Functions.

```csharp
public static void Run(string myBlob, ILogger log)
{
    log.LogInformation($"C# Blob trigger function processed: {myBlob}");
}
```

> [!NOTE]
> Para obtener información sobre un marco de trabajo de registro más reciente que puede usar en lugar de `TraceWriter`, consulte [Escribir registros en funciones de C#](functions-monitoring.md#write-logs-in-c-functions) en el artículo **Supervisión de Azure Functions**.

## <a name="async"></a>Async

Para convertir una función en [asincrónica](/dotnet/csharp/programming-guide/concepts/async/), use la palabra clave `async` y devuelva un objeto `Task`.

```csharp
public async static Task ProcessQueueMessageAsync(
        string blobName,
        Stream blobInput,
        Stream blobOutput)
{
    await blobInput.CopyToAsync(blobOutput, 4096);
}
```

No puede usar parámetros `out` en funciones asincrónicas. Para los enlaces de salida, utilice el [valor devuelto de función](#binding-to-method-return-value) o un [objeto recopilador](#writing-multiple-output-values) en su lugar.

## <a name="cancellation-tokens"></a>Tokens de cancelación

Una función puede aceptar un parámetro [CancellationToken](/dotnet/api/system.threading.cancellationtoken) que permite que el sistema operativo notifique al código cuando la función esté a punto de finalizar. Puede utilizar esta notificación para asegurarse de que la función no se termina inesperadamente en una forma que deje los datos en un estado incoherente.

En el ejemplo siguiente se muestra cómo comprobar la finalización inminente de la función.

```csharp
using System;
using System.IO;
using System.Threading;

public static void Run(
    string inputText,
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
```

## <a name="importing-namespaces"></a>Importación de espacios de nombres

Si necesita importar espacios de nombres, puede hacerlo de la manera habitual, con la cláusula `using` .

```csharp
using System.Net;
using System.Threading.Tasks;
using Microsoft.Extensions.Logging;

public static Task<HttpResponseMessage> Run(HttpRequestMessage req, ILogger log)
```

Los siguientes espacios de nombres se importan automáticamente y, por tanto, son opcionales:

* `System`
* `System.Collections.Generic`
* `System.IO`
* `System.Linq`
* `System.Net.Http`
* `System.Threading.Tasks`
* `Microsoft.Azure.WebJobs`
* `Microsoft.Azure.WebJobs.Host`

## <a name="referencing-external-assemblies"></a>Referencia a ensamblados externos

Para los ensamblados de marco, agregue referencias mediante la directiva `#r "AssemblyName"` .

```csharp
#r "System.Web.Http"

using System.Net;
using System.Net.Http;
using System.Threading.Tasks;
using Microsoft.Extensions.Logging;

public static Task<HttpResponseMessage> Run(HttpRequestMessage req, ILogger log)
```

El entorno de hospedaje de Azure Functions agrega automáticamente los siguientes ensamblados:

* `mscorlib`
* `System`
* `System.Core`
* `System.Xml`
* `System.Net.Http`
* `Microsoft.Azure.WebJobs`
* `Microsoft.Azure.WebJobs.Host`
* `Microsoft.Azure.WebJobs.Extensions`
* `System.Web.Http`
* `System.Net.Http.Formatting`

Se puede hacer referencia a los ensamblados siguientes con nombre simple (por ejemplo, `#r "AssemblyName"`):

* `Newtonsoft.Json`
* `Microsoft.WindowsAzure.Storage`
* `Microsoft.ServiceBus`
* `Microsoft.AspNet.WebHooks.Receivers`
* `Microsoft.AspNet.WebHooks.Common`
* `Microsoft.Azure.NotificationHubs`

## <a name="referencing-custom-assemblies"></a>Hacer referencia a ensamblados personalizados

Para hacer referencia a un ensamblado personalizado, puede usar un ensamblado *compartido* o un ensamblado *privado*:

* Los ensamblados compartidos se comparten en todas las funciones dentro de una aplicación de función. Para hacer referencia a un ensamblado personalizado, cargue el ensamblado en una carpeta denominada `bin` en su [carpeta raíz de aplicaciones de función](functions-reference.md#folder-structure) (wwwroot).

* Los ensamblados privados forman parte del contexto de una función determinada y admiten la instalación de prueba de versiones diferentes. Los ensamblados privados se deben cargar en una carpeta `bin` en el directorio de la función. Haga referencia a ellos con el nombre de archivo, como `#r "MyAssembly.dll"`.

Para más información sobre cómo cargar archivos en su carpeta de función, consulte la sección sobre [administración de paquetes](#using-nuget-packages).

### <a name="watched-directories"></a>Directorios inspeccionados

El directorio que contiene el archivo de script de función se inspecciona automáticamente para buscar cambios en los ensamblados. Para inspeccionar los cambios de los ensamblado en otros directorios, agréguelos a la lista `watchDirectories` en [host.json](functions-host-json.md).

## <a name="using-nuget-packages"></a>Uso de paquetes NuGet
Para usar paquetes NuGet en una función de C# de la versión 2.x y posteriores, cargue un archivo *function.proj* en la carpeta de la función del sistema de archivos de la aplicación de funciones. Este es un ejemplo del archivo *function.proj* en el que se agrega una referencia a la versión *1.1.0* de *Microsoft.ProjectOxford.Face*:

```xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <TargetFramework>netstandard2.0</TargetFramework>
    </PropertyGroup>

    <ItemGroup>
        <PackageReference Include="Microsoft.ProjectOxford.Face" Version="1.1.0" />
    </ItemGroup>
</Project>
```

Para usar una fuente NuGet personalizada, especifique la fuente en un archivo *Nuget.Config* en la raíz de la aplicación la función. Para más información, vea [Configuring NuGet behavior](/nuget/consume-packages/configuring-nuget-behavior) (Configuración del comportamiento de NuGet).

> [!NOTE]
> En las funciones de C# de la versión 1.x, se hace referencia a los paquetes NuGet con un archivo *project.json* en lugar de un archivo *function.proj*.

Para las funciones de la versión 1.x, use un archivo *project.json* en su lugar. Aquí tiene un ejemplo de un archivo *project.json*:

```json
{
  "frameworks": {
    "net46":{
      "dependencies": {
        "Microsoft.ProjectOxford.Face": "1.1.0"
      }
    }
   }
}
```

### <a name="using-a-functionproj-file"></a>Uso de un archivo function.proj

1. Abra la función en Azure Portal. La pestaña de registros muestra el resultado de la instalación del paquete.
2. Para cargar un archivo *function.proj*, use uno de los métodos descritos en [Actualización de los archivos de aplicación de funciones](functions-reference.md#fileupdate) en el tema de referencia para desarrolladores de Azure Functions.
3. Una vez que haya cargado el archivo *function.proj*, verá un resultado similar al del ejemplo siguiente en el registro de streaming de la función:

```
2018-12-14T22:00:48.658 [Information] Restoring packages.
2018-12-14T22:00:48.681 [Information] Starting packages restore
2018-12-14T22:00:57.064 [Information] Restoring packages for D:\local\Temp\9e814101-fe35-42aa-ada5-f8435253eb83\function.proj...
2016-04-04T19:02:50.511 Restoring packages for D:\home\site\wwwroot\HttpTriggerCSharp1\function.proj...
2018-12-14T22:01:00.844 [Information] Installing Newtonsoft.Json 10.0.2.
2018-12-14T22:01:01.041 [Information] Installing Microsoft.ProjectOxford.Common.DotNetStandard 1.0.0.
2018-12-14T22:01:01.140 [Information] Installing Microsoft.ProjectOxford.Face.DotNetStandard 1.0.0.
2018-12-14T22:01:09.799 [Information] Restore completed in 5.79 sec for D:\local\Temp\9e814101-fe35-42aa-ada5-f8435253eb83\function.proj.
2018-12-14T22:01:10.905 [Information] Packages restored.
```

## <a name="environment-variables"></a>Variables de entorno

Para obtener una variable de entorno o un valor de configuración de aplicación, use `System.Environment.GetEnvironmentVariable`, como se muestra en el ejemplo de código siguiente:

```csharp
public static void Run(TimerInfo myTimer, ILogger log)
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
```

<a name="imperative-bindings"></a>

## <a name="binding-at-runtime"></a>Enlace en tiempo de ejecución

En C# y otros lenguajes .NET, puede usar un patrón de enlace [imperativo](https://en.wikipedia.org/wiki/Imperative_programming), en contraposición a los enlaces [*declarativos*](https://en.wikipedia.org/wiki/Declarative_programming) de *function.json*. Los enlaces imperativos resultan útiles cuando los parámetros de enlace tienen que calcularse en tiempo de ejecución, en lugar de en el tiempo de diseño. Con este patrón, se pueden establecer enlaces compatibles de entrada y salida sobre la marcha en el código de función.

Defina un enlace imperativo como se indica a continuación:

- **No** incluya una entrada en *function.json* para los enlaces imperativos deseados.
- Pase un parámetro de entrada [`Binder binder`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Host/Bindings/Runtime/Binder.cs) o [`IBinder binder`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/IBinder.cs).
- Utilice el siguiente patrón de C# para realizar el enlace de datos.

```cs
using (var output = await binder.BindAsync<T>(new BindingTypeAttribute(...)))
{
    ...
}
```

`BindingTypeAttribute` es el atributo de .NET que define el enlace y `T` es un tipo de entrada o de salida compatible con ese tipo de enlace. `T` no puede ser un tipo de parámetro `out` (como `out JObject`). Por ejemplo, el enlace de salida de la tabla de Mobile Apps admite [seis tipos de salida](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.MobileApps/MobileTableAttribute.cs#L17-L22), pero solo se puede usar [ICollector\<T>](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/ICollector.cs) o [`IAsyncCollector<T>`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/IAsyncCollector.cs) para `T`.

### <a name="single-attribute-example"></a>Ejemplo de un único atributo

El ejemplo de código siguiente crea un [enlace de salida al blob de almacenamiento](functions-bindings-storage-blob-output.md) con la ruta de acceso al blob definida en tiempo de ejecución y, a continuación, escribe una cadena en el blob.

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host.Bindings.Runtime;

public static async Task Run(string input, Binder binder)
{
    using (var writer = await binder.BindAsync<TextWriter>(new BlobAttribute("samples-output/path")))
    {
        writer.Write("Hello World!!");
    }
}
```

[BlobAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs/BlobAttribute.cs) define el enlace de entrada o salida del [blob de almacenamiento](functions-bindings-storage-blob.md), y [TextWriter](/dotnet/api/system.io.textwriter) es un tipo de enlace de salida admitido.

### <a name="multiple-attribute-example"></a>Ejemplo de varios atributos

En el ejemplo anterior se obtiene el valor de la aplicación para la cadena de conexión en la cuenta de almacenamiento principal de la aplicación de función (que es `AzureWebJobsStorage`). Se puede especificar una configuración personalizada de la aplicación para utilizarla para la cuenta de almacenamiento agregando el atributo [StorageAccountAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs) y pasando la matriz de atributos a `BindAsync<T>()`. Use un parámetro `Binder`, no `IBinder`.  Por ejemplo:

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host.Bindings.Runtime;

public static async Task Run(string input, Binder binder)
{
    var attributes = new Attribute[]
    {
        new BlobAttribute("samples-output/path"),
        new StorageAccountAttribute("MyStorageAccount")
    };

    using (var writer = await binder.BindAsync<TextWriter>(attributes))
    {
        writer.Write("Hello World!");
    }
}
```

En la tabla siguiente, aparecen los atributos de .NET para cada tipo de enlace, así como los paquetes en los que se definen.

> [!div class="mx-codeBreakAll"]
> | Enlace | Atributo | Agregar referencia |
> |------|------|------|
> | Cosmos DB | [`Microsoft.Azure.WebJobs.DocumentDBAttribute`](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.CosmosDB/CosmosDBAttribute.cs) | `#r "Microsoft.Azure.WebJobs.Extensions.CosmosDB"` |
> | Event Hubs | [`Microsoft.Azure.WebJobs.ServiceBus.EventHubAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/v2.x/src/Microsoft.Azure.WebJobs.ServiceBus/EventHubs/EventHubAttribute.cs), [`Microsoft.Azure.WebJobs.ServiceBusAccountAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/b798412ad74ba97cf2d85487ae8479f277bdd85c/test/Microsoft.Azure.WebJobs.ServiceBus.UnitTests/ServiceBusAccountTests.cs) | `#r "Microsoft.Azure.Jobs.ServiceBus"` |
> | Mobile Apps | [`Microsoft.Azure.WebJobs.MobileTableAttribute`](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.MobileApps/MobileTableAttribute.cs) | `#r "Microsoft.Azure.WebJobs.Extensions.MobileApps"` |
> | Notification Hubs | [`Microsoft.Azure.WebJobs.NotificationHubAttribute`](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/src/WebJobs.Extensions.NotificationHubs/NotificationHubAttribute.cs) | `#r "Microsoft.Azure.WebJobs.Extensions.NotificationHubs"` |
> | Azure Service Bus | [`Microsoft.Azure.WebJobs.ServiceBusAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/b798412ad74ba97cf2d85487ae8479f277bdd85c/test/Microsoft.Azure.WebJobs.ServiceBus.UnitTests/ServiceBusAttributeTests.cs), [`Microsoft.Azure.WebJobs.ServiceBusAccountAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/b798412ad74ba97cf2d85487ae8479f277bdd85c/test/Microsoft.Azure.WebJobs.ServiceBus.UnitTests/ServiceBusAccountTests.cs) | `#r "Microsoft.Azure.WebJobs.ServiceBus"` |
> | Cola de almacenamiento | [`Microsoft.Azure.WebJobs.QueueAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/), [`Microsoft.Azure.WebJobs.StorageAccountAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs) | |
> | Blob de almacenamiento | [`Microsoft.Azure.WebJobs.BlobAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs/BlobAttribute.cs), [`Microsoft.Azure.WebJobs.StorageAccountAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs) | |
> | Tabla de almacenamiento | [`Microsoft.Azure.WebJobs.TableAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs), [`Microsoft.Azure.WebJobs.StorageAccountAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs) | |
> | Twilio | [`Microsoft.Azure.WebJobs.TwilioSmsAttribute`](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.Twilio/TwilioSMSAttribute.cs) | `#r "Microsoft.Azure.WebJobs.Extensions.Twilio"` |

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre los desencadenadores y los enlaces](functions-triggers-bindings.md)

> [!div class="nextstepaction"]
> [Más información sobre procedimientos recomendados para Azure Functions](functions-best-practices.md)
