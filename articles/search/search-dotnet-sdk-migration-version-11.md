---
title: Actualización a la versión 11 del SDK de .NET
titleSuffix: Azure Cognitive Search
description: Migración del código a la versión 11 del SDK de .NET para Azure Cognitive Search a partir de versiones anteriores. Obtenga información sobre las novedades y los cambios de código necesarios.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 08/20/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 5a72f864d16ff89636b77709e439dd04134b64b7
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89002732"
---
# <a name="upgrade-to-azure-cognitive-search-net-sdk-version-11"></a>Actualización a la versión 11 del SDK de .NET para Azure Cognitive Search

Si usa la versión 10.0 o anterior del [SDK de .NET](/dotnet/api/overview/azure/search), este artículo lo ayudará a actualizar a la versión 11.

La versión 11 es una biblioteca cliente totalmente rediseñada que ha publicado el equipo de desarrollo de Azure SDK (las versiones anteriores fueron creadas por el equipo de desarrollo de Azure Cognitive Search). La biblioteca se ha rediseñado para que sea más coherente con otras bibliotecas cliente de Azure, tomando una dependencia de [Azure.Core](/dotnet/api/azure.core) y [System.Text.Json](/dotnet/api/system.text.json) e implementando enfoques conocidos para las tareas comunes.

Algunas de las principales diferencias que observará en la nueva versión son estas:

+ Un paquete y una biblioteca en lugar de varios
+ Un nuevo nombre de paquete: `Azure.Search.Documents` en lugar de `Microsoft.Azure.Search`
+ Tres clientes en lugar de dos: `SearchClient`, `SearchIndexClient` y `SearchIndexerClient`
+ Diferencias de nomenclatura en una serie de API y pequeñas diferencias estructurales que simplifican algunas tareas

> [!NOTE]
> Revise el [**registro de cambios**](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/CHANGELOG.md) para obtener una lista detallada de los cambios en el SDK de .NET versión 11.

## <a name="package-and-library-consolidation"></a>Consolidación de paquetes y bibliotecas

La versión 11 consolida varios paquetes y bibliotecas en uno solo. Después de la migración, tendrá menos bibliotecas para administrar.

+ [Paquete Azure.Search.Documents](https://www.nuget.org/packages/Azure.Search.Documents/)

+ [Referencia de API de la biblioteca cliente](/dotnet/api/overview/azure/search.documents-readme?view=azure-dotnet)

## <a name="client-differences"></a>Diferencias de cliente

En la siguiente tabla se relacionan las bibliotecas cliente de las dos versiones.

| Ámbito de operaciones | Microsoft.Azure.Search&nbsp;(v. 10) | Azure.Search.Documents&nbsp;(v. 11) |
|---------------------|------------------------------|------------------------------|
| Cliente usado para las consultas y para rellenar un índice. | [SearchIndexClient](/dotnet/api/azure.search.documents.indexes.searchindexclient) | [SearchClient](/dotnet/api/azure.search.documents.searchclient) |
| Cliente usado para índices, analizadores y mapas de sinónimos. | [SearchServiceClient](/dotnet/api/microsoft.azure.search.searchserviceclient) | [SearchIndexClient](/dotnet/api/azure.search.documents.indexes.searchindexclient) |
| Cliente usado para indizadores, orígenes de datos y conjuntos de aptitudes. | [SearchServiceClient](/dotnet/api/microsoft.azure.search.searchserviceclient) | [SearchIndexerClient (**nueva**)](/dotnet/api/azure.search.documents.indexes.searchindexerclient) |

> [!Important]
> `SearchIndexClient` está en ambas versiones, pero no admite las mismas cosas. En la versión 10, `SearchIndexClient` crea índices y otros objetos. En la versión 11, `SearchIndexClient` funciona con índices ya creados. Para evitar confusiones al actualizar el código, tenga en cuenta el orden en que se actualizan las referencias de cliente. Si sigue la secuencia de [pasos de actualización](#UpgradeSteps), debería poder mitigar cualquier problema de reemplazo de cadenas.

<a name="naming-differences"></a>

## <a name="naming-and-other-api-differences"></a>Nomenclatura y otras diferencias de API

Además de las diferencias de cliente (indicadas anteriormente y que, por tanto, se omitieron aquí), se ha cambiado el nombre de varias API y, en algunos casos, se han rediseñado. A continuación se resumen las diferencias en los nombres de clase. Esta lista no es exhaustiva, pero agrupa los cambios de API por tarea, lo que puede resultar útil para las revisiones en bloques de código específicos. Para ver una lista de actualizaciones de API, consulte el [registro de cambios](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/CHANGELOG.md) de `Azure.Search.Documents` en GitHub.

### <a name="authentication-and-encryption"></a>Autenticación y cifrado

| Versión 10 | Equivalente en la versión 11 |
|------------|-----------------------|
| [SearchCredentials](/dotnet/api/microsoft.azure.search.searchcredentials) | [AzureKeyCredential](/dotnet/api/azure.azurekeycredential) |
| `EncryptionKey` (estaba en el [SDK en versión preliminar](https://www.nuget.org/packages/Microsoft.Azure.Search/8.0.0-preview) como una característica disponible con carácter general) | [SearchResourceEncryptionKey](/dotnet/api/azure.search.documents.indexes.models.searchresourceencryptionkey) |

### <a name="indexes-analyzers-synonym-maps"></a>Índices, analizadores y mapas de sinónimos

| Versión 10 | Equivalente en la versión 11 |
|------------|-----------------------|
| [Index](/dotnet/api/microsoft.azure.documents.index) | [SearchIndex](/dotnet/api/azure.search.documents.indexes.models.searchindex) |
| [Campo](/dotnet/api/microsoft.azure.search.models.field) | [SearchField](/dotnet/api/azure.search.documents.indexes.models.searchfield) |
| [DataType](/dotnet/api/microsoft.azure.search.models.datatype) | [SearchFieldDataType](/dotnet/api/azure.search.documents.indexes.models.searchfielddatatype) |
| [ItemError](/dotnet/api/microsoft.azure.search.models.itemerror) | [SearchIndexerError](/dotnet/api/azure.search.documents.indexes.models.searchindexererror) |
| [Analyzer](/dotnet/api/microsoft.azure.search.models.analyzer) | [LexicalAnalyzer](/dotnet/api/azure.search.documents.indexes.models.lexicalanalyzer) (también, de `AnalyzerName` a `LexicalAnalyzerName`) |
| [AnalyzeRequest](/dotnet/api/microsoft.azure.search.models.analyzerequest) | [AnalyzeTextOptions](/dotnet/api/azure.search.documents.indexes.models.analyzetextoptions) |
| [StandardAnalyzer](/dotnet/api/microsoft.azure.search.models.standardanalyzer) | [LuceneStandardAnalyzer](//dotnet/api/azure.search.documents.indexes.models.lucenestandardanalyzer) |
| [StandardTokenizer](/dotnet/api/microsoft.azure.search.models.standardtokenizer) | [LuceneStandardTokenizer](/dotnet/api/azure.search.documents.indexes.models.lucenestandardtokenizer) (también, de `StandardTokenizerV2` a `LuceneStandardTokenizerV2`) |
| [TokenInfo](/dotnet/api/microsoft.azure.search.models.tokeninfo) | [AnalyzedTokenInfo](/dotnet/api/azure.search.documents.indexes.models.analyzedtokeninfo) |
| [Tokenizador](/dotnet/api/microsoft.azure.search.models.tokenizer) | [LexicalTokenizer](/dotnet/api/azure.search.documents.indexes.models.lexicaltokenizer) (también, de `TokenizerName` a `LexicalTokenizerName`) |
| [SynonymMap.Format](/dotnet/api/microsoft.azure.search.models.synonymmap.format) | Ninguno. Se eliminan las referencias a `Format`. |

Se han optimizado las definiciones de campo: [SearchableField](/dotnet/api/azure.search.documents.indexes.models.searchablefield), [SimpleField](/dotnet/api/azure.search.documents.indexes.models.simplefield) y [ComplexField](/dotnet/api/azure.search.documents.indexes.models.complexfield) son nuevas API para crear definiciones de campo.

### <a name="indexers-datasources-skillsets"></a>Indizadores, orígenes de datos y conjuntos de aptitudes

| Versión 10 | Equivalente en la versión 11 |
|------------|-----------------------|
| [Indizador](/dotnet/api/microsoft.azure.search.models.indexer) | [SearchIndexer](/dotnet/api/azure.search.documents.indexes.models.searchindexer) |
| [DataSource](/dotnet/api/microsoft.azure.search.models.datasource) | [SearchIndexerDataSourceConnection](//dotnet/api/azure.search.documents.indexes.models.searchindexerdatasourceconnection) |
| [Habilidad](/dotnet/api/microsoft.azure.search.models.skill) | [SearchIndexerSkill](/dotnet/api/azure.search.documents.indexes.models.searchindexerskill) |
| [Conjunto de aptitudes](/dotnet/api/microsoft.azure.search.models.skillset) | [SearchIndexerSkillset](/dotnet/api/azure.search.documents.indexes.models.searchindexerskill) |
| [DataSourceType](/dotnet/api/microsoft.azure.search.models.datasourcetype) | [SearchIndexerDataSourceType](/dotnet/api/azure.search.documents.indexes.models.searchindexerdatasourcetype) |

### <a name="data-import"></a>Importación de datos

| Versión 10 | Equivalente en la versión 11 |
|------------|-----------------------|
| [IndexAction](/dotnet/api/microsoft.azure.search.models.indexaction) | [IndexDocumentsAction](/dotnet/api/azure.search.documents.models.indexdocumentsaction) |
| [IndexBatch](/dotnet/api/microsoft.azure.search.models.indexbatch) | [IndexDocumentsBatch](/dotnet/api/azure.search.documents.models.indexdocumentsbatch) |

### <a name="query-definitions-and-results"></a>Definiciones de consulta y resultados

| Versión 10 | Equivalente en la versión 11 |
|------------|-----------------------|
| [DocumentSearchResult](/dotnet/api/microsoft.azure.search.models.documentsearchresult-1) | [SearchResult](/dotnet/api/azure.search.documents.models.searchresult-1) o [SearchResults](/dotnet/api/azure.search.documents.models.searchresults-1), en función de si el resultado está en un único documento o en varios. |
| [DocumentSuggestResult](/dotnet/api/microsoft.azure.search.models.documentsuggestresult-1) | [SuggestResults](/dotnet/api/azure.search.documents.models.suggestresults-1) |
| [SearchParameters](/dotnet/api/microsoft.azure.search.models.searchparameters) |  [SearchOptions](/dotnet/api/azure.search.documents.searchoptions)  |

<a name="WhatsNew"></a>

## <a name="whats-in-version-11"></a>Novedades de la versión 11

Cada versión de una biblioteca cliente de Azure Cognitive Search tiene como destino su versión correspondiente de la API REST. La API REST se considera fundamental para el servicio, con SDK individuales que encapsulan una versión de la API REST. Como desarrollador de .NET, puede ser útil revisar la [documentación de la API REST](/rest/api/searchservice/) para obtener más información sobre objetos u operaciones específicos.

La versión 11 tiene como destino el [servicio de búsqueda 2020-06-30](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/search/data-plane/Azure.Search/preview/2020-06-30/searchservice.json). Dado que la versión 11 también es una nueva biblioteca cliente creada desde cero, la mayor parte del trabajo de desarrollo se ha centrado en la equivalencia con la versión 10, y queda aún pendiente la compatibilidad con algunas características de la API REST.

La versión 11.0 es totalmente compatible con los siguientes objetos y operaciones:

+ Creación y administración de índices
+ Creación y administración de mapas de sinónimos
+ Todos los tipos de consulta y sintaxis (excepto los filtros geoespaciales)
+ Objetos de indizadores y operaciones para indexar orígenes de datos de Azure, incluidos los orígenes de datos y los conjuntos de habilidades

La versión 11.1 agrega lo siguiente:

+ [FieldBuilder](/dotnet/api/azure.search.documents.indexes.fieldbuilder) (se agregó en la versión 11.1)
+ [Propiedad Serializer](/dotnet/api/azure.search.documents.searchclientoptions.serializer) (se agregó en la versión 11.1) para admitir la serialización personalizada

### <a name="pending-features"></a>Características pendientes

Las siguientes características de la versión 10 aún no están disponibles en la 11. Si necesita estas características, posponga la migración hasta que se admitan.

+ Tipos geoespaciales
+ [Almacén de conocimiento](knowledge-store-concept-intro.md)

<a name="UpgradeSteps"></a>

## <a name="steps-to-upgrade"></a>Pasos para actualizar

Los pasos siguientes lo ayudarán a comenzar a realizar una migración de código recorriendo el primer conjunto de tareas necesarias, especialmente en lo que respecta a las referencias de cliente.

1. Instale el paquete [Azure.Search.Documents](https://www.nuget.org/packages/Azure.Search.Documents/) haciendo clic con el botón derecho en las referencias del proyecto y seleccione "Administrar paquetes NuGet..." en Visual Studio.

1. Reemplace las directivas "using" de Microsoft.Azure.Search por lo siguiente:

   ```csharp
   using Azure;
   using Azure.Search.Documents;
   using Azure.Search.Documents.Indexes;
   using Azure.Search.Documents.Indexes.Models;
   using Azure.Search.Documents.Models;
   ```

1. Si hay clases que requieren la serialización de JSON, reemplace `using Newtonsoft.Json` por `using System.Text.Json.Serialization`.

1. Revise el código de autenticación del cliente. En versiones anteriores, se usarían propiedades en el objeto de cliente para establecer la clave de API (por ejemplo, la propiedad [SearchServiceClient. Credentials](/dotnet/api/microsoft.azure.search.searchserviceclient.credentials)). En la versión actual, use la clase [AzureKeyCredential](/dotnet/api/azure.azurekeycredential) para pasar la clave como una credencial, de modo que, si es necesario, pueda actualizar la clave de API sin crear objetos de cliente.

   Las propiedades del cliente se han simplificado a solo `Endpoint`, `ServiceName` y `IndexName` (cuando es necesario). En el ejemplo siguiente se usa la clase [Uri](/dotnet/api/system.uri) del sistema para proporcionar el punto de conexión y la clase [Environment](/dotnet/api/system.environment) para leer el valor de la clave:

   ```csharp
   Uri endpoint = new Uri(Environment.GetEnvironmentVariable("SEARCH_ENDPOINT"));
   AzureKeyCredential credential = new AzureKeyCredential(
      Environment.GetEnvironmentVariable("SEARCH_API_KEY"));
   SearchIndexClient indexClient = new SearchIndexClient(endpoint, credential);
   ```

1. Agregue nuevas referencias de cliente para los objetos relacionados con indexadores. Si usa indexadores, orígenes de usuario o conjuntos de habilidades, cambie las referencias de cliente a [SearchIndexerClient](/dotnet/api/azure.search.documents.indexes.searchindexerclient). Este cliente es nuevo en la versión 11 y no tiene antecedente.

1. Actualice las referencias de cliente para las consultas y la importación de datos. Las instancias de [SearchIndexClient](/dotnet/api/microsoft.azure.search.searchindexclient) deben cambiarse a [SearchClient](/dotnet/api/azure.search.documents.searchclient). Para evitar confusión con los nombres, asegúrese de detectar todas las instancias antes de continuar con el siguiente paso.

1. Actualice las referencias de cliente en los objetos index, indexer, synonym map y analyzer. Las instancias de [SearchServiceClient](/dotnet/api/microsoft.azure.search.searchserviceclient) deben cambiarse a [SearchIndexClient](/dotnet/api/microsoft.azure.search.searchindexclient). 

1. En la medida de lo posible, actualice las clases, los métodos y las propiedades para usar las API de la nueva biblioteca. La sección [Nomenclatura y otras diferencias de API](#naming-differences) es un buen sitio para empezar, pero también puede revisar el [registro de cambios](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/CHANGELOG.md).

   Si tiene problemas para encontrar API equivalentes, se recomienda registrar un problema en [https://github.com/MicrosoftDocs/azure-docs/issues](https://github.com/MicrosoftDocs/azure-docs/issues) para que podamos mejorar la documentación o investigar el problema.

1. Recompilar la solución. Después de corregir las advertencias o los errores de compilación, puede realizar cambios en la aplicación para aprovechar las ventajas de la [nueva funcionalidad](#WhatsNew).

<a name="ListOfChanges"></a>

## <a name="breaking-changes-in-version-11"></a>Cambios importantes en la versión 11

Dado los cambios radicales producidos en las bibliotecas y las API, una actualización a la versión 11 no es algo trivial y constituye un cambio importante en el sentido de que el código ya no será compatible con la versión 10 y anteriores. Para ver todas las diferencias, vea el [registro de cambios](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/CHANGELOG.md) de `Azure.Search.Documents`.

En lo que respecta a las actualizaciones de la versión del servicio, donde los cambios de código de la versión 11 se relacionan con la funcionalidad existente (y no solo con la refactorización de las API), encontrará los siguientes cambios de comportamiento:

+ El [algoritmo de clasificación BM25](index-ranking-similarity.md) reemplaza el algoritmo de clasificación anterior por una tecnología más reciente. Los servicios nuevos usarán este algoritmo de manera automática. En el caso de los servicios existentes, debe establecer los parámetros para usar el algoritmo nuevo.

+ Los [resultados ordenados](search-query-odata-orderby.md) de los valores NULL han cambiado en esta versión y aparecen en primer lugar si la ordenación es `asc` y en el último si la ordenación es `desc`. Si ha escrito código para controlar cómo se ordenan los valores NULL, debe revisar y, posiblemente, quitar ese código si ya no es necesario.

## <a name="next-steps"></a>Pasos siguientes

+ [Paquete Azure.Search.Documents](https://www.nuget.org/packages/Azure.Search.Documents/)
+ [Ejemplos en GitHub](https://github.com/azure/azure-sdk-for-net/tree/Azure.Search.Documents_11.0.0/sdk/search/Azure.Search.Documents/samples)
+ [Referencia de la API de Azure.Search.Document](/dotnet/api/overview/azure/search.documents-readme?view=azure-dotnet)