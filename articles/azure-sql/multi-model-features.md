---
title: Funcionalidades de varios modelos
description: Microsoft Azure SQL permite trabajar con varios modelos de datos en la misma base de datos.
services: sql-database
ms.service: sql-db-mi
ms.subservice: features
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: ''
ms.date: 12/17/2018
ms.openlocfilehash: 59a709a206eb29b875272674ee19e414023cc37f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87073309"
---
# <a name="multi-model-capabilities-of-azure-sql-database--sql-managed-instance"></a>Funcionalidades de varios modelos de Azure SQL Database e Instancia administrada de SQL
[!INCLUDE[appliesto-sqldb-sqlmi](includes/appliesto-sqldb-sqlmi.md)]

Las bases de datos de varios modelos permiten almacenar y trabajar con datos representados en varios formatos como datos relacionales, grafos, documentos JSON o XML, pares clave-valor, etc.

## <a name="when-to-use-multi-model-capabilities"></a>Cuándo usar las funcionalidades multimodelo

La [familia de productos de Azure SQL](azure-sql-iaas-vs-paas-what-is-overview.md) está diseñada para trabajar con el modelo relacional que proporciona el mejor rendimiento en la mayoría de los casos para diversas aplicaciones de uso general. Sin embargo, la familia de productos de Azure SQL no se limita únicamente a los datos relacionales. La familia de productos de Azure SQL permite usar una variedad de formatos no relacionales que están estrechamente integrados en el modelo relacional.
Considere la posibilidad de usar funcionalidades de varios modelos de la familia de productos de Azure SQL en los casos siguientes:

- Tiene cierta información o estructuras que se ajustan mejor a para los modelos de NoSQL y no quiere usar una base de datos de NoSQL independiente.
- La mayoría de los datos es adecuada para el modelo relacional y necesita modelar algunas partes de los datos en el estilo de NoSQL.
- Quiere aprovechar el amplio lenguaje Transact-SQL para consultar y analizar datos relacionales y NoSQL, e integrarlos con una variedad de herramientas y aplicaciones que pueden usar el lenguaje SQL.
- Quiere aplicar características de base de datos, como [tecnologías en memoria](in-memory-oltp-overview.md) para mejorar el rendimiento del análisis o el procesamiento de las estructuras de datos NoSQL, usar la [replicación transaccional](managed-instance/replication-transactional-overview.md) o [réplicas legibles](database/read-scale-out.md) para crear la copia de los datos en el otro lugar y descargar algunas cargas de trabajo de análisis desde la base de datos principal.

## <a name="overview"></a>Información general

La familia de productos de Azure SQL proporciona las siguientes características de varios modelos:

- Las [características de Graph](#graph-features) permiten representar los datos como un conjunto de nodos y bordes, además de usar las consultas de Transact-SQL estándares mejoradas con el operador `MATCH` de grafos para consultar los datos del grafo.
- Las [características de JSON](#json-features) permiten colocar documentos JSON en tablas, transformar datos relacionales para documentos JSON y viceversa. Puede usar el lenguaje Transact-SQL estándar mejorado con las funciones JSON para analizar documentos y usar índices no agrupados, índices de almacén de columnas o tablas optimizadas para memoria, a fin de optimizar las consultas.
- Las [características espaciales](#spatial-features) permiten almacenar datos geográficos y geométricos, indexarlos con los índices espaciales y recuperar los datos mediante consultas espaciales.
- Las [características XML](#xml-features) permiten almacenar e indexar datos XML en la base de datos y utilizar operaciones XQuery y XPath nativas para trabajar con datos XML. La familia de productos de Azure SQL tiene un motor de consultas XML especializado integrado que procesa datos XML.
- Los [pares clave-valor](#key-value-pairs) no se admiten de forma explícita como características especiales, ya que se pueden modelar de forma nativa como tablas de dos columnas.

  > [!Note]
  > Puede usar la expresión de ruta de acceso JSON, las expresiones XQuery y XPath, las funciones espaciales y las expresiones de consulta de grafos en la misma consulta de Transact-SQL para acceder a los datos almacenados en la base de datos. Además, cualquier herramienta o lenguaje de programación que puede ejecutar consultas de Transact-SQL, también puede usar esa interfaz de consulta para obtener acceso a datos de varios modelos. Esta es la diferencia clave en comparación con las bases de datos de varios modelos como [Azure Cosmos DB](/azure/cosmos-db/) que proporciona una API especializada para diferentes modelos de datos.

En las secciones siguientes, aprenderá las funcionalidades de varios modelos más importantes de la familia de productos de Azure SQL.

## <a name="graph-features"></a>Características de grafos

La familia de productos de Azure SQL ofrece funcionalidades de base de datos de grafos para modelar las relaciones de varios a varios en la base de datos. Un grafo es una colección de nodos (o vértices) y bordes (o relaciones). Un nodo representa una entidad (por ejemplo, una persona o una organización) y un borde representa una relación entre los dos nodos que conecta (por ejemplo, "Me gusta" o amigos). Estas son algunas características que hacen que una base de datos de grafos sea única:

- Los bordes o las relaciones son entidades de primera clase en una base de datos de grafos y pueden tener atributos o propiedades asociados a ellas.
- Un solo borde puede conectar flexiblemente varios nodos en una base de datos de grafos.
- Puede expresar fácilmente coincidencias de patrones y consultas de navegación en saltos múltiples.
- Puede expresar fácilmente consultas polimórficas y cierres transitivos.

Las [relaciones de grafos y funcionalidades de consulta de grafos](/sql/relational-databases/graphs/sql-graph-overview) están integradas en Transact-SQL y reciben las ventajas de usar el motor de base de datos de SQL Server como sistema fundamental de administración de base de datos.

### <a name="when-to-use-a-graph-capability"></a>Cuándo usar una funcionalidad de grafo

No hay nada que una base de datos de grafos puede conseguir, que no se puede lograr mediante una base de datos relacional. Sin embargo, una base de datos de grafos puede facilitar la expresión de determinadas consultas. La decisión para elegir una u otra puede basarse en los siguientes factores:

- Datos jerárquicos del modelo donde un nodo puede tener varios elementos primarios, por lo que no se puede usar HierarchyId.
- La aplicación tiene relaciones complejas de varios a varios; a medida que la aplicación evoluciona, se agregan nuevas relaciones.
- Necesita analizar las relaciones y los datos interconectados.

## <a name="json-features"></a>Característica de JSON

La familia de productos de Azure SQL permite analizar y consultar datos representados en formato de notación de objetos JavaScript [(JSON)](https://www.json.org/), y exportar los datos relacionales como texto JSON.

JSON es un formato de datos conocido que se usa para intercambiar datos en aplicaciones web y móviles modernas. JSON también se utiliza para almacenar datos semiestructurados en archivos de registro o en bases de datos NoSQL como [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/). Muchos servicios web REST devuelven resultados con formato de texto JSON o bien aceptan datos con este formato. La mayoría de los servicios de Azure, como [Azure Cognitive Search](https://azure.microsoft.com/services/search/), [Azure Storage](https://azure.microsoft.com/services/storage/) y [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/), tienen puntos de conexión REST que devuelven o consumen JSON.


La familia de productos de Azure SQL le permite trabajar fácilmente con datos JSON e integrar su base de datos con servicios modernos; además, proporciona las siguientes funciones para trabajar con datos JSON:

![Funciones JSON](./media/multi-model-features/image_1.png)

Si tiene texto JSON, puede extraer datos de JSON o comprobar que el formato JSON sea correcto con las funciones integradas [JSON_VALUE](/sql/t-sql/functions/json-value-transact-sql), [JSON_QUERY](/sql/t-sql/functions/json-query-transact-sql) y [ISJSON](/sql/t-sql/functions/isjson-transact-sql). La función [JSON_MODIFY](/sql/t-sql/functions/json-modify-transact-sql) le permite actualizar valores dentro del texto JSON. Para consultas y análisis más avanzados, la función [OPENJSON](/sql/t-sql/functions/openjson-transact-sql) puede transformar una matriz de objetos JSON en un conjunto de filas. Se puede ejecutar cualquier consulta SQL en el conjunto de resultados devuelto. Por último, hay una cláusula [FOR JSON](/sql/relational-databases/json/format-query-results-as-json-with-for-json-sql-server) que permite dar formato de texto JSON a los datos almacenados en las tablas relacionales.

Para más información, consulte [Cómo trabajar con datos JSON](database/json-features.md).
[JSON](/sql/relational-databases/json/json-data-sql-server) es una característica principal del motor de base de datos de SQL Server.

### <a name="when-to-use-a-json-capability"></a>Cuándo usar una funcionalidad de JSON

Los modelos de documento se pueden utilizar en lugar de los modelos relacionales en algunos escenarios concretos:

- La alta normalización del esquema no ofrece ventajas significativas, ya que accede a todos los campos de objetos a la vez o nunca actualiza las partes normalizadas de los objetos. Sin embargo, el modelo normalizado aumenta la complejidad de las consultas debido al gran número de tablas que se deben unir para obtener los datos.
- Está trabajando con las aplicaciones que usan de forma nativa los documentos de JSON como modelos de datos o comunicación y no desea introducir capas adicionales que transforman los datos relacionales en JSON y viceversa.
- Se necesita simplificar el modelo de datos anulando la normalización de las tablas secundarias o los patrones de valor de objeto de entidad.
- Deberá cargar o exportar los datos almacenados en formato JSON sin alguna herramienta adicional que analice los datos.

## <a name="spatial-features"></a>Características espaciales

Los datos espaciales representan información sobre la ubicación física y la forma de objetos geométricos. Estos objetos pueden ser ubicaciones de punto u objetos más complejos, como países o regiones, carreteras o lagos.

 Los dos tipos de datos espaciales admitidos son: 

- El tipo geométrico representa los datos en un sistema de coordenadas euclidiano (plano).
- El tipo geográfico representa los datos en un sistema de coordenadas elipsoidales.

Hay una serie de objetos espaciales que se pueden usar en la familia de productos de Azure SQL, que permiten analizar y consultar los datos representados en formato de notación de objetos JavaScript [(JSON)](https://www.json.org/) y exportar los datos relacionales como texto JSON.
Por ejemplo, [Point](/sql/relational-databases/spatial/point), [LineString](/sql/relational-databases/spatial/linestring), [Polygon](/sql/relational-databases/spatial/polygon), etc.

La familia de productos de Azure SQL proporciona también [índices espaciales](/sql/relational-databases/spatial/spatial-indexes-overview) especializados que se pueden usar para mejorar el rendimiento de las consultas espaciales.

La [compatibilidad espacial](/sql/relational-databases/spatial/spatial-data-sql-server) es una característica fundamental del motor de base de datos de SQL Server.

## <a name="xml-features"></a>Características de XML

El motor de base de datos de SQL Server proporciona una plataforma eficaz para desarrollar aplicaciones completas para la administración de datos semiestructurados. La compatibilidad con XML está integrada en todos los componentes del motor de base de datos e incluye lo siguiente:

- El tipo de datos XML. Los valores XML se pueden almacenar de forma nativa en una columna de tipo de datos xml cuyo tipo se puede determinar según una colección de esquemas XML, o se puede dejar sin tipo. Puede indexar la columna XML.
- La capacidad de especificar una consulta con datos XQuery almacenados en columnas y variables del tipo xml. Las funcionalidades de XQuery pueden usarse en cualquier consulta de Transact-SQL que tiene acceso a cualquier modelo de datos que se utiliza en la base de datos.
- Indexe automáticamente todos los elementos en documentos XML mediante el [índice XML principal](/sql/relational-databases/xml/xml-indexes-sql-server#primary-xml-index) o especifique las rutas de acceso exactas que se deben indexar mediante el [índice XML secundario](/sql/relational-databases/xml/xml-indexes-sql-server#secondary-xml-indexes).
- OPENROWSET que permite la carga masiva de datos XML.
- Transforme datos relacionales en formato XML.

[XML](/sql/relational-databases/xml/xml-data-sql-server) es una característica fundamental del motor de base de datos de SQL Server.

### <a name="when-to-use-an-xml-capability"></a>Cuándo usar una funcionalidad de XML

Los modelos de documento se pueden utilizar en lugar de los modelos relacionales en algunos escenarios concretos:

- La alta normalización del esquema no ofrece ventajas significativas, ya que accede a todos los campos de objetos a la vez o nunca actualiza las partes normalizadas de los objetos. Sin embargo, el modelo normalizado aumenta la complejidad de las consultas debido al gran número de tablas que se deben unir para obtener los datos.
- Está trabajando con las aplicaciones que usan de forma nativa los documentos de XML como modelos de datos o comunicación y no desea introducir capas adicionales que transforman los datos relacionales en XML y viceversa.
- Se necesita simplificar el modelo de datos anulando la normalización de las tablas secundarias o los patrones de valor de objeto de entidad.
- Deberá cargar o exportar los datos almacenados en formato XML sin alguna herramienta adicional que analice los datos.

## <a name="key-value-pairs"></a>Pares de clave-valor

La familia de productos de Azure SQL no tiene tipos ni estructuras especializados que admitan pares de clave-valor, puesto que las estructuras de clave-valor pueden representarse de forma nativa como tablas relacionales estándar:

```sql
CREATE TABLE Collection (
  Id int identity primary key,
  Data nvarchar(max)
)
```

Puede personalizar esta estructura clave-valor para satisfacer sus necesidades sin restricciones. Por ejemplo, el valor puede ser el documento XML en lugar del tipo `nvarchar(max)`, si el valor es el documento JSON, puede poner la restricción `CHECK` que comprueba la validez del contenido JSON. Puede colocar cualquier número de valores relacionados con una clave en las columnas adicionales, agregar columnas calculadas e índices para simplificar y optimizar el acceso a datos, definir la tabla como tabla de solo esquema optimizado y de memoria para obtener un mejor rendimiento, etc.

Consulte [Cómo BWin usa OLTP en memoria para lograr un rendimiento y escala sin precedentes](https://blogs.msdn.microsoft.com/sqlcat/20../../how-bwin-is-using-sql-server-2016-in-memory-oltp-to-achieve-unprecedented-performance-and-scale/) para su solución de almacenamiento en caché de ASP.NET que logró 1.200.000 lotes por segundo, como ejemplo de cómo un modelo relacional se puede usar eficazmente como solución de par clave-valor en la práctica.

## <a name="next-steps"></a>Pasos siguientes

Las funcionalidades de varios modelos de la familia de productos de Azure SQL también son las características fundamentales del motor de base de datos de SQL Server que se comparten entre la familia de productos de Azure SQL. Para obtener más información acerca de estas características, visite las páginas de documentación de base de datos relacional de SQL:

- [Procesamiento de Graph](/sql/relational-databases/graphs/sql-graph-overview)
- [Datos de JSON](/sql/relational-databases/json/json-data-sql-server)
- [Soporte espacial](/sql/relational-databases/spatial/spatial-data-sql-server)
- [Datos XML](/sql/relational-databases/xml/xml-data-sql-server)
