---
title: Solución de problemas de las excepciones de tasa de solicitudes demasiado grande en Azure Cosmos DB
description: Aprenda a diagnosticar y corregir las excepciones de tasa de solicitudes demasiado grande.
author: j82w
ms.service: cosmos-db
ms.date: 07/13/2020
ms.author: jawilley
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: f8b1129c94ecf80efb60a13a0b80b1cc1817ff3f
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88871095"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-request-rate-too-large-exceptions"></a>Diagnóstico y solución de problemas de las excepciones de tasa de solicitud demasiado grande en Azure Cosmos DB
El mensaje "Tasa de solicitudes demasiado grande" o el código de error 429 indican que las solicitudes están siendo limitadas.

## <a name="troubleshooting-steps"></a>Pasos para solucionar problemas
En la sección siguiente se indican las causas y las soluciones conocidas para la excepción de demasiadas solicitudes.

### <a name="check-the-metrics"></a>Consulta de las métricas
Consulte [Supervisión de Azure Cosmos DB](monitor-cosmos-db.md) para comprobar el número de excepciones 429.

#### <a name="cause"></a>Causa:
El rendimiento consumido (unidades de solicitud por segundo) ha superado el [rendimiento aprovisionado](set-throughput.md). El SDK reintentará automáticamente las solicitudes según la directiva de reintentos especificada. Si recibe este error con frecuencia, considere la posibilidad de aumentar el rendimiento de la colección. Compruebe las métricas del portal para ver si está obteniendo errores 429. Revise su clave de partición para asegurarse de que se produce una [distribución uniforme del volumen de solicitudes y almacenamiento](partition-data.md).

#### <a name="solution"></a>Solución:
1. Use el [portal o el SDK](set-throughput.md) para aumentar el rendimiento aprovisionado.
1. Cambie la base de datos o el contenedor a [Escalado automático](provision-throughput-autoscale.md).

## <a name="next-steps"></a>Pasos siguientes
* [Diagnóstico y solución de problemas](troubleshoot-dot-net-sdk.md) al utilizar el SDK de Azure Cosmos DB para .NET.
* Más información sobre las directrices de rendimiento de [.NET v3](performance-tips-dotnet-sdk-v3-sql.md) y [.NET v2](performance-tips.md).