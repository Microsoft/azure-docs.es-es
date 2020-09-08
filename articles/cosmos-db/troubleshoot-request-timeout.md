---
title: Solución de problemas de excepciones de tiempo de espera de solicitud del servicio Azure Cosmos DB
description: Obtenga información sobre el diagnóstico y la corrección de excepciones de tiempo de espera de solicitud del servicio Cosmos DB.
author: j82w
ms.service: cosmos-db
ms.date: 07/13/2020
ms.author: jawilley
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: 76a1558534728613dcdedc78b64a0366f2bd643d
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88871078"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-request-timeout-exceptions"></a>Diagnóstico y solución de problemas de excepciones de tiempo de espera de solicitud en Azure Cosmos DB
Azure Cosmos DB ha devuelto un error HTTP 408 Tiempo de espera de solicitud.

## <a name="troubleshooting-steps"></a>Pasos para solucionar problemas
La lista siguiente contiene las causas y las soluciones conocidas para las excepciones de tiempo de espera de solicitud.

### <a name="check-the-sla"></a>Compruebe el Acuerdo de Nivel de Servicio.
Consulte [Supervisión de Azure Cosmos DB](monitor-cosmos-db.md) para comprobar si hay excepciones con error 408 que infrinjan el Acuerdo de Nivel de Servicio de Azure Cosmos DB.

#### <a name="solution-1-it-didnt-violate-the-azure-cosmos-db-sla"></a>Solución 1: No se ha infringido el Acuerdo de Nivel de Servicio de Azure Cosmos DB
La aplicación debe controlar este escenario y volver a intentarlo tras estos errores transitorios.

#### <a name="solution-2-it-did-violate-the-azure-cosmos-db-sla"></a>Solución 2: Se ha infringido el Acuerdo de Nivel de Servicio de Cosmos DB
Póngase en contacto con el [soporte técnico de Azure](https://aka.ms/azure-support).
 
### <a name="hot-partition-key"></a>Clave de partición activa
Azure Cosmos DB distribuye el rendimiento general aprovisionado de forma uniforme entre las particiones físicas. Cuando hay una partición activa, una o varias claves de partición lógica de una partición física consumen todas las unidades de solicitud por segundo (RU/s) de la partición física. Al mismo tiempo, las RU/s de otras particiones físicas no se usan. Como síntoma, el total de RU/s consumidas será inferior al total de RU/s aprovisionado globalmente en la base de datos o el contenedor. Seguirá viendo limitaciones (errores 429) en las solicitudes en la clave de partición lógica activa. Use la [métrica de consumo normalizado de RU](monitor-normalized-request-units.md) para ver si la carga de trabajo está detectando una partición activa. 

#### <a name="solution"></a>Solución:
Elija una buena clave de partición que distribuya uniformemente el volumen y el almacenamiento de solicitudes. Obtenga más información acerca de [cómo cambiar la clave de partición](https://devblogs.microsoft.com/cosmosdb/how-to-change-your-partition-key/).

## <a name="next-steps"></a>Pasos siguientes
* [Diagnóstico y solución de problemas](troubleshoot-dot-net-sdk.md) al utilizar el SDK de Azure Cosmos DB para .NET.
* Más información sobre las directrices de rendimiento de [.NET v3](performance-tips-dotnet-sdk-v3-sql.md) y [.NET v2](performance-tips.md).