---
title: Solución de problemas de conexión comunes relacionados con Azure SQL Database
description: Proporciona los pasos para solucionar problemas de conexión de Azure SQL Database y resolver otros problemas específicos de Azure SQL Database o Azure SQL Managed Instance.
services: sql-database
ms.service: sql-db-mi
ms.subservice: development
ms.topic: troubleshooting
ms.custom: seo-lt-2019, OKR 11/2019, sqldbrb=1
author: ramakoni1
ms.author: ramakoni
ms.reviewer: carlrab,vanto
ms.date: 01/14/2020
ms.openlocfilehash: 4a1cfcbf110ab375a0fb357c1856fd0567a1c57a
ms.sourcegitcommit: 4a7a4af09f881f38fcb4875d89881e4b808b369b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89459426"
---
# <a name="troubleshooting-connectivity-issues-and-other-errors-with-azure-sql-database-and-azure-sql-managed-instance"></a>Solución de problemas de conectividad y otros errores con Azure SQL Database y Azure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Cuando la conexión a Azure SQL Database o Azure SQL Managed Instance no se logra establecer, se reciben mensajes de error. El origen de estos problemas de conexión puede ser la reconfiguración, la configuración del firewall, el tiempo de expiración de conexión, la información de inicio de sesión incorrecta o un error al aplicar procedimientos recomendados e instrucciones de diseño durante el proceso de [diseño de la aplicación](develop-overview.md). Además, si se alcanza el límite máximo en algunos recursos de Azure SQL Database o Instancia administrada de SQL, no será posible volver a establecer la conexión.

## <a name="transient-fault-error-messages-40197-40613-and-others"></a>Mensajes de error transitorios (40197, 40613 y otros)

La infraestructura de Azure ofrece la posibilidad de volver a configurar dinámicamente servidores cuando surgen cargas de trabajo pesadas en el servicio de SQL Database.  Este comportamiento dinámico podría dar lugar a que el programa cliente perdiera su conexión a la base de datos o la instancia. Este tipo de condición de error se conoce como *error transitorio*. Los eventos de reconfiguración de la base de datos se producen debido a un evento planeado (por ejemplo, una actualización de software) o a un evento no planeado (por ejemplo, el bloqueo de un proceso o un equilibrio de carga). La mayoría de los eventos de reconfiguración son generalmente de corta duración y se completarán en menos de 60 segundos. Sin embargo, ocasionalmente estos eventos pueden tardar más tiempo de finalizar, por ejemplo, cuando una transacción grande produce una recuperación de larga duración. En la tabla siguiente se enumeran varios errores transitorios que las aplicaciones pueden recibir al conectarse a SQL Database

### <a name="list-of-transient-fault-error-codes"></a>Lista de códigos de error transitorios

| Código de error | severity | Descripción |
| ---:| ---:|:--- |
| 4060 |16 |No se puede abrir la base de datos "%.&#x2a;ls" solicitada por el inicio de sesión. Error de inicio de sesión. Para obtener más información, consulte [los errores de 4000 a 4999](https://docs.microsoft.com/sql/relational-databases/errors-events/database-engine-events-and-errors#errors-4000-to-4999).|
| 40197 |17 |Error en el servicio al procesar la solicitud. Inténtelo de nuevo. Código de error %d.<br/><br/>Recibirá este error cuando el servicio esté inactivo debido a actualizaciones de software o hardware, errores de hardware u otros problemas de conmutación por error. El código de error (%d) incrustado en el mensaje de error 40197 proporciona información adicional sobre el tipo de error o conmutación por error que se ha producido. Algunos ejemplos de los códigos de error que se incrustan dentro del mensaje de error 40197 son 40020, 40143, 40166 y 40540.<br/><br/>Al volver a conectarse, se conectará automáticamente a una copia correcta de su base de datos. La aplicación debe detectar el error 40197, registrar el código de error incrustado (%d) dentro del mensaje para solucionar problemas y volver a conectarse a SQL Database hasta que los recursos estén disponibles; entonces, la conexión se establecerá de nuevo. Para obtener más información, vea [Errores transitorios](troubleshoot-common-connectivity-issues.md#transient-errors-transient-faults).|
| 40501 |20 |El servicio está ocupado actualmente. Vuelva a intentar la solicitud después de 10 segundos. Identificador de incidente: %ls. Código: %d. Para más información, consulte: <br/>&bull; &nbsp;[Límites de recursos de servidores SQL lógicos](resource-limits-logical-server.md)<br/>&bull; &nbsp;[Niveles de servicio en el modelo de compra basado en DTU](service-tiers-dtu.md)<br/>&bull; &nbsp;[Límites de recursos para grupos elásticos que usan el modelo de compra de DTU](resource-limits-dtu-elastic-pools.md)<br/>&bull; &nbsp;[Límites de recursos para bases de datos únicas que usan el modelo de compra en núcleos virtuales](resource-limits-vcore-single-databases.md)<br/>&bull; &nbsp;[Límites de recursos para grupos elásticos que usan el modelo de compra de núcleo virtual](resource-limits-vcore-elastic-pools.md)<br/>&bull; &nbsp;[Límites de recursos de Instancia administrada de Azure SQL](../managed-instance/resource-limits.md)|
| 40613 |17 |La base de datos '%.&#x2a;ls' en el servidor '%.&#x2a;ls' no está disponible actualmente. Vuelva a intentar la conexión más tarde. Si el problema continúa, póngase en contacto con el servicio de soporte al cliente y proporcióneles el id. de seguimiento de sesión de '%.&#x2a;ls'.<br/><br/> Este error puede producirse si ya existe una conexión de administrador dedicada (DAC) establecida en la base de datos. Para obtener más información, vea [Errores transitorios](troubleshoot-common-connectivity-issues.md#transient-errors-transient-faults).|
| 49918 |16 |No se puede procesar la solicitud. No hay suficientes recursos para procesar la solicitud.<br/><br/>El servicio está ocupado actualmente. Vuelva a intentar realizar la solicitud más tarde. Para más información, consulte: <br/>&bull; &nbsp;[Límites de recursos de servidores SQL lógicos](resource-limits-logical-server.md)<br/>&bull; &nbsp;[Niveles de servicio en el modelo de compra basado en DTU](service-tiers-dtu.md)<br/>&bull; &nbsp;[Límites de recursos para grupos elásticos que usan el modelo de compra de DTU](resource-limits-dtu-elastic-pools.md)<br/>&bull; &nbsp;[Límites de recursos para bases de datos únicas que usan el modelo de compra en núcleos virtuales](resource-limits-vcore-single-databases.md)<br/>&bull; &nbsp;[Límites de recursos para grupos elásticos que usan el modelo de compra de núcleo virtual](resource-limits-vcore-elastic-pools.md)<br/>&bull; &nbsp;[Límites de recursos de Instancia administrada de Azure SQL](../managed-instance/resource-limits.md) |
| 49919 |16 |No se puede procesar, crear ni actualizar la solicitud. Hay demasiadas operaciones de creación o actualización en curso para la suscripción "%ld".<br/><br/>El servicio está ocupado procesando varias solicitudes de creación o actualización para su suscripción o servidor. Actualmente las solicitudes están bloqueadas para la optimización de recursos. Consulte [sys.dm_operation_status](https://msdn.microsoft.com/library/dn270022.aspx) para ver las operaciones pendientes. Espere a que se completen las solicitudes de creación o actualización pendientes o elimine una de las solicitudes pendientes y vuelva a intentar la solicitud más tarde. Para más información, consulte: <br/>&bull; &nbsp;[Límites de recursos de servidores SQL lógicos](resource-limits-logical-server.md)<br/>&bull; &nbsp;[Niveles de servicio en el modelo de compra basado en DTU](service-tiers-dtu.md)<br/>&bull; &nbsp;[Límites de recursos para grupos elásticos que usan el modelo de compra de DTU](resource-limits-dtu-elastic-pools.md)<br/>&bull; &nbsp;[Límites de recursos para bases de datos únicas que usan el modelo de compra en núcleos virtuales](resource-limits-vcore-single-databases.md)<br/>&bull; &nbsp;[Límites de recursos para grupos elásticos que usan el modelo de compra de núcleo virtual](resource-limits-vcore-elastic-pools.md)<br/>&bull; &nbsp;[Límites de recursos de Instancia administrada de Azure SQL](../managed-instance/resource-limits.md) |
| 49920 |16 |No se puede procesar la solicitud. Hay demasiadas operaciones en curso para la suscripción "%ld".<br/><br/>El servicio está ocupado procesando varias solicitudes para esta suscripción. Actualmente las solicitudes están bloqueadas para la optimización de recursos. Consulta [sys.dm_operation_status](https://msdn.microsoft.com/library/dn270022.aspx) para el estado de la operación. Espere a que las solicitudes pendientes se hayan completado o elimine una de las solicitudes pendientes y vuelva a intentar la solicitud más tarde. Para más información, consulte: <br/>&bull; &nbsp;[Límites de recursos de servidores SQL lógicos](resource-limits-logical-server.md)<br/>&bull; &nbsp;[Niveles de servicio en el modelo de compra basado en DTU](service-tiers-dtu.md)<br/>&bull; &nbsp;[Límites de recursos para grupos elásticos que usan el modelo de compra de DTU](resource-limits-dtu-elastic-pools.md)<br/>&bull; &nbsp;[Límites de recursos para bases de datos únicas que usan el modelo de compra en núcleos virtuales](resource-limits-vcore-single-databases.md)<br/>&bull; &nbsp;[Límites de recursos para grupos elásticos que usan el modelo de compra de núcleo virtual](resource-limits-vcore-elastic-pools.md)<br/>&bull; &nbsp;[Límites de recursos de Instancia administrada de Azure SQL](../managed-instance/resource-limits.md) |
| 4221 |16 |No se pudo iniciar sesión en el secundario de lectura debido a una espera prolongada en HADR_DATABASE_WAIT_FOR_TRANSITION_TO_VERSIONING. La réplica no está disponible para el inicio de sesión porque faltan las versiones de las filas de transacciones que estaban en proceso cuando se recicló la réplica. Se puede resolver el problema al revertir o asignar las transacciones activas en la réplica principal. Se pueden minimizar las repeticiones de esta condición al evitar escribir transacciones en la principal. |

### <a name="steps-to-resolve-transient-connectivity-issues"></a>Pasos para resolver los problemas de conectividad transitorios

1. Compruebe el [panel de Estado de Microsoft Azure](https://azure.microsoft.com/status) para comprobar si hay interrupciones conocidas que se hayan producido durante el tiempo en el que la aplicación informó de los errores.
2. Para las aplicaciones que se conectan a un servicio en la nube, como Azure SQL Database, se deben prever eventos periódicos de reconfiguración e implementación de la lógica de reintento para gestionar estos errores en lugar de mostrarlos como errores de la aplicación.
3. Conforme una base de datos se acerca a sus límites de recursos, puede parecer un problema de conectividad transitorio. Consulte [Límites de los recursos](resource-limits-logical-server.md#what-happens-when-database-resource-limits-are-reached).
4. Si los problemas de conectividad continúan, si el tiempo de detección del error por parte de la aplicación supera los 60 segundos o si el error se repite varias veces en un día determinado, realice una solicitud de soporte técnico a Azure; para ello, seleccione **Obtener soporte** en el sitio [Soporte técnico de Azure](https://azure.microsoft.com/support/options) .

#### <a name="implementing-retry-logic"></a>Implementar lógica de reintento

Se recomienda encarecidamente que el programa cliente tenga lógica de reintento para poder tratar de restablecer una conexión después de dar tiempo a que se corrijan los errores transitorios.  Se recomienda un retraso de 5 segundos antes del primer reintento. Si se vuelve a intentar después de un retraso menor de 5 segundos, se correrá el riesgo de sobrecargar el servicio en la nube. Para cada intento siguiente el retraso debe aumentar exponencialmente, hasta un máximo de 60 segundos.

Para obtener ejemplos de lógica de reintento, vea:

- [Conectar de forma resistente a SQL con ADO.NET](https://docs.microsoft.com/sql/connect/ado-net/step-4-connect-resiliently-sql-ado-net)
- [Paso 4: Conectar de forma resistente a SQL con PHP](https://docs.microsoft.com/sql/connect/php/step-4-connect-resiliently-to-sql-with-php)

Para más información sobre el control de errores transitorios en la aplicación, consulte [Solución de problemas de errores de conexión transitorios en SQL Database](troubleshoot-common-connectivity-issues.md).

Una explicación del *período de bloqueo* para clientes que usan ADO.NET está disponible en [Grupos de conexión (ADO.NET)](https://msdn.microsoft.com/library/8xx3tyca.aspx).

## <a name="a-network-related-or-instance-specific-error-occurred-while-establishing-a-connection-to-your-server"></a>Se ha producido un error relacionado con la red o específico de la instancia al establecer una conexión con el servidor.

El problema se produce si la aplicación no se puede conectar al servidor.

Para resolver este problema, pruebe los pasos (en orden) de la sección [Pasos para solucionar problemas de conexión comunes](#steps-to-fix-common-connection-issues).

## <a name="the-serverinstance-was-not-found-or-was-not-accessible-errors-26-40-10053"></a>No se detectó el servidor/instancia o no estaba accesible (errores 26, 40, 10053)

### <a name="error-26-error-locating-server-specified"></a>Error 26: Error al buscar el servidor especificado

``System.Data.SqlClient.SqlException: A network-related or instance-specific error occurred while establishing a connection to SQL Server. The server was not found or was not accessible. Verify that the instance name is correct and that SQL Server is configured to allow remote connections.(provider: SQL Network Interfaces, error: 26 – Error Locating Server/Instance Specified)``

#### <a name="error-40-could-not-open-a-connection-to-the-server"></a>Error 40: No se pudo abrir una conexión al servidor

``A network-related or instance-specific error occurred while establishing a connection to SQL Server. The server was not found or was not accessible. Verify that the instance name is correct and that SQL Server is configured to allow remote connections. (provider: Named Pipes Provider, error: 40 - Could not open a connection to SQL Server)``

#### <a name="error-10053-a-transport-level-error-has-occurred-when-receiving-results-from-the-server"></a>Error 10053: Error en el nivel del transporte al recibir los resultados del servidor.

``10053: A transport-level error has occurred when receiving results from the server. (Provider: TCP Provider, error: 0 - An established connection was aborted by the software in your host machine)``

Estos problemas se producen si la aplicación no se puede conectar al servidor.

Para resolver estos problemas, pruebe los pasos (en orden) de la sección [Pasos para solucionar problemas de conexión comunes](#steps-to-fix-common-connection-issues).

## <a name="cannot-connect-to-server-due-to-firewall-issues"></a>No se puede conectar al servidor por problemas de firewall

### <a name="error-40615-cannot-connect-to--servername-"></a>Error 40615: No se puede conectar a <nombredelservidor>

Para resolver este problema, [configure un firewall en SQL Database con Azure Portal](firewall-configure.md).

### <a name="error-5-cannot-connect-to--servername-"></a>Error 5: No se puede conectar a <nombredelservidor>

Para resolver este problema, asegúrese de que el puerto 1433 está abierto para las conexiones salientes en todos los firewalls entre el cliente e Internet.

## <a name="unable-to-log-in-to-the-server-errors-18456-40531"></a>No se puede iniciar sesión en el servidor (errores 18456, 40531)

### <a name="login-failed-for-user--user-name-"></a>Error de inicio de sesión del usuario "<nombre de usuario>"

``Login failed for user '<User name>'.This session has been assigned a tracing ID of '<Tracing ID>'. Provide this tracing ID to customer support when you need assistance. (Microsoft SQL Server, Error: 18456)``

Para resolver este problema, póngase en contacto con el administrador de servicios para que le proporcione un nombre de usuario y una contraseña válidos.

Normalmente, el administrador de servicio puede usar los siguientes pasos para agregar las credenciales de inicio de sesión:

1. Inicie sesión en el servidor mediante SQL Server Management Studio (SSMS).
2. Ejecute la siguiente consulta SQL para comprobar si el nombre de inicio de sesión está deshabilitado:

   ```sql
   SELECT name, is_disabled FROM sys.sql_logins
   ```

3. Si el nombre correspondiente está deshabilitado, habilítelo mediante la instrucción siguiente:

   ```sql
   Alter login <User name> enable
   ```

4. Si el nombre de usuario de inicio de sesión SQL no existe, créelo con estos pasos:

   1. En SSMS, haga doble clic en **Seguridad** para expandirlo.
   2. Haga clic con el botón derecho en **Inicios de sesión** y, a continuación, seleccione **Nuevo inicio de sesión**.
   3. En el script generado con marcadores de posición, edite y ejecute la siguiente consulta SQL:

   ```sql
   CREATE LOGIN <SQL_login_name, sysname, login_name>
   WITH PASSWORD = ‘<password, sysname, Change_Password>’
   GO
   ```

5. Haga doble clic en **Base de datos**.
6. Seleccione la base de datos para la que desea conceder permiso al usuario.
7. Haga doble clic en **Seguridad**.
8. Haga clic con el botón derecho en **Usuarios** y, a continuación, seleccione **Nuevo usuario**.
9. En el script generado con marcadores de posición, edite y ejecute la siguiente consulta SQL:

   ```sql
   CREATE USER <user_name, sysname, user_name>
   FOR LOGIN <login_name, sysname, login_name>
   WITH DEFAULT_SCHEMA = <default_schema, sysname, dbo>
   GO
   -- Add user to the database owner role

   EXEC sp_addrolemember N’db_owner’, N’<user_name, sysname, user_name>’
   GO
   ```

   > [!NOTE]
   > también puede utilizar `sp_addrolemember` para asignar usuarios específicos a roles de base de datos específicos.

Para más información, consulte [Administración de bases de datos e inicios de sesión en Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-manage-logins).

## <a name="connection-timeout-expired-errors"></a>Errores de tiempo de espera de conexión agotado

### <a name="systemdatasqlclientsqlexception-0x80131904-connection-timeout-expired"></a>System.Data.SqlClient.SqlException (0x80131904): Tiempo de espera de conexión agotado

``System.Data.SqlClient.SqlException (0x80131904): Connection Timeout Expired. The timeout period elapsed while attempting to consume the pre-login handshake acknowledgement. This could be because the pre-login handshake failed or the server was unable to respond back in time. The duration spent while attempting to connect to this server was - [Pre-Login] initialization=3; handshake=29995;``

### <a name="systemdatasqlclientsqlexception-0x80131904-timeout-expired"></a>System.Data.SqlClient.SqlException (0x80131904): Tiempo de espera agotado

``System.Data.SqlClient.SqlException (0x80131904): Timeout expired. The timeout period elapsed prior to completion of the operation or the server is not responding.``

### <a name="systemdataentitycoreentityexception-the-underlying-provider-failed-on-open"></a>System.Data.Entity.Core.EntityException: Error del proveedor subyacente en [Abierto]

``System.Data.Entity.Core.EntityException: The underlying provider failed on Open. -> System.Data.SqlClient.SqlException: Timeout expired. The timeout period elapsed prior to completion of the operation or the server is not responding. -> System.ComponentModel.Win32Exception: The wait operation timed out``

### <a name="cannot-connect-to--server-name-"></a>No se puede conectar a <nombre del servidor>

``Cannot connect to <server name>.ADDITIONAL INFORMATION:Connection Timeout Expired. The timeout period elapsed during the post-login phase. The connection could have timed out while waiting for server to complete the login process and respond; Or it could have timed out while attempting to create multiple active connections. The duration spent while attempting to connect to this server was - [Pre-Login] initialization=231; handshake=983; [Login] initialization=0; authentication=0; [Post-Login] complete=13000; (Microsoft SQL Server, Error: -2) For help, click: http://go.microsoft.com/fwlink?ProdName=Microsoft%20SQL%20Server&EvtSrc=MSSQLServer&EvtID=-2&LinkId=20476 The wait operation timed out``

Estas excepciones pueden producirse por problemas de conexión o de consulta. Para confirmar que este error se debe a problemas de conectividad, consulte [Confirmación de si un error se debe a un problema de conectividad](#confirm-whether-an-error-is-caused-by-a-connectivity-issue).

Los tiempos de espera de conexión se producen porque la aplicación no se puede conectar al servidor. Para resolver este problema, pruebe los pasos (en orden) de la sección [Pasos para solucionar problemas de conexión comunes](#steps-to-fix-common-connection-issues).

## <a name="resource-governance-errors"></a>Errores de gobernanza de recursos

### <a name="error-10928-resource-id-d"></a>Error 10928: Id. de recurso: %d

``10928: Resource ID: %d. The %s limit for the database is %d and has been reached. See http://go.microsoft.com/fwlink/?LinkId=267637 for assistance. The Resource ID value in error message indicates the resource for which limit has been reached. For sessions, Resource ID = 2.``

Para resolver este problema, pruebe uno de los métodos siguientes:

- Compruebe si hay consultas de larga duración:

  > [!NOTE]
  > Se trata de un enfoque minimalista que puede que no resuelva el problema.

1. Ejecute la siguiente consulta SQL para comprobar la vista [sys.dm_exec_requests](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql) y ver las solicitudes de bloqueo:

   ```sql
   SELECT * FROM dm_exec_requests
   ```

2. Determine el **búfer de entrada** para el bloqueador de encabezado.
3. Optimice la consulta del bloqueador de encabezado.

   Para un procedimiento de solución de problemas detallado, consulte [¿Se ejecuta correctamente mi consulta en la nube?](https://docs.microsoft.com/archive/blogs/sqlblog/is-my-query-running-fine-in-the-cloud)

Si la base de datos alcanza el límite constantemente a pesar del bloqueo y las consultas de ejecución prolongada, considere la posibilidad de actualizar a una edición con más recursos en [Ediciones](https://azure.microsoft.com/pricing/details/sql-database/).

Para más información sobre las vistas de administración dinámica, consulte [Vistas de administración dinámica del sistema](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/system-dynamic-management-views).

Para más información sobre los límites de la base de datos, consulte [Límites de recursos de SQL Database para servidores](https://docs.microsoft.com/azure/sql-database/sql-database-resource-limits-database-server).

### <a name="error-10929-resource-id-1"></a>Error 10929: Identificador de recurso: 1

``10929: Resource ID: 1. The %s minimum guarantee is %d, maximum limit is %d and the current usage for the database is %d. However, the server is currently too busy to support requests greater than %d for this database. See http://go.microsoft.com/fwlink/?LinkId=267637 for assistance. Otherwise, please try again later.``

### <a name="error-40501-the-service-is-currently-busy"></a>Error 40501: El servicio está ocupado actualmente

``40501: The service is currently busy. Retry the request after 10 seconds. Incident ID: %ls. Code: %d.``

Se trata de un error de limitación del motor, que indica que se superan los límites de los recursos.

Para más información sobre los límites de recursos, consulte [Límites de recursos de servidores SQL lógicos](https://docs.microsoft.com/azure/sql-database/sql-database-resource-limits-database-server).

### <a name="error-40544-the-database-has-reached-its-size-quota"></a>Error 40544: La base de datos ha alcanzado su cuota de tamaño

``40544: The database has reached its size quota. Partition or delete data, drop indexes, or consult the documentation for possible resolutions. Incident ID: <ID>. Code: <code>.``

Este error se produce cuando la base de datos ha alcanzado su cuota de tamaño.

Los pasos siguientes pueden ayudarle a solucionar el problema o proporcionarle opciones adicionales:

1. Compruebe el tamaño actual de la base de datos mediante el panel en Azure Portal.

   > [!NOTE]
   > Para identificar qué tablas consumen más espacio y, por lo tanto, son posibles candidatos para la limpieza, ejecute la siguiente consulta SQL:

   ```sql
   SELECT o.name,
    SUM(p.row_count) AS 'Row Count',
    SUM(p.reserved_page_count) * 8.0 / 1024 AS 'Table Size (MB)'
   FROM sys.objects o
   JOIN sys.dm_db_partition_stats p on p.object_id = o.object_id
   GROUP BY o.name
   ORDER BY [Table Size (MB)] DESC
   ```

2. Si el tamaño actual no supera el tamaño máximo admitido para la versión, puede usar ALTER DATABASE para aumentar el valor de MAXSIZE.
3. Si la base de datos ya supera el tamaño máximo admitido para la edición, pruebe uno o varios de los pasos siguientes:

   - Realice actividades normales de limpieza de la base de datos. Por ejemplo, limpie los datos no deseados con truncar o eliminar, o bien mueva los datos con SQL Server Integration Services (SSIS) o el programa de copia masiva (bcp).
   - Cree particiones o elimine datos, quite índices o consulte la documentación para obtener soluciones posibles.
   - Para el escalado de la base de datos, consulte [Escalar recursos de base de datos única](https://docs.microsoft.com/azure/sql-database/sql-database-single-database-scale) y [Escalar recursos de grupos elásticos](https://docs.microsoft.com/azure/sql-database/sql-database-elastic-pool-scale).

### <a name="error-40549-session-is-terminated-because-you-have-a-long-running-transaction"></a>Error 40549: La sesión terminó porque tiene una transacción de larga duración

``40549: Session is terminated because you have a long-running transaction. Try shortening your transaction.``

Si se encuentra con frecuencia este mensaje de error, siga estos pasos para intentar resolver el problema:

1. Compruebe la vista sys.dm_exec_requests para ver todas las sesiones abiertas que tengan un valor alto para la columna total_elapsed_time. Para realizar esta comprobación, ejecute el siguiente script de SQL:

   ```sql
   SELECT * FROM dm_exec_requests
   ```

2. Determine el búfer de entrada de la consulta de larga duración.
3. Ajuste de la consulta.

Considere también el procesamiento por lotes de las consultas. Para más información sobre el procesamiento por lotes, consulte [Uso del procesamiento por lotes para mejorar el rendimiento de las aplicaciones de SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-use-batching-to-improve-performance).

Para un procedimiento de solución de problemas detallado, consulte [¿Se ejecuta correctamente mi consulta en la nube?](https://docs.microsoft.com/archive/blogs/sqlblog/is-my-query-running-fine-in-the-cloud)

### <a name="error-40551-the-session-has-been-terminated-because-of-excessive-tempdb-usage"></a>Error 40551: La sesión ha terminado debido al uso excesivo de TEMPDB

``40551: The session has been terminated because of excessive TEMPDB usage. Try modifying your query to reduce the temporary table space usage.``

Para resolver el problema, siga estos pasos:

1. Modifique las consultas para reducir el uso del espacio de la tabla temporal.
2. Quite los objetos temporales cuando ya no se necesiten.
3. Trunque las tablas o quite las que no se usen.

### <a name="error-40552-the-session-has-been-terminated-because-of-excessive-transaction-log-space-usage"></a>Error 40552: La sesión ha terminado debido al uso excesivo del espacio de registro de transacciones

``40552: The session has been terminated because of excessive transaction log space usage. Try modifying fewer rows in a single transaction.``

Para resolver este problema, pruebe los métodos siguientes:

- El problema puede producirse debido a operaciones de inserción, actualización o eliminación.
Intente reducir el número de filas que en las que se trabaja inmediatamente al implementar el procesamiento por lotes o al dividir las transacciones en varias más pequeñas.
- El problema puede producirse debido a operaciones de recompilación de índices. Para solucionar este problema, asegúrese de que el número de filas afectadas de la tabla * (tamaño medio del campo que se actualiza en bytes + 80) < 2 gigabytes (GB).

  > [!NOTE]
  > Para la recompilación del índice, el tamaño medio del campo que se actualiza debe sustituirse por el tamaño medio del índice.

### <a name="error-40553-the-session-has-been-terminated-because-of-excessive-memory-usage"></a>Error 40553: La sesión ha terminado debido al uso excesivo de la memoria

``40553 : The session has been terminated because of excessive memory usage. Try modifying your query to process fewer rows.``

Para solucionar este problema, intente optimizar la consulta.

Para un procedimiento de solución de problemas detallado, consulte [¿Se ejecuta correctamente mi consulta en la nube?](https://docs.microsoft.com/archive/blogs/sqlblog/is-my-query-running-fine-in-the-cloud)

### <a name="table-of-additional-resource-governance-error-messages"></a>Tabla de mensajes de error adicionales de gobernanza de recursos

| Código de error | severity | Descripción |
| ---:| ---:|:--- |
| 10928 |20 |Id. de recurso: %d. El límite %s para la base de datos es %d y se ha alcanzado. Para más información, consulte [Límites de recursos de SQL Database para bases de datos únicas y agrupadas](resource-limits-logical-server.md).<br/><br/>El identificador de recurso indica el recurso que ha alcanzado el límite. Para subprocesos de trabajo, el id. de recurso = 1. Para las sesiones, Identificador de recurso = 2.<br/><br/>Para más información sobre este error y cómo solucionarlo, consulte: <br/>&bull; &nbsp;[Límites de recursos de servidores SQL lógicos](resource-limits-logical-server.md)<br/>&bull; &nbsp;[Niveles de servicio en el modelo de compra basado en DTU](service-tiers-dtu.md)<br/>&bull; &nbsp;[Límites de recursos para grupos elásticos que usan el modelo de compra de DTU](resource-limits-dtu-elastic-pools.md)<br/>&bull; &nbsp;[Límites de recursos para bases de datos únicas que usan el modelo de compra en núcleos virtuales](resource-limits-vcore-single-databases.md)<br/>&bull; &nbsp;[Límites de recursos para grupos elásticos que usan el modelo de compra de núcleo virtual](resource-limits-vcore-elastic-pools.md)<br/>&bull; &nbsp;[Límites de recursos de Instancia administrada de Azure SQL](../managed-instance/resource-limits.md) |
| 10929 |20 |Id. de recurso: %d. La garantía mínima de %s es de %d, el límite máximo es %d y el uso actual de la base de datos es %d. Sin embargo, el servidor está demasiado ocupado en estos momentos para admitir solicitudes mayores que %d para esta base de datos. El identificador de recurso indica el recurso que ha alcanzado el límite. Para subprocesos de trabajo, el id. de recurso = 1. Para las sesiones, Identificador de recurso = 2. Para más información, consulte: <br/>&bull; &nbsp;[Límites de recursos de servidores SQL lógicos](resource-limits-logical-server.md)<br/>&bull; &nbsp;[Niveles de servicio en el modelo de compra basado en DTU](service-tiers-dtu.md)<br/>&bull; &nbsp;[Límites de recursos para grupos elásticos que usan el modelo de compra de DTU](resource-limits-dtu-elastic-pools.md)<br/>&bull; &nbsp;[Límites de recursos para bases de datos únicas que usan el modelo de compra en núcleos virtuales](resource-limits-vcore-single-databases.md)<br/>&bull; &nbsp;[Límites de recursos para grupos elásticos que usan el modelo de compra de núcleo virtual](resource-limits-vcore-elastic-pools.md)<br/>&bull; &nbsp;[Límites de recursos de Instancia administrada de Azure SQL](../managed-instance/resource-limits.md) <br/>De lo contrario, inténtelo de nuevo más tarde. |
| 40544 |20 |La base de datos ha alcanzado su cuota de tamaño. Cree particiones o elimine datos, quite índices o consulte la documentación para obtener soluciones posibles. Para el escalado de la base de datos, consulte [Escalar recursos de base de datos única](single-database-scale.md) y [Escalar recursos de grupos elásticos](elastic-pool-scale.md).|
| 40549 |16 |La sesión terminó porque tiene una transacción de larga duración. Intente reducir la transacción. Para más información sobre el procesamiento por lotes, consulte [Uso del procesamiento por lotes para mejorar el rendimiento de las aplicaciones de SQL Database](../performance-improve-use-batching.md).|
| 40550 |16 |La sesión ha terminado porque ha adquirido demasiados bloqueos. Intente leer o modificar menos filas en una sola transacción. Para más información sobre el procesamiento por lotes, consulte [Uso del procesamiento por lotes para mejorar el rendimiento de las aplicaciones de SQL Database](../performance-improve-use-batching.md).|
| 40551 |16 |La sesión ha terminado debido al uso excesivo de `TEMPDB` . Intente modificar la consulta para reducir el uso de espacio de la tabla temporal.<br/><br/>Si está usando objetos temporales, puede ahorrar espacio en la base de datos `TEMPDB` quitándolos una vez que la sesión ya no los necesite. Para obtener más información sobre el uso de tempdb en SQL Database, consulte [Base de datos tempdb en SQL Database](https://docs.microsoft.com/sql/relational-databases/databases/tempdb-database#tempdb-database-in-sql-database).|
| 40552 |16 |La sesión ha terminado debido al excesivo uso de espacio del registro de transacciones. Intente modificar menos filas en una sola transacción. Para más información sobre el procesamiento por lotes, consulte [Uso del procesamiento por lotes para mejorar el rendimiento de las aplicaciones de SQL Database](../performance-improve-use-batching.md).<br/><br/>Si realiza inserciones masivas con la utilidad `bcp.exe` o la clase `System.Data.SqlClient.SqlBulkCopy`, intente usar las opciones `-b batchsize` o `BatchSize` para limitar el número de filas copiadas al servidor en cada transacción. Si está volviendo a crear un índice con la instrucción `ALTER INDEX`, intente usar la opción `REBUILD WITH ONLINE = ON`. Para obtener información sobre los tamaños del registro de transacciones para el modelo de compra de núcleos virtuales, consulte: <br/>&bull; &nbsp;[Límites de recursos para bases de datos únicas que usan el modelo de compra en núcleos virtuales](resource-limits-vcore-single-databases.md)<br/>&bull; &nbsp;[Límites de recursos para grupos elásticos que usan el modelo de compra de núcleo virtual](resource-limits-vcore-elastic-pools.md)<br/>&bull; &nbsp;[Límites de recursos de Instancia administrada de Azure SQL](../managed-instance/resource-limits.md)|
| 40553 |16 |La sesión ha terminado debido al uso excesivo de la memoria. Intente modificar la consulta para procesar menos filas.<br/><br/>La reducción del número de operaciones `ORDER BY` y `GROUP BY` en el código Transact-SQL disminuye los requisitos de memoria de la consulta. Para el escalado de la base de datos, consulte [Escalar recursos de base de datos única](single-database-scale.md) y [Escalar recursos de grupos elásticos](elastic-pool-scale.md).|

## <a name="elastic-pool-errors"></a>Errores de grupos elásticos

Los errores siguientes están relacionados con la creación y el uso de grupos elásticos:

| Código de error | severity | Descripción | Acción correctiva |
|:--- |:--- |:--- |:--- |
| 1132 | 17 |El grupo elástico ha alcanzado su límite de almacenamiento. El uso del almacenamiento del grupo elástico no puede superar (%d) MB. Se ha intentado escribir datos en una base de datos cuando se ha alcanzado el límite de almacenamiento del grupo elástico. Para obtener información sobre los límites de recursos, consulte: <br/>&bull; &nbsp;[Límites de recursos para grupos elásticos que usan el modelo de compra de DTU](resource-limits-dtu-elastic-pools.md)<br/>&bull; &nbsp;[Límites de recursos para grupos elásticos que usan el modelo de compra basado en núcleo virtual](resource-limits-vcore-elastic-pools.md). <br/> |Considere la posibilidad de incrementar el número de DTU y de agregar almacenamiento al grupo elástico si es posible para aumentar su límite de almacenamiento, reducir el almacenamiento usado por las bases de datos individuales del grupo elástico o quitar bases de datos de este. Para la escalada del grupo elástico, consulte [Escalar recursos de grupos elásticos](elastic-pool-scale.md).|
| 10929 | 16 |La garantía mínima de %s es de %d, el límite máximo es %d y el uso actual de la base de datos es %d. Sin embargo, el servidor está demasiado ocupado en estos momentos para admitir solicitudes mayores que %d para esta base de datos. Para obtener información sobre los límites de recursos, consulte: <br/>&bull; &nbsp;[Límites de recursos para grupos elásticos que usan el modelo de compra de DTU](resource-limits-dtu-elastic-pools.md)<br/>&bull; &nbsp;[Límites de recursos para grupos elásticos que usan el modelo de compra basado en núcleo virtual](resource-limits-vcore-elastic-pools.md). <br/> De lo contrario, inténtelo de nuevo más tarde. Número mínimo de DTU o de núcleos virtuales por base de datos; número máximo de DTU o de núcleos virtuales por base de datos. El número total de trabajadores simultáneos (solicitudes) de todas las bases de datos del grupo elástico intentó superar el límite del grupo. |Considere la posibilidad de incrementar el número de DTU o de núcleos virtuales del grupo elástico si es posible para aumentar el límite de trabajadores, o bien quite bases de datos del grupo elástico. |
| 40844 | 16 |La base de datos '%ls' del servidor '%ls' es una base de datos de la versión '%ls' de un grupo elástico y no puede tener una relación de copia continua.  |N/D |
| 40857 | 16 |Grupo elástico no encontrado para el servidor: '%ls', nombre del grupo elástico: '%ls'. El grupo elástico especificado no existe en el servidor especificado. | Especifique un nombre de grupo elástico válido. |
| 40858 | 16 |El grupo elástico '%ls' ya existe en el servidor: '%ls'. El grupo elástico especificado ya existe en el servidor especificado. | Proporcione un nuevo nombre de grupo elástico. |
| 40859 | 16 |El grupo elástico no admite el nivel de servicio '%ls'. El nivel de servicio especificado no se admite para el aprovisionamiento de grupo elástico. |Especifique la versión correcta o deje el nivel de servicio en blanco para usar el nivel de servicio predeterminado. |
| 40860 | 16 |La combinación del grupo elástico '%ls' y del objetivo de servicio '%ls' no es válida. El grupo elástico y el nivel de servicio pueden especificarse juntos solo si se especifica el objetivo de servicio como "ElasticPool". |Especifique la combinación correcta de grupo elástico y nivel de servicio. |
| 40861 | 16 |La versión de la base de datos '%.*ls' no puede ser distinta del nivel de servicio del grupo elástico, que es '%.* ls'. La versión de la base de datos es distinta del nivel de servicio del grupo elástico. |No especifique una versión de la base de datos distinta del nivel de servicio del grupo elástico.  Tenga en cuenta que no es necesario especificar la versión de la base de datos. |
| 40862 | 16 |Debe especificarse el nombre del grupo elástico si se especifica el objetivo de servicio del grupo elástico. El objetivo del servicio de grupo elástico no identifica de manera única un grupo elástico. |Especifique el nombre del grupo elástico si usa el objetivo del servicio de grupo elástico. |
| 40864 | 16 |El número de DTU del grupo elástico debe ser de al menos (%d) DTU para el nivel de servicio ' %.* ls'. Se intentó establecer el número de DTU para el grupo elástico por debajo del límite mínimo. |Vuelva a intentar establecer el número de DTU para el grupo elástico al menos en el límite mínimo. |
| 40865 | 16 |El número de DTU del grupo elástico no puede superar (%d) DTU para el nivel de servicio ' %.* ls'. Se intentó establecer el número de DTU para el grupo elástico por encima del máximo. |Vuelva a intentar establecer el número de DTU para el grupo elástico en un valor inferior al límite máximo. |
| 40867 | 16 |El número máximo de DTU por base de datos debe ser al menos (%d) para el nivel de servicio '%.*ls'. Se intentó establecer el número máximo de DTU por base de datos por debajo del límite admitido. | Considere la posibilidad de utilizar el nivel de servicio del grupo elástico que admita la configuración deseada. |
| 40868 | 16 |El número máximo de DTU por base de datos no puede superar (%d) para el nivel de servicio '%.*ls'. Se intentó establecer el número máximo de DTU por base de datos por encima del límite admitido. | Considere la posibilidad de utilizar el nivel de servicio del grupo elástico que admita la configuración deseada. |
| 40870 | 16 |El número mínimo de DTU por base de datos no puede superar (%d) para el nivel de servicio '%.*ls'. Se intentó establecer el número mínimo de DTU por base de datos por encima del límite admitido. | Considere la posibilidad de utilizar el nivel de servicio del grupo elástico que admita la configuración deseada. |
| 40873 | 16 |El número de bases de datos (%d) y el mínimo de DTU por base de datos (%d) no puede superar el número de DTU del grupo elástico (%d). Se intentó especificar un número mínimo de DTU para las bases de datos del grupo elástico que supera el número de DTU del grupo elástico. | Considere la posibilidad de aumentar el número de DTU del grupo elástico, reduzca el número mínimo de DTU por base de datos, o bien reduzca el número de bases de datos del grupo elástico. |
| 40877 | 16 |No se puede eliminar un grupo elástico a menos que no contenga ninguna base de datos. El grupo elástico contiene una o más bases de datos y no se puede eliminar. |Quite las bases de datos del grupo elástico para poder eliminarlo. |
| 40881 | 16 |El grupo elástico ' %.* ls' ha alcanzado el límite de recuento de bases de datos.  El límite de bases de datos del grupo elástico no puede superar (%d) para un grupo elástico con (%d) DTU. Se intentó crear una base de datos en el grupo elástico, o agregar una base de datos al grupo elástico, cuando se ha alcanzado el límite de bases de datos del grupo elástico. | Considere la posibilidad de incrementar el número de DTU del grupo elástico si es posible para aumentar el límite de bases de datos, o bien quite bases de datos del grupo elástico. |
| 40889 | 16 |El límite de DTU o de almacenamiento del grupo elástico ' %.* ls' no se puede reducir, puesto que no proporcionaría suficiente espacio de almacenamiento para sus bases de datos. Se intentó reducir el límite de almacenamiento del grupo elástico por debajo de su uso de almacenamiento. | Considere la posibilidad de reducir el uso de almacenamiento de bases de datos individuales del grupo elástico, o bien quite bases de datos del grupo para reducir su número de DTU o el límite de almacenamiento. |
| 40891 | 16 |El número mínimo de DTU por base de datos (%d) no puede superar el número máximo de DTU por base de datos (%d). Se intentó establecer el número mínimo de DTU por base de datos con un valor superior al número máximo de DTU por base de datos. |Asegúrese de que el número mínimo de DTU por base de datos no supere el número máximo de DTU por base de datos. |
| TBD | 16 |El tamaño de almacenamiento para una base de datos individual de un grupo elástico no puede superar el tamaño máximo permitido por el grupo elástico del nivel de servicio '%.*ls'. El tamaño máximo de la base de datos supera el tamaño máximo permitido por el nivel de servicio del grupo elástico. |Establezca el tamaño máximo de la base de datos por debajo de los límites de tamaño máximo permitidos por el nivel de servicio del grupo elástico. |

## <a name="cannot-open-database-master-requested-by-the-login-the-login-failed"></a>No se puede abrir la base de datos maestra solicitada por el inicio de sesión. Error de inicio de sesión

Este problema se produce porque la cuenta no tiene permiso de acceso a la base de datos maestra. Sin embargo, SQL Server Management Studio (SSMS) intenta conectarse a la base de datos maestra.

Para resolver el problema, siga estos pasos:

1. En la pantalla de inicio de sesión de SSMS, seleccione **Opciones** y, luego, **Propiedades de la conexión**.
2. En el campo **Conectar con base de datos**, escriba el nombre de la base de datos predeterminada del usuario como la base de datos de inicio de sesión predeterminada y seleccione **Conectar**.

   ![Propiedades de la conexión](./media/troubleshoot-common-errors-issues/cannot-open-database-master.png)

## <a name="confirm-whether-an-error-is-caused-by-a-connectivity-issue"></a>Confirmación de si un error se debe a un problema de conectividad

Para confirmar si un error se debe a un problema de conectividad, revise el seguimiento de la pila de los marcos que muestran llamadas para abrir una conexión como las siguientes (observe la referencia a la clase **SqlConnection**):

```
System.Data.SqlClient.SqlConnection.TryOpen(TaskCompletionSource`1 retry)
 at System.Data.SqlClient.SqlConnection.Open()
 at AzureConnectionTest.Program.Main(String[] args)
ClientConnectionId:<Client connection ID>
```

Cuando se produce la excepción por problemas de consulta, observará una pila de llamadas similar a la siguiente (observe la referencia a la clase **SqlCommand**). En esta situación, [optimice las consultas ](https://docs.microsoft.com/archive/blogs/sqlblog/is-my-query-running-fine-in-the-cloud).

```
  at System.Data.SqlClient.SqlCommand.ExecuteReader()
  at AzureConnectionTest.Program.Main(String[] args)
  ClientConnectionId:<Client ID>
```

Para instrucciones adicionales sobre cómo ajustar el rendimiento, consulte los siguientes recursos:

- [Mantenimiento de índices y estadísticas de Azure SQL](https://techcommunity.microsoft.com/t5/Azure-Database-Support-Blog/How-to-maintain-Azure-SQL-Indexes-and-Statistics/ba-p/368787)
- [Ajuste manual del rendimiento de consultas en Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-performance-guidance)
- [Supervisión del rendimiento de Azure SQL Database con vistas de administración dinámica](https://docs.microsoft.com/azure/sql-database/sql-database-monitoring-with-dmvs)
- [Funcionamiento del almacén de consultas de Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-operate-query-store)

## <a name="steps-to-fix-common-connection-issues"></a>Pasos para solucionar problemas de conexión comunes

1. Asegúrese de que TCP/IP esté habilitado como protocolo de cliente en el servidor de aplicaciones. Para más información, consulte [Configurar protocolos de cliente](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-client-protocols). En los servidores de aplicaciones en los que no tiene instaladas herramientas de SQL, compruebe que TCP/IP esté habilitado mediante la ejecución de **cliconfg.exe** (utilidad de red de cliente de SQL Server).
2. Compruebe la cadena de conexión de la aplicación para asegurarse de que está configurada correctamente. Por ejemplo, asegúrese de que la cadena de conexión especifica el puerto correcto (1433) y el nombre completo del servidor.
Consulte [Obtención de información sobre la conexión](https://docs.microsoft.com/azure/sql-database/sql-database-connect-query-ssms#get-sql-server-connection-information).
3. Intente aumentar el valor de tiempo de espera de la conexión. Se recomienda el uso de un tiempo de espera de conexión de al menos 30 segundos.
4. Pruebe la conectividad entre el servidor de aplicaciones y la instancia de Azure SQL Database mediante [SQL Server Management Studio (SSMS)](https://docs.microsoft.com/azure/sql-database/sql-database-connect-query-ssms), un archivo UDL, ping o Telnet. Para más información, consulte [Solución de problemas de conectividad](https://support.microsoft.com/help/4009936/solving-connectivity-errors-to-sql-server) y [Diagnóstico de problemas de conexión](https://docs.microsoft.com/azure/sql-database/sql-database-connectivity-issues#diagnostics).

   > [!NOTE]
   > Como paso de solución de problemas, también puede probar la conectividad en otro equipo cliente.

5. Como procedimiento recomendado, asegúrese de que la lógica de reintento esté implantada. Para más información sobre la lógica de reintento, consulte [Solución de problemas temporales y de conexión a SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-connectivity-issues).

Si con los pasos anteriores no se resuelve el problema, intente recopilar más datos y luego póngase en contacto con el soporte técnico. Si la aplicación es un servicio en la nube, habilite el registro. Este paso devuelve una marca de tiempo UTC del error. Además, SQL Database devuelve el identificador de seguimiento. [Los servicios de soporte técnico de Microsoft](https://azure.microsoft.com/support/options/) pueden usar esta información.

Para más información sobre cómo habilitar el registro, consulte [Habilitar el registro de diagnósticos para las aplicaciones de Azure App Service](https://azure.microsoft.com/documentation/articles/web-sites-enable-diagnostic-log/).

## <a name="next-steps"></a>Pasos siguientes

- [Arquitectura de conectividad de Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-connectivity-architecture)
- [Controles de acceso a la red de Azure SQL Database y Azure Synapse Analytics](https://docs.microsoft.com/azure/sql-database/sql-database-networkaccess-overview)
