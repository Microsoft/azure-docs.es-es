---
title: Artículo sobre la conocida solución de problemas de errores o problemas comunes asociados al uso de Azure Database Migration Service | Microsoft Docs
description: Obtenga información sobre cómo solucionar problemas comunes conocidos problemas/errores asociados con el uso de Azure Database Migration Service.
services: database-migration
author: HJToland3
ms.author: jtoland
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: mvc
ms.topic: article
ms.date: 05/14/2019
ms.openlocfilehash: dc8ba315d08f3a130ff0adf91afc90f545baf4e4
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65604422"
---
# <a name="troubleshoot-common-azure-database-migration-service-issues-and-errors"></a>Solucionar problemas de Azure Database Migration Service y errores comunes

En este artículo se describe algunos problemas comunes y errores que pueden experimentar los usuarios de Azure Database Migration Service. El artículo también incluye información sobre cómo resolver estos problemas y errores.

## <a name="migration-activity-in-queued-state"></a>Actividad de migración de estado en cola

Al crear nuevas actividades en un proyecto de Azure Database Migration Service, las actividades permanecen en un estado en cola.

| Causa         | Resolución |
| ------------- | ------------- |
| Este problema se produce cuando la instancia de Azure Database Migration Service ha alcanzado la capacidad máxima para las tareas en curso que se ejecutan simultáneamente. Cualquier nueva actividad se pone en cola hasta que la capacidad esté disponible. | Validar la instancia tiene actividades en ejecución en proyectos de Data Migration Service. Puede seguir crear actividades nuevas que se agregan automáticamente a la cola para su ejecución. Tan pronto como complete cualquiera de las actividades de ejecución existentes, la siguiente actividad en cola empieza a ejecutarse y el estado cambia a estado de ejecución automáticamente. No es necesario realizar ninguna acción adicional para iniciar la migración de la actividad en cola.<br> |

## <a name="max-number-of-databases-selected-for-migration"></a>Número máximo de bases de datos seleccionadas para la migración

El siguiente error se produce cuando la instancia administrada de creación de una actividad para un proyecto de migración de base de datos para migrar a Azure SQL Database o Azure SQL Database:

* **Error**: Error de validación de la configuración de migración","errorDetail":"objetos de número máximo de más de '4' de 'Databases' se ha seleccionado para la migración."

| Causa         | Resolución |
| ------------- | ------------- |
| Este error se muestra cuando haya seleccionado más de cuatro bases de datos para una actividad de migración. En este momento, cada actividad de migración está limitada a cuatro bases de datos. | Seleccione las bases de datos de cuatro o menos por actividad de migración. Si tiene que migrar más de cuatro bases de datos en paralelo, aprovisionar otra instancia de Azure Database Migration Service. Actualmente, cada suscripción admite hasta dos instancias de Azure Database Migration Service.<br><br> |

## <a name="errors-for-mysql-migration-to-azure-mysql-with-recovery-failures"></a>Errores de migración de MySQL a Azure MySQL con errores de recuperación

Al migrar desde MySQL a Azure Database for MySQL mediante Azure Database Migration Service, la actividad de migración genera el error siguiente:

* **Error**: Error de migración de base de datos - tarea 'TaskID' se ha suspendido debido a errores de recuperación sucesivos de [n].

| Causa         | Resolución |
| ------------- | ------------- |
| Este error puede producirse cuando el usuario que realiza la migración no tiene el rol ReplicationAdmin o privilegios del cliente de la replicación, la réplica de replicación y SUPER (versiones anteriores a MySQL 5.6.6).<br> <br><br><br> <br> <br> <br> <br> <br> <br> | Asegúrese de que el [privilegios previos](https://docs.microsoft.com/azure/dms/tutorial-mysql-azure-mysql-online#prerequisites) para el usuario cuenta están configuradas con precisión en la base de datos de Azure para la instancia de MySQL. Por ejemplo, se pueden seguir los pasos siguientes para crear un usuario denominado 'migrateuser' con los privilegios necesarios:<br>1. Crear usuario migrateuser@'%' identificado por 'secret'; <br>2. Conceder todos los privilegios en db_name.* a 'migrateuser'@'%' identificado por 'secret'; Repita este paso para conceder acceso en varias bases de datos <br>3. Subordinado de la replicación de concesión en *.* para 'migrateuser'@'%' identificado por 'secret';<br>4. Cliente de replicación de concesión en *.* para 'migrateuser'@'%' identificado por 'secret';<br>5. Privilegios de vaciado; |

## <a name="error-when-attempting-to-stop-azure-database-migration-service"></a>Error al intentar detener el servicio de migración de base de datos de Azure

Recibe el siguiente error al detener la instancia de Azure Database Migration Service:

* **Error**: Servicio no se pudo detener. Error: {"error": {'code': 'InvalidRequest', 'message':' se están ejecutando actualmente una o más actividades. Para detener el servicio, espere hasta que las actividades han completado o deténgalas manualmente y vuelva a intentarlo. "}}

| Causa         | Resolución |
| ------------- | ------------- |
| Este error se muestra cuando la instancia de servicio que está intentando detener incluye actividades que siguen en ejecución o presentan en los proyectos de migración. <br><br><br><br><br><br> | Asegúrese de que no hay ninguna actividad que se ejecuta en la instancia de Azure Database Migration Service está intentando detener. También puede eliminar las actividades o proyectos antes de intentar detener el servicio. Los siguientes pasos muestran cómo quitar proyectos para limpiar la instancia de servicio de migración mediante la eliminación de todas las tareas en ejecución:<br>1. Install-Module-nombre AzureRM.DataMigration <br>2. Login-AzureRmAccount <br>3. SELECT-AzureRmSubscription - SubscriptionName "<subName>" <br> 4. Remove-AzureRmDataMigrationProject -Name <projectName> -ResourceGroupName <rgName> -ServiceName <serviceName> -DeleteRunningTask |

## <a name="error-restoring-database-while-migrating-sql-to-azure-sql-db-managed-instance"></a>Error al restaurar la base de datos mientras la instancia administrada de SQL migrar a Azure SQL DB

Al realizar una migración en línea de SQL Server a una instancia administrada de Azure SQL Database, la transición se produce un error con el siguiente error:

* **Error**: Error de operación "operationId" identificador de operación de restauración. El código 'AuthorizationFailed', mensaje "el cliente 'clientId' con Id. de objeto 'objectId' no tiene autorización para realizar la acción 'Microsoft.Sql/locations/managedDatabaseRestoreAzureAsyncOperation/read' en el ámbito ' / Subscriptions / subscriptionId'. '.

| Causa         | Resolución    |
| ------------- | ------------- |
| Este error indica que la entidad de seguridad de la aplicación que se va a usar para la migración en línea de SQL Server a una instancia administrada de Azure SQL Database no tenga permiso en la suscripción de colaboración. Ciertas llamadas de API con instancia administrada en la actualidad requieren este permiso en la suscripción para la operación de restauración. <br><br><br><br><br><br><br><br><br><br> | Use la `Get-AzureADServicePrincipal` cmdlet de PowerShell con `-ObjectId` disponible en el mensaje de error para mostrar el nombre para mostrar del identificador de aplicación que se va a usar.<br><br> Validar los permisos para esta aplicación y asegurarse de que tiene el [rol Colaborador](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#contributor) en el nivel de suscripción. <br><br> Está trabajando el equipo de ingeniería de servicio de Azure Database Migration para restringir la requiere acceso actual contribuir rol en la suscripción. Si tiene un requisito empresarial que no permite el uso de contribuir con el rol, póngase en contacto con soporte técnico de Azure para obtener ayuda adicional. |

## <a name="error-when-deleting-nic-associated-with-azure-database-migration-service"></a>Error al eliminar la NIC asociada con Azure Database Migration Service

Al intentar eliminar una tarjeta de interfaz de red asociado con Azure Database Migration Service, el intento de eliminación se produce este error:

* **Error**: No se puede eliminar la NIC asociada a Azure Database Migration Service debido a que el servicio DMS utilizando la NIC

| Causa         | Resolución    |
| ------------- | ------------- |
| Este problema se produce cuando la instancia de Azure Database Migration Service todavía puede estar presente y consumo de la NIC. <br><br><br><br><br><br> | Para eliminar esta NIC, elimine la instancia de servicio DMS que elimina la NIC usada por el servicio de forma automática.<br><br> **Importante**: Asegúrese de que la instancia de Azure Database Migration Service que se va a eliminar no tiene ninguna actividad de ejecución.<br><br> Después de eliminarán todos los proyectos y las actividades asociadas a la instancia de Azure Database Migration Service, puede eliminar la instancia de servicio. La NIC usada por la instancia de servicio se limpia automáticamente como parte de la eliminación del servicio. |

## <a name="connection-error-when-using-expressroute"></a>Error de conexión al usar ExpressRoute

Al intentar conectarse al origen en el Asistente para proyecto de Azure Database Migration service, la conexión produce un error tras el tiempo de espera largos si el origen está usando ExpressRoute para la conectividad.

| Causa         | Resolución    |
| ------------- | ------------- |
| Cuando se usa [ExpressRoute](https://azure.microsoft.com/services/expressroute/), Azure Database Migration Service [requiere](https://docs.microsoft.com/azure/dms/tutorial-sql-server-azure-sql-online) tres puntos de conexión de servicio en la subred de red Virtual asociada con el servicio de aprovisionamiento:<br> : Punto de conexión del Bus de servicio<br> : Punto de conexión de almacenamiento<br> --Punto de conexión de base de datos (por ejemplo, el punto de conexión SQL, el punto de conexión de Cosmos DB) de destino<br><br><br><br> | [Habilitar](https://docs.microsoft.com/azure/dms/tutorial-sql-server-azure-sql-online) los puntos de conexión de servicio necesaria para la conectividad de ExpressRoute entre el origen y Azure Database Migration Service. <br><br><br><br><br><br><br><br> |

## <a name="timeout-error-when-migrating-a-mysql-database-to-azure-mysql"></a>Error de tiempo de espera al migrar una base de datos MySQL a Azure para MySQL

Al migrar una base de datos MySQL a una base de datos de Azure para la instancia de MySQL a través de Azure Database Migration Service, la migración se produce un error con el siguiente error de tiempo de espera:

* **Error**: Error de migración de base de datos: no se pudo cargar el archivo: no se pudo iniciar el proceso de carga de archivo "n" RetCode: SQL_ERROR SqlState: NativeError HY000: Mensaje 1205: [MySQL] [controlador ODBC] [mysqld] espera de bloqueo de tiempo de espera superado; Pruebe a reiniciar la transacción

| Causa         | Resolución    |
| ------------- | ------------- |
| Este error se produce cuando migración debido a la espera de bloqueo durante la migración.<br><br> | Considere la posibilidad de aumentar el valor del parámetro de servidor **'innodb_lock_wait_timeout'**. El valor máximo permitido es 1073741824. |

## <a name="additional-known-issues"></a>Otros problemas conocidos

* [Limitaciones de migración y problemas conocidos con las migraciones en línea a Azure SQL Database](https://docs.microsoft.com/azure/dms/known-issues-azure-sql-online)
* [Limitaciones de migración y problemas conocidos con migraciones en línea a Azure Database for MySQL](https://docs.microsoft.com/azure/dms/known-issues-azure-mysql-online)
* [Limitaciones de migración y problemas conocidos con las migraciones en línea a la base de datos de Azure Database for PostgreSQL](https://docs.microsoft.com/azure/dms/known-issues-azure-postgresql-online)

## <a name="next-steps"></a>Pasos siguientes

* Ver el artículo [PowerShell de Azure Database Migration Service](https://docs.microsoft.com/powershell/module/azurerm.datamigration/?view=azurermps-6.13.0#data_migration).
* Ver el artículo [cómo configurar los parámetros del servidor de base de datos de Azure para MySQL mediante el portal de Azure](https://docs.microsoft.com/azure/mysql/howto-server-parameters).
* Ver el artículo [información general sobre los requisitos previos para usar Azure Database Migration Service](https://docs.microsoft.com/azure/dms/pre-reqs).
* Consulte la [preguntas más frecuentes sobre el uso de Azure Database Migration Service](https://docs.microsoft.com/azure/dms/faq).
