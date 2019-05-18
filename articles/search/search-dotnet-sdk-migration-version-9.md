---
title: Actualice a SDK .NET de Azure Search versión 9 - Azure Search
description: Migrar el código para el SDK de .NET de Azure Search versión 9 desde versiones anteriores. Obtenga información sobre las novedades y los cambios de código necesarios.
author: brjohnstmsft
manager: jlembicz
services: search
ms.service: search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 05/10/2019
ms.author: brjohnst
ms.custom: seodec2018
ms.openlocfilehash: a59deed4ac0cec669ddc5e0335f7274586c702e8
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65541762"
---
# <a name="upgrade-to-the-azure-search-net-sdk-version-9"></a>Actualizar a la versión 9 SDK de .NET de Azure Search

Si está usando la versión 7.0-preview o anterior de la [SDK para .NET de Azure Search](https://aka.ms/search-sdk), este artículo le ayudará a actualizar la aplicación para usar la versión 9.

> [!NOTE]
> Si desea usar la versión 8.0-versión preliminar para evaluar las características que aún no están disponibles con carácter general, también puede seguir las instrucciones de este artículo para actualizar a la 8.0-preview de las versiones anteriores.

Para obtener un tutorial más general del SDK que incluya ejemplos, vea [Cómo usar Azure Search desde una aplicación .NET](search-howto-dotnet-sdk.md).

9 de la versión del SDK de .NET de Azure Search contiene muchos cambios de versiones anteriores. Algunos de estos cambios son importantes, pero solo deben requerir relativamente pequeños cambios en el código. Vea [Pasos para actualizar](#UpgradeSteps) , a fin de obtener instrucciones sobre cómo cambiar el código para usar la nueva versión del SDK.

> [!NOTE]
> Si se usa la versión 4.0-preview o anterior, debe actualizar primero a la versión 5 y, a continuación, actualice a la versión 9. Consulte [actualizar para el SDK de .NET de Azure Search versión 5](search-dotnet-sdk-migration-version-5.md) para obtener instrucciones.
>
> La instancia del servicio Azure Search es compatible con varias versiones de API de REST, incluida la más reciente. Puede seguir usando una versión aunque no sea la más reciente, pero es recomendable que migre el código para usar la versión más actualizada. Cuando se usa la API de REST, debe especificar la versión de API en cada solicitud realizada a través del parámetro api-version. Al usar el SDK de .NET, la versión del SDK que usa determina la versión correspondiente de la API de REST. Si usa un SDK anterior, puede seguir ejecutando ese código sin realizar ningún cambio, incluso si el servicio se ha actualizado para admitir una versión más reciente de la API.

<a name="WhatsNew"></a>

## <a name="whats-new-in-version-9"></a>Novedades de la versión 9
Versión 9 del SDK de .NET de Azure Search tiene como destino la última versión disponible con carácter general de la API de REST de Azure Search, concretamente 2019-05-06. Esto hace posible el uso de características nuevas de Azure Search desde una aplicación. NET, incluidas las siguientes:

* [Búsqueda cognitiva](cognitive-search-concept-intro.md) es una característica de inteligencia artificial en Azure Search, que se usa para extraer el texto de imágenes, blobs y otros orígenes de datos no estructurados - enriquecer el contenido para facilitar su búsqueda más de un índice de Azure Search.
* Compatibilidad con [tipos complejos](search-howto-complex-data-types.md) le permite modelar casi cualquier estructura anidada JSON en un índice de Azure Search.
* [Autocompletar](search-autocomplete-tutorial.md) proporciona una alternativa a la **sugerir** API para implementar el comportamiento de búsqueda como-al escribir. Autocompletar "finaliza" la palabra o frase que un usuario está escribiendo actualmente.
* [Modo de análisis de JsonLines](search-howto-index-json-blobs.md), parte de Azure Blob de indexación, crea un documento de búsqueda por entidad JSON que está separado por una línea nueva.

### <a name="new-preview-features-in-version-80-preview"></a>Nuevas características de vista previa en la versión 8.0-preview
Versión 8.0-preview del SDK de .NET de Azure Search tiene como destino la API versión 2017-11-11-Preview. Esta versión incluye las mismas características de versión 9, además de:

* [Las claves de cifrado administradas por el cliente](search-security-manage-encryption-keys.md) para el cifrado en reposo de lado del servicio es una nueva característica de vista previa. Además de la integradas cifrado en reposo administrada por Microsoft, puede aplicar una capa adicional de donde está el titular exclusivo de las claves de cifrado.

<a name="UpgradeSteps"></a>

## <a name="steps-to-upgrade"></a>Pasos para actualizar
En primer lugar, actualice la referencia de NuGet para `Microsoft.Azure.Search` mediante la Consola del Administrador de paquetes NuGet, o bien haga clic con el botón derecho en las referencias del proyecto y seleccione "Administrar paquetes NuGet" en Visual Studio.

Una vez que NuGet ha descargado los nuevos paquetes y sus dependencias, recompile el proyecto. Dependiendo de cómo se estructura el código, se podrá volver a compilar correctamente. Si lo consigue, ya estará listo para empezar.

Si se produce un error en la compilación, deberá corregir cada error de compilación. Consulte [importantes cambios en la versión 9](#ListOfChanges) para obtener más información sobre cómo resolver cada posible error de compilación.

Puede ver las advertencias de compilación adicionales relacionadas con propiedades o métodos obsoletos. Las advertencias incluyen instrucciones sobre qué utilizar en lugar de la característica en desuso. Por ejemplo, si la aplicación utiliza el `DataSourceType.DocumentDb` propiedad, recibirá una advertencia que dice "este miembro está en desuso. Use CosmosDb en su lugar".

Una vez que haya solucionado los errores o las advertencias de compilación, puede efectuar cambios en la aplicación para aprovechar las ventajas de la nueva funcionalidad, si lo desea. Nuevas características en el SDK se detallan en [cuáles son las novedades en la versión 9](#WhatsNew).

<a name="ListOfChanges"></a>

## <a name="breaking-changes-in-version-9"></a>Cambios importantes en la versión 9

Hay varios cambios importantes en la versión 9 que puede requerir cambios en el código además de volver a generar la aplicación.

> [!NOTE]
> La lista de cambios siguiente no es exhaustiva. Algunos cambios es probable que no dará como resultado errores de compilación, pero técnicamente son importantes ya que anulan la compatibilidad binaria con los ensamblados que dependen de las versiones anteriores de los ensamblados de SDK de .NET de Azure Search. Estos cambios no se muestran a continuación. Vuelva a generar la aplicación al actualizar a la versión 9 para evitar problemas de compatibilidad binaria.

### <a name="immutable-properties"></a>Propiedades inmutables

Las propiedades públicas de varias clases de modelo ahora son inmutables. Si necesita crear instancias personalizadas de estas clases para realizar pruebas, puede usar los nuevos constructores con parámetros:

  - `AutocompleteItem`
  - `DocumentSearchResult`
  - `DocumentSuggestResult`
  - `FacetResult`
  - `SearchResult`
  - `SuggestResult`

### <a name="changes-to-field"></a>Cambios realizados en el campo

La `Field` clase ha cambiado ahora que también pueden representar campos complejos.

La siguiente `bool` propiedades ahora admiten valores NULL:

  - `IsFilterable`
  - `IsFacetable`
  - `IsSearchable`
  - `IsSortable`
  - `IsRetrievable`
  - `IsKey`

Esto es porque estas propiedades deben ser ahora `null` en el caso de campos complejos. Si tiene código que lee estas propiedades, tiene que estar preparado para controlar `null`. Tenga en cuenta que todas las demás propiedades de `Field` siempre han sido y continúan siendo que acepta valores NULL, y algunas de ellas también se `null` en el caso de campos complejos--específicamente lo siguiente:

  - `Analyzer`
  - `SearchAnalyzer`
  - `IndexAnalyzer`
  - `SynonymMaps`

El constructor sin parámetros de `Field` se ha realizado `internal`. De ahora en adelante, cada `Field` requiere un tipo de datos y el nombre explícito en el momento de la construcción.

### <a name="simplified-batch-and-results-types"></a>Tipos de lote y los resultados simplificados

En la versión 7.0-preview y anterior, las distintas clases que encapsulan los grupos de documentos se han estructurado en jerarquías de clases paralelas:

  -  `DocumentSearchResult` y `DocumentSearchResult<T>` heredado de `DocumentSearchResultBase`
  -  `DocumentSuggestResult` y `DocumentSuggestResult<T>` heredado de `DocumentSuggestResultBase`
  -  `IndexAction` y `IndexAction<T>` heredado de `IndexActionBase`
  -  `IndexBatch` y `IndexBatch<T>` heredado de `IndexBatchBase`
  -  `SearchResult` y `SearchResult<T>` heredado de `SearchResultBase`
  -  `SuggestResult` y `SuggestResult<T>` heredado de `SuggestResultBase`

Los tipos derivados sin un parámetro de tipo genérico se diseñadas para usarse en escenarios de "tipo dinámico" y se supone el uso de la `Document` tipo.

A partir de la versión 8.0-preview, las clases base y clases derivadas no genérica todos se han quitado. Para escenarios de tipo dinámico, puede usar `IndexBatch<Document>`, `DocumentSearchResult<Document>`, y así sucesivamente.
 
### <a name="removed-extensibleenum"></a>ExtensibleEnum quitado

El `ExtensibleEnum` ha quitado la clase base. Todas las clases que derivan de él ahora son structs, como `AnalyzerName`, `DataType`, y `DataSourceType` por ejemplo. Sus `Create` métodos también se han quitado. Solo puede quitar las llamadas a `Create` puesto que estos tipos son implícitamente convertibles de cadenas. Si esto da como resultado errores del compilador, puede invocar explícitamente el operador de conversión a través de conversión para eliminar la ambigüedad de los tipos. Por ejemplo, puede cambiar el código similar al siguiente:

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

Las propiedades que mantienen los valores opcionales de estos tipos tienen ahora un tipo explícito que acepta valores NULL por lo que sigan estando opcional.

### <a name="removed-facetresults-and-hithighlights"></a>Quitado FacetResults y HitHighlights

El `FacetResults` y `HitHighlights` se han quitado las clases. Ahora se escriben los resultados de faceta como `IDictionary<string, IList<FacetResult>>` y posicionamiento destaca como `IDictionary<string, IList<string>>`. Una forma rápida para resolver errores de compilación introducidos por este cambio consiste en agregar `using` alias en la parte superior de cada archivo que usa los tipos quitados. Por ejemplo:

```csharp
using FacetResults = System.Collections.Generic.IDictionary<string, System.Collections.Generic.IList<Models.FacetResult>>;
using HitHighlights = System.Collections.Generic.IDictionary<string, System.Collections.Generic.IList<string>>;
```

### <a name="change-to-synonymmap"></a>Cambie a SynonymMap 

El constructor `SynonymMap` ya no tiene un parámetro `enum` para `SynonymMapFormat`. Esta enumeración solo tenía un valor y, por tanto, era redundante. Si como resultado se observan errores de compilación, basta con eliminar las referencias al parámetro `SynonymMapFormat`.

### <a name="miscellaneous-model-class-changes"></a>Cambios de la clase de varios modelos

El `AutocompleteMode` propiedad de `AutocompleteParameters` ya no es que acepta valores NULL. Si tiene código que asigna esta propiedad en `null`, simplemente puede quitarlo y automáticamente se inicializará la propiedad en el valor predeterminado.

El orden de los parámetros para el `IndexAction` constructor ha cambiado ahora que este constructor es generado automáticamente. En lugar de usar el constructor, se recomienda usar los métodos de generador `IndexAction.Upload`, `IndexAction.Merge`, y así sucesivamente.

### <a name="removed-preview-features"></a>Características quitadas de la versión preliminar

Si va a actualizar desde la versión 8.0-preview a la versión 9, tenga en cuenta que el cifrado con claves administradas por el cliente se ha quitado ya que esta característica está aún en versión preliminar. En concreto, el `EncryptionKey` propiedades de `Index` y `SynonymMap` se han quitado.

Si la aplicación tiene una dependencia fuerte sobre esta característica, no podrá actualizar a la versión 9 del SDK de .NET de Azure Search. Aún puede usar la versión 8.0-preview. Sin embargo, tenga en cuenta que **no se recomienda usar SDK de versión preliminar en aplicaciones de producción**. Las características de versión preliminar son solo para evaluación y pueden cambiar.

> [!NOTE]
> Si ha creado cifrados índices o sinónimo con versión 8.0-preview del SDK de mapas, todavía podrá usar y modificar sus definiciones con la versión 9 del SDK sin afectar negativamente a su estado de cifrado. 9 de la versión del SDK no enviará el `encryptionKey` propiedad a la API de REST y como resultado, la API de REST no cambiará el estado de cifrado del recurso. 

### <a name="behavioral-change-in-data-retrieval"></a>Cambio de comportamiento en la recuperación de datos

Si usa el "tipo dinámico" `Search`, `Suggest`, o `Get` API que devuelven instancias de tipo `Document`, tenga en cuenta que ahora deserializar las matrices JSON vacías para `object[]` en lugar de `string[]`.

## <a name="conclusion"></a>Conclusión
Si necesita más detalles sobre el uso del SDK de .NET para Azure Search, consulte la [guía de .NET](search-howto-dotnet-sdk.md).

Agradecemos sus comentarios sobre el SDK. Si tiene problemas, no dude en pedirnos ayuda en [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-search). Si encuentra un error, puede enviarlo al [repositorio de GitHub del SDK de .NET para Azure](https://github.com/Azure/azure-sdk-for-net/issues). Asegúrese de agregar "[Azure Search]" como prefijo en el título del problema.

Gracias por usar Azure Search.
