---
title: Configuración y acceso a los registros en un servidor flexible de Azure Database for PostgreSQL
description: Procedimiento para acceder a los registros de base de datos para un servidor flexible de Azure Database for PostgreSQL
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: 0b7b09696cbbe12a57d066e452b4c8ea7a7b8f27
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932174"
---
# <a name="configure-and-access-logs-in-azure-database-for-postgresql---flexible-server"></a>Configuración y acceso a los registros en un servidor flexible de Azure Database for PostgreSQL

> [!IMPORTANT]
> La opción de implementación Servidor flexible de Azure Database for PostgreSQL está en versión preliminar

Los registros de PostgreSQL están disponibles en todos los nodos de un servidor flexible. Puede enviar registros a un servidor de almacenamiento o a un servicio de análisis. Los registros se pueden usar para identificar, solucionar y reparar errores de configuración y casos de rendimiento no óptimo.

## <a name="configure-diagnostic-settings"></a>Configuración de valores de diagnóstico

Puede habilitar la configuración de diagnóstico para el servidor de Postgres mediante Azure Portal, la CLI, la API REST y PowerShell. La categoría de registro que se debe seleccionar es **PostgreSQLLogs**

Para habilitar los registros de recursos mediante Azure Portal:

1. En el portal, vaya a *Configuración de diagnóstico*  en el menú de navegación del servidor de Postgres.
   
2. Seleccione *Agregar configuración de diagnóstico*.
   :::image type="content" source="media/howto-logging/diagnostic-settings.png" alt-text="Botón Agregar configuración de diagnóstico":::

3. Asigne un nombre a esta configuración. 

4. Seleccione el punto de conexión preferido (cuenta de almacenamiento, centro de eventos, análisis de registros). 

5. Seleccione el tipo de registro **PostgreSQLLogs**.
   :::image type="content" source="media/howto-logging/diagnostic-create-setting.png" alt-text="Elección de los registros de PostgreSQL":::

7. Guarde la configuración.

Para habilitar registros de recursos mediante PowerShell, la CLI o la API REST, vea el artículo sobre la [configuración del diagnóstico](../../azure-monitor/platform/diagnostic-settings.md).

### <a name="access-resource-logs"></a>Acceso a los registros de recursos

La forma de acceder a los registros depende del punto de conexión que elija. Si se trata de Azure Storage, consulte el artículo sobre la [cuenta de almacenamiento de registros](../../azure-monitor/platform/resource-logs-collect-storage.md). Si se trata de Event Hubs, consulte el artículo [Transmisión de los registros de Azure](../../azure-monitor/platform/resource-logs-stream-event-hubs.md).

Si se trata de los registros de Azure Monitor, los registros se envían al área de trabajo seleccionada. Los registros de Postgres usan el modo de recopilación **AzureDiagnostics**, por lo que se pueden consultar desde la tabla AzureDiagnostics. A continuación se describen los campos de la tabla. Obtenga más información acerca de las consultas y las alertas en [Introducción a las consultas de registro en Azure Monitor](../../azure-monitor/log-query/log-query-overview.md).

A continuación se muestran consultas que puede intentar iniciar. Puede configurar alertas basadas en las consultas.

Buscar todos los registros de Postgres de un servidor determinado del último día

```kusto
AzureDiagnostics
| where LogicalServerName_s == "myservername"
| where Category == "PostgreSQLLogs"
| where TimeGenerated > ago(1d) 
```

Buscar todos los intentos de conexión que no son de localhost

```kusto
AzureDiagnostics
| where Message contains "connection received" and Message !contains "host=127.0.0.1"
| where Category == "PostgreSQLLogs" and TimeGenerated > ago(6h)
```

La consulta anterior mostrará los resultados de las últimas 6 horas para cualquier servidor Postgres que inicie sesión en esta área de trabajo.

## <a name="next-steps"></a>Pasos siguientes

- [Introducción a las consultas de Log Analytics](https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-portal)
- Información acerca de [Azure Event Hubs](https://docs.microsoft.com/azure/event-hubs/event-hubs-about)
