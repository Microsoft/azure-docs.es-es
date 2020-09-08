---
title: Enlaces de Mobile Apps para Azure Functions
description: Descubra cómo utilizar los enlaces de Mobile Apps en Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.custom: devx-track-csharp
ms.date: 11/21/2017
ms.author: cshoe
ms.openlocfilehash: 5ea58cc3d9f3615a74249b36f3f9ffb79caddda1
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88212238"
---
# <a name="mobile-apps-bindings-for-azure-functions"></a>Enlaces de Mobile Apps para Azure Functions 

> [!NOTE]
> Los enlaces de Azure Mobile Apps solo están disponibles para Azure Functions 1.x. No se admiten en Azure Functions 2.x y versiones posteriores.

En este artículo se explica cómo trabajar con enlaces de [Azure Mobile Apps](/previous-versions/azure/app-service-mobile/app-service-mobile-value-prop) en Azure Functions. Azure Functions admite enlaces de entrada y salida para Mobile Apps.

Los enlaces de Mobile Apps permiten leer y actualizar las tablas de datos en las aplicaciones móviles.

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

## <a name="packages---functions-1x"></a>Paquetes: Functions 1.x

Los enlaces de Mobile Apps se proporcionan en el paquete NuGet [Microsoft.Azure.WebJobs.Extensions.MobileApps](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.MobileApps), versión 1.x. El código fuente del paquete está en el repositorio [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/src/WebJobs.Extensions.MobileApps/) de GitHub.

[!INCLUDE [functions-package](../../includes/functions-package.md)]

## <a name="input"></a>Entrada

El enlace de entrada de Mobile Apps carga un registro desde un punto de conexión de tabla móvil y lo pasa a la función. En funciones de C# y F#, los cambios realizados en el registro se enviarán automáticamente a la tabla una vez que la función se complete correctamente.

## <a name="input---example"></a>Ejemplo de entrada

Vea el ejemplo específico del lenguaje:

* [Script de C# (.csx)](#input---c-script-example)
* JavaScript

### <a name="input---c-script-example"></a>Entrada: ejemplo de script de C#

En el ejemplo siguiente se muestra un enlace de entrada de Mobile Apps en un archivo de *function.json* y una [función de script de C#](functions-reference-csharp.md) que usa el enlace. Un mensaje de cola que tiene un identificador de registro desencadena la función. La función lee el registro especificado y modifica su propiedad `Text`.

Estos son los datos de enlace del archivo *function.json*:

```json
{
"bindings": [
    {
        "name": "myQueueItem",
        "queueName": "myqueue-items",
        "connection": "",
        "type": "queueTrigger",
        "direction": "in"
    },
    {
        "name": "record",
        "type": "mobileTable",
        "tableName": "MyTable",
        "id": "{queueTrigger}",
        "connection": "My_MobileApp_Url",
        "apiKey": "My_MobileApp_Key",
        "direction": "in"
    }
]
}
```
En la sección de [configuración](#input---configuration) se explican estas propiedades.

Este es el código de script de C#:

```cs
#r "Newtonsoft.Json"    
using Newtonsoft.Json.Linq;

public static void Run(string myQueueItem, JObject record)
{
    if (record != null)
    {
        record["Text"] = "This has changed.";
    }    
}
```

### <a name="input---javascript"></a>Entrada: JavaScript

En el ejemplo siguiente se muestra un enlace de entrada de Mobile Apps en un archivo de *function.json* y una [función de JavaScript](functions-reference-node.md) que usa el enlace. Un mensaje de cola que tiene un identificador de registro desencadena la función. La función lee el registro especificado y modifica su propiedad `Text`.

Estos son los datos de enlace del archivo *function.json*:

```json
{
"bindings": [
    {
        "name": "myQueueItem",
        "queueName": "myqueue-items",
        "connection": "",
        "type": "queueTrigger",
        "direction": "in"
    },
    {
        "name": "record",
        "type": "mobileTable",
        "tableName": "MyTable",
        "id": "{queueTrigger}",
        "connection": "My_MobileApp_Url",
        "apiKey": "My_MobileApp_Key",
        "direction": "in"
    }
]
}
```
En la sección de [configuración](#input---configuration) se explican estas propiedades.

Este es el código de JavaScript:

```javascript
module.exports = function (context, myQueueItem) {    
    context.log(context.bindings.record);
    context.done();
};
```

## <a name="input---attributes"></a>Entrada: atributos

En las [bibliotecas de clases de C#](functions-dotnet-class-library.md), use el atributo [MobileTable](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.MobileApps/MobileTableAttribute.cs).

Para información sobre las propiedades de atributo que puede configurar, consulte [la sección de configuración siguiente](#input---configuration).

## <a name="input---configuration"></a>Entrada: configuración

En la siguiente tabla se explican las propiedades de configuración de enlace que se definen en el archivo *function.json* y el atributo `MobileTable`.

|Propiedad de function.json | Propiedad de atributo |Descripción|
|---------|---------|----------------------|
| **type**| N/D | Se debe establecer en "mobileTable"|
| **direction**| N/D |Se debe establecer en "in"|
| **name**| N/D | Nombre del parámetro de entrada en la firma de la función.|
|**tableName** |**TableName**|Nombre de la tabla de datos de la aplicación móvil|
| **id**| **Id** | El identificador único del registro que se va a recuperar. Puede ser estático o se puede basar en el desencadenador que invoca la función. Por ejemplo, si usa un desencadenador de cola para la función, `"id": "{queueTrigger}"` usa el valor de cadena del mensaje de cola como el identificador de registro que se va a recuperar.|
|**connection**|**Connection**|El nombre de una configuración de aplicación que tiene la dirección URL de la aplicación móvil. La función utiliza esta dirección URL para construir las operaciones de REST necesarias en su aplicación móvil. Cree una configuración de aplicación en la aplicación de función que contiene la dirección URL de la aplicación móvil y, luego, especifique el nombre de la configuración de la aplicación en la propiedad `connection` del enlace de entrada. La dirección URL tiene el aspecto siguiente: `http://<appname>.azurewebsites.net`.
|**apiKey**|**ApiKey**|El nombre de la configuración de una aplicación que tiene la clave de API de la aplicación móvil. Proporcione la clave de API si [implementa una clave de API en la aplicación móvil Node.js](https://github.com/Azure/azure-mobile-apps-node/tree/master/samples/api-key) o [implemente una clave de API en la aplicación móvil .NET](https://github.com/Azure/azure-mobile-apps-net-server/wiki/Implementing-Application-Key). Para proporcionar la clave, cree una configuración de la aplicación en la aplicación de la función que contiene la clave de API y, luego, agregue la propiedad `apiKey` en el enlace de entrada con el nombre de la configuración de la aplicación. |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

> [!IMPORTANT]
> No comparta la clave de API con los clientes de aplicación móvil. Solo se debe distribuir de forma segura a los clientes del servicio, como Azure Functions. Azure Functions almacena la información de conexión y las claves de API como configuración de la aplicación de forma que no se protegen en el repositorio de control de código fuente. Esto protege la información confidencial.

## <a name="input---usage"></a>Uso de entradas

En las funciones de C#, cuando se encuentra el registro con el identificador especificado, se pasa al parámetro [JObject](https://www.newtonsoft.com/json/help/html/t_newtonsoft_json_linq_jobject.htm) con nombre. Si el registro no se encuentra, el valor del parámetro es `null`. 

En las funciones de JavaScript, el registro se pasa en el objeto `context.bindings.<name>`. Si el registro no se encuentra, el valor del parámetro es `null`. 

En funciones de C# y F#, los cambios realizados en el registro de entrada (parámetro de entrada) se devuelven automáticamente a la tabla cuando la función termina correctamente. No se puede modificar un registro en las funciones de JavaScript.

## <a name="output"></a>Output

Use el enlace de salida de Mobile Apps para escribir un registro nuevo en una tabla de Mobile Apps.  

## <a name="output---example"></a>Salida: ejemplo

Vea el ejemplo específico del lenguaje:

* [C#](#output---c-example)
* [Script de C# (.csx)](#output---c-script-example)
* [JavaScript](#output---javascript-example)

### <a name="output---c-example"></a>Salida: ejemplo de C#

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que se desencadena mediante un mensaje de cola y crea un registro en una tabla de aplicación móvil.

```csharp
[FunctionName("MobileAppsOutput")]        
[return: MobileTable(ApiKeySetting = "MyMobileAppKey", TableName = "MyTable", MobileAppUriSetting = "MyMobileAppUri")]
public static object Run(
    [QueueTrigger("myqueue-items", Connection = "AzureWebJobsStorage")] string myQueueItem,
    TraceWriter log)
{
    return new { Text = $"I'm running in a C# function! {myQueueItem}" };
}
```

### <a name="output---c-script-example"></a>Salida: ejemplo de script de C#

En el ejemplo siguiente se muestra un enlace de salida de Mobile Apps en un archivo de *function.json* y una [función de script de C#](functions-reference-csharp.md) que usa el enlace. La función se desencadena mediante un mensaje de cola y crea un registro nuevo con un valor codificado de forma rígida para la propiedad `Text`.

Estos son los datos de enlace del archivo *function.json*:

```json
{
"bindings": [
    {
        "name": "myQueueItem",
        "queueName": "myqueue-items",
        "connection": "",
        "type": "queueTrigger",
        "direction": "in"
    },
    {
        "name": "record",
        "type": "mobileTable",
        "tableName": "MyTable",
        "connection": "My_MobileApp_Url",
        "apiKey": "My_MobileApp_Key",
        "direction": "out"
    }
]
}
```

En la sección de [configuración](#output---configuration) se explican estas propiedades.

Este es el código de script de C#:

```cs
public static void Run(string myQueueItem, out object record)
{
    record = new {
        Text = $"I'm running in a C# function! {myQueueItem}"
    };
}
```

### <a name="output---javascript-example"></a>Salida: ejemplo de JavaScript

En el ejemplo siguiente se muestra un enlace de salida de Mobile Apps en un archivo de *function.json* y una [función de JavaScript](functions-reference-node.md) que usa el enlace. La función se desencadena mediante un mensaje de cola y crea un registro nuevo con un valor codificado de forma rígida para la propiedad `Text`.

Estos son los datos de enlace del archivo *function.json*:

```json
{
"bindings": [
    {
        "name": "myQueueItem",
        "queueName": "myqueue-items",
        "connection": "",
        "type": "queueTrigger",
        "direction": "in"
    },
    {
        "name": "record",
        "type": "mobileTable",
        "tableName": "MyTable",
        "connection": "My_MobileApp_Url",
        "apiKey": "My_MobileApp_Key",
        "direction": "out"
    }
],
"disabled": false
}
```

En la sección de [configuración](#output---configuration) se explican estas propiedades.

Este es el código de JavaScript:

```javascript
module.exports = function (context, myQueueItem) {

    context.bindings.record = {
        text : "I'm running in a Node function! Data: '" + myQueueItem + "'"
    }   

    context.done();
};
```

## <a name="output---attributes"></a>Salida: atributos

En las [bibliotecas de clases de C#](functions-dotnet-class-library.md), use el atributo [MobileTable](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.MobileApps/MobileTableAttribute.cs).

Para información sobre las propiedades de atributo que puede configurar, consulte [Salida: configuración](#output---configuration). A continuación, se muestra un ejemplo del atributo `MobileTable` en una signatura de método:

```csharp
[FunctionName("MobileAppsOutput")]        
[return: MobileTable(ApiKeySetting = "MyMobileAppKey", TableName = "MyTable", MobileAppUriSetting = "MyMobileAppUri")]
public static object Run(
    [QueueTrigger("myqueue-items", Connection = "AzureWebJobsStorage")] string myQueueItem,
    TraceWriter log)
{
    ...
}
```

Para obtener un ejemplo completo, consulte [Salida: ejemplo de C#](#output---c-example).

## <a name="output---configuration"></a>Salida: configuración

En la siguiente tabla se explican las propiedades de configuración de enlace que se definen en el archivo *function.json* y el atributo `MobileTable`.

|Propiedad de function.json | Propiedad de atributo |Descripción|
|---------|---------|----------------------|
| **type**| N/D | Se debe establecer en "mobileTable"|
| **direction**| N/D |Debe establecerse en "out"|
| **name**| N/D | Nombre del parámetro de salida en la firma de la función.|
|**tableName** |**TableName**|Nombre de la tabla de datos de la aplicación móvil|
|**connection**|**MobileAppUriSetting**|El nombre de una configuración de aplicación que tiene la dirección URL de la aplicación móvil. La función utiliza esta dirección URL para construir las operaciones de REST necesarias en su aplicación móvil. Cree una configuración de aplicación en la aplicación de función que contiene la dirección URL de la aplicación móvil y, luego, especifique el nombre de la configuración de la aplicación en la propiedad `connection` del enlace de entrada. La dirección URL tiene el aspecto siguiente: `http://<appname>.azurewebsites.net`.
|**apiKey**|**ApiKeySetting**|El nombre de la configuración de una aplicación que tiene la clave de API de la aplicación móvil. Proporcione la clave de API si [implementa una clave de API en el back-end de aplicación móvil de Node.js](https://github.com/Azure/azure-mobile-apps-node/tree/master/samples/api-key) o [implementa una clave de API en el back-end de aplicación móvil de .NET](https://github.com/Azure/azure-mobile-apps-net-server/wiki/Implementing-Application-Key). Para proporcionar la clave, cree una configuración de la aplicación en la aplicación de la función que contiene la clave de API y, luego, agregue la propiedad `apiKey` en el enlace de entrada con el nombre de la configuración de la aplicación. |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

> [!IMPORTANT]
> No comparta la clave de API con los clientes de aplicación móvil. Solo se debe distribuir de forma segura a los clientes del servicio, como Azure Functions. Azure Functions almacena la información de conexión y las claves de API como configuración de la aplicación de forma que no se protegen en el repositorio de control de código fuente. Esto protege la información confidencial.

## <a name="output---usage"></a>Uso de salidas

En funciones de script de C#, use un parámetro de salida con nombre de tipo `out object` para acceder al registro de salida. En las bibliotecas de clases de C#, el atributo `MobileTable` se puede usar con cualquiera de los tipos siguientes:

* `ICollector<T>` o `IAsyncCollector<T>`, donde `T` es `JObject` o cualquier tipo con una propiedad `public string Id`.
* `out JObject`
* `out T` o `out T[]`, donde `T` es cualquier tipo con una propiedad `public string Id`.

En las funciones de Node.js, use `context.bindings.<name>` para acceder al registro de salida.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre desencadenadores y enlaces de Azure Functions](functions-triggers-bindings.md)
