---
title: 'Manejabilidad y supervisión: actividad de consultas y uso de recursos'
description: Obtenga información sobre las funcionalidades disponibles para administrar y supervisar Azure Synapse Analytics. Utilice Azure Portal y vistas de administración dinámica (DMV) para comprender la actividad de consultas y el uso de recursos del almacenamiento de datos.
services: synapse-analytics
author: kevinvngo
manager: craigg-msft
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 04/09/2020
ms.author: kevin
ms.reviewer: jrasnick
ms.custom: azure-synapse
ms.openlocfilehash: 334d37ad36336d1aa737894482fd8f66e929c822
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87077691"
---
# <a name="monitoring-resource-utilization-and-query-activity-in-azure-synapse-analytics"></a>Supervisión del uso de recursos y la actividad de consultas en Azure Synapse Analytics

Azure Synapse Analytics proporciona una experiencia de supervisión enriquecida dentro de Azure Portal que expone la información relacionada con la carga de trabajo del almacenamiento de datos. Azure Portal es la herramienta recomendada al supervisar el almacenamiento de datos, ya que proporciona períodos de retención configurables, alertas, recomendaciones, y gráficos y paneles personalizables para métricas y registros. El portal también le permite integrarse con otros servicios de supervisión de Azure tales como Azure Monitor (registros) con Log Analytics para ofrecer una experiencia de supervisión holística no solo para el almacenamiento de datos sino también para toda la plataforma Azure Analytics como experiencia de supervisión integrada. Esta documentación describe las funcionalidades de supervisión disponibles para optimizar y administrar la plataforma de análisis con Synapse SQL.

## <a name="resource-utilization"></a>Utilización de recursos

En Azure Portal para Synapse SQL hay disponibles las métricas siguientes. Estas métricas se exponen a través de [Azure Monitor](../../azure-monitor/platform/data-collection.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#metrics).

| Nombre de la métrica             | Descripción                                                  | Tipo de agregación |
| ----------------------- | ------------------------------------------------------------ | ---------------- |
| Porcentaje de CPU          | Uso de CPU en todos los nodos del almacenamiento de datos      | Prom., Mín., Máx.    |
| Porcentaje de E/S de datos      | Uso de E/S en todos los nodos del almacenamiento de datos       | Prom., Mín., Máx.    |
| Porcentaje de memoria       | Uso de la memoria (SQL Server) en todos los nodos del almacenamiento de datos | Prom., Mín., Máx.   |
| Consultas activas          | Número de consultas activas que se ejecutan en el sistema             | Sum              |
| Consultas en cola          | Número de consultas en cola en espera para iniciar la ejecución          | Sum              |
| Conexiones correctas  | Número de conexiones correctas (inicios de sesión) a la base de datos | Suma, Recuento       |
| Conexiones con errores      | Número de conexiones incorrectas (inicios de sesión) a la base de datos | Suma, Recuento       |
| Bloqueado por el firewall     | Número de inicios de sesión en el almacenamiento de datos que se ha bloqueado     | Suma, Recuento       |
| Límite de DWU               | Objetivo de nivel de servicio del almacenamiento de datos                | Prom., Mín., Máx.    |
| Porcentaje de DWU          | Máximo entre el porcentaje de CPU y porcentaje de E/S de datos        | Prom., Mín., Máx.    |
| DWU utilizada                | Límite de DWU * Porcentaje de DWU                                   | Prom., Mín., Máx.    |
| Porcentaje de aciertos de caché    | (aciertos de caché/error de caché) * 100 donde aciertos de caché corresponde al total de todos los aciertos de segmentos del almacén de columnas en la caché de SSD local y error de caché corresponde a la suma de los errores de segmentos del almacén de columnas en la caché SSD local de todos los nodos | Prom., Mín., Máx.    |
| Porcentaje de caché usada   | (caché usada/capacidad de la memoria caché) * 100 donde la memoria caché usada corresponde al total de todos los bytes en la caché SSD local en todos los nodos y capacidad de la memoria caché corresponde al total de la capacidad de almacenamiento de la caché SSD local en todos los nodos | Prom., Mín., Máx.    |
| Porcentaje de tempdb local | El uso de tempdb local en todos los nodos de ejecución. Los valores se emiten cada cinco minutos | Prom., Mín., Máx.    |

Aspectos que se deben tener en cuenta al visualizar métricas y establecer alertas:

- Las unidades de almacenamiento de datos son solo una **representación de alto nivel del uso** en el grupo de SQL y no se pretende que constituyan un indicador exclusivo de utilización. Para determinar si realizar un escalado o reducción vertical, tenga en cuenta todos los factores que podrían verse afectados por el indicador Unidad de almacenamiento de datos como, por ejemplo, la simultaneidad, la memoria, tempdb y la capacidad de la caché adaptable. Le recomendamos [ejecutar la carga de trabajo con valores diferentes de DWU](sql-data-warehouse-manage-compute-overview.md#finding-the-right-size-of-data-warehouse-units) para averiguar cuál funciona mejor para satisfacer sus objetivos de negocio.
- Las conexiones erróneas y correctas se indican para un almacenamiento de datos determinado, no para el propio servidor.
- El porcentaje de memoria refleja el uso incluso si el almacenamiento de datos se encuentra en estado de inactividad (no refleja el consumo de memoria de carga de trabajo activa). Use y realice un seguimiento de esta métrica junto con otras (tempdb, memoria caché de Gen2) para tomar una decisión holística sobre si el escalado para conseguir capacidad de la memoria caché adicional aumentará el rendimiento de la carga de trabajo para cumplir sus requisitos.

## <a name="query-activity"></a>Actividad de consultas

Para brindar una experiencia de programación al supervisar Synapse SQL mediante T-SQL, el servicio proporciona un conjunto de vistas de administración dinámica (DMV). Estas vistas resultan útiles para la solución de problemas y la identificación activas de cuellos de botella de rendimiento con la carga de trabajo.

Para ver la lista de las DMV que se aplican al SQL de Synapse, vea esta [documentación](../sql/reference-tsql-system-views.md#sql-pool-dynamic-management-views-dmvs). 

## <a name="metrics-and-diagnostics-logging"></a>Métricas y registros de diagnóstico 

Las métricas y los registros se pueden exportar a Azure Monitor, específicamente al componente de [registros de Azure Monitor](../../azure-monitor/log-query/log-query-overview.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json), y se puede acceder a ellos mediante programación a través de las [consultas de registros](../../azure-monitor/log-query/get-started-portal.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). La latencia del registro para Synapse SQL es de entre 10 y 15 minutos. Para más detalles sobre los factores que afectan a la latencia, consulte la siguiente documentación.

## <a name="next-steps"></a>Pasos siguientes

En la siguiente guía de procedimientos se describen escenarios comunes y casos de uso al supervisar y administrar el almacenamiento de datos:

- [Supervisión de la carga de trabajo mediante DMV](sql-data-warehouse-manage-monitor.md)
