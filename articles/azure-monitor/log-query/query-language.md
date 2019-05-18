---
title: Consultas de registros de Azure Monitor | Microsoft Docs
description: Referencias a recursos para aprender a escribir consultas de registros en Azure Monitor.
services: log-analytics
documentationcenter: ''
author: bwren
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 01/11/2019
ms.author: bwren
ms.openlocfilehash: 6fae245e4d72f7f6f7809d85da17023deb1518e5
ms.sourcegitcommit: 179918af242d52664d3274370c6fdaec6c783eb6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65560849"
---
# <a name="azure-monitor-log-queries"></a>Consultas de registros de Azure Monitor
Los registros de Azure Monitor se crean en Azure Data Explorer y las consultas de registros de Azure Monitor usan una versión del mismo lenguaje de consulta de Kusto. La [documentación del lenguaje de consulta de Kusto](/azure/kusto/query) incluye todos los detalles del lenguaje y debe ser el principal recurso para escribir consultas de registros de Azure Monitor. Esta página proporciona vínculos a otros recursos para aprender a escribir consultas y conocer las diferencias con la implementación de Azure Monitor del lenguaje.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="getting-started"></a>Introducción

- [Introducción a Log Analytics de Azure Monitor](get-started-portal.md) es una lección para escribir consultas y trabajar con los resultados en el portal de Azure.
- [Introducción a las consultas de registros de Azure Monitor](get-started-queries.md) es una lección sobre cómo escribir consultas con los datos de registro de Azure Monitor.

## <a name="concepts"></a>Conceptos
- El [análisis de los datos de registro en Azure Monitor](../../azure-monitor/log-query/log-query-overview.md) ofrece una breve introducción a las consultas de registros y describe cómo se estructuran los datos de Azure Monitor.
- En [Visualización y análisis de datos de registro en Azure Monitor](../../azure-monitor/log-query/portals.md) se explican los portales donde se crean y ejecutan las consultas de registros.

## <a name="reference"></a>Referencia

- [Referencia del lenguaje de consulta](/azure/kusto/query) es la referencia completa del lenguaje de consulta de Kusto.
- [Diferencias del lenguaje de consulta de registros de Azure Monitor](data-explorer-difference.md) describe las diferencias entre las versiones del lenguaje de consulta de Kusto.
- [Propiedades estándar de los registros de Azure Monitor](../../azure-monitor/platform/log-standard-properties.md) describe las propiedades que son estándar para todos los datos de registro de Azure Monitor.
- [Realización de consultas de registros entre recursos en Azure Monitor](../../azure-monitor/log-query/cross-workspace-query.md) indica cómo escribir consultas de registros que usan datos procedentes de varias áreas de trabajo de Log Analytics y aplicaciones de Application Insights.


## <a name="examples"></a>Ejemplos

- [Ejemplos de consultas de registros de Azure Monitor](examples.md) proporciona ejemplos de consultas con datos de registro de Azure Monitor.



## <a name="lessons"></a>Lecciones

- [Trabajo con cadenas en consultas de registros de Azure Monitor](string-operations.md) describe cómo trabajar con datos de cadena.
- [Trabajar con valores de fecha y hora en consultas de registros de Azure Monitor](datetime-operations.md) describe cómo trabajar con datos de fecha y hora. 
- [Agregaciones en consultas de registros de Azure Monitor](aggregations.md) y [Agregaciones avanzadas en consultas de registros de Azure Monitor](advanced-aggregations.md) describen cómo agregar y resumir datos.
- [Combinaciones en consultas de registros de Azure Monitor](joins.md) describe cómo combinar datos de varias tablas.
- [Trabajo con JSON y estructuras de datos en consultas de registros de Azure Monitor](json-data-structures.md) describe cómo analizar datos JSON.
- [Escritura de consultas de registros avanzadas en Azure Monitor](advanced-query-writing.md) describe las estrategias para crear consultas complejas y reutilizar código.
- [Creación de gráficos y diagramas a partir de consultas de registros de Azure Monitor](charts.md) describe cómo visualizar los datos de una consulta de registros.

## <a name="cheatsheets"></a>Hojas de características clave

-  [Consulta de registros de SQL en Azure Monitor](sql-cheatsheet.md) ayuda a los usuarios que ya están familiarizados con SQL.
-  [Consulta de registros de Splunk en Azure Monitor](splunk-cheatsheet.md) ayuda a los usuarios que ya están familiarizados con Splunk.
 
## <a name="next-steps"></a>Pasos siguientes

- Acceda a la [documentación de referencia completa del lenguaje de consulta de Kusto](/azure/kusto/query/).
