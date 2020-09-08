---
title: Replicación transaccional
titleSuffix: Azure SQL Managed Instance
description: Aprenda a usar la replicación transaccional de SQL Server con Instancia administrada de Azure SQL.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: data-movement
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: MashaMSFT
ms.author: mathoma
ms.reviewer: carlrab
ms.date: 04/20/2020
ms.openlocfilehash: ec1dfa3edea5364151c543889d974944a1a1cd5a
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87920133"
---
# <a name="transactional-replication-with-azure-sql-managed-instance"></a>Replicación transaccional con Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

La replicación transaccional es una característica de Instancia administrada de Azure SQL y SQL Server que permite replicar datos de una tabla de Instancia administrada de Azure SQL o SQL Server en tablas colocadas en bases de datos remotas. Esta característica permite sincronizar varias tablas en bases de datos diferentes. 

La replicación transaccional está actualmente en versión preliminar pública para SQL Managed Instance. 

## <a name="overview"></a>Información general

Puede usar la replicación transaccional para insertar los cambios realizados en una instancia administrada de Azure SQL en:

- Una base de datos de SQL Server, en el entorno local o en una máquina virtual de Azure
- Una base de datos de Azure SQL Database
- Una base de datos de instancia en Azure SQL Managed Instance

  > [!NOTE]
  > Para usar todas las características de Instancia administrada de Azure SQL, debe tener las versiones más recientes de [SQL Server Management Studio (SSMS)](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) y [SQL Server Data Tools (SSDT)](https://docs.microsoft.com/sql/ssdt/download-sql-server-data-tools-ssdt).

### <a name="components"></a>Componentes

Los componentes clave de la replicación transaccional son el **publicador**, el **distribuidor** y el **suscriptor**, como se muestra en la siguiente imagen:  

![replicación con SQL Database](./media/replication-transactional-overview/replication-to-sql-database.png)

| Role | Azure SQL Database | Instancia administrada de Azure SQL |
| :----| :------------- | :--------------- |
| **Publicador** | No | Sí |
| **Distribuidor** | No | Sí|
| **Suscriptor de extracción** | No | Sí|
| **Suscriptor de inserción**| Sí | Sí|
| &nbsp; | &nbsp; | &nbsp; |

El **publicador** publica los cambios realizados en algunas tablas (artículos) mediante el envío de las actualizaciones al distribuidor. El publicador puede ser una instancia administrada de Azure SQL o una instancia de SQL Server.

El **distribuidor** recopila los cambios en los artículos de un publicador y los distribuye a los suscriptores. El distribuidor puede ser una instancia administrada de Azure SQL o una instancia de SQL Server (cualquier versión, siempre que sea igual o superior a la versión del publicador).

El **suscriptor** recibe los cambios realizados en el publicador. Una instancia de SQL Server y una instancia administrada de Azure SQL pueden ser suscriptores de inserción y de extracción, aunque una suscripción de extracción no se admite si el distribuidor es una instancia de administrada de Azure SQL y el suscriptor no. Una base de datos de Azure SQL Database solo puede ser un suscriptor de inserción.

Instancia administrada de Azure SQL puede ser un suscriptor de las siguientes versiones de SQL Server:

- SQL Server 2016 y posterior
- SQL Server 2014 [RTM CU10 (12.0.4427.24)](https://support.microsoft.com/help/3094220/cumulative-update-10-for-sql-server-2014) o [SP1 CU3 (12.0.2556.4)](https://support.microsoft.com/help/3094221/cumulative-update-3-for-sql-server-2014-service-pack-1)
- SQL Server 2012 [SP2 CU8 (11.0.5634.1)](https://support.microsoft.com/help/3082561/cumulative-update-8-for-sql-server-2012-sp2) o [SP3 (11.0.6020.0)](https://www.microsoft.com/download/details.aspx?id=49996)

   > [!NOTE]
   >
   > - Para otras versiones de SQL Server que no admiten la publicación en los objetos de Azure e puede utilizar el método de [volver a publicar datos](https://docs.microsoft.com/sql/relational-databases/replication/republish-data) para mover datos a versiones más recientes de SQL Server.
   > - Al intentar configurar la replicación con una versión anterior, puede producirse el error número MSSQL_REPL20084 (El proceso no pudo conectarse al suscriptor) y MSSQL_REPL40532 (No se puede abrir el servidor \<name> solicitado por el inicio de sesión. Error de inicio de sesión).

### <a name="types-of-replication"></a>Tipos de replicación

Existen distintos [tipos de replicación](https://docs.microsoft.com/sql/relational-databases/replication/types-of-replication):

| Replicación | Azure SQL Database | Instancia administrada de Azure SQL |
| :----| :------------- | :--------------- |
| [**Transaccional estándar**](https://docs.microsoft.com/sql/relational-databases/replication/transactional/transactional-replication) | Sí (solo como suscriptor) | Sí |
| [**Instantánea**](https://docs.microsoft.com/sql/relational-databases/replication/snapshot-replication) | Sí (solo como suscriptor) | Sí|
| [**Replicación de mezcla**](https://docs.microsoft.com/sql/relational-databases/replication/merge/merge-replication) | No | No|
| [**Punto a punto**](https://docs.microsoft.com/sql/relational-databases/replication/transactional/peer-to-peer-transactional-replication) | No | No|
| [**Bidireccional**](https://docs.microsoft.com/sql/relational-databases/replication/transactional/bidirectional-transactional-replication) | No | Sí|
| [**Suscripciones actualizables**](https://docs.microsoft.com/sql/relational-databases/replication/transactional/updatable-subscriptions-for-transactional-replication) | No | No|
| &nbsp; | &nbsp; | &nbsp; |

### <a name="supportability-matrix"></a>Matriz de compatibilidad

  La matriz de compatibilidad de la replicación transaccional con Instancia administrada de Azure SQL es la misma que con SQL Server.
  
| **Publicador**   | **Distribuidor** | **Suscriptor** |
| :------------   | :-------------- | :------------- |
| SQL Server 2019 | SQL Server 2019 | SQL Server 2019 <br/> SQL Server 2017 <br/> SQL Server 2016 <br/>  |
| SQL Server 2017 | SQL Server 2019 <br/>SQL Server 2017 | SQL Server 2019 <br/> SQL Server 2017 <br/> SQL Server 2016 <br/> SQL Server 2014 |
| SQL Server 2016 | SQL Server 2019 <br/>SQL Server 2017 <br/> SQL Server 2016 | SQL Server 2019 <br/> SQL Server 2017 <br/>SQL Server 2016 <br/> SQL Server 2014 <br/> SQL Server 2012 |
| SQL Server 2014 | SQL Server 2019 <br/> SQL Server 2017 <br/> SQL Server 2016 <br/> SQL Server 2014 <br/>| SQL Server 2017 <br/> SQL Server 2016 <br/> SQL Server 2014 <br/> SQL Server 2012 <br/> SQL Server 2008 R2 <br/> SQL Server 2008 |
| SQL Server 2012 | SQL Server 2019 <br/> SQL Server 2017 <br/> SQL Server 2016 <br/> SQL Server 2014 <br/>SQL Server 2012 <br/> | SQL Server 2016 <br/> SQL Server 2014 <br/> SQL Server 2012 <br/> SQL Server 2008 R2 <br/> SQL Server 2008 |
| SQL Server 2008 R2 <br/> SQL Server 2008 | SQL Server 2019 <br/> SQL Server 2017 <br/> SQL Server 2016 <br/> SQL Server 2014 <br/>SQL Server 2012 <br/> SQL Server 2008 R2 <br/> SQL Server 2008 |  SQL Server 2014 <br/> SQL Server 2012 <br/> SQL Server 2008 R2 <br/> SQL Server 2008 <br/>  |
| &nbsp; | &nbsp; | &nbsp; |

## <a name="when-to-use"></a>Cuándo se usa

La replicación transaccional resulta útil en los siguientes escenarios:

- Publique los cambios realizados en una o varias tablas de una base de datos y distribúyalos a una o varias bases de datos de una instancia de SQL Server o Azure SQL Database suscrita a los cambios.
- Mantenga varias bases de datos distribuidas en estado sincronizado.
- Migre las bases de datos de una instancia de SQL Server o Instancia administrada de Azure SQL a otra base de datos mediante la publicación continua de los cambios.

### <a name="compare-data-sync-with-transactional-replication"></a>Comparación de Data Sync con replicación transaccional

| Category | Sincronización de datos | Replicación transaccional |
|---|---|---|
| Ventajas | - Compatibilidad activo-activo<br/>- Bidireccional entre el entorno local y Azure SQL Database | - Menor latencia<br/>- Coherencia transaccional<br/>- Reutilización de la topología existente después de la migración |
| Inconvenientes | - 5 minutos o más de latencia<br/>- Sin coherencia transaccional<br/>- Mayor impacto en el rendimiento | - No se puede publicar desde Azure SQL Database <br/>- Alto costo de mantenimiento |

## <a name="common-configurations"></a>Configuraciones comunes

En general, el publicador y el distribuidor deben estar en la nube o en el entorno local. Se admiten las siguientes configuraciones:

### <a name="publisher-with-local-distributor-on-sql-managed-instance"></a>Publicador con distribuidor local en Instancia administrada de SQL

![Instancia única como publicador y distribuidor](./media/replication-transactional-overview/01-single-instance-asdbmi-pubdist.png)

El publicador y el distribuidor se configuran en una sola instancia administrada de SQL y los cambios se distribuyen a otra instancia administrada de SQL, SQL Database o SQL Server.

### <a name="publisher-with-remote-distributor-on-sql-managed-instance"></a>Publicador con distribuidor remoto en Instancia administrada de SQL

En esta configuración, una instancia administrada publica los cambios en un distribuidor que se encuentra en otra instancia administrada de SQL que puede atender a muchas instancias administradas de SQL de origen y distribuir los cambios a uno o varios destinos de Azure SQL Database, Instancia administrada de Azure SQL o SQL Server.

![Instancias independientes para el publicador y el distribuidor](./media/replication-transactional-overview/02-separate-instances-asdbmi-pubdist.png)

El publicador y el distribuidor se configuran en dos Instancias administradas. Esta configuración presenta algunas restricciones:

- Las dos instancias administradas están en la misma red virtual.
- Las dos Instancias administradas están en la misma ubicación.

### <a name="on-premises-publisherdistributor-with-remote-subscriber"></a>Publicador o distribuidor local con un suscriptor remoto

![Azure SQL Database como suscriptor](./media/replication-transactional-overview/03-azure-sql-db-subscriber.png)

En esta configuración, una base de datos de Azure SQL Database o Instancia administrada de Azure SQL es un suscriptor. Esta configuración admite la migración desde el entorno local a Azure. Si un suscriptor es una base de datos de Azure SQL Database, debe estar en modo de inserción.  

## <a name="requirements"></a>Requisitos

- Use la autenticación de SQL para la conectividad entre los participantes en la replicación.
- Use un recurso compartido de cuenta de Azure Storage para el directorio de trabajo empleado por la replicación.
- Abra el puerto de salida TCP 445 en las reglas de seguridad de la subred para acceder al recurso compartido de archivos de Azure.
- Abra el puerto de salida TCP 1433 cuando Instancia administrada de SQL sea el publicador o el distribuidor y no el suscriptor. Es posible que también deba cambiar la regla de seguridad de salida del grupo de seguridad de red de Instancia administrada de SQL en `allow_linkedserver_outbound` de la **Etiqueta de servicio de destino** del puerto 1433 de `virtualnetwork` a `internet`.
- Coloque el publicador y el distribuidor en la nube, o ambos en local.
- Configure el emparejamiento de VPN entre las redes virtuales de los participantes en la replicación si las redes virtuales son diferentes.

> [!NOTE]
> Podría producirse el error 53 al conectarse a un archivo de Azure Storage si el puerto 445 del grupo de seguridad de red (NSG) de salida está bloqueado cuando el distribuidor es una base de datos de Instancia administrada de Azure SQL y el suscriptor es local. [Actualice el NSG de la red virtual](/azure/storage/files/storage-troubleshoot-windows-file-connection-problems) para resolver este problema.

## <a name="with-failover-groups"></a>Con grupos de conmutación por error

La [replicación geográfica activa](../database/active-geo-replication-overview.md) no es compatible con una instancia administrada de SQL con replicación transaccional. En lugar de replicación geográfica activa, use [grupos de conmutación por error automática](../database/auto-failover-group-overview.md), pero tenga en cuenta que la publicación se debe [eliminar manualmente](transact-sql-tsql-differences-sql-server.md#replication) de la instancia administrada principal y volver a crearse en la instancia administrada de SQL secundaria después de la conmutación por error.

Si la replicación geográfica está habilitada en un **publicador** o **distribuidor** de Instancia administrada de SQL en un [grupo de conmutación por error](../database/auto-failover-group-overview.md), el administrador de Instancia administrada de SQL debe limpiar todas las publicaciones de la instancia principal anterior y volver a configurarlas en la nueva instancia principal después de una conmutación por error. En este escenario, debe llevar a cabo las siguientes acciones:

1. Detenga todos los trabajos de replicación que se ejecutan en la base de datos, si hay alguno.
1. Quite los metadatos de suscripción del publicador. Para ello, ejecute el siguiente script en la base de datos del publicador:

   ```sql
   EXEC sp_dropsubscription @publication='<name of publication>', @article='all',@subscriber='<name of subscriber>'
   ```

1. Quite los metadatos de suscripción del suscriptor. Ejecute el siguiente script en la base de datos de suscripciones de la instancia administrada de SQL de suscriptor:

   ```sql
   EXEC sp_subscription_cleanup
      @publisher = N'<full DNS of publisher, e.g. example.ac2d23028af5.database.windows.net>',
      @publisher_db = N'<publisher database>',
      @publication = N'<name of publication>';
   ```

1. Quite forzosamente todos los objetos de replicación del publicador. Para ello, ejecute el siguiente script en la base de datos publicada:

   ```sql
   EXEC sp_removedbreplication
   ```

1. Quite forzosamente el distribuidor anterior de la instancia administrada de SQL principal original (si realiza la conmutación por recuperación a una instancia principal anterior que solía tener un distribuidor). Ejecute el siguiente script en la base de datos maestra de la instancia administrada de SQL de distribuidor anterior:

   ```sql
   EXEC sp_dropdistributor 1,1
   ```

Si la replicación geográfica está habilitada en una instancia de **suscriptor** de un grupo de conmutación por error, la publicación debe configurarse para conectarse al punto de conexión del cliente de escucha del grupo de conmutación por error para la instancia administrada del suscriptor. En el caso de una conmutación por error, la acción posterior por parte del administrador de instancia administrada depende del tipo de conmutación por error que se produjo:

- Para una conmutación por error sin pérdida de datos, la replicación seguirá funcionando después de la conmutación por error.
- En el caso de una conmutación por error con pérdida de datos, la replicación también funcionará. Se replicarán de nuevo los cambios perdidos.
- En una conmutación por error con pérdida de datos fuera del período de retención de la base de datos de distribución, el administrador de Instancia administrada de SQL tiene que reinicializar la base de datos de suscripciones.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre la configuración de la replicación transaccional, vea los siguientes tutoriales:

- [Configuración de la replicación entre un publicador y un suscriptor de Instancia administrada de SQL](../managed-instance/replication-between-two-instances-configure-tutorial.md)
- [Configuración de la replicación entre un publicador de Instancia administrada de SQL, un distribuidor de Instancia administrada de SQL y un suscriptor de SQL Server](../managed-instance/replication-two-instances-and-sql-server-configure-tutorial.md)
- [Cree una publicación](https://docs.microsoft.com/sql/relational-databases/replication/publish/create-a-publication).
- [Creación de una suscripción de inserción](https://docs.microsoft.com/sql/relational-databases/replication/create-a-push-subscription) mediante el nombre del servidor como suscriptor (por ejemplo, `N'azuresqldbdns.database.windows.net`) y el nombre de Azure SQL Database como base de datos de destino (por ejemplo, **AdventureWorks**. )

## <a name="see-also"></a>Consulte también  

- [Replicación con Instancia administrada de SQL y un grupo de conmutación por error](transact-sql-tsql-differences-sql-server.md#replication)
- [Replicación en SQL Database](../database/replication-to-sql-database.md)
- [Replicación en una instancia administrada](../managed-instance/replication-between-two-instances-configure-tutorial.md)
- [Create a Publication](https://docs.microsoft.com/sql/relational-databases/replication/publish/create-a-publication) (Creación de una publicación)
- [Create a Push Subscription](https://docs.microsoft.com/sql/relational-databases/replication/create-a-push-subscription/) (Creación de una suscripción de inserción)
- [Tipos de replicación](https://docs.microsoft.com/sql/relational-databases/replication/types-of-replication)
- [Supervisión (replicación)](https://docs.microsoft.com/sql/relational-databases/replication/monitor/monitoring-replication)
- [Inicializar una suscripción](https://docs.microsoft.com/sql/relational-databases/replication/initialize-a-subscription)  
