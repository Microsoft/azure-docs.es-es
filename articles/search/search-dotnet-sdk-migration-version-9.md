---
title: Actualización a la versión 9 del SDK de .NET para Azure Search
titleSuffix: Azure Cognitive Search
description: Migre el código a la versión 9 del SDK de Azure Search para .NET a partir de versiones anteriores. Obtenga información sobre las novedades y los cambios de código necesarios.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 11/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 7726877efcb72caf5bb4c92e2e1c65e5df68f270
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89002715"
---
# <a name="upgrade-to-azure-search-net-sdk-version-9"></a>Actualización a la versión 9 del SDK de .NET para Azure Search

Si usa la versión preliminar 7.0 o anterior del [SDK de .NET para Azure Search](/dotnet/api/overview/azure/search), este artículo le ayudará a actualizar la aplicación para que use la versión 9.

> [!NOTE]
> Si quiere usar la versión 8.0-preview para evaluar las características que aún no están disponibles con carácter general, también puede seguir las instrucciones de este artículo para actualizar a la versión 8.0-preview desde versiones anteriores.

Para obtener un tutorial más general del SDK que incluya ejemplos, vea [Cómo usar Azure Search desde una aplicación .NET](search-howto-dotnet-sdk.md).

La versión 9 del SDK de .NET para Azure Search contiene muchos cambios respecto a las versiones anteriores. Algunos de estos cambios son importantes, pero solo deberían requerir modificaciones relativamente pequeñas en el código. Vea [Pasos para actualizar](#UpgradeSteps) , a fin de obtener instrucciones sobre cómo cambiar el código para usar la nueva versión del SDK.

> [!NOTE]
> Si usa la versión 4.0-preview o anterior, debe actualizar primero a la versión 5 y luego a la versión 9. Consulte [Actualización a la versión 5 del SDK de .NET para Azure Search](search-dotnet-sdk-migration-version-5.md) para obtener instrucciones.
>
> La instancia del servicio Azure Search es compatible con varias versiones de API de REST, incluida la más reciente. Puede seguir usando una versión aunque no sea la más reciente, pero es recomendable que migre el código para usar la versión más actualizada. Cuando se usa la API de REST, debe especificar la versión de API en cada solicitud realizada a través del parámetro api-version. Al usar el SDK de .NET, la versión del SDK que usa determina la versión correspondiente de la API de REST. Si usa un SDK anterior, puede seguir ejecutando ese código sin realizar ningún cambio, incluso si el servicio se ha actualizado para admitir una versión más reciente de la API.

<a name="WhatsNew"></a>

## <a name="whats-new-in-version-9"></a>Novedades de la versión 9
La versión 9 del SDK de .NET de Azure Search tiene como destino la versión 2019-05-06 de la API de REST de Azure Search, con las siguientes características:

* El [enriquecimiento con inteligencia artificial](cognitive-search-concept-intro.md) es la capacidad de extraer el texto de imágenes, blobs y otros orígenes de datos no estructurados, enriqueciendo el contenido para facilitar las búsquedas en un índice de Azure Search.
* La compatibilidad con [tipos complejos](search-howto-complex-data-types.md) le permite modelar casi cualquier estructura anidada JSON en un índice de Azure Search.
* [Autocomplete](search-autocomplete-tutorial.md) proporciona una alternativa a la API de la función **Sugerir** para implementar el comportamiento de búsqueda mientras se escribe. Autocomplete "finaliza" la palabra o frase que un usuario está escribiendo actualmente.
* El [modo de análisis de JsonLines](search-howto-index-json-blobs.md), parte de la indexación de Azure Blob, crea un documento de búsqueda por entidad JSON que está separado por una línea nueva.

### <a name="new-preview-features-in-version-80-preview"></a>Nuevas características de vista previa en la versión 8.0-preview
La versión 8.0-preview del SDK de .NET para Azure Search tiene como destino la versión de la API 2017-11-11-Preview. Esta versión incluye las mismas características de la versión 9, además de:

* [Claves de cifrado administradas por el cliente](search-security-manage-encryption-keys.md) para el cifrado en reposo del lado del servicio es una nueva característica de vista previa. Además del cifrado en reposo integrado administrado por Microsoft, puede aplicar una capa adicional de cifrado, donde será el titular exclusivo de las claves de cifrado.

<a name="UpgradeSteps"></a>

## <a name="steps-to-upgrade"></a>Pasos para actualizar
En primer lugar, actualice la referencia de NuGet para `Microsoft.Azure.Search` mediante la Consola del Administrador de paquetes NuGet, o bien haga clic con el botón derecho en las referencias del proyecto y seleccione "Administrar paquetes NuGet" en Visual Studio.

Una vez que NuGet ha descargado los nuevos paquetes y sus dependencias, recompile el proyecto. Dependiendo de cómo se estructura el código, se podrá volver a compilar correctamente. Si lo consigue, ya estará listo para empezar.

Si se produce un error en la compilación, deberá corregir cada error de compilación. Consulte [Cambios importantes en la versión 9](#ListOfChanges) para obtener más información sobre cómo resolver cada posible error de compilación.

Puede ver las advertencias de compilación adicionales relacionadas con propiedades o métodos obsoletos. Las advertencias incluyen instrucciones sobre qué utilizar en lugar de la característica en desuso. Por ejemplo, si la aplicación usa la propiedad `DataSourceType.DocumentDb`, recibirá una advertencia que indica "This member is deprecated. Use CosmosDb instead" ("Este miembro está en desuso. Use CosmosDb en su lugar").

Una vez que haya solucionado los errores o las advertencias de compilación, puede efectuar cambios en la aplicación para aprovechar las ventajas de la nueva funcionalidad, si lo desea. Las nuevas características en el SDK se detallan en [Novedades de la versión 9](#WhatsNew).

<a name="ListOfChanges"></a>

## <a name="breaking-changes-in-version-9"></a>Cambios importantes en la versión 9

Hay varios cambios importantes en la versión 9 que pueden requerir cambios de código además de la recompilación de la aplicación.

> [!NOTE]
> La lista de cambios siguiente no es exhaustiva. Algunos cambios probablemente no tendrán como resultado errores de compilación, pero técnicamente son importantes, ya que anulan la compatibilidad binaria con los ensamblados que dependen de las versiones anteriores de los ensamblados del SDK de .NET de Azure Search. Estos cambios no se muestran a continuación. Vuelva a generar la aplicación al actualizar a la versión 9 para evitar problemas de compatibilidad binaria.

### <a name="immutable-properties"></a>Propiedades inmutables

Las propiedades públicas de varias clases de modelos ahora son inmutables. Si necesita crear instancias personalizadas de estas clases para realizar pruebas, puede usar los nuevos constructores con parámetros:

  - `AutocompleteItem`
  - `DocumentSearchResult`
  - `DocumentSuggestResult`
  - `FacetResult`
  - `SearchResult`
  - `SuggestResult`

### <a name="changes-to-field"></a>Cambios a la clase Field

La clase `Field` ha cambiado, ahora también puede representar campos complejos.

Las siguientes propiedades `bool` ahora aceptan valores NULL.

  - `IsFilterable`
  - `IsFacetable`
  - `IsSearchable`
  - `IsSortable`
  - `IsRetrievable`
  - `IsKey`

Esto se debe a que estas propiedades ahora deben ser `null` en el caso de campos complejos. Si tiene código que lee estas propiedades, tiene que estar preparado para controlar `null`. Tenga en cuenta que las demás propiedades de `Field` siempre han aceptado y seguirán aceptando valores NULL, y algunas de ellas también serán `null` en el caso de campos complejos, específicamente las siguientes:

  - `Analyzer`
  - `SearchAnalyzer`
  - `IndexAnalyzer`
  - `SynonymMaps`

El constructor sin parámetros de `Field` se ha hecho `internal`. A partir de ahora, cada clase `Field` requiere un tipo de datos y un nombre explícitos en el momento de la construcción.

### <a name="simplified-batch-and-results-types"></a>Tipos de lotes y resultados simplificados

En la versión 7.0-preview y anteriores, las distintas clases que encapsulan los grupos de documentos estaban estructuradas en jerarquías de clases paralelas:

  -  `DocumentSearchResult` y `DocumentSearchResult<T>` heredadas de `DocumentSearchResultBase`
  -  `DocumentSuggestResult` y `DocumentSuggestResult<T>` heredadas de `DocumentSuggestResultBase`
  -  `IndexAction` y `IndexAction<T>` heredadas de `IndexActionBase`
  -  `IndexBatch` y `IndexBatch<T>` heredadas de `IndexBatchBase`
  -  `SearchResult` y `SearchResult<T>` heredadas de `SearchResultBase`
  -  `SuggestResult` y `SuggestResult<T>` heredadas de `SuggestResultBase`

Los tipos derivados sin un parámetro de tipo genérico estaban diseñados para usarse en escenarios de "tipo dinámico" y presuponían el uso del tipo `Document`.

A partir de la versión 8.0-preview, se han quitado todas las clases base y las clases derivadas no genéricas. Para escenarios de tipo dinámico, puede usar `IndexBatch<Document>`, `DocumentSearchResult<Document>`, etc.
 
### <a name="removed-extensibleenum"></a>Clase ExtensibleEnum quitada

Se ha quitado la clase base `ExtensibleEnum`. Todas las clases que derivan de ella ahora son estructuras como, por ejemplo, `AnalyzerName`, `DataType` y `DataSourceType`. También se han quitado sus métodos `Create`. Solo puede quitar las llamadas a `Create`, puesto que estos tipos son implícitamente convertibles de cadenas. Si esto da como resultado errores del compilador, puede invocar explícitamente el operador de conversión a través de la conversión para eliminar la ambigüedad de los tipos. Por ejemplo, puede cambiar un código como este:

```csharp
var index = new Index()
{
    Fields = new[]
    {
        new Field("id", DataType.String) { IsKey = true },
        new Field("message", AnalyzerName.Create("my_email_analyzer")) { IsSearchable = true }
    },
    ...
}
```

a esto:

```csharp
var index = new Index()
{
    Fields = new[]
    {
        new Field("id", DataType.String) { IsKey = true },
        new Field("message", (AnalyzerName)"my_email_analyzer") { IsSearchable = true }
    },
    ...
}
```

Las propiedades que mantienen los valores opcionales de estos tipos tienen ahora un tipo explícito que acepta valores NULL para que sigan siendo opcionales.

### <a name="removed-facetresults-and-hithighlights"></a>Clases FacetResults y HitHighlights quitadas

Se han quitado las clases `FacetResults` y `HitHighlights`. Los resultados de faceta ahora tienen el tipo `IDictionary<string, IList<FacetResult>>` y los elementos destacados el tipo `IDictionary<string, IList<string>>`. Una forma rápida de resolver errores de compilación provocados por este cambio consiste en agregar alias `using` en la parte superior de cada archivo que use los tipos quitados. Por ejemplo:

```csharp
using FacetResults = System.Collections.Generic.IDictionary<string, System.Collections.Generic.IList<Models.FacetResult>>;
using HitHighlights = System.Collections.Generic.IDictionary<string, System.Collections.Generic.IList<string>>;
```

### <a name="change-to-synonymmap"></a>Cambio a SynonymMap 

El constructor `SynonymMap` ya no tiene un parámetro `enum` para `SynonymMapFormat`. Esta enumeración solo tenía un valor y, por tanto, era redundante. Si como resultado se observan errores de compilación, basta con eliminar las referencias al parámetro `SynonymMapFormat`.

### <a name="miscellaneous-model-class-changes"></a>Cambios varios de la clase de modelo

La propiedad `AutocompleteMode` de `AutocompleteParameters` ya no acepta valores NULL. Si tiene código que asigna esta propiedad en `null`, puede simplemente quitarlo y la propiedad automáticamente se inicializará en el valor predeterminado.

El orden de los parámetros para el constructor `IndexAction` ha cambiado, ahora este constructor se genera automáticamente. En lugar de usar el constructor, se recomienda usar los métodos de generador `IndexAction.Upload`, `IndexAction.Merge`, etc.

### <a name="removed-preview-features"></a>Características quitadas de la versión preliminar

Si va a actualizar desde la versión 8.0-preview a la versión 9, tenga en cuenta que se ha quitado el cifrado con claves administradas por el cliente, ya que esta característica está aún en versión preliminar. En concreto, se han quitado las propiedades `EncryptionKey` de `Index` y `SynonymMap`.

Si la aplicación tiene una dependencia fuerte de esta característica, no podrá actualizar a la versión 9 del SDK de .NET para Azure Search. Podrá seguir usando la versión 8.0-preview. Sin embargo, tenga en cuenta que **no se recomienda usar SDK de versión preliminar en aplicaciones de producción**. Las características de versión preliminar son solo para evaluación y pueden cambiar.

> [!NOTE]
> Si ha creado índices cifrados o asignaciones de sinónimos con la versión 8.0-preview del SDK, todavía podrá usar y modificar sus definiciones con la versión 9 del SDK sin que afecte negativamente a su estado de cifrado. La versión 9 del SDK no enviará la propiedad `encryptionKey` a la API de REST y, como resultado, la API de REST no cambiará el estado de cifrado del recurso. 

### <a name="behavioral-change-in-data-retrieval"></a>Cambio de comportamiento en la recuperación de datos

Si usa las API de "tipo dinámico" `Search`, `Suggest` o `Get` que devuelven instancias de tipo `Document`, tenga en cuenta que ahora estas deserializan las matrices JSON vacías a `object[]` en lugar de `string[]`.

## <a name="conclusion"></a>Conclusión
Si necesita más detalles sobre el uso del SDK de .NET para Azure Search, consulte la [guía de .NET](search-howto-dotnet-sdk.md).

Agradecemos sus comentarios sobre el SDK. Si tiene algún problema, no dude en pedirnos ayuda en [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-search). Si encuentra un error, puede enviarlo al [repositorio de GitHub del SDK de .NET para Azure](https://github.com/Azure/azure-sdk-for-net/issues). Asegúrese de agregar "[Azure Search]" como prefijo en el título del problema.

Gracias por usar Azure Search.