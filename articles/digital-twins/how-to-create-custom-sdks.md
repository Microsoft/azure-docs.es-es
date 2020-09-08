---
title: Creación de SDK personalizados para Azure Digital Twins con AutoRest
titleSuffix: Azure Digital Twins
description: Vea cómo generar SDK personalizados para usar Azure Digital Twins con lenguajes distintos de C#.
author: baanders
ms.author: baanders
ms.date: 4/24/2020
ms.topic: how-to
ms.service: digital-twins
ms.custom: devx-track-javascript
ms.openlocfilehash: 3cf14ce3e8ef9b1d783191fe6c01c5e311d57786
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88855952"
---
# <a name="create-custom-sdks-for-azure-digital-twins-using-autorest"></a>Creación de SDK personalizados para Azure Digital Twins con AutoRest

En este momento, el único SDK de plano de datos publicado para interactuar con las API de Azure Digital Twins es para .NET (C#). Puede leer sobre el SDK de .NET y las API en general, en [*Procedimiento: las API y los SDK de Azure Digital Twins*](how-to-use-apis-sdks.md). Si está trabajando en otro lenguaje, este artículo le mostrará cómo generar su propio SDK de plano de datos en el lenguaje de su elección mediante AutoRest.

>[!NOTE]
> También puede usar AutoRest para generar un SDK de plano de control si lo desea. Para ello, complete los pasos descritos en este artículo con el [archivo Swagger (OpenAPI) del plano de control](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/digitaltwins/resource-manager/Microsoft.DigitalTwins/preview/2020-03-01-preview) en lugar del plano de datos.

## <a name="set-up-your-machine"></a>Configuración de la máquina

Para generar un SDK, necesitará:
* [AutoRest](https://github.com/Azure/autorest) versión 2.0.4413 (la versión 3 no se admite actualmente)
* [Node.js](https://nodejs.org) como requisito previo para AutoRest
* El [archivo Swagger (OpenAPI) del plano de datos](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/digitaltwins/data-plane/Microsoft.DigitalTwins/preview/2020-05-31-preview) de Azure Digital Twins titulado *digitaltwins.json*, y su correspondiente carpeta de ejemplos. Descargue el archivo Swagger y su carpeta de ejemplos en la máquina local.

Una vez que la máquina esté equipada con todos los elementos de la lista anterior, estará listo para usar AutoRest para crear el SDK.

## <a name="create-the-sdk-with-autorest"></a>Creación del SDK con AutoRest 

Si tiene Node.js instalado, puede ejecutar este comando para asegurarse de que tiene instalada la versión correcta de AutoRest:
```cmd/sh
npm install -g autorest@2.0.4413
```

Para ejecutar AutoRest con el archivo Swagger de Azure Digital Twins, siga estos pasos:
1. Copie el archivo Swagger de Azure Digital Twins y su correspondiente carpeta de ejemplos en un directorio de trabajo.
2. Use una ventana del símbolo del sistema para cambiar a ese directorio de trabajo.
3. Ejecute AutoRest con el siguiente comando. Reemplace el marcador de posición `<language>` por el lenguaje que prefiera: `python`, `java`, `go`, etc. (Puede encontrar la lista completa de opciones en el archivo [Léame de AutoRest](https://github.com/Azure/autorest)).

```cmd/sh
autorest --input-file=digitaltwins.json --<language> --output-folder=ADTApi --add-credentials --azure-arm --namespace=ADTApi
```

Como resultado, verá una nueva carpeta denominada *ADTApi* en el directorio de trabajo. Los archivos de SDK generados tendrán el espacio de nombres *ADTApi*. Seguirá usando ese espacio de nombres en el resto de los ejemplos de uso de este artículo.

AutoRest es compatible con una amplia gama de generadores de código de lenguaje.

## <a name="add-the-sdk-to-a-visual-studio-project"></a>Agregar el SDK a un proyecto de Visual Studio

Puede incluir los archivos generados mediante AutoRest directamente en una solución de .NET. Sin embargo, es probable que quiera incluir el SDK de Azure Digital Twins en varios proyectos independientes (aplicaciones cliente, aplicaciones de Azure Functions, etc.). Por esta razón, puede ser útil crear un proyecto independiente (una biblioteca de clases .NET) a partir de los archivos generados. A continuación, puede incluir este proyecto de biblioteca de clases en varias soluciones como referencia de proyecto.

En esta sección se proporcionan instrucciones sobre cómo compilar el SDK como una biblioteca de clases, que es su propio proyecto y se puede incluir en otros proyectos. Estos pasos se basan en **Visual Studio** (puede instalar la versión más reciente desde [aquí](https://visualstudio.microsoft.com/downloads/)).

He aquí los pasos:

1. Creación de una nueva solución de Visual Studio para una biblioteca de clases
2. Use *ADTApi* como nombre del proyecto.
3. En el Explorador de soluciones, seleccione con el botón derecho el proyecto *ADTApi* de la solución generada y elija *Agregar > Elemento existente...*
4. Busque la carpeta en la que generó el SDK y seleccione los archivos en el nivel raíz.
5. Presione "Aceptar".
6. Agregue una carpeta al proyecto (haga clic con el botón derecho para seleccionar el proyecto en el Explorador de soluciones y elija *Agregar > Nueva carpeta*).
7. Asigne a la carpeta el nombre *Models*.
8. En el Explorador de soluciones, seleccione con el botón derecho la carpeta *Models* y elija *Agregar > Elemento existente...*
9. Seleccione los archivos de la carpeta *Models* del SDK generado y presione "Aceptar".

Para compilar el SDK correctamente, el proyecto necesitará estas referencias:
* `Microsoft.Rest.ClientRuntime`
* `Microsoft.Rest.ClientRuntime.Azure`

Para agregarlas, abra *Herramientas > Administrador de paquetes NuGet > Administrar paquetes NuGet para la solución...*

1. En el panel, asegúrese de que esté seleccionada la pestaña *Examinar*.
2. Busque *Microsoft.Rest*.
3. Seleccione los paquetes `ClientRuntime` y `ClientRuntime.Azure`, y agréguelos a la solución.

Ahora puede compilar el proyecto e incluirlo como una referencia de proyecto en cualquier aplicación de Azure Digital Twins que escriba.

## <a name="general-guidelines-for-generated-sdks"></a>Directrices generales para los SDK generados

En esta sección se incluye información general sobre el SDK generado, así como instrucciones para usarlo.

### <a name="synchronous-and-asynchronous-calls"></a>Llamadas sincrónicas y asincrónicas

Todas las funciones del SDK cuentan con versiones sincrónicas y asincrónicas.

### <a name="typed-and-untyped-data"></a>Datos con tipo y sin tipo

Por lo general, las llamadas API de REST devuelven objetos fuertemente tipados. Sin embargo, dado que Azure Digital Twins permite a los usuarios definir sus propios tipos personalizados para gemelos, no hay ninguna manera de predefinir datos devueltos estáticos para muchas llamadas de Azure Digital Twins. En su lugar, las API devuelven tipos de contenedor fuertemente tipados donde corresponde, y los datos de gemelos con tipos personalizados están en objetos Json.NET (que se usan siempre que aparece el tipo de datos "object" en las firmas de API). Puede convertir estos objetos según corresponda en el código.

### <a name="error-handling"></a>Control de errores

Siempre que se produzca un error en el SDK (incluidos los errores HTTP, como 404), el SDK devolverá una excepción. Por esta razón, es importante encapsular todas las llamadas API dentro de bloques try/catch.

A continuación se muestra un fragmento de código que intenta agregar un gemelo y detecta cualquier error en este proceso:

```csharp
try
{
    await client.DigitalTwins.AddAsync(id, initData);
    Console.WriteLine($"Created a twin successfully: {id}");
}
catch (ErrorResponseException e)
{
    Console.WriteLine($"*** Error creating twin {id}: {e.Response.StatusCode}"); 
}
```

### <a name="paging"></a>Paginación

AutoRest genera dos tipos de patrones de paginación para el SDK:
* Uno para todas las API, excepto la API de consulta
* Uno para la API de consulta

En el patrón de paginación que no es de consulta, hay dos versiones de cada llamada:
* Una versión para establecer la llamada inicial (como `DigitalTwins.ListEdges()`)
* Una versión para obtener las páginas siguientes. Estas llamadas tienen el sufijo "Next" (como `DigitalTwins.ListEdgesNext()`)

A continuación se incluye un fragmento de código que muestra cómo recuperar una lista paginada de relaciones de salida de Azure Digital Twins:
```csharp
try
{
    // List to hold the results in
    List<object> relList = new List<object>();
    // Enumerate the IPage object returned to get the results
    // ListAsync will throw if an error occurs
    IPage<object> relPage = await client.DigitalTwins.ListEdgesAsync(id);
    relList.AddRange(relPage);
    // If there are more pages, the NextPageLink in the page is set
    while (relPage.NextPageLink != null)
    {
        // Get more pages...
        relPage = await client.DigitalTwins.ListEdgesNextAsync(relPage.NextPageLink);
        relList.AddRange(relPage);
    }
    Console.WriteLine($"Found {relList.Count} relationships on {id}");
    // Do something with each object found
    // As relationships are custom types, they are JSON.Net types
    foreach (JObject r in relList)
    {
        string relId = r.Value<string>("$edgeId");
        string relName = r.Value<string>("$relationship");
        Console.WriteLine($"Found relationship {relId} from {id}");
    }
}
catch (ErrorResponseException e)
{
    Console.WriteLine($"*** Error retrieving relationships on {id}: {e.Response.StatusCode}");
}
```

El segundo patrón solo se genera para la API de consulta. Usa `continuationToken` explícitamente.

A continuación, se indica un ejemplo con este patrón:

```csharp
string query = "SELECT * FROM digitaltwins";
string conToken = null; // continuation token from the query
int page = 0;
try
{
    // Repeat the query while there are pages
    do
    {
        QuerySpecification spec = new QuerySpecification(query, conToken);
        QueryResult qr = await client.Query.QueryTwinsAsync(spec);
        page++;
        Console.WriteLine($"== Query results page {page}:");
        if (qr.Items != null)
        {
            // Query returns are JObjects
            foreach(JObject o in qr.Items)
            {
                string twinId = o.Value<string>("$dtId");
                Console.WriteLine($"  Found {twinId}");
            }
        }
        Console.WriteLine($"== End query results page {page}");
        conToken = qr.ContinuationToken;
    } while (conToken != null);
} catch (ErrorResponseException e)
{
    Console.WriteLine($"*** Error in twin query: ${e.Response.StatusCode}");
}
```

## <a name="next-steps"></a>Pasos siguientes

Siga los pasos para crear una aplicación cliente donde pueda usar el SDK:
* [*Tutorial: Programación de una aplicación cliente*](tutorial-code.md)