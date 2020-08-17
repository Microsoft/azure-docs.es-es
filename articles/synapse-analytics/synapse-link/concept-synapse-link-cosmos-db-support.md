---
title: Características admitidas de Azure Synapse Link (versión preliminar) para Azure Cosmos DB
description: Descripción de la lista actual de acciones que admite Azure Synapse Link para Azure Cosmos DB
services: synapse-analytics
author: ArnoMicrosoft
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: synapse-link
ms.date: 04/21/2020
ms.author: acomet
ms.reviewer: jrasnick
ms.openlocfilehash: 7fbc7b1cb8119a6ee9403bf0139380aa5dcd0613
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87089131"
---
# <a name="azure-synapse-link-preview-for-azure-cosmos-db-supported-features"></a>Características admitidas de Azure Synapse Link (versión preliminar) para Azure Cosmos DB

En este artículo se describen las funcionalidades que se admiten actualmente en Azure Synapse Link para Azure Cosmos DB.

## <a name="azure-synapse-support"></a>Compatibilidad con Azure Synapse

Hay dos tipos de contenedores en Azure Cosmos DB:
* Contenedor de HTAP: un contenedor con Synapse Link habilitado. Este contenedor tiene almacén de transacciones y uno analítico. 
* Contenedor de OLTP: un contenedor que solo incluye el almacén de transacciones; Synapse Link no está habilitado. 

> [!IMPORTANT]
> Azure Synapse Link para Azure Cosmos DB es actualmente compatible con las áreas de trabajo que no tienen habilitada la red virtual administrada. 

Puede conectarse a un contenedor de Azure Cosmos DB sin habilitar Synapse Link, en cuyo caso solo podrá leer o escribir contenido en el almacén transaccional. A continuación, se muestra una lista de las características admitidas actualmente en Synapse Link para Azure Cosmos DB. 

| Category              | Descripción |[Spark](https://docs.microsoft.com/azure/synapse-analytics/sql/on-demand-workspace-overview) | [SQL sin servidor](https://docs.microsoft.com/azure/synapse-analytics/sql/on-demand-workspace-overview) |
| -------------------- | ----------------------------------------------------------- |----------------------------------------------------------- | ----------------------------------------------------------- | ----------------------------------------------------------- |
| **Compatibilidad con el tiempo de ejecución** |Compatibilidad con la lectura o escritura del tiempo de ejecución de Azure Synapse| ✓ | [Póngase en contacto con nosotros](mailto:AskSynapse@microsoft.com?subject=[Enable%20Preview%20Feature]%20SQL%20serverless%20for%20Cosmos%20DB)|
| **Compatibilidad con la API de Azure Cosmos DB** |Compatibilidad con la API como Synapse Link| SQL/MongoDB | SQL/MongoDB |
| **Object**  |Objetos como una tabla que se puede crear y apunta directamente al contenedor de Azure Cosmos DB| Vista, tabla | Ver |
| **Lectura**    |Permite leer datos de un contenedor de Azure Cosmos DB.| OLTP/HTAP | HTAP  |
| **Escritura**   |Permite escribir datos desde el tiempo de ejecución en un contenedor de Azure Cosmos DB.| OLTP | N/D |

* Si escribe datos en un contenedor de Azure Cosmos DB desde Spark, este proceso se llevará a cabo a través del almacén transaccional de Azure Cosmos DB y afectará al rendimiento transaccional de Azure Cosmos DB mediante el consumo de unidades de solicitud.
* Actualmente, no se admite la integración del grupo de SQL a través de tablas externas.

## <a name="supported-code-generated-actions-for-spark"></a>Acciones generadas por el código compatibles con Spark

| Gesto              | Descripción |OLTP |HTAP  |
| -------------------- | ----------------------------------------------------------- |----------------------------------------------------------- |----------------------------------------------------------- |
| **Cargar en DataFrame** |Permite cargar y leer datos en un DataFrame de Spark. |X| ✓ |
| **Crear una tabla de Spark** |Permite crear una tabla que apunta a un contenedor de Azure Cosmos DB.|X| ✓ |
| **Escribir un DataFrame en el contenedor** |Permite escribir un DataFrame en el contenedor.|✓| ✓ |
| **Cargar un DataFrame de streaming desde un contenedor** |Permite hacer streaming de datos con la fuente de cambios de Azure Cosmos DB.|✓| ✓ |
| **Escribir un DataFrame de streaming en un contenedor** |Permite hacer streaming de datos con la fuente de cambios de Azure Cosmos DB.|✓| ✓ |



## <a name="supported-code-generated-actions-for-sql-serverless"></a>Acciones generadas por el código compatibles con SQL sin servidor

| Gesto              | Descripción |OLTP |HTAP |
| -------------------- | ----------------------------------------------------------- |----------------------------------------------------------- |----------------------------------------------------------- |
| **Seleccionar los 100 primeros** |Permite obtener una vista previa de los primeros 100 elementos de un contenedor.|X| ✓ |
| **Crear vista** |Permite crear una vista para tener acceso a BI directamente en un contenedor a través de Synapse SQL.|X| ✓ |

## <a name="next-steps"></a>Pasos siguientes

* Consulte el artículo [Conexión a Synapse Link para Azure Cosmos DB](../quickstart-connect-synapse-link-cosmos-db.md).
* [Información sobre cómo realizar consultas en el almacén analítico con Spark](how-to-query-analytical-store-spark.md)
