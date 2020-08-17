---
title: Desencadenador de preparación de Azure Functions
description: Aprenda a usar el desencadenador de preparación en Azure Functions.
documentationcenter: na
author: craigshoemaker
manager: gwallace
keywords: Azure Functions, funciones, procesamiento de eventos, preparación, arranque en frío, premium, proceso dinámico, arquitectura sin servidor
ms.service: azure-functions
ms.topic: reference
ms.date: 11/08/2019
ms.author: cshoe
ms.openlocfilehash: dade31fd9a5a75c875274ea45607fea81a2de027
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87383722"
---
# <a name="azure-functions-warm-up-trigger"></a>Desencadenador de preparación de Azure Functions

En este artículo se explica cómo trabajar con el desencadenador de preparación en Azure Functions. El desencadenador de preparación solo se admite para las aplicaciones de funciones que se ejecutan en un [plan Premium](functions-premium-plan.md). Un desencadenador de preparación se invoca cuando se agrega una instancia para escalar una aplicación de funciones en ejecución. Puede usar un desencadenador de preparación para cargar con antelación las dependencias personalizadas durante el [proceso de preparación](./functions-premium-plan.md#pre-warmed-instances), de modo que las funciones estén listas para empezar a procesar solicitudes inmediatamente. 

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

## <a name="packages---functions-2x-and-higher"></a>Paquetes: Functions 2.x y versiones posteriores

El paquete NuGet [Microsoft.Azure.WebJobs.Extensions](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions), versión **3.0.5 o posterior**. El código fuente del paquete está en el repositorio [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.Http/) de GitHub. 

[!INCLUDE [functions-package](../../includes/functions-package-auto.md)]

## <a name="trigger"></a>Desencadenador

El desencadenador de preparación permite definir una función que se ejecutará en una nueva instancia cuando se agregue a la aplicación en ejecución. Puede usar una función de preparación para abrir conexiones, cargar dependencias o ejecutar cualquier otra lógica personalizada antes de que la aplicación comience a recibir tráfico. 

El desencadenador de preparación está diseñado para crear dependencias compartidas que usarán las demás funciones de la aplicación. [Vea ejemplos de dependencias compartidas aquí](./manage-connections.md#client-code-examples).

Tenga en cuenta que solo se llama al desencadenador de preparación durante las operaciones de escalado horizontal, no durante los reinicios u otros inicios sin escala. Debe asegurarse de que la lógica pueda cargar todas las dependencias necesarias sin usar el desencadenador de preparación. La carga diferida es un buen patrón para lograrlo.

## <a name="trigger---example"></a>Desencadenador: ejemplo

# <a name="c"></a>[C#](#tab/csharp)

En el ejemplo siguiente se muestra una [función C# ](functions-dotnet-class-library.md) que se ejecutará en cada nueva instancia cuando se agregue a la aplicación. No se requiere un atributo de valor devuelto.


* La función debe denominarse ```warmup``` (sin distinguir entre mayúsculas y minúsculas) y solo puede haber una función de preparación por aplicación.
* Para usar la preparación como una función de la biblioteca de clases .NET, asegúrese de que tiene una referencia de paquete para **Microsoft.Azure.WebJobs.Extensions >= 3.0.5**
    * ```<PackageReference Include="Microsoft.Azure.WebJobs.Extensions" Version="3.0.5" />```


Los comentarios del marcador de posición muestran el lugar de la aplicación en el que se deben declarar e inicializar las dependencias compartidas. 
[Obtenga más información sobre las dependencias compartidas aquí](./manage-connections.md#client-code-examples).

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Extensions.Logging;
 
namespace WarmupSample
{

    //Declare shared dependencies here

    public static class Warmup
    {
        [FunctionName("Warmup")]
        public static void Run([WarmupTrigger()] WarmupContext context,
            ILogger log)
        {
            //Initialize shared dependencies here
            
            log.LogInformation("Function App instance is warm 🌞🌞🌞");
        }
    }
}
```
# <a name="c-script"></a>[Script de C#](#tab/csharp-script)


En el ejemplo siguiente se muestra un desencadenador de preparación en un archivo *function.json* y una [función de script C#](functions-reference-csharp.md) que se ejecutará en cada nueva instancia cuando se agregue a la aplicación.

La función debe denominarse ```warmup``` (sin distinguir entre mayúsculas y minúsculas) y solo puede haber una función de preparación por aplicación.

Este es el archivo *function.json*:

```json
{
    "bindings": [
        {
            "type": "warmupTrigger",
            "direction": "in",
            "name": "warmupContext"
        }
    ]
}
```

En la sección de [configuración](#trigger---configuration) se explican estas propiedades.

Este es el código de script de C# que se enlaza a `HttpRequest`:

```cs
public static void Run(ILogger log)
{
    log.LogInformation("Function App instance is warm 🌞🌞🌞");  
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

En el ejemplo siguiente se muestra un desencadenador de preparación en un archivo *function.json* y una [función JavaScript](functions-reference-node.md) que se ejecutará en cada nueva instancia cuando se agregue a la aplicación.

La función debe denominarse ```warmup``` (sin distinguir entre mayúsculas y minúsculas) y solo puede haber una función de preparación por aplicación.

Este es el archivo *function.json*:

```json
{
    "bindings": [
        {
            "type": "warmupTrigger",
            "direction": "in",
            "name": "warmupContext"
        }
    ]
}
```

En la sección de [configuración](#trigger---configuration) se explican estas propiedades.

Este es el código de JavaScript:

```javascript
module.exports = async function (context, warmupContext) {
    context.log('Function App instance is warm 🌞🌞🌞');
};
```

# <a name="python"></a>[Python](#tab/python)

En el ejemplo siguiente se muestra un desencadenador de preparación en un archivo *function.json* y una [función Python](functions-reference-python.md) que se ejecutará en cada nueva instancia cuando se agregue a la aplicación.

La función debe denominarse ```warmup``` (sin distinguir entre mayúsculas y minúsculas) y solo puede haber una función de preparación por aplicación.

Este es el archivo *function.json*:

```json
{
    "bindings": [
        {
            "type": "warmupTrigger",
            "direction": "in",
            "name": "warmupContext"
        }
    ]
}
```

En la sección de [configuración](#trigger---configuration) se explican estas propiedades.

Este es el código de Python:

```python
import logging
import azure.functions as func


def main(warmupContext: func.Context) -> None:
    logging.info('Function App instance is warm 🌞🌞🌞')
```

# <a name="java"></a>[Java](#tab/java)

En el ejemplo siguiente, se muestra un desencadenador de calentamiento que se ejecuta cada vez que se agrega una nueva instancia a la aplicación.

La función debe denominarse `warmup` (sin distinguir entre mayúsculas y minúsculas) y solo puede haber una función de preparación por aplicación.

```java
@FunctionName("Warmup")
public void run( ExecutionContext context) {
       context.getLogger().info("Function App instance is warm 🌞🌞🌞");
}
```

---

## <a name="trigger---attributes"></a>Desencadenador: atributos

En las [bibliotecas de clases de C#](functions-dotnet-class-library.md), está disponible el atributo `WarmupTrigger` para configurar la función.

# <a name="c"></a>[C#](#tab/csharp)

En este ejemplo se muestra cómo usar el atributo [warmup](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/dev/src/WebJobs.Extensions/Extensions/Warmup/Trigger/WarmupTriggerAttribute.cs).

Tenga en cuenta que la función debe denominarse ```Warmup``` y solo puede haber una función de preparación por aplicación.

```csharp
 [FunctionName("Warmup")]
        public static void Run(
            [WarmupTrigger()] WarmupContext context, ILogger log)
        {
            ...
        }
```

Para obtener un ejemplo completo, vea el [ejemplo del desencadenador](#trigger---example).

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

El script de C# no admite atributos.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

JavaScript no admite atributos.

# <a name="python"></a>[Python](#tab/python)

Python no admite atributos.

# <a name="java"></a>[Java](#tab/java)

El desencadenador de preparación no se admite en Java como atributo.

---

## <a name="trigger---configuration"></a>Desencadenador: configuración

En la siguiente tabla se explican las propiedades de configuración de enlace que se definen en el archivo *function.json* y el atributo `WarmupTrigger`.

|Propiedad de function.json | Propiedad de atributo |Descripción|
|---------|---------|----------------------|
| **type** | N/D| Requerida: se debe establecer en `warmupTrigger`. |
| **direction** | N/D| Requerida: se debe establecer en `in`. |
| **name** | N/D| Requerido: nombre de la variable que se usa en el código de función.|

## <a name="trigger---usage"></a>Desencadenador: uso

No se proporciona información adicional para una función desencadenada de preparación cuando se invoca.

## <a name="trigger---limits"></a>Desencadenador: límites

* El desencadenador de preparación solo está disponible para las aplicaciones que se ejecutan en el [plan Premium](./functions-premium-plan.md).
* Solo se llama al desencadenador de preparación durante las operaciones de escalado horizontal, no durante los reinicios u otros inicios sin escala. Debe asegurarse de que la lógica pueda cargar todas las dependencias necesarias sin usar el desencadenador de preparación. La carga diferida es un buen patrón para lograrlo.
* No se puede invocar el desencadenador de preparación cuando ya hay una instancia en ejecución.
* Solo puede haber una función de desencadenador de preparación por cada aplicación de funciones.

## <a name="next-steps"></a>Pasos siguientes

[Más información sobre desencadenadores y enlaces de Azure Functions](functions-triggers-bindings.md)
