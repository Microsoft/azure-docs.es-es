---
title: Replicación geográfica activa
description: Use la replicación geográfica activa para crear bases de datos secundarias legibles de bases de datos individuales en Azure SQL Database en las mismas regiones del centro de datos o en otras diferentes.
services: sql-database
ms.service: sql-database
ms.subservice: high-availability
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: mathoma, carlrab
ms.date: 04/28/2020
ms.openlocfilehash: 10c0d3d5f043d31454810b55e808cd6df01467a4
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87448739"
---
# <a name="creating-and-using-active-geo-replication---azure-sql-database"></a>Creación y uso de la replicación geográfica activa: Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

La replicación geográfica activa es una característica de Azure SQL Database que permite crear bases de datos secundarias legibles de bases de datos individuales en un servidor en el mismo centro de datos (región) u otro diferente.

> [!NOTE]
> La replicación geográfica activa no es compatible con Instancia administrada de Azure SQL. Para la conmutación por error geográfica de Instancia administrada de SQL, use [Grupos de conmutación por error automática](auto-failover-group-overview.md).

La replicación geográfica activa se ha diseñado como solución de continuidad empresarial que permite que la aplicación realice una rápida recuperación ante desastres de bases de datos individuales en el caso de que se produzca un desastre regional o una interrupción a gran escala. Si la replicación geográfica está habilitada, la aplicación puede iniciar la conmutación por error en una base de datos secundaria de otra región de Azure. Se admiten hasta cuatro bases de datos secundarias en las mismas o en otras regiones, y las secundarias también se pueden usar para las consultas de acceso de solo lectura. La aplicación o el usuario deben iniciar manualmente la conmutación por error. Después de la conmutación por error, el nuevo elemento principal tiene un punto de conexión diferente.

> [!NOTE]
> La replicación geográfica activa replica los cambios al transmitir el registro de transacciones de la base de datos. No está relacionada con la [replicación transaccional](https://docs.microsoft.com/sql/relational-databases/replication/transactional/transactional-replication), que replica los cambios mediante la ejecución de comandos DML (INSERT, UPDATE, DELETE).

En el siguiente diagrama se ilustra una configuración típica de una aplicación de nube con redundancia geográfica mediante la replicación geográfica activa.

![replicación geográfica activa](./media/active-geo-replication-overview/geo-replication.png )

> [!IMPORTANT]
> SQL Database también admite grupos de conmutación por error automática. Para más información, consulte cómo se usan los [grupos de conmutación por error automática](auto-failover-group-overview.md).

Si, por cualquier motivo, se produce un error en la base de datos principal o, simplemente, debe desconectarse, puede iniciar la conmutación por error a cualquiera de las bases de datos secundarias. Cuando se activa la conmutación por error a una de las bases de datos secundarias, las demás bases de datos secundarias se vinculan automáticamente a la nueva base de datos principal.

Con la replicación geográfica activa puede administrar la replicación y la conmutación por error de una base de datos individual o de un conjunto de bases de datos en un servidor o en un grupo elástico. Para ello, puede utilizar:

- [Azure Portal](active-geo-replication-configure-portal.md)
- [PowerShell: base de datos única](scripts/setup-geodr-and-failover-database-powershell.md)
- [PowerShell: grupo elástico](scripts/setup-geodr-and-failover-elastic-pool-powershell.md)
- [Transact-SQL: base de datos única o grupo elástico](/sql/t-sql/statements/alter-database-azure-sql-database)
- [API REST: base de datos única](https://docs.microsoft.com/rest/api/sql/replicationlinks)

La replicación geográfica activa aprovecha la tecnología de [Grupos de disponibilidad Always On](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server) del motor de base de datos para replicar de forma asincrónica las transacciones confirmadas en la base de datos principal en una base de datos secundaria mediante aislamiento de instantáneas. Los grupos de conmutación por error automática proporcionan la semántica de grupo sobre la replicación geográfica activa, pero se usa el mismo mecanismo de replicación asincrónico. Mientras que, en cualquier momento dado, la base de datos secundaria puede ir ligeramente por detrás de la base de datos principal, se garantiza que los datos secundarios nunca tengan transacciones parciales. La redundancia entre regiones permite que las aplicaciones se recuperen rápidamente de la pérdida permanente de todo un centro de datos, o de partes de él, causada por desastres naturales, errores humanos catastróficos o actos malintencionados. Los datos específicos de RPO se encuentran en [Introducción a la continuidad empresarial](business-continuity-high-availability-disaster-recover-hadr-overview.md).

> [!NOTE]
> Si se produce un error de conexión entre dos regiones, intentamos volver a establecer las conexiones cada 10 segundos.

> [!IMPORTANT]
> Para garantizar que un cambio importante en la base de datos principal se replique en una secundaria antes de la conmutación por error, puede forzar la sincronización para garantizar la replicación de cambios importantes (por ejemplo, las actualizaciones de contraseñas). La sincronización forzada afecta al rendimiento al bloquear el subproceso de llamada hasta que todas las transacciones confirmadas se replican. Para más información, consulte [sp_wait_for_database_copy_sync](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/active-geo-replication-sp-wait-for-database-copy-sync). Para supervisar el retraso de replicación entre la base de datos principal y la base de datos secundaria geográficamente, consulte [sys.dm_geo_replication_link_status](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-geo-replication-link-status-azure-sql-database).

En la siguiente ilustración, se muestra un ejemplo de replicación geográfica activa configurada con una principal en la región centro-norte de EE. UU. y una secundaria en la región centro-sur de EE. UU.

![Relación de replicación geográfica](./media/active-geo-replication-overview/geo-replication-relationship.png)

Debido a que las bases de datos secundarias son legibles, se pueden usar para descargar cargas de trabajo de solo lectura como trabajos de informes. Si usa la replicación geográfica activa, la base de datos secundaria se puede crear en la misma región que la base de datos primaria; sin embargo, esto no aumenta la resistencia de la aplicación ante errores catastróficos. Si usa grupos de conmutación por error automática, las bases de datos secundarias siempre se crean en otra región.

Además de para la recuperación ante desastres, la replicación geográfica activa se puede usar en los escenarios siguientes:

- **Migración de base de datos**: puede usar la replicación geográfica activa para migrar una base de datos de un servidor a otro en línea con un tiempo de inactividad mínimo.
- **Actualizaciones de aplicaciones**: puede crear una base de datos secundaria adicional como copia de conmutación por recuperación durante las actualizaciones de aplicaciones.

Para lograr una verdadera continuidad empresarial, agregar redundancia de base de datos entre centros de datos es solo parte de la solución. Para recuperar una aplicación (un servicio) de un extremo a otro tras un error catastrófico, es necesario recuperar todos los componentes que constituyen el servicio y cualquier servicio dependiente. Algunos ejemplos de estos componentes son el software cliente (por ejemplo, un explorador con JavaScript personalizado), los front-end web, el almacenamiento y DNS. Es fundamental que todos los componentes sean resistentes a los mismos errores y que estén disponibles en el plazo del objetivo de tiempo de recuperación (RTO) de la aplicación. Por lo tanto, debe identificar todos los servicios dependientes y comprender las garantías y capacidades que ofrecen. A continuación, debe seguir los pasos adecuados para asegurarse de que el servicio funcione durante la conmutación por error de los servicios de los que depende. Para más información sobre el diseño de soluciones para la recuperación ante desastres, consulte [Diseño de soluciones de nube para la continuidad empresarial mediante la replicación geográfica activa](designing-cloud-solutions-for-disaster-recovery.md).

## <a name="active-geo-replication-terminology-and-capabilities"></a>Funcionalidades y terminología de la replicación geográfica activa

- **Replicación asincrónica automática**

  Solo se puede crear una base de datos secundaria mediante su adición a una existente. La base de datos secundaria se puede crear en cualquier servidor. Una vez creada, la base de datos secundaria se rellena con los datos copiados de la base de datos principal. Este proceso se conoce como propagación. Después de crear y propagar la base de datos secundaria, las actualizaciones de la base de datos principal se replican asincrónicamente en la base de datos secundaria de forma automática. La replicación asincrónica significa que las transacciones se confirman en la base de datos principal antes de replicarse en la secundaria.

- **Bases de datos secundarias legibles**

  Una aplicación puede acceder a una base de datos secundaria para operaciones de solo lectura con las mismas entidades de seguridad que las usadas para tener acceso a la base de datos principal u otras diferentes. Las bases de datos secundarias funcionan en el modo de aislamiento de instantánea para asegurarse de que no se retrasa la replicación de las actualizaciones de la base de datos principal (reproducción de registro) por las consultas ejecutadas en la secundaria.

> [!NOTE]
> La reproducción de registros se retrasa en la base de datos secundaria si hay actualizaciones del esquema en el servidor principal. Lo último requiere un bloqueo del esquema en la base de datos secundaria.

> [!IMPORTANT]
> Puede usar la replicación geográfica para crear una base de datos secundaria en la misma región que la principal. Puede utilizar esta base de datos secundaria para equilibrar cargas de trabajo de solo lectura en la misma región. Sin embargo, una base de datos secundaria en la misma región no proporciona resistencia adicional a los errores y, por lo tanto, no es un objetivo de conmutación por error adecuado para la recuperación ante desastres. Tampoco garantiza el aislamiento de la zona de disponibilidad. Use el nivel de servicio Premium o Crítico para la empresa con [configuración con redundancia de zona](high-availability-sla.md#zone-redundant-configuration) para lograr el aislamiento de la zona de disponibilidad.
>

- **Conmutación por error planeada**

  La conmutación por error planeada cambia los roles de las bases de datos principales y secundarias una vez que se ha llevado a cabo la sincronización completa. Se trata de una operación en línea que no provoca pérdida de datos. La duración de la operación depende del tamaño del registro de transacciones en la base de datos principal que debe sincronizarse. La conmutación por error planeada está diseñada para los siguientes escenarios: (a) para realizar simulacros de recuperación ante desastres en producción cuando la pérdida de datos no es aceptable; (b) para reubicar la base de datos en una región diferente; y (c) para devolver la base de datos a la región primaria después de que se ha mitigado la interrupción (conmutación por recuperación).

- **Conmutación por error no planeada**

  La conmutación por error no planeada o forzada cambia inmediatamente el rol principal a la base de datos secundaria sin sincronización con la principal. Se perderán todas las transacciones confirmadas en la base de datos principal que no se hayan replicado en la secundaria. Esta operación está diseñada como método de recuperación durante las interrupciones cuando no se puede acceder a la principal, pero la disponibilidad de la base de datos debe restaurarse rápidamente. Cuando la base de datos principal original vuelve a estar en línea, se vuelve a conectar automáticamente y se convierte en una nueva base de datos secundaria. Todas las transacciones no sincronizadas antes de la conmutación por error se conservarán en el archivo de copia de seguridad, pero no se sincronizarán con la nueva base de datos principal para evitar conflictos. Estas transacciones tendrán que combinarse manualmente con la versión más reciente de la base de datos principal.

- **Varias bases de datos secundarias**

  Pueden crearse hasta cuatro bases de datos secundarias por cada principal. Si solo hay una base de datos secundaria y se produce un error, la aplicación está expuesta a un mayor riesgo hasta que se crea una nueva base de datos secundaria. Si existen varias bases de datos secundarias, la aplicación sigue estando protegida incluso si se produce un error en una de ellas. También se pueden usar bases de datos secundarias adicionales para escalar horizontalmente las cargas de trabajo de solo lectura

  > [!NOTE]
  > Si usa la replicación geográfica activa para compilar una aplicación distribuida globalmente y necesita proporcionar acceso de solo lectura a los datos en más de cuatro regiones, puede crear una base de datos secundaria a partir de otra base de datos secundaria (este proceso se conoce como encadenamiento). De este modo, puede alcanzar una escala prácticamente ilimitada de replicación de la base de datos. Además, el encadenamiento disminuye la sobrecarga de la replicación desde la base de datos principal. El inconveniente es el mayor retraso de replicación en las últimas bases de datos secundarias.

- **Replicación geográfica de bases de datos en un grupo elástico**

  Cada base de datos secundaria puede participar por separado en un grupo elástico o no estar en ningún grupo elástico. La opción de grupo de cada base de datos secundaria es independiente y no depende de la configuración de ninguna otra base de datos secundaria (ya sea principal o secundaria). Cada grupo elástico se encuentra dentro de una única región; por lo tanto, varias bases de datos secundarias de la misma topología nunca pueden compartir un grupo elástico.

- **Conmutación por error y conmutación por recuperación controladas por el usuario**

  La aplicación o el usuario puede cambiar explícitamente una base de datos secundaria al rol principal en cualquier momento. Durante una interrupción real, debe utilizarse la opción "no planeada", que promueve inmediatamente una base de datos secundaria para que sea la principal. Cuando la base de datos principal que generó el error se recupera y vuelve a estar disponible, el sistema la marca automáticamente como secundaria y la pone al día con la nueva base de datos principal. Por la naturaleza asincrónica de la replicación, se puede perder una pequeña cantidad de datos durante las conmutaciones por error no planeadas antes de que se repliquen los cambios más recientes a la base de datos secundaria. Cuando una base de datos principal con varias secundarias conmuta por error, el sistema vuelve a configurar automáticamente las relaciones de replicación y vincula las bases de datos secundarias restantes a la principal recién promovida sin necesidad de que intervenga el usuario. Una vez que se mitiga la interrupción que causó la conmutación por error, sería conveniente devolver la aplicación a la región primaria. Para ello, se debe invocar el comando de conmutación por error con la opción "planeada".

## <a name="preparing-secondary-database-for-failover"></a>Preparación de la base de datos secundaria para la conmutación por error

Para asegurarse de que la aplicación pueda acceder de inmediato a la nueva base de datos principal después de la conmutación por error, compruebe que los requisitos de autenticación de la base de datos y el servidor secundarios estén configurados correctamente. Para obtener más información, consulte [Administración de la seguridad de Azure SQL Database después de la recuperación ante desastres](active-geo-replication-security-configure.md). Para garantizar el cumplimiento después de la conmutación por error, asegúrese de que la directiva de retención de copias de seguridad de la base de datos secundaria coincide con la de la principal. Esta configuración no forma parte de la base de datos y no se replica. De forma predeterminada, la base de datos secundaria se configurará con un período de retención PITR predeterminado de siete días. Para más información, consulte [Copias de seguridad automatizadas de SQL Database](automated-backups-overview.md).

> [!IMPORTANT]
> Si la base de datos es miembro de un grupo de conmutación por error, no puede iniciar su conmutación por error mediante el comando de conmutación por error de replicación geográfica. Use el comando de conmutación por error para el grupo. Si necesita realizar la conmutación por error de una base de datos individual, primero debe quitarla del grupo de conmutación por error. Consulte [grupos de conmutación por error](auto-failover-group-overview.md) para más información.

## <a name="configuring-secondary-database"></a>Configuración de una base de datos secundaria

Es necesario que tanto la base de datos principal como las secundarias tengan el mismo nivel de servicio. También se recomienda encarecidamente que la base de datos secundaria se cree con el mismo tamaño de proceso (unidades de transacción de base de datos o núcleos virtuales) que la principal. Si la base de datos principal está experimentando una carga de trabajo de escritura intensiva, es posible que una base de datos secundaria con un tamaño de proceso menor no pueda mantenerla. Provocará el retardo de la fase de puesta al día en la base de datos secundaria y una posible falta de disponibilidad. Para mitigar estos riesgos, una replicación geográfica activa limitará la velocidad de los registros de transacción de la base de datos principal si es necesario para permitir que sus secundarias se pongan al día.

Otra consecuencia de una configuración de base de datos secundaria no equilibrada es que después de la conmutación por error, el rendimiento de la aplicación puede verse afectado debido a una capacidad de proceso insuficiente de la nueva base de datos principal. En ese caso, se deberá escalar verticalmente el objetivo del servicio de base de datos al nivel necesario, lo que puede tardar mucho tiempo y requerir un volumen considerable de recursos de proceso, así como una conmutación por error de [alta disponibilidad](high-availability-sla.md) al final del proceso de escalado vertical.

Si decide crear la base de datos secundaria con un tamaño de proceso más bajo, el gráfico de porcentaje de E/S de registro en Azure Portal proporciona un buen método para estimar el tamaño de proceso mínimo de la base de datos secundaria que es necesario para mantener la carga de replicación. Por ejemplo, si la base de datos principal es P6 (1000 DTU) y su porcentaje de escritura de registro es 50 %, la base de datos secundaria debe ser al menos P4 (500 DTU). Para recuperar datos históricos de E/S del registro, use la vista [sys.resource_stats](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database). Para recuperar los datos de escritura de registro recientes con una granularidad mayor que refleje mejor los picos a corto plazo en la velocidad de registro, utilice la vista [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database).

La limitación de la velocidad del registro de transacciones en la base de datos principal debido a un tamaño de proceso inferior en una base de datos secundaria se envía mediante el tipo de espera HADR_THROTTLE_LOG_RATE_MISMATCHED_SLO, visible en las vistas de bases de datos [sys.dm_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql) y [sys.dm_os_wait_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-os-wait-stats-transact-sql).

> [!NOTE]
> La velocidad del registro de transacciones de la base de datos principal puede estar limitada por motivos no relacionados con el tamaño de proceso inferior de una base de datos secundaria. Este tipo de limitación puede producirse aunque la base de datos secundaria tenga el mismo tamaño de proceso que la principal o uno superior. Para obtener más información, incluidos los tipos de espera para diferentes tipos de limitaciones de velocidad de registro, vea [Gobernanza de la velocidad del registro de transacciones](resource-limits-logical-server.md#transaction-log-rate-governance).

Para más información sobre los tamaños de proceso de SQL Database, consulte [¿Qué son los niveles de servicio de SQL Database?](purchasing-models.md)

## <a name="cross-subscription-geo-replication"></a>Replicación geográfica entre suscripciones

Para configurar la replicación geográfica activa entre dos bases de datos que pertenecen a distintas suscripciones (ya sea en el mismo inquilino o en otro), debe seguir el procedimiento especial descrito en esta sección.  El procedimiento se basa en comandos SQL y requiere:

- La creación de un inicio de sesión con privilegios en ambos servidores.
- La adición de la dirección IP a la lista de permitidos del cliente que realiza el cambio en ambos servidores (como la dirección IP del host que ejecuta SQL Server Management Studio).

El cliente que realiza los cambios necesita acceso de red al servidor principal. Aunque se debe agregar la misma dirección IP del cliente a la lista de permitidos en el servidor secundario, la conectividad de red con el servidor secundario no es estrictamente necesaria.

### <a name="on-the-master-of-the-primary-server"></a>En la base de datos maestra del servidor principal

1. Agregue la dirección IP a la lista de permitidos del cliente que realiza los cambios (para más información, consulte [Configuración del firewall](firewall-configure.md)).
1. Cree un inicio de sesión dedicado para configurar la replicación geográfica activa (y ajuste las credenciales según sea necesario):

   ```sql
   create login geodrsetup with password = 'ComplexPassword01'
   ```

1. Cree un usuario correspondiente y asígnele el rol dbmanager:

   ```sql
   create user geodrsetup for login geodrsetup
   alter role dbmanager add member geodrsetup
   ```

1. Anote el SID del nuevo inicio de sesión con esta consulta:

   ```sql
   select sid from sys.sql_logins where name = 'geodrsetup'
   ```

### <a name="on-the-source-database-on-the-primary-server"></a>En la base de datos de origen del servidor principal

1. Cree un usuario para el mismo inicio de sesión:

   ```sql
   create user geodrsetup for login geodrsetup
   ```

1. Agregue el usuario al rol db_owner:

   ```sql
   alter role db_owner add member geodrsetup
   ```

### <a name="on-the-master-of-the-secondary-server"></a>En la base de datos maestra del servidor secundario

1. Agregue la dirección IP a la lista de permitidos del cliente que realiza los cambios. Debe ser exactamente la misma dirección IP del servidor principal.
1. Cree el mismo inicio de sesión que en el servidor principal, con el mismo nombre de usuario, contraseña y SID:

   ```sql
   create login geodrsetup with password = 'ComplexPassword01', sid=0x010600000000006400000000000000001C98F52B95D9C84BBBA8578FACE37C3E
   ```

1. Cree un usuario correspondiente y asígnele el rol dbmanager:

   ```sql
   create user geodrsetup for login geodrsetup;
   alter role dbmanager add member geodrsetup
   ```

### <a name="on-the-master-of-the-primary-server"></a>En la base de datos maestra del servidor principal

1. Inicie sesión en la base de datos maestra del servidor principal con el nuevo inicio de sesión.
1. Cree una réplica secundaria de la base de datos de origen en el servidor secundario (ajuste el nombre de la base de datos y del servidor según sea necesario):

   ```sql
   alter database dbrep add secondary on server <servername>
   ```

Tras la configuración inicial, se pueden quitar los usuarios, los inicios de sesión y las reglas de firewall creados.

## <a name="keeping-credentials-and-firewall-rules-in-sync"></a>Mantenimiento de las credenciales y las reglas de firewall sincronizadas

Se recomienda usar [reglas de firewall de direcciones IP de nivel de base de datos](firewall-configure.md) para las bases de datos con replicación geográfica. Así, estas reglas se pueden replicar con la base de datos para garantizar que todas las bases de datos secundarias tengan las mismas reglas de firewall que la principal. Este enfoque elimina la necesidad de que los clientes configuren y mantengan manualmente las reglas de firewall en los servidores que hospedan tanto la base de datos principal como las secundarias. Igualmente, la utilización de [usuarios de base de datos independiente](logins-create-manage.md) para el acceso a los datos garantiza que la base de datos principal y las secundarias tengan siempre las mismas credenciales de usuario. Por tanto, durante una conmutación por error, no hay interrupciones debidas a discrepancias en los inicios de sesión y las contraseñas. Con la adición de [Azure Active Directory](../../active-directory/fundamentals/active-directory-whatis.md), los clientes pueden administrar el acceso de usuarios a la base de datos principal y a las secundarias, por lo que ya no es necesario administrar credenciales en las bases de datos.

## <a name="upgrading-or-downgrading-primary-database"></a>Actualización o degradación de una base de datos principal

Puede actualizar o degradar una base de datos principal a un tamaño de proceso diferente (en el mismo nivel de servicio, no entre De uso general y Crítico para la empresa) sin desconectar las bases de datos secundarias. Al actualizar, se recomienda que actualice la base de datos secundaria en primer lugar y, a continuación, la principal. Al degradar, invierta el orden: degrade primero la base de datos principal y, después, la secundaria. Cuando actualiza la base de datos a un nivel de servicio diferente, o la cambia a una versión anterior, se aplica esta recomendación.

> [!NOTE]
> Si creó una base de datos secundaria como parte de la configuración del grupo de conmutación por error, no se recomienda que la degrade. De este modo, se garantiza que la capa de datos tiene la capacidad suficiente para procesar la carga de trabajo habitual una vez que se activa la conmutación por error.

> [!IMPORTANT]
> La base de datos principal de un grupo de conmutación por error no puede escalar a un nivel superior a menos que la base de datos secundaria se escale primero al nivel superior. Si intenta escalar la base de datos principal antes de escalar la base de datos secundaria, es posible que reciba el siguiente error:
>
> `Error message: The source database 'Primaryserver.DBName' cannot have higher edition than the target database 'Secondaryserver.DBName'. Upgrade the edition on the target before upgrading the source.`
>

## <a name="preventing-the-loss-of-critical-data"></a>Evitar la pérdida de datos críticos

Debido a la elevada latencia de las redes de área extensa, la copia continua usa un mecanismo de replicación asincrónica. La replicación asincrónica hace inevitable la pérdida de cierta cantidad datos si se produce un error. Sin embargo, es posible que algunas aplicaciones exijan que no se pierdan datos. Para proteger estas actualizaciones críticas, un desarrollador de aplicaciones puede llamar al procedimiento del sistema [sp_wait_for_database_copy_sync](/sql/relational-databases/system-stored-procedures/active-geo-replication-sp-wait-for-database-copy-sync) inmediatamente después de confirmar la transacción. La llamada a **sp_wait_for_database_copy_sync** bloquea el subproceso de llamada hasta que se transmite la última transacción confirmada en la base de datos secundaria. Sin embargo, no espera para que las transacciones transmitidas se reproduzcan y se confirmen en la base de datos secundaria. **sp_wait_for_database_copy_sync** está dirigida a un vínculo de copia continua específico. Cualquier usuario con derechos de conexión para la base de datos principal puede llamar a este procedimiento.

> [!NOTE]
> **sp_wait_for_database_copy_sync** evita la pérdida de datos después de la conmutación por error, pero no garantiza la sincronización completa para el acceso de lectura. El retraso causado por una llamada al procedimiento **sp_wait_for_database_copy_sync** puede ser considerable y depende del tamaño del registro de transacciones en el momento de la llamada.

## <a name="monitoring-geo-replication-lag"></a>Supervisión del retardo de la replicación geográfica

Para supervisar el retardo con respecto a RPO, utilice la columna *replication_lag_sec* de [sys.dm_geo_replication_link_status](/sql/relational-databases/system-dynamic-management-views/sys-dm-geo-replication-link-status-azure-sql-database) en la base de datos principal. Muestra el retardo en segundos entre las transacciones confirmadas en la base de datos principal y las que persisten en la secundaria. Por ejemplo, si el valor del retardo es de 1 segundo, significa que si la base de datos principal se ve afectada por una interrupción en este momento y se inicia la conmutación por error, no se guardará 1 segundo de las transacciones más recientes.

Para medir el retardo con respecto a los cambios en la base de datos principal que se han aplicado en la base de datos secundaria, es decir, que están disponibles para su lectura desde la base de datos secundaria, compare el tiempo de *last_commit* en la base de datos secundaria con el mismo valor en la base de datos principal.

> [!NOTE]
> A veces *replication_lag_sec* en la base de datos principal tiene un valor NULL, lo que significa que la base de datos principal no sabe actualmente lo lejos que está la secundaria.   Esto ocurre típicamente después de reiniciar el proceso y debería ser una condición temporal. Considere la posibilidad de alertar a la aplicación si *replication_lag_sec* devuelve NULL durante un período prolongado. Esto indicaría que la base de datos secundaria no puede comunicarse con la principal debido a un error de conectividad permanente. También hay condiciones que podrían hacer que la diferencia entre el tiempo de *last_commit* en la base de datos secundaria y en la base de datos principal sea mayor. Por ejemplo, si se realiza una confirmación en la base de datos principal después de un largo período sin cambios, la diferencia saltará a un valor grande antes de volver rápidamente a 0. Considéralo una condición de error cuando la diferencia entre estos dos valores permanece grande durante mucho tiempo.

## <a name="programmatically-managing-active-geo-replication"></a>Administración mediante programación de la replicación geográfica activa

Como se dijo antes, la replicación geográfica activa también puede administrarse mediante programación con Azure PowerShell y la API REST. En las tablas siguientes se describe el conjunto de comandos disponibles. La replicación geográfica activa incluye un conjunto de API de Azure Resource Manager para la administración, en el que se incluyen la [API REST de Azure SQL Database](https://docs.microsoft.com/rest/api/sql/) y los [cmdlets de Azure PowerShell](https://docs.microsoft.com/powershell/azure/). Estas API requieren que se usen grupos de recursos y admiten la seguridad basada en roles (RBAC). Para más información sobre cómo implementar los roles de acceso, consulte [Control de acceso basado en roles de Azure (Azure RBAC)](../../role-based-access-control/overview.md).

### <a name="t-sql-manage-failover-of-single-and-pooled-databases"></a>T-SQL: Administración de la conmutación por error de bases de datos únicas y agrupadas

> [!IMPORTANT]
> Estos comandos de Transact-SQL solo se aplican a la replicación geográfica activa, no a los grupos de conmutación por error. Por lo tanto, tampoco se aplican a las Instancias administradas de SQL, ya que solo admiten grupos de conmutación por error.

| Get-Help | Descripción |
| --- | --- |
| [ALTER DATABASE](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current) |Se utiliza el argumento ADD SECONDARY ON SERVER a fin de crear una base de datos secundaria para una base de datos existente e iniciar la replicación de datos |
| [ALTER DATABASE](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current) |Se utiliza FAILOVER o FORCE_FAILOVER_ALLOW_DATA_LOSS para cambiar una base de datos de secundaria a principal e iniciar la conmutación por error. |
| [ALTER DATABASE](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current) |Se utiliza REMOVE SECONDARY ON SERVER para finalizar una replicación de datos entre una instancia de SQL Database y la base de datos secundaria especificada. |
| [sys.geo_replication_links](/sql/relational-databases/system-dynamic-management-views/sys-geo-replication-links-azure-sql-database) |Devuelve información sobre todos los vínculos de replicación existentes para cada base de datos en un servidor. |
| [sys.dm_geo_replication_link_status](/sql/relational-databases/system-dynamic-management-views/sys-dm-geo-replication-link-status-azure-sql-database) |Obtiene la hora de la última replicación, el retraso de la última replicación y otro tipo de información sobre el vínculo de replicación para una base de datos determinada. |
| [sys.dm_operation_status](/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database) |Muestra el estado de todas las operaciones de base de datos, incluido el estado de los vínculos de replicación. |
| [sp_wait_for_database_copy_sync](/sql/relational-databases/system-stored-procedures/active-geo-replication-sp-wait-for-database-copy-sync) |Hace que la aplicación espere a que se repliquen todas las transacciones confirmadas y a que las reconozca la base de datos secundaria activa. |
|  | |

### <a name="powershell-manage-failover-of-single-and-pooled-databases"></a>PowerShell: Administración de la conmutación por error de bases de datos únicas y agrupadas

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> El módulo de Azure Resource Manager para PowerShell todavía es compatible con Azure SQL Database, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. Para estos cmdlets, consulte [AzureRM.Sql](https://docs.microsoft.com/powershell/module/AzureRM.Sql/). Los argumentos para los comandos del módulo Az y los módulos AzureRm son esencialmente idénticos.

| Cmdlet | Descripción |
| --- | --- |
| [Get-AzSqlDatabase](https://docs.microsoft.com/powershell/module/az.sql/get-azsqldatabase) |Obtiene una o más bases de datos. |
| [New-AzSqlDatabaseSecondary](https://docs.microsoft.com/powershell/module/az.sql/new-azsqldatabasesecondary) |Crea una base de datos secundaria para una base de datos existente e inicia la replicación de datos. |
| [Set-AzSqlDatabaseSecondary](https://docs.microsoft.com/powershell/module/az.sql/set-azsqldatabasesecondary) |Convierte una base de datos secundaria en principal para iniciar la conmutación por error. |
| [Remove-AzSqlDatabaseSecondary](https://docs.microsoft.com/powershell/module/az.sql/remove-azsqldatabasesecondary) |Finaliza una replicación de datos entre SQL Database y la base de datos secundaria especificada. |
| [Get-AzSqlDatabaseReplicationLink](https://docs.microsoft.com/powershell/module/az.sql/get-azsqldatabasereplicationlink) |Obtiene los vínculos de replicación geográfica entre una instancia de Azure SQL Database y un grupo de recursos o servidor SQL lógico. |
|  | |

> [!IMPORTANT]
> Si desea scripts de ejemplo, consulte [Configuración y conmutación por error de una base de datos única mediante la replicación geográfica activa](scripts/setup-geodr-and-failover-database-powershell.md) y [Configuración y conmutación por error de una base de datos agrupada mediante la replicación geográfica activa](scripts/setup-geodr-and-failover-elastic-pool-powershell.md).

### <a name="rest-api-manage-failover-of-single-and-pooled-databases"></a>API REST: Administración de la conmutación por error de bases de datos únicas y agrupadas

| API | Descripción |
| --- | --- |
| [Crear o actualizar base de datos (createMode=Restore)](https://docs.microsoft.com/rest/api/sql/databases/createorupdate) |Crea, actualiza o restaura una base de datos principal o secundaria. |
| [Obtener el estado de creación o actualización de la base de datos](https://docs.microsoft.com/rest/api/sql/databases/createorupdate) |Devuelve el estado durante una operación de creación. |
| [Establecer la base de datos secundaria como principal (conmutación por error planeada)](https://docs.microsoft.com/rest/api/sql/replicationlinks/failover) |Define qué base de datos secundaria es principal mediante la conmutación por error desde la base de datos principal. **Esta opción no se admite para Instancia administrada de SQL.**|
| [Establecer la base de datos secundaria como principal (conmutación por error no planeada)](https://docs.microsoft.com/rest/api/sql/replicationlinks/failoverallowdataloss) |Define qué base de datos secundaria es principal mediante la conmutación por error desde la base de datos principal. Esta operación puede ocasionar pérdida de datos. **Esta opción no se admite para Instancia administrada de SQL.**|
| [Obtener vínculo de replicación](https://docs.microsoft.com/rest/api/sql/replicationlinks/get) |Obtiene un vínculo de replicación específico para una base de datos determinada en una asociación de replicación geográfica. Recupera la información visible en la vista de catálogo sys.geo_replication_links. **Esta opción no se admite para Instancia administrada de SQL.**|
| [Vínculos de replicación: lista por base de datos](https://docs.microsoft.com/rest/api/sql/replicationlinks/listbydatabase) | Obtiene todos los vínculos de replicación para una base de datos determinada en una asociación de replicación geográfica. Recupera la información visible en la vista de catálogo sys.geo_replication_links. |
| [Eliminar vínculo de replicación](https://docs.microsoft.com/rest/api/sql/replicationlinks/delete) | Elimina un vínculo de replicación de base de datos. No se puede realizar durante la conmutación por error. |
|  | |

## <a name="next-steps"></a>Pasos siguientes

- Para los scripts de ejemplo, vea:
  - [Configuración y conmutación por error de una base de datos única mediante la replicación geográfica activa](scripts/setup-geodr-and-failover-database-powershell.md)
  - [Configuración y conmutación por error de una base de datos agrupada mediante la replicación geográfica activa](scripts/setup-geodr-and-failover-elastic-pool-powershell.md)
- SQL Database también admite grupos de conmutación por error automática. Para más información, consulte cómo se usan los [grupos de conmutación por error automática](auto-failover-group-overview.md).
- Para obtener una descripción general y los escenarios de la continuidad empresarial, consulte [Continuidad empresarial con Base de datos SQL de Azure](business-continuity-high-availability-disaster-recover-hadr-overview.md)
- Para saber en qué consisten las copias de seguridad automatizadas de Azure SQL Database, consulte [Información general: copias de seguridad automatizadas de SQL Database](automated-backups-overview.md).
- Si quiere saber cómo usar las copias de seguridad automatizadas para procesos de recuperación, consulte [Recuperación de una base de datos a partir de copias de seguridad iniciadas por un servicio](recovery-using-backups.md).
- Para obtener información acerca de los requisitos de autenticación para un nuevo servidor principal y la base de datos, consulte [Administración de la seguridad de Azure SQL Database después de la recuperación ante desastres](active-geo-replication-security-configure.md).
