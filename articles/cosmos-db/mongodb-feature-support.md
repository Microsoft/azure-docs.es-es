---
title: 'API de Azure Cosmos DB para MongoDB (versión 3.2): características y sintaxis que se admiten'
description: 'Conozca más información sobre la API de Azure Cosmos DB para MongoDB (versión 3.2): características y sintaxis que se admiten.'
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: overview
ms.date: 10/16/2019
author: sivethe
ms.author: sivethe
ms.openlocfilehash: 7c5164a032e77d85c995384473935b134ff528e5
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88009314"
---
# <a name="azure-cosmos-dbs-api-for-mongodb-32-version-supported-features-and-syntax"></a>API de Azure Cosmos DB para MongoDB (versión 3.2): características y sintaxis que se admiten

Azure Cosmos DB es un servicio de base de datos con varios modelos y de distribución global de Microsoft. Puede comunicarse con la API de Azure Cosmos DB para MongoDB mediante cualquiera de los [controladores](https://docs.mongodb.org/ecosystem/drivers) del cliente de MongoDB de código abierto. La API de Azure Cosmos DB para MongoDB permite usar los controladores de cliente existentes mediante la adhesión al [protocolo de conexión](https://docs.mongodb.org/manual/reference/mongodb-wire-protocol) de MongoDB.

Con la API de Azure Cosmos DB para MongoDB, puede disfrutar de las ventajas de MongoDB a las que está acostumbrado, con todas las funcionalidades empresariales que ofrece Cosmos DB: [distribución global](distribute-data-globally.md), [particionamiento automático](partition-data.md), garantías de disponibilidad y latencia, indexación automática de cada campo, cifrado en reposo, copias de seguridad y mucho más.

> [!NOTE]
> Este artículo trata sobre la API de Azure Cosmos DB para MongoDB 3.2. Para MongoDB versión 3.6, consulte las [características y sintaxis que se admiten para MongoDB 3.6](mongodb-feature-support-36.md).

## <a name="protocol-support"></a>Compatibilidad con protocolos

Todas las cuentas nuevas de la API de Azure Cosmos DB para MongoDB son compatibles de manera predeterminada con la versión del servidor **3.6** de MongoDB. Este artículo trata sobre la versión 3.2 de MongoDB. A continuación se enumeran los operadores admitidos y las limitaciones o excepciones. Cualquier controlador de cliente que reconozca estos protocolos podrá conectarse a la API de Azure Cosmos DB para MongoDB.

## <a name="query-language-support"></a>Compatibilidad con lenguajes de consulta

La API de Azure Cosmos DB para MongoDB proporciona una compatibilidad completa con las construcciones del lenguaje de consulta de MongoDB. A continuación, encontrará una lista detallada de las opciones, comandos, fases, operadores y operaciones compatibles actualmente.

## <a name="database-commands"></a>Comandos de base de datos

La API de Azure Cosmos DB para MongoDB admite los siguientes comandos de base de datos:

### <a name="query-and-write-operation-commands"></a>Comandos de operación de consulta y escritura

- delete
- find
- findAndModify
- getLastError
- getMore
- insert
- update

### <a name="authentication-commands"></a>Comandos de autenticación

- logout
- authenticate
- getnonce

### <a name="administration-commands"></a>Comandos de administración

- dropDatabase
- listCollections
- drop
- create
- filemd5
- createIndexes
- listIndexes
- dropIndexes
- connectionStatus
- reIndex

### <a name="diagnostics-commands"></a>Comandos de diagnóstico

- buildInfo
- collStats
- dbStats
- hostInfo
- listDatabases
- whatsmyuri

<a name="aggregation-pipeline"></a>

## <a name="aggregation-pipelinea"></a>Canalización de agregación</a>

Cosmos DB es compatible con la canalización de agregación para MongoDB 3.2 en la versión preliminar pública. Vea el [blog de Azure](https://azure.microsoft.com/blog/azure-cosmosdb-extends-support-for-mongodb-aggregation-pipeline-unique-indexes-and-more/) para obtener instrucciones sobre cómo incorporarse a la versión preliminar pública.

### <a name="aggregation-commands"></a>Comandos de agregación

- aggregate
- count
- distinct

### <a name="aggregation-stages"></a>Fases de agregación

- $project
- $match
- $limit
- $skip
- $unwind
- $group
- $sample
- $sort
- $lookup
- $out
- $count
- $addFields

### <a name="aggregation-expressions"></a>Expresiones de agregación

#### <a name="boolean-expressions"></a>Expresiones booleanas

- $and
- $or
- $not

#### <a name="set-expressions"></a>Expresiones de conjunto

- $setEquals
- $setIntersection
- $setUnion
- $setDifference
- $setIsSubset
- $anyElementTrue
- $allElementsTrue

#### <a name="comparison-expressions"></a>Expresiones de comparación

- $cmp
- $eq
- $gt
- $gte
- $lt
- $lte
- $ne

#### <a name="arithmetic-expressions"></a>Expresiones aritméticas

- $abs
- $add
- $ceil
- $divide
- $exp
- $floor
- $ln
- $log
- $log10
- $mod
- $multiply
- $pow
- $sqrt
- $subtract
- $trunc

#### <a name="string-expressions"></a>Expresiones de cadena

- $concat
- $indexOfBytes
- $indexOfCP
- $split
- $strLenBytes
- $strLenCP
- $strcasecmp
- $substr
- $substrBytes
- $substrCP
- $toLower
- $toUpper

#### <a name="array-expressions"></a>Expresiones de matriz

- $arrayElemAt
- $concatArrays
- $filter
- $indexOfArray
- $isArray
- $range
- $reverseArray
- $size
- $slice
- $in

#### <a name="date-expressions"></a>Expresiones de fecha

- $dayOfYear
- $dayOfMonth
- $dayOfWeek
- $year
- $month
- $week
- $hour
- $minute
- $second
- $millisecond
- $isoDayOfWeek
- $isoWeek

#### <a name="conditional-expressions"></a>Expresiones condicionales

- $cond
- $ifNull

## <a name="aggregation-accumulators"></a>Acumuladores de agregación

- $sum
- $avg
- $first
- $last
- $max
- $min
- $push
- $addToSet

## <a name="operators"></a>Operadores

Los siguientes operadores son compatibles con los correspondientes ejemplos de su uso. Tenga en cuenta este documento de muestra utilizada en las consultas siguientes:

```json
{
  "Volcano Name": "Rainier",
  "Country": "United States",
  "Region": "US-Washington",
  "Location": {
    "type": "Point",
    "coordinates": [
      -121.758,
      46.87
    ]
  },
  "Elevation": 4392,
  "Type": "Stratovolcano",
  "Status": "Dendrochronology",
  "Last Known Eruption": "Last known eruption from 1800-1899, inclusive"
}
```

Operator | Ejemplo |
--- | --- |
$eq | `{ "Volcano Name": { $eq: "Rainier" } }` |  | -
$gt | `{ "Elevation": { $gt: 4000 } }` |  | -
$gte | `{ "Elevation": { $gte: 4392 } }` |  | -
$lt | `{ "Elevation": { $lt: 5000 } }` |  | -
$lte | `{ "Elevation": { $lte: 5000 } }` | | -
$ne | `{ "Elevation": { $ne: 1 } }` |  | -
$in | `{ "Volcano Name": { $in: ["St. Helens", "Rainier", "Glacier Peak"] } }` |  | -
$nin | `{ "Volcano Name": { $nin: ["Lassen Peak", "Hood", "Baker"] } }` | | -
$or | `{ $or: [ { Elevation: { $lt: 4000 } }, { "Volcano Name": "Rainier" } ] }` |  | -
$and | `{ $and: [ { Elevation: { $gt: 4000 } }, { "Volcano Name": "Rainier" } ] }` |  | -
$not | `{ "Elevation": { $not: { $gt: 5000 } } }`|  | -
$nor | `{ $nor: [ { "Elevation": { $lt: 4000 } }, { "Volcano Name": "Baker" } ] }` |  | -
$exists | `{ "Status": { $exists: true } }`|  | -
$type | `{ "Status": { $type: "string" } }`|  | -
$mod | `{ "Elevation": { $mod: [ 4, 0 ] } }` |  | -
$regex | `{ "Volcano Name": { $regex: "^Rain"} }`|  | -

### <a name="notes"></a>Notas

En las consultas de $regex, las expresiones ancladas a la izquierda permiten la búsqueda de índice. Sin embargo, si utiliza el modificador 'i' (no distingue mayúsculas y minúsculas) y el modificador 'm' modificador (multilínea), se realiza el examen de colección de todas las expresiones.
Cuando es necesario incluir '$' o '|', es mejor crear dos (o más) consultas regex.
Por ejemplo, dada la siguiente consulta original: ```find({x:{$regex: /^abc$/})```, tiene que modificarse de la siguiente forma: ```find({x:{$regex: /^abc/, x:{$regex:/^abc$/}})```.
La primera parte utilizará el índice para restringir la búsqueda a esos documentos que empiezan por ^ abc y la segunda parte buscará coincidencias con los datos exactos.
El operador de barra '|' actúa como una función "or": la consulta ```find({x:{$regex: /^abc|^def/})``` coincide con los documentos con los que el campo "x" tiene un valor que comienza por "abc" o "def". Para utilizar el índice, se recomienda dividir la consulta en dos consultas distintas combinadas mediante el operador $or: ```find( {$or : [{x: $regex: /^abc/}, {$regex: /^def/}] })```.

### <a name="update-operators"></a>Operadores de actualización

#### <a name="field-update-operators"></a>Operadores de actualización de campo

- $inc
- $mul
- $rename
- $setOnInsert
- $set
- $unset
- $min
- $max
- $currentDate

#### <a name="array-update-operators"></a>Operadores de actualización de matriz

- $addToSet
- $pop
- $pullAll
- $pull (Nota: No se admite $pull con condición)
- $pushAll
- $push
- $each
- $slice
- $sort
- $position

#### <a name="bitwise-update-operator"></a>Operador de actualización bit a bit

- $bit

### <a name="geospatial-operators"></a>Operadores de geoespaciales

Operator | Ejemplo | Compatible |
--- | --- | --- |
$geoWithin | ```{ "Location.coordinates": { $geoWithin: { $centerSphere: [ [ -121, 46 ], 5 ] } } }``` | Sí |
$geoIntersects |  ```{ "Location.coordinates": { $geoIntersects: { $geometry: { type: "Polygon", coordinates: [ [ [ -121.9, 46.7 ], [ -121.5, 46.7 ], [ -121.5, 46.9 ], [ -121.9, 46.9 ], [ -121.9, 46.7 ] ] ] } } } }``` | Sí |
$near | ```{ "Location.coordinates": { $near: { $geometry: { type: "Polygon", coordinates: [ [ [ -121.9, 46.7 ], [ -121.5, 46.7 ], [ -121.5, 46.9 ], [ -121.9, 46.9 ], [ -121.9, 46.7 ] ] ] } } } }``` | Sí |
$nearSphere | ```{ "Location.coordinates": { $nearSphere : [ -121, 46  ], $maxDistance: 0.50 } }``` | Sí |
$geometry | ```{ "Location.coordinates": { $geoWithin: { $geometry: { type: "Polygon", coordinates: [ [ [ -121.9, 46.7 ], [ -121.5, 46.7 ], [ -121.5, 46.9 ], [ -121.9, 46.9 ], [ -121.9, 46.7 ] ] ] } } } }``` | Sí |
$minDistance | ```{ "Location.coordinates": { $nearSphere : { $geometry: {type: "Point", coordinates: [ -121, 46 ]}, $minDistance: 1000, $maxDistance: 1000000 } } }``` | Sí |
$maxDistance | ```{ "Location.coordinates": { $nearSphere : [ -121, 46  ], $maxDistance: 0.50 } }``` | Sí |
$center | ```{ "Location.coordinates": { $geoWithin: { $center: [ [-121, 46], 1 ] } } }``` | Sí |
$centerSphere | ```{ "Location.coordinates": { $geoWithin: { $centerSphere: [ [ -121, 46 ], 5 ] } } }``` | Sí |
$box | ```{ "Location.coordinates": { $geoWithin: { $box:  [ [ 0, 0 ], [ -122, 47 ] ] } } }``` | Sí |
$polygon | ```{ "Location.coordinates": { $near: { $geometry: { type: "Polygon", coordinates: [ [ [ -121.9, 46.7 ], [ -121.5, 46.7 ], [ -121.5, 46.9 ], [ -121.9, 46.9 ], [ -121.9, 46.7 ] ] ] } } } }``` | Sí |

## <a name="sort-operations"></a>Ordenar operaciones

Cuando se usa la operación `findOneAndUpdate`, se admiten las operaciones de ordenación en un solo campo, pero no se admiten las operaciones de ordenación en varios campos.

## <a name="additional-operators"></a>Operadores adicionales

Operator | Ejemplo | Notas
--- | --- | --- |
$all | ```{ "Location.coordinates": { $all: [-121.758, 46.87] } }``` |
$elemMatch | ```{ "Location.coordinates": { $elemMatch: {  $lt: 0 } } }``` |
$size | ```{ "Location.coordinates": { $size: 2 } }``` |
$comment |  ```{ "Location.coordinates": { $elemMatch: {  $lt: 0 } }, $comment: "Negative values"}``` |
$text |  | No compatible. Utilice $regex en su lugar.

## <a name="unsupported-operators"></a>Operadores no compatibles

Los operadores ```$where``` y ```$eval``` no son compatibles con Azure Cosmos DB.

### <a name="methods"></a>Métodos

Los siguientes métodos son compatibles:

#### <a name="cursor-methods"></a>Métodos de cursor

Método | Ejemplo | Notas
--- | --- | --- |
cursor.sort() | ```cursor.sort({ "Elevation": -1 })``` | No se devuelven los documentos sin criterio de ordenación.

## <a name="unique-indexes"></a>Índices únicos

Cosmos DB indexa cada campo de los documentos que se escriben en la base de datos de forma predeterminada. Los índices únicos garantizan que un campo concreto no tiene valores duplicados en todos los documentos de una colección, de forma similar a la manera en que se conserva la singularidad en la clave `_id` predeterminada. Puede crear índices personalizados en Cosmos DB mediante el comando createIndex, incluida la restricción "unique".

Hay disponibles índices únicos para todas las cuentas de Cosmos que usan la API de Azure Cosmos DB para MongoDB.

## <a name="time-to-live-ttl"></a>Período de vida (TTL)

Cosmos DB admite un período de vida (TTL) en función de la marca de tiempo del documento. TTL se puede habilitar para las colecciones mediante [Azure Portal](https://portal.azure.com).

## <a name="user-and-role-management"></a>Administración de usuarios y roles

Cosmos DB no admite aún usuarios y roles. Sin embargo, Cosmos DB admite el control de acceso basado en rol de Azure (Azure RBAC) y claves o contraseñas de solo lectura y escritura que se pueden obtener mediante [Azure Portal](https://portal.azure.com) (página Cadena de conexión).

## <a name="replication"></a>Replicación

Cosmos DB admite la replicación automática y nativa en las capas más inferiores. Esta lógica se amplía para lograr también una replicación global de baja latencia. Cosmos DB no es compatible con comandos de replicación manuales.

## <a name="write-concern"></a>Write Concern

Algunas aplicaciones se basan en [Write Concern](https://docs.mongodb.com/manual/reference/write-concern/), que especifica el número de respuestas requeridas durante una operación de escritura. Debido a la forma en la que Cosmos DB controla la replicación en un segundo plano todas las escrituras se establecen automáticamente como cuórum de forma predeterminada. Cualquier nivel de Write Concern especificado por el código de cliente se ignora. Más información en el artículo sobre el [Uso de los niveles de coherencia para maximizar la disponibilidad y el rendimiento](consistency-levels.md).

## <a name="sharding"></a>Particionamiento

Azure Cosmos DB admite el particionamiento de servidor automático. Administra la creación de particiones, la ubicación y el equilibrio de forma automática. Azure Cosmos DB no admite comandos de particionamiento manual, lo que significa que no tiene que invocar comandos como shardCollection, addShard, balancerStart, moveChunk, etc. Solo tiene que especificar la clave de partición al crear los contenedores o consultar los datos.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [usar Studio 3T](mongodb-mongochef.md) con la API de Azure Cosmos DB para MongoDB.
- Aprenda a [usar Robo 3T](mongodb-robomongo.md) con la API de Azure Cosmos DB para MongoDB.
- Explore [ejemplos](mongodb-samples.md) de MongoDB con la API de Azure Cosmos DB para MongoDB.

<sup>Nota: En este artículo se describe una característica de Azure Cosmos DB que proporciona compatibilidad del protocolo de conexión con bases de datos de MongoDB. Microsoft no ejecuta bases de datos de MongoDB que ofrezcan este servicio. Azure Cosmos DB no está afiliado a MongoDB, Inc.</sup>
