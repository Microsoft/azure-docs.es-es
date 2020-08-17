---
title: Ejemplos de Python para la API de SQL para Azure Cosmos DB
description: Busque ejemplos de Python en GitHub para tareas comunes en Azure Cosmos DB, incluidas las operaciones CRUD.
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: python
ms.topic: sample
ms.date: 05/20/2020
ms.author: sngun
ms.custom: devx-track-python
ms.openlocfilehash: 6602d757e210837aadadd431b27bb7cf76f06715
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87873354"
---
# <a name="azure-cosmos-db-python-examples"></a>Ejemplos de Python para Azure Cosmos DB

> [!div class="op_single_selector"]
> * [Ejemplos del SDK de .NET V2](sql-api-dotnet-samples.md)
> * [Ejemplos del SDK de .NET V3](sql-api-dotnet-v3sdk-samples.md)
> * [Ejemplos del SDK de Java V4](sql-api-java-sdk-samples.md)
> * [Ejemplos de Node.js](sql-api-nodejs-samples.md)
> * [Ejemplos de Python](sql-api-python-samples.md)
> * [Galería de ejemplos de código de Azure](https://azure.microsoft.com/resources/samples/?sort=0&service=cosmos-db)

En el repositorio de GitHub [azure-documentdb-python](https://github.com/Azure/azure-documentdb-python) se incluyen soluciones de ejemplo que realizan operaciones CRUD y otras operaciones comunes sobre recursos de Azure Cosmos DB. Este artículo ofrece:

* Vínculos a las tareas de cada uno de los archivos de proyecto de ejemplo de Python.
* Vínculos al contenido de referencia de la API relacionada.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

- Puede [activar los beneficios de suscripción a Visual Studio](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio): su suscripción a Visual Studio le proporciona créditos todos los meses que puede usar para servicios de Azure de pago.

[!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

También necesita el [SDK de Python](sql-api-sdk-python.md).

   > [!NOTE]
   > Cada ejemplo es independiente, es decir, se configura a sí mismo y posteriormente se limpia solo. Los ejemplos emiten varias llamadas a `CosmosClient.CreateContainer`. Cada vez que lo haga, se le facturará una hora de uso a su suscripción. Para más información sobre la facturación de Azure Cosmos DB, consulte los [precios de Azure Cosmos DB](https://azure.microsoft.com/pricing/details/cosmos-db/).

## <a name="database-examples"></a>Ejemplos de base de datos

En el ejemplo [database_management.py](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/database_management.py) de Python, se muestra cómo realizar las siguientes tareas. Para información sobre las bases de datos de Azure Cosmos antes de ejecutar los siguientes ejemplos, consulte el artículo conceptual [Uso de bases de datos, contenedores y elementos](databases-containers-items.md).

| Tarea | Referencia de API |
| --- | --- |
| [Creación de una base de datos](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/database_management.py#L48-L56) |CosmosClient.create_database|
| [Lectura de una base de datos por identificador](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/database_management.py#L59-L67) |CosmosClient.get_database_client|
| [Enumeración de las bases de datos](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/database_management.py#L32-L67) |CosmosClient.query_databases|
| [Enumeración de las bases de datos de una cuenta](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/database_management.py#L70-L81) |CosmosClient.list_databases|
| [Eliminación de una base de datos](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/database_management.py#L84-L93) |CosmosClient.delete_database|

## <a name="container-examples"></a>Ejemplos de contenedor

En el ejemplo [container_management.py](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/container_management.py) de Python, se muestra cómo realizar las siguientes tareas. Para información sobre las colecciones de Azure Cosmos antes de ejecutar los siguientes ejemplos, consulte el artículo conceptual [Uso de bases de datos, contenedores y elementos](databases-containers-items.md).

| Tarea | Referencia de API |
| --- | --- |
| [Búsqueda de un contenedor](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/container_management.py#L51-L66) |database.query_containers |
| [Creación de un contenedor](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/container_management.py#L69-L163) |database.create_container |
| [Lectura de todos los contenedores de una base de datos](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/container_management.py#L206-L217) |database.list_containers |
| [Obtención de un contenedor por su identificador](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/container_management.py#L195-L203) |database.get_container_client |
| [Administración del rendimiento aprovisionado del contenedor](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/container_management.py#L166-L192) |container.read_offer, container.replace_throughput|
| [Eliminación de un contenedor](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/container_management.py#L220-L229) |database.delete_container |

## <a name="item-examples"></a>Ejemplos de elementos

En el ejemplo [item_management.py](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/document_management.py) de Python, se muestra cómo realizar las siguientes tareas. Para información sobre los documentos de Azure Cosmos antes de ejecutar los ejemplos siguientes, consulte el artículo conceptual [Uso de bases de datos, contenedores y elementos](databases-containers-items.md).

| Tarea | Referencia de API |
| --- | --- |
| [Creación de los elementos de un contenedor](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/document_management.py#L26-L38) |container.create_item |
| [Lectura de un elemento por su identificador](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/document_management.py#L41-L49) |container.read_item |
| [Lectura de todos los elementos de un contenedor](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/document_management.py#L52-L63) |container.read_all_items |
| [Consulta de un elemento por su identificador](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/document_management.py#L66-L78) |container.query_items |
| [Reemplazo de un elemento](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/document_management.py#L81-L88) |container.replace_items |
| [Realización de una operación upsert en un elemento](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/document_management.py#L91-L98) |container.upsert_item |
| [Eliminación de un elemento](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/document_management.py#L101-L106) |container.delete_item |
| [Obtención de la fuente de cambios de los elementos de un contenedor](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/change_feed_management.py) |container.query_items_change_feed |

## <a name="indexing-examples"></a>Ejemplos de indización

En el ejemplo [index_management.py](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/index_management.py) de Python, se muestra cómo realizar las siguientes tareas. Para información sobre la indexación en Azure Cosmos DB antes de ejecutar los ejemplos siguientes, consulte los artículos conceptuales sobre las [directivas de indexación](index-policy.md), los [tipos de indexación](index-types.md) y las [rutas de acceso de indexación](index-paths.md).

| Tarea | Referencia de API |
| --- | --- |
| [Exclusión de un elemento determinado para la indexación](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/index_management.py#L145-L201) | documents.IndexingDirective.Exclude|
| [Indexación manual con elementos específicos indexados](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/index_management.py#L204-L263) | documents.IndexingDirective.Include |
| [Exclusión de rutas de acceso de la indexación](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/index_management.py#L266-L336) |Definición de las rutas de acceso que se van a excluir en la propiedad `IndexingPolicy` |
| [Uso de índices de intervalo en cadenas](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/index_management.py#L401-L485) | Defina la directiva de indexación con índices de intervalo en el tipo de datos de cadena. `'kind': documents.IndexKind.Range`, `'dataType': documents.DataType.String`|
| [Realización de una transformación de índices](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/index_management.py#L488-L544) |database.replace_container (use la directiva de indexación actualizada)|
| [Uso de exámenes cuando solo existe un índice de hash en la ruta de acceso](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/index_management.py#L339-L398) | Establecimiento de `enable_scan_in_query=True` y `enable_cross_partition_query=True` al consultar los elementos |
