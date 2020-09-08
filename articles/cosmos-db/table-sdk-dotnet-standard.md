---
title: SDK y recursos de .NET Standard para Table API de Azure Cosmos DB
description: Obtenga toda la información sobre Table API de Azure Cosmos DB y el SDK de .NET Standard, incluidas la fechas de lanzamiento, las fechas de retirada y los cambios realizados entre las versiones.
author: sakash279
ms.author: akshanka
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: dotnet
ms.topic: reference
ms.date: 03/18/2019
ms.custom: devx-track-dotnet
ms.openlocfilehash: bf8563274d7aa677249335612d0156d6a5ecbd95
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018457"
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
|**Ejemplo**|[Ejemplo de Table API de Cosmos DB para .NET](https://github.com/Azure-Samples/azure-cosmos-table-dotnet-core-getting-started)|
|**Guía de inicio rápido**|[Guía de inicio rápido](create-table-dotnet.md)|
|**Tutorial**|[Tutorial](tutorial-develop-table-dotnet.md)|
|**Plataforma admitida actualmente**|[Microsoft .NET Standard 2.0](https://www.nuget.org/packages/NETStandard.Library)|
|**Informar sobre un problema**|[Informar sobre un problema](https://github.com/Azure/azure-cosmos-table-dotnet/issues)|

## <a name="release-notes-for-200-series"></a>Notas de la versión para la serie 2.0.0
La serie 2.0.0 toma la dependencia de [Microsoft.Azure.Cosmos](https://www.nuget.org/packages/Microsoft.Azure.Cosmos/), con mejoras en el rendimiento y la consolidación de los espacios de nombres en el punto de conexión de Cosmos DB.

### <a name="200-preview"></a><a name="2.0.0-preview"></a>2.0.0-preview
* Versión preliminar inicial del SDK de Table 2.0.0 que toma la dependencia de [Microsoft.Azure.Cosmos](https://www.nuget.org/packages/Microsoft.Azure.Cosmos/), con mejoras en el rendimiento y la consolidación de los espacios de nombres en el punto de conexión de Cosmos DB. La API pública sigue siendo la misma.

## <a name="release-notes-for-100-series"></a>Notas de la versión de la serie 1.0.0
La serie 1.0.0 toma la dependencia de [Microsoft.Azure.DocumentDB.Core](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB.Core/).

### <a name="108"></a><a name="1.0.8"></a>1.0.8
* Agrega compatibilidad para establecer la propiedad TTL si es el punto de conexión de cosmosdb. 
* Respeta la directiva de reintentos en caso de tiempo de espera y excepción de tarea cancelada.
* Corrige la excepción de tarea intermitente cancelada en aplicaciones ASP .NET.
* Corrige el modo de ubicación de recuperación de Azure Table Storage desde el punto de conexión secundario.
* Actualiza la versión de la dependencia de `Microsoft.Azure.DocumentDB.Core` a 2.11.2, lo que corrige la excepción de referencia nula intermitente.
* Actualiza la versión de la dependencia de `Odata.Core` a 7.6.4, lo que corrige el conflicto de compatibilidad con Azure Shell.

### <a name="107"></a><a name="1.0.7"></a>1.0.7
* Mejora del rendimiento al establecer el nivel de seguimiento predeterminado de SDK de Table en SourceLevels.Off, aplicable mediante app.config.

### <a name="105"></a><a name="1.0.5"></a>1.0.5
* Presenta una nueva configuración en TableClientConfiguration para usar el ejecutor de REST para comunicarse con Table API de Cosmos DB

### <a name="105-preview"></a><a name="1.0.5-preview"></a>1.0.5-preview
* Corrección de errores

### <a name="104"></a><a name="1.0.4"></a>1.0.4
* Corrección de errores
* Proporcione la opción HttpClientTimeout para RestExecutorConfiguration.

### <a name="104-preview"></a><a name="1.0.4-preview"></a>1.0.4-preview
* Corrección de errores
* Proporcione la opción HttpClientTimeout para RestExecutorConfiguration.

### <a name="101"></a><a name="1.0.1"></a>1.0.1
* Corrección de errores

### <a name="100"></a><a name="1.0.0"></a>1.0.0
* Versión de disponibilidad general

### <a name="0110-preview"></a><a name="0.11.0-preview"></a>0.11.0-preview
* Se realizaron cambios en cómo se puede configurar CloudTableClient. Ahora toma una un objeto TableClientConfiguration durante la construcción. TableClientConfiguration proporciona diferentes propiedades para configurar el comportamiento del cliente dependiendo de si el punto de conexión de destino es Table API de Cosmos DB o Table API de Azure Storage.
* Se agregó compatibilidad para TableQuery para que devuelva resultados en el criterio de ordenación de una columna personalizada. Esta característica solo se admite en los puntos de conexión de Table de Cosmos DB.
* Se agregó compatibilidad para exponer RequestCharges en varios tipos de resultados. Esta característica solo se admite en los puntos de conexión de Table de Cosmos DB.

### <a name="0101-preview"></a><a name="0.10.1-preview"></a>0.10.1-preview
* Agregue compatibilidad con el token de SAS, las operaciones de TablePermissions, ServiceProperties y ServiceStats sobre los puntos de conexión de Azure Storage. 
   > [!NOTE]
   > Algunas funcionalidades de los SDK anteriores de tabla de Azure Storage todavía no se admiten, como el cifrado del lado cliente.

### <a name="0100-preview"></a><a name="0.10.0-preview"></a>0.10.0 (versión preliminar)
* Se ha agregado compatibilidad con operaciones básicas de CRUD, por lotes y consultas sobre los puntos de conexión de tabla de Azure Storage. 
   > [!NOTE]
   > Algunas funcionalidades de los SDK anteriores de tabla de Azure Storage todavía no se admiten, como el cifrado del lado cliente.

### <a name="091-preview"></a><a name="0.9.1-preview"></a>0.9.1 (versión preliminar)
* El SDK de .NET Standard para Table API de Azure Cosmos DB es una biblioteca de .NET multiplataforma que ofrece acceso eficaz al modelo de datos de tabla en Cosmos DB. Esta versión inicial es compatible con el conjunto completo de CRUD de tabla y entidad + funcionalidades de consulta con API similares como el [SDK de Table API de Cosmos DB para .NET Framework](table-sdk-dotnet.md). 
   > [!NOTE]
   >  Los puntos de conexión de tabla de Azure Storage aún no son compatibles con la versión preliminar de 0.9.1.

## <a name="release-and-retirement-dates"></a>Fechas de lanzamiento y de retirada
Microsoft notifica la retirada de un SDK con al menos **12 meses** de antelación para facilitar la transición a una versión compatible o más reciente.

Esta biblioteca estándar de .NET multiplataforma [Microsoft.Azure.Cosmos.Table](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table) reemplazará a la biblioteca de .NET Framework [Microsoft.Azure.CosmosDB.Table](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.Table).

### <a name="200-series"></a>Serie 2.0.0
| Versión | Fecha de la versión | Fecha de retirada |
| --- | --- | --- |
| [2.0.0-preview](#2.0.0-preview) |22 de agosto de 2019 |--- |

### <a name="100-series"></a>Serie 1.0.0
| Versión | Fecha de la versión | Fecha de retirada |
| --- | --- | --- |
| [1.0.5](#1.0.5) |13 de septiembre de 2019 |--- |
| [1.0.5-preview](#1.0.5-preview) |20 de agosto de 2019 |--- |
| [1.0.4](#1.0.4) |12 de agosto de 2019 |--- |
| [1.0.4-preview](#1.0.4-preview) |26 de julio de 2019 |--- |
| 1.0.2-preview |2 de mayo de 2019 |--- |
| [1.0.1](#1.0.1) |19 de abril de 2019 |--- |
| [1.0.0](#1.0.0) |13 de marzo de 2019 |--- |
| [0.11.0-preview](#0.11.0-preview) |5 de marzo de 2019 |--- |
| [0.10.1-preview](#0.10.1-preview) |22 de enero de 2019 |--- |
| [0.10.0 (versión preliminar)](#0.10.0-preview) |18 de diciembre de 2018 |--- |
| [0.9.1 (versión preliminar)](#0.9.1-preview) |18 de octubre de 2018 |--- |


## <a name="faq"></a>Preguntas más frecuentes

[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="see-also"></a>Consulte también
Para obtener más información sobre Table API de Azure Cosmos DB, consulte [Introducción a Table API de Azure Cosmos DB](table-introduction.md).
