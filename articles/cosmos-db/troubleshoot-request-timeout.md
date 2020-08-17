---
title: Solución de problemas de excepción de tiempo de espera de solicitud de servicio de Azure Cosmos DB
description: Diagnóstico y corrección de excepción de tiempo de espera de solicitud de servicio de Cosmos DB
author: j82w
ms.service: cosmos-db
ms.date: 07/13/2020
ms.author: jawilley
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: bd8102ee73721e2a56185709d1076e489ecc7607
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87293856"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-request-timeout"></a>Diagnóstico y solución de problemas de tiempo de espera de solicitud de Azure Cosmos DB
Azure Cosmos DB ha devuelto un error HTTP 408 Tiempo de espera de solicitud

## <a name="troubleshooting-steps"></a>Pasos para solucionar problemas
La lista siguiente contiene las causas y las soluciones conocidas para las excepciones de tiempo de espera de solicitud.

### <a name="1-check-the-sla"></a>1. Compruebe el Acuerdo de Nivel de Servicio.
El cliente debe consultar la [supervisión de Azure Cosmos DB](monitor-cosmos-db.md) para comprobar si hay excepciones con el número 408 que infrinjan el Acuerdo de Nivel de Servicio de Cosmos DB.

#### <a name="solution-1-it-did-not-violate-the-cosmos-db-sla"></a>Solución 1: No se ha infringido el Acuerdo de Nivel de Servicio de Cosmos DB.
La aplicación debe controlar este escenario y volver a intentarlo tras estos errores transitorios.

#### <a name="solution-2-it-did-violate-the-cosmos-db-sla"></a>Solución 2: Se ha infringido el Acuerdo de Nivel de Servicio de Cosmos DB.
Póngase en contacto con el soporte técnico de Azure: https://aka.ms/azure-support.
 
### <a name="2-hot-partition-key"></a>2. Clave de partición activa
Azure Cosmos DB distribuye el rendimiento general aprovisionado de forma uniforme entre las particiones físicas. Cuando hay una partición activa, una o más claves de partición lógicas de una partición física consumen todos las RU/s de la partición física, mientras que las RU/s de otras particiones físicas no se usan. Como síntoma, el total de RU/s consumido será inferior al total de RU/s aprovisionadas en la base de datos o el contenedor, pero seguirá viendo la limitación (429s) en las solicitudes de la clave de la partición lógica activa. Use la [métrica de consumo normalizado de RU](monitor-normalized-request-units.md) para ver si la carga de trabajo está detectando una partición activa. 

#### <a name="solution"></a>Solución:
Elija una buena clave de partición que distribuya uniformemente el volumen y el almacenamiento de solicitudes. Obtenga más información acerca de [cómo cambiar la clave de partición](https://devblogs.microsoft.com/cosmosdb/how-to-change-your-partition-key/).

## <a name="next-steps"></a>Pasos siguientes
* [Diagnóstico y solución de problemas](troubleshoot-dot-net-sdk.md) al usar el SDK de .NET de Azure Cosmos DB
* Más información sobre las directrices de rendimiento para [.NET V3](performance-tips-dotnet-sdk-v3-sql.md) y [.NET V2](performance-tips.md)