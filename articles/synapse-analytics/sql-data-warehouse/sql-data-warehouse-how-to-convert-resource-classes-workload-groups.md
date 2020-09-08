---
title: Conversión de la clase de recursos en un grupo de cargas de trabajo
description: Obtenga información sobre cómo crear un grupo de cargas de trabajo similar a una clase de recursos en Azure SQL Data Warehouse.
services: synapse-analytics
author: ronortloff
manager: craigg
ms.service: synapse-analytics
ms.subservice: sql-dw
ms.topic: conceptual
ms.date: 08/13/2020
ms.author: rortloff
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: c61e8df05c4bc199c0d91b8ed0cbd73fa6f196cf
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88192316"
---
# <a name="convert-resource-classes-to-workload-groups"></a>Conversión de las clases de recursos en grupos de cargas de trabajo

Los grupos de cargas de trabajo proporcionan un mecanismo para aislar y contener recursos del sistema.  Además, los grupos de cargas de trabajo permiten establecer reglas de ejecución para las solicitudes que se ejecutan en estos.  Una regla de ejecución de tiempo de expiración de la consulta permite la cancelación de consultas descontroladas sin intervención del usuario.  En este artículo se explica cómo tomar una clase de recursos existente y crear un grupo de cargas de trabajo con una configuración similar.  Además, se agrega una regla opcional de tiempo de espera de la consulta.

> [!NOTE]
> Vea la sección [Combinación de asignaciones de clase de recursos con clasificadores](sql-data-warehouse-workload-classification.md#mixing-resource-class-assignments-with-classifiers) del documento del concepto [Clasificación de la carga de trabajo](sql-data-warehouse-workload-classification.md) para obtener una guía sobre el uso de grupos de cargas de trabajo y clases de recursos al mismo tiempo.

## <a name="understanding-the-existing-resource-class-configuration"></a>Descripción de la configuración de la clase de recursos existente

Los grupos de cargas de trabajo requieren un parámetro denominado `REQUEST_MIN_RESOURCE_GRANT_PERCENT`, que especifica el porcentaje de recursos generales del sistema asignados por solicitud.  La asignación de recursos para las [clases de recursos](resource-classes-for-workload-management.md#what-are-resource-classes) se realiza mediante la asignación de espacios de simultaneidad.  Para determinar el valor que se va a especificar para `REQUEST_MIN_RESOURCE_GRANT_PERCENT`, use la DMV de <link tbd> sys.dm_workload_management_workload_groups_stats.  Por ejemplo, la consulta siguiente devuelve un valor que puede usar el parámetro `REQUEST_MIN_RESOURCE_GRANT_PERCENT` con el fin de crear un grupo de cargas de trabajo similar a staticrc40.

```sql
SELECT Request_min_resource_grant_percent = Effective_request_min_resource_grant_percent
  FROM sys.dm_workload_management_workload_groups_stats
  WHERE name = 'staticrc40'
```

> [!NOTE]
> Los grupos de cargas de trabajo operan según el porcentaje de recursos generales del sistema.  

Dado que los grupos de cargas de trabajo operan en función del porcentaje de recursos generales del sistema, a medida que se escala y se reduce verticalmente, se cambia el porcentaje de recursos asignados a las clases de recursos estáticos en relación con los recursos generales del sistema.  Por ejemplo, staticrc40 en DW1000c asigna el 19,2 % de los recursos generales del sistema.  En DW2000c, se asigna el 9,6 %.  Este modelo es parecido si se quiere escalar verticalmente para la simultaneidad o si se quieren asignar más recursos por solicitud.

## <a name="create-workload-group"></a>Creación de un grupo de cargas de trabajo

Con el valor `REQUEST_MIN_RESOURCE_GRANT_PERCENT`conocido, se puede usar la sintaxis de <link> CREATE WORKLOAD GROUP para crear el grupo de cargas de trabajo.  Opcionalmente, puede especificar un valor `MIN_PERCENTAGE_RESOURCE` mayor que cero para aislar los recursos del grupo de cargas de trabajo.  Además, puede especificar opcionalmente un valor `CAP_PERCENTAGE_RESOURCE` menor de 100 para limitar la cantidad de recursos que puede consumir el grupo de cargas de trabajo.  

Usando mediumrc como base para un ejemplo, el código siguiente establece el valor `MIN_PERCENTAGE_RESOURCE` para dedicar el 10 % de los recursos del sistema a `wgDataLoads` y se garantiza que se podrá ejecutar una consulta todas las horas.  Además, `CAP_PERCENTAGE_RESOURCE` se establece en el 40 % y limita este grupo de cargas de trabajo a cuatro solicitudes simultáneas.  Al establecer el parámetro `QUERY_EXECUTION_TIMEOUT_SEC` en 3600, las consultas que se ejecuten durante más de 1 hora se cancelarán automáticamente.

```sql
CREATE WORKLOAD GROUP wgDataLoads WITH  
( REQUEST_MIN_RESOURCE_GRANT_PERCENT = 10
 ,MIN_PERCENTAGE_RESOURCE = 10
 ,CAP_PERCENTAGE_RESOURCE = 40
 ,QUERY_EXECUTION_TIMEOUT_SEC = 3600)
```

## <a name="create-the-classifier"></a>Creación del clasificador

Anteriormente, la asignación de consultas a las clases de recursos se realizaba con [sp_addrolemember](resource-classes-for-workload-management.md#change-a-users-resource-class).  Para lograr la misma funcionalidad y asignar las solicitudes a los grupos de cargas de trabajo, use la sintaxis [CREATE WORKLOAD CLASSIFIER](/sql/t-sql/statements/create-workload-classifier-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).  El uso de sp_addrolemember solo permite asignar recursos a una solicitud basada en un inicio de sesión.  Un clasificador proporciona opciones adicionales además del inicio de sesión, como las siguientes:
    - etiqueta
    - hora de sesión
    - En el ejemplo siguiente se asignan consultas del inicio de sesión de `AdfLogin` que también tienen la [ETIQUETA DE OPCIÓN](sql-data-warehouse-develop-label.md) establecida en `factloads` al grupo de cargas de trabajo `wgDataLoads` creado anteriormente.

```sql
CREATE WORKLOAD CLASSIFIER wcDataLoads WITH  
( WORKLOAD_GROUP = 'wgDataLoads'
 ,MEMBERNAME = 'AdfLogin'
 ,WLM_LABEL = 'factloads')
```

## <a name="test-with-a-sample-query"></a>Prueba con una consulta de ejemplo

A continuación se muestra una consulta de ejemplo y una consulta DMV para garantizar que el grupo de cargas de trabajo y el clasificador se configuran correctamente.

```sql
SELECT SUSER_SNAME() --should be 'AdfLogin'

--change to a valid table AdfLogin has access to
SELECT TOP 10 *
  FROM nation
  OPTION (label='factloads')

SELECT request_id, [label], classifier_name, group_name, command
  FROM sys.dm_pdw_exec_requests
  WHERE [label] = 'factloads'
  ORDER BY submit_time DESC
```

## <a name="next-steps"></a>Pasos siguientes

- [Aislamiento de cargas de trabajo](sql-data-warehouse-workload-isolation.md)
- [Creación de un grupo de carga de trabajo](quickstart-configure-workload-isolation-tsql.md)
- [CREATE WORKLOAD CLASSIFIER (Transact-SQL)](/sql/t-sql/statements/create-workload-classifier-transact-sql?&view=azure-sqldw-latest)
- [CREATE WORKLOAD GROUP (Transact-SQL)](/sql/t-sql/statements/create-workload-group-transact-sql?view=azure-sqldw-latest)
