---
title: Solución de problemas de Synapse SQL
description: Solucione problemas de Synapse SQL en Azure Synapse Analytics.
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 02/04/2019
ms.author: kevin
ms.reviewer: jrasnick
ms.custom: azure-synapse
ms.openlocfilehash: bce5e14db133ad55a2035c0c8074486ed1b8a6ec
ms.sourcegitcommit: 4a7a4af09f881f38fcb4875d89881e4b808b369b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89460514"
---
# <a name="troubleshooting-synapse-sql-in-azure-synapse-analytics"></a>Solución de problemas de Synapse SQL en Azure Synapse Analytics

En este artículo se enumeran incidencias de solución de problemas en Synapse SQL.

## <a name="connecting"></a>Connecting

| Incidencia                                                        | Resolución                                                   |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Error de inicio de sesión del usuario 'NT AUTHORITY\ANONYMOUS LOGON'. (Microsoft SQL Server, Error: 18456) | Este error se produce cuando un usuario de Azure AD intenta conectarse a la base de datos maestra, pero no tiene un usuario en esta.  Para corregir este problema, especifique el grupo de SQL al que desea conectarse en el momento de la conexión o agregue el usuario a la base de datos maestra.  Consulte el artículo [Información general sobre seguridad](sql-data-warehouse-overview-manage-security.md) para más detalles. |
| La entidad de seguridad del servidor "MyUserName" no puede obtener acceso a la base de datos "maestra" en el contexto de seguridad actual. No se puede abrir la base de datos predeterminada del usuario. Error de inicio de sesión. Error de inicio de sesión del usuario 'MyUserName'. (Microsoft SQL Server, Error: 916) | Este error se produce cuando un usuario de Azure AD intenta conectarse a la base de datos maestra, pero no tiene un usuario en esta.  Para corregir este problema, especifique el grupo de SQL al que desea conectarse en el momento de la conexión o agregue el usuario a la base de datos maestra.  Consulte el artículo [Información general sobre seguridad](sql-data-warehouse-overview-manage-security.md) para más detalles. |
| Error CTAIP                                                  | Este error puede producirse cuando se ha creado un inicio de sesión en la base de datos maestra de SQL Database, pero no en la base de datos SQL específica.  Si se produce este error, eche un vistazo al artículo sobre la [información general de seguridad](sql-data-warehouse-overview-manage-security.md) .  En este artículo se explica cómo crear un inicio de sesión y un usuario en la base de datos maestra y cómo crear un usuario en una base de datos SQL. |
| Bloqueado por el firewall                                          | Los grupos de SQL están protegidos por firewalls para garantizar que solo las direcciones IP conocidas tengan acceso a una base de datos. Los firewalls están protegidos de manera predeterminada, lo que significa que debe habilitar explícitamente una dirección IP o un intervalo de direcciones para poder conectarse.  Para configurar el firewall para el acceso, siga los pasos de la sección de [configuración del acceso de nivel de firewall para el cliente IP](create-data-warehouse-portal.md) en las [instrucciones de aprovisionamiento](create-data-warehouse-portal.md). |
| No se puede conectar con una herramienta o un controlador                           | El grupo de SQL de Synapse recomienda usar [SSMS](/sql/ssms/download-sql-server-management-studio-ssms?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest), [SSDT para Visual Studio](sql-data-warehouse-install-visual-studio.md) o [sqlcmd](sql-data-warehouse-get-started-connect-sqlcmd.md) para consultar los datos. Para más información sobre los controladores y la conexión a Azure Synapse, consulte los artículos [Controladores para Azure Synapse](sql-data-warehouse-connection-strings.md) y [Conexión a Azure Synapse](sql-data-warehouse-connect-overview.md). |

## <a name="tools"></a>Herramientas

| Incidencia                                                        | Resolución                                                   |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| El Explorador de objetos de Visual Studio no muestra usuarios de Azure AD           | Este es un problema conocido.  Como solución alternativa, vea los usuarios de [sys.database_principals](/sql/relational-databases/system-catalog-views/sys-database-principals-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).  Consulte [Autenticación en Azure Synapse](sql-data-warehouse-authentication.md) para más información sobre el uso de Azure Active Directory con un grupo de SQL de Synapse. |
| El scripting manual, mediante el Asistente para scripting, o la conexión a través de SSMS es lenta, no responde o genera errores | Asegúrese de que se han creado usuarios en la base de datos maestra. En las opciones de scripting, asegúrese también de que la edición del motor se ha establecido como "Microsoft Azure Synapse Analytics Edition" y que el tipo de motor es "Microsoft Azure SQL Database". |
| Errores de generación de scripts en SSMS                               | La generación de un script para un grupo de SQL de Synapse produce un error si la opción "Generar script para objetos dependientes" está establecida en "True". Como solución alternativa, los usuarios deben ir manualmente a **Herramientas -> Opciones ->Explorador de objetos de SQL Server -> opción "Generar script para objetos dependientes" y establecerla en False**. |

## <a name="performance"></a>Rendimiento

| Incidencia                                                        | Resolución                                                   |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Solución de problemas de rendimiento de consultas                            | Si está intentando solucionar los problemas de una consulta determinada, comience por [aprender a supervisar las consultas](sql-data-warehouse-manage-monitor.md#monitor-query-execution). |
| Problemas de espacio de TempDB | [Supervise el uso del espacio de TempDB](sql-data-warehouse-manage-monitor.md#monitor-tempdb).  Las causas comunes para quedarse sin espacio de TempDB son:<br>- No hay suficientes recursos asignados a la consulta, lo que provoca que los datos se derramen en TempDB.  Consulte [Administración de cargas de trabajos](resource-classes-for-workload-management.md) <br>- Las estadísticas faltan o no están actualizadas, lo que provoca un movimiento de datos excesivo.  Para más información sobre cómo crear estadísticas, vea [Mantenimiento de estadísticas de tablas](sql-data-warehouse-tables-statistics.md).<br>- El espacio de TempDB se asigna por nivel de servicio.  El [escalado del grupo de SQL](sql-data-warehouse-manage-compute-overview.md#scaling-compute) a una configuración de DWU superior asigna más espacio de TempDB.|
| Un bajo rendimiento de las consultas y unos planes mal diseñados suelen ser el resultado de la falta de estadísticas | La causa más común del rendimiento ineficiente es la falta de estadísticas en las tablas.  Para obtener más información sobre cómo crear estadísticas y por qué son tan importantes para el rendimiento, vea [Mantenimiento de estadísticas de tablas](sql-data-warehouse-tables-statistics.md). |
| Baja simultaneidad o consultas en cola                             | Para comprender el modo de equilibrar la asignación de memoria con la simultaneidad, es importante entender la [administración de la carga de trabajo](resource-classes-for-workload-management.md) . |
| Implementación de procedimientos recomendados                              | El mejor lugar para empezar a aprender formas de mejorar el rendimiento de las consultas es el artículo [Procedimientos recomendados para grupos de SQL de Synapse](sql-data-warehouse-best-practices.md). |
| Mejora del rendimiento con el escalado                      | En ocasiones, la solución para mejorar el rendimiento consiste simplemente en agregar más potencia de proceso a las consultas mediante el [escalado del grupo de SQL](sql-data-warehouse-manage-compute-overview.md). |
| Bajo rendimiento de las consultas como resultado de poca calidad del índice     | A veces, la velocidad de las consultas se puede reducir debido a la [baja calidad del índice de almacén de columnas](sql-data-warehouse-tables-index.md#causes-of-poor-columnstore-index-quality).  Para más información al respecto y conocer el modo de [volver a generar los índices para mejorar la calidad del segmento](sql-data-warehouse-tables-index.md#rebuilding-indexes-to-improve-segment-quality), consulte este artículo. |

## <a name="system-management"></a>Administración del sistema

| Incidencia                                                        | Resolución                                                   |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Mens. 40847: No se pudo realizar la operación porque el servidor superaría la cuota de la unidad de transacción de la base de datos permitida de 45000. | Reduzca la unidad [DWU](what-is-a-data-warehouse-unit-dwu-cdwu.md) de la base de datos que intenta crear o [ solicite un aumento de la cuota](sql-data-warehouse-get-started-create-support-ticket.md). |
| Investigación del uso del espacio                              | Consulte los [tamaños de tabla](sql-data-warehouse-tables-overview.md#table-size-queries) para comprender el uso del espacio del sistema. |
| Ayuda con la administración de tablas                                    | Para obtener ayuda con la administración de las tablas, vea la [información general sobre las tablas](sql-data-warehouse-tables-overview.md).  Este artículo también incluye vínculos a temas más detallados como [tipos de datos de tabla](sql-data-warehouse-tables-data-types.md), [distribución de una tabla](sql-data-warehouse-tables-distribute.md), [indexación de una tabla](sql-data-warehouse-tables-index.md), [creación de particiones de una tabla](sql-data-warehouse-tables-partition.md), [mantenimiento de estadísticas de tabla](sql-data-warehouse-tables-statistics.md) y [tablas temporales](sql-data-warehouse-tables-temporary.md). |
| La barra de progreso de Cifrado de datos transparente (TDE) no se actualiza en Azure Portal | El estado de TDE se puede ver a través de [PowerShell](/powershell/module/az.sql/get-azsqldatabasetransparentdataencryption?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). |

## <a name="differences-from-sql-database"></a>Diferencias con respecto a SQL Database

| Incidencia                                 | Resolución                                                   |
| :------------------------------------ | :----------------------------------------------------------- |
| Características de SQL Database no admitidas     | Consulte [Características no compatibles de las tablas](sql-data-warehouse-tables-overview.md#unsupported-table-features). |
| Tipos de datos de SQL Database no admitidos   | Consulte [Tipos de datos no admitidos](sql-data-warehouse-tables-data-types.md#identify-unsupported-data-types).        |
| Limitaciones de DELETE y UPDATE         | Consulte las [soluciones alternativas para UPDATE](sql-data-warehouse-develop-ctas.md#ansi-join-replacement-for-update-statements), las [soluciones alternativas para DELETE](sql-data-warehouse-develop-ctas.md#ansi-join-replacement-for-delete-statements) y el [uso de CTAS para resolver la sintaxis de UPDATE y DELETE no admitida](sql-data-warehouse-develop-ctas.md). |
| No se admite la instrucción MERGE      | Consulte las [soluciones alternativas para MERGE](sql-data-warehouse-develop-ctas.md#replace-merge-statements).                  |
| Limitaciones de procedimientos almacenados          | Consulte [Limitaciones de procedimientos almacenados](sql-data-warehouse-develop-stored-procedures.md#limitations) para conocer algunas de las limitaciones de los procedimientos almacenados. |
| Los UDF no admiten instrucciones SELECT | Se trata de una limitación actual de nuestros UDF.  Consulte [CREATE FUNCTION](/sql/t-sql/statements/create-function-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) para comprobar la sintaxis que se admite. |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más ayuda en la búsqueda de soluciones para su problema, aquí tiene algunos otros recursos que puede probar.

* [Blogs](https://azure.microsoft.com/blog/tag/azure-sql-data-warehouse/)
* [Solicitud de función](https://feedback.azure.com/forums/307516-sql-data-warehouse)
* [Vídeos](https://azure.microsoft.com/documentation/videos/index/?services=sql-data-warehouse)
* [Creación de una incidencia de soporte técnico](sql-data-warehouse-get-started-create-support-ticket.md)
* [Página de preguntas y respuestas de Microsoft](https://docs.microsoft.com/answers/topics/azure-synapse-analytics.html)
* [Foro Stack Overflow](https://stackoverflow.com/questions/tagged/azure-sqldw)
* [Twitter](https://twitter.com/hashtag/SQLDW)
