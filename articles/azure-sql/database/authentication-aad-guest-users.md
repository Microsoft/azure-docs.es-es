---
title: Creación de usuarios invitados en Azure AD
description: Aprenda a crear usuarios invitados en Azure AD y a configurarlos como administradores sin usar grupos de Azure AD en Azure SQL Database, Azure SQL Managed Instance y Azure Synapse Analytics
ms.service: sql-db-mi
ms.subservice: security
ms.custom: azure-synapse
ms.topic: conceptual
author: shohamMSFT
ms.author: shohamd
ms.reviewer: vanto
ms.date: 07/27/2020
ms.openlocfilehash: 2c8aa39b6819e135181e2d153825e89686359538
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88053769"
---
# <a name="create-azure-ad-guest-users-and-set-as-an-azure-ad-admin"></a>Crear usuarios invitados de Azure AD y establecerlos como administradores de Azure AD

[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

> [!NOTE]
> Este artículo está en **versión preliminar pública**.

Los usuarios invitados de Azure Active Directory (Azure AD) son usuarios que se han importado en Azure AD desde otras instancias de Azure Active Directory o desde sitios externos. Por ejemplo, los usuarios invitados podrían ser usuarios de otras instancias de Azure Active Directory o de cuentas como *\@outlook.com*, *\@hotmail.com*, *\@live.com* o *\@gmail.com*. En este artículo, aprenderá a crear un usuario invitado de Azure AD y a configurarlo como administrador de Azure AD en el servidor lógico de Azure SQL sin necesidad de que forme parte de un grupo de Azure AD.

## <a name="feature-description"></a>Descripción de la característica

Esta característica mejora la limitación actual, que solo permite que los usuarios invitados puedan conectarse a Azure SQL Database, SQL Managed Instance o Azure Synapse Analytics cuando son miembros de un grupo creado en Azure AD. Además, el grupo tenía que asignarse manualmente a un usuario utilizando la instrucción [CREATE USER (Transact-SQL)](https://docs.microsoft.com/sql/t-sql/statements/create-user-transact-sql) en una base de datos determinada. Una vez creado el usuario de base de datos para el grupo de Azure AD que contenía el usuario invitado, el usuario invitado podía iniciar sesión en la base de datos utilizando Azure Active Directory con la autenticación multifactor. En esta **versión preliminar pública**, los usuarios invitados se pueden crear y conectar directamente a SQL Database, SQL Managed Instance o Azure Synapse sin necesidad de agregarlos primero a un grupo de Azure AD y de crear después un usuario de base de datos para ese grupo de Azure AD.

Esta característica también permite establecer directamente el usuario invitado como administrador de Azure AD en el servidor lógico de Azure SQL. La funcionalidad existente, en la que el usuario invitado puede formar parte de un grupo de Azure AD y dicho grupo puede configurarse como administrador de Azure AD en el servidor lógico de Azure SQL, no se ha visto afectada. Los usuarios invitados de la base de datos que forman parte de un grupo de Azure AD tampoco se han visto afectados por este cambio.

Para más información sobre la funcionalidad que permite usar usuarios invitados con grupos de Azure AD, consulte [Uso de la autenticación multifactor de Azure Active Directory](authentication-mfa-ssms-overview.md).

## <a name="prerequisite"></a>Requisito previo

- Si se utiliza PowerShell para configurar un usuario invitado como administrador de Azure AD para el servidor lógico de Azure SQL, se necesita el módulo [Az.Sql 2.9.0](https://www.powershellgallery.com/packages/Az.Sql/2.9.0) o una versión posterior.

## <a name="create-database-user-for-azure-ad-guest-user"></a>Creación de un usuario de base de datos como usuario invitado de Azure AD 

Siga estos pasos para crear un usuario de base de datos que se utilice como usuario invitado de Azure AD.

### <a name="create-guest-user-in-sql-database-and-azure-synapse"></a>Creación de un usuario invitado en Azure SQL Database y Azure Synapse

1. Asegúrese de que el usuario invitado (por ejemplo, `user1@gmail.com`) ya esté agregado en Azure AD y de que se haya configurado un administrador de Azure AD para el servidor de bases de datos. Para poder autenticarse en Azure Active Directory, se necesita un administrador de Azure AD.

1. Conéctese a la base de datos SQL como administrador de Azure AD o como un usuario de Azure AD con permisos de SQL suficientes como para crear usuarios y ejecute el comando siguiente en la base de datos en la que debe agregarse el usuario invitado:

    ```sql
    CREATE USER [user1@gmail.com] FROM EXTERNAL PROVIDER
    ```

1. Ahora debería haber un usuario de base de datos para el usuario invitado `user1@gmail.com`.

1. Ejecute el comando siguiente para comprobar si el usuario de base de datos se creó correctamente:

    ```sql
    SELECT * FROM sys.database_principals
    ```

1. Desconéctese de la base de datos e inicie sesión de nuevo como el usuario invitado `user1@gmail.com` utilizando [SQL Server Management Studio (SSMS)](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) y el método de autenticación **Azure Active Directory - Universal con MFA**. Para más información, consulte [Uso de la autenticación multifactor de Azure Active Directory](authentication-mfa-ssms-overview.md).

### <a name="create-guest-user-in-sql-managed-instance"></a>Creación de un usuario invitado en Azure SQL Managed Instance

> [!NOTE]
> Azure SQL Managed Instance admite los inicios de sesión de usuarios de Azure AD, así como de los usuarios de base de datos independiente de Azure AD. El procedimiento siguiente permite crear un inicio de sesión y un usuario para un usuario invitado de Azure AD en SQL Managed Instance. Si lo desea, también puede crear un [usuario de base de datos independiente](https://docs.microsoft.com/sql/relational-databases/security/contained-database-users-making-your-database-portable) en Azure SQL Managed Instance siguiendo el método que se describe en la sección [Creación de un usuario invitado en Azure SQL Database y Azure Synapse](#create-guest-user-in-sql-database-and-azure-synapse).

1. Asegúrese de que el usuario invitado (por ejemplo, `user1@gmail.com`) ya esté agregado en Azure AD y de que se haya configurado un administrador de Azure AD para Azure SQL Managed Instance. Para poder autenticarse en Azure Active Directory, se necesita un administrador de Azure AD.

1. Conéctese al servidor de Azure SQL Managed Instance como administrador de Azure AD o como un usuario de Azure AD con permisos de SQL suficientes para crear usuarios y ejecute el comando siguiente en la base de datos `master`, con lo que creará un inicio de sesión para el usuario invitado:

    ```sql
    CREATE LOGIN [user1@gmail.com] FROM EXTERNAL PROVIDER
    ```

1. Ahora debería haber un usuario de base de datos para el usuario invitado `user1@gmail.com` en la base de datos `master`.

1. Ejecute el comando siguiente para comprobar si el inicio de sesión se creó correctamente:

    ```sql
    SELECT * FROM sys.server_principals
    ```

1. Ejecute el comando siguiente en la base de datos en la que debe agregarse el usuario invitado: 

    ```sql
    CREATE USER [user1@gmail.com] FROM LOGIN [user1@gmail.com]
    ```

1. Ahora debería haber un usuario de base de datos para el usuario invitado `user1@gmail.com`.

1. Desconéctese de la base de datos e inicie sesión de nuevo como el usuario invitado `user1@gmail.com` utilizando [SQL Server Management Studio (SSMS)](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) y el método de autenticación **Azure Active Directory - Universal con MFA**. Para más información, consulte [Uso de la autenticación multifactor de Azure Active Directory](authentication-mfa-ssms-overview.md).

## <a name="setting-a-guest-user-as-an-azure-ad-admin"></a>Configuración de un usuario invitado como administrador de Azure AD

Siga estos pasos para configurar un usuario invitado de Azure AD como administrador de Azure AD en el servidor lógico de SQL.

### <a name="set-azure-ad-admin-for-sql-database-and-azure-synapse"></a>Configuración de un administrador de Azure AD para Azure SQL Database y Azure Synapse

1. Asegúrese de que el usuario invitado (por ejemplo, `user1@gmail.com`) ya se haya agregado a Azure AD.

1. Ejecute el siguiente comando de PowerShell para agregar el usuario invitado como administrador de Azure AD en el servidor lógico de Azure SQL:

    - Sustituya `<ResourceGroupName>` por el nombre del grupo de recursos de Azure que contiene el servidor lógico de Azure SQL.
    - Sustituya `<ServerName>` por el nombre del servidor lógico de Azure SQL. Si el nombre del servidor es `myserver.database.windows.net`, sustituya `<Server Name>` por `myserver`.
    - Sustituya `<DisplayNameOfGuestUser>` por el nombre del usuario invitado.

    ```powershell
    Set-AzSqlServerActiveDirectoryAdministrator -ResourceGroupName <ResourceGroupName> -ServerName <ServerName> -DisplayName <DisplayNameOfGuestUser>
    ```

    También puede usar el comando [az sql server ad-admin](https://docs.microsoft.com/cli/azure/sql/server/ad-admin) de la CLI de Azure para establecer el usuario invitado como un administrador de Azure AD en el servidor lógico de Azure SQL.

### <a name="set-azure-ad-admin-for-sql-managed-instance"></a>Configuración del administrador de Azure AD en SQL Managed Instance

1. Asegúrese de que el usuario invitado (por ejemplo, `user1@gmail.com`) ya se haya agregado a Azure AD.

1. Vaya a [Azure Portal](https://portal.azure.com) y acceda al recurso **Azure Active Directory**. En **Administrar**, vaya al panel **Usuarios**. Seleccione el usuario invitado y anote el valor de `Object ID`. 

1. Ejecute el siguiente comando de PowerShell para agregar el usuario invitado como administrador de Azure AD en Azure SQL Managed Instance:

    - Sustituya `<ResourceGroupName>` por el nombre del grupo de recursos de Azure que contiene la instancia de Azure SQL Managed Instance.
    - Sustituya `<ManagedInstanceName>` por el nombre de la instancia de Azure SQL Managed Instance.
    - Sustituya `<DisplayNameOfGuestUser>` por el nombre del usuario invitado.
    - Sustituya `<AADObjectIDOfGuestUser>` por el valor de `Object ID` que anotó anteriormente.

    ```powershell
    Set-AzSqlInstanceActiveDirectoryAdministrator -ResourceGroupName <ResourceGroupName> -InstanceName "<ManagedInstanceName>" -DisplayName <DisplayNameOfGuestUser> -ObjectId <AADObjectIDOfGuestUser>
    ```

    También puede usar el comando [az sql server ad-admin](https://docs.microsoft.com/cli/azure/sql/mi/ad-admin) de la CLI de Azure para establecer el usuario invitado como administrador de Azure AD en Azure SQL Managed Instance.

## <a name="limitations"></a>Limitaciones

Azure Portal tiene una limitación que impide seleccionar un usuario invitado de Azure AD como administrador de Azure AD en SQL Managed Instance. En el caso de las cuentas de invitado ajenas a Azure AD, como *\@outlook.com*, *\@hotmail.com*, *\@live.com* o *\@gmail.com*, estas cuentas aparecerán en el selector de administrador de AD, pero estarán atenuadas y no podrán seleccionarse. Utilice los [comandos de PowerShell o la CLI](#setting-a-guest-user-as-an-azure-ad-admin) anteriores para configurar el administrador de Azure AD. Si lo desea, también puede establecer como administrador de Azure AD en SQL Managed Instance a un grupo de Azure AD que contenga al usuario invitado.

Esta funcionalidad estará habilitada en SQL Managed Instance antes de que esté disponible con carácter general.

## <a name="next-steps"></a>Pasos siguientes

- [Configuración y administración de la autenticación de Azure AD con Azure SQL](authentication-aad-configure.md)
- [Uso de la autenticación multifactor de Azure Active Directory](authentication-mfa-ssms-overview.md)
- [CREATE USER (Transact-SQL)](https://docs.microsoft.com/sql/t-sql/statements/create-user-transact-sql)
