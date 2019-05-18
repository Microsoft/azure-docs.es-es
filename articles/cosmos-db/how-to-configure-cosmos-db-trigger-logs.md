---
title: Registros de desencadenador de Azure Cosmos DB
description: Aprenda a exponer los registros de desencadenador de Azure Cosmos DB a la canalización de registro de Azure Functions.
author: ealsur
ms.service: cosmos-db
ms.topic: sample
ms.date: 5/2/2019
ms.author: maquaran
ms.openlocfilehash: a598842ecde9508a6c2185a6097f689252fda60c
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/09/2019
ms.locfileid: "65515023"
---
# <a name="how-to-configure-and-read-the-azure-cosmos-db-trigger-logs"></a>Configuración y lectura de los registros de desencadenador de Azure Cosmos DB

En este artículo se describe cómo se puede configurar el entorno de Azure Functions para enviar los registros de desencadenador de Azure Cosmos DB a su [solución de supervisión](../azure-functions/functions-monitoring.md) configurada.

## <a name="included-logs"></a>Registros incluidos

El desencadenador de Azure Cosmos DB utiliza la [biblioteca de procesador de fuente de cambios](./change-feed-processor.md) internamente y la biblioteca genera un conjunto de registros de mantenimiento que se pueden usar para supervisar las operaciones internas con [fines de solución de problemas](./troubleshoot-changefeed-functions.md).

Los registros de mantenimiento describen cómo se comporta el desencadenador de Azure Cosmos DB cuando se intentan realizar operaciones durante escenarios de equilibrio de carga o de inicialización.

## <a name="enabling-logging"></a>Habilitación del registro

Para habilitar el registro del desencadenador de Azure Cosmos DB, busque el archivo `host.json` en el proyecto de Azure Functions o en la aplicación de Azure Functions y [configure el nivel de registro necesario](../azure-functions/functions-monitoring.md#log-configuration-in-hostjson). Tendrá que habilitar el seguimiento de `Host.Triggers.CosmosDB`, tal como se muestra en el ejemplo siguiente:

```js
{
  "version": "2.0",
  "logging": {
    "fileLoggingMode": "always",
    "logLevel": {
      "Host.Triggers.CosmosDB": "Trace"
    }
  }
}
```

Después de implementar la función de Azure con la configuración actualizada, verá los registros del desencadenador de Azure Cosmos DB como parte de los seguimientos. Puede ver los registros en un proveedor de registro configurado en la *categoría* `Host.Triggers.CosmosDB`.

## <a name="query-the-logs"></a>Consulta de los registros

Ejecute la siguiente consulta para consultar los registros generados por el desencadenador de Azure Cosmos DB en [Analytics de Azure Application Insights](../azure-monitor/app/analytics.md):

```sql
traces
| where customDimensions.Category == "Host.Triggers.CosmosDB"
```

## <a name="next-steps"></a>Pasos siguientes

* [Habilite la supervisión](../azure-functions/functions-monitoring.md) en las aplicaciones de Azure Functions.
* Obtenga información sobre cómo [diagnosticar y solucionar problemas comunes](./troubleshoot-changefeed-functions.md) al usar el desencadenador de Azure Cosmos DB en Azure Functions.