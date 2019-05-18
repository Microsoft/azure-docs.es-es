---
title: SDK y recursos de .NET Standard para Table API de Azure Cosmos DB
description: Obtenga toda la información sobre Table API de Azure Cosmos DB y el SDK de .NET Standard, incluidas la fechas de lanzamiento, las fechas de retirada y los cambios realizados entre las versiones.
author: wmengmsft
ms.author: wmeng
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: dotnet
ms.topic: reference
ms.date: 10/18/2018
ms.openlocfilehash: 8e8baf9ba8fca2966825620d0588ab005abd37b9
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65604386"
---
# <a name="azure-cosmos-db-table-net-standard-api-download-and-release-notes"></a>Table API de .NET Standard de Azure Cosmos DB: descarga y notas de la versión
> [!div class="op_single_selector"]
> 
> * [.NET](table-sdk-dotnet.md)
> * [.NET Standard](table-sdk-dotnet-standard.md)
> * [Java](table-sdk-java.md)
> * [Node.js](table-sdk-nodejs.md)
> * [Python](table-sdk-python.md)

|   |   |
|---|---|
|**Descarga del SDK**|[NuGet](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table)|
|**Ejemplo**|[Ejemplo de COSMOS DB Table API de .NET](https://github.com/Azure-Samples/azure-cosmos-table-dotnet-core-getting-started)|
|**Guía de inicio rápido**|[Guía de inicio rápido](create-table-dotnet.md)|
|**Tutorial**|[Tutorial](tutorial-develop-table-dotnet.md)|
|**Plataforma admitida actualmente**|[Microsoft .NET Standard 2.0](https://www.nuget.org/packages/NETStandard.Library)|

## <a name="release-notes"></a>Notas de la versión

### <a name="a-name101101"></a><a name="1.0.1"/>1.0.1
* Corrección de errores

### <a name="a-name100100"></a><a name="1.0.0"/>1.0.0
* Versión de disponibilidad general

### <a name="a-name0110-preview0110-preview"></a><a name="0.11.0-preview"/>0.11.0-Preview
* Se realizaron cambios en cómo se puede configurar CloudTableClient. Ahora toma una un objeto TableClientConfiguration durante la construcción. TableClientConfiguration proporciona diferentes propiedades para configurar el comportamiento del cliente dependiendo de si el extremo de destino es Cosmos DB Table API o Table API de Azure Storage.
* Se agregó compatibilidad para TableQuery para devolver resultados en el criterio de ordenación en una columna personalizada. Esta característica solo se admite en los puntos de conexión de Cosmos DB Table.
* Se agregó compatibilidad para exponer RequestCharges en varios tipos de resultados. Esta característica solo se admite en los puntos de conexión de Cosmos DB Table.

### <a name="a-name0101-preview0101-preview"></a><a name="0.10.1-preview"/>0.10.1-preview
* Agregue compatibilidad con el token de SAS, las operaciones de TablePermissions, ServiceProperties y ServiceStats sobre los puntos de conexión de Azure Storage. 
   > [!NOTE]
   > Algunas funcionalidades de los SDK anteriores de tabla de Azure Storage todavía no se admiten, como el cifrado del lado cliente.

### <a name="a-name0100-preview0100-preview"></a><a name="0.10.0-preview"/>0.10.0 (versión preliminar)
* Se ha agregado compatibilidad con operaciones básicas de CRUD, por lotes y consultas sobre los puntos de conexión de tabla de Azure Storage. 
   > [!NOTE]
   > Algunas funcionalidades de los SDK anteriores de tabla de Azure Storage todavía no se admiten, como el cifrado del lado cliente.

### <a name="a-name091-preview091-preview"></a><a name="0.9.1-preview"/>0.9.1 (versión preliminar)
* El SDK de .NET Standard para Table API de Azure Cosmos DB es una biblioteca de .NET multiplataforma que ofrece acceso eficaz al modelo de datos de tabla en Cosmos DB. Esta versión inicial es compatible con el conjunto completo de CRUD de tabla y entidad + funcionalidades de consulta con API similares como el [SDK de Table API de Cosmos DB para .NET Framework](table-sdk-dotnet.md). 
   > [!NOTE]
   >  Los puntos de conexión de tabla de Azure Storage aún no son compatibles con la versión preliminar de 0.9.1.

## <a name="release-and-retirement-dates"></a>Fechas de lanzamiento y de retirada
Microsoft notifica la retirada de un SDK con al menos **12 meses** de antelación para facilitar la transición a una versión compatible o más reciente.

Esta biblioteca estándar de .NET multiplataforma [Microsoft.Azure.Cosmos.Table](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table) reemplazará a la biblioteca de .NET Framework [Microsoft.Azure.CosmosDB.Table](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.Table).

| Version | Fecha de lanzamiento | Fecha de retirada |
| --- | --- | --- |
| [1.0.1](#1.0.1) |19 de abril de 2019 |--- |
| [1.0.0](#1.0.0) |13 de marzo de 2019 |--- |
| [0.11.0-preview](#0.11.0-preview) |5 de marzo de 2019 |--- |
| [0.10.1-preview](#0.10.1-preview) |22 de enero de 2019 |--- |
| [0.10.0 (versión preliminar)](#0.10.0-preview) |18 de diciembre de 2018 |--- |
| [0.9.1 (versión preliminar)](#0.9.1-preview) |18 de octubre de 2018 |--- |


## <a name="faq"></a>Preguntas más frecuentes

[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="see-also"></a>Vea también
Para obtener más información sobre Table API de Azure Cosmos DB, consulte [Introducción a Table API de Azure Cosmos DB](table-introduction.md). 
