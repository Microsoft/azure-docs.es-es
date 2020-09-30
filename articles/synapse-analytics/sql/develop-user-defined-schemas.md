---
title: Esquemas definidos por el usuario en SQL de Synapse
description: En las secciones siguientes encontrará varias sugerencias sobre el uso de esquemas definidos por el usuario de T-SQL para desarrollar soluciones con la funcionalidad de SQL de Synapse de Azure Synapse Analytics.
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: v-stazar
ms.reviewer: jrasnick
ms.openlocfilehash: aebe1d995f3cb6da4663876b8d39d36a1a8b16c9
ms.sourcegitcommit: 3fc3457b5a6d5773323237f6a06ccfb6955bfb2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90030174"
---
# <a name="user-defined-schemas-within-synapse-sql"></a>Esquemas definidos por el usuario en SQL de Synapse

En las secciones siguientes encontrará varias sugerencias sobre el uso de esquemas definidos por el usuario de T-SQL para desarrollar soluciones en SQL de Synapse.

## <a name="schemas-for-application-boundaries"></a>Esquemas para los límites de la aplicación

La arquitectura de análisis tradicional suele usar bases de datos independientes para crear los límites de la aplicación en función de la carga de trabajo, el dominio o la seguridad. Por ejemplo, una arquitectura de análisis de SQL Server tradicional podría incluir una base de datos provisional, una base de datos de análisis y algunas bases de datos data mart. En esta topología, cada base de datos funciona como una carga de trabajo y límite de seguridad de la arquitectura.

En su lugar, SQL de Synapse ejecuta toda la carga de trabajo de análisis en una base de datos. No se permiten las combinaciones entre bases de datos. SQL de Synapse espera que todas las tablas usadas por el almacenamiento se almacenen en una sola base de datos.

> [!NOTE]
> Los grupos de SQL no admiten ningún tipo de consultas entre bases de datos. En consecuencia, deben revisarse las implementaciones de análisis que utilizan este patrón. SQL a petición (versión preliminar) admite las consultas entre bases de datos.

## <a name="user-defined-schema-recommendations"></a>Recomendaciones para esquemas definidos por el usuario

Se incluyen recomendaciones para consolidar los límites de cargas de trabajo, seguridad, dominios y funcionalidad con esquemas definidos por el usuario:

- Use una base de datos para ejecutar toda la carga de trabajo de análisis.
- Consolide el entorno de análisis actual para usar una base de datos.
- Utilice **esquemas definidos por el usuario** para proporcionar el límite implementado anteriormente con las bases de datos.

Si anteriormente no se han usado los esquemas definidos por el usuario, comenzará desde cero. Use el nombre de la base de datos anterior como punto de partida para los esquemas definidos por el usuario en la base de datos de SQL de Synapse.

Si ya se han usado los esquemas, tiene algunas opciones:

- Quitar los nombres de esquemas heredados y empezar desde cero.
- Conservar los nombres de esquemas heredados al anteponer el nombre del esquema heredado al nombre de la tabla.
- Conservar los nombres de esquemas heredados mediante la implementación de vistas sobre la tabla en un esquema adicional, de modo que se vuelve a crear la estructura del esquema anterior.

> [!NOTE]
> En la primera inspección, la opción 3 puede resultar la más atractiva. Las vistas son de solo lectura en SQL de Synapse. Cualquier modificación de datos o tablas tendría que realizarse según la tabla de base. La opción 3 también introduce una capa de vistas en el sistema. Debería considerar esta opción si ya usa vistas en la arquitectura.
> 
> 

### <a name="examples"></a>Ejemplos

Implementar los esquemas definidos por el usuario en función de los nombres de base de datos.

```sql
CREATE SCHEMA [stg]; -- stg previously database name for staging database
GO
CREATE SCHEMA [edw]; -- edw previously database name for the analytics
GO
CREATE TABLE [stg].[customer] -- create staging tables in the stg schema
(       CustKey BIGINT NOT NULL
,       ...
);
GO
CREATE TABLE [edw].[customer] -- create analytics tables in the edw schema
(       CustKey BIGINT NOT NULL
,       ...
);
```

Mantener los nombres de los esquemas heredados al anteponerlos al nombre de la tabla. Utilizar esquemas para el límite de carga de trabajo.

```sql
CREATE SCHEMA [stg]; -- stg defines the staging boundary
GO
CREATE SCHEMA [edw]; -- edw defines the analytics boundary
GO
CREATE TABLE [stg].[dim_customer] --pre-pend the old schema name to the table and create in the staging boundary
(       CustKey BIGINT NOT NULL
,       ...
);
GO
CREATE TABLE [edw].[dim_customer] --pre-pend the old schema name to the table and create in the analytics boundary
(       CustKey BIGINT NOT NULL
,       ...
);
```

Conservar los nombres de los esquemas heredados mediante vistas.

```sql
CREATE SCHEMA [stg]; -- stg defines the staging boundary
GO
CREATE SCHEMA [edw]; -- stg defines the analytics boundary
GO
CREATE SCHEMA [dim]; -- edw defines the legacy schema name boundary
GO
CREATE TABLE [stg].[customer] -- create the base staging tables in the staging boundary
(       CustKey    BIGINT NOT NULL
,       ...
)
GO
CREATE TABLE [edw].[customer] -- create the base analytics tables in the analytics boundary
(       CustKey    BIGINT NOT NULL
,       ...
)
GO
CREATE VIEW [dim].[customer] -- create a view in the legacy schema name boundary for presentation consistency purposes only
AS
SELECT  CustKey
,       ...
FROM    [edw].customer
;
```

> [!NOTE]
> Cualquier cambio en la estrategia de esquema requiere de una revisión del modelo de seguridad de la base de datos. En muchos casos puede simplificar el modelo de seguridad mediante la asignación de permisos a nivel de esquema.

Si se requieren permisos más específicos, puede usar los roles de base de datos. Para obtener más información acerca de los roles de base de datos, consulte [Administración de usuarios y roles de base de datos](../../analysis-services/analysis-services-database-users.md).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más sugerencias sobre desarrollo, consulte la [información general sobre desarrollo de SQL de Synapse](develop-overview.md).
