---
title: Trabajo con datos geoespaciales en la cuenta de SQL API de Azure Cosmos DB
description: Aprenda a crear, indexar y consultar objetos espaciales con Azure Cosmos DB y la API de SQL.
author: SnehaGunda
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 11/01/2017
ms.author: sngun
ms.openlocfilehash: 26eb43060a0a2b564a7274bb1579151e66b2bfee
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65604205"
---
# <a name="use-geospatial-and-geojson-location-data-with-azure-cosmos-db-sql-api-account"></a>Uso de datos geoespaciales de ubicación y de GeoJSON con la cuenta de SQL API de Azure Cosmos DB

Este artículo es una introducción a la funcionalidad geoespacial de Azure Cosmos DB. Actualmente, solo las cuentas de SQL API de Cosmos DB admiten el almacenamiento y acceso a datos geoespaciales. Después de leer este artículo, podrá responder a las siguientes preguntas:

* ¿Cómo almaceno los datos espaciales en Azure Cosmos DB?
* ¿Cómo puedo consultar los datos geoespaciales en Azure Cosmos DB en SQL y LINQ?
* ¿Qué tengo que hacer para habilitar y deshabilitar la indexación en Azure Cosmos DB?

En este artículo se muestra cómo trabajar con datos espaciales con la API de SQL. Consulte este [proyecto de GitHub](https://github.com/Azure/azure-documentdb-dotnet/blob/master/samples/code-samples/Geospatial/Program.cs) para obtener ejemplos de código.

## <a name="introduction-to-spatial-data"></a>Introducción a los datos espaciales
Los datos espaciales describen la posición y la forma de los objetos en el espacio. En la mayoría de las aplicaciones, estos se refieren a objetos situados en la tierra y datos geoespaciales. Los datos espaciales pueden usarse para representar la ubicación de una persona, un lugar de interés o el límite de una ciudad o un lago. Los casos de uso más comunes suelen implicar las consultas de búsqueda por proximidad; por ejemplo, "encontrar todas las cafeterías cerca de la ubicación actual". 

### <a name="geojson"></a>GeoJSON
Azure Cosmos DB admite la indexación y consulta de datos de puntos geoespaciales que se representan mediante la [especificación GeoJSON](https://tools.ietf.org/html/rfc7946). Las estructuras de datos de GeoJSON son siempre objetos JSON válidos, por lo que se pueden almacenar y consultar mediante Azure Cosmos DB sin tener que usar herramientas ni bibliotecas especializadas. El SDK de Azure Cosmos DB proporcionan clases y métodos auxiliares que facilitan el trabajo con datos espaciales. 

### <a name="points-linestrings-and-polygons"></a>Elementos Point, LineString y Polygon
Un **punto** denota una posición única en el espacio. En los datos geoespaciales, un elemento Point (punto) representa la ubicación exacta, que puede ser la dirección de una tienda de ultramarinos, un quiosco, un automóvil o una ciudad.  Un punto se representa en GeoJSON (y Azure Cosmos DB) mediante su par de coordenadas o su longitud y latitud. Este es un ejemplo JSON para un punto.

**Puntos en Azure Cosmos DB**

```json
{
    "type":"Point",
    "coordinates":[ 31.9, -4.8 ]
}
```

> [!NOTE]
> La especificación de GeoJSON especifica primero la longitud y segundo la latitud. Al igual que en otras aplicaciones de mapeado, la longitud y la latitud son ángulos y se representan en grados. Los valores de longitud se miden a partir del meridiano cero y están comprendidos entre -180 y 180,0 grados, mientras que los valores de latitud se miden a partir del Ecuador y están comprendidos entre -90,0 y 90,0 grados. 
> 
> Azure Cosmos DB interpreta las coordenadas tal como están representadas por el sistema de referencia WGS-84. Consulte la información que tiene a continuación para obtener más detalles acerca de los sistemas de coordenadas de referencia.
> 
> 

Esto se puede insertar en un documento Azure Cosmos DB tal como se muestra en este ejemplo de un perfil de usuario que contiene datos de ubicación:

**Uso de perfil con una ubicación almacenada en Azure Cosmos DB**

```json
{
    "id":"cosmosdb-profile",
    "screen_name":"@CosmosDB",
    "city":"Redmond",
    "topics":[ "global", "distributed" ],
    "location":{
        "type":"Point",
        "coordinates":[ 31.9, -4.8 ]
    }
}
```

Además de los puntos, GeoJSON también admite LineStrings y polígonos. **LineStrings** representan una serie de dos o más puntos en el espacio y los segmentos que los conectan. En los datos geoespaciales, los elementos LineString se usan normalmente para representar autopistas o ríos. Un elemento **Polygon** (polígono) es un área delimitada por puntos conectados que forman un elemento LineString cerrado. Los polígonos se usan normalmente para representar formaciones naturales como lagos, o jurisdicciones políticas como estados y ciudades. Este es un ejemplo de un Polygon en Azure Cosmos DB. 

**Polygons en GeoJSON**

```json
{
    "type":"Polygon",
    "coordinates":[ [
        [ 31.8, -5 ],
        [ 31.8, -4.7 ],
        [ 32, -4.7 ],
        [ 32, -5 ],
        [ 31.8, -5 ]
    ] ]
}
```

> [!NOTE]
> La especificación de GeoJSON requiere que para que un elemento Polygon sea válido, el último par de coordenadas indicado tiene que ser el mismo que el primero, para así crear una forma cerrada.
> 
> El orden de los puntos dentro de un elemento Polygon debe especificarse siguiendo el sentido contrario al de las agujas del reloj. Un elemento Polygon cuyos puntos se hayan especificado en el sentido de las agujas del reloj representa el inverso de la región dentro de él.
> 
> 

Además de elementos Point, LineString y Polygon, GeoJSON también especifica la representación que indica cómo agrupar varias ubicaciones geoespaciales, así como la forma de asociar propiedades arbitrarias a la geolocalización como si fueran una **Característica**. Dado que estos objetos son JSON válidos, se pueden almacenar y procesar todos en Azure Cosmos DB. Sin embargo, Azure Cosmos DB solo admite la indexación automática de puntos.

### <a name="coordinate-reference-systems"></a>Sistemas de coordenadas de referencia
Dado que la forma de la tierra es irregular, las coordenadas de los datos geoespaciales se representan en muchos sistemas de coordenadas de referencia (CRS), cada uno con sus propios marcos de referencia y unidades de medida. Por ejemplo, la "National Grid of Britain" es un sistema de referencia preciso para el Reino Unido, pero no fuera de él. 

El sistema de coordenadas de referencia más popular en uso hoy en día es el Sistema Geodésico Mundial [WGS 84](http://earth-info.nga.mil/GandG/wgs84/). Los dispositivos GPS y muchos servicios de mapeado como Google Maps y API de Bing Maps, usan WGS 84. Azure Cosmos DB admite indexación y consulta de datos geoespaciales usando solo el sistema de coordenadas WGS 84. 

## <a name="creating-documents-with-spatial-data"></a>Creación de documentos con datos espaciales
Al crear documentos que contengan valores GeoJSON, se indexan de forma automática con un índice espacial de acuerdo con la directiva de indexación del contenedor. Si está trabajando con un SDK de Azure Cosmos DB en un lenguaje de tipo dinámico como Python o Node.js, debe crear especificaciones GeoJSON válidas.

**Creación de documentos con datos geoespaciales en Node.js**

```json
var userProfileDocument = {
    "name":"cosmosdb",
    "location":{
        "type":"Point",
        "coordinates":[ -122.12, 47.66 ]
    }
};

client.createDocument(`dbs/${databaseName}/colls/${collectionName}`, userProfileDocument, (err, created) => {
    // additional code within the callback
});
```

Si está trabajando con las API de SQL, puede usar las clases `Point` y `Polygon` en el espacio de nombres `Microsoft.Azure.Documents.Spatial` para insertar información de ubicación dentro de los objetos de la aplicación. Estas clases ayudan a simplificar la serialización y deserialización de datos espaciales en GeoJSON.

**Creación de documentos con datos geoespaciales en .NET**

```json
using Microsoft.Azure.Documents.Spatial;

public class UserProfile
{
    [JsonProperty("name")]
    public string Name { get; set; }

    [JsonProperty("location")]
    public Point Location { get; set; }

    // More properties
}

await client.CreateDocumentAsync(
    UriFactory.CreateDocumentCollectionUri("db", "profiles"), 
    new UserProfile 
    { 
        Name = "cosmosdb", 
        Location = new Point (-122.12, 47.66) 
    });
```

Si no tiene la información de latitud y longitud, pero tiene las direcciones físicas o nombre de la ubicación como ciudad o país o región, puede buscar las coordenadas reales mediante el uso de un servicio de geocodificación como servicios de REST de mapas de Bing. Obtener más información acerca de la codificación geográfica de Bing Maps [aquí](https://msdn.microsoft.com/library/ff701713.aspx).

## <a name="querying-spatial-types"></a>Consulta de tipos espaciales
Ahora que ya hemos visto cómo insertar datos geoespaciales, echemos un vistazo a cómo consultar estos datos mediante Azure Cosmos DB con SQL y LINQ.

### <a name="spatial-sql-built-in-functions"></a>Funciones integradas SQL espaciales
Azure Cosmos DB admite las siguientes funciones integradas de Open Geospatial Consortium (OGC) para realizar consultas geoespaciales. Para más información sobre el conjunto completo de funciones integradas en el lenguaje SQL, vea [Consultas de Azure Cosmos DB](how-to-sql-query.md).

|**Uso**|**Descripción**|
|---|---|
| ST_DISTANCE (spatial_expr, spatial_expr) | Devuelve la distancia entre dos expresiones Point, Polygon o LineString de GeoJSON.|
|ST_WITHIN (spatial_expr, spatial_expr) | Devuelve una expresión booleana que indica si el primer objeto de GeoJSON (Point, Polygon o LineString) está en el segundo objeto de GeoJSON (Point, Polygon o LineString).|
|ST_INTERSECTS (spatial_expr, spatial_expr)| Devuelve una expresión booleana que indica si los dos objetos de GeoJSON especificados (Point, Polygon o LineString) intersecan.|
|ST_ISVALID| Devuelve un valor booleano que indica si la expresión de Point, Polygon o LineString de GeoJSON especificada es válida.|
| ST_ISVALIDDETAILED| Devuelve un valor JSON que contiene un valor booleano si la expresión de Point, Polygon o LineString de GeoJSON especificada es válida; si no es válida, devuelve también el motivo como un valor de cadena.|

Las funciones espaciales pueden usarse para realizar consultas de proximidad con datos espaciales. Por ejemplo, aquí hay una consulta que devuelve todos los documentos de la familia que estén dentro de un radio de 30 km de la ubicación especificada mediante la función integrada ST_DISTANCE. 

**Consultar**

    SELECT f.id 
    FROM Families f 
    WHERE ST_DISTANCE(f.location, {'type': 'Point', 'coordinates':[31.9, -4.8]}) < 30000

**Resultados**

    [{
      "id": "WakefieldFamily"
    }]

Si incluye la indexación espacial en la directiva de indexación, las "consultas de distancia" se atenderán eficazmente a través del índice. Para más información sobre la indexación espacial, consulte la sección siguiente. Aunque no tenga un índice espacial para las rutas de acceso especificadas, aún podrá realizar consultas espaciales mediante la especificación del encabezado de solicitud `x-ms-documentdb-query-enable-scan` con el valor establecido en "true". En. NET, para hacerlo es preciso pasar el argumento **FeedOptions** opcional en consultas en las que [EnableScanInQuery](https://msdn.microsoft.com/library/microsoft.azure.documents.client.feedoptions.enablescaninquery.aspx#P:Microsoft.Azure.Documents.Client.FeedOptions.EnableScanInQuery) está establecido en true. 

ST_WITHIN puede usarse para comprobar si un punto se encuentra dentro de un elemento Polygon. Normalmente, los elementos Polygon se usan para representar límites, como códigos postales, fronteras o formaciones naturales. Una vez más, si incluye la indexación espacial en la directiva de indexación, las consultas "interiores" se atenderán eficazmente a través del índice. 

Los argumentos de Polygon en ST_WITHIN solo pueden contener un anillo individual; es decir, los elementos Polygon no deben contener orificios. 

**Consultar**

    SELECT * 
    FROM Families f 
    WHERE ST_WITHIN(f.location, {
        'type':'Polygon', 
        'coordinates': [[[31.8, -5], [32, -5], [32, -4.7], [31.8, -4.7], [31.8, -5]]]
    })

**Resultados**

    [{
      "id": "WakefieldFamily",
    }]

> [!NOTE]
> De forma parecida a cómo funcionan los tipos no coincidentes en una consulta de Azure Cosmos DB, si el valor de ubicación especificado en cualquier argumento está mal formado o no es válido, se evalúa con el valor **sin definir** y el documento evaluado se omite de los resultados de la consulta. Si la consulta no devuelve resultados, ejecute ST_ISVALIDDETAILED para depurarla y saber por qué el tipo espacial no es válido.     
> 
> 

Azure Cosmos DB también admite la realización de consultas inversas; es decir, puede indexar elementos polígonos o líneas en Azure Cosmos DB y, a continuación, consultar las áreas que contienen un punto especificado. Este patrón se utiliza habitualmente en logística para identificar, por ejemplo, el momento en que un camión entra o sale de un área designada. 

**Consultar**

    SELECT * 
    FROM Areas a 
    WHERE ST_WITHIN({'type': 'Point', 'coordinates':[31.9, -4.8]}, a.location)


**Resultados**

    [{
      "id": "MyDesignatedLocation",
      "location": {
        "type":"Polygon", 
        "coordinates": [[[31.8, -5], [32, -5], [32, -4.7], [31.8, -4.7], [31.8, -5]]]
      }
    }]

ST_ISVALID y ST_ISVALIDDETAILED pueden usarse para comprobar si un objeto espacial es válido. Por ejemplo, la consulta siguiente comprueba la validez de un punto con un valor de latitud fuera del intervalo (-132,8). ST_ISVALID devuelve solo un valor booleano y ST_ISVALIDDETAILED devuelve el valor booleano y una cadena que contiene el motivo por el que se considera no válida.

**Consultar**

    SELECT ST_ISVALID({ "type": "Point", "coordinates": [31.9, -132.8] })

**Resultados**

    [{
      "$1": false
    }]

Estas funciones también se pueden usar para validar elementos Polygon. Por ejemplo, aquí usamos ST_ISVALIDDETAILED para validar un elemento Polygon que no está cerrado. 

**Consultar**

    SELECT ST_ISVALIDDETAILED({ "type": "Polygon", "coordinates": [[ 
        [ 31.8, -5 ], [ 31.8, -4.7 ], [ 32, -4.7 ], [ 32, -5 ] 
        ]]})

**Resultados**

    [{
       "$1": { 
            "valid": false, 
            "reason": "The Polygon input is not valid because the start and end points of the ring number 1 are not the same. Each ring of a Polygon must have the same start and end points." 
          }
    }]

### <a name="linq-querying-in-the-net-sdk"></a>Consultas LINQ en el SDK de .NET
El SDK de .NET de SQL también proporciona los métodos de código auxiliar `Distance()` y `Within()` para su uso en expresiones LINQ. El proveedor LINQ de SQL traduce estas llamadas al método a las llamadas de función integrada de SQL equivalentes (ST_DISTANCE y ST_WITHIN respectivamente). 

Este es un ejemplo de una consulta LINQ que busca todos los documentos de la colección de Azure Cosmos DB cuyo valor de "ubicación" se encuentra en un radio de 30 km del punto especificado mediante LINQ.

**Consulta LINQ de distancia**

    foreach (UserProfile user in client.CreateDocumentQuery<UserProfile>(UriFactory.CreateDocumentCollectionUri("db", "profiles"))
        .Where(u => u.ProfileType == "Public" && a.Location.Distance(new Point(32.33, -4.66)) < 30000))
    {
        Console.WriteLine("\t" + user);
    }

De forma similar, aquí vemos una consulta para buscar todos los documentos cuya "ubicación" está dentro del cuadro o del elemento Polygon especificado. 

**Consulta LINQ interna**

    Polygon rectangularArea = new Polygon(
        new[]
        {
            new LinearRing(new [] {
                new Position(31.8, -5),
                new Position(32, -5),
                new Position(32, -4.7),
                new Position(31.8, -4.7),
                new Position(31.8, -5)
            })
        });

    foreach (UserProfile user in client.CreateDocumentQuery<UserProfile>(UriFactory.CreateDocumentCollectionUri("db", "profiles"))
        .Where(a => a.Location.Within(rectangularArea)))
    {
        Console.WriteLine("\t" + user);
    }


Ahora que hemos visto cómo consultar documentos con LINQ y SQL, echemos un vistazo a cómo configurar Azure Cosmos DB para la indexación espacial.

## <a name="indexing"></a>Indización
Como se describe en el artículo [Indexación independiente de esquemas con Azure Cosmos DB](https://www.vldb.org/pvldb/vol8/p1668-shukla.pdf), diseñamos el motor de base de datos de Azure Cosmos DB para que sea realmente independiente de los esquemas y proporcione compatibilidad de primera clase con JSON. Ahora el motor de base de datos de escritura optimizado de Azure Cosmos DB también entiende de forma nativa los datos espaciales (puntos, elementos Polygon y líneas) representados en el estándar GeoJSON.

En pocas palabras, la geometría se proyecta a partir de coordenadas geodésicas en un plano 2D; después, se divide progresivamente en celdas con un **quadtree**. Estas celdas se asignan a 1D según la ubicación de la celda dentro de una **curva de Hilbert**, que conserva la localidad de los puntos. Además, al indexar los datos de ubicación, estos pasan por un proceso conocido como **teselación**; es decir, todas las celdas que se cruzan en una ubicación se identifican y se almacenan como claves en el índice de Azure Cosmos DB. En el momento de la consulta, los argumentos como puntos y elementos Polygon también se teselan para extraer los intervalos de Id. de celda pertinentes, y luego se usan para recuperar los datos del índice.

Si especifica una directiva de indexación que incluye el índice espacial para / * (todas las rutas de acceso), entonces todos los puntos dentro de la colección se indexan para que las consultas espaciales resulten eficaces (ST_WITHIN y ST_DISTANCE). Los índices espaciales no tienen un valor de precisión y usan siempre un valor de precisión predeterminado.

> [!NOTE]
> Azure Cosmos DB admite la indexación automática de puntos, elementos Polygon y LineString.
> 
> 

El siguiente fragmento de código de JSON se muestra una directiva de indexación con la indexación espacial habilitada; es decir, se indexa cualquier punto GeoJSON que se encuentre dentro de los documentos para la consulta espacial. Si va a modificar la directiva de indexación mediante Azure Portal, puede especificar el siguiente elemento JSON para que la directiva de indexación habilite la indexación espacial en la colección.

**JSON de directiva de indexación de una colección con la característica espacial habilitada para puntos y elementos Polygon**

    {
       "automatic":true,
       "indexingMode":"Consistent",
       "includedPaths":[
          {
             "path":"/*",
             "indexes":[
                {
                   "kind":"Range",
                   "dataType":"String",
                   "precision":-1
                },
                {
                   "kind":"Range",
                   "dataType":"Number",
                   "precision":-1
                },
                {
                   "kind":"Spatial",
                   "dataType":"Point"
                },
                {
                   "kind":"Spatial",
                   "dataType":"Polygon"
                }                
             ]
          }
       ],
       "excludedPaths":[
       ]
    }

Aquí vemos un fragmento de código de .NET que muestra cómo crear una colección con índices espaciales activados para todas las rutas que contengan puntos. 

**Creación de una colección con indexación espacial**

    DocumentCollection spatialData = new DocumentCollection()
    spatialData.IndexingPolicy = new IndexingPolicy(new SpatialIndex(DataType.Point)); //override to turn spatial on by default
    collection = await client.CreateDocumentCollectionAsync(UriFactory.CreateDatabaseUri("db"), spatialData);

Y aquí vemos cómo puede modificar una colección existente para aprovechar las ventajas de la indexación espacial de los puntos que se almacenan en los documentos.

**Modificación de una colección ya existente con indexación espacial**

    Console.WriteLine("Updating collection with spatial indexing enabled in indexing policy...");
    collection.IndexingPolicy = new IndexingPolicy(new SpatialIndex(DataType.Point));
    await client.ReplaceDocumentCollectionAsync(collection);

    Console.WriteLine("Waiting for indexing to complete...");
    long indexTransformationProgress = 0;
    while (indexTransformationProgress < 100)
    {
        ResourceResponse<DocumentCollection> response = await client.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri("db", "coll"));
        indexTransformationProgress = response.IndexTransformationProgress;

        await Task.Delay(TimeSpan.FromSeconds(1));
    }

> [!NOTE]
> Si el valor GeoJSON de la ubicación que se encuentra en el documento es incorrecto o no válido, no se indexará para realizar consultas espaciales. Puede validar los valores de ubicación mediante ST_ISVALID y ST_ISVALIDDETAILED.
> 
> Si la definición de la colección incluye una clave de partición, no se notifica el progreso de transformación de la indexación. 
> 
> 

## <a name="next-steps"></a>Pasos siguientes
Ahora que ya sabe cómo empezar a trabajar con la compatibilidad geoespacial en Azure Cosmos DB, puede hacer lo siguiente:

* Comenzar a codificar con los [ejemplos de código geoespacial .NET en GitHub](https://github.com/Azure/azure-documentdb-dotnet/blob/fcf23d134fc5019397dcf7ab97d8d6456cd94820/samples/code-samples/Geospatial/Program.cs)
* Empezar a realizar consultas geoespaciales en el [Área de consultas de Azure Cosmos DB](https://www.documentdb.com/sql/demo#geospatial)
* Obtener más información sobre [Consultar Azure Cosmos DB](how-to-sql-query.md)
* Obtener más información sobre [Directivas de indexación de Azure Cosmos DB](index-policy.md)

