---
title: Actualización a la versión 3 del SDK de .NET para Azure Search
titleSuffix: Azure Cognitive Search
description: Migre el código a la versión 3 del SDK de Azure Search para .NET a partir de versiones anteriores. Obtenga información sobre las novedades y los cambios de código necesarios.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 11/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 0e9f4602582a8c24f74d6603b732e7b91d97fa8d
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89002698"
---
# <a name="upgrade-to-azure-search-net-sdk-version-3"></a>Actualización a la versión 3 del SDK de .NET para Azure Search

<!--- DETAILS in the word doc
cosmosdb
NER v1 skill 
Indexer execution result errors no longer have status
the data source API will no longer return in the response of any REST operation, the connection string specified by the user.
--->

Si usa la versión preliminar 2.0 o posterior del [SDK de .NET para Azure Search](/dotnet/api/overview/azure/search), este artículo le ayudará a actualizar la aplicación para que use la versión 3.

Para obtener un tutorial más general del SDK que incluya ejemplos, vea [Cómo usar Azure Search desde una aplicación .NET](search-howto-dotnet-sdk.md).

La versión 3 del SDK de .NET para Azure Search contiene algunos cambios de versiones anteriores. La mayoría son de menor importancia, por lo que cambiar el código solo precisará del mínimo esfuerzo. Vea [Pasos para actualizar](#UpgradeSteps) , a fin de obtener instrucciones sobre cómo cambiar el código para usar la nueva versión del SDK.

> [!NOTE]
> Si usa la versión 1.0.2-preview o anterior, debe actualizar primero a la versión 1.1 y luego a la versión 3. Consulte [Actualización a la versión 1.1 del SDK de .NET para Azure Search](search-dotnet-sdk-migration-version-1.md) para instrucciones.
>
> La instancia del servicio Azure Search es compatible con varias versiones de API de REST, incluida la más reciente. Puede seguir usando una versión aunque no sea la más reciente, pero es recomendable que migre el código para usar la versión más actualizada. Cuando se usa la API de REST, debe especificar la versión de API en cada solicitud realizada a través del parámetro api-version. Al usar el SDK de .NET, la versión del SDK que usa determina la versión correspondiente de la API de REST. Si usa un SDK anterior, puede seguir ejecutando ese código sin realizar ningún cambio, incluso si el servicio se ha actualizado para admitir una versión más reciente de la API.

<a name="WhatsNew"></a>

## <a name="whats-new-in-version-3"></a>Novedades de la versión 3
La versión 3 del SDK de .NET para Azure Search tiene como destino la versión más reciente con disponibilidad general de la API de REST de Azure Search, en concreto la del 01-09-2016. Esto hace posible el uso de muchas características de Azure Search desde una aplicación. NET, incluidas las siguientes:

* [Analizadores personalizados](index-add-custom-analyzers.md)
* Compatibilidad del indexador de [Azure Blob Storage](search-howto-indexing-azure-blob-storage.md) y [Azure Table Storage](search-howto-indexing-azure-tables.md)
* Personalización de indizador mediante [asignaciones de campos](search-indexer-field-mappings.md)
* Compatibilidad con ETags para permitir actualizaciones seguras simultáneas de definiciones de índice, indizadores y orígenes de datos
* Compatibilidad con la generación de definiciones de campos de índice de forma declarativa mediante la decoración de la clase model y el uso de la nueva clase `FieldBuilder`.
* Compatibilidad con .NET Core y .NET Portable Profile 111

<a name="UpgradeSteps"></a>

## <a name="steps-to-upgrade"></a>Pasos para actualizar
En primer lugar, actualice la referencia de NuGet para `Microsoft.Azure.Search` mediante la Consola del Administrador de paquetes NuGet, o bien haga clic con el botón derecho en las referencias del proyecto y seleccione "Administrar paquetes NuGet" en Visual Studio.

Una vez que NuGet ha descargado los nuevos paquetes y sus dependencias, recompile el proyecto. Dependiendo de cómo se estructura el código, se podrá volver a compilar correctamente. Si lo consigue, ya estará listo para empezar.

Si se produce un error en la compilación, verá un mensaje similar al siguiente:

```output
Program.cs(31,45,31,86): error CS0266: Cannot implicitly convert type 'Microsoft.Azure.Search.ISearchIndexClient' to 'Microsoft.Azure.Search.SearchIndexClient'. An explicit conversion exists (are you missing a cast?)
```

El paso siguiente consiste en corregir los errores de compilación. Consulte [Cambios importantes en la versión 3](#ListOfChanges) para más información sobre las causas del error y cómo corregirlo.

Puede ver las advertencias de compilación adicionales relacionadas con propiedades o métodos obsoletos. Las advertencias incluyen instrucciones sobre qué utilizar en lugar de la característica en desuso. Por ejemplo, si la aplicación utiliza la propiedad `IndexingParameters.Base64EncodeKeys`, recibirá una advertencia que indica lo siguiente: `"This property is obsolete. Please create a field mapping using 'FieldMapping.Base64Encode' instead."`

Una vez que haya solucionado los errores de compilación, puede realizar cambios a la aplicación para aprovechar las ventajas de la nueva funcionalidad, si lo desea. Las nuevas características en el SDK se detallan en [Novedades de la versión 3](#WhatsNew).

<a name="ListOfChanges"></a>

## <a name="breaking-changes-in-version-3"></a>Cambios importantes en la versión 3
Hay solo algunos cambios importantes en la versión 3 que pueden requerir cambios de código además de la recompilación de la aplicación.

### <a name="indexesgetclient-return-type"></a>Tipo de valor devuelto de Indexes.GetClient
El método `Indexes.GetClient` tiene un nuevo tipo de valor devuelto. Anteriormente devolvía `SearchIndexClient`, pero se ha cambiado a `ISearchIndexClient` en la versión 2.0-preview, y ese cambio se traslada a la versión 3. Sirve para admitir clientes que deseen simular el método `GetClient` para las pruebas unitarias devolviendo una implementación ficticia de `ISearchIndexClient`.

#### <a name="example"></a>Ejemplo
Si el código tiene el siguiente aspecto:

```csharp
SearchIndexClient indexClient = serviceClient.Indexes.GetClient("hotels");
```

Puede cambiarlo por este para corregir cualquier error de compilación:

```csharp
ISearchIndexClient indexClient = serviceClient.Indexes.GetClient("hotels");
```

### <a name="analyzername-datatype-and-others-are-no-longer-implicitly-convertible-to-strings"></a>AnalyzerName, DataType y otros ya no se pueden convertir de manera implícita en cadenas
Hay muchos tipos en el SDK de .NET para Azure Search que derivan de `ExtensibleEnum`. Anteriormente, todos estos tipos eran convertibles de manera implícita en el tipo `string`. Sin embargo, se detectó un error en la implementación de `Object.Equals` para estas clases y para corregir el error fue necesario deshabilitar esta conversión implícita. La conversión explícita a `string` todavía se permite.

#### <a name="example"></a>Ejemplo
Si el código tiene el siguiente aspecto:

```csharp
var customTokenizerName = TokenizerName.Create("my_tokenizer"); 
var customTokenFilterName = TokenFilterName.Create("my_tokenfilter"); 
var customCharFilterName = CharFilterName.Create("my_charfilter"); 
 
var index = new Index();
index.Analyzers = new Analyzer[] 
{ 
    new CustomAnalyzer( 
        "my_analyzer",  
        customTokenizerName,  
        new[] { customTokenFilterName },  
        new[] { customCharFilterName }), 
}; 
```

Puede cambiarlo por este para corregir cualquier error de compilación:

```csharp
const string CustomTokenizerName = "my_tokenizer"; 
const string CustomTokenFilterName = "my_tokenfilter"; 
const string CustomCharFilterName = "my_charfilter"; 
 
var index = new Index();
index.Analyzers = new Analyzer[] 
{ 
    new CustomAnalyzer( 
        "my_analyzer",  
        CustomTokenizerName,  
        new TokenFilterName[] { CustomTokenFilterName },  
        new CharFilterName[] { CustomCharFilterName })
}; 
```

### <a name="removed-obsolete-members"></a>Miembros obsoletos quitados

Es posible que vea errores de compilación relacionados con métodos o propiedades que se marcaron como obsoletos en la versión 2.0-preview y que posteriormente se eliminaron en la versión 3. Si se encuentra con tales errores, aquí le decimos cómo resolverlos:

- Si usaba este constructor: `ScoringParameter(string name, string value)`, utilice en su lugar este: `ScoringParameter(string name, IEnumerable<string> values)`
- Si usaba la propiedad `ScoringParameter.Value`, utilice en su lugar la propiedad `ScoringParameter.Values` o el método `ToString`.
- Si usaba la propiedad `SearchRequestOptions.RequestId`, utilice en su lugar la propiedad `ClientRequestId`.

### <a name="removed-preview-features"></a>Características quitadas de la versión preliminar

Si va a actualizar de la versión 2.0-preview a la versión 3, debe saber que se ha eliminado la compatibilidad con el análisis de JSON y CSV en los indexadores de blobs dado que estás características se encuentran aún en versión preliminar. En concreto, se han quitado los siguientes métodos de la clase `IndexingParametersExtensions`:

- `ParseJson`
- `ParseJsonArrays`
- `ParseDelimitedTextFiles`

Si la aplicación tiene una dependencia fuerte de estas características, no podrá actualizar a la versión 3 del SDK de Azure Search para .NET. Podrá seguir usando la versión 2.0-preview. Sin embargo, tenga en cuenta que **no se recomienda usar SDK de versión preliminar en aplicaciones de producción**. Las características de versión preliminar son solo para evaluación y pueden cambiar.

## <a name="conclusion"></a>Conclusión
Si necesita más detalles sobre el uso del SDK de .NET para Azure Search, consulte la [guía de .NET](search-howto-dotnet-sdk.md).

Agradecemos sus comentarios sobre el SDK. Si tiene algún problema, no dude en pedirnos ayuda en [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-search). Si encuentra un error, puede enviarlo al [repositorio de GitHub del SDK de .NET para Azure](https://github.com/Azure/azure-sdk-for-net/issues). Asegúrese de agregar "[Azure Search]" como prefijo en el título del problema.

Gracias por usar Azure Search.