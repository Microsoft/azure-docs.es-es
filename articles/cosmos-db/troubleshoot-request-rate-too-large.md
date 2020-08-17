---
title: Solución de problemas de la excepción de tasa de solicitudes demasiado grande en Azure Cosmos DB
description: Diagnóstico y corrección de excepción de tasa de solicitudes demasiado grande
author: j82w
ms.service: cosmos-db
ms.date: 07/13/2020
ms.author: jawilley
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: b20ee2e10648d2d8eb248fc3f98c4fb33805f914
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87293853"
---
# <a name="diagnose-and-troubleshoot-cosmos-db-too-many-requests"></a>Diagnóstico y solución de problemas de demasiadas solicitudes en Cosmos DB
El mensaje "Tasa de solicitudes demasiado grande" o el código de error 429 indican que las solicitudes están limitadas.

## <a name="troubleshooting-steps"></a>Pasos para solucionar problemas
La lista siguiente contiene las causas y las soluciones conocidas para la excepción de demasiadas solicitudes.

### <a name="1-check-the-metrics"></a>1. Consultar las métricas
El cliente debe consultar la [supervisión de Azure Cosmos DB](monitor-cosmos-db.md) para comprobar si hay excepciones con el número 429.

## <a name="cause"></a>Causa:
El rendimiento consumido (RU/s) ha superado el [rendimiento aprovisionado](set-throughput.md). El SDK reintentará automáticamente las solicitudes según la directiva de reintentos especificada. Si recibe este error con frecuencia, considere la posibilidad de aumentar el rendimiento de la colección. Compruebe las métricas del portal para ver si está obteniendo errores 429. Revise su clave de partición para asegurarse de que se produce una [distribución uniforme del volumen de solicitudes y almacenamiento](partition-data.md).

## <a name="solution"></a>Solución:
1. Use el [portal o el SDK](set-throughput.md) para aumentar el rendimiento aprovisionado.
2. Cambie la base de datos o el contenedor a [Escalado automático](provision-throughput-autoscale.md).

## <a name="next-steps"></a>Pasos siguientes
* [Diagnóstico y solución de problemas](troubleshoot-dot-net-sdk.md) al usar el SDK de .NET de Azure Cosmos DB
* Más información sobre las directrices de rendimiento para [.NET V3](performance-tips-dotnet-sdk-v3-sql.md) y [.NET V2](performance-tips.md)