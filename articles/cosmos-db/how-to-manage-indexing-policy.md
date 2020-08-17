---
title: Administración de directivas de indexación en Azure Cosmos DB
description: Obtenga información sobre cómo administrar directivas de indexación, incluir o excluir una propiedad de la indexación y cómo definir la indexación mediante diferentes SDK de Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.topic: how-to
ms.date: 08/04/2020
ms.author: tisande
ms.custom: devx-track-python, devx-track-javascript, devx-track-azurecli
ms.openlocfilehash: de9cdaf1dcc398c5db55de78643a19179f6bfdbc
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87876485"
---
# <a name="manage-indexing-policies-in-azure-cosmos-db"></a>Administración de directivas de indexación en Azure Cosmos DB

En Azure Cosmos DB, para indexar los datos se usan las [directivas de indexación](index-policy.md) que se definen para cada contenedor. La directiva de indexación predeterminada para los contenedores recién creados exige que se usen índices de intervalo para todas las cadenas o números. Puede invalidar esta directiva con su propia directiva de indexación personalizada.

## <a name="indexing-policy-examples"></a>Ejemplos de directiva de indexación

Estos son algunos ejemplos de directivas de indexación que se muestran en [su formato JSON](index-policy.md#include-exclude-paths), que es cómo se exponen en Azure Portal. Los mismos parámetros se pueden establecer a través de la CLI de Azure o cualquier SDK.

### <a name="opt-out-policy-to-selectively-exclude-some-property-paths"></a>Directiva de rechazo que excluye de forma selectiva algunas rutas de acceso de propiedades

```json
    {
        "indexingMode": "consistent",
        "includedPaths": [
            {
                "path": "/*"
            }
        ],
        "excludedPaths": [
            {
                "path": "/path/to/single/excluded/property/?"
            },
            {
                "path": "/path/to/root/of/multiple/excluded/properties/*"
            }
        ]
    }
```

Esta directiva de indexación es equivalente a la siguiente, que establece manualmente ```kind```, ```dataType``` y ```precision``` en sus valores predeterminados. Estas propiedades ya no son necesarias para establecerse explícitamente y puede omitirlas por completo en la directiva de indexación (como se muestra en el ejemplo anterior).

```json
    {
        "indexingMode": "consistent",
        "includedPaths": [
            {
                "path": "/*",
                "indexes": [
                    {
                        "kind": "Range",
                        "dataType": "Number",
                        "precision": -1
                    },
                    {
                        "kind": "Range",
                        "dataType": "String",
                        "precision": -1
                    }
                ]
            }
        ],
        "excludedPaths": [
            {
                "path": "/path/to/single/excluded/property/?"
            },
            {
                "path": "/path/to/root/of/multiple/excluded/properties/*"
            }
        ]
    }
```

### <a name="opt-in-policy-to-selectively-include-some-property-paths"></a>Directiva de participación que incluye de forma selectiva algunas rutas de acceso de propiedades

```json
    {
        "indexingMode": "consistent",
        "includedPaths": [
            {
                "path": "/path/to/included/property/?"
            },
            {
                "path": "/path/to/root/of/multiple/included/properties/*"
            }
        ],
        "excludedPaths": [
            {
                "path": "/*"
            }
        ]
    }
```

Esta directiva de indexación es equivalente a la siguiente, que establece manualmente ```kind```, ```dataType``` y ```precision``` en sus valores predeterminados. Estas propiedades ya no son necesarias para establecerse explícitamente y puede omitirlas por completo en la directiva de indexación (como se muestra en el ejemplo anterior).

```json
    {
        "indexingMode": "consistent",
        "includedPaths": [
            {
                "path": "/path/to/included/property/?",
                "indexes": [
                    {
                        "kind": "Range",
                        "dataType": "Number"
                    },
                    {
                        "kind": "Range",
                        "dataType": "String"
                    }
                ]
            },
            {
                "path": "/path/to/root/of/multiple/included/properties/*",
                "indexes": [
                    {
                        "kind": "Range",
                        "dataType": "Number"
                    },
                    {
                        "kind": "Range",
                        "dataType": "String"
                    }
                ]
            }
        ],
        "excludedPaths": [
            {
                "path": "/*"
            }
        ]
    }
```

> [!NOTE]
> Por lo general se recomienda usar una directiva de indexación de **rechazo** para permitir que Azure Cosmos DB indexe de forma proactiva todas las propiedades nuevas que se puedan agregar a un modelo.

### <a name="using-a-spatial-index-on-a-specific-property-path-only"></a>Uso de un índice espacial en una ruta de acceso de una propiedad concreta solo

```json
{
    "indexingMode": "consistent",
    "automatic": true,
    "includedPaths": [
        {
            "path": "/*"
        }
    ],
    "excludedPaths": [
        {
            "path": "/_etag/?"
        }
    ],
    "spatialIndexes": [
        {
            "path": "/path/to/geojson/property/?",
            "types": [
                "Point",
                "Polygon",
                "MultiPolygon",
                "LineString"
            ]
        }
    ]
}
```

## <a name="composite-indexing-policy-examples"></a>Ejemplos de la directiva de índice compuesto

Además de incluir o excluir rutas de acceso para las propiedades individuales, también puede especificar un índice compuesto. Si quiere hacer una consulta que tiene una cláusula `ORDER BY` para varias propiedades, necesita un [índice compuesto](index-policy.md#composite-indexes) en esas propiedades. Además, los índices compuestos tendrán una ventaja de rendimiento para las consultas que tienen un filtro y tienen una cláusula ORDER BY en distintas propiedades.

> [!NOTE]
> Las rutas de acceso compuestas tienen un carácter `/?` implícito, ya que solo se indexa el valor escalar de esa ruta de acceso. El carácter comodín `/*` no se admite en las rutas de acceso compuestas. No debe especificar `/?` ni `/*` en las rutas de acceso compuestas.

### <a name="composite-index-defined-for-name-asc-age-desc"></a>Índice compuesto definido para (name asc, age desc):

```json
    {  
        "automatic":true,
        "indexingMode":"Consistent",
        "includedPaths":[  
            {  
                "path":"/*"
            }
        ],
        "excludedPaths":[],
        "compositeIndexes":[  
            [  
                {  
                    "path":"/name",
                    "order":"ascending"
                },
                {  
                    "path":"/age",
                    "order":"descending"
                }
            ]
        ]
    }
```

El índice compuesto anterior sobre el nombre y la antigüedad es necesarios para las consultas #1 y #2:

Consulta 1:

```sql
    SELECT *
    FROM c
    ORDER BY c.name ASC, c.age DESC
```

Consulta 2:

```sql
    SELECT *
    FROM c
    ORDER BY c.name DESC, c.age ASC
```

Este índice compuesto beneficiará las consultas #3 y #4 y optimizará los filtros:

Consulta #3:

```sql
SELECT *
FROM c
WHERE c.name = "Tim"
ORDER BY c.name DESC, c.age ASC
```

Consulta #4:

```sql
SELECT *
FROM c
WHERE c.name = "Tim" AND c.age > 18
```

### <a name="composite-index-defined-for-name-asc-age-asc-and-name-asc-age-desc"></a>Índice compuesto definido para (name ASC, age ASC) y (name ASC, age DESC):

Puede definir varios índices compuestos distintos dentro de la misma directiva de índice.

```json
    {  
        "automatic":true,
        "indexingMode":"Consistent",
        "includedPaths":[  
            {  
                "path":"/*"
            }
        ],
        "excludedPaths":[],
        "compositeIndexes":[  
            [  
                {  
                    "path":"/name",
                    "order":"ascending"
                },
                {  
                    "path":"/age",
                    "order":"ascending"
                }
            ],
            [  
                {  
                    "path":"/name",
                    "order":"ascending"
                },
                {  
                    "path":"/age",
                    "order":"descending"
                }
            ]
        ]
    }
```

### <a name="composite-index-defined-for-name-asc-age-asc"></a>Índice compuesto definido para (name ASC, age ASC):

Si quiere, puede especificar el orden. Si no lo especifica, el orden es ascendente.

```json
{  
        "automatic":true,
        "indexingMode":"Consistent",
        "includedPaths":[  
            {  
                "path":"/*"
            }
        ],
        "excludedPaths":[],
        "compositeIndexes":[  
            [  
                {  
                    "path":"/name",
                },
                {  
                    "path":"/age",
                }
            ]
        ]
}
```

### <a name="excluding-all-property-paths-but-keeping-indexing-active"></a>Exclusión de las rutas de acceso de todas las propiedades, pero manteniendo la indexación activa

Esta directiva se puede usar en las situaciones en que la [característica Time-to-Live (TTL)](time-to-live.md) está activa, pero no se requiere un índice secundario (para usar Azure Cosmos DB como almacén de clave-valor puro).

```json
    {
        "indexingMode": "consistent",
        "includedPaths": [],
        "excludedPaths": [{
            "path": "/*"
        }]
    }
```

### <a name="no-indexing"></a>Sin indexación

Esta directiva desactivará la indexación. Si `indexingMode` está establecido en `none`, no puede establecer un TTL en el contenedor.

```json
    {
        "indexingMode": "none"
    }
```

## <a name="updating-indexing-policy"></a>Actualización de la directiva de indexación

En Azure Cosmos DB, la directiva de indexación se puede actualizar mediante cualquiera de los métodos siguientes:

- desde Azure Portal
- mediante la CLI de Azure
- mediante PowerShell
- mediante uno de los SDK

Las [actualizaciones de las directivas de indexación](index-policy.md#modifying-the-indexing-policy) desencadenan transformaciones de índices. También se puede realizar un seguimiento del progreso de esta transformación desde los SDK.

> [!NOTE]
> Al actualizar la directiva de indexación, las escrituras en Azure Cosmos DB no se interrumpirán. Obtenga más información sobre las [transformaciones de indexación](indexing-policy.md#modifying-the-indexing-policy).

## <a name="use-the-azure-portal"></a>Uso de Azure Portal

Los contenedores de Azure Cosmos almacenan su directiva de indexación en forma de documento JSON que Azure Portal permite editar directamente.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Cree una cuenta de Azure Cosmos DB o seleccione una ya existente.

1. Abra el panel **Data Explorer** y seleccione el contenedor en el que quiere trabajar.

1. Haga clic en **Scale & Settings** (Escala y configuración).

1. Modifique el documento JSON de la directiva de indexación (vea los ejemplos [a continuación](#indexing-policy-examples))

1. Cuando haya terminado, haga clic en **Guardar**.

:::image type="content" source="./media/how-to-manage-indexing-policy/indexing-policy-portal.png" alt-text="Administración de la indexación mediante Azure Portal":::

## <a name="use-the-azure-cli"></a>Uso de la CLI de Azure

Para crear un contenedor con una directiva de indexación personalizada, consulte [Creación de un contenedor con una directiva de indexación personalizada mediante la CLI](manage-with-cli.md#create-a-container-with-a-custom-index-policy).

## <a name="use-powershell"></a>Uso de PowerShell

Para crear un contenedor con una directiva de indexación personalizada, consulte [Creación de un contenedor con una directiva de indexación personalizada mediante PowerShell](manage-with-powershell.md#create-container-custom-index).

## <a name="use-the-net-sdk"></a><a id="dotnet-sdk"></a> Uso del SDK de .NET

# <a name="net-sdk-v2"></a>[SDK de .NET V2](#tab/dotnetv2)

El objeto `DocumentCollection` del [SDK de .NET v2](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/) expone una propiedad `IndexingPolicy` que permite cambiar `IndexingMode` y agregar o quitar `IncludedPaths` y `ExcludedPaths`.

```csharp
// Retrieve the container's details
ResourceResponse<DocumentCollection> containerResponse = await client.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri("database", "container"));
// Set the indexing mode to consistent
containerResponse.Resource.IndexingPolicy.IndexingMode = IndexingMode.Consistent;
// Add an included path
containerResponse.Resource.IndexingPolicy.IncludedPaths.Add(new IncludedPath { Path = "/*" });
// Add an excluded path
containerResponse.Resource.IndexingPolicy.ExcludedPaths.Add(new ExcludedPath { Path = "/name/*" });
// Add a spatial index
containerResponse.Resource.IndexingPolicy.SpatialIndexes.Add(new SpatialSpec() { Path = "/locations/*", SpatialTypes = new Collection<SpatialType>() { SpatialType.Point } } );
// Add a composite index
containerResponse.Resource.IndexingPolicy.CompositeIndexes.Add(new Collection<CompositePath> {new CompositePath() { Path = "/name", Order = CompositePathSortOrder.Ascending }, new CompositePath() { Path = "/age", Order = CompositePathSortOrder.Descending }});
// Update container with changes
await client.ReplaceDocumentCollectionAsync(containerResponse.Resource);
```

Para realizar un seguimiento del progreso de transformación del índice, utilice un objeto `RequestOptions` que establezca la propiedad `PopulateQuotaInfo` en `true`.

```csharp
// retrieve the container's details
ResourceResponse<DocumentCollection> container = await client.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri("database", "container"), new RequestOptions { PopulateQuotaInfo = true });
// retrieve the index transformation progress from the result
long indexTransformationProgress = container.IndexTransformationProgress;
```

# <a name="net-sdk-v3"></a>[SDK de .NET V3](#tab/dotnetv3)

El objeto `ContainerProperties` del [SDK de .NET v3](https://www.nuget.org/packages/Microsoft.Azure.Cosmos/) (consulte [este inicio rápido](create-sql-api-dotnet.md) para conocer su uso) expone una propiedad `IndexingPolicy` que permite cambiar `IndexingMode` y agregar o quitar `IncludedPaths` y `ExcludedPaths`.

```csharp
// Retrieve the container's details
ContainerResponse containerResponse = await client.GetContainer("database", "container").ReadContainerAsync();
// Set the indexing mode to consistent
containerResponse.Resource.IndexingPolicy.IndexingMode = IndexingMode.Consistent;
// Add an included path
containerResponse.Resource.IndexingPolicy.IncludedPaths.Add(new IncludedPath { Path = "/*" });
// Add an excluded path
containerResponse.Resource.IndexingPolicy.ExcludedPaths.Add(new ExcludedPath { Path = "/name/*" });
// Add a spatial index
SpatialPath spatialPath = new SpatialPath
{
    Path = "/locations/*"
};
spatialPath.SpatialTypes.Add(SpatialType.Point);
containerResponse.Resource.IndexingPolicy.SpatialIndexes.Add(spatialPath);
// Add a composite index
containerResponse.Resource.IndexingPolicy.CompositeIndexes.Add(new Collection<CompositePath> { new CompositePath() { Path = "/name", Order = CompositePathSortOrder.Ascending }, new CompositePath() { Path = "/age", Order = CompositePathSortOrder.Descending } });
// Update container with changes
await client.GetContainer("database", "container").ReplaceContainerAsync(containerResponse.Resource);
```

Para realizar un seguimiento del progreso de transformación, utilice un objeto `RequestOptions` que establezca la propiedad `PopulateQuotaInfo` en `true` y después recupere el valor del encabezado de respuesta `x-ms-documentdb-collection-index-transformation-progress`.

```csharp
// retrieve the container's details
ContainerResponse containerResponse = await client.GetContainer("database", "container").ReadContainerAsync(new ContainerRequestOptions { PopulateQuotaInfo = true });
// retrieve the index transformation progress from the result
long indexTransformationProgress = long.Parse(containerResponse.Headers["x-ms-documentdb-collection-index-transformation-progress"]);
```

Al definir una directiva de indexación personalizada durante la creación de un nuevo contenedor, la API fluida del SDK V3 le permite escribir esta definición de una manera concisa y eficaz:

```csharp
await client.GetDatabase("database").DefineContainer(name: "container", partitionKeyPath: "/myPartitionKey")
    .WithIndexingPolicy()
        .WithIncludedPaths()
            .Path("/*")
        .Attach()
        .WithExcludedPaths()
            .Path("/name/*")
        .Attach()
        .WithSpatialIndex()
            .Path("/locations/*", SpatialType.Point)
        .Attach()
        .WithCompositeIndex()
            .Path("/name", CompositePathSortOrder.Ascending)
            .Path("/age", CompositePathSortOrder.Descending)
        .Attach()
    .Attach()
    .CreateIfNotExistsAsync();
```
---

## <a name="use-the-java-sdk"></a>Uso del SDK de Java

El objeto `DocumentCollection` del [SDK de Java](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb) (para aprender a usarlo, vea [este inicio rápido](create-sql-api-java.md)) expone los métodos `getIndexingPolicy()` y `setIndexingPolicy()`. El objeto `IndexingPolicy` que manipulan permite cambiar el modo de indexación y agregar o quitar las rutas de acceso incluidas y excluidas.

```java
// Retrieve the container's details
Observable<ResourceResponse<DocumentCollection>> containerResponse = client.readCollection(String.format("/dbs/%s/colls/%s", "database", "container"), null);
containerResponse.subscribe(result -> {
DocumentCollection container = result.getResource();
IndexingPolicy indexingPolicy = container.getIndexingPolicy();

// Set the indexing mode to consistent
indexingPolicy.setIndexingMode(IndexingMode.Consistent);

// Add an included path

Collection<IncludedPath> includedPaths = new ArrayList<>();
IncludedPath includedPath = new IncludedPath();
includedPath.setPath("/*");
includedPaths.add(includedPath);
indexingPolicy.setIncludedPaths(includedPaths);

// Add an excluded path

Collection<ExcludedPath> excludedPaths = new ArrayList<>();
ExcludedPath excludedPath = new ExcludedPath();
excludedPath.setPath("/name/*");
excludedPaths.add(excludedPath);
indexingPolicy.setExcludedPaths(excludedPaths);

// Add a spatial index

Collection<SpatialSpec> spatialIndexes = new ArrayList<SpatialSpec>();
Collection<SpatialType> collectionOfSpatialTypes = new ArrayList<SpatialType>();

SpatialSpec spec = new SpatialSpec();
spec.setPath("/locations/*");
collectionOfSpatialTypes.add(SpatialType.Point);
spec.setSpatialTypes(collectionOfSpatialTypes);
spatialIndexes.add(spec);

indexingPolicy.setSpatialIndexes(spatialIndexes);

// Add a composite index

Collection<ArrayList<CompositePath>> compositeIndexes = new ArrayList<>();
ArrayList<CompositePath> compositePaths = new ArrayList<>();

CompositePath nameCompositePath = new CompositePath();
nameCompositePath.setPath("/name");
nameCompositePath.setOrder(CompositePathSortOrder.Ascending);

CompositePath ageCompositePath = new CompositePath();
ageCompositePath.setPath("/age");
ageCompositePath.setOrder(CompositePathSortOrder.Descending);

compositePaths.add(ageCompositePath);
compositePaths.add(nameCompositePath);

compositeIndexes.add(compositePaths);
indexingPolicy.setCompositeIndexes(compositeIndexes);

// Update the container with changes

 client.replaceCollection(container, null);
});
```

Para realizar un seguimiento del progreso de transformación del índice en un contenedor, utilice un objeto `RequestOptions` que solicite la información de cuota que se debe rellenar y, después, recupere el valor del encabezado de respuesta `x-ms-documentdb-collection-index-transformation-progress`.

```java
// set the RequestOptions object
RequestOptions requestOptions = new RequestOptions();
requestOptions.setPopulateQuotaInfo(true);
// retrieve the container's details
Observable<ResourceResponse<DocumentCollection>> containerResponse = client.readCollection(String.format("/dbs/%s/colls/%s", "database", "container"), requestOptions);
containerResponse.subscribe(result -> {
    // retrieve the index transformation progress from the response headers
    String indexTransformationProgress = result.getResponseHeaders().get("x-ms-documentdb-collection-index-transformation-progress");
});
```

## <a name="use-the-nodejs-sdk"></a>Uso del SDK de Node.js

La interfaz `ContainerDefinition` del [SDK de Node.js](https://www.npmjs.com/package/@azure/cosmos) (consulte [este inicio rápido](create-sql-api-nodejs.md) para conocer su uso) expone una propiedad `indexingPolicy` que permite cambiar `indexingMode` y agregar o eliminar `includedPaths` y `excludedPaths`.

Recuperar los detalles del contenedor

```javascript
const containerResponse = await client.database('database').container('container').read();
```

Establecer el modo de indexación en coherente

```javascript
containerResponse.body.indexingPolicy.indexingMode = "consistent";
```

Agregar una ruta de acceso incluida, incluido un índice espacial

```javascript
containerResponse.body.indexingPolicy.includedPaths.push({
    includedPaths: [
      {
        path: "/age/*",
        indexes: [
          {
            kind: cosmos.DocumentBase.IndexKind.Range,
            dataType: cosmos.DocumentBase.DataType.String
          },
          {
            kind: cosmos.DocumentBase.IndexKind.Range,
            dataType: cosmos.DocumentBase.DataType.Number
          }
        ]
      },
      {
        path: "/locations/*",
        indexes: [
          {
            kind: cosmos.DocumentBase.IndexKind.Spatial,
            dataType: cosmos.DocumentBase.DataType.Point
          }
        ]
      }
    ]
  });
```

Agregar una ruta de acceso excluida

```javascript
containerResponse.body.indexingPolicy.excludedPaths.push({ path: '/name/*' });
```

Actualizar el contenedor con cambios

```javascript
const replaceResponse = await client.database('database').container('container').replace(containerResponse.body);
```

Para realizar un seguimiento del progreso de transformación del índice en un contenedor, utilice un objeto `RequestOptions` que establezca la propiedad `populateQuotaInfo` en `true` y después recupere el valor del encabezado de respuesta `x-ms-documentdb-collection-index-transformation-progress`.

```javascript
// retrieve the container's details
const containerResponse = await client.database('database').container('container').read({
    populateQuotaInfo: true
});
// retrieve the index transformation progress from the response headers
const indexTransformationProgress = replaceResponse.headers['x-ms-documentdb-collection-index-transformation-progress'];
```

## <a name="use-the-python-sdk"></a>Uso del SDK de Python

# <a name="python-sdk-v3"></a>[SDK de Python V3](#tab/pythonv3)

Cuando se usa el [SDK de Python V3](https://pypi.org/project/azure-cosmos/) (consulte [este inicio rápido](create-sql-api-python.md) para saber cómo se usa), la configuración del contenedor se administra como un diccionario. Desde este diccionario es posible acceder a la directiva de indexación y a todos sus atributos.

Recuperar los detalles del contenedor

```python
containerPath = 'dbs/database/colls/collection'
container = client.ReadContainer(containerPath)
```

Establecer el modo de indexación en coherente

```python
container['indexingPolicy']['indexingMode'] = 'consistent'
```

Definir una directiva de indexación con una ruta de acceso incluida y un índice espacial

```python
container["indexingPolicy"] = {

    "indexingMode":"consistent",
    "spatialIndexes":[
                {"path":"/location/*","types":["Point"]}
             ],
    "includedPaths":[{"path":"/age/*","indexes":[]}],
    "excludedPaths":[{"path":"/*"}]
}
```

Definir una directiva de indexación con una ruta de acceso excluida

```python
container["indexingPolicy"] = {
    "indexingMode":"consistent",
    "includedPaths":[{"path":"/*","indexes":[]}],
    "excludedPaths":[{"path":"/name/*"}]
}
```

Agregar un índice compuesto

```python
container['indexingPolicy']['compositeIndexes'] = [
                [
                    {
                        "path": "/name",
                        "order": "ascending"
                    },
                    {
                        "path": "/age",
                        "order": "descending"
                    }
                ]
                ]
```

Actualizar el contenedor con cambios

```python
response = client.ReplaceContainer(containerPath, container)
```

# <a name="python-sdk-v4"></a>[SDK de Python V4](#tab/pythonv4)

Cuando se usa el [SDK de Python V4](https://pypi.org/project/azure-cosmos/), la configuración del contenedor se administra como un diccionario. Desde este diccionario es posible acceder a la directiva de indexación y a todos sus atributos.

Recuperar los detalles del contenedor

```python
database_client = cosmos_client.get_database_client('database')
container_client = database_client.get_container_client('container')
container = container_client.read()
```

Establecer el modo de indexación en coherente

```python
indexingPolicy = {
    'indexingMode': 'consistent'
}
```

Definir una directiva de indexación con una ruta de acceso incluida y un índice espacial

```python
indexingPolicy = {
    "indexingMode":"consistent",
    "spatialIndexes":[
        {"path":"/location/*","types":["Point"]}
    ],
    "includedPaths":[{"path":"/age/*","indexes":[]}],
    "excludedPaths":[{"path":"/*"}]
}
```

Definir una directiva de indexación con una ruta de acceso excluida

```python
indexingPolicy = {
    "indexingMode":"consistent",
    "includedPaths":[{"path":"/*","indexes":[]}],
    "excludedPaths":[{"path":"/name/*"}]
}
```

Agregar un índice compuesto

```python
indexingPolicy['compositeIndexes'] = [
    [
        {
            "path": "/name",
            "order": "ascending"
        },
        {
            "path": "/age",
            "order": "descending"
        }
    ]
]
```

Actualizar el contenedor con cambios

```python
response = database_client.replace_container(container_client, container['partitionKey'], indexingPolicy)
```
---

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información acerca de la indexación en los siguientes artículos:

- [Introducción a la indexación](index-overview.md)
- [Directiva de indexación](index-policy.md)
