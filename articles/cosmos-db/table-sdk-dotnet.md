---
title: SDK y recursos de .NET para Table API de Azure Cosmos DB
description: Obtenga toda la información acerca de Table API de Azure Cosmos DB, incluidas las fechas de lanzamiento, las fechas de retirada y los cambios realizados en cada versión.
author: sakash279
ms.author: akshanka
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: dotnet
ms.topic: reference
ms.date: 08/17/2018
ms.openlocfilehash: f735668e1519a6a12082537dabfe17be3e96ffdd
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87460018"
---
# <a name="azure-cosmos-db-table-net-api-download-and-release-notes"></a>Table API .NET de Azure Cosmos DB: descarga y notas de la versión

> [!div class="op_single_selector"]
> * [.NET](table-sdk-dotnet.md)
> * [.NET Standard](table-sdk-dotnet-standard.md)
> * [Java](table-sdk-java.md)
> * [Node.js](table-sdk-nodejs.md)
> * [Python](table-sdk-python.md)

|   |   |
|---|---|
|**Descarga del SDK**|[NuGet](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.Table)|
|**Guía de inicio rápido**|[Azure Cosmos DB: compilación de una aplicación con .NET y Table API](create-table-dotnet.md)|
|**Tutorial**|[Azure Cosmos DB: desarrollo con Table API en .NET](tutorial-develop-table-dotnet.md)|
|**Plataforma admitida actualmente**|[Microsoft .NET Framework 4.5.1](https://www.microsoft.com/en-us/download/details.aspx?id=40779)|

> [!IMPORTANT]
> El SDK de .NET Framework [Microsoft.Azure.CosmosDB.Table](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.Table) se encuentra en modo de mantenimiento y dejará de utilizarse pronto. Actualice a la nueva biblioteca de .NET Standard [Microsoft.Azure.Cosmos.Table](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table) para continuar recibiendo las últimas características compatibles con Table API.

> Si ha creado una cuenta de Table API durante la versión preliminar, cree una [nueva cuenta de Table API](create-table-dotnet.md#create-a-database-account) para trabajar con los SDK de Table API disponibles con carácter general.
>

## <a name="release-notes"></a>Notas de la versión

### <a name="212"></a><a name="2.1.2"></a>2.1.2

* Corrección de errores

### <a name="210"></a><a name="2.1.0"></a>2.1.0

* Corrección de errores

### <a name="200"></a><a name="2.0.0"></a>2.0.0

* Compatibilidad con escrituras de varias regiones agregada
* Se corrigieron las dependencias del paquete NuGet en Microsoft.Azure.DocumentDB, Microsoft.OData.Core, Microsoft.OData.Edm, Microsoft.Spatial

### <a name="113"></a><a name="1.1.3"></a>1.1.3

* Se corrigieron las dependencias de Microsoft.Azure.Storage.Common and Microsoft.Azure.DocumentDB del paquete de NuGet.
* Correcciones de errores en la serialización de tablas cuando se configura JsonConvert.DefaultSettings.

### <a name="111"></a><a name="1.1.1"></a>1.1.1

* Se agregó validación para ETAG con formato incorrecto en el modo directo.
* Se corrigió el error de consulta LINQ en modo de puerta de enlace.
* Las API sincrónicas ahora se ejecutan en el grupo de subprocesos con SynchronizationContext.

### <a name="110"></a><a name="1.1.0"></a>1.1.0

* Incorporación de TableQueryMaxItemCount, TableQueryEnableScan, TableQueryMaxDegreeOfParallelism y TableQueryContinuationTokenLimitInKb a TableRequestOptions
* Correcciones de errores

### <a name="100"></a><a name="1.0.0"></a>1.0.0

* Versión de disponibilidad general

### <a name="090-preview"></a><a name="0.1.0-preview"></a>0.9.0-preview

* Versión preliminar inicial

## <a name="release-and-retirement-dates"></a>Fechas de lanzamiento y de retirada

Microsoft notifica la retirada de un SDK con al menos **12 meses** de antelación para facilitar la transición a una versión compatible o más reciente.

En la actualidad, la biblioteca `Microsoft.Azure.CosmosDB.Table` solo está disponible para .NET Framework, está en modo de mantenimiento y dejará pronto de usarse. Las nuevas características y funcionalidades y las optimizaciones solo se agregan a la biblioteca [Microsoft.Azure.Cosmos.Table](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table) de .NET Standard, por lo que se recomienda actualizar a [Microsoft.Azure.Cosmos.Table](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table).

El paquete de versión preliminar [WindowsAzure.Storage-PremiumTable](https://www.nuget.org/packages/WindowsAzure.Storage-PremiumTable/0.1.0-preview) está en desuso. El SDK de WindowsAzure.Storage PremiumTable se retirará el 15 de noviembre de 2018, momento en el cual las solicitudes al SDK retirado no se permitirán.

| Versión | Fecha de la versión | Fecha de retirada |
| --- | --- | --- |
| [2.1.2](#2.1.2) |16 de septiembre de 2019| |
| [2.1.0](#2.1.0) |22 de enero de 2019|1 de abril de 2020 |
| [2.0.0](#2.0.0) |26 de septiembre de 2018|1 de marzo de 2020 |
| [1.1.3](#1.1.3) |17 de julio de 2018|1 de diciembre de 2019 |
| [1.1.1](#1.1.1) |26 de marzo de 2018|1 de diciembre de 2019 |
| [1.1.0](#1.1.0) |21 de febrero de 2018|1 de diciembre de 2019 |
| [1.0.0](#1.0.0) |15 de noviembre de 2017|15 de noviembre de 2019 |
| 0.9.0-preview |11 de noviembre de 2017 |11 de noviembre de 2019 |

## <a name="troubleshooting"></a>Solución de problemas

Si se produce un error 

```
Unable to resolve dependency 'Microsoft.Azure.Storage.Common'. Source(s) used: 'nuget.org', 
'CliFallbackFolder', 'Microsoft Visual Studio Offline Packages', 'Microsoft Azure Service Fabric SDK'`
```

al intentar usar el paquete de NuGet Microsoft.Azure.CosmosDB.Table, tiene dos opciones para solucionar el problema:

* Usar la consola de administración de paquetes para instalar el paquete Microsoft.Azure.CosmosDB.Table y sus dependencias. Para ello, escriba lo siguiente en la consola de administración de paquetes de la solución. 

    ```powershell
    Install-Package Microsoft.Azure.CosmosDB.Table -IncludePrerelease
    ```

    
* Usar su herramienta preferida de administración de paquetes de NuGet para instalar el paquete de NuGet Microsoft.Azure.Storage.Common antes de instalar Microsoft.Azure.CosmosDB.Table.

## <a name="faq"></a>Preguntas más frecuentes

[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="see-also"></a>Consulte también

Para obtener más información sobre Table API de Azure Cosmos DB, consulte [Introducción a Table API de Azure Cosmos DB](table-introduction.md). 
