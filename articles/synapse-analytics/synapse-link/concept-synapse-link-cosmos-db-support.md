---
title: Características admitidas de Azure Synapse Link (versión preliminar) para Azure Cosmos DB
description: Descripción de la lista actual de acciones que admite Azure Synapse Link para Azure Cosmos DB
services: synapse-analytics
author: ArnoMicrosoft
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: synapse-link
ms.date: 09/15/2020
ms.author: acomet
ms.reviewer: jrasnick
ms.openlocfilehash: 336409b8b6f804b224b87d5fb11fded0654b8619
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90895520"
---
# <a name="azure-synapse-link-preview-for-azure-cosmos-db-supported-features"></a>Características admitidas de Azure Synapse Link (versión preliminar) para Azure Cosmos DB

En este artículo se describen las funcionalidades que se admiten actualmente en Azure Synapse Link para Azure Cosmos DB.

## <a name="azure-synapse-support"></a>Compatibilidad con Azure Synapse

Hay dos tipos de contenedores en Azure Cosmos DB:
* Contenedor de HTAP: un contenedor con Synapse Link habilitado. Este contenedor tiene almacén de transacciones y uno analítico. 
* Contenedor de OLTP: un contenedor con Synapse Link sin habilitar. Este contenedor solo tiene almacén de transacciones y no tiene uno analítico.

> [!IMPORTANT]
> Azure Synapse Link para Azure Cosmos DB es actualmente compatible con las áreas de trabajo de Synapse que no tienen habilitada la red virtual administrada. 

Puede conectarse a un contenedor de Azure Cosmos DB sin habilitar Synapse Link, en cuyo caso solo podrá leer o escribir contenido en el almacén transaccional. A continuación, se muestra la lista de las características admitidas actualmente en Synapse Link para Azure Cosmos DB. 

| Category              | Descripción |[Spark](https://docs.microsoft.com/azure/synapse-analytics/sql/on-demand-workspace-overview) | [SQL sin servidor](https://docs.microsoft.com/azure/synapse-analytics/sql/on-demand-workspace-overview) |
| -------------------- | ----------------------------------------------------------- |----------------------------------------------------------- | ----------------------------------------------------------- |
| **Compatibilidad con el tiempo de ejecución** |Tiempo de ejecución de Azure Synapse compatible para acceder a Azure Cosmos DB| ✓ | [Póngase en contacto con nosotros](mailto:cosmosdbsynapselink@microsoft.com?subject=[Enable%20Preview%20Feature]%20SQL%20serverless%20for%20Cosmos%20DB) |
| **Compatibilidad con la API de Azure Cosmos DB** | Tipo de Azure Cosmos DB API compatible | SQL/MongoDB | SQL/MongoDB |
| **Object**  |Objetos como una tabla que se puede crear y apunta directamente al contenedor de Azure Cosmos DB| Dataframe, Vista, Tabla | Ver |
| **Lectura**    | Tipo de contenedor de Azure Cosmos DB que se puede leer | OLTP/HTAP | HTAP  |
| **Escritura**   | ¿Se puede usar el tiempo de ejecución de Azure Synapse para escribir datos en un contenedor de Azure Cosmos DB? | Sí | No |

* Si escribe datos en un contenedor de Azure Cosmos DB desde Spark, esto se llevará a cabo a través del almacén transaccional de Azure Cosmos DB y afectará al rendimiento de las cargas de trabajo transaccionales de Azure Cosmos DB y consumirá unidades de solicitud.
* Actualmente no se admite la integración del grupo de Synapse SQL a través de tablas externas.

## <a name="supported-code-generated-actions-for-spark"></a>Acciones generadas por el código compatibles con Spark

| Gesto              | Descripción |OLTP |HTAP  |
| -------------------- | ----------------------------------------------------------- |----------------------------------------------------------- |----------------------------------------------------------- |
| **Cargar en DataFrame** |Permite cargar y leer datos en un DataFrame de Spark. |✓| ✓ |
| **Crear una tabla de Spark** |Permite crear una tabla que apunta a un contenedor de Azure Cosmos DB.|✓| ✓ |
| **Escribir un DataFrame en el contenedor** |Permite escribir un DataFrame en el contenedor.|✓| ✓ |
| **Cargar un DataFrame de streaming desde un contenedor** |Permite hacer streaming de datos con la fuente de cambios de Azure Cosmos DB.|✓| ✓ |
| **Escribir un DataFrame de streaming en un contenedor** |Permite hacer streaming de datos con la fuente de cambios de Azure Cosmos DB.|✓| ✓ |



## <a name="supported-code-generated-actions-for-sql-serverless"></a>Acciones generadas por el código compatibles con SQL sin servidor

| Gesto              | Descripción |OLTP |HTAP |
| -------------------- | ----------------------------------------------------------- |----------------------------------------------------------- |----------------------------------------------------------- |
| **Exploración de datos** |Exploración de datos de un contenedor con sintaxis T-SQL familiar e inferencia de esquemas automática|X| ✓ |
| **Creación de vistas y generación de informes de BI** |Creación de una vista SQL a fin de tener acceso directo a un contenedor para BI a través de Synapse SQL sin servidor |X| ✓ |
| **Combinación de orígenes de datos dispares junto con datos de Cosmos DB** | Almacenamiento de los resultados de la consulta que lee datos de contenedores de Cosmos DB junto con datos en Azure Blob Storage o Azure Data Lake Storage mediante CETAS |X| ✓ |

## <a name="next-steps"></a>Pasos siguientes

* Consulte el artículo [Conexión a Synapse Link para Azure Cosmos DB](../quickstart-connect-synapse-link-cosmos-db.md).
* [Información sobre cómo realizar consultas en el almacén analítico con Spark](how-to-query-analytical-store-spark.md)
