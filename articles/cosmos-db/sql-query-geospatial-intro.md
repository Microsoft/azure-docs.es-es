---
title: Datos de ubicación geoespaciales y GeoJSON en Azure Cosmos DB
description: Aprenda a crear objetos espaciales con Azure Cosmos DB y SQL API.
author: timsander1
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 02/20/2020
ms.author: tisande
ms.custom: devx-track-javascript
ms.openlocfilehash: 25150722e2d42625731cb741be80b86645c857e0
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87420097"
---
# <a name="geospatial-and-geojson-location-data-in-azure-cosmos-db"></a>Datos de ubicación geoespaciales y GeoJSON en Azure Cosmos DB

Este artículo es una introducción a la funcionalidad geoespacial de Azure Cosmos DB. Actualmente, solo las cuentas de SQL API de Azure Cosmos DB admiten almacenamiento y acceso a datos geoespaciales. Después de leer la documentación sobre la indexación geoespacial, podrá responder a las siguientes preguntas:

* ¿Cómo almaceno los datos espaciales en Azure Cosmos DB?
* ¿Cómo puedo consultar los datos geoespaciales en Azure Cosmos DB en SQL y LINQ?
* ¿Qué tengo que hacer para habilitar y deshabilitar la indexación en Azure Cosmos DB?

## <a name="spatial-data-use-cases"></a>Casos de uso de datos espaciales

Los datos geoespaciales suelen implicar las consultas de búsqueda por proximidad; por ejemplo, "encontrar todas las cafeterías cerca de la ubicación actual". Los casos de uso comunes son los siguientes:

* Análisis de geolocalización, impulsar iniciativas de marketing específicas ubicadas.
* Personalización basada en la ubicación, para varios sectores como la venta al por menor y la atención sanitaria.
* Mejora de la logística para la optimización del transporte.
* Análisis de riesgos, especialmente para compañías de seguros y financieras.
* Reconocimiento de situaciones, para alertas y notificaciones.

## <a name="introduction-to-spatial-data"></a>Introducción a los datos espaciales

Los datos espaciales describen la posición y la forma de los objetos en el espacio. En la mayoría de las aplicaciones, estos se refieren a objetos situados en la tierra y datos geoespaciales. Los datos espaciales pueden usarse para representar la ubicación de una persona, un lugar de interés o el límite de una ciudad o un lago.

SQL API de Azure Cosmos DB admite dos tipos de datos espaciales: el tipo de datos **geométricos** y el tipo de datos **geográficos**.

- El tipo **geométrico** representa los datos en un sistema de coordenadas euclidiano (plano).
- El tipo **geography** representa los datos en un sistema de coordenadas de tierra redonda.

## <a name="supported-data-types"></a>Tipos de datos admitidos

Azure Cosmos DB admite la indexación y consulta de datos de puntos geoespaciales que se representan mediante la [especificación GeoJSON](https://tools.ietf.org/html/rfc7946). Las estructuras de datos de GeoJSON son siempre objetos JSON válidos, por lo que se pueden almacenar y consultar mediante Azure Cosmos DB sin tener que usar herramientas ni bibliotecas especializadas.

Azure Cosmos DB admite los siguientes tipos de datos espaciales:

- Punto
- LineString
- Polygon
- MultiPolygon

### <a name="points"></a>Puntos

Un **punto** denota una posición única en el espacio. En los datos geoespaciales, un elemento Point (punto) representa la ubicación exacta, que puede ser la dirección de una tienda de ultramarinos, un quiosco, un automóvil o una ciudad.  Un punto se representa en GeoJSON (y Azure Cosmos DB) mediante su par de coordenadas o su longitud y latitud.

Este es un ejemplo JSON de un punto:

**Puntos en Azure Cosmos DB**

```json
{
    "type":"Point",
    "coordinates":[ 31.9, -4.8 ]
}
```

Los tipos de datos espaciales se pueden insertar en un documento de Azure Cosmos DB, como se muestra en este ejemplo de un perfil de usuario que contiene datos de ubicación:

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

### <a name="points-in-a-geometry-coordinate-system"></a>Puntos de un sistema de coordenadas de geometría

En el caso del tipo de datos **geométricos**, la especificación de GeoJSON especifica primero el eje horizontal y, después, el eje vertical.

### <a name="points-in-a-geography-coordinate-system"></a>Puntos de un sistema de coordenadas de geografía

Para el tipo de datos **geography**, la especificación GeoJSON indica la longitud primero y la latitud después. Al igual que en otras aplicaciones de mapeado, la longitud y la latitud son ángulos y se representan en grados. Los valores de longitud se miden a partir del meridiano cero y están comprendidos entre -180 y 180,0 grados, mientras que los valores de latitud se miden a partir del Ecuador y están comprendidos entre -90,0 y 90,0 grados.

Azure Cosmos DB interpreta las coordenadas tal como están representadas por el sistema de referencia WGS-84. Consulte la información que tiene a continuación para obtener más detalles acerca de los sistemas de coordenadas de referencia.

### <a name="linestrings"></a>LineStrings

**LineStrings** representan una serie de dos o más puntos en el espacio y los segmentos que los conectan. En los datos geoespaciales, los elementos LineString se usan normalmente para representar autopistas o ríos.

**LineStrings en GeoJSON**

```json
    "type":"LineString",
    "coordinates":[ [
        [ 31.8, -5 ],
        [ 31.8, -4.7 ]
    ] ]
```

### <a name="polygons"></a>Polígonos

Un elemento **Polygon** (polígono) es un área delimitada por puntos conectados que forman un elemento LineString cerrado. Los polígonos se usan normalmente para representar formaciones naturales como lagos, o jurisdicciones políticas como estados y ciudades. Este es un ejemplo de un elemento Polygon en Azure Cosmos DB.

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

### <a name="multipolygons"></a>MultiPolygons

Un elemento **MultiPolygon** es una matriz de cero o más elementos Polygon. Los elementos **MultiPolygon** no pueden superponerse a los lados ni tener ninguna área común. Pueden tocarse en uno o varios puntos.

**MultiPolygons en GeoJSON**

```json
{
    "type":"MultiPolygon",
    "coordinates":[[[
        [52.0, 12.0],
        [53.0, 12.0],
        [53.0, 13.0],
        [52.0, 13.0],
        [52.0, 12.0]
        ]],
        [[
        [50.0, 0.0],
        [51.0, 0.0],
        [51.0, 5.0],
        [50.0, 5.0],
        [50.0, 0.0]
        ]]]
}
```

## <a name="coordinate-reference-systems"></a>Sistemas de coordenadas de referencia

Dado que la forma de la tierra es irregular, las coordenadas de los datos geoespaciales de geografía se representan en muchos sistemas de coordenadas de referencia (CRS), cada uno con sus propios marcos de referencia y unidades de medida. Por ejemplo, la "National Grid of Britain" es un sistema de referencia preciso para el Reino Unido, pero no fuera de él.

El sistema de coordenadas de referencia más popular en uso hoy en día es el Sistema Geodésico Mundial [WGS 84](https://earth-info.nga.mil/GandG/update/index.php). Los dispositivos GPS y muchos servicios de mapeado como Google Maps y API de Bing Maps, usan WGS 84. Azure Cosmos DB admite la indexación y consulta de datos geoespaciales de geografía usando solo el CSR WGS-84.

## <a name="creating-documents-with-spatial-data"></a>Creación de documentos con datos espaciales
Al crear documentos que contengan valores GeoJSON, se indexan de forma automática con un índice espacial de acuerdo con la directiva de indexación del contenedor. Si está trabajando con un SDK de Azure Cosmos DB en un lenguaje de tipo dinámico como Python o Node.js, debe crear especificaciones GeoJSON válidas.

**Creación de documentos con datos geoespaciales en Node.js**

```javascript
var userProfileDocument = {
    "id":"cosmosdb",
    "location":{
        "type":"Point",
        "coordinates":[ -122.12, 47.66 ]
    }
};

client.createDocument(`dbs/${databaseName}/colls/${collectionName}`, userProfileDocument, (err, created) => {
    // additional code within the callback
});
```

Si va a trabajar con instancias de SQL API, puede usar las clases `Point`, `LineString`, `Polygon` y `MultiPolygon` en el espacio de nombres `Microsoft.Azure.Cosmos.Spatial` para insertar la información de ubicación en los objetos de la aplicación. Estas clases ayudan a simplificar la serialización y deserialización de datos espaciales en GeoJSON.

**Creación de documentos con datos geoespaciales en .NET**

```csharp
using Microsoft.Azure.Cosmos.Spatial;

public class UserProfile
{
    [JsonProperty("id")]
    public string id { get; set; }

    [JsonProperty("location")]
    public Point Location { get; set; }

    // More properties
}

await container.CreateItemAsync( new UserProfile
    {
        id = "cosmosdb",
        Location = new Point (-122.12, 47.66)
    });
```

Si no dispone de la información de latitud y longitud, pero tiene los nombres de ubicación como ciudad o país/región o direcciones físicas, puede buscar las coordenadas reales mediante un servicio de codificación geográfica, como los servicios de REST de Bing Maps. Obtener más información acerca de la codificación geográfica de Bing Maps [aquí](https://msdn.microsoft.com/library/ff701713.aspx).

## <a name="next-steps"></a>Pasos siguientes

Ahora que ya sabe cómo empezar a trabajar con la compatibilidad geoespacial en Azure Cosmos DB, puede hacer lo siguiente:

* Obtener más información sobre [Consultar Azure Cosmos DB](sql-query-getting-started.md)
* Más información sobre la [consulta de datos espaciales con Azure Cosmos DB](sql-query-geospatial-query.md)
* Más información sobre la [indexación de datos espaciales con Azure Cosmos DB](sql-query-geospatial-index.md)
