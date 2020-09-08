---
title: Diferencias de T-SQL entre SQL Server y una Instancia administrada de Azure SQL
description: En este artículo se describen las diferencias de Transact-SQL (T-SQL) entre una Instancia administrada de Azure SQL y SQL Server.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.devlang: ''
ms.topic: conceptual
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein, carlrab, bonova, danil
ms.date: 06/02/2020
ms.custom: seoapril2019, sqldbrb=1
ms.openlocfilehash: d611fc7eff2efa7a632f4b5467b5829a8374b95e
ms.sourcegitcommit: e0785ea4f2926f944ff4d65a96cee05b6dcdb792
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88705392"
---
# <a name="t-sql-differences-between-sql-server--azure-sql-managed-instance"></a>Diferencias de T-SQL entre SQL Server y una Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Este artículo resume y explica las diferencias de sintaxis y comportamiento entre Instancia administrada de Azure SQL y SQL Server. 


Instancia administrada de SQL proporciona una alta compatibilidad con el motor de base de datos de SQL Server, y la mayoría de las características se admiten en una Instancia administrada de SQL.

![Migración sencilla desde SQL Server](./media/transact-sql-tsql-differences-sql-server/migration.png)

Existen algunas limitaciones de PaaS que se introdujeron en Instancia administrada de SQL y algunos cambios de comportamiento en comparación con SQL Server. Las diferencias se dividen en las siguientes categorías: <a name="Differences"></a>

- [Disponibilidad](#availability), incluye las diferencias en [Grupos de disponibilidad AlwaysOn](#always-on-availability-groups) y [Copias de seguridad](#backup).
- [Seguridad](#security), incluye las diferencias en [Auditoría](#auditing), [Certificados](#certificates), [Credenciales](#credential), [Proveedores de servicios criptográficos](#cryptographic-providers), [Inicios de sesión y usuarios](#logins-and-users), [Clave maestra de servicio y clave de servicio](#service-key-and-service-master-key).
- [Configuración](#configuration), incluye las diferencias en [Extensión del grupo de búferes](#buffer-pool-extension), [Intercalación](#collation), [Niveles de compatibilidad](#compatibility-levels),[Creación de reflejo de la base de datos ](#database-mirroring), [Opciones de base de datos](#database-options), [Agente SQL Server](#sql-server-agent) y [Opciones de tabla](#tables).
- [Funcionalidades](#functionalities), incluidas [BULK INSERT/OPENROWSET](#bulk-insert--openrowset), [CLR](#clr), [DBCC](#dbcc), [Transacciones distribuidas](#distributed-transactions), [Eventos extendidos](#extended-events), [Bibliotecas externas](#external-libraries), [Filestream y Filetable](#filestream-and-filetable), [Búsqueda semántica de texto completo](#full-text-semantic-search), [Servidores vinculados](#linked-servers), [PolyBase](#polybase), [Replicación](#replication), [RESTORE](#restore-statement), [Service Broker](#service-broker), [Funciones, procedimientos almacenados y desencadenadores](#stored-procedures-functions-and-triggers).
- [Configuración del entorno](#Environment), como las configuraciones de redes virtuales y subredes.

La mayoría de estas características son restricciones de arquitectura y representan características de servicio.

En la [página de notas de la versión](../database/doc-changes-updates-release-notes.md) se explican los problemas temporales conocidos que se han detectado en Instancia administrada de SQL y que se resolverán en el futuro.

## <a name="availability"></a>Disponibilidad

### <a name="always-on-availability-groups"></a><a name="always-on-availability-groups"></a>Grupos de disponibilidad AlwaysOn

La [alta disponibilidad](../database/high-availability-sla.md) está integrada en la Instancia administrada de SQL y no la pueden controlar los usuarios. No se admiten las siguientes instrucciones:

- [CREATE ENDPOINT … FOR DATABASE_MIRRORING](/sql/t-sql/statements/create-endpoint-transact-sql)
- [CREATE AVAILABILITY GROUP](/sql/t-sql/statements/create-availability-group-transact-sql)
- [ALTER AVAILABILITY GROUP](/sql/t-sql/statements/alter-availability-group-transact-sql)
- [DROP AVAILABILITY GROUP](/sql/t-sql/statements/drop-availability-group-transact-sql)
- La cláusula [SET HADR](/sql/t-sql/statements/alter-database-transact-sql-set-hadr) de la instrucción [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql)

### <a name="backup"></a>Copia de seguridad

La Instancia administrada de SQL tiene copias de seguridad automáticas, por lo que los usuarios pueden crear copias de seguridad `COPY_ONLY` de bases de datos completas. No se admiten copias de seguridad de instantáneas de archivos, de registro ni diferenciales.

- Con una Instancia administrada de SQL, puede hacer una copia de seguridad de una base de datos de instancia solo en una cuenta de Azure Blob Storage:
  - Solo se admite `BACKUP TO URL`.
  - No se admiten `FILE`, `TAPE` ni dispositivos de copia de seguridad.
- Se admite la mayoría de las opciones de `WITH` generales.
  - `COPY_ONLY` es obligatorio.
  - `FILE_SNAPSHOT` no se admite.
  - No se admiten las opciones de cinta `REWIND`, `NOREWIND`, `UNLOAD` y `NOUNLOAD`.
  - No se admiten las opciones específicas de registro `NORECOVERY`, `STANDBY` y `NO_TRUNCATE`.

Limitaciones: 

- Con una Instancia administrada de SQL, puede hacer una copia de seguridad de una base de datos de instancia en una copia de seguridad con hasta 32 franjas, lo cual es suficiente para bases de datos de hasta 4 TB si se usa la compresión de copia de seguridad.
- No se puede ejecutar `BACKUP DATABASE ... WITH COPY_ONLY` en una base de datos cifrada con Cifrado de datos transparente (TDE) administrado por el servicio. TDE administrado por un servicio hace que las copias de seguridad se cifren con una clave interna de TDE. No es posible exportar la clave, por lo que no se puede restaurar la copia de seguridad. Use copias de seguridad automáticas y restauración a un momento dado, o use [Cifrado de datos transparente administrado por el cliente (BYOK)](../database/transparent-data-encryption-tde-overview.md#customer-managed-transparent-data-encryption---bring-your-own-key) en su lugar. También puede deshabilitar el cifrado en la base de datos.
- El tamaño máximo de una franja de copia de seguridad con el uso del comando `BACKUP` en una Instancia administrada de SQL es de 195 GB, lo cual es el tamaño máximo del blob. Aumente el número de franjas en el comando de copia de seguridad para reducir el tamaño de las franjas y permanecer dentro de este límite.

    > [!TIP]
    > Para solucionar esta limitación, cuando realice una copia de seguridad de una base de datos desde un servidor de SQL Server en un entorno local o en una máquina virtual, puede:
    >
    > - Realizar la copia de seguridad en `DISK` en lugar de hacerlo en `URL`.
    > - Cargar los archivos de copia de seguridad en Blob Storage.
    > - Restaurar en la Instancia administrada de SQL.
    >
    > El comando `Restore` de una Instancia administrada de SQL admite tamaños de blob más grandes en los archivos de copia de seguridad porque se usa un tipo de blob distinto para el almacenamiento de los archivos de copia de seguridad cargados.

Para más información acerca de las copias de seguridad mediante T-SQL, consulte [BACKUP](/sql/t-sql/statements/backup-transact-sql).

## <a name="security"></a>Seguridad

### <a name="auditing"></a>Auditoría

Las principales diferencias entre la auditoría en Microsoft Azure SQL y en SQL Server son las siguientes:

- Con Instancia administrada de SQL, la auditoría funciona en el nivel de servidor. Los archivos de registro `.xel` se almacenan en Azure Blob Storage.
- En Azure SQL Database, la auditoría funciona en el nivel de base de datos. Los archivos de registro `.xel` se almacenan en Azure Blob Storage.
- Con servidores de SQL Server locales o en los de máquinas virtuales, la auditoría funciona en el nivel de servidor. Los eventos se almacenan en el sistema de archivos o en los registros de eventos de Windows.
 
En Instancia administrada de SQL, la auditoría de XEvent admite Azure Blob Storage como destino. No se admiten archivos ni registros de Windows.

Las principales diferencias en la sintaxis de `CREATE AUDIT` para la auditoría en Azure Blob Storage son:

- Se proporciona una nueva sintaxis de `TO URL` que puede usar para especificar la dirección URL del contenedor de Azure Blob Storage donde se colocarán los archivos `.xel`.
- La sintaxis `TO FILE` no se admite porque la Instancia administrada de SQL no puede acceder a los recursos compartidos de archivos de Windows.

Para más información, consulte: 

- [CREATE SERVER AUDIT](/sql/t-sql/statements/create-server-audit-transact-sql) 
- [ALTER SERVER AUDIT](/sql/t-sql/statements/alter-server-audit-transact-sql)
- [Auditoría](/sql/relational-databases/security/auditing/sql-server-audit-database-engine)

### <a name="certificates"></a>Certificados

Una Instancia administrada de SQL no puede acceder a los recursos compartidos de archivos ni a las carpetas de Windows, por lo que se aplican las siguientes restricciones:

- El archivo `CREATE FROM`/`BACKUP TO` no se admite para certificados.
- No se admite el certificado `CREATE`/`BACKUP` de `FILE`/`ASSEMBLY`. No se pueden usar archivos de clave privada. 

Consulte [CREATE CERTIFICATE](/sql/t-sql/statements/create-certificate-transact-sql) y [BACKUP CERTIFICATE](/sql/t-sql/statements/backup-certificate-transact-sql). 
 
**Solución alternativa**: En lugar de crear una copia de seguridad del certificado y restaurar la copia de seguridad, [obtenga el contenido binario del certificado y la clave privada, almacénelo como archivo .sql y cree a partir del binario](/sql/t-sql/functions/certencoded-transact-sql#b-copying-a-certificate-to-another-database):

```sql
CREATE CERTIFICATE  
   FROM BINARY = asn_encoded_certificate
WITH PRIVATE KEY (<private_key_options>)
```

### <a name="credential"></a>Credential:

Solo se admiten identidades de Azure Key Vault y `SHARED ACCESS SIGNATURE`. No se admiten usuarios de Windows.

Consulte [CREATE CREDENTIAL](/sql/t-sql/statements/create-credential-transact-sql) y [ALTER CREDENTIAL](/sql/t-sql/statements/alter-credential-transact-sql).

### <a name="cryptographic-providers"></a>Proveedores de servicios criptográficos

Una Instancia administrada de SQL no puede acceder a archivos, por lo que no se pueden crear proveedores de servicios criptográficos:

- `CREATE CRYPTOGRAPHIC PROVIDER` no se admite. Consulte [CREATE CRYPTOGRAPHIC PROVIDER](/sql/t-sql/statements/create-cryptographic-provider-transact-sql).
- `ALTER CRYPTOGRAPHIC PROVIDER` no se admite. Consulte [ALTER CRYPTOGRAPHIC PROVIDER](/sql/t-sql/statements/alter-cryptographic-provider-transact-sql).

### <a name="logins-and-users"></a>Inicios de sesión y usuarios

- Se admiten los inicios de sesión de SQL creados con `FROM CERTIFICATE`, `FROM ASYMMETRIC KEY` y `FROM SID`. Consulte [CREATE LOGIN](/sql/t-sql/statements/create-login-transact-sql).
- Se admiten las entidades de seguridad (inicios de sesión) del servidor de Azure Active Directory (Azure AD) creadas con la sintaxis [CREATE LOGIN](/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current) o la sintaxis [CREATE USER FROM LOGIN [Azure AD Login]](/sql/t-sql/statements/create-user-transact-sql?view=azuresqldb-mi-current). Estos inicios de sesión se crean en el nivel de servidor.

    La Instancia administrada de SQL admite las entidades de seguridad de la base de datos de Azure AD con la sintaxis `CREATE USER [AADUser/AAD group] FROM EXTERNAL PROVIDER`. Esta característica también se conoce como usuarios de bases de datos independientes de Azure AD.

- No se admiten los inicios de sesión de Windows creados con la sintaxis `CREATE LOGIN ... FROM WINDOWS`. Use los usuarios e inicios de sesión de Azure Active Directory.
- El usuario de Azure AD que creó la instancia tiene [privilegios de administrador sin restricciones](../database/logins-create-manage.md).
- Los usuarios de nivel de base de datos de Azure AD que no son administradores pueden crearse con la sintaxis `CREATE USER ... FROM EXTERNAL PROVIDER`. Consulte [CREATE USER ... FROM EXTERNAL PROVIDER](../database/authentication-aad-configure.md#create-contained-users-mapped-to-azure-ad-identities).
- Las entidades de seguridad (inicios de sesión) del servidor de Azure AD admiten las características SQL en una única Instancia administrada de SQL. No se admiten las características que requieren una interacción entre instancias, con independencia de que se encuentren en el mismo inquilino de Azure AD o en inquilinos diferentes, para los usuarios de Azure AD. Ejemplos de estas características son los siguientes:

  - Replicación transaccional de SQL.
  - Servidor de vínculos.

- No se admite el establecimiento de un inicio de sesión de Azure AD asignado a un grupo de Azure AD como propietario de la base de datos.
- Se admite la suplantación de las entidades de seguridad en el nivel de servidor de Azure AD mediante otras entidades de seguridad de Azure AD, como la cláusula [EXECUTE AS](/sql/t-sql/statements/execute-as-transact-sql). Las limitaciones de EXECUTE AS son:

  - No se admite la cláusula EXECUTE AS USER para usuarios de Azure AD cuando el nombre es diferente del nombre de inicio de sesión. Por ejemplo, cuando el usuario se crea mediante la sintaxis CREATE USER [myAadUser] FROM LOGIN [john@contoso.com] y se intenta suplantar la identidad mediante EXEC AS USER = _myAadUser_. Cuando cree un usuario en **USER** a partir de una entidad de seguridad (inicio de sesión) de un servidor de Azure AD, especifique un valor de user_name igual que el valor de login_name que se obtiene de **LOGIN**.
  - Solo las entidades de seguridad (inicios de sesión) a nivel de servidor SQL que forman parte del rol `sysadmin` pueden ejecutar las siguientes operaciones dirigidas a las entidades de seguridad de Azure AD:

    - EXECUTE AS USER
    - EXECUTE AS LOGIN

- Se admite la exportación e importación de bases de datos con archivos bacpac para usuarios de Azure AD en una Instancia administrada de SQL mediante [SSMS V18.4 o una versión posterior](/sql/ssms/download-sql-server-management-studio-ssms) o [SQLPackage.exe](/sql/tools/sqlpackage-download).
  - Se admiten las siguientes configuraciones mediante el uso del archivo bacpac de la base de datos: 
    - Exportar o importar una base de datos entre diferentes instancias administradas en el mismo dominio de Azure AD.
    - Exportar una base de datos desde una Instancia administrada de SQL e importarla a SQL Database en el mismo dominio de Azure AD. 
    - Exportar una base de datos desde SQL Database e importarla a una Instancia administrada de SQL en el mismo dominio de Azure AD.
    - Exportar una base de datos desde una Instancia administrada de SQL e importarla a SQL Server (versión 2012 o posterior).
      - En esta configuración, todos los usuarios de Azure AD se crean como entidades de seguridad de base de datos de SQL Server (usuarios) sin inicios de sesión. El tipo de usuario se muestra como `SQL` y es visible como `SQL_USER` en sys.database_principals). Sus permisos y roles se conservan en los metadatos de la base de datos de SQL Server y se pueden usar para la suplantación. Sin embargo, no se pueden usar para obtener acceso e iniciar sesión en SQL Server con sus credenciales.

- Solo el inicio de sesión de la entidad de seguridad a nivel de servidor (el que crea el proceso de aprovisionamiento de la Instancia administrada de SQL), los miembros de los roles de servidor, como `securityadmin` o `sysadmin`, u otros inicios de sesión con permiso ALTER ANY LOGIN en el nivel de servidor pueden crear entidades de seguridad (inicios de sesión) de servidor de Azure AD en la base de datos maestra para la Instancia administrada de SQL.
- Si el inicio de sesión es una entidad de seguridad de SQL, solo los inicios de sesión que forman parte del rol `sysadmin` pueden utilizar el comando create para crear inicios de sesión para una cuenta de Azure AD.
- El inicio de sesión de Azure AD debe ser miembro de una instancia de Azure AD dentro del mismo directorio que se usa para la Instancia administrada de Azure SQL.
- Las entidades de seguridad (inicio de sesión) del servidor de Azure AD son visibles en el explorador de objetos a partir de la versión preliminar 5 de SQL Server Management Studio 18.0.
- Se permite la superposición de las entidades de seguridad (inicios de sesión) del servidor de Azure AD con una cuenta de administrador de Azure AD. Las entidades de seguridad (inicios de sesión) del servidor de Azure AD tienen prioridad sobre el administrador de Azure AD cuando se resuelve la entidad de seguridad y se aplican permisos a la Instancia administrada de SQL.
- Durante la autenticación, se aplica la siguiente secuencia para resolver la entidad de seguridad de autenticación:

    1. Si la cuenta de Azure AD existe como directamente asignada a la entidad de seguridad (inicio de sesión) del servidor de Azure AD que está presente en sys.server_principals como tipo "E", conceda acceso y aplique los permisos de la entidad de seguridad (inicio de sesión) del servidor de Azure AD.
    2. Si la cuenta de Azure AD es un miembro de un grupo de Azure AD que está asignado a la entidad de seguridad (inicio de sesión) del servidor de Azure AD (presente en sys.server_principals como tipo "X"), conceda acceso y aplique los permisos del inicio de sesión del grupo de Azure AD.
    3. Si la cuenta de Azure AD es un administrador de Azure AD configurado en un portal especial para la Instancia administrada de SQL, que no existe en las vistas de sistema de la Instancia administrada de SQL, aplique permisos fijos especiales del administrador de Azure AD para la Instancia administrada de SQL (modo heredado).
    4. Si la cuenta de Azure AD existe como directamente asignada a un usuario de Azure AD de una base de datos, presente en sys.database_principals como tipo "E", conceda acceso y aplique los permisos del usuario de la base de datos de Azure AD.
    5. Si la cuenta de Azure AD es un miembro de un grupo de Azure AD que está asignado a un usuario de Azure AD de una base de datos, presente en sys.database_principals como tipo "X", conceda acceso y aplique los permisos del inicio de sesión del grupo de Azure AD.
    6. Si hay un inicio de sesión de Azure AD asignado a una cuenta de usuario de Azure AD o a una cuenta de grupo de Azure AD, la cual resuelve la autenticación del usuario, se aplicarán todos los permisos de este inicio de sesión de Azure AD.

### <a name="service-key-and-service-master-key"></a>Clave maestra de servicio y clave de servicio

- No se admiten las [copias de seguridad de la clave maestra](/sql/t-sql/statements/backup-master-key-transact-sql) (administrada por el servicio de SQL Database).
- No se admite la [restauración de la clave maestra](/sql/t-sql/statements/restore-master-key-transact-sql) (administrada por el servicio de SQL Database).
- No se admiten las [copias de seguridad de la clave maestra del servicio](/sql/t-sql/statements/backup-service-master-key-transact-sql) (administrada por el servicio de SQL Database).
- No se admite la [restauración de la clave maestra del servicio](/sql/t-sql/statements/restore-service-master-key-transact-sql) (administrada por el servicio de SQL Database).

## <a name="configuration"></a>Configuración

### <a name="buffer-pool-extension"></a>Extensión del grupo de búferes

- No se admite la [extensión del grupo de búferes](/sql/database-engine/configure-windows/buffer-pool-extension).
- `ALTER SERVER CONFIGURATION SET BUFFER POOL EXTENSION` no se admite. Consulte [ALTER SERVER CONFIGURATION](/sql/t-sql/statements/alter-server-configuration-transact-sql).

### <a name="collation"></a>Intercalación

La intercalación de la instancia predeterminada es `SQL_Latin1_General_CP1_CI_AS` y puede especificarse como un parámetro de creación. Consulte [Intercalaciones](/sql/t-sql/statements/collations).

### <a name="compatibility-levels"></a>Niveles de compatibilidad

- Los niveles de compatibilidad admitidos son 100, 110, 120, 130, 140 y 150.
- No se admiten los niveles de compatibilidad menores que 100.
- El nivel de compatibilidad predeterminado es 140 para las bases de datos nuevas. Para las bases de datos restauradas, el nivel de compatibilidad no cambiará si era 100 o superior.

Consulte [Nivel de compatibilidad de ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql-compatibility-level).

### <a name="database-mirroring"></a>Creación de reflejo de la base de datos

No se admite la creación de reflejo de la base de datos.

- Las opciones `ALTER DATABASE SET PARTNER` y `SET WITNESS` no se admiten.
- `CREATE ENDPOINT … FOR DATABASE_MIRRORING` no se admite.

Para más información, consulte [ALTER DATABASE SET PARTNER y SET WITNESS](/sql/t-sql/statements/alter-database-transact-sql-database-mirroring) y [CREATE ENDPOINT … FOR DATABASE_MIRRORING](/sql/t-sql/statements/create-endpoint-transact-sql).

### <a name="database-options"></a>Opciones de base de datos

- No se permite usar varios archivos de registro.
- No se admiten objetos en memoria caché en el nivel de servicio de uso general. 
- Hay un límite de 280 archivos por instancia de uso general, lo cual implica un máximo de 280 archivos por base de datos. Los datos y archivos de registro en el nivel de uso general cuentan para este límite. [El nivel Crítico para la empresa admite 32 767 archivos por base de datos](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-resource-limits#service-tier-characteristics).
- La base de datos no puede contener grupos de archivos que contengan datos de secuencia de archivos. Se producirá un error en la restauración si el archivo .bak contiene datos `FILESTREAM`. 
- Todos los archivos se colocan en Azure Blob Storage. La E/S y el rendimiento por archivo dependen del tamaño de cada archivo individual.

#### <a name="create-database-statement"></a>Instrucción CREATE DATABASE

Se aplican las siguientes limitaciones a `CREATE DATABASE`:

- No se pueden definir archivos y grupos de archivos. 
- La opción `CONTAINMENT` no se admite. 
- Las opciones `WITH` no se admiten. 
   > [!TIP]
   > Como alternativa, use `ALTER DATABASE` después de `CREATE DATABASE` para establecer las opciones de la base de datos para agregar archivos o establecer el contenedor. 

- La opción `FOR ATTACH` no se admite.
- La opción `AS SNAPSHOT OF` no se admite.

Para más información, consulte [CREATE DATABASE](/sql/t-sql/statements/create-database-sql-server-transact-sql).

#### <a name="alter-database-statement"></a>instrucción ALTER DATABASE

Algunas propiedades de archivo no se pueden establecer ni cambiar:

- No se puede especificar una ruta de acceso del archivo en la instrucción T-SQL `ALTER DATABASE ADD FILE (FILENAME='path')`. Quite `FILENAME` del script porque una Instancia administrada de SQL coloca automáticamente los archivos. 
- El nombre del archivo no se puede cambiar mediante la instrucción `ALTER DATABASE`.

Las siguientes opciones se establecen de forma predeterminada y no se pueden cambiar:

- `MULTI_USER`
- `ENABLE_BROKER ON`
- `AUTO_CLOSE OFF`

Las opciones siguientes no se pueden modificar:

- `AUTO_CLOSE`
- `AUTOMATIC_TUNING(CREATE_INDEX=ON|OFF)`
- `AUTOMATIC_TUNING(DROP_INDEX=ON|OFF)`
- `DISABLE_BROKER`
- `EMERGENCY`
- `ENABLE_BROKER`
- `FILESTREAM`
- `HADR`
- `NEW_BROKER`
- `OFFLINE`
- `PAGE_VERIFY`
- `PARTNER`
- `READ_ONLY`
- `RECOVERY BULK_LOGGED`
- `RECOVERY_SIMPLE`
- `REMOTE_DATA_ARCHIVE` 
- `RESTRICTED_USER`
- `SINGLE_USER`
- `WITNESS`

Para más información, consulte [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql-file-and-filegroup-options).

### <a name="sql-server-agent"></a>Agente SQL Server

- La habilitación o deshabilitación del Agente SQL Server no se admite actualmente en la Instancia administrada de SQL. El Agente SQL se ejecuta de forma continua.
- La configuración del Agente SQL Server es de solo lectura. El procedimiento `sp_set_agent_properties` no se admite en la Instancia administrada de SQL. 
- Trabajos
  - Se admiten los pasos de trabajo de T-SQL.
  - Se admiten los siguientes trabajos de replicación:
    - Lector del registro de transacciones
    - Instantánea
    - Distribuidor.
  - Se admiten los pasos de trabajo de SSIS.
  - Actualmente no se admiten estos otros tipos de pasos de trabajo:
    - No se admite el paso de trabajo de replicación de mezcla. 
    - No se admite el lector de colas. 
    - Aún no se admite el shell de comandos.
  - La Instancia administrada de SQL no puede acceder a los recursos externos como, por ejemplo, a los recursos compartidos de red a través de robocopy. 
  - No se admite SQL Server Analysis Services.
- Las notificaciones se admiten parcialmente.
- Se admite la notificación por correo electrónico, aunque es necesario configurar un perfil de Correo electrónico de base de datos. Agente SQL Server solo puede usar un perfil de Correo electrónico de base de datos y se debe llamar `AzureManagedInstance_dbmail_profile`. 
  - El buscapersonas no se admite.
  - No se admite NetSend.
  - Aún no se admiten las alertas.
  - No se admiten los servidores proxy.
- No se admite EventLog.

Actualmente, no se admiten las siguientes características del Agente SQL:

- Servidores proxy
- Programación de trabajos en una CPU inactiva
- Habilitar o deshabilitar un agente
- Alertas

Para más información acerca del Agente SQL Server, consulte [Agente SQL Server](/sql/ssms/agent/sql-server-agent).

### <a name="tables"></a>Tablas

No se admiten los siguientes tipos de tablas:

- [FILESTREAM](/sql/relational-databases/blob/filestream-sql-server)
- [FILETABLE](/sql/relational-databases/blob/filetables-sql-server)
- [EXTERNAL TABLE](/sql/t-sql/statements/create-external-table-transact-sql) (Polybase)
- [MEMORY_OPTIMIZED](/sql/relational-databases/in-memory-oltp/introduction-to-memory-optimized-tables) (únicamente no se admite en el nivel Uso general)

Para más información sobre cómo crear y modificar tablas, consulte [CREATE TABLE](/sql/t-sql/statements/create-table-transact-sql) y [ALTER TABLE](/sql/t-sql/statements/alter-table-transact-sql).

## <a name="functionalities"></a>Funcionalidades

### <a name="bulk-insert--openrowset"></a>BULK INSERT/OPENROWSET

Una Instancia administrada de SQL no puede acceder a los recursos compartidos de archivos ni carpetas de Windows, por lo que los archivos se deben importar desde Azure Blob Storage:

- `DATASOURCE` es necesario en el comando `BULK INSERT` durante la importación de archivos desde Azure Blob Storage. Consulte [BULK INSERT](/sql/t-sql/statements/bulk-insert-transact-sql).
- `DATASOURCE` se necesita en la función `OPENROWSET` cuando se lee el contenido de un archivo desde Azure Blob Storage. Consulte [OPENROWSET](/sql/t-sql/functions/openrowset-transact-sql).
- `OPENROWSET` se puede usar para leer datos de Azure SQL Database, Instancia administrada de Azure SQL o instancias de SQL Server. No se admiten otros orígenes, como bases de datos de Oracle o archivos de Excel.

### <a name="clr"></a>CLR

Una Instancia administrada de SQL no puede acceder a los recursos compartidos de archivos ni a las carpetas de Windows, por lo que se aplican las siguientes restricciones:

- Solo se admite `CREATE ASSEMBLY FROM BINARY`. Consulte [CREATE ASSEMBLY FROM BINARY](/sql/t-sql/statements/create-assembly-transact-sql). 
- `CREATE ASSEMBLY FROM FILE` no se admite. Consulte [CREATE ASSEMBLY FROM FILE](/sql/t-sql/statements/create-assembly-transact-sql).
- `ALTER ASSEMBLY` no puede hacer referencia a archivos. Consulte [ALTER ASSEMBLY](/sql/t-sql/statements/alter-assembly-transact-sql).

### <a name="database-mail-db_mail"></a>Correo electrónico de base de datos: (db_mail)
 - `sp_send_dbmail` no puede enviar datos adjuntos con el parámetro @file_attachments. El sistema de archivos local y los recursos compartidos externos, o Azure Blob Storage, no son accesibles con este procedimiento.
 - Vea los problemas conocidos relacionados con el parámetro `@query` y la autenticación.
 
### <a name="dbcc"></a>DBCC

La Instancia administrada de SQL no admite instrucciones DBCC no documentadas que estén habilitadas en SQL Server.

- Solo se admite un número limitado de marcas de seguimiento globales. No se admiten las `Trace flags` en el nivel de sesión. Consulte [Marcas de seguimiento](/sql/t-sql/database-console-commands/dbcc-traceon-trace-flags-transact-sql).
- [DBCC TRACEOFF](/sql/t-sql/database-console-commands/dbcc-traceoff-transact-sql) y [DBCC TRACEON](/sql/t-sql/database-console-commands/dbcc-traceon-transact-sql) funcionan con el número limitado de marcas trace-flags globales.
- No se puede usar [DBCC CHECKDB](/sql/t-sql/database-console-commands/dbcc-checkdb-transact-sql) con las opciones REPAIR_ALLOW_DATA_LOSS, REPAIR_FAST y REPAIR_REBUILD porque la base de datos no se puede establecer en el modo `SINGLE_USER`. Consulte las [diferencias de ALTER DATABASE](#alter-database-statement). El equipo de soporte técnico de Azure se encarga de los posibles daños en la base de datos. Póngase en contacto con el soporte técnico de Azure si hay algún signo de daños en la base de datos.

### <a name="distributed-transactions"></a>Distributed transactions

Actualmente no se admite MSDTC ni las [transacciones elásticas](../database/elastic-transactions-overview.md) en la Instancia administrada de SQL.

### <a name="extended-events"></a>Eventos extendidos

No se admiten algunos destinos específicos de Windows para Extended Events (XEvents):

- No se admite el destino `etw_classic_sync`. Almacene los archivos `.xel` en Azure Blob Storage. Consulte [Destino etw_classic_sync](/sql/relational-databases/extended-events/targets-for-extended-events-in-sql-server#etw_classic_sync_target-target).
- No se admite el destino `event_file`. Almacene los archivos `.xel` en Azure Blob Storage. Consulte [Destino event_file](/sql/relational-databases/extended-events/targets-for-extended-events-in-sql-server#event_file-target).

### <a name="external-libraries"></a>Bibliotecas externas

Las bibliotecas externas de R y Python en base de datos se admiten en la versión preliminar pública limitada. Consulte [Machine Learning Services de Azure SQL Managed Instance (versión preliminar)](machine-learning-services-overview.md).

### <a name="filestream-and-filetable"></a>FileStream y FileTable

- No se admiten datos de secuencia de archivos.
- La base de datos no puede contener grupos de archivos con datos `FILESTREAM`.
- `FILETABLE` no se admite.
- Las tablas no pueden tener tipos `FILESTREAM`.
- No se admiten las siguientes funciones:
  - `GetPathLocator()`
  - `GET_FILESTREAM_TRANSACTION_CONTEXT()`
  - `PathName()`
  - `GetFileNamespacePat)`
  - `FileTableRootPath()`

Para más información, consulte [FILESTREAM](/sql/relational-databases/blob/filestream-sql-server) y [FileTables](/sql/relational-databases/blob/filetables-sql-server).

### <a name="full-text-semantic-search"></a>Búsqueda semántica de texto completo

No se admite la [búsqueda semántica](/sql/relational-databases/search/semantic-search-sql-server).

### <a name="linked-servers"></a>Servidores vinculados

Los servidores vinculados en la Instancia administrada de SQL admiten un número limitado de destinos:

- Los destinos admitidos son Instancia administrada de SQL, SQL Database, Azure Synapse SQL e instancias de SQL Server. 
- Los servidores vinculados no admiten transacciones de escritura distribuidas (MS DTC).
- Destinos no admitidos: archivos, Analysis Services y otros RDBMS. Intente usar la importación de CSV nativa desde Azure Blob Storage mediante `BULK INSERT` o `OPENROWSET` como alternativa para la importación de archivos.

Operaciones: 

- No se admiten las transacciones de escritura entre instancias.
- Se admite `sp_dropserver` para quitar un servidor vinculado. Consulte [sp_dropserver](/sql/relational-databases/system-stored-procedures/sp-dropserver-transact-sql).
- La función `OPENROWSET` se puede usar para ejecutar consultas solo en instancias de SQL Server. Se pueden administrar de forma local o en máquinas virtuales. Consulte [OPENROWSET](/sql/t-sql/functions/openrowset-transact-sql).
- La función `OPENDATASOURCE` se puede usar para ejecutar consultas solo en instancias de SQL Server. Se pueden administrar de forma local o en máquinas virtuales. Solo se admiten los valores `SQLNCLI`, `SQLNCLI11` y `SQLOLEDB` como proveedor. Un ejemplo es `SELECT * FROM OPENDATASOURCE('SQLNCLI', '...').AdventureWorks2012.HumanResources.Employee`. Consulte [OPENDATASOURCE](/sql/t-sql/functions/opendatasource-transact-sql).
- Los servidores vinculados no se pueden usar para leer archivos (Excel y CSV) de los recursos compartidos de red. Intente usar [BULK INSERT](/sql/t-sql/statements/bulk-insert-transact-sql#e-importing-data-from-a-csv-file) o [OPENROWSET](/sql/t-sql/functions/openrowset-transact-sql#g-accessing-data-from-a-csv-file-with-a-format-file) que lee archivos CSV desde Azure Blob Storage. Realice un seguimiento de estas solicitudes en el [elemento de comentarios de la Instancia administrada de SQL](https://feedback.azure.com/forums/915676-sql-managed-instance/suggestions/35657887-linked-server-to-non-sql-sources)|.

### <a name="polybase"></a>PolyBase

No se admiten tablas externas que hacen referencia a archivos en HDFS o Azure Blob Storage. Para más información acerca de Polybase, consulte [Polybase](/sql/relational-databases/polybase/polybase-guide).

### <a name="replication"></a>Replicación

- Se admiten los tipos de replicación de instantánea y bidireccional. No se admite la replicación de mezcla, la replicación punto a punto y las suscripciones actualizables.
- La [replicación transaccional](replication-transactional-overview.md) está disponible para la versión preliminar pública en la Instancia administrada de SQL con algunas restricciones:
    - Todos los tipos de participantes de la replicación (editor, distribuidor, suscriptor de extracción y suscriptor de inserción) se pueden colocar en una Instancia administrada de SQL, pero el editor y el distribuidor deben estar tanto en la nube como en el entorno local.
    - La Instancia administrada de SQL puede comunicarse con las versiones recientes de SQL Server. Para obtener más información, consulte la [matriz de versiones compatibles](replication-transactional-overview.md#supportability-matrix).
    - La replicación transaccional tiene algunos [requisitos de red adicionales](replication-transactional-overview.md#requirements).

Para obtener más información sobre la configuración de la replicación transaccional, vea los siguientes tutoriales:
- [Replicación entre un editor de Instancia administrada de SQL y un suscriptor de Instancia administrada de SQL](replication-between-two-instances-configure-tutorial.md)
- [Replicación entre un editor de Instancia administrada de SQL, un distribuidor de Instancia administrada de SQL y un suscriptor de SQL Server](replication-two-instances-and-sql-server-configure-tutorial.md)

### <a name="restore-statement"></a>Instrucción RESTORE 

- Sintaxis admitida:
  - `RESTORE DATABASE`
  - `RESTORE FILELISTONLY ONLY`
  - `RESTORE HEADER ONLY`
  - `RESTORE LABELONLY ONLY`
  - `RESTORE VERIFYONLY ONLY`
- Sintaxis no admitida:
  - `RESTORE LOG ONLY`
  - `RESTORE REWINDONLY ONLY`
- Origen: 
  - La única opción admitida es `FROM URL` (Azure Blob Storage).
  - No se admite `FROM DISK`/`TAPE`/dispositivo de copia de seguridad.
  - No se admiten los conjuntos de copia de seguridad.
- Las opciones `WITH` no se admiten. Los intentos de restauración que incluyan `WITH`, como `DIFFERENTIAL`, `STATS`, `REPLACE`, etc., generarán errores.
- `ASYNC RESTORE`: la restauración continúa aunque se interrumpa la conexión con el cliente. Si la conexión se interrumpe, puede usar `sys.dm_operation_status` para ver el estado de una operación de restauración y para crear y eliminar una base de datos. Consulte [sys.dm_operation_status](/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database). 

Las siguientes opciones de base de datos se establecen o invalidan, y no se pueden cambiar más adelante: 

- `NEW_BROKER` si el agente no está habilitado en el archivo .bak. 
- `ENABLE_BROKER` si el agente no está habilitado en el archivo .bak. 
- `AUTO_CLOSE=OFF` si una base de datos en el archivo .bak tiene `AUTO_CLOSE=ON`. 
- `RECOVERY FULL` si una base de datos en el archivo .bak tiene los modos de recuperación `SIMPLE` o `BULK_LOGGED`.
- Se agrega un grupo de archivos optimizado para memoria y se le asigna el nombre XTP si aún no se encuentra en el archivo .bak de origen. 
- Se cambia el nombre de todos los grupos de archivos optimizados para memoria existentes a XTP. 
- Las opciones `SINGLE_USER` y `RESTRICTED_USER` se convierten en `MULTI_USER`.

Limitaciones: 

- Las copias de seguridad de las bases de datos dañadas pueden restaurarse en función del tipo de daños, pero no se realizarán copias de seguridad automatizadas mientras no se corrija el daño. Asegúrese de ejecutar `DBCC CHECKDB` en la Instancia administrada de SQL de origen y de usar la copia de seguridad `WITH CHECKSUM` para evitar este problema.
- La restauración de un archivo `.BAK` de una base de datos que contenga alguna de las limitaciones descritas en este documento (por ejemplo, objetos `FILESTREAM` o `FILETABLE`) no se puede llevar a cabo en la Instancia administrada de SQL.
- Los archivos `.BAK` que contienen varios conjuntos de copia de seguridad no se pueden restaurar. 
- Los archivos `.BAK` que contienen varios archivos de registro no se pueden restaurar.
- Las copias de seguridad que contienen bases de datos con un tamaño superior a 8 TB, objetos OLTP en memoria activos o una cantidad de archivos superior a 280 por instancia no se pueden restaurar en una instancia de Uso general. 
- Las copias de seguridad que contienen bases de datos con un tamaño superior a 4 TB u objetos OLTP en memoria con un tamaño total superior al descrito en los [límites de recursos](resource-limits.md) no se pueden restaurar en una instancia de tipo Crítico para la empresa.
Para más información acerca de las instrucciones Restore, consulte [Instrucciones RESTORE](/sql/t-sql/statements/restore-statements-transact-sql).

 > [!IMPORTANT]
 > Las mismas limitaciones se aplican a una operación de restauración a un momento dado integrada. Por ejemplo, no es posible restaurar una base de datos de Uso general de más de 4 TB en una instancia de tipo Crítico para la empresa. Una base de datos de tipo Crítico para la empresa con archivos OLTP en memoria o más de 280 archivos no se puede restaurar en la instancia de Uso general.

### <a name="service-broker"></a>Service Broker

No se admite el agente de servicio entre instancias:

- `sys.routes`: Como requisito previo, debe seleccionar la dirección de sys.routes. La dirección debe ser LOCAL en todas las rutas. Consulte [sys.routes](/sql/relational-databases/system-catalog-views/sys-routes-transact-sql).
- `CREATE ROUTE`: no se puede usar `CREATE ROUTE` con un valor de `ADDRESS` distinto de `LOCAL`. Consulte [CREATE ROUTE](/sql/t-sql/statements/create-route-transact-sql).
- `ALTER ROUTE`: no se puede usar `ALTER ROUTE` con un valor de `ADDRESS` distinto de `LOCAL`. Consulte [ALTER ROUTE](/sql/t-sql/statements/alter-route-transact-sql). 

### <a name="stored-procedures-functions-and-triggers"></a>Procedimientos almacenados, funciones y desencadenadores

- `NATIVE_COMPILATION` no se admite en el nivel De uso general.
- No se admiten las siguientes opciones de [sp_configure](/sql/relational-databases/system-stored-procedures/sp-configure-transact-sql): 
  - `allow polybase export`
  - `allow updates`
  - `filestream_access_level`
  - `remote access`
  - `remote data archive`
  - `remote proc trans`
- `sp_execute_external_scripts` no se admite. Consulte [sp_execute_external_scripts](/sql/relational-databases/system-stored-procedures/sp-execute-external-script-transact-sql#examples).
- `xp_cmdshell` no se admite. Consulte [xp_cmdshell](/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql).
- No se admiten `Extended stored procedures`, lo cual incluye `sp_addextendedproc` y `sp_dropextendedproc`. Consulte [Procedimientos almacenados extendidos](/sql/relational-databases/system-stored-procedures/general-extended-stored-procedures-transact-sql).
- No se admiten `sp_attach_db`, `sp_attach_single_file_db`, y `sp_detach_db`. Consulte [sp_attach_db](/sql/relational-databases/system-stored-procedures/sp-attach-db-transact-sql), [sp_attach_single_file_db](/sql/relational-databases/system-stored-procedures/sp-attach-single-file-db-transact-sql), y [sp_detach_db](/sql/relational-databases/system-stored-procedures/sp-detach-db-transact-sql).

### <a name="system-functions-and-variables"></a>Variables y funciones del sistema

Las siguientes variables, funciones y vistas devuelven resultados diferentes:

- `SERVERPROPERTY('EngineEdition')` devuelve el valor 8. Esta propiedad identifica de forma única una Instancia administrada de SQL. Consulte [SERVERPROPERTY](/sql/t-sql/functions/serverproperty-transact-sql).
- `SERVERPROPERTY('InstanceName')` devuelve NULL, porque el concepto de instancia tal y como existe para SQL Server no se aplica a una Instancia administrada de SQL. Consulte [SERVERPROPERTY('InstanceName')](/sql/t-sql/functions/serverproperty-transact-sql).
- `@@SERVERNAME` devuelve el nombre DNS completo "conectable", por ejemplo, my-managed-instance.wcus17662feb9ce98.database.windows.net. Consulte [@@SERVERNAME](/sql/t-sql/functions/servername-transact-sql). 
- `SYS.SERVERS` devuelve un nombre DNS completo "conectable", por ejemplo, `myinstance.domain.database.windows.net`, para las propiedades "name" y "data_source". Consulte [SYS.SERVERS](/sql/relational-databases/system-catalog-views/sys-servers-transact-sql).
- `@@SERVICENAME` devuelve NULL, porque el concepto de servicio tal y como existe para SQL Server no se aplica a una Instancia administrada de SQL. Consulte [@@SERVICENAME](/sql/t-sql/functions/servicename-transact-sql).
- `SUSER_ID` es compatible. Devuelve NULL si el inicio de sesión de Azure AD no está en sys.syslogins. Consulte [SUSER_ID](/sql/t-sql/functions/suser-id-transact-sql). 
- `SUSER_SID` no se admite. Se devuelven los datos incorrectos, lo cual es un problema temporal conocido. Consulte [SUSER_SID](/sql/t-sql/functions/suser-sid-transact-sql). 

## <a name="environment-constraints"></a><a name="Environment"></a>Restricciones del entorno

### <a name="subnet"></a>Subnet
-  No se puede colocar ningún otro recurso (por ejemplo, máquinas virtuales) en la subred en la que ha implementado la Instancia administrada de SQL. Implemente estos recursos con una subred diferente.
- La subred debe tener un número de [direcciones IP](connectivity-architecture-overview.md#network-requirements) disponibles suficiente. El mínimo es 16, aunque se recomienda tener al menos 32 direcciones IP en la subred.
- [No se pueden asociar los puntos de conexión de servicio con la subred de la Instancia administrada de SQL](connectivity-architecture-overview.md#network-requirements). Asegúrese de que la opción de puntos de conexión de servicio esté deshabilitada al crear la red virtual.
- El número de núcleos virtuales y tipos de instancias que se pueden implementar en una región tiene algunas [restricciones y límites](resource-limits.md#regional-resource-limitations).
- Hay algunas [reglas de seguridad que se deben aplicar en la subred](connectivity-architecture-overview.md#network-requirements).

### <a name="vnet"></a>VNET
- La red virtual se puede implementar mediante el modelo con Resource Manager. No se admite el modelo clásico.
- Después de crear una Instancia administrada de SQL, no se admite el traslado de dicha Instancia administrada de SQL o la red virtual a otro grupo de recursos o suscripción.
- Algunos servicios, como App Service Environment, Logic Apps y la Instancia administrada de SQL (que se usan para la replicación geográfica, la replicación transaccional o a través de servidores vinculados), no pueden acceder a la Instancia administrada de SQL de diferentes regiones si sus redes virtuales están conectadas mediante [emparejamiento global](../../virtual-network/virtual-networks-faq.md#what-are-the-constraints-related-to-global-vnet-peering-and-load-balancers). Se puede conectar a estos recursos a través de ExpressRoute o de red virtual a red virtual a través de puertas de enlace de red virtuales.

### <a name="failover-groups"></a>Grupos de conmutación por error
Las bases de datos del sistema no se replican en la instancia secundaria de un grupo de conmutación por error. Por lo tanto, los escenarios que dependen de objetos de las bases de datos del sistema serán imposibles en la instancia secundaria, a menos que los objetos se creen manualmente en la secundaria.

### <a name="failover-groups"></a>Grupos de conmutación por error
Las bases de datos del sistema no se replican en la instancia secundaria de un grupo de conmutación por error. Por lo tanto, los escenarios que dependen de objetos de las bases de datos del sistema serán imposibles en la instancia secundaria, a menos que los objetos se creen manualmente en la secundaria.

### <a name="tempdb"></a>TEMPDB

El tamaño máximo del archivo `tempdb` no puede ser mayor de 24 GB por núcleo en un nivel De uso general. El tamaño máximo de `tempdb` en un nivel Crítico para la empresa está limitado por el tamaño de almacenamiento de la Instancia administrada de SQL. El tamaño del archivo de registro `Tempdb` está limitado a 120 GB en el nivel de uso general. Algunas consultas podrían devolver un error si necesitan más de 24 GB por núcleo en `tempdb` o si producen datos de registro superiores a 120 GB.

### <a name="msdb"></a>MSDB

Los siguientes esquemas MSDB de la Instancia administrada de SQL deben ser propiedad de sus respectivos roles predefinidos:

- Roles generales
  - TargetServersRole
- [Roles fijos de base de datos](https://docs.microsoft.com/sql/ssms/agent/sql-server-agent-fixed-database-roles?view=sql-server-ver15)
  - SQLAgentUserRole
  - SQLAgentReaderRole
  - SQLAgentOperatorRole
- [Roles de DatabaseMail](https://docs.microsoft.com/sql/relational-databases/database-mail/database-mail-configuration-objects?view=sql-server-ver15#DBProfile):
  - DatabaseMailUserRole
- [Roles de Integration Services](https://docs.microsoft.com/sql/integration-services/security/integration-services-roles-ssis-service?view=sql-server-ver15):
  - db_ssisadmin
  - db_ssisltduser
  - db_ssisoperator
  
> [!IMPORTANT]
> El cambio de los nombres de roles predefinidos, los nombres de esquema y los propietarios de esquemas por parte de los clientes afectará al funcionamiento normal del servicio. Los cambios que se realicen en estos se revertirán a los valores predefinidos en cuanto se detecten, o bien en la siguiente actualización del servicio en la última para garantizar el funcionamiento normal del servicio.

### <a name="error-logs"></a>Registros de error

Una Instancia administrada de SQL coloca información detallada en los registros de errores. Existen muchos eventos internos del sistema que se archivan en el registro de errores. use un procedimiento personalizado para leer los registros de errores que filtran algunas entradas que no son pertinentes. Para obtener más información, vea [Instancia administrada de SQL – sp_readmierrorlog](https://blogs.msdn.microsoft.com/sqlcat/2018/05/04/azure-sql-db-managed-instance-sp_readmierrorlog/) o [Extensión de Instancia administrada de SQL (versión preliminar)](/sql/azure-data-studio/azure-sql-managed-instance-extension#logs) para Azure Data Studio.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre Instancia administrada de SQL, vea [¿Qué es Instancia administrada de SQL?](sql-managed-instance-paas-overview.md)
- Para obtener una lista de características y ver una comparativa, consulte [Comparación de características de Instancia administrada de SQL](../database/features-comparison.md).
- Para obtener las actualizaciones de versión y el estado de los problemas conocidos, vea [Notas de la versión de Instancia administrada de SQL](../database/doc-changes-updates-release-notes.md).
- Para consultar un inicio rápido que muestra cómo crear una nueva Instancia administrada de SQL, consulte [Creación de una Instancia administrada de SQL](instance-create-quickstart.md).
