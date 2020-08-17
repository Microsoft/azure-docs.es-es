---
title: 'Azure Cosmos DB: API, SDK y recursos de Java asincrónico para SQL'
description: Obtenga toda la información sobre la API y el SDK de Java asincrónico para SQL, incluidas la fechas de lanzamiento, fechas de retirada y cambios realizados entre las versiones del SDK de Java asincrónico para SQL de Azure Cosmos DB.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: reference
ms.date: 05/11/2020
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: 5b656286a2958957e8518deca88ec22d7279d767
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87323866"
---
# <a name="azure-cosmos-db-async-java-sdk-for-sql-api-release-notes-and-resources"></a>SDK de Java asincrónico de Azure Cosmos DB para API de SQL: Notas de la versión y recursos
> [!div class="op_single_selector"]
> * [.NET SDK v3](sql-api-sdk-dotnet-standard.md)
> * [SDK de .NET, versión 2](sql-api-sdk-dotnet.md)
> * [SDK de .NET Core, versión 2](sql-api-sdk-dotnet-core.md)
> * [SDK de fuente de cambios de .NET, versión 2](sql-api-sdk-dotnet-changefeed.md)
> * [Node.js](sql-api-sdk-node.md)
> * [SDK de Java v4](sql-api-sdk-java-v4.md)
> * [Versión 2 del SDK de Java asincrónico](sql-api-sdk-async-java.md)
> * [SDK de Java v2 sincrónico](sql-api-sdk-java.md)
> * [Python](sql-api-sdk-python.md)
> * [REST](/rest/api
> * [Proveedor de recursos de REST](/azure/azure-resource-manager/management/azure-services-resource-providers)
> * [SQL](sql-api-query-reference.md)
> * [Bulk Executor: .NET v2](sql-api-sdk-bulk-executor-dot-net.md)
> * [Bulk Executor: Java](sql-api-sdk-bulk-executor-java.md)

El SDK de Java asincrónico de API de SQL es distinto al SDK de Java de API de SQL al proporcionar a las operaciones asincrónicas compatibilidad con la [biblioteca de Netty](https://netty.io/). El [SDK de Java de API de SQL](sql-api-sdk-java.md) no admite las operaciones asincrónicas. 

> [!IMPORTANT]  
> *No* se trata de la versión de SDK de Azure Cosmos DB para Java más reciente. Considere la posibilidad de usar el [SDK de Azure Cosmos DB para Java v4](sql-api-sdk-java-v4.md) en su proyecto. Para realizar la actualización, siga las instrucciones que se indican en la guía [Migración de la aplicación para usar el SDK de Azure Cosmos DB para Java v4](migrate-java-v4-sdk.md) y la guía de [Reactor frente a RxJava](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/reactor-rxjava-guide.md). 
>

| |  |
|---|---|
| **Descarga del SDK** | [Maven](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb) |
|**Documentación de la API** |[Documentación de referencia de API](https://docs.microsoft.com/java/api/com.microsoft.azure.cosmosdb.rx.asyncdocumentclient?view=azure-java-stable) | 
|**Contribuya al SDK** | [GitHub](https://github.com/Azure/azure-cosmosdb-java) | 
|**Introducción** | [Introducción al SDK de Java asincrónico](https://github.com/Azure-Samples/azure-cosmos-db-sql-api-async-java-getting-started) | 
|**Código de ejemplo** | [GitHub](https://github.com/Azure/azure-cosmosdb-java#usage-code-sample)| 
| **Consejos de rendimiento**| [Léame de GitHub](https://github.com/Azure/azure-cosmosdb-java#guide-for-prod)| 
| **Tiempo de ejecución mínimo admitido**|[JDK 8](/java/azure/jdk/?view=azure-java-stable) | 

[!INCLUDE [Release notes](~/azure-cosmosdb-java-v2/changelog/README.md)]
## <a name="faq"></a>Preguntas más frecuentes
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="see-also"></a>Consulte también
Para más información sobre Cosmos DB, consulte la página del servicio [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).

