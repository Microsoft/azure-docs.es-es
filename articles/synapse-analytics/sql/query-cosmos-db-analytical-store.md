---
title: Consulta de datos de Azure Cosmos DB mediante SQL a petición en Azure Synapse Link (versión preliminar)
description: En este artículo, aprenderá a consultar Azure Cosmos DB mediante SQL a petición en Azure Synapse Link (versión preliminar).
services: synapse analytics
author: jovanpop-msft
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 09/15/2020
ms.author: jovanpop
ms.reviewer: jrasnick
ms.openlocfilehash: c64a42c66a3b1c1810c17347e18979d599b36b6f
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932097"
---
# <a name="query-azure-cosmos-db-data-using-sql-on-demand-in-azure-synapse-link-preview"></a>Consulta de datos de Azure Cosmos DB mediante SQL a petición en Azure Synapse Link (versión preliminar)

SQL sin servidor (anteriormente SQL a petición) permite analizar los datos de los contenedores de Azure Cosmos DB que están habilitados con [Azure Synapse Link](../../cosmos-db/synapse-link.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) casi en tiempo real, sin afectar al rendimiento de las cargas de trabajo transaccionales. Ofrece una sintaxis T-SQL familiar para consultar los datos del [almacén analítico](../../cosmos-db/analytical-store-introduction.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) y la conectividad integrada en una amplia gama de herramientas de consulta ad hoc y de inteligencia empresarial a través de la interfaz de T-SQL.

> [!NOTE]
> La compatibilidad para consultar el almacén analítico de Azure Cosmos DB con SQL a petición se encuentra actualmente en versión preliminar controlada. 

Para consultar Azure Cosmos DB, se admite el área expuesta de [SELECT](/sql/t-sql/queries/select-transact-sql.md?view=sql-server-ver15&preserve-view=true) completa a través de la función [OPENROWSET](develop-openrowset.md), incluida la mayoría de las [funciones y operadores de SQL](overview-features.md). También puede almacenar los resultados de la consulta que lee los datos de Azure Cosmos DB junto con los de Azure Blob Storage o Azure Data Lake Storage mediante [create external table as select](develop-tables-cetas.md#cetas-in-sql-on-demand). Actualmente no se pueden almacenar los resultados de consultas de SQL a petición en Azure Cosmos DB mediante [CETAS](develop-tables-cetas.md#cetas-in-sql-on-demand).

En este artículo, aprenderá a escribir una consulta mediante SQL a petición que consultará datos de contenedores de Azure Cosmos DB que están habilitados para Synapse Link. Después, puede obtener más información sobre la creación de vistas SQL a petición en contenedores de Azure Cosmos DB y cómo conectarlas a modelos de Power BI en [este](./tutorial-data-analyst.md) tutorial. 

## <a name="overview"></a>Información general

Para admitir la consulta y el análisis de datos en el almacén analítico de Azure Cosmos DB, SQL a petición usa la siguiente sintaxis `OPENROWSET`:

```sql
OPENROWSET( 
       'CosmosDB',
       '<Azure Cosmos DB connection string>',
       <Container name>
    )  [ < with clause > ]
```

La cadena de conexión de Azure Cosmos DB especifica el nombre de la cuenta de Azure Cosmos DB, el nombre de la base de datos, la clave maestra de la cuenta de base de datos y un nombre de región opcional para la función `OPENROWSET`. La cadena de conexión de dispositivo tiene el formato siguiente:
```sql
'account=<database account name>;database=<database name>;region=<region name>;key=<database account master key>'
```

El nombre del contenedor de Azure Cosmos DB se especifica sin comillas con la sintaxis `OPENROWSET`. Si el nombre del contenedor tiene algún carácter especial (por ejemplo, un guion "-"), se debe encapsular dentro de `[]` (corchetes) con la sintaxis `OPENROWSET`.

> [!NOTE]
> SQL a petición no admite la consulta del almacén transaccional de Azure Cosmos DB.

## <a name="sample-data-set"></a>Conjunto de datos de ejemplo

Los ejemplos de este artículo se basan en datos de [European Centre for Disease Prevention and Control (ECDC) COVID-19 Cases](https://azure.microsoft.com/services/open-datasets/catalog/ecdc-covid-19-cases/) (Casos de COVID-19 del Centro Europeo para la Prevención y Control de Enfermedades) y [COVID-19 Open Research Dataset (CORD-19), doi:10.5281/zenodo.3715505](https://azure.microsoft.com/services/open-datasets/catalog/covid-19-open-research/). 

Puede ver la licencia y la estructura de los datos en estas páginas, y descargar los datos de ejemplo para los conjuntos de datos [ECDC](https://pandemicdatalake.blob.core.windows.net/public/curated/covid-19/ecdc_cases/latest/ecdc_cases.json) y [Cord19](https://azureopendatastorage.blob.core.windows.net/covid19temp/comm_use_subset/pdf_json/000b7d1517ceebb34e1e3e817695b6de03e2fa78.json).

Para seguir con este artículo, en el que se muestra cómo consultar datos de Cosmos DB con SQL a petición, asegúrese de que crea los recursos siguientes:
* Una cuenta de base de datos Azure Cosmos DB [habilitada para Synapse Link](../../cosmos-db/configure-synapse-link.md)
* Una base de datos de Azure Cosmos DB con el nombre `covid`
* Dos contenedores de Azure Cosmos DB denominados `EcdcCases` y `Cord19` con los conjuntos de datos de ejemplo anteriores cargados.

## <a name="explore-azure-cosmos-db-data-with-automatic-schema-inference"></a>Exploración de datos de Azure Cosmos DB con inferencia automática del esquema

La forma más fácil de explorar datos en Azure Cosmos DB consiste en aprovechar la función de inferencia automática del esquema. Al omitir la cláusula `WITH` de la instrucción `OPENROWSET`, se le indica a SQL a petición que detecte automáticamente (infiera) el esquema del almacén analítico del contenedor de Azure Cosmos DB.

```sql
SELECT TOP 10 *
FROM OPENROWSET( 
       'CosmosDB',
       'account=MyCosmosDbAccount;database=covid;region=westus2;key=C0Sm0sDbKey==',
       EcdcCases) as documents
```
En el ejemplo anterior, se indica a SQL a petición que se conecte a la base de datos `covid` en la cuenta de Azure Cosmos DB `MyCosmosDbAccount` autenticada con la clave de Azure Cosmos DB (en el ejemplo es ficticia). Después, se accede al almacén analítico del contenedor `EcdcCases` en la región `West US 2`. Como no se proyectan propiedades específicas, la función `OPENROWSET` devolverá todas las propiedades de los elementos de Azure Cosmos DB.

Si tiene que explorar datos del otro contenedor de la misma base de datos de Azure Cosmos DB, puede usar la misma cadena de conexión y hacer referencia al contenedor necesario como tercer parámetro:

```sql
SELECT TOP 10 *
FROM OPENROWSET( 
       'CosmosDB',
       'account=MyCosmosDbAccount;database=covid;region=westus2;key=C0Sm0sDbKey==',
       Cord19) as cord19
```

## <a name="explicitly-specify-schema"></a>Especificación explícita del esquema

Aunque la funcionalidad de inferencia de esquema automática en `OPENROWSET` proporciona una experiencia fácil de usar, es posible que para escenarios empresariales tenga que especificar de forma explícita el esquema para leer solo las propiedades pertinentes de los datos de Azure Cosmos DB.

`OPENROWSET` le permite especificar de forma explícita las propiedades que quiere leer de los datos del contenedor, junto a sus tipos de datos. Imagine que ha importado algunos datos del [conjunto de datos ECDC COVID](https://azure.microsoft.com/services/open-datasets/catalog/ecdc-covid-19-cases/) con la siguiente estructura en Azure Cosmos DB:

```json
{"date_rep":"2020-08-13","cases":254,"countries_and_territories":"Serbia","geo_id":"RS"}
{"date_rep":"2020-08-12","cases":235,"countries_and_territories":"Serbia","geo_id":"RS"}
{"date_rep":"2020-08-11","cases":163,"countries_and_territories":"Serbia","geo_id":"RS"}
```

Estos documentos JSON planos en Azure Cosmos DB se pueden representar como un conjunto de filas y columnas en Synapse SQL. La función `OPENROWSET` le permite especificar un subconjunto de propiedades que quiere leer y los tipos de columna exactos en la cláusula `WITH`:

```sql
SELECT TOP 10 *
FROM OPENROWSET(
      'CosmosDB',
      'account=MyCosmosDbAccount;database=covid;region=westus2;key=C0Sm0sDbKey==',
       EcdcCases
    ) with ( date_rep varchar(20), cases bigint, geo_id varchar(6) ) as rows
```

El resultado de esta consulta podría ser similar al siguiente:

| date_rep | cases | geo_id |
| --- | --- | --- |
| 2020-08-13 | 254 | RS |
| 2020-08-12 | 235 | RS |
| 2020-08-11 | 163 | RS |

Revise las [reglas de asignaciones de tipos SQL](#azure-cosmos-db-to-sql-type-mappings) al final del artículo para obtener más información sobre los tipos de SQL que se deben usar para los valores de Azure Cosmos DB.

## <a name="querying-nested-objects-and-arrays"></a>Consulta de objetos y matrices anidados

Azure Cosmos DB permite representar modelos de datos más complejos si se crean como matrices u objetos anidados. La funcionalidad de sincronización automática de Synapse Link para Azure Cosmos DB administra la representación del esquema en el almacén analítico de forma integrada, lo que incluye el control de los tipos de datos anidados, para así poder realizar consultas enriquecidas desde SQL a petición.

Por ejemplo, el conjunto de datos [CORD-19](https://azure.microsoft.com/services/open-datasets/catalog/covid-19-open-research/) tiene documentos JSON con la estructura siguiente:

```json
{
    "paper_id": <str>,                   # 40-character sha1 of the PDF
    "metadata": {
        "title": <str>,
        "authors": <array of objects>    # list of author dicts, in order
        ...
     }
     ...
}
```

Los objetos y las matrices anidados de Azure Cosmos DB se representan como cadenas JSON en el resultado de la consulta cuando los lee la función `OPENROWSET`. Una opción para leer los valores de estos tipos complejos como columnas SQL consiste en usar funciones JSON de SQL:

```sql
SELECT
    title = JSON_VALUE(metadata, '$.title'),
    authors = JSON_QUERY(metadata, '$.authors'),
    first_author_name = JSON_VALUE(metadata, '$.authors[0].first')
FROM
    OPENROWSET(
      'CosmosDB',
      'account=MyCosmosDbAccount;database=covid;region=westus2;key=C0Sm0sDbKey==',
       Cord19
    WITH ( metadata varchar(MAX) ) AS docs;
```

El resultado de esta consulta podría ser similar al siguiente:

| title | authors | first_autor_name |
| --- | --- | --- |
| Información complementaria Un estudio de epide… |   `[{"first":"Julien","last":"Mélade","suffix":"","affiliation":{"laboratory":"Centre de Recher…` | Julien |  

Como opción, también puede especificar las rutas de acceso a los valores anidados en los objetos al usar la cláusula `WITH`:

```sql
SELECT
    *
FROM
    OPENROWSET(
      'CosmosDB',
      'account=MyCosmosDbAccount;database=covid;region=westus2;key=C0Sm0sDbKey==',
       Cord19
    WITH ( title varchar(1000) '$.metadata.title',
           authors varchar(max) '$.metadata.authors'
    ) AS docs;
```

Obtenga más información sobre cómo analizar [tipos de datos complejos en Synapse Link](../how-to-analyze-complex-schema.md) y [estructuras anidadas en SQL a petición](query-parquet-nested-types.md).

> [!IMPORTANT]
> Si ve caracteres inesperados en el texto, como `MÃƒÂ©lade` en lugar de `Mélade`, la intercalación de la base de datos no está establecida en [UTF8](https://docs.microsoft.com/sql/relational-databases/collations/collation-and-unicode-support#utf8). 
> [Cambie la intercalación de la base de datos](https://docs.microsoft.com/sql/relational-databases/collations/set-or-change-the-database-collation#to-change-the-database-collation) por alguna intercalación UTF8 mediante una instrucción SQL como `ALTER DATABASE MyLdw COLLATE LATIN1_GENERAL_100_CI_AS_SC_UTF8`.


## <a name="flattening-nested-arrays"></a>Acoplamiento de matrices anidadas

Es posible que los datos de Azure Cosmos DB tengan submatrices anidadas como la matriz authors del conjunto de datos [Cord19](https://azure.microsoft.com/services/open-datasets/catalog/covid-19-open-research/):

```json
{
    "paper_id": <str>,                      # 40-character sha1 of the PDF
    "metadata": {
        "title": <str>,
        "authors": [                        # list of author dicts, in order
            {
                "first": <str>,
                "middle": <list of str>,
                "last": <str>,
                "suffix": <str>,
                "affiliation": <dict>,
                "email": <str>
            },
            ...
        ],
        ...
}
```

En algunos casos, puede que tenga que "combinar" las propiedades del elemento superior (metadata) con todos los elementos de la matriz (authors). SQL a petición permite acoplar estructuras anidadas si se aplica la función `OPENJSON` en la matriz anidada:

```sql
SELECT
    *
FROM
    OPENROWSET(
      'CosmosDB',
      'account=MyCosmosDbAccount;database=covid;region=westus2;key=C0Sm0sDbKey==',
       Cord19
    ) WITH ( title varchar(1000) '$.metadata.title',
             authors varchar(max) '$.metadata.authors' ) AS docs
      CROSS APPLY OPENJSON ( authors )
                  WITH (
                       first varchar(50),
                       last varchar(50),
                       affiliation nvarchar(max) as json
                  ) AS a
```

El resultado de esta consulta podría ser similar al siguiente:

| title | authors | first | last | affiliation |
| --- | --- | --- | --- | --- |
| Información complementaria Un estudio de epide… |   `[{"first":"Julien","last":"Mélade","suffix":"","affiliation":{"laboratory":"Centre de Recher…` | Julien | Mélade | `   {"laboratory":"Centre de Recher…` |
Información complementaria Un estudio de epide… | `[{"first":"Nicolas","last":"4#","suffix":"","affiliation":{"laboratory":"","institution":"U…` | Nicolas | N.° 4 |`{"laboratory":"","institution":"U…` | 
| Información complementaria Un estudio de epide… |   `[{"first":"Beza","last":"Ramazindrazana","suffix":"","affiliation":{"laboratory":"Centre de Recher…` | Beza | Ramazindrazana | `{"laboratory":"Centre de Recher…` |
| Información complementaria Un estudio de epide… |   `[{"first":"Olivier","last":"Flores","suffix":"","affiliation":{"laboratory":"UMR C53 CIRAD, …` | Olivier | Flores |`{"laboratory":"UMR C53 CIRAD, …` |     

> [!IMPORTANT]
> Si ve caracteres inesperados en el texto, como `MÃƒÂ©lade` en lugar de `Mélade`, la intercalación de la base de datos no está establecida en [UTF8](https://docs.microsoft.com/sql/relational-databases/collations/collation-and-unicode-support#utf8). 
> [Cambie la intercalación de la base de datos](https://docs.microsoft.com/sql/relational-databases/collations/set-or-change-the-database-collation#to-change-the-database-collation) por alguna intercalación UTF8 mediante una instrucción SQL como `ALTER DATABASE MyLdw COLLATE LATIN1_GENERAL_100_CI_AS_SC_UTF8`.

## <a name="azure-cosmos-db-to-sql-type-mappings"></a>Asignaciones de Azure Cosmos DB a tipos de SQL

Es importante indicar primero que, mientras que el almacén transaccional de Azure Cosmos DB es independiente del esquema, el almacén analítico está esquematizado para la optimización del rendimiento de las consultas analíticas. Con la funcionalidad de sincronización automática de Synapse Link, Azure Cosmos DB administra la representación del esquema en el almacén analítico de forma integrada, lo que incluye el control de los tipos de datos anidados. Dado que SQL a petición consulta el almacén analítico, es importante comprender cómo asignar tipos de datos de entrada de Azure Cosmos DB a tipos de datos de SQL.

Las cuentas de Azure Cosmos DB de SQL (Core) API admiten tipos de propiedad JSON de número, cadena, booleano, nulo, objeto anidado o matriz. Tendrá que elegir tipos SQL que coincidan con estos tipos JSON si usa la cláusula `WITH` en `OPENROWSET`. Vea los tipos de columna SQL siguientes que se deben usar para los distintos tipos de propiedad en Azure Cosmos DB.

| Tipo de propiedad de Azure Cosmos DB | Tipo de columna SQL |
| --- | --- |
| Boolean | bit |
| Entero | bigint |
| Decimal | FLOAT |
| String | varchar (intercalación de base de datos UTF8) |
| Fecha y hora (cadena con formato ISO) | varchar(30) |
| Fecha y hora (marca de tiempo de UNIX) | bigint |
| Null | `any SQL type` 
| Objeto o matriz anidados | varchar(max) (intercalación de base de datos UTF8), serializado como texto JSON |

Para consultar cuentas de Azure Cosmos DB del tipo Mongo DB API, [aquí](../../cosmos-db/analytical-store-introduction.md#analytical-schema) puede obtener más información sobre la representación de esquemas de fidelidad completa en el almacén analítico y los nombres de propiedad extendidos que se van a usar.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los siguientes artículos.

- [Procedimiento para crear y usar vistas en SQL a petición](create-use-views.md) 
- [Tutorial sobre la creación de vistas de SQL a petición en Azure Cosmos DB y su conexión a modelos de Power BI a través de DirectQuery](./tutorial-data-analyst.md)
