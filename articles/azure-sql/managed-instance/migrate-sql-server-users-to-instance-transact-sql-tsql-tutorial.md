---
title: Migración de los usuarios y los grupos de Windows de SQL Server a Instancia administrada de SQL mediante T-SQL
description: Obtenga información sobre cómo migrar los usuarios y los grupos de Windows de una instancia de SQL Server a Instancia administrada de Azure SQL
services: sql-database
ms.service: sql-managed-instance
ms.subservice: security
ms.custom: seo-lt-2019, sqldbrb=1
ms.topic: tutorial
author: GitHubMirek
ms.author: mireks
ms.reviewer: vanto
ms.date: 10/30/2019
ms.openlocfilehash: f2dd34ab7c6ee5be26836e4abb86960605ee44ee
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "84708692"
---
# <a name="tutorial-migrate-windows-users-and-groups-in-a-sql-server-instance-to-azure-sql-managed-instance-using-t-sql-ddl-syntax"></a>Tutorial: Migración de los usuarios y los grupos de Windows de una instancia de SQL Server a Instancia administrada de Azure SQL mediante la sintaxis DDL de T-SQL
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

> [!NOTE]
> La sintaxis que se usa para migrar los usuarios y los grupos Instancia administrada de SQL de este artículo se encuentra en la **versión preliminar pública**.

Este artículo le guiará por el proceso de migración de los usuarios y los grupos locales de Windows de SQL Server a Instancia administrada de Azure SQL mediante la sintaxis de T-SQL.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> - Crear inicios de sesión para SQL Server
> - Crear una base de datos de prueba para la migración
> - Crear inicios de sesión, usuarios y roles
> - Realizar una copia de seguridad y restauración de la base de datos a Instancia administrada de SQL (MI)
> - Migrar manualmente los usuarios a la instancia administrada mediante la sintaxis ALTER USER
> - Probar la autenticación con los nuevos usuarios asignados

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial son necesarios los siguientes requisitos previos:

- El dominio de Windows debe estar federado con Azure Active Directory (Azure AD).
- Acceso a Active Directory para crear usuarios y grupos.
- Una instancia de SQL Server ya existente en el entorno local.
- Una instancia administrada de SQL ya existente. Consulte [Quickstart: Creación de una Instancia administrada de SQL](instance-create-quickstart.md).
  - Para crear inicios de sesión de Azure AD, se debe usar `sysadmin` en la instancia administrada de SQL.
- [Cree un administrador de Azure AD para Instancia administrada de SQL](../database/authentication-aad-configure.md#provision-azure-ad-admin-sql-managed-instance).
- Puede conectarse a la instancia administrada de SQL dentro de la red. Para más información, consulte los artículos siguientes:
  - [Conexión de la aplicación a Instancia administrada de Azure SQL](connect-application-instance.md)
  - [Inicio rápido: Configuración de una conexión de punto a sitio a una instancia administrada de Azure SQL desde el entorno local](point-to-site-p2s-configure.md)
  - [Configuración de un punto de conexión público en Instancia administrada de Azure SQL](public-endpoint-configure.md)

## <a name="t-sql-ddl-syntax"></a>Sintaxis DDL de T-SQL

A continuación se muestra la sintaxis DDL de T-SQL que se usa para realizar la migración de usuarios y grupos de Windows desde una instancia de SQL Server a Instancia administrada de SQL con autenticación de Azure AD.

```sql
-- For individual Windows users with logins
ALTER USER [domainName\userName] WITH LOGIN = [loginName@domainName.com];

--For individual groups with logins
ALTER USER [domainName\groupName] WITH LOGIN=[groupName]
```

## <a name="arguments"></a>Argumentos

_domainName_</br>
Especifica el nombre de dominio del usuario.

_userName_</br>
Especifica el nombre del usuario identificado en la base de datos.

_= loginName\@domainName.com_</br>
Reasigna un usuario al inicio de sesión de Azure AD

_groupName_</br>
Especifica el nombre del grupo identificado en la base de datos.

## <a name="part-1-create-logins-in-sql-server-for-windows-users-and-groups"></a>Parte 1: Creación de inicios de sesión en SQL Server para usuarios y grupos de Windows

> [!IMPORTANT]
> La siguiente sintaxis crea un inicio de sesión de un usuario y un grupo en SQL Server. Deberá asegurarse de que el usuario y el grupo existen dentro de Active Directory antes de ejecutar la siguiente sintaxis. </br> </br>
> Usuarios: testUser1, testGroupUser </br>
> Grupo: migration - testGroupUser debe pertenecer al grupo de migración de AD

En el ejemplo siguiente se crea un inicio de sesión en SQL Server para una cuenta denominada _testUser1_ en el dominio _aadsqlmi_.

```sql
-- Sign into SQL Server as a sysadmin or a user that can create logins and databases

use master;  
go

-- Create Windows login
create login [aadsqlmi\testUser1] from windows;
go;

/** Create a Windows group login which contains one user [aadsqlmi\testGroupUser].
testGroupUser will need to be added to the migration group in Active Directory
**/
create login [aadsqlmi\migration] from windows;
go;


-- Check logins were created
select * from sys.server_principals;
go;
```

Cree una base de datos para esta prueba.

```sql
-- Create a database called [migration]
create database migration
go
```

## <a name="part-2-create-windows-users-and-groups-then-add-roles-and-permissions"></a>Parte 2: Creación de usuarios y grupos de Windows y, a continuación, incorporación de roles y permisos

Use la siguiente sintaxis para crear el usuario de prueba.

```sql
use migration;  
go

-- Create Windows user [aadsqlmi\testUser1] with login
create user [aadsqlmi\testUser1] from login [aadsqlmi\testUser1];
go
```

Compruebe los permisos de usuario:

```sql
-- Check the user in the Metadata
select * from sys.database_principals;
go

-- Display the permissions – should only have CONNECT permissions
select user_name(grantee_principal_id), * from sys.database_permissions;
go
```

Cree un rol y asigne a este rol el usuario de prueba:

```sql
-- Create a role with some permissions and assign the user to the role
create role UserMigrationRole;
go

grant CONNECT, SELECT, View DATABASE STATE, VIEW DEFINITION to UserMigrationRole;
go

alter role UserMigrationRole add member [aadsqlmi\testUser1];
go
```

Use la siguiente consulta para mostrar los nombres de usuario asignados a un rol específico:

```sql
-- Display user name assigned to a specific role
SELECT DP1.name AS DatabaseRoleName,
   isnull (DP2.name, 'No members') AS DatabaseUserName
 FROM sys.database_role_members AS DRM
 RIGHT OUTER JOIN sys.database_principals AS DP1
   ON DRM.role_principal_id = DP1.principal_id
 LEFT OUTER JOIN sys.database_principals AS DP2
   ON DRM.member_principal_id = DP2.principal_id
WHERE DP1.type = 'R'
ORDER BY DP1.name;
```

Use la sintaxis siguiente para crear un grupo. Después, agregue el grupo al rol `db_owner`.

```sql
-- Create Windows group
create user [aadsqlmi\migration] from login [aadsqlmi\migration];
go

-- ADD 'db_owner' role to this group
sp_addrolemember 'db_owner', 'aadsqlmi\migration';
go

--Check the db_owner role for 'aadsqlmi\migration' group
select is_rolemember('db_owner', 'aadsqlmi\migration')
go
-- Output  ( 1 means YES)
```

Cree una tabla de prueba y agregue algunos datos con la siguiente sintaxis:

```sql
-- Create a table and add data
create table test ( a int, b int);
go

insert into test values (1,10)
go

-- Check the table values
select * from test;
go
```

## <a name="part-3-backup-and-restore-the-individual-user-database-to-sql-managed-instance"></a>Parte 3: Copia de seguridad y restauración de la base de datos de usuario individual en Instancia administrada de SQL

Cree una copia de seguridad de la base de datos de migración mediante las indicaciones del artículo [Copy Databases with Backup and Restore](/sql/relational-databases/databases/copy-databases-with-backup-and-restore) (Copia de bases de datos con copia de seguridad y restauración), o use la siguiente sintaxis:

```sql
use master;
go
backup database migration to disk = 'C:\Migration\migration.bak';
go
```

Consulte [Inicio rápido: Restauración de una base de datos en una instancia administrada de SQL](restore-sample-database-quickstart.md).

## <a name="part-4-migrate-users-to-sql-managed-instance"></a>Parte 4: Migración de usuarios a Instancia administrada de SQL

Ejecute el comando ALTER USER para completar el proceso de migración en Instancia administrada de SQL.

1. Inicie sesión en la instancia administrada de SQL mediante la cuenta de administrador de Azure AD para Instancia administrada de SQL. A continuación, cree el inicio de sesión de Azure AD en la instancia administrada de SQL con la siguiente sintaxis. Para más información, consulte el [Tutorial: Seguridad de Instancia administrada de SQL en Azure SQL Database mediante entidades de seguridad del servidor de Azure AD (inicios de sesión)](aad-security-configure-tutorial.md).

    ```sql
    use master
    go

    -- Create login for AAD user [testUser1@aadsqlmi.net]
    create login [testUser1@aadsqlmi.net] from external provider
    go

    -- Create login for the Azure AD group [migration]. This group contains one user [testGroupUser@aadsqlmi.net]
    create login [migration] from external provider
    go

    --Check the two new logins
    select * from sys.server_principals
    go
    ```

1. Compruebe la migración de la base de datos, la tabla y las entidades de seguridad correctas.

    ```sql
    -- Switch to the database migration that is already restored for MI
    use migration;
    go

    --Check if the restored table test exist and contain a row
    select * from test;
    go

    -- Check that the SQL on-premises Windows user/group exists  
    select * from sys.database_principals;
    go
    -- the old user aadsqlmi\testUser1 should be there
    -- the old group aadsqlmi\migration should be there
    ```

1. Use la sintaxis ALTER USER para asignar el usuario local al inicio de sesión de Azure AD.

    ```sql
    /** Execute the ALTER USER command to alter the Windows user [aadsqlmi\testUser1]
    to map to the Azure AD user testUser1@aadsqlmi.net
    **/
    alter user [aadsqlmi\testUser1] with login = [testUser1@aadsqlmi.net];
    go

    -- Check the principal
    select * from sys.database_principals;
    go
    -- New user testUser1@aadsqlmi.net should be there instead
    --Check new user permissions  - should only have CONNECT permissions
    select user_name(grantee_principal_id), * from sys.database_permissions;
    go

    -- Check a specific role
    -- Display Db user name assigned to a specific role
    SELECT DP1.name AS DatabaseRoleName,
    isnull (DP2.name, 'No members') AS DatabaseUserName
    FROM sys.database_role_members AS DRM
    RIGHT OUTER JOIN sys.database_principals AS DP1
    ON DRM.role_principal_id = DP1.principal_id
    LEFT OUTER JOIN sys.database_principals AS DP2
    ON DRM.member_principal_id = DP2.principal_id
    WHERE DP1.type = 'R'
    ORDER BY DP1.name;
    ```

1. Use la sintaxis ALTER USER para asignar el grupo local al inicio de sesión de Azure AD.

    ```sql
    /** Execute ALTER USER command to alter the Windows group [aadsqlmi\migration]
    to the Azure AD group login [migration]
    **/
    alter user [aadsqlmi\migration] with login = [migration];
    -- old group migration is changed to Azure AD migration group
    go

    -- Check the principal
    select * from sys.database_principals;
    go

    --Check the group permission - should only have CONNECT permissions
    select user_name(grantee_principal_id), * from sys.database_permissions;
    go

    --Check the db_owner role for 'aadsqlmi\migration' user
    select is_rolemember('db_owner', 'migration')
    go
    -- Output 1 means 'YES'
    ```

## <a name="part-5-testing-azure-ad-user-or-group-authentication"></a>Parte 5: Prueba de autenticación de un usuario o grupo de Azure AD

Pruebe la autenticación en Instancia administrada de SQL mediante el usuario asignado previamente al inicio de sesión de Azure AD con la sintaxis ALTER USER.

1. Inicie sesión en la máquina virtual federada mediante su suscripción de Instancia administrada de Azure SQL como `aadsqlmi\testUser1`
1. Con SQL Server Management Studio (SSMS), inicie sesión en su instancia administrada de SQL con la autenticación **integrada de Active Directory** y conéctese a la base de datos `migration`.
    1. También puede iniciar sesión mediante las credenciales de testUser1@aadsqlmi.net con la opción de SSMS **Active Directory – Universal compatible con MFA**. Sin embargo, en este caso, no puede usar el mecanismo de inicio de sesión único y debe escribir una contraseña. No tendrá que usar una máquina virtual federada para iniciar sesión en la instancia administrada de SQL.
1. Como parte del miembro del rol **SELECT**, puede seleccionar en la tabla `test`

    ```sql
    Select * from test  --  and see one row (1,10)
    ```

Pruebe la autenticación en una instancia administrada de SQL con un miembro de un grupo `migration` de Windows. El usuario `aadsqlmi\testGroupUser` debe haberse agregado al grupo `migration` antes de la migración.

1. Inicie sesión en la máquina virtual federada mediante su suscripción de Instancia administrada de Azure SQL como `aadsqlmi\testGroupUser`
1. El uso de SSMS con la autenticación **integrada de Active Directory** permite conectarse al servidor de Instancia administrada de Azure SQL y a la base de datos `migration`
    1. También puede iniciar sesión mediante las credenciales de testGroupUser@aadsqlmi.net con la opción de SSMS **Active Directory – Universal compatible con MFA**. Sin embargo, en este caso, no puede usar el mecanismo de inicio de sesión único y debe escribir una contraseña. No tendrá que usar una máquina virtual federada para iniciar sesión en su instancia administrada de SQL.
1. Como parte del rol `db_owner`, puede crear una nueva tabla.

    ```sql
    -- Create table named 'new' with a default schema
    Create table dbo.new ( a int, b int)
    ```

> [!NOTE]
> Debido a un problema de diseño conocido de Azure SQL Databas, se producirá el siguiente error al crear una instrucción de tabla ejecutada como miembro de un grupo: </br> </br>
> `Msg 2760, Level 16, State 1, Line 4
The specified schema name "testGroupUser@aadsqlmi.net" either does not exist or you do not have permission to use it.` </br> </br>
> La solución actual es crear una tabla con un esquema existente en el caso anterior <dbo.new>

## <a name="next-steps"></a>Pasos siguientes

[Tutorial: Migración de SQL Server a Instancia administrada de Azure SQL sin conexión con Database Migration Service](../../dms/tutorial-sql-server-to-managed-instance.md?toc=/azure/sql-database/toc.json)
