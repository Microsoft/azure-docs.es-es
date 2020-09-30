---
title: Indexación en Azure Cosmos DB
description: Conozca el funcionamiento de la indexación en Azure Cosmos DB, y cómo se admiten los diferentes tipos de índices como los compuestos, espaciales o de intervalo.
author: timsander1
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/21/2020
ms.author: tisande
ms.openlocfilehash: 3d07657fc3345ddd8dfadd163dc3c9f957d77af3
ms.sourcegitcommit: 1fe5127fb5c3f43761f479078251242ae5688386
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/14/2020
ms.locfileid: "90068394"
---
# <a name="indexing-in-azure-cosmos-db---overview"></a>Indexación en Azure Cosmos DB: introducción

Azure Cosmos DB es una base de datos independiente del esquema que le permite iterar en la aplicación sin tener que tratar con la administración de esquemas o índices. De forma predeterminada, Azure Cosmos DB indexa automáticamente todas las propiedades de todos los elementos de su [contenedor](databases-containers-items.md#azure-cosmos-containers) sin tener que definir ningún esquema ni configurar índices secundarios.

El objetivo de este artículo es explicar cómo Azure Cosmos DB indexa datos y cómo usa índices para mejorar el rendimiento de las consultas. Se recomienda revisar esta sección antes de explorar cómo personalizar las [directivas de indexación](index-policy.md).

## <a name="from-items-to-trees"></a>De elementos a árboles

Cada vez que un elemento se almacena en un contenedor, su contenido se proyecta como un documento JSON y luego se convierte en una representación de árbol. Eso significa que todas las propiedades de ese elemento se representan como nodos de un árbol. Para todas las propiedades de primer nivel del elemento, se crea un pseudonodo raíz como elemento primario. Los nodos hoja contienen los valores escalares reales pertenecientes a un elemento.

Por ejemplo, considere este elemento:

```json
    {
        "locations": [
            { "country": "Germany", "city": "Berlin" },
            { "country": "France", "city": "Paris" }
        ],
        "headquarters": { "country": "Belgium", "employees": 250 },
        "exports": [
            { "city": "Moscow" },
            { "city": "Athens" }
        ]
    }
```

Se podría representar mediante el árbol siguiente:

:::image type="content" source="./media/index-overview/item-as-tree.png" alt-text="El elemento anterior representado como árbol" border="false":::

Observe cómo se codifican las matrices en el árbol: cada entrada de una matriz obtiene un nodo intermedio etiquetado con el índice de esa entrada dentro de la matriz (0, 1, etc.).

## <a name="from-trees-to-property-paths"></a>De árboles a rutas de acceso de propiedades

El motivo por el que Azure Cosmos DB transforma los elementos en árboles es porque permite hacer referencia a las propiedades mediante sus rutas de acceso dentro de esos árboles. Para obtener la ruta de acceso de una propiedad, podemos recorrer el árbol desde el nodo raíz hasta esa propiedad y concatenar las etiquetas de cada nodo recorrido.

Estas son las rutas de acceso de cada propiedad del elemento de ejemplo descrito anteriormente:

- /locations/0/country: "Germany"
- /locations/0/city: "Berlin"
- /locations/1/country: "France"
- /locations/1/city: "Paris"
- /headquarters/country: "Belgium"
- /headquarters/employees: 250
- /exports/0/city: "Moscow"
- /exports/1/city: "Athens"

Cuando se escribe un elemento, Azure Cosmos DB indexa eficazmente la ruta de acceso de cada propiedad y su valor correspondiente.

## <a name="index-kinds"></a>Tipos de índices

Actualmente, Azure Cosmos DB admite tres tipos de índices.

### <a name="range-index"></a>Índice de rangos

El **índice de rangos** se basa en una estructura de árbol ordenada. El tipo de índice de rango se usa con:

- Consultas de igualdad:

    ```sql
   SELECT * FROM container c WHERE c.property = 'value'
   ```

   ```sql
   SELECT * FROM c WHERE c.property IN ("value1", "value2", "value3")
   ```

   Coincidencia de igualdad numérica en un elemento de matriz
   ```sql
    SELECT * FROM c WHERE ARRAY_CONTAINS(c.tags, "tag1")
    ```

- Consultas por rango:

   ```sql
   SELECT * FROM container c WHERE c.property > 'value'
   ```
  (funciona para `>`, `<`, `>=`, `<=`, `!=`).

- Comprobación de la presencia de una propiedad:

   ```sql
   SELECT * FROM c WHERE IS_DEFINED(c.property)
   ```

- Funciones del sistema de cadena:

   ```sql
   SELECT * FROM c WHERE CONTAINS(c.property, "value")
   ```

   ```sql
   SELECT * FROM c WHERE STRINGEQUALS(c.property, "value")
   ```

- Consultas `ORDER BY`:

   ```sql
   SELECT * FROM container c ORDER BY c.property
   ```

- Consultas `JOIN`:

   ```sql
   SELECT child FROM container c JOIN child IN c.properties WHERE child = 'value'
   ```

Los índices de rango se pueden usar en valores escalares (cadena o número).

### <a name="spatial-index"></a>Índice espacial

Los índices **espaciales** permiten realizar consultas eficaces en objetos geoespaciales como puntos, líneas, polígonos y multipolígonos. Estas consultas usan las palabras clave ST_DISTANCE, ST_WITHIN, ST_INTERSECTS. A continuación se muestran algunos ejemplos que usan el tipo de índice espacial:

- Consultas de distancia geoespaciales:

   ```sql
   SELECT * FROM container c WHERE ST_DISTANCE(c.property, { "type": "Point", "coordinates": [0.0, 10.0] }) < 40
   ```

- Información geoespacial dentro de consultas:

   ```sql
   SELECT * FROM container c WHERE ST_WITHIN(c.property, {"type": "Point", "coordinates": [0.0, 10.0] })
   ```

- Consultas de intersecciones geoespaciales:

   ```sql
   SELECT * FROM c WHERE ST_INTERSECTS(c.property, { 'type':'Polygon', 'coordinates': [[ [31.8, -5], [32, -5], [31.8, -5] ]]  })  
   ```

Los índices espaciales pueden usarse en objetos [GeoJSON](geospatial.md) con un formato correcto. Actualmente, se admiten Points, LineStrings, Polygons y MultiPolygons.

### <a name="composite-indexes"></a>Índices compuestos

Los índices **compuestos** aumentan la eficacia al realizar operaciones en varios campos. El tipo de índice compuesto se usa con:

- Consultas `ORDER BY` en varias propiedades:

```sql
 SELECT * FROM container c ORDER BY c.property1, c.property2
```

- Consultas con un filtro y `ORDER BY`. Estas consultas pueden emplear un índice compuesto si la propiedad de filtro se agrega a la cláusula `ORDER BY`.

```sql
 SELECT * FROM container c WHERE c.property1 = 'value' ORDER BY c.property1, c.property2
```

- Consultas con un filtro en dos o más propiedades en las que al menos una propiedad es un filtro de igualdad

```sql
 SELECT * FROM container c WHERE c.property1 = 'value' AND c.property2 > 'value'
```

Siempre y cuando un predicado de filtro use un tipo de índice, el motor de consultas evaluará ese primero antes de examinar el resto. Por ejemplo, si tiene una consulta SQL como `SELECT * FROM c WHERE c.firstName = "Andrew" and CONTAINS(c.lastName, "Liu")`

* La consulta anterior primero filtrará las entradas en las que firstName = "Andrew" mediante el índice. Después, pasa todas las entradas firstName = "Andrew" a través de una canalización subsiguiente para evaluar el predicado de filtro CONTAINS.

* Puede acelerar las consultas y evitar exámenes de todo el contenedor cuando emplee funciones que no usen el índice (por ejemplo, CONTAINS) mediante la incorporación de predicados de filtro adicionales que utilicen el índice. El orden de las cláusulas de filtro no es importante. El motor de consultas determinará qué predicados son más selectivos y ejecutará la consulta en consecuencia.

## <a name="querying-with-indexes"></a>Consultas con índices

Las rutas de acceso extraídas al indexar datos facilitan la tarea de buscar el índice al procesar una consulta. Al comparar la cláusula `WHERE` de una consulta con la lista de rutas de acceso indexadas, es posible identificar rápidamente los elementos que coinciden con el predicado de consulta.

Por ejemplo, considere la consulta siguiente: `SELECT location FROM location IN company.locations WHERE location.country = 'France'`. El predicado de consulta (filtrado por elementos, donde cualquier ubicación tiene "France" como su país) coincidiría con la ruta de acceso resaltada en rojo a continuación:

:::image type="content" source="./media/index-overview/matching-path.png" alt-text="Coincidencia con una ruta de acceso específica dentro de un árbol" border="false":::

> [!NOTE]
> Una cláusula `ORDER BY` que ordena por una sola propiedad *siempre* necesita un índice de rango y dará error si la ruta de acceso a la que hace referencia no tiene uno. Del mismo modo, una consulta `ORDER BY` que se ordena por varias propiedades *siempre* necesita un índice compuesto.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información acerca de la indexación en los siguientes artículos:

- [Directiva de indexación](index-policy.md)
- [Cómo administrar la directiva de indexación](how-to-manage-indexing-policy.md)
