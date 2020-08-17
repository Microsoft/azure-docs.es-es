---
title: Ejemplos de Node.js para administrar datos en Azure Cosmos DB
description: Busque ejemplos de Node.js en GitHub para tareas comunes en Azure Cosmos DB, incluidas las operaciones CRUD.
author: deborahc
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: sample
ms.date: 08/23/2019
ms.author: dech
ms.custom: devx-track-javascript
ms.openlocfilehash: 2c0e2b7a63f02559f95b647bc1b4ef46b8a157cd
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87421831"
---
# <a name="nodejs-examples-to-manage-data-in-azure-cosmos-db"></a>Ejemplos de Node.js para administrar datos en Azure Cosmos DB

> [!div class="op_single_selector"]
> * [Ejemplos del SDK de .NET V2](sql-api-dotnet-samples.md)
> * [Ejemplos del SDK de .NET V3](sql-api-dotnet-v3sdk-samples.md)
> * [Ejemplos del SDK de Java V4](sql-api-java-sdk-samples.md)
> * [Ejemplos de Node.js](sql-api-nodejs-samples.md)
> * [Ejemplos de Python](sql-api-python-samples.md)
> * [Galería de ejemplos de código de Azure](https://azure.microsoft.com/resources/samples/?sort=0&service=cosmos-db)
> 
> 

En el repositorio de GitHub [azure-cosmos-js](https://github.com/Azure/azure-cosmos-js/tree/master/samples) se incluyen soluciones de ejemplo que realizan operaciones CRUD y otras operaciones comunes en recursos de Azure Cosmos DB. Este artículo ofrece:

* Vínculos a las tareas de cada uno de los archivos de proyecto de ejemplo de Node.js.
* Vínculos al contenido de referencia de la API relacionada.

**Requisitos previos**

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

- Puede [activar los beneficios de suscripción a Visual Studio](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio): su suscripción a Visual Studio le proporciona créditos todos los meses que puede usar para servicios de Azure de pago.

[!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

También necesita el [SDK de JavaScript](sql-api-sdk-node.md).
   
   > [!NOTE]
   > Cada ejemplo es independiente, es decir, se configura a sí mismo y posteriormente se limpia solo. Por tanto, los ejemplos emiten varias llamadas a [Containers.create](https://docs.microsoft.com/javascript/api/%40azure/cosmos/containers?view=azure-node-latest). Cada vez que esto ocurre, se cobra a la suscripción una hora de uso de acuerdo con el nivel de rendimiento del contenedor que se va a crear.
   > 
   > 

## <a name="database-examples"></a>Ejemplos de base de datos

El archivo [DatabaseManagement ](https://github.com/Azure/azure-cosmos-js/blob/master/samples/DatabaseManagement.ts) muestra cómo realizar las operaciones CRUD en la base de datos. Para información sobre las bases de datos de Azure Cosmos antes de ejecutar los siguientes ejemplos, consulte el artículo conceptual [Uso de bases de datos, contenedores y elementos](databases-containers-items.md). 

| Tarea | Referencia de API |
| --- | --- |
| [Creación de una base de datos si no hay una](https://github.com/Azure/azure-cosmos-js/blob/master/samples/DatabaseManagement.ts#L12-L14) |[Databases.createIfNotExists](/javascript/api/@azure/cosmos/databases?view=azure-node-latest#createifnotexists-databaserequest--requestoptions-) |
| [Enumeración de las bases de datos de una cuenta](https://github.com/Azure/azure-cosmos-js/blob/master/samples/DatabaseManagement.ts#L16-L18) |[Databases.readAll](/javascript/api/@azure/cosmos/databases?view=azure-node-latest#readall-feedoptions-) |
| [Lectura de una base de datos por identificador](https://github.com/Azure/azure-cosmos-js/blob/master/samples/DatabaseManagement.ts#L20-L29) |[Database.read](/javascript/api/@azure/cosmos/database?view=azure-node-latest#read-requestoptions-) |
| [Eliminación de una base de datos](https://github.com/Azure/azure-cosmos-js/blob/master/samples/DatabaseManagement.ts#L31-L32) |[Database.delete](/javascript/api/@azure/cosmos/database?view=azure-node-latest#delete-requestoptions-) |

## <a name="container-examples"></a>Ejemplos de contenedor

El archivo [ContainerManagement](https://github.com/Azure/azure-cosmos-js/blob/master/samples/ContainerManagement.ts) muestra cómo realizar las operaciones CRUD en el contenedor. Para información sobre las colecciones de Azure Cosmos antes de ejecutar los siguientes ejemplos, consulte el artículo conceptual [Uso de bases de datos, contenedores y elementos](databases-containers-items.md). 

| Tarea | Referencia de API |
| --- | --- |
| [Creación de un contenedor si no hay uno](https://github.com/Azure/azure-cosmos-js/blob/master/samples/ContainerManagement.ts#L14-L15) |[Containers.createIfNotExists](/javascript/api/@azure/cosmos/containers?view=azure-node-latest#createifnotexists-containerrequest--requestoptions-) |
| [Enumeración de contenedores de una cuenta](https://github.com/Azure/azure-cosmos-js/blob/master/samples/ContainerManagement.ts#L17-L21) |[Containers.readAll](/javascript/api/@azure/cosmos/containers?view=azure-node-latest#readall-feedoptions-) |
| [Lectura del rendimiento de un contenedor](https://github.com/Azure/azure-cosmos-js/blob/master/samples/ContainerManagement.ts#L23-L26) |[Container.read](/javascript/api/@azure/cosmos/container?view=azure-node-latest#read-requestoptions-) |
| [Eliminación de un contenedor](https://github.com/Azure/azure-cosmos-js/blob/master/samples/ContainerManagement.ts#L28-L30) |[Container.delete](/javascript/api/@azure/cosmos/container?view=azure-node-latest#delete-requestoptions-) |

## <a name="item-examples"></a>Ejemplos de elementos

El archivo [ItemManagement](https://github.com/Azure/azure-cosmos-js/blob/master/samples/ItemManagement.ts) muestra cómo realizar las operaciones CRUD en el elemento. Para información sobre los documentos de Azure Cosmos antes de ejecutar los ejemplos siguientes, consulte el artículo conceptual [Uso de bases de datos, contenedores y elementos](databases-containers-items.md). 

| Tarea | Referencia de API |
| --- | --- |
| [Creación de elementos](https://github.com/Azure/azure-cosmos-js/blob/master/samples/ItemManagement.ts#L18-L21) |[Items.create](/javascript/api/@azure/cosmos/items?view=azure-node-latest#create-t--requestoptions-) |
| [Lectura de todos los elementos de un contenedor](https://github.com/Azure/azure-cosmos-js/blob/master/samples/ItemManagement.ts#L23-L28) |[Items.readAll](/javascript/api/@azure/cosmos/items?view=azure-node-latest#readall-feedoptions-) |
| [Lectura de un elemento por identificador](https://github.com/Azure/azure-cosmos-js/blob/master/samples/ItemManagement.ts#L30-L33) |[Item.read](/javascript/api/@azure/cosmos/item?view=azure-node-latest#read-requestoptions-) |
| [Lectura de un elemento solo si el elemento ha cambiado](https://github.com/Azure/azure-cosmos-js/blob/master/samples/ItemManagement.ts#L45-L56) |[Item.read](https://docs.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest)<br/>[RequestOptions.accessCondition](https://docs.microsoft.com/javascript/api/%40azure/cosmos/requestoptions?view=azure-node-latest#accesscondition) |
| [Consulta de documentos](https://github.com/Azure/azure-cosmos-js/blob/master/samples/ItemManagement.ts#L58-L79) |[Items.query](https://docs.microsoft.com/javascript/api/%40azure/cosmos/items?view=azure-node-latest) |
| [Reemplazo de un elemento](https://github.com/Azure/azure-cosmos-js/blob/master/samples/ItemManagement.ts#L81-L96) |[Item.replace](https://docs.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest) |
| [Reemplazo de un artículo con la comprobación de ETag condicional](https://github.com/Azure/azure-cosmos-js/blob/master/samples/ItemManagement.ts#L98-L135) |[Item.replace](https://docs.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest)<br/>[RequestOptions.accessCondition](https://docs.microsoft.com/javascript/api/%40azure/cosmos/requestoptions?view=azure-node-latest#accesscondition) |
| [Eliminación de un elemento](https://github.com/Azure/azure-cosmos-js/blob/master/samples/ItemManagement.ts#L137-L140) |[Item.delete](https://docs.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest) |

## <a name="indexing-examples"></a>Ejemplos de indización

El archivo [IndexManagement](https://github.com/Azure/azure-cosmos-js/blob/master/samples/IndexManagement.ts) muestra cómo administrar la indexación. Para información sobre la indexación en Azure Cosmos DB antes de ejecutar los ejemplos siguientes, consulte los artículos conceptuales sobre las [directivas de indexación](index-policy.md), los [tipos de indexación](index-types.md) y las [rutas de acceso de indexación](index-paths.md). 

| Tarea | Referencia de API |
| --- | --- |
| [Indexación manual de un elemento determinado](https://github.com/Azure/azure-cosmos-js/blob/master/samples/IndexManagement.ts#L52-L75) |[RequestOptions.indexingDirective: 'include'](https://docs.microsoft.com/javascript/api/%40azure/cosmos/requestoptions?view=azure-node-latest#indexingdirective) |
| [Exclusión manual de un elemento determinado del índice](https://github.com/Azure/azure-cosmos-js/blob/master/samples/IndexManagement.ts#L17-L29) |[RequestOptions.indexingDirective: 'exclude'](https://docs.microsoft.com/javascript/api/%40azure/cosmos/requestoptions?view=azure-node-latest#indexingdirective) |
| [Exclusión de una ruta de acceso del índice](https://github.com/Azure/azure-cosmos-js/blob/master/samples/IndexManagement.ts#L142-L167) |[IndexingPolicy.ExcludedPath](https://docs.microsoft.com/javascript/api/%40azure/cosmos/indexingpolicy?view=azure-node-latest#excludedpaths) |
| [Crear un índice de intervalo en una ruta de acceso de cadena](https://github.com/Azure/azure-cosmos-js/blob/master/samples/IndexManagement.ts#L87-L112) |[IndexKind.Range](https://docs.microsoft.com/javascript/api/%40azure/cosmos/indexkind?view=azure-node-latest), [IndexingPolicy](https://docs.microsoft.com/javascript/api/%40azure/cosmos/indexingpolicy?view=azure-node-latest), [Items.query](https://docs.microsoft.com/javascript/api/%40azure/cosmos/items?view=azure-node-latest) |
| [Creación de un contenedor con indexPolicy de forma predeterminada y posterior actualización en línea](https://github.com/Azure/azure-cosmos-js/blob/master/samples/IndexManagement.ts#L13-L15) |[Containers.create](https://docs.microsoft.com/javascript/api/%40azure/cosmos/containers?view=azure-node-latest)

## <a name="server-side-programming-examples"></a>Ejemplos de programación en el servidor

El archivo [index.ts](https://github.com/Azure/azure-cosmos-js/blob/master/samples/ServerSideScripts/index.ts) del proyecto [ServerSideScripts](https://github.com/Azure/azure-cosmos-js/tree/master/samples/ServerSideScripts) muestra cómo realizar las siguientes tareas. Para información sobre la programación del lado servidor en Azure Cosmos DB antes de ejecutar los ejemplos siguientes, consulte el artículo conceptual [Procedimientos almacenados, desencadenadores y funciones definidas por el usuario](stored-procedures-triggers-udfs.md). 

| Tarea | Referencia de API |
| --- | --- |
| [Creación de un procedimiento almacenado](https://github.com/Azure/azure-cosmos-js/blob/master/samples/ServerSideScripts/upsert.js) |[StoredProcedures.create](https://docs.microsoft.com/javascript/api/%40azure/cosmos/storedprocedures?view=azure-node-latest) |
| [Ejecución de un procedimiento almacenado](https://github.com/Azure/azure-cosmos-js/blob/master/samples/ServerSideScripts/index.ts) |[StoredProcedure.execute](https://docs.microsoft.com/javascript/api/%40azure/cosmos/storedprocedure?view=azure-node-latest) |

Para más información sobre la programación en el servidor, consulte [Programación en el servidor de Azure Cosmos DB: procedimientos almacenados, desencadenadores de base de datos y UDF](stored-procedures-triggers-udfs.md).

