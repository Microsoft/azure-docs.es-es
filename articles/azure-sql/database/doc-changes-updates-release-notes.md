---
title: Novedades
titleSuffix: Azure SQL Database & SQL Managed Instance
description: Obtenga información sobre las nuevas características y mejoras de la documentación en Azure SQL Database y SQL Managed Instance.
services: sql-database
author: stevestein
ms.service: sql-db-mi
ms.subservice: service
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
ms.date: 06/17/2020
ms.author: sstein
ms.openlocfilehash: af19b72846c78ef80ba170b6d6e0cec97fa2b96e
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87533366"
---
# <a name="whats-new-in-azure-sql-database--sql-managed-instance"></a>Novedades de Azure SQL Database e Instancia administrada de SQL
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

En este artículo se enumeran las características de Azure SQL Database e Instancia administrada de SQL que se encuentran actualmente en versión preliminar pública. En el caso de las actualizaciones y mejoras de SQL Database e Instancia administrada de SQL, vea las [actualizaciones en los servicios SQL Database e Instancia administrada de SQL](https://azure.microsoft.com/updates/?product=sql-database). Para ver las actualizaciones y mejoras en otros servicios de Azure, consulte [actualizaciones de los servicios](https://azure.microsoft.com/updates).

## <a name="whats-new"></a>Novedades

La documentación sobre Azure SQL Database e Instancia administrada de Azure SQL se ha dividido en dos secciones independientes. También se ha actualizado la forma de hacer referencia a una instancia administrada, de *instancia administrada de Azure SQL Database* a *Instancia administrada de Azure SQL*.

Se debe a que algunas características y funcionalidades varían significativamente entre una única base de datos y una instancia administrada, y cada vez resulta más complicado explicar matices complejos entre Azure SQL Database y Azure SQL Managed Instance en los artículos individuales compartidos.

Esta aclaración entre los distintos productos de Azure SQL debe simplificar y optimizar el proceso de trabajo con el motor de base de datos de SQL Server en Azure, independientemente de que se trate de una única base de datos administrada en Azure SQL Database, de una instancia administrada completa que hospeda varias bases de datos en Azure SQL Managed Instance o del conocido producto SQL Server local hospedado en una máquina virtual de Azure.

Tenga en cuenta que se trata de un trabajo en curso y que no todos los artículos se han actualizado todavía. Por ejemplo, la documentación de las instrucciones Transact-SQL (T-SQL), los procedimientos almacenados y muchas características compartidas entre Azure SQL Database e Instancia administrada de Azure SQL aún no se han completado, por lo que le agradecemos su paciencia a medida que seguimos aclarando el contenido. 

En esta tabla se proporciona una comparación rápida del cambio en la terminología: 


|**Nuevo término**  | **Término anterior**  |**Explicación** |
|---------|---------|---------|
|**Instancia administrada de Azure SQL** | *Instancia administrada* de Azure SQL Database| Instancia administrada de Azure SQL es su propio producto dentro de la familia de Azure SQL, en lugar de tratarse exclusivamente de una opción de implementación en Azure SQL Database. | 
|**Azure SQL Database**|*Base de datos única* de Azure SQL Database| A menos que se especifique explícitamente lo contrario, el nombre del producto Azure SQL Database incluye bases de datos únicas y bases de datos implementadas en un grupo elástico. |
|**Azure SQL Database**|*Grupo elástico* de Azure SQL Database| A menos que se especifique explícitamente lo contrario, el nombre del producto Azure SQL Database incluye bases de datos únicas y bases de datos implementadas en un grupo elástico.  |
|**Azure SQL Database** |Azure SQL Database | Aunque el término no varía, ahora solo se aplica a las implementaciones en grupos elásticos y a las bases de datos únicas, y no incluye la instancia administrada. |
| **SQL de Azure**| N/D | Esto se refiere a la familia de productos del motor de base de datos de SQL Server que están disponibles en Azure: Azure SQL Database, Instancia administrada de Azure SQL y SQL Server en máquinas virtuales de Azure. | 

## <a name="features-in-public-preview"></a>Características en versión preliminar pública

### <a name="azure-sql-database"></a>[Azure SQL Database](#tab/single-database)

| Característica | Detalles |
| ---| --- |
| Recuperación de base de datos acelerada con bases de datos únicas y grupos elásticos | Para más información, consulte [Recuperación de base de datos acelerada](../accelerated-database-recovery.md).|
| Clasificación y detección de datos  |Para obtener información, vea [Detección y clasificación de datos de Azure SQL Database y Azure Synapse Analytics](data-discovery-and-classification-overview.md).|
| Trabajos de base de datos elástica | Para más información, consulte [Creación, configuración y administración de trabajos elásticos](elastic-jobs-overview.md). |
| Consultas elásticas | Para más información, consulte [Introducción a la consulta elástica](elastic-query-overview.md). |
| Transacciones elásticas | [Transacciones distribuidas en bases de datos en la nube](elastic-transactions-overview.md). |
| Editor de consultas de Azure Portal |Para más información, consulte [Uso del editor de consultas SQL de Azure Portal para conectarse a datos y consultarlos](connect-query-portal.md).|
| R services o aprendizaje automático con bases de datos únicas y grupos elásticos |Para más información, consulte [Machine Learning Services en Azure SQL Database](https://docs.microsoft.com/sql/advanced-analytics/what-s-new-in-sql-server-machine-learning-services?view=sql-server-2017#machine-learning-services-in-azure-sql-database).|
|SQL Analytics|Para más información, consulte [Azure SQL Analytics](../../azure-monitor/insights/azure-sql.md).|
| &nbsp; |

### <a name="azure-sql-managed-instance"></a>[Instancia administrada de Azure SQL](#tab/managed-instance)

| Característica | Detalles |
| ---| --- |
| <a href="/azure/sql-database/sql-database-instance-pools">Grupos de instancias</a> | Una manera útil y rentable de migrar pequeñas instancias de SQL a la nube. |
| <a href="https://aka.ms/managed-instance-aadlogins">Entidades de seguridad (inicios de sesión) del servidor de Azure AD con SSMS</a> | Cree inicios de sesión a nivel de instancia con una instrucción <a href="https://docs.microsoft.com/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current">CREATE LOGIN FROM EXTERNAL PROVIDER</a>. |
| [Replicación transaccional](../managed-instance/replication-transactional-overview.md) | Replique los cambios de las tablas en otras bases de datos de SQL Managed Instance, SQL Database o SQL Server. También puede actualizar las tablas al cambiarse algunas filas en otras instancias de SQL Managed Instance o SQL Server. Para más información, vea [Configuración de la replicación en una Instancia administrada de Azure SQL](../managed-instance/replication-between-two-instances-configure-tutorial.md). |
| Detección de amenazas |Para más información, vea [Configuración de la detección de amenazas en Instancia administrada de Azure SQL](../managed-instance/threat-detection-configure.md).|
| Retención de copia de seguridad a largo plazo | Para más información, consulte [Configuración de la retención de copia de seguridad a largo plazo en Azure SQL Managed Instance](../managed-instance/long-term-backup-retention-configure.md), que actualmente se encuentra en versión preliminar pública limitada. | 

---

## <a name="sql-managed-instance-new-features-and-known-issues"></a>Nuevas características y problemas conocidos de SQL Managed Instance

### <a name="sql-managed-instance-h2-2019-updates"></a>Actualizaciones de la Instancia administrada de SQL del segundo semestre de 2019

- [Configuración de la subred asistida por servicio](https://azure.microsoft.com/updates/service-aided-subnet-configuration-for-managed-instance-in-azure-sql-database-available/) es una forma segura y práctica de administrar la configuración de la subred, donde el usuario controla el tráfico de datos, mientras que SQL Managed Instance garantiza el flujo ininterrumpido de tráfico de administración.
- [Cifrado de datos transparente (TDE) con Bring Your Own Key (BYOK)](https://azure.microsoft.com/updates/general-avilability-transparent-data-encryption-with-customer-managed-keys-for-azure-sql-database-managed-instance/) habilita un escenario Bring Your Own Key (BYOK) para la protección de datos en reposo y permite a las organizaciones implementar la separación de tareas en la administración de claves y datos.
- [Los grupos de conmutación por error automática](https://azure.microsoft.com/updates/azure-sql-database-auto-failover-groups-feature-now-available-in-all-regions/) permiten replicar todas las bases de datos de la instancia principal en una instancia secundaria de otra región.
- [Las marcas de seguimiento globales](https://azure.microsoft.com/updates/global-trace-flags-are-now-available-in-azure-sql-database-managed-instance/) permiten configurar el comportamiento de SQL Managed Instance.

### <a name="sql-managed-instance-h1-2019-updates"></a>Actualizaciones de la Instancia administrada de SQL del primer semestre de 2019

Las características siguientes están habilitadas en el modelo de implementación de SQL Managed Instance del primer semestre de 2019:
  - Compatibilidad con las suscripciones con el <a href="https://aka.ms/sql-mi-visual-studio-subscribers"> crédito mensual de Azure para suscriptores de Visual Studio</a> y mayores [límites regionales](../managed-instance/resource-limits.md#regional-resource-limitations).
  - Compatibilidad con <a href="https://docs.microsoft.com/sharepoint/administration/deploy-azure-sql-managed-instance-with-sharepoint-servers-2016-2019"> SharePoint 2016 y SharePoint 2019 </a> y <a href="https://docs.microsoft.com/business-applications-release-notes/october18/dynamics365-business-central/support-for-azure-sql-database-managed-instance"> Dynamics 365 Business Central. </a>
  - Creación de una instancia administrada con la <a href="https://aka.ms/managed-instance-collation">intercalación a nivel de instancia</a> y la <a href="https://azure.microsoft.com/updates/managed-instance-time-zone-ga/">zona horaria</a> que elija.
  - Las instancias administradas están protegidas por el [firewall integrado](../managed-instance/management-endpoint-verify-built-in-firewall.md).
  - Configuración de SQL Managed Instance para que usen [puntos de conexión públicos](../managed-instance/public-endpoint-configure.md), la conexión de [invalidación de proxy](connectivity-architecture.md#connection-policy) para obtener un mejor rendimiento de red, <a href="https://aka.ms/four-cores-sql-mi-update">cuatro núcleos virtuales en la generación de hardware de gen5</a> o <a href="https://aka.ms/managed-instance-configurable-backup-retention">configuración de la retención de copia de seguridad hasta 35 días</a> para la restauración a un momento dado. La [retención de copias de seguridad a largo plazo](long-term-retention-overview.md#sql-managed-instance-support) (hasta 10 años) se encuentra actualmente en versión preliminar pública limitada.  
  - Las nuevas funcionalidades permiten <a href="https://medium.com/@jocapc/geo-restore-your-databases-on-azure-sql-instances-1451480e90fa">restaurar geográficamente la base de datos en otro centro de datos mediante PowerShell</a>, [cambiar el nombre de la base de datos](https://azure.microsoft.com/updates/azure-sql-database-managed-instance-database-rename-is-supported/) y [eliminar un clúster virtual](../managed-instance/virtual-cluster-delete.md).
  - El nuevo [rol de colaborador de instancia](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#sql-managed-instance-contributor) integrado permite el cumplimiento de la separación de los derechos (SoD) con los principios de seguridad y el cumplimiento de los estándares de la empresa.
  - La Instancia administrada de SQL está disponible en las siguientes regiones de Azure Government de disponibilidad general (US Gov Texas y US Gov Arizona), así como en Norte de China 2 y Este de China 2. También está disponible en las siguientes regiones públicas: Centro de Australia, Centro de Australia 2, Sur de Brasil, Sur de Francia, Centro de Emiratos Árabes Unidos, Norte de Emiratos Árabes Unidos, Norte de Sudáfrica, Oeste de Sudáfrica.

### <a name="known-issues"></a>Problemas conocidos

|Incidencia  |Fecha de detección  |Estado  |Fecha de resolución  |
|---------|---------|---------|---------|
|[La restauración de la copia de seguridad manual sin CHECKSUM puede devolver un error](#restoring-manual-backup-without-checksum-might-fail)|Mayo de 2020|Resuelto|Junio de 2020|
|[El agente deja de responder al modificar, deshabilitar o habilitar los trabajos existentes](#agent-becomes-unresponsive-upon-modifying-disabling-or-enabling-existing-jobs)|Mayo de 2020|Resuelto|Junio de 2020|
|[Permisos en el grupo de recursos no aplicados a Instancia administrada de SQL](#permissions-on-resource-group-not-applied-to-sql-managed-instance)|Febrero de 2020|Tiene solución alternativa||
|[Limitación de la conmutación por error manual a través del portal para grupos de conmutación por error](#limitation-of-manual-failover-via-portal-for-failover-groups)|Enero de 2020|Tiene solución alternativa||
|[Los roles del Agente SQL necesitan permisos de ejecución (EXECUTE) explícitos para los inicios de sesión que no sean sysadmin](#in-memory-oltp-memory-limits-are-not-applied)|Diciembre de 2019|Tiene solución alternativa||
|[Los trabajos del Agente SQL pueden ser interrumpidos por el reinicio del proceso del agente](#sql-agent-jobs-can-be-interrupted-by-agent-process-restart)|Diciembre de 2019|Resuelto|Marzo de 2020|
|[No se admiten inicios de sesión y usuarios de Azure AD en SSDT](#azure-ad-logins-and-users-are-not-supported-in-ssdt)|Noviembre de 2019|No hay solución alternativa||
|[No se aplican los límites de memoria de OLTP en memoria](#in-memory-oltp-memory-limits-are-not-applied)|Octubre de 2019|Tiene solución alternativa||
|[Se mostró un error al intentar quitar un archivo que no está vacío](#wrong-error-returned-while-trying-to-remove-a-file-that-is-not-empty)|Octubre de 2019|Tiene solución alternativa||
|[Las operaciones de cambio de nivel de servicio y creación de instancia se bloquean con la restauración en curso de la base de datos](#change-service-tier-and-create-instance-operations-are-blocked-by-ongoing-database-restore)|Septiembre de 2019|Tiene solución alternativa||
|[Es posible que sea necesario volver a configurar Resource Governor en el nivel de servicio Crítico para la empresa después de la conmutación por error](#resource-governor-on-business-critical-service-tier-might-need-to-be-reconfigured-after-failover)|Septiembre de 2019|Tiene solución alternativa||
|[Los cuadros de diálogo de Service Broker entre bases de datos se deben volver a inicializar después de la actualización del nivel de servicio](#cross-database-service-broker-dialogs-must-be-reinitialized-after-service-tier-upgrade)|Agosto de 2019|Tiene solución alternativa||
|[No se admite la suplantación de tipos de inicio de sesión de Azure AD](#impersonation-of-azure-ad-login-types-is-not-supported)|Julio de 2019|No hay solución alternativa||
|[No se admite el parámetro @query en sp_send_db_mail](#-parameter-not-supported-in-sp_send_db_mail)|Abril de 2019|No hay solución alternativa||
|[La replicación transaccional debe volver a configurarse después de la conmutación por error geográfica](#transactional-replication-must-be-reconfigured-after-geo-failover)|Marzo de 2019|No hay solución alternativa||
|[La base de datos temporal se usa durante la operación RESTORE](#temporary-database-is-used-during-restore-operation)||Tiene solución alternativa||
|[Se vuelve a crear la estructura y el contenido de TEMPDB](#tempdb-structure-and-content-is-re-created)||No hay solución alternativa||
|[Exceder el espacio de almacenamiento con archivos de base de datos pequeños](#exceeding-storage-space-with-small-database-files)||Tiene solución alternativa||
|[Se muestran valores de GUID en lugar de nombres de base de datos](#guid-values-shown-instead-of-database-names)||Tiene solución alternativa||
|[Los registros de errores no se conservan](#error-logs-arent-persisted)||No hay solución alternativa||
|[No se admite el ámbito de transacción en dos bases de datos de la misma instancia](#transaction-scope-on-two-databases-within-the-same-instance-isnt-supported)||Tiene solución alternativa|Marzo de 2020|
|[Los módulos de CLR y los servidores vinculados en algún momento no pueden hacer referencia a una dirección IP local](#clr-modules-and-linked-servers-sometimes-cant-reference-a-local-ip-address)||Tiene solución alternativa||
|No se verifica la coherencia de la base de datos mediante DBCC CHECKDB después de restaurar la base de datos de Azure Blob Storage.||Resuelto|Noviembre de 2019|
|La restauración de una base de datos a un momento dado del nivel de servicio Crítico para la empresa al De uso general no se completará correctamente si la base de datos de origen contiene objetos OLTP en memoria.||Resuelto|Octubre de 2019|
|Característica Correo electrónico de base de datos con servidores de correo externos (que no son de Azure) mediante una conexión segura||Resuelto|Octubre de 2019|
|Las bases de datos independientes no se admiten en la Instancia administrada de SQL||Resuelto|Agosto de 2019|


### <a name="restoring-manual-backup-without-checksum-might-fail"></a>La restauración de la copia de seguridad manual sin CHECKSUM puede devolver un error

En determinadas circunstancias, es posible que no se restaure la copia de seguridad manual de las bases de datos que se realizó en una instancia administrada sin CHECKSUM. En tal caso, vuelva a intentar restaurar la copia de seguridad hasta que complete el proceso correctamente.

**Solución alternativa**: realice copias de seguridad manuales de las bases de datos en instancias administradas con CHECKSUM habilitado.

### <a name="agent-becomes-unresponsive-upon-modifying-disabling-or-enabling-existing-jobs"></a>El agente deja de responder al modificar, deshabilitar o habilitar los trabajos existentes

En determinadas circunstancias, la modificación, deshabilitación o habilitación de un trabajo existente puede hacer que el agente deje de responder. El problema se mitiga automáticamente tras la detección, lo que da como resultado un reinicio del proceso del agente.

### <a name="permissions-on-resource-group-not-applied-to-sql-managed-instance"></a>Permisos en el grupo de recursos no aplicados a la Instancia administrada de SQL

Cuando el rol de Azure del colaborador de SQL Managed Instance se aplica a un grupo de recursos (RG), no se aplica a SQL Managed Instance y no tiene ningún efecto.

**Solución alternativa**: configure un rol de colaborador de SQL Managed Instance para los usuarios en el nivel de suscripción.

### <a name="limitation-of-manual-failover-via-portal-for-failover-groups"></a>Limitación de la conmutación por error manual a través del portal para grupos de conmutación por error

Si un grupo de conmutación por error abarca instancias de distintas suscripciones o grupos de recursos de Azure, la conmutación por error manual no se puede iniciar desde la instancia principal del grupo de conmutación por error.

**Solución alternativa**: inicie la conmutación por error mediante el portal desde la base de datos geográfica secundaria.

### <a name="sql-agent-roles-need-explicit-execute-permissions-for-non-sysadmin-logins"></a>Los roles del Agente SQL necesitan permisos de ejecución (EXECUTE) explícitos para los inicios de sesión que no sean sysadmin

Si los inicios de sesión que no son de sysadmin se agregan a cualquiera de los [roles fijos de base de datos del Agente SQL](https://docs.microsoft.com/sql/ssms/agent/sql-server-agent-fixed-database-roles), existe un problema en el que los permisos EXECUTE explícitos deben concederse a los procedimientos almacenados maestros para que estos inicios de sesión funcionen. Si se encuentra este problema, aparece el mensaje de error "Se denegó el permiso EXECUTE en el objeto <nombre_objeto> (Microsoft SQL Server, error: 229)".

**Solución alternativa**: Una vez que agregue inicios de sesión a un rol fijo de base de datos del Agente SQL (SQLAgentUserRole, SQLAgentReaderRole o SQLAgentOperatorRole), para cada uno de los inicios de sesión agregados a estos roles, ejecute el siguiente script T-SQL para conceder explícitamente permisos EXECUTE a los procedimientos almacenados enumerados.

```tsql
USE [master]
GO
CREATE USER [login_name] FOR LOGIN [login_name]
GO
GRANT EXECUTE ON master.dbo.xp_sqlagent_enum_jobs TO [login_name]
GRANT EXECUTE ON master.dbo.xp_sqlagent_is_starting TO [login_name]
GRANT EXECUTE ON master.dbo.xp_sqlagent_notify TO [login_name]
```

### <a name="sql-agent-jobs-can-be-interrupted-by-agent-process-restart"></a>Los trabajos del Agente SQL pueden ser interrumpidos por el reinicio del proceso del agente

**(Se resuelve en marzo de 2020)** El Agente SQL crea una sesión cada vez que se inicia un trabajo, lo que aumenta gradualmente el consumo de memoria. Para evitar alcanzar el límite de memoria interna que bloquearía la ejecución de los trabajos programados, el proceso del agente se reiniciará una vez que el consumo de memoria alcance el umbral. Puede provocar interrumpir la ejecución de los trabajos que se ejecutan en el momento del reinicio.

### <a name="in-memory-oltp-memory-limits-are-not-applied"></a>No se aplican los límites de memoria de OLTP en memoria

El nivel de servicio Crítico para la empresa no aplicará correctamente los [límites de memoria máximos para los objetos optimizados para memoria](../managed-instance/resource-limits.md#in-memory-oltp-available-space) en algunos casos. La Instancia administrada de SQL puede permitir que la carga de trabajo use más memoria para las operaciones de OLTP en memoria, lo que puede afectar a la disponibilidad y la estabilidad de la instancia. Es posible que las consultas de OLTP en memoria que alcanzan los límites no generen errores de inmediato. Pronto se resolverá este problema. Las consultas que usan en mayor grado OLTP en memoria producirán un error antes si llegan a los [límites](../managed-instance/resource-limits.md#in-memory-oltp-available-space).

**Solución alternativa**: [Supervise el uso de almacenamiento de OLTP en memoria ](https://docs.microsoft.com/azure/azure-sql/in-memory-oltp-monitor-space) con [SQL Server Management Studio](/sql/relational-databases/in-memory-oltp/monitor-and-troubleshoot-memory-usage#bkmk_Monitoring) para asegurarse de que la carga de trabajo no esté usando más memoria de la disponible. Aumente los límites de memoria que dependen del número de núcleos virtuales, u optimice la carga de trabajo para usar menos memoria.
 
### <a name="wrong-error-returned-while-trying-to-remove-a-file-that-is-not-empty"></a>Se mostró un error al intentar quitar un archivo que no está vacío

SQL Server y SQL Managed Instance [no permiten al usuario quitar un archivo que no está vacío](/sql/relational-databases/databases/delete-data-or-log-files-from-a-database#Prerequisites). Si intenta quitar un archivo de datos no vacío mediante una instrucción `ALTER DATABASE REMOVE FILE`, el error `Msg 5042 – The file '<file_name>' cannot be removed because it is not empty` no se mostrará inmediatamente. SQL Managed Instance seguirá intentando quitar el archivo y se producirá un error en la operación después de 30 minutos con `Internal server error`.

**Solución alternativa**: quite el contenido del archivo mediante el comando `DBCC SHRINKFILE (N'<file_name>', EMPTYFILE)`. Si este es el único archivo del grupo de archivos, debe eliminar los datos de la tabla o partición asociada a este grupo de archivos antes de reducir el archivo y, opcionalmente, cargar estos datos en otra tabla o partición.

### <a name="change-service-tier-and-create-instance-operations-are-blocked-by-ongoing-database-restore"></a>Las operaciones de cambio de nivel de servicio y creación de instancia se bloquean con la restauración en curso de la base de datos

La instrucción `RESTORE` en curso, el proceso de migración del servicio de migración de datos y la restauración a un momento dado integrada bloquean la actualización de un nivel de servicio o del cambio de tamaño de la instancia existente, así como la creación de nuevas instancias hasta que finalice el proceso de restauración. 

El proceso de restauración bloqueará estas operaciones en las instancias administradas y en los grupos de instancias de la misma subred en la que se ejecute el proceso de restauración. Las instancias de los grupos de instancias no se ven afectadas. No se producirá ningún error en las operaciones de creación o cambio del nivel de servicio ni se agotará su tiempo de espera. Continuarán una vez que el proceso de restauración se complete o cancele.

**Solución alternativa**: espere a que finalice el proceso de restauración; también puede cancelarlo si la operación de creación o actualización del nivel de servicio tiene mayor prioridad.

### <a name="resource-governor-on-business-critical-service-tier-might-need-to-be-reconfigured-after-failover"></a>Es posible que sea necesario volver a configurar Resource Governor en el nivel de servicio Crítico para la empresa después de la conmutación por error

La característica [Resource Governor](/sql/relational-databases/resource-governor/resource-governor) que le permite limitar los recursos asignados a la carga de trabajo de usuario puede clasificar incorrectamente alguna carga de trabajo de usuario después de una conmutación por error o un cambio de nivel de servicio iniciado por el usuario (por ejemplo, el cambio de número máximo de núcleos virtuales o tamaño máximo de almacenamiento de instancia).

**Solución alternativa**: Ejecute `ALTER RESOURCE GOVERNOR RECONFIGURE` periódicamente o como parte de un trabajo del Agente SQL que ejecuta la tarea de SQL cuando la instancia se inicia si usa [Resource Governor](/sql/relational-databases/resource-governor/resource-governor).

### <a name="cross-database-service-broker-dialogs-must-be-reinitialized-after-service-tier-upgrade"></a>Los cuadros de diálogo de Service Broker entre bases de datos se deben volver a inicializar después de la actualización del nivel de servicio

Los cuadros de diálogo de Service Broker entre bases de datos dejarán de enviar mensajes a los servicios de otras bases de datos después de la operación para cambiar el nivel de servicio. Los mensajes *no se pierden* y se pueden encontrar en la cola del remitente. Todos los cambios de tamaño de almacenamiento en núcleos virtuales o instancias de SQL Managed Instance harán que cambie un valor `service_broke_guid` de la vista [sys.databases](/sql/relational-databases/system-catalog-views/sys-databases-transact-sql) para todas las bases de datos. Todos los `DIALOG` creados con la instrucción [BEGIN DIALOG](/sql/t-sql/statements/begin-dialog-conversation-transact-sql) que hace referencia a los Service Broker de otra base de datos dejarán de entregar mensajes al servicio de destino.

**Solución alternativa**: detenga todas las actividades que usen conversaciones con cuadros de diálogo de Service Broker entre bases de datos antes de actualizar un nivel de servicio y vuelva a inicializarlos después. Si quedan mensajes que no se entregaron después del cambio de nivel de servicio, lea los mensajes de la cola de origen y vuelva a enviarlos a la cola de destino.

### <a name="impersonation-of-azure-ad-login-types-is-not-supported"></a>No se admite la suplantación de tipos de inicio de sesión de Azure AD

No se admite la suplantación con `EXECUTE AS USER` o `EXECUTE AS LOGIN` de las entidades de seguridad de Azure Active Directory (Azure AD) siguientes:
-   Usuarios de Azure AD con alias. Se devuelve el error siguiente en este caso: `15517`.
- Inicios de sesión y usuarios de Azure AD basados en aplicaciones de Azure AD o entidades de servicio. Se devuelven los errores siguientes en este caso: `15517` y `15406`.

### <a name="query-parameter-not-supported-in-sp_send_db_mail"></a>No se admite el parámetro @query en sp_send_db_mail

El parámetro `@query` del procedimiento [sp_send_db_mail](/sql/relational-databases/system-stored-procedures/sp-send-dbmail-transact-sql) no funciona.

### <a name="transactional-replication-must-be-reconfigured-after-geo-failover"></a>La replicación transaccional debe volver a configurarse después de la conmutación por error geográfica

Si la replicación transaccional se habilita en una base de datos en un grupo de conmutación por error automática, el administrador de la Instancia administrada de SQL debe limpiar todas las publicaciones de la base de datos principal anterior y volver a configurarlas en la nueva base principal después de que se produzca una conmutación por error a otra región. Para más información, consulte [Replicación](../managed-instance/transact-sql-tsql-differences-sql-server.md#replication).

### <a name="azure-ad-logins-and-users-are-not-supported-in-ssdt"></a>No se admiten inicios de sesión y usuarios de Azure AD en SSDT

SQL Server Data Tools no es totalmente compatible con los inicios de sesión y los usuarios de Azure AD.

### <a name="temporary-database-is-used-during-restore-operation"></a>La base de datos temporal se usa durante la operación RESTORE

Cuando una base de datos se restaura en SQL Managed Instance, el servicio de restauración crea primero una base de datos vacía con el nombre deseado para asignar el nombre a la instancia. Después de un tiempo, esta base de datos se quitará y se iniciará la restauración de la base de datos real. 

La base de datos que se encuentra en estado de *restauración* será temporal y tendrá un valor de GUID aleatorio en lugar del nombre. El nombre temporal se cambiará al nombre deseado especificado en la instrucción `RESTORE` una vez que finalice el proceso de restauración. 

En la fase inicial, un usuario puede acceder a la base de datos vacía e incluso crear tablas o cargar datos en esta base de datos. Esta base de datos temporal se quitará cuando el servicio de restauración inicie la segunda fase.

**Solución alternativa**: No acceda a la base de datos que va a restaurar hasta que vea que la restauración se ha completado.

### <a name="tempdb-structure-and-content-is-re-created"></a>Se vuelve a crear la estructura y el contenido de TEMPDB

La base de datos `tempdb` siempre se divide en 12 archivos de datos y la estructura de archivos no se puede cambiar. No se puede cambiar el tamaño máximo por archivo y no se pueden agregar nuevos archivos a `tempdb`. Siempre se vuelve a crear `Tempdb` como base de datos vacía cuando se inicia la instancia o se realiza una conmutación por error. Cualquier cambio que se realice en `tempdb` no se conservará.

### <a name="exceeding-storage-space-with-small-database-files"></a>Exceder el espacio de almacenamiento con archivos de base de datos pequeños

Se puede producir un error en las instrucciones `CREATE DATABASE`, `ALTER DATABASE ADD FILE` y `RESTORE DATABASE` porque la instancia puede alcanzar el límite de almacenamiento de Azure.

Cada instancia de uso general de SQL Managed Instance tiene hasta 35 TB de almacenamiento reservado para el espacio en disco Premium de Azure. Cada archivo de base de datos se coloca en un disco físico independiente. Los posibles tamaños de disco son: 128 GB, 256 GB, 512 GB, 1 TB o 4 TB. El espacio no utilizado en el disco no se cobra, pero la suma total de los tamaños de disco Premium de Azure no puede superar los 35 TB. En algunos casos, una instancia administrada que no necesita 8 TB en total puede superar los 35 TB de límite de Azure en tamaño de almacenamiento debido a la fragmentación interna.

Por ejemplo, una instancia de uso general de SQL Managed Instance puede tener un archivo grande de 1,2 TB almacenado en un disco de 4 TB. También podría tener 248 archivos cada uno de 1 GB situados en discos independientes de 128 GB. En este ejemplo:

- El tamaño de almacenamiento total del disco es de 1 x 4 TB + 248 x 128 GB = 35 TB.
- El espacio total reservado para las bases de datos en la instancia es de 1 x 1,2 TB + 248 x 1 GB = 1,4 TB.

Este ejemplo ilustra que, en determinadas circunstancias, debido a una distribución específica de archivos, una instancia de SQL Managed Instance podría alcanzar el límite de 35 TB que está reservado para un disco adjunto Premium de Azure cuando no se lo espere.

En este ejemplo, las bases de datos existentes seguirán funcionando y pueden crecer sin ningún problema, siempre y cuando no se agreguen nuevos archivos. No se podrían crear ni restaurar nuevas bases de datos porque no hay suficiente espacio para nuevas unidades de disco, incluso si el tamaño total de todas las bases de datos no alcanza el límite de tamaño de la instancia. El error que se devuelve en ese caso no está claro.

También puede [identificar el número de archivos restantes](https://medium.com/azure-sqldb-managed-instance/how-many-files-you-can-create-in-general-purpose-azure-sql-managed-instance-e1c7c32886c1) mediante el uso de vistas del sistema. Si se alcanza este límite, intente [vaciar y eliminar algunos de los archivos más pequeños mediante la instrucción DBCC SHRINKFILE](/sql/t-sql/database-console-commands/dbcc-shrinkfile-transact-sql#d-emptying-a-file) o cambie al [nivel Crítico para la empresa, que no tiene este límite](/azure/sql-database/sql-database-managed-instance-resource-limits#service-tier-characteristics).

### <a name="guid-values-shown-instead-of-database-names"></a>Se muestran valores de GUID en lugar de nombres de base de datos

Varias vistas del sistema, contadores de rendimiento, mensajes de error, XEvents y entradas de registro de errores muestran identificadores de base de datos GUID en lugar de los nombres reales de base de datos. No confíe en estos identificadores GUID porque se reemplazarán por los nombres reales de las bases de datos en el futuro.

**Solución alternativa**: Use la vista sys.databases para resolver el nombre real de la base de datos del nombre de la base de datos física, especificado en forma de identificadores de base de datos GUID:

```tsql
SELECT name as ActualDatabaseName, physical_database_name as GUIDDatabaseIdentifier 
FROM sys.databases
WHERE database_id > 4
```

### <a name="error-logs-arent-persisted"></a>Los registros de errores no se conservan

Los registros de errores que están disponibles en la Instancia administrada de SQL no se conservan y su tamaño no está incluido en el límite de almacenamiento máximo. Es posible que se borren automáticamente los registros de errores en caso de conmutación por error. Puede haber huecos en el historial del registro de errores porque la Instancia administrada de SQL se ha migrado varias veces en varias máquinas virtuales.

### <a name="transaction-scope-on-two-databases-within-the-same-instance-isnt-supported"></a>No se admite el ámbito de transacción en dos bases de datos de la misma instancia

**(Se resuelve en marzo de 2020)** La clase `TransactionScope` de .NET no funciona si dos consultas se envían a dos bases de datos de la misma instancia en el mismo ámbito de transacción:

```csharp
using (var scope = new TransactionScope())
{
    using (var conn1 = new SqlConnection("Server=quickstartbmi.neu15011648751ff.database.windows.net;Database=b;User ID=myuser;Password=mypassword;Encrypt=true"))
    {
        conn1.Open();
        SqlCommand cmd1 = conn1.CreateCommand();
        cmd1.CommandText = string.Format("insert into T1 values(1)");
        cmd1.ExecuteNonQuery();
    }

    using (var conn2 = new SqlConnection("Server=quickstartbmi.neu15011648751ff.database.windows.net;Database=b;User ID=myuser;Password=mypassword;Encrypt=true"))
    {
        conn2.Open();
        var cmd2 = conn2.CreateCommand();
        cmd2.CommandText = string.Format("insert into b.dbo.T2 values(2)");        cmd2.ExecuteNonQuery();
    }

    scope.Complete();
}

```

**Solución alternativa (no es necesaria desde marzo de 2020)** : use [SqlConnection.ChangeDatabase(String)](/dotnet/api/system.data.sqlclient.sqlconnection.changedatabase) para utilizar otra base de datos en un contexto de conexión en lugar de usar dos conexiones.

### <a name="clr-modules-and-linked-servers-sometimes-cant-reference-a-local-ip-address"></a>Los módulos de CLR y los servidores vinculados en algún momento no pueden hacer referencia a una dirección IP local.

Los módulos de CLR en SQL Managed Instance y las consultas distribuidas o los servidores vinculados que hacen referencia a una instancia actual en algún momento no pueden resolver la dirección IP de una instancia local. Este error es un problema transitorio.

**Solución alternativa**: use conexiones de contexto en un módulo de CLR, si es posible.

## <a name="updates"></a>Actualizaciones

Para obtener una lista de las mejoras y actualizaciones de SQL Database, consulte las [actualizaciones del servicio SQL Database](https://azure.microsoft.com/updates/?product=sql-database).

Para ver las actualizaciones y mejoras de todos los servicios de Azure, consulte las [actualizaciones de los servicios](https://azure.microsoft.com/updates).

## <a name="contribute-to-content"></a>Contribución al contenido

Para colaborar en la documentación de Azure SQL, consulte la [Guía para colaboradores de Microsoft Docs](https://docs.microsoft.com/contribute/).
