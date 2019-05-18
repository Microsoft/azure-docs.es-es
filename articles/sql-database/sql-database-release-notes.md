---
title: Notas de la base de datos SQL de Azure | Microsoft Docs
description: Obtenga información sobre las nuevas características y mejoras en el servicio de Azure SQL Database y en la documentación de Azure SQL Database
services: sql-database
author: stevestein
manager: craigg
ms.service: sql-database
ms.subservice: service
ms.devlang: ''
ms.topic: conceptual
ms.date: 05/15/2019
ms.author: sstein
ms.openlocfilehash: d527c4fed9c43e62d815078c049d4d8e6f8a46b7
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65787716"
---
# <a name="sql-database-release-notes"></a>Notas de la base de datos SQL

En este artículo se enumera las nuevas características y mejoras en el servicio de base de datos SQL y en la documentación de la base de datos SQL. Para mejorar el servicio SQL Database, consulte [actualizaciones del servicio de base de datos SQL](https://azure.microsoft.com/updates/?product=sql-database). Mejoras a otros servicios de Azure, consulte [las actualizaciones del servicio](https://azure.microsoft.com/updates).

## <a name="features-in-public-preview"></a>Características en versión preliminar pública

| Característica | Detalles |
| ---| --- |
| Recuperación de base de datos acelerada con bases de datos únicas y grupos elásticos | Para obtener información, consulte [acelerado de recuperación de base de datos](sql-database-accelerated-database-recovery.md).|
|Recuento aproximado de valores distintos|Para obtener información, consulte [aproximado Count Distinct](https://docs.microsoft.com/sql/relational-databases/performance/intelligent-query-processing#approximate-query-processing).|
|Modo por lotes en el almacén de filas (en el nivel de compatibilidad 150)|Para obtener información, consulte [modo por lotes en el almacén de filas](https://docs.microsoft.com/sql/relational-databases/performance/intelligent-query-processing#batch-mode-on-rowstore).|
| Clasificación y detección de datos  |Para obtener información, consulte [clasificación y detección de datos de Azure SQL Database y SQL Data Warehouse](sql-database-data-discovery-and-classification.md).|
| Trabajos de base de datos elásticos | Para obtener información, consulte [crear, configurar y administrar trabajos elásticos](elastic-jobs-overview.md). |
| Consultas elásticas | Para obtener información, consulte [información general de la consulta elástica](sql-database-elastic-query-overview.md). |
| Transacciones elásticas | [Las transacciones distribuidas entre bases de datos en la nube](sql-database-elastic-transactions-overview.md). |
| Intercalación de la instancia con instancias administradas |Para obtener información, consulte [usar PowerShell con la plantilla de Azure Resource Manager para crear una instancia administrada en Azure SQL Database](./scripts/sql-managed-instance-create-powershell-azure-resource-manager-template.md).|
|Comentarios de concesión de memoria (modo de fila) (en el nivel de compatibilidad 150)|Para obtener información, consulte [comentarios de concesión de memoria (modo de fila)](https://docs.microsoft.com/sql/relational-databases/performance/intelligent-query-processing#row-mode-memory-grant-feedback).|
| Editor de consultas en el portal de Azure |Para obtener información, consulte [usar el editor de consultas SQL del portal de Azure para conectarse y consultar datos](sql-database-connect-query-portal.md).|
| R services / machine learning con bases de datos únicas y grupos elásticos |Para obtener información, consulte [Machine Learning Services en Azure SQL Database](https://docs.microsoft.com/sql/advanced-analytics/what-s-new-in-sql-server-machine-learning-services?view=sql-server-2017#machine-learning-services-in-azure-sql-database).|
| Volver a crear las bases de datos con las instancias administradas |Para obtener información, consulte [volver a crear puede quitar las bases de datos en Azure SQL Managed Instance](https://medium.com/azure-sqldb-managed-instance/re-create-dropped-databases-in-azure-sql-managed-instance-dc369ed60266).|
| Replicación con las instancias administradas |Para obtener información, consulte [configurar la replicación en una base de datos de instancia administrada de Azure SQL Database](replication-with-sql-database-managed-instance.md).|
| Nivel de servicio de informática sin servidor | Para obtener información, consulte [SQL Database sin servidor (versión preliminar)](sql-database-serverless.md).|
|SQL Analytics|Para obtener información, consulte [Azure SQL Analytics](../azure-monitor/insights/azure-sql.md).|
|Compilación diferida Variable tabla (en el nivel de compatibilidad 150)|Para obtener información, consulte [compilación diferida Variable de tabla](https://docs.microsoft.com/sql/relational-databases/performance/intelligent-query-processing#table-variable-deferred-compilation).|
| Detección de amenazas con las instancias administradas |Para obtener información, consulte [instancia administrada de configurar la detección de amenazas en Azure SQL Database](sql-database-managed-instance-threat-detection.md).|
| Compatibilidad de zona horaria para las instancias administradas|Para obtener más información, consulte [zona horaria en la instancia administrada de Azure SQL Database](sql-database-managed-instance-timezone.md).|
| Cifrado de datos transparente (TDE) con Bring Your Own Key (BYOK) con las instancias administradas |Para obtener información, consulte [Azure SQL Transparent Data Encryption with claves administradas por el cliente en Azure Key Vault: Poner el soporte técnico de su propia clave](transparent-data-encryption-byok-azure-sql.md).|
| &nbsp; |

## <a name="may-2019"></a>Mayo de 2019

### <a name="service-improvements"></a>Mejoras en el servicio

| Mejoras en el servicio | Detalles |
| --- | --- |
|Nivel de servicio de hiperescala publicado para disponibilidad general| Para obtener más información, consulte [a gran escala nivel de servicio para hasta 100 TB](sql-database-service-tier-hyperscale.md) y [obtener alto rendimiento escalado para las cargas de trabajo de la base de datos de Azure con hiperescala](https://azure.microsoft.com/blog/get-high-performance-scaling-for-your-azure-database-workloads-with-hyperscale/).|
|Nivel de proceso sin servidor publicado para la versión preliminar pública | Para obtener más información, consulte [SQL Database sin servidor (versión preliminar)](sql-database-serverless.md).|
| Aumentaron las tasas de registro de transacciones y las IOPS de destino para bases de datos mediante el modelo de compra basado en núcleos virtuales| Para obtener más información, consulte [límites de recursos de bases de datos únicas mediante el modelo de compra basado en núcleos virtuales](https://docs.microsoft.com/azure/sql-database/sql-database-vcore-resource-limits-single-databases) y [límites de recursos de bases de datos únicas mediante el modelo de compra basado en DTU](https://docs.microsoft.com/azure/sql-database/sql-database-dtu-resource-limits-single-databases).
| &nbsp; |

### <a name="documentation-improvements"></a>Mejoras en la documentación

| Mejoras en la documentación | Detalles |
| --- | --- |
| Docs de nivel de servicio a gran escala que se actualizó con la versión de GA| Para obtener más información, consulte [a gran escala nivel de servicio para hasta 100 TB](sql-database-service-tier-hyperscale.md).|
|Docs de nivel de proceso sin servidor publicados con la versión preliminar pública| Para obtener más información, consulte [SQL Database sin servidor (versión preliminar)](sql-database-serverless.md).|
| &nbsp; |

## <a name="april-2019"></a>Abril de 2019

### <a name="service-improvements"></a>Mejoras en el servicio

| Mejoras en el servicio | Detalles |
| --- | --- |
| Puntos de conexión públicos para la instancia administrada en versión preliminar pública| Para obtener más información, consulte [uso de Azure SQL Database administra la instancia de forma segura con el punto de conexión público](sql-database-managed-instance-public-endpoint-securely.md).|
| Compatibilidad de zona horaria para la instancia en versión preliminar pública de administrada| Para obtener más información, consulte [zona horaria en la instancia administrada de Azure SQL Database](sql-database-managed-instance-timezone.md).|
| Proteger bases de datos de SQL Azure con identidades administradas en la segunda versión preliminar pública de versión| Consulte [es ahora más sencillo de la protección de Azure SQL Database con identidades administradas](https://azure.microsoft.com/blog/securing-azure-sql-databases-with-managed-identities-just-got-easier/).|
| &nbsp; |

### <a name="documentation-improvements"></a>Mejoras en la documentación

| Mejoras en la documentación | Detalles |
| --- | --- |
| Puntos de conexión públicos para la instancia administrada en versión preliminar pública| Para obtener más información, consulte [uso de Azure SQL Database administra la instancia de forma segura con el punto de conexión público](sql-database-managed-instance-public-endpoint-securely.md).|
| Compatibilidad de zona horaria para la instancia en versión preliminar pública de administrada| Para obtener más información, consulte [zona horaria en la instancia administrada de Azure SQL Database](sql-database-managed-instance-timezone.md). |
| Regulación de recursos en Azure SQL Database | Para obtener más información, consulte [regulación de recursos en Azure SQL Database](https://azure.microsoft.com/blog/resource-governance-in-azure-sql-database/). || &nbsp; |

## <a name="march-2019"></a>Marzo de 2019

### <a name="service-improvements"></a>Mejoras en el servicio

| Mejoras en el servicio | Detalles |
| --- | --- |
| Escalado horizontal de lectura para publicado para disponibilidad general de Azure SQL Database | Para obtener más información, consulte [escalado horizontal de lectura](sql-database-read-scale-out.md).|
| &nbsp; |

### <a name="documentation-improvements"></a>Mejoras en la documentación

| Mejoras en la documentación | Detalles |
| --- | --- |
| Límites del registro se ha agregado para bases de datos únicas|Para obtener más información, consulte [límites de recursos de memoria con núcleo virtual de base de datos únicas](sql-database-vcore-resource-limits-single-databases.md).|
| Límites del registro se ha agregado para grupos elásticos y bases de datos agrupadas|Para obtener más información, consulte [los límites de recursos de memoria con núcleo virtual de los grupos elásticos](sql-database-vcore-resource-limits-elastic-pools.md).|
| Se ha agregado gobierno de tasa de registro transacciones| Agrega contenido nuevo para [gobierno de tasa de registro de transacciones](sql-database-resource-limits-database-server.md#transaction-log-rate-governance).|
| Ejemplos de PowerShell actualizados para bases de datos únicas y grupos elásticos para usar el módulo az.sql | Para obtener más información, consulte [ejemplos de PowerShell para grupos elásticos y bases de datos únicas](sql-database-powershell-samples.md#single-database-and-elastic-pools).|
| &nbsp; |

## <a name="february-2019"></a>Febrero de 2019

### <a name="service-improvements"></a>Mejoras en el servicio

| Mejoras en el servicio | Detalles |
| --- | --- |
|Crear un índice en línea reanudable ahora está disponible con carácter general| Para obtener más información, consulte [Create Index](https://docs.microsoft.com/sql/t-sql/statements/create-index-transact-sql).|
|Compatibilidad con instancia administrada ha mejorado de las tablas de rutas| Para obtener más información, consulte [requisitos de red](sql-database-managed-instance-connectivity-architecture.md#network-requirements).|
|Cambio de nombre de base de datos compatible con instancia administrada | Para obtener más información, consulte el [ALTER DATABASE](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-mi-current) y [sp_rename](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-rename-transact-sql) sintaxis.|
|Base de datos de SQL como origen de datos de referencia para Stream Analytics. | Para obtener más información, consulte [Stream Analytics](https://azure.microsoft.com/services/stream-analytics/) y [Azure Stream Analytics ahora es compatible con Azure SQL Database como entrada de datos de referencia](https://azure.microsoft.com/blog/azure-stream-analytics-now-supports-azure-sql-database-as-reference-data-input/).|
|Asistencia para la migración de datos agrega compatibilidad para la instancia administrada. |Para obtener más información, consulte [cuáles son las novedades en DMA](https://docs.microsoft.com/sql/dma/dma-whatsnew).|
|SQL Server Migration Assistant agrega compatibilidad para la evaluación de disponibilidad de destino para la instancia administrada. | Para obtener más información, consulte [SQL Server Migration Assistant](https://docs.microsoft.com/sql/ssma/sql-server-migration-assistant).
|Servicio de migración de datos admite la migración desde Amazon RDS a instancia administrada | Para más información, consulte [Tutorial: Migración de SQL Server de RDS a Azure SQL Database o en línea una instancia administrada de Azure SQL Database mediante DMS](../dms/tutorial-rds-sql-server-azure-sql-and-managed-instance-online.md).|
| &nbsp; |

### <a name="documentation-improvements"></a>Mejoras en la documentación

| Mejoras en la documentación | Detalles |
| --- | --- |
|Agregar administrados aclaraciones de opción de implementación de instancia|Actualizar muchos artículos para clarificar la aplicabilidad a la base de datos única, grupo elástico y las opciones de implementación instancia administrada. |
|Tamaños de tempdb actualizada para el modelo de compra basado en DTU | Para obtener más información, consulte [base de datos Tempdb en SQL Database](https://docs.microsoft.com/sql/relational-databases/databases/tempdb-database#tempdb-database-in-sql-database).|
|Actualizado importación y exportación con el archivo bacpac para la compatibilidad con instancia administrada| Para obtener más información, consulte [importar desde BACPAC](sql-database-import.md) y [exportar al BACPAC](sql-database-export.md). |
| &nbsp; |

## <a name="january-2019"></a>Enero de 2019

### <a name="service-improvements"></a>Mejoras en el servicio

| Mejoras en el servicio | Detalles |
| --- | --- |
| Opciones de granularidad adicional para los recursos de proceso | El propósito general y crítico para la empresa de servicio para los niveles de [bases de datos únicas](sql-database-vcore-resource-limits-single-databases.md) y [grupos elásticos](sql-database-vcore-resource-limits-elastic-pools.md) ahora tiene más opciones de proceso específica.|
| Ver los registros de auditoría para la instancia administrada en Azure portal | Ver [registros para las instancias administradas de auditoría](sql-database-managed-instance-auditing.md) en Azure portal ahora se admite.|
| Característica de detección de amenazas avanzada ha cambiado a seguridad avanzada de datos | Cambiar el nombre de característica de detección de amenazas avanzada para [seguridad avanzada de datos](sql-advanced-threat-protection.md) para instancias administradas, grupos elásticos y bases de datos únicas. |
| &nbsp; |

### <a name="documentation-improvements"></a>Mejoras en la documentación

| Mejoras en la documentación | Detalles |
| --- | --- |
| Las instancias administradas y la replicación transaccional | Se ha agregado artículo sobre el uso de [la replicación transaccional con las instancias administradas](replication-with-sql-database-managed-instance.md). |
| Se ha agregado a Azure AD con el tutorial de la instancia administrada | Esto [Azure AD con instancia administrada](sql-database-managed-instance-aad-security-tutorial.md) tendrá que configurar y probar el tutorial se muestra administra la seguridad de instancia mediante inicios de sesión de Azure AD. |
| Contenido actualizado de automatización de trabajos mediante scripts de Transact-SQL | Actualiza y se ha aclarado el contenido para el uso de [automatización del trabajo con scripts de Transact-SQL](sql-database-job-automation-overview.md) para instancias administradas, grupos elásticos y bases de datos únicas. |
| Contenido de seguridad para las instancias administradas actualizado | Actualiza y se ha aclarado el contenido de la [modelo de seguridad para las instancias administradas](sql-database-security-overview.md)y diferencias con el modelo de seguridad de bases de datos únicas y grupos elásticos. |
| Actualiza todos los inicios rápidos y tutoriales | Todos los inicios rápidos y tutoriales en la [documentación](https://docs.microsoft.com/azure/sql-database) se han actualizado y se actualiza para que coincida con los cambios en el portal de Azure. |
| Guías de inicio rápido se ha agregado información general | Agrega una guía de información general de la Guía de inicio rápido para [bases de datos únicas](sql-database-quickstart-guide.md) y [las instancias administradas](sql-database-managed-instance-quickstart-guide.md). |
| Glosario de base de datos SQL se ha agregado de términos | Esto [Glosario de términos](sql-database-glossary-terms.md) artículo proporciona una lista definitiva de los términos de base de datos SQL y vínculos a la página principal de conceptual que explica el término en el contexto. |
| &nbsp; |

## <a name="contribute-to-content-improvement"></a>Contribuir a la mejora de contenido

El conjunto de documentación de SQL Azure es código abierto. Trabajar en el cuadro Abrir proporciona varias ventajas:

- Repositorios de código abierto se planean para obtener comentarios sobre qué documentos son más necesarios.
- Repositorios de código abierto se revisan para publicar el contenido más útil en la primera versión.
- Repositorios de código abierto se actualizan para que sea más fácil mejorar continuamente el contenido.

Para contribuir al contenido de la documentación de Azure SQL Database, consulte el [Introducción a la Guía de colaboradores de Docs Microsoft](https://docs.microsoft.com/contribute/). La experiencia del usuario en [docs.microsoft.com](https://docs.microsoft.com/) integra [GitHub](https://github.com/) flujos de trabajo directamente para que sea más fácil. Empiece por [editando el documento que está viendo](https://docs.microsoft.com/contribute/#quick-edits-to-existing-documents). O bien, para ayudar al [revisar nuevos temas](https://docs.microsoft.com/contribute/#review-open-prs), o [crear problemas de calidad](https://docs.microsoft.com/contribute/#create-quality-issues).
