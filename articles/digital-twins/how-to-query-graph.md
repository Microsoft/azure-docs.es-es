---
title: Consulta del grafo de gemelos
titleSuffix: Azure Digital Twins
description: Vea cómo consultar el grafo gemelo de Azure Digital Twins para información.
author: baanders
ms.author: baanders
ms.date: 3/26/2020
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: 3250e4c35f6b898f4431d0f2fe15f84d915c1c8e
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87760403"
---
# <a name="query-the-azure-digital-twins-twin-graph"></a>Consulta del grafo gemelo de Azure Digital Twins

En este artículo se ofrecen ejemplos y más detalles sobre el uso del [lenguaje de consultas de Azure Digital Twins](concepts-query-language.md) para consultar el [grafo de gemelos](concepts-twins-graph.md) y obtener información. Las consultas se ejecutan en el grafo con las [**API de consulta**](how-to-use-apis-sdks.md) de Azure Digital Twins.

[!INCLUDE [digital-twins-query-operations.md](../../includes/digital-twins-query-operations.md)]

En el resto de este artículo se proporcionan ejemplos de cómo usar estas operaciones.

## <a name="query-syntax"></a>Sintaxis de consulta

Esta sección incluye consultas de ejemplo que muestran la estructura del lenguaje de consulta y realizan las operaciones de consulta posibles.

Obtención de [Digital Twins](concepts-twins-graph.md) por propiedades (incluidos el identificador y los metadatos):
```sql
SELECT  * 
FROM DigitalTwins T  
WHERE T.firmwareVersion = '1.1'
AND T.$dtId in ['123', '456']
AND T.Temperature = 70
```

> [!TIP]
> El identificador de un gemelo digital se consulta con el campo de metadatos `$dtId`.

También puede obtener gemelos por sus propiedades de *etiqueta*, como se describe en [Incorporación de etiquetas a gemelos digitales](how-to-use-tags.md):
```sql
select * from digitaltwins where is_defined(tags.red) 
```

### <a name="select-top-items"></a>Selección de los elementos principales

Puede seleccionar los diversos elementos "principales" en una consulta mediante la cláusula `Select TOP`.

```sql
SELECT TOP (5)
FROM DIGITALTWINS
WHERE property = 42
```

### <a name="query-by-model"></a>Consulta por modelo

El operador `IS_OF_MODEL` se puede utilizar para filtrar en función del [modelo](concepts-models.md) del gemelo. Admite la herencia y tiene varias opciones de sobrecarga.

El uso más simple de `IS_OF_MODEL` solo toma un parámetro `twinTypeName`: `IS_OF_MODEL(twinTypeName)`.
A continuación, se muestra un ejemplo de consulta que pasa un valor en este parámetro:

```sql
SELECT * FROM DIGITALTWINS WHERE IS_OF_MODEL('dtmi:sample:thing;1')
```

Para especificar una colección gemela para buscar cuando hay más de uno (como cuando se usa `JOIN`), agregue el parámetro `twinCollection`: `IS_OF_MODEL(twinCollection, twinTypeName)`.
A continuación, se muestra un ejemplo de consulta que agrega un valor en este parámetro:

```sql
SELECT * FROM DIGITALTWINS DT WHERE IS_OF_MODEL(DT, 'dtmi:sample:thing;1')
```

Para realizar una coincidencia exacta, agregue el parámetro `exact`: `IS_OF_MODEL(twinTypeName, exact)`.
A continuación, se muestra un ejemplo de consulta que agrega un valor en este parámetro:

```sql
SELECT * FROM DIGITALTWINS WHERE IS_OF_MODEL('dtmi:sample:thing;1', exact)
```

También puede pasar los tres argumentos juntos: `IS_OF_MODEL(twinCollection, twinTypeName, exact)`.
A continuación, se muestra un ejemplo de consulta que especifica un valor para los tres parámetros:

```sql
SELECT ROOM FROM DIGITALTWINS DT WHERE IS_OF_MODEL(DT, 'dtmi:sample:thing;1', exact)
```

### <a name="query-based-on-relationships"></a>Consulta basada en relaciones

Al realizar consultas basadas en relaciones de Digital Twins, el lenguaje de consultas de Azure Digital Twins tiene una sintaxis especial.

Las relaciones se extraen en el ámbito de la consulta en la cláusula `FROM`. Una diferencia importante de los lenguajes de tipo SQL "clásico" es que cada expresión de esta cláusula `FROM` no es una tabla; en su lugar, la cláusula `FROM` expresa un recorrido de relación entre entidades y se escribe con una versión de Azure Digital Twins de `JOIN`. 

Recuerde que, con las funcionalidades del [modelo](concepts-models.md) de Azure Digital Twins, las relaciones no existen de forma independiente de los gemelos. Esto significa que la operación `JOIN` del lenguaje de consultas de Azure Digital Twins es un poco diferente de la operación `JOIN` general de SQL, ya que las relaciones aquí no se pueden consultar de forma independiente y deben estar vinculadas a un gemelo.
Para incorporar esta diferencia, se usa la palabra clave `RELATED` en la cláusula `JOIN` para hacer referencia a un conjunto de relaciones de un gemelo. 

En la siguiente sección se proporcionan varios ejemplos de lo que se ve a continuación.

> [!TIP]
> Conceptualmente, esta característica imita la funcionalidad centrada en documentos de CosmosDB, donde se puede llevar a cabo `JOIN` en los objetos secundarios dentro de un documento. CosmosDB usa la palabra clave `IN` para indicar que `JOIN` está pensado para recorrer en iteración los elementos de la matriz en el documento de contexto actual.

#### <a name="relationship-based-query-examples"></a>Ejemplos de consultas basadas en relaciones

Para obtener un conjunto de resultados que incluya relaciones, use una única instrucción `FROM` seguida de N instrucciones `JOIN`, donde las instrucciones `JOIN` expresan relaciones en el resultado de una instrucción `FROM` o `JOIN` anterior.

A continuación se muestra un ejemplo de consulta basada en relaciones. Este fragmento de código selecciona todos los gemelos digitales con una propiedad *ID* "ABC" y todos los gemelos digitales relacionados con estos gemelos digitales a través de una relación *contains*. 

```sql
SELECT T, CT
FROM DIGITALTWINS T
JOIN CT RELATED T.contains
WHERE T.$dtId = 'ABC' 
```

>[!NOTE] 
> El desarrollador no necesita poner en correlación esta operación `JOIN` con un valor de clave en la cláusula `WHERE` (ni especificar un valor de clave insertado con la definición de `JOIN`). El sistema calcula esta correlación automáticamente, ya que las propias propiedades de la relación identifican la entidad de destino.

#### <a name="query-the-properties-of-a-relationship"></a>Consulta de las propiedades de una relación

Del mismo modo que los gemelos digitales tienen propiedades que se describen a través de DTDL, las relaciones también pueden tener propiedades. El lenguaje de consultas de Azure Digital Twins permite filtrar y proyectar relaciones mediante la asignación de un alias a la relación dentro de la cláusula `JOIN`. 

Como ejemplo, considere una relación *servicedBy* que tiene una propiedad *reportedCondition*. En la consulta siguiente, a esta relación se le asigna el alias "R" para hacer referencia a su propiedad.

```sql
SELECT T, SBT, R
FROM DIGITALTWINS T
JOIN SBT RELATED T.servicedBy R
WHERE T.$dtId = 'ABC' 
AND R.reportedCondition = 'clean'
```

En el ejemplo anterior, observe cómo *reportedCondition* es una propiedad de la propia relación *servicedBy* (NO de un gemelo digital que tiene una relación *servicedBy*).

## <a name="run-queries-with-an-api-call"></a>Ejecución de consultas con una llamada a la API

Una vez que haya decidido una cadena de consulta, puede ejecutarla realizando una llamada a la **API de consulta**.
En el fragmento de código siguiente se muestra esta llamada desde la aplicación cliente:

```csharp
var client = new AzureDigitalTwinsAPIClient(<your-credentials>);
client.BaseUri = new Uri(<your-Azure-Digital-Twins-instance-URL>);

QuerySpecification spec = new QuerySpecification("SELECT * FROM digitaltwins");
QueryResult result = await client.Query.QueryTwinsAsync(spec);
```

Esta llamada devuelve los resultados de la consulta en forma de un objeto QueryResult. 

Las llamadas de consulta admiten la paginación. Este es un ejemplo completo con control de errores y paginación:

```csharp
string query = "SELECT * FROM digitaltwins";
try
{
    AsyncPageable<string> qresult = client.QueryAsync(query);
    await foreach (string item in qresult) 
    {
        // Do something with each result
    }
}
catch (RequestFailedException e)
{
    Log.Error($"Error {e.Status}: {e.Message}");
    return null;
}
```

## <a name="query-limitations"></a>Limitaciones de las consultas

Puede haber un retraso de hasta 10 segundos antes de que los cambios en la instancia se reflejen en las consultas. Por ejemplo, si completa una operación como la creación o la eliminación de gemelos con la API de DigitalTwins, es posible que el resultado no se refleje inmediatamente en las solicitudes de la API de consulta. Para que se resuelva debería bastar con esperar un poco.

Hay otras limitaciones en el uso de `JOIN` durante la versión preliminar.
* No se admiten subconsultas en la instrucción `FROM`.
* No se admite la semántica `OUTER JOIN`, lo que significa que, si la relación tiene un rango de cero, se elimina toda la "fila" del conjunto de resultados de salida.
* Durante la versión preliminar, la profundidad transversal del grafo está restringida a cinco niveles `JOIN` por consulta.
* El origen de las operaciones `JOIN` está restringido: la consulta debe declarar los gemelos donde comienza la consulta.

## <a name="query-best-practices"></a>Procedimientos recomendados sobre las consultas

A continuación se muestran algunas sugerencias para realizar consultas con Azure Digital Twins.

* Considere el patrón de consulta durante la fase de diseño del modelo. Intente asegurarse de que las relaciones que deben responderse en una sola consulta se modelan como una relación de un solo nivel.
* Diseñe las propiedades de forma que se eviten grandes conjuntos de resultados del recorrido del grafo.
* Puede reducir significativamente el número de consultas que necesita si crea una matriz de gemelos y consulta con el operador `IN`. Por ejemplo, considere un escenario en el que *Buildings* contenga a *Floors* y *Floors* contenga a *Rooms*. Para buscar las habitaciones que estén activas dentro de un edificio, puede:

    1. Buscar pisos en el edificio en función de la relación `contains`
        ```sql
        SELECT Floor
        FROM DIGITALTWINS Building
        JOIN Floor RELATED Building.contains
        WHERE Building.$dtId = @buildingId
        ``` 
    2. Para buscar las habitaciones, en lugar de tener que considerar los pisos uno por uno y ejecutar una consulta `JOIN` para buscar las habitaciones de cada uno, puede realizar consultas con una colección de los pisos del edificio (con el nombre *Floor* en la consulta siguiente).

        En la aplicación cliente:
        ```csharp
        var floors = "['floor1','floor2', ..'floorn']"; 
        ```
        En la consulta:
        ```sql
        SELECT Room
        FROM DIGITALTWINS Floor
        JOIN Room RELATED Floor.contains
        WHERE Floor.$dtId IN ['floor1','floor2', ..'floorn']
        AND Room. Temperature > 72
        AND IS_OF_MODEL(Room, 'dtmi:com:contoso:Room;1')
        ```
* Los nombres y valores de propiedad distinguen mayúsculas de minúsculas, por lo que debe tener cuidado de usar los nombres exactos definidos en los modelos. Si los nombres de propiedad están mal escritos o usan las mayúsculas de forma incorrecta, el conjunto de resultados está vacío y no se devuelven errores.


## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre las [API y los SDK de Azure Digital Twins](how-to-use-apis-sdks.md), incluida la API de consulta que se usa para ejecutar las consultas de este artículo.
