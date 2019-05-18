---
title: Trabajar con bases de datos, contenedores y los elementos de Azure Cosmos DB
description: En este artículo se describe cómo crear y usar bases de datos, los contenedores y los elementos en Azure Cosmos DB.
author: rimman
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 04/17/2019
ms.author: rimman
ms.reviewer: sngun
ms.openlocfilehash: ea3ba91859bbfb1a7c589cdb36e9fb87b52a89b8
ms.sourcegitcommit: 179918af242d52664d3274370c6fdaec6c783eb6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65560282"
---
# <a name="work-with-databases-containers-and-items-in-azure-cosmos-db"></a>Trabajar con bases de datos, contenedores y los elementos de Azure Cosmos DB

Después de crear una [cuenta de Azure Cosmos DB](account-overview.md) en su suscripción a Azure, puede administrar los datos en su cuenta mediante la creación de bases de datos, contenedores y elementos. En este artículo se describe cada una de estas entidades. 

La siguiente imagen muestra la jerarquía de entidades diferentes en una cuenta de Azure Cosmos DB:

![Entidades de cuenta de Azure Cosmos](./media/databases-containers-items/cosmos-entities.png)

## <a name="azure-cosmos-databases"></a>Bases de datos de Azure Cosmos

Puede crear una o varias bases de datos de Azure Cosmos en su cuenta. Una base de datos es análogo a un espacio de nombres. Una base de datos es la unidad de administración para un conjunto de contenedores de Azure Cosmos. En la siguiente tabla se muestra la forma de asignar una base de datos de Azure Cosmos a varias entidades específicas de API:

| Azure Cosmos entity | API DE SQL | Cassandra API | Azure Cosmos DB API para MongoDB | API de Gremlin | Table API |
| --- | --- | --- | --- | --- | --- |
|Base de datos de Azure Cosmos | Base de datos | Keyspace | Base de datos | Base de datos | N/D |

> [!NOTE]
> Con las cuentas de Table API, al crear la primera tabla, una base de datos predeterminada se crea automáticamente en su cuenta de Azure Cosmos.

### <a name="operations-on-an-azure-cosmos-database"></a>Operaciones en una base de datos de Azure Cosmos

Puede interactuar con una base de datos de Azure Cosmos con las API de Azure Cosmos tal como se describe en la tabla siguiente:

| Operación | Azure CLI | API DE SQL | Cassandra API | Azure Cosmos DB API para MongoDB | API de Gremlin | Table API |
| --- | --- | --- | --- | --- | --- | --- |
|Enumerar todas las bases de datos| Sí | Sí | Sí (la base de datos está asignada a un espacio de claves). | Sí | N/D | N/D |
|Leer la base de datos| Sí | Sí | Sí (la base de datos está asignada a un espacio de claves). | Sí | N/D | N/D |
|Crear nueva base de datos| Sí | Sí | Sí (la base de datos está asignada a un espacio de claves). | Sí | N/D | N/D |
|Actualizar la base de datos| Sí | Sí | Sí (la base de datos está asignada a un espacio de claves). | Sí | N/D | N/D |


## <a name="azure-cosmos-containers"></a>Contenedores de Azure Cosmos

Un contenedor de Azure Cosmos es la unidad de escalabilidad para almacenamiento y rendimiento aprovisionado. Un contenedor se divide de forma horizontal y luego se replica en varias regiones. Los elementos que agregue al contenedor y el rendimiento que aprovisione en él se distribuyen automáticamente en un conjunto de particiones lógicas basadas en la clave de partición. Para obtener más información sobre la creación de particiones y las claves de partición, vea [particionar datos](partition-data.md). 

Cuando se crea un contenedor de Azure Cosmos, configurar el rendimiento en uno de los siguientes modos:

* **Modo de rendimiento aprovisionado dedicado**: El rendimiento aprovisionado en un contenedor se reserva exclusivamente para ese contenedor y está respaldado por los SLA. Para obtener más información, consulte [cómo aprovisionar el rendimiento de un contenedor de Azure Cosmos](how-to-provision-container-throughput.md).

* **Modo de rendimiento aprovisionado compartido**: Estos contenedores comparten el rendimiento aprovisionado con los otros contenedores en la misma base de datos (excepto los contenedores que se han configurado con rendimiento aprovisionado dedicado). En otras palabras, el rendimiento aprovisionado en la base de datos se comparte entre todos los contenedores de "rendimiento compartido". Para obtener más información, consulte [cómo aprovisionar el rendimiento de una base de datos de Azure Cosmos](how-to-provision-database-throughput.md).

Un contenedor de Azure Cosmos puede escalar elásticamente, si crea contenedores mediante el uso de los modos compartido o dedicado rendimiento aprovisionado.

Un contenedor de Azure Cosmos es un contenedor de elementos sin esquemas. Los elementos de un contenedor pueden tener esquemas arbitrarios. Por ejemplo, un elemento que representa una persona y un elemento que representa un automóvil pueden colocarse en el *mismo contenedor*. De forma predeterminada, todos los elementos que se agregan a un contenedor se indizan automáticamente sin necesidad de administración de esquema o índice explícito. Puede personalizar el comportamiento de la indización configurando el [directiva de indexación](index-overview.md) en un contenedor. 

Puede establecer [período de vida (TTL)](time-to-live.md) en los elementos seleccionados en un contenedor de Azure Cosmos o para todo el contenedor purgar correctamente los elementos del sistema. Azure Cosmos DB elimina automáticamente los elementos cuando expiran. También garantiza que una consulta realizada en el contenedor no devuelve los elementos dentro de un límite fijo expirados. Para obtener más información, consulte [configurar TTL en el contenedor](how-to-time-to-live.md).

Puede usar [fuente de cambios](change-feed.md) para suscribirse en el registro de operaciones que se administra para cada partición lógica de su contenedor. Fuente de cambios proporciona el registro de todas las actualizaciones realizadas en el contenedor, junto con el antes y después de las imágenes de los elementos. Para obtener más información, consulte [cree aplicaciones reactivas mediante el uso de la fuente de cambios](serverless-computing-database.md). También puede configurar la duración de retención para la fuente mediante el uso de la directiva en el contenedor de fuente de cambios de cambios. 

Puede registrar [procedimientos almacenados, desencadenadores, funciones definidas por el usuario (UDF)](stored-procedures-triggers-udfs.md), y [mezcla procedimientos](how-to-manage-conflicts.md) para el contenedor de Azure Cosmos. 

Puede especificar un [restricción unique key](unique-keys.md) en el contenedor de Azure Cosmos. Al crear una directiva de clave única, garantiza la exclusividad de uno o varios valores por clave de partición lógica. Si crea un contenedor mediante el uso de una directiva de clave única, no se puede crear ningún elemento nuevo o actualizado con los valores que dupliquen los valores especificados por la restricción de clave única. Para obtener más información, consulte [Unique key constraints](unique-keys.md) (Restricciones de clave única).

Un contenedor de Azure Cosmos está especializado en las entidades específicas de la API, tal como se muestra en la tabla siguiente:

| Azure Cosmos entity | API DE SQL | Cassandra API | Azure Cosmos DB API para MongoDB | API de Gremlin | Table API |
| --- | --- | --- | --- | --- | --- |
|Contenedor de Azure Cosmos | Recolección | Tabla | Recolección | Gráfico | Tabla |

### <a name="properties-of-an-azure-cosmos-container"></a>Propiedades de un contenedor de Azure Cosmos

Un contenedor de Azure Cosmos tiene un conjunto de propiedades definido por el sistema. Dependiendo de qué API utilice, algunas propiedades podrían no exponerse directamente. En la tabla siguiente se describe la lista de propiedades definido por el sistema:

| Propiedad definida por el sistema | Configurable por el usuario o generados por el sistema | Propósito | API DE SQL | Cassandra API | Azure Cosmos DB API para MongoDB | API de Gremlin | Table API |
| --- | --- | --- | --- | --- | --- | --- | --- |
|\_id | Generado por el sistema | Identificador único del contenedor | Sí | No | No | No | No |
|\_etag | Generado por el sistema | Etiqueta de entidad que se usa para el control de simultaneidad optimista | Sí | No | No | No | No |
|\_ts | Generado por el sistema | Última actualización de la marca de tiempo del contenedor | Sí | No | No | No | No |
|\_self | Generado por el sistema | URI direccionable del contenedor | Sí | No | No | No | No |
|id | User-configurable | Nombre único que ha definido el usuario del contenedor | Sí | Sí | Sí | Sí | Sí |
|indexingPolicy | User-configurable | Proporciona la capacidad de cambiar la ruta de acceso de índice, el tipo de índice y el modo de índice | Sí | No | No | No | Sí |
|TimeToLive | User-configurable | Proporciona la capacidad de eliminar automáticamente los elementos de un contenedor tras un período de tiempo establecido. Para obtener más información, consulte [período de vida](time-to-live.md). | Sí | No | No | No | Sí |
|changeFeedPolicy | User-configurable | Se usa para leer los cambios realizados en los elementos de un contenedor. Para obtener más información, consulte [fuente de cambios](change-feed.md). | Sí | No | No | No | Sí |
|uniqueKeyPolicy | User-configurable | Se utiliza para garantizar la unicidad de uno o varios valores en una partición lógica. Para obtener más información, consulte [restricciones de clave única](unique-keys.md). | Sí | No | No | No | Sí |

### <a name="operations-on-an-azure-cosmos-container"></a>Operaciones en un contenedor de Azure Cosmos

Un contenedor de Azure Cosmos admite las siguientes operaciones cuando se usa cualquiera de las API de Cosmos Azure:

| Operación | Azure CLI | API DE SQL | Cassandra API | Azure Cosmos DB API para MongoDB | API de Gremlin | Table API |
| --- | --- | --- | --- | --- | --- | --- |
| Enumerar los contenedores en una base de datos | Sí | Sí | Sí | Sí | N/D | N/D |
| Leer un contenedor | Sí | Sí | Sí | Sí | N/D | N/D |
| Crear contenedor nuevo | Sí | Sí | Sí | Sí | N/D | N/D |
| Actualización de un contenedor | Sí | Sí | Sí | Sí | N/D | N/D |
| Eliminación de un contenedor | Sí | Sí | Sí | Sí | N/D | N/D |

## <a name="azure-cosmos-items"></a>Elementos de Azure Cosmos

Dependiendo de qué API que se utiliza, un elemento de Azure Cosmos puede representan un documento en una colección, una fila en una tabla, o un nodo o perimetral en un gráfico. En la tabla siguiente se muestra la asignación de las entidades de API específicas para un elemento de Azure Cosmos:

| Entidades de COSMOS | API DE SQL | Cassandra API | Azure Cosmos DB API para MongoDB | API de Gremlin | Table API |
| --- | --- | --- | --- | --- | --- |
|Elemento de Azure Cosmos | Documento | Fila | Documento | Nodo o perimetral | Elemento |

### <a name="properties-of-an-item"></a>Propiedades de un elemento

Cada elemento de Azure Cosmos tiene las siguientes propiedades definidas por el sistema. Dependiendo de qué API utilice, algunos de ellos podrían no exponerse directamente.

| Propiedad definida por el sistema | Configurable por el usuario o generados por el sistema| Propósito | API DE SQL | Cassandra API | Azure Cosmos DB API para MongoDB | API de Gremlin | Table API |
| --- | --- | --- | --- | --- | --- | --- | --- |
|\_id | Generado por el sistema | Identificador único del elemento | Sí | No | No | No | No |
|\_etag | Generado por el sistema | Etiqueta de entidad que se usa para el control de simultaneidad optimista | Sí | No | No | No | No |
|\_ts | Generado por el sistema | Marca de tiempo de la última actualización del elemento | Sí | No | No | No | No |
|\_self | Generado por el sistema | URI direccionable del elemento | Sí | No | No | No | No |
|id | Es posible usar el | Definido por el usuario el nombre único en una partición lógica. Si el usuario no especifica el identificador, el sistema genera automáticamente uno. | Sí | Sí | Sí | Sí | Sí |
|Propiedades arbitrarias que define el usuario | Definido por el usuario | Propiedades definidas por el usuario representadas en la representación de la API nativa (incluido JSON, BSON y CQL) | Sí | Sí | Sí | Sí | Sí |

### <a name="operations-on-items"></a>Operaciones en elementos

Elementos de Cosmos Azure admiten las siguientes operaciones. Puede usar cualquiera de las API de Azure Cosmos para realizar las operaciones.

| Operación | Azure CLI | API DE SQL | Cassandra API | Azure Cosmos DB API para MongoDB | API de Gremlin | Table API |
| --- | --- | --- | --- | --- | --- | --- |
| Insertar, reemplazar, eliminar, upsert, leer | No | Sí | Sí | Sí | Sí | Sí |

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre estos conceptos y tareas:

* [Aprovisionamiento del rendimiento en una base de datos de Azure Cosmos](how-to-provision-database-throughput.md)
* [Aprovisionamiento del rendimiento en un contenedor de Azure Cosmos](how-to-provision-container-throughput.md)
* [Trabajar con particiones lógicas](partition-data.md)
* [Configuración de TTL en un contenedor de Azure Cosmos](how-to-time-to-live.md)
* [Cree aplicaciones reactivas mediante el uso de la fuente de cambios](change-feed.md)
* [Configurar una restricción de clave única en el contenedor de Azure Cosmos](unique-keys.md)
