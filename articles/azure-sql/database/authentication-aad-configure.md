---
title: Configuración de la autenticación de Azure Active Directory
titleSuffix: Azure SQL Database & SQL Managed Instance & Azure Synapse Analytics
description: Aprenda a conectarse a SQL Database, una Instancia administrada de SQL y Azure Synapse Analytics mediante la autenticación de Azure Active Directory, después de configurar Azure AD.
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
ms.custom: azure-synapse, has-adal-ref, sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
author: GithubMirek
ms.author: mireks
ms.reviewer: vanto, carlrab
ms.date: 07/27/2020
ms.openlocfilehash: f98e540a6764869f1d37edfbb0f00bf8d1cc2198
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87499184"
---
# <a name="configure-and-manage-azure-ad-authentication-with-azure-sql"></a>Configuración y administración de la autenticación de Azure AD con Azure SQL

[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

En este artículo se muestra cómo crear y rellenar una instancia de Azure Active Directory (Azure AD) y, a continuación, usarla con [Azure SQL Database](sql-database-paas-overview.md), una [Instancia administrada de Azure SQL](../managed-instance/sql-managed-instance-paas-overview.md) y [Azure Synapse Analytics (anteriormente, Azure SQL Data Warehouse)](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md). Para obtener información general, consulte [Autenticación con Azure Active Directory](authentication-aad-overview.md).

## <a name="azure-ad-authentication-methods"></a>Métodos de autenticación de Azure AD

La autenticación de Azure AD admite los siguientes métodos de autenticación:

- Identidades solo en la nube de Azure AD
- Identidades híbridas de Azure AD que admiten:
  - Autenticación en la nube con dos opciones asociadas al inicio de sesión único de conexión directa
    - Autenticación de hash de contraseñas de Azure AD
    - Autenticación de paso a través de Azure AD
  - Autenticación federada

Para obtener más información sobre los métodos de autenticación de Azure AD y cuál debe elegir, vea [Seleccionar el método de autenticación adecuado para su solución de identidad híbrida de Azure Active Directory](../../active-directory/hybrid/choose-ad-authn.md).

Para obtener más información sobre las identidades híbridas de Azure AD, la configuración y la sincronización, consulte los siguientes artículos:

- Autenticación de hash de contraseñas - [Implementación de la sincronización de hash de contraseñas con la sincronización de Azure AD Connect](../../active-directory/hybrid/how-to-connect-password-hash-synchronization.md)
- Autenticación de paso a través: [Autenticación de paso a través de Azure Active Directory](../../active-directory/hybrid/how-to-connect-pta-quick-start.md).
- Autenticación federada: [Implementación de Servicios de federación de Active Directory en Azure](/windows-server/identity/ad-fs/deployment/how-to-connect-fed-azure-adfs) y [Azure AD Connect y la federación](../../active-directory/hybrid/how-to-connect-fed-whatis.md)

## <a name="create-and-populate-an-azure-ad-instance"></a>Creación y relleno de una instancia de Azure AD

Cree una instancia de Azure AD y rellénela con usuarios y grupos. Azure AD puede ser el dominio administrado de Azure AD inicial. Azure AD también puede ser una instancia local de Active Directory Domain Services que se federa con Azure AD.

Para obtener más información, consulte [Integrating your on-premises identities with Azure Active Directory](../../active-directory/hybrid/whatis-hybrid-identity.md) (Integración de sus identidades locales con Azure Active Directory), [Incorporación de su nombre de dominio personalizado a Azure Active Directory](../../active-directory/fundamentals/add-custom-domain.md), [Microsoft Azure now supports federation with Windows Server Active Directory](https://azure.microsoft.com/blog/20../../windows-azure-now-supports-federation-with-windows-server-active-directory/) (Microsoft Azure admite ahora Windows Server Active Directory), [Administración de su directorio de Azure AD](../../active-directory/fundamentals/active-directory-whatis.md), [Administrar Azure AD mediante Windows PowerShell](/powershell/azure/) y [Hybrid Identity Required Ports and Protocols](../../active-directory/hybrid/reference-connect-ports.md) (Puertos y protocolos requeridos para la identidad híbrida).

## <a name="associate-or-add-an-azure-subscription-to-azure-active-directory"></a>Asociación o adición de una suscripción de Azure a Azure Active Directory

1. Asocie la suscripción de Azure a Azure Active Directory mediante la creación del directorio como un directorio de confianza para la suscripción de Azure que hospeda la base de datos. Para obtener detalles, consulte [Asociación o incorporación de una suscripción de Azure al inquilino de Azure Active Directory](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).

2. Use el modificador del directorio en Azure Portal para cambiar a la suscripción asociada con el dominio.

   > [!IMPORTANT]
   > Cada suscripción de Azure tiene una relación de confianza con una instancia de Azure AD. Esto significa que confía en ese directorio para autenticar usuarios, servicios y dispositivos. Varias suscripciones pueden confiar en el mismo directorio, pero una suscripción confía solo en un único directorio. Esta relación de confianza que tiene una suscripción con un directorio es diferente de la relación que tiene una suscripción con todos los demás recursos de Azure (sitios web, bases de datos etc.), que son más parecidos a los recursos secundarios de una suscripción. Si una suscripción expira, el acceso a esos otros recursos asociados a la suscripción también se detiene. Sin embargo, el directorio permanece en Azure y puede asociar otra suscripción a ese directorio y continuar con la administración de los usuarios del directorio. Para más información sobre recursos, consulte [Descripción de acceso a los recursos de Azure](../../active-directory/b2b/add-users-administrator.md). Para más información sobre esta relación de confianza, vea [Asociación o adición de una suscripción de Azure a Azure Active Directory](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).

## <a name="azure-ad-admin-with-a-server-in-sql-database"></a>Administrador de Azure AD con un servidor en SQL Database

Cada [servidor](logical-servers.md) de Azure (que hospeda SQL Database o Azure Synapse) se inicia con una cuenta de administrador del servidor única, que es el administrador de todo el servidor. Cree una segunda cuenta de administrador como cuenta de Azure AD. Esta entidad de seguridad se crea como un usuario de base de datos independiente en la base de datos maestra del servidor. Las cuentas de administrador son miembros del rol **db_owner** en todas las bases de datos de usuarios y acceden a cada base de datos de usuario con el usuario **dbo**. Para obtener más información sobre las cuentas de administrador, consulte [cómo administrar bases de datos e inicios de sesión](logins-create-manage.md).

Cuando se usa Azure Active Directory con replicación geográfica, el administrador de Azure Active Directory debe configurarse para los servidores principal y secundarios. Si un servidor no tiene ningún administrador de Azure Active Directory, los usuarios y los inicios de sesión de Azure Active Directory reciben un error "`Cannot connect` con el servidor".

> [!NOTE]
> Los usuarios que no se basen en una cuenta de Azure AD (incluida la cuenta de administrador del servidor) no pueden crear usuarios basados en Azure AD porque no tienen ningún permiso para validar a los usuarios de la base de datos propuesta con Azure AD.

## <a name="provision-azure-ad-admin-sql-managed-instance"></a>Aprovisionamiento de un administrador de Azure AD (Instancia administrada de SQL)

> [!IMPORTANT]
> Siga estos pasos únicamente si va a aprovisionar una Instancia administrada de Azure SQL. Esta operación solo la puede ejecutar el administrador global o de la compañía, o un administrador con un rol con privilegios en Azure AD. Los pasos siguientes describen el proceso de concesión de permisos a los usuarios con privilegios diferentes en el directorio.

La Instancia administrada de SQL necesita permisos para leer en Azure AD a fin de realizar correctamente tareas como la autenticación de usuarios mediante la pertenencia a grupos de seguridad o la creación de nuevos usuarios. Para que esto funcione, debe conceder el permiso a la Instancia administrada de SQL para leer en Azure AD. Para ello, puede usar Azure Portal o PowerShell.

### <a name="azure-portal"></a>Azure portal

Para conceder a la Instancia administrada de SQL permiso de lectura de Azure AD mediante Azure Portal, inicie sesión como administrador global o de empresa en Azure AD y siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), en la esquina superior derecha, seleccione la conexión en una lista desplegable de posibles instancias de Active Directory.

2. Elija el Active Directory correcto como el valor predeterminado de Azure AD.

   En este paso se vincula la suscripción asociada a Active Directory con la Instancia administrada de SQL, de modo que se asegura que se usa la misma suscripción tanto para la instancia de Azure AD como para la Instancia administrada de SQL.

3. Vaya a la Instancia administrada de SQL que quiera usar para la integración de Azure AD.

   ![aad](./media/authentication-aad-configure/aad.png)

4. Seleccione el banner encima de la página de administrador de Active Directory y conceda permiso al usuario actual.

    ![grant permissions-portal](./media/authentication-aad-configure/grant-permissions.png)

5. Después de que la operación se complete correctamente, la siguiente notificación se mostrará en la esquina superior derecha:

    ![success](./media/authentication-aad-configure/success.png)

6. Ahora puede elegir el administrador de Azure AD para su Instancia administrada de SQL. Para ello, en la página Administrador de Active Directory, seleccione el comando **Establecer administrador**.

    ![set-admin](./media/authentication-aad-configure/set-admin.png)

7. En la página de administrador de Azure AD, busque un usuario, seleccione el usuario o grupo que se convertirá en administrador y, después, elija **Seleccionar**.

   En la hoja Administrador de Active Directory se muestran todos los miembros y grupos de Active Directory. No se pueden seleccionar los usuarios o grupos que aparecen atenuados porque no se admiten como administradores de Azure AD. Consulte la lista de administradores admitidos en la sección [Características y limitaciones de Azure AD](authentication-aad-overview.md#azure-ad-features-and-limitations). El control de acceso basado en rol (RBAC) se aplica solo a Azure Portal y no se propaga a SQL Database, la Instancia administrada de SQL ni Azure Synapse.

    ![Incorporación del administrador de Azure Active Directory](./media/authentication-aad-configure/add-azure-active-directory-admin.png)

8. En la parte superior de la página Administrador de Active Directory, seleccione **Guardar**.

    ![Guardar](./media/authentication-aad-configure/save.png)

    El proceso de cambio del el administrador puede tardar varios minutos. El nuevo administrador aparecerá después en el cuadro Administrador de Active Directory.

Después de aprovisionar un administrador de Azure AD para la Instancia administrada de SQL, puede empezar a crear entidades de seguridad de servidor de Azure AD (inicios de sesión) con la sintaxis <a href="/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current">CREATE LOGIN</a>. Para obtener más información, consulte [Información general sobre la Instancia administrada de SQL](../managed-instance/sql-managed-instance-paas-overview.md#azure-active-directory-integration).

> [!TIP]
> Si más adelante desea quitar un administrador, en la parte superior de la página Administrador de Active Directory, seleccione **Quitar administrador** y después **Guardar**.

### <a name="powershell"></a>PowerShell

Para conceder a la Instancia administrada de SQL el permiso de lectura de Azure AD mediante el uso de PowerShell, ejecute este script:

```powershell
# Gives Azure Active Directory read permission to a Service Principal representing the SQL Managed Instance.
# Can be executed only by a "Company Administrator", "Global Administrator", or "Privileged Role Administrator" type of user.

$aadTenant = "<YourTenantId>" # Enter your tenant ID
$managedInstanceName = "MyManagedInstance"

# Get Azure AD role "Directory Users" and create if it doesn't exist
$roleName = "Directory Readers"
$role = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq $roleName}
if ($role -eq $null) {
    # Instantiate an instance of the role template
    $roleTemplate = Get-AzureADDirectoryRoleTemplate | Where-Object {$_.displayName -eq $roleName}
    Enable-AzureADDirectoryRole -RoleTemplateId $roleTemplate.ObjectId
    $role = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq $roleName}
}

# Get service principal for your SQL Managed Instance
$roleMember = Get-AzureADServicePrincipal -SearchString $managedInstanceName
$roleMember.Count
if ($roleMember -eq $null) {
    Write-Output "Error: No Service Principals with name '$    ($managedInstanceName)', make sure that managedInstanceName parameter was     entered correctly."
    exit
}
if (-not ($roleMember.Count -eq 1)) {
    Write-Output "Error: More than one service principal with name pattern '$    ($managedInstanceName)'"
    Write-Output "Dumping selected service principals...."
    $roleMember
    exit
}

# Check if service principal is already member of readers role
$allDirReaders = Get-AzureADDirectoryRoleMember -ObjectId $role.ObjectId
$selDirReader = $allDirReaders | where{$_.ObjectId -match     $roleMember.ObjectId}

if ($selDirReader -eq $null) {
    # Add principal to readers role
    Write-Output "Adding service principal '$($managedInstanceName)' to     'Directory Readers' role'..."
    Add-AzureADDirectoryRoleMember -ObjectId $role.ObjectId -RefObjectId     $roleMember.ObjectId
    Write-Output "'$($managedInstanceName)' service principal added to     'Directory Readers' role'..."

    #Write-Output "Dumping service principal '$($managedInstanceName)':"
    #$allDirReaders = Get-AzureADDirectoryRoleMember -ObjectId $role.ObjectId
    #$allDirReaders | where{$_.ObjectId -match $roleMember.ObjectId}
}
else {
    Write-Output "Service principal '$($managedInstanceName)' is already     member of 'Directory Readers' role'."
}
```

### <a name="powershell-for-sql-managed-instance"></a>PowerShell para la Instancia administrada de SQL

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para ejecutar los cmdlets de PowerShell, necesitará tener Azure PowerShell instalado y en marcha. Para obtener información detallada, vea [Instalación y configuración de Azure PowerShell](/powershell/azure/).

> [!IMPORTANT]
> El módulo de Azure Resource Manager (RM) de PowerShell todavía es compatible con la Instancia administrada de Azure SQL, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. El módulo de AzureRM continuará recibiendo correcciones de errores hasta diciembre de 2020 como mínimo.  Los argumentos para los comandos del módulo Az y los módulos AzureRm son esencialmente idénticos. Para obtener más información sobre la compatibilidad, vea [Presentación del nuevo módulo Az de Azure PowerShell](/powershell/azure/new-azureps-module-az).

Para aprovisionar un administrador de Azure AD, debe ejecutar los siguientes comandos de Azure PowerShell:

- Connect-AzAccount
- Select-AzSubscription

Los cmdlets que se usan para aprovisionar y administrar el administrador de Azure AD para la Instancia administrada de SQL se enumeran en la tabla siguiente:

| Nombre del cmdlet | Descripción |
| --- | --- |
| [Set-AzSqlInstanceActiveDirectoryAdministrator](/powershell/module/az.sql/set-azsqlinstanceactivedirectoryadministrator) |Aprovisiona un administrador de Azure AD para la Instancia administrada de SQL de la suscripción actual. (Debe ser de la suscripción actual).|
| [Remove-AzSqlInstanceActiveDirectoryAdministrator](/powershell/module/az.sql/remove-azsqlinstanceactivedirectoryadministrator) |Quita un administrador de Azure AD para la Instancia administrada de SQL de la suscripción actual. |
| [Get-AzSqlInstanceActiveDirectoryAdministrator](/powershell/module/az.sql/get-azsqlinstanceactivedirectoryadministrator) |Devuelve información sobre un administrador de Azure AD para la Instancia administrada de SQL de la suscripción actual.|

El siguiente comando obtiene información sobre un administrador de Azure AD para una Instancia administrada de SQL, denominada ManagedInstance01, que está asociada a un grupo de recursos denominado ResourceGroup01.

```powershell
Get-AzSqlInstanceActiveDirectoryAdministrator -ResourceGroupName "ResourceGroup01" -InstanceName "ManagedInstance01"
```

El siguiente comando aprovisiona un grupo de administradores de Azure AD denominado DBAs para la Instancia administrada de SQL denominada ManagedInstance01. Este servidor está asociado al grupo de recursos ResourceGroup01.

```powershell
Set-AzSqlInstanceActiveDirectoryAdministrator -ResourceGroupName "ResourceGroup01" -InstanceName "ManagedInstance01" -DisplayName "DBAs" -ObjectId "40b79501-b343-44ed-9ce7-da4c8cc7353b"
```

El siguiente comando elimina al administrador de Azure AD para la Instancia administrada de SQL, denominada ManagedInstanceName01, que está asociada a un grupo de recursos denominado ResourceGroup01.

```powershell
Remove-AzSqlInstanceActiveDirectoryAdministrator -ResourceGroupName "ResourceGroup01" -InstanceName "ManagedInstanceName01" -Confirm -PassThru
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

También puede aprovisionar un administrador de Azure AD para la Instancia administrada de SQL mediante una llamada a los siguientes comandos de la CLI:

| Get-Help | Descripción |
| --- | --- |
|[az sql mi ad-admin create](/cli/azure/sql/mi/ad-admin#az-sql-mi-ad-admin-create) | Aprovisiona un administrador de Azure Active Directory para la Instancia administrada de SQL (debe ser de la suscripción actual). |
|[az sql mi ad-admin delete](/cli/azure/sql/mi/ad-admin#az-sql-mi-ad-admin-delete) | Elimina un administrador de Azure Active Directory para la Instancia administrada de SQL. |
|[az sql mi ad-admin list](/cli/azure/sql/mi/ad-admin#az-sql-mi-ad-admin-list) | Devuelve información sobre un administrador de Azure Active Directory configurado actualmente para la Instancia administrada de SQL. |
|[az sql mi ad-admin update](/cli/azure/sql/mi/ad-admin#az-sql-mi-ad-admin-update) | Actualiza el administrador de Active Directory para la Instancia administrada de SQL. |

Para más información sobre los comandos de la CLI, consulte [az sql mi](/cli/azure/sql/mi).

* * *

## <a name="provision-azure-ad-admin-sql-database"></a>Aprovisionamiento del administrador de Azure AD (SQL Database)

> [!IMPORTANT]
> Simplemente siga estos pasos si va a aprovisionar un [servidor](logical-servers.md) de SQL Database o Azure Synapse.

Los dos procedimientos siguientes muestran cómo aprovisionar un administrador de Azure Active Directory para el servidor en Azure Portal y mediante el uso de PowerShell.

### <a name="azure-portal"></a>Azure portal

1. En [Azure Portal](https://portal.azure.com/), en la esquina superior derecha, seleccione la conexión para desplegar una lista de posibles instancias de Active Directory. Elija el Active Directory correcto como el valor predeterminado de Azure AD. En este paso se vincula la instancia de Active Directory asociada a la suscripción con el servidor, de modo que se asegura de que la misma suscripción se use tanto para Azure AD como para el servidor.

2. Busque y seleccione **SQL Server**.

    ![Buscar y seleccionar servidores SQL](./media/authentication-aad-configure/search-for-and-select-sql-servers.png)

    >[!NOTE]
    > En esta página, antes de seleccionar **Servidores SQL Server**, puede seleccionar la **estrella** junto al nombre para incluir la categoría en *favoritos* y agregar **Servidores SQL Server** a la barra de navegación izquierda.

3. En la página **SQL Server**, seleccione **Administrador de Active Directory**.

4. En la página **Administrador de Active Directory**, seleccione **Establecer administrador**.

    ![Conjunto de servidores SQL en Administrador de Active Directory](./media/authentication-aad-configure/sql-servers-set-active-directory-admin.png)  

5. En la página **Agregar administrador**, busque un usuario, seleccione el usuario o grupo que convertirá en administrador y después haga clic en **Seleccionar**. En la hoja Administrador de Active Directory se muestran todos los miembros y grupos de Active Directory. No se pueden seleccionar los usuarios o grupos que aparecen atenuados porque no se admiten como administradores de Azure AD. (Consulte la lista de administradores admitidos en la sección **Características y limitaciones de Azure AD** del artículo [Uso de la autenticación de Azure Active Directory con SQL Database o Azure Synapse](authentication-aad-overview.md)). El control de acceso basado en rol (RBAC) se aplica solo al portal y no se propaga a SQL Server.

    ![Seleccionar administrador de Azure Active Directory](./media/authentication-aad-configure/select-azure-active-directory-admin.png)  

6. En la parte superior de la página **Administrador de Active Directory**, seleccione **GUARDAR**.

    ![guardar administrador](./media/authentication-aad-configure/save-admin.png)

El proceso de cambio del el administrador puede tardar varios minutos. El nuevo administrador aparece en el cuadro **Administrador de Active Directory** .

   > [!NOTE]
   > Al configurar el administrador de Azure AD, el nuevo nombre del administrador (usuario o grupo) no puede estar en la base de datos maestra virtual como usuario de autenticación del servidor. Si estuviera presente, se producirá un error en el programa de instalación del administrador de Azure AD y, por lo tanto, deberá revertir su creación e indicar que ese administrador (nombre) ya existe. Dado que dicho usuario de autenticación del servidor no forma parte de Azure AD, se producirá un error cada vez que intente conectarse al servidor mediante la autenticación de Azure AD.

Si más adelante desea quitar un administrador, en la parte superior de la página **Administrador de Active Directory**, seleccione **Quitar administrador** y después **Guardar**.

### <a name="powershell-for-sql-database-and-azure-synapse"></a>PowerShell para SQL Database y Azure Synapse

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para ejecutar los cmdlets de PowerShell, necesitará tener Azure PowerShell instalado y en marcha. Para obtener información detallada, vea [Instalación y configuración de Azure PowerShell](/powershell/azure/). Para aprovisionar un administrador de Azure AD, debe ejecutar los siguientes comandos de Azure PowerShell:

- Connect-AzAccount
- Select-AzSubscription

Los cmdlets que se usan para aprovisionar y administrar el administrador de Azure AD para SQL Database y Azure Synapse son los siguientes:

| Nombre del cmdlet | Descripción |
| --- | --- |
| [Set-AzSqlServerActiveDirectoryAdministrator](/powershell/module/az.sql/set-azsqlserveractivedirectoryadministrator) |Aprovisiona un administrador de Azure Active Directory para el servidor que hospeda SQL Database o Azure Synapse. (Debe ser de la suscripción actual). |
| [Remove-AzSqlServerActiveDirectoryAdministrator](/powershell/module/az.sql/remove-azsqlserveractivedirectoryadministrator) |Quita un administrador de Azure Active Directory para el servidor que hospeda SQL Database o Azure Synapse.|
| [Get-AzSqlServerActiveDirectoryAdministrator](/powershell/module/az.sql/get-azsqlserveractivedirectoryadministrator) |Devuelve información sobre un administrador de Azure Active Directory configurado actualmente para el servidor que hospeda SQL Database o Azure Synapse. |

Use el comando de PowerShell get-help para ver más información de cada uno de estos comandos. Por ejemplo, `get-help Set-AzSqlServerActiveDirectoryAdministrator`.

El script siguiente aprovisiona un grupo de administradores de Azure AD denominado **DBA_Group** (id. de objeto `40b79501-b343-44ed-9ce7-da4c8cc7353f`) para el servidor **demo_server**, en un grupo de recursos llamado **Group-23**:

```powershell
Set-AzSqlServerActiveDirectoryAdministrator -ResourceGroupName "Group-23" -ServerName "demo_server" -DisplayName "DBA_Group"
```

El parámetro de entrada **DisplayName** acepta tanto el nombre para mostrar de Azure AD, como el nombre principal de usuario. Por ejemplo, ``DisplayName="John Smith"`` y ``DisplayName="johns@contoso.com"``. En los grupos de Azure AD solo se admite el nombre para mostrar de Azure AD.

> [!NOTE]
> El comando ```Set-AzSqlServerActiveDirectoryAdministrator``` de Azure PowerShell no le impedirá aprovisionar administradores de Azure AD para usuarios no admitidos. Es posible aprovisionar un usuario no admitido, pero no podrá conectarse a una base de datos.

En el ejemplo siguiente se usa el elemento opcional **ObjectID**:

```powershell
Set-AzSqlServerActiveDirectoryAdministrator -ResourceGroupName "Group-23" -ServerName "demo_server" `
    -DisplayName "DBA_Group" -ObjectId "40b79501-b343-44ed-9ce7-da4c8cc7353f"
```

> [!NOTE]
> El elemento **ObjectID** de Azure AD es necesario cuando el valor **DisplayName** no es único. Para recuperar los valores **ObjectID** y **DisplayName**, use la sección Active Directory del Portal de Azure clásico y visualice las propiedades de un usuario o grupo.

En el ejemplo siguiente se devuelve información sobre el administrador actual de Azure AD para el servidor:

```powershell
Get-AzSqlServerActiveDirectoryAdministrator -ResourceGroupName "Group-23" -ServerName "demo_server" | Format-List
```

En el ejemplo siguiente se quita un administrador de Azure AD:

```powershell
Remove-AzSqlServerActiveDirectoryAdministrator -ResourceGroupName "Group-23" -ServerName "demo_server"
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Puede aprovisionar un administrador de Azure AD mediante una llamada a los siguientes comandos de la CLI:

| Get-Help | Descripción |
| --- | --- |
|[az sql server ad-admin create](/cli/azure/sql/server/ad-admin#az-sql-server-ad-admin-create) | Aprovisiona un administrador de Azure Active Directory para el servidor que hospeda SQL Database o Azure Synapse. (Debe ser de la suscripción actual). |
|[az sql server ad-admin delete](/cli/azure/sql/server/ad-admin#az-sql-server-ad-admin-delete) | Quita un administrador de Azure Active Directory para el servidor que hospeda SQL Database o Azure Synapse. |
|[az sql server ad-admin list](/cli/azure/sql/server/ad-admin#az-sql-server-ad-admin-list) | Devuelve información sobre un administrador de Azure Active Directory configurado actualmente para el servidor que hospeda SQL Database o Azure Synapse. |
|[az sql server ad-admin update](/cli/azure/sql/server/ad-admin#az-sql-server-ad-admin-update) | Actualiza un administrador de Active Directory para el servidor que hospeda SQL Database o Azure Synapse. |

Para más información sobre los comandos de la CLI, consulte [az sql server](/cli/azure/sql/server).

* * *

> [!NOTE]
> También puede aprovisionar un administrador de Azure Active Directory mediante las API de REST. Para más información, consulte [Service Management REST API Reference and Operations for Azure SQL Databases](/rest/api/sql/) (Referencia de la API REST de administración de servicio y operaciones para bases de datos de Azure SQL).

## <a name="configure-your-client-computers"></a>Configurar los equipos cliente.

En todas las máquinas cliente, desde las que las aplicaciones o los usuarios se conectan a SQL Database o a Azure Synapse mediante identidades de Azure AD, debe instalar el software siguiente:

- .NET Framework 4.6 o posterior desde [https://msdn.microsoft.com/library/5a4x27ek.aspx](https://msdn.microsoft.com/library/5a4x27ek.aspx).
- Biblioteca de autenticación de Azure Active Directory para SQL Server (*ADAL.DLL*). A continuación se muestran los vínculos de descarga para instalar la versión más reciente del controlador SSMS, ODBC y OLE DB que contiene la biblioteca *ADAL.DLL*.
  - [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms)
  - [Controlador ODBC 17 para SQL Server](https://www.microsoft.com/download/details.aspx?id=56567)
  - [Controlador OLE DB 18 para SQL Server](https://www.microsoft.com/download/details.aspx?id=56730)

Puede cumplir estos requisitos mediante:

- La instalación de la versión más reciente de [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) o de [SQL Server Data Tools](/sql/ssdt/download-sql-server-data-tools-ssdt) cumple los requisitos de .NET Framework 4.6.
  - SSMS instala la versión x86 de *ADAL.DLL*.
  - SSDT instala la versión amd64 de *ADAL.DLL*.
  - La versión más reciente de Visual Studio de la sección [Descargas de Visual Studio](https://www.visualstudio.com/downloads/download-visual-studio-vs) cumple con los requisitos de .NET Framework 4.6, pero no instala la versión amd64 necesaria de *ADAL.DLL*.

## <a name="create-contained-users-mapped-to-azure-ad-identities"></a>Creación de usuarios independientes asignados a identidades de Azure AD

Dado que la Instancia administrada de SQL admite entidades de seguridad de servidor de Azure AD (inicios de sesión), no es necesario utilizar usuarios de bases de datos independientes. Las entidades de seguridad de servidor de Azure AD (inicios de sesión) le permiten crear inicios de sesión de usuarios, grupos o aplicaciones de Azure AD. Esto significa que puede autenticarse con su Instancia administrada de SQL mediante el inicio de sesión de servidor de Azure AD en lugar de un usuario de base de datos independiente. Para obtener más información, consulte [Información general sobre la Instancia administrada de SQL](../managed-instance/sql-managed-instance-paas-overview.md#azure-active-directory-integration). Para conocer la sintaxis de creación de entidades de seguridad (inicios de sesión) de un servidor de Azure AD, consulte <a href="/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current">CREATE LOGIN</a>.

Sin embargo, el uso de la autenticación de Azure Active Directory con SQL Database y Azure Synapse requiere la utilización de usuarios de bases de datos independientes basados en una identidad de Azure AD. Un usuario de base de datos independiente no cuenta con ningún inicio de sesión en la base de datos maestra, y se asigna a una identidad en Azure AD que esta asociada a la base de datos. La identidad de Azure AD puede ser una cuenta de usuario individual o un grupo. Para más información sobre los usuarios de bases de datos independientes, vea [Usuarios de bases de datos independientes: cómo hacer que la base de datos sea portátil](https://msdn.microsoft.com/library/ff929188.aspx).

> [!NOTE]
> Los usuarios de base de datos (a excepción de los administradores) no se pueden crear mediante Azure Portal. Los roles de Azure no se propagan a la base de datos en SQL Database, SQL Managed Instance o Azure Synapse. Los roles de Azure se utilizan para administrar los recursos de Azure y no se aplican a los permisos de base de datos. Por ejemplo, el rol **Colaborador de SQL Server** no concede acceso para conectarse a la base de datos en SQL Database, la Instancia administrada de SQL o Azure Synapse. El permiso de acceso tiene que concederse directamente en la base de datos mediante instrucciones de Transact-SQL.

> [!WARNING]
> No se admiten caracteres especiales, como los dos puntos (`:`) o la "y" comercial (`&`), en los nombres de usuario de las instrucciones `CREATE LOGIN` y `CREATE USER` de T-SQL.

Para crear un usuario de base de datos independiente basada en Azure AD (que no sea el administrador del servidor que es el propietario de la base de datos), conéctese a la base de datos con una identidad de Azure AD, como un usuario con al menos el permiso **ALTER ANY USER** . Después, use la sintaxis de Transact-SQL siguiente:

```sql
CREATE USER <Azure_AD_principal_name> FROM EXTERNAL PROVIDER;
```

*Azure_AD_principal_name* puede ser el nombre principal de usuario de un usuario de Azure AD, o el nombre para mostrar de un grupo de Azure AD.

**Ejemplos:** para crear un usuario de base de datos independiente que represente un usuario de dominio administrado o federado de Azure AD:

```sql
CREATE USER [bob@contoso.com] FROM EXTERNAL PROVIDER;
CREATE USER [alice@fabrikam.onmicrosoft.com] FROM EXTERNAL PROVIDER;
```

Para crear un usuario de base de datos contenido que represente un grupo de dominio federado o de Azure AD, proporcione el nombre para mostrar de un grupo de seguridad:

```sql
CREATE USER [ICU Nurses] FROM EXTERNAL PROVIDER;
```

Para crear un usuario de base de datos independiente que represente a una aplicación que se conecta mediante un token de Azure AD:

```sql
CREATE USER [appName] FROM EXTERNAL PROVIDER;
```

> [!NOTE]
> Este comando requiere acceso de SQL a Azure AD ("proveedor externo") en nombre del usuario que ha iniciado sesión. A veces, surgen circunstancias que hacen que Azure AD devuelva una excepción a SQL. En estos casos, el usuario verá el error 33134 de SQL, que debe contener el mensaje de error específico de Azure AD. La mayor parte de las veces el error indica que se deniega el acceso, que el usuario debe inscribirse en MFA para acceder al recurso, o que el acceso entre las aplicaciones propias debe controlarse con autorización previa. En los dos primeros casos, el problema se debe normalmente a las directivas de acceso condicional que se establecen en el inquilino de Azure AD del usuario, ya que impiden que el usuario acceda al proveedor externo. Si se actualizan las directivas de la entidad de certificación para permitir el acceso a la aplicación "00000002-0000-0000-c000-000000000000" (identificador de aplicación de Graph API de Azure AD), se debería resolver el problema. En caso de que el error indique que el acceso entre las aplicaciones propias debe controlarse con autorización previa, el problema se debe a que el usuario ha iniciado sesión como entidad de servicio. El comando debe ejecutarse correctamente si lo ejecuta un usuario.

> [!TIP]
> No puede crear directamente un usuario a partir de una instancia de Azure Active Directory distinta a la que esté asociada a su suscripción de Azure. Sin embargo, se pueden agregar miembros de otras instancias de AD que sean usuarios importados en el Active Directory asociado (que se conocen como "usuarios externos") a un grupo de Active Directory del AD inquilino. Al crear un usuario de la base de datos independiente para ese grupo de AD, los usuarios del Active Directory externo pueden obtener acceso a SQL Database.

Para más información sobre la creación de usuarios de bases de datos independientes basados en identidades de Azure Active Directory, vea [CREAR USUARIO (Transact-SQL)](https://msdn.microsoft.com/library/ms173463.aspx).

> [!NOTE]
> Si se quita el administrador de Azure Active Directory del servidor, se impide que cualquier usuario de autenticación de Azure AD pueda conectarse al servidor. Si es necesario, un administrador de SQL Database puede quitar manualmente a usuarios de Azure AD no utilizados.

> [!NOTE]
> Si recibe la notificación **Tiempo de espera de conexión agotado**, es posible que deba establecer el parámetro `TransparentNetworkIPResolution` de la cadena de conexión en False. Para más información, consulte [Connection timeout issue with .NET Framework 4.6.1 – TransparentNetworkIPResolution](https://blogs.msdn.microsoft.com/dataaccesstechnologies/20../../connection-timeout-issue-with-net-framework-4-6-1-transparentnetworkipresolution/) (Problema de tiempo de espera de conexión agotado con .NET Framework 4.6.1: TransparentNetworkIPResolution).

Cuando se crea un usuario de base de datos, dicho usuario recibe el permiso **CONNECT** y puede conectarse a esa base de datos como un miembro con el rol **PUBLIC**. En un principio, los únicos permisos disponibles para el usuario son los permisos que se conceden al rol **PUBLIC** o cualquier otro permiso que se conceda a los grupos de Azure AD de los que sea miembro. Cuando se aprovisiona un usuario de base de datos de independiente basada en AD Azure, se pueden conceder permisos adicionales al usuario, del mismo modo que se conceden permisos a cualquier otro tipo de usuario. Normalmente, se conceden permisos a roles de base de datos y después se agregan usuarios a los roles. Para obtener más información, consulte [Conceptos básicos de los permisos de los motores de las bases de datos](https://social.technet.microsoft.com/wiki/contents/articles/4433.database-engine-permission-basics.aspx). Para obtener más información sobre los roles especiales de SQL Database, consulte [Administración de bases de datos e inicios de sesión en Azure SQL Database](logins-create-manage.md).
Una cuenta de usuario de dominio federado que se importa en un dominio administrado como un usuario externo debe usar la identidad del dominio administrado.

> [!NOTE]
> Los usuarios de Azure AD se marcan en los metadatos de la base de datos con el tipo E (EXTERNAL_USER) y en los grupos con el tipo X (EXTERNAL_GROUPS). Para obtener más información, consulte [sys.database_principals (Transact-SQL)](https://msdn.microsoft.com/library/ms187328.aspx).

## <a name="connect-to-the-database-using-ssms-or-ssdt"></a>Conexión a la base de datos mediante SSMS o SSDT  

Para confirmar que el administrador de Azure AD está correctamente configurado, conéctese a la base de datos **maestra** con la cuenta de administrador de Azure AD.
Para aprovisionar un usuario de base de datos independiente basada en un directorio de Azure AD (que no sea el administrador del servidor que es el propietario de la base de datos), conéctese a la base de datos con una identidad de Azure AD que tenga acceso a la base de datos.

> [!IMPORTANT]
> La autenticación de Azure Active Directory es compatible con [SQL Server 2016 Management Studio](https://msdn.microsoft.com/library/mt238290.aspx) y [SQL Server Data Tools](https://msdn.microsoft.com/library/mt204009.aspx) en Visual Studio 2015. La versión de agosto de 2016 de SSMS también incluye compatibilidad con la autenticación universal de Active Directory, que permite a los administradores requerir Multi-Factor Authentication con una llamada de teléfono, un mensaje de texto, tarjetas inteligentes con PIN o una notificación de aplicación móvil.

## <a name="using-an-azure-ad-identity-to-connect-using-ssms-or-ssdt"></a>Uso de una identidad de Azure AD para conectarse mediante SQL Server Management Studio o SQL Server Database Tools

En los procedimientos siguientes se muestra cómo puede conectarse a una instancia de SQL Database con una identidad de Azure AD mediante SQL Server Management Studio o SQL Server Database Tools.

### <a name="active-directory-integrated-authentication"></a>Autenticación integrada de Active Directory

Use este método si ha iniciado sesión en Windows con sus credenciales de Azure Active Directory desde un dominio federado, o bien desde un dominio administrado que esté configurado para el inicio de sesión único de conexión directa con autenticación de paso a través y hash de contraseña. Para más información, consulte [Inicio de sesión único de conexión directa de Azure Active Directory](../../active-directory/hybrid/how-to-connect-sso.md).

1. Inicie Management Studio o Data Tools y, en el cuadro de diálogo **Conectar con el servidor** (o **Conectarse al motor de base de datos**), en el cuadro **Autenticación**, seleccione **Azure Active Directory: integrado**. No se necesita ni se puede especificar una contraseña para la conexión, ya que aparecen las credenciales existentes.

   ![Selección de autenticación integrada de Active Directory][11]

2. Seleccione el botón **Opciones** y, en la página **Propiedades de conexión**, en el cuadro **Conectar con base de datos**, escriba el nombre de la base de datos de usuarios a la que quiere conectarse. Para obtener más información, consulte el artículo [Autenticación multifactor de Azure AD](authentication-mfa-ssms-overview.md#azure-ad-domain-name-or-tenant-id-parameter) sobre las diferencias entre las propiedades de conexión de SSMS 17.x y 18.x.

   ![Selección del nombre de la base de datos][13]

### <a name="active-directory-password-authentication"></a>Autenticación de contraseña de Active Directory

Use este método al conectarse con un nombre de entidad de seguridad de Azure AD mediante el dominio administrado de Azure AD. También puede utilizarlo para la cuenta federada sin acceso al dominio, por ejemplo, cuando se trabaja de forma remota.

Use este método para autenticarse en la base de datos de SQL Database o la Instancia administrada de SQL con usuarios de identidad solo en la nube de Azure AD, o aquellos que usan identidades híbridas de Azure AD. Este método admite a los usuarios que quieren usar sus credenciales de Windows, pero su máquina local no está unida al dominio (por ejemplo, mediante un acceso remoto). En este caso, un usuario de Windows puede indicar su cuenta de dominio y contraseña, y autenticarse en la base de datos de SQL Database, la Instancia administrada de SQL o Azure Synapse.

1. Inicie Management Studio o Data Tools y, en el cuadro de diálogo **Conectar con el servidor** (o **Conectarse al motor de base de datos**), en el cuadro **Autenticación**, seleccione **Azure Active Directory: contraseña**.

2. En el cuadro **Nombre de usuario**, escriba el nombre de usuario de Azure Active Directory con el formato **nombreDeUsuario\@dominio.com**. Los nombres de usuario deben corresponderse con una cuenta de Azure Active Directory o una cuenta de un dominio administrado o federado con Azure Active Directory.

3. En el cuadro **Contraseña**, escriba la contraseña de usuario de la cuenta de Azure Active Directory o de la cuenta de dominio administrado o federado.

    ![Selección de autenticación de contraseña de Active Directory][12]

4. Seleccione el botón **Opciones** y, en la página **Propiedades de conexión**, en el cuadro **Conectar con base de datos**, escriba el nombre de la base de datos de usuarios a la que quiere conectarse. (Consulte el gráfico que aparece en la opción anterior).

### <a name="active-directory-interactive-authentication"></a>Autenticación interactiva de Active Directory

Use este método para la autenticación interactiva con o sin Multi-Factor Authentication (MFA), con la contraseña solicitada de forma interactiva. Este método se puede usar para autenticarse en la base de datos de SQL Database, la Instancia administrada de SQL y Azure Synapse para usuarios de identidad solo en la nube de Azure AD, o aquellos que usan identidades híbridas de Azure AD.

Para obtener más información, consulte [Uso de la autenticación multifactor de Azure AD con SQL Database y Azure Synapse (compatibilidad de SSMS con MFA)](authentication-mfa-ssms-overview.md).

## <a name="using-an-azure-ad-identity-to-connect-from-a-client-application"></a>Uso de una identidad de Azure AD para conectarse desde una aplicación cliente

En los procedimientos siguientes se muestra cómo puede conectarse a una instancia de SQL Database con una identidad de Azure AD desde una aplicación cliente.

### <a name="active-directory-integrated-authentication"></a>Autenticación integrada de Active Directory

Para usar la autenticación integrada de Windows, la instancia de Active Directory del dominio debe estar federada con Azure Active Directory, o debe ser un dominio administrado que esté configurado para el inicio de sesión único de conexión directa para la autenticación de paso a través o de hash de contraseña. Para más información, consulte [Inicio de sesión único de conexión directa de Azure Active Directory](../../active-directory/hybrid/how-to-connect-sso.md).

> [!NOTE]
> [MSAL.NET (Microsoft.Identity.Client)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki#roadmap) para autenticación integrada de Windows no es compatible con el inicio de sesión único de conexión directa con autenticación de paso a través y hash de contraseña.

La aplicación cliente (o un servicio) que se conecte a la base de datos debe ejecutarse en un equipo unido a un dominio con las credenciales de dominio de un usuario.

Para conectarse a una base de datos mediante autenticación integrada y una identidad de Azure AD, la palabra clave Authentication de la cadena de conexión de la base de datos debe establecerse como `Active Directory Integrated`. En el siguiente ejemplo de código de C# se usa ADO.NET.

```csharp
string ConnectionString = @"Data Source=n9lxnyuzhv.database.windows.net; Authentication=Active Directory Integrated; Initial Catalog=testdb;";
SqlConnection conn = new SqlConnection(ConnectionString);
conn.Open();
```

No se admite la palabra clave `Integrated Security=True` de la cadena de conexión para conectarse a Azure SQL Database. Al crear una conexión ODBC tendrá que quitar espacios y establecer la autenticación en 'ActiveDirectoryIntegrated'.

### <a name="active-directory-password-authentication"></a>Autenticación de contraseña de Active Directory

Para conectarse a una base de datos mediante cuentas de usuario de identidad solo en la nube de Azure AD o las que usan identidades híbridas de Azure AD, la palabra clave Authentication debe establecerse en `Active Directory Password`. La cadena de conexión debe contener las palabras clave y los valores User ID/UID y Password/PWD. En el siguiente ejemplo de código de C# se usa ADO.NET.

```csharp
string ConnectionString =
@"Data Source=n9lxnyuzhv.database.windows.net; Authentication=Active Directory Password; Initial Catalog=testdb;  UID=bob@contoso.onmicrosoft.com; PWD=MyPassWord!";
SqlConnection conn = new SqlConnection(ConnectionString);
conn.Open();
```

Puede encontrar más información sobre los métodos de autenticación de Azure AD con los ejemplos de código de demostración que se encuentran disponibles en la [demostración de la autenticación de Azure AD de GitHub](https://github.com/Microsoft/sql-server-samples/tree/master/samples/features/security/azure-active-directory-auth).

## <a name="azure-ad-token"></a>Token de Azure AD

Este método de autenticación permite a los servicios de nivel intermedio obtener tokens de tipo [JSON Web Token (JWT)](../../active-directory/develop/id-tokens.md) para conectarse a la base de datos de SQL Database, la Instancia administrada de SQL o Azure Synapse mediante la obtención de un token de Azure AD. Este método habilita varios escenarios de aplicación, incluidas las identidades de servicio, las entidades de servicio y las aplicaciones que usan la autenticación basada en certificados. Para usar la autenticación de token de Azure AD tiene que realizar cuatro pasos básicos:

1. Registrar la aplicación con Azure Active Directory y obtener el identificador de cliente para el código.
2. Crear un usuario de base de datos que represente a la aplicación. (Completado anteriormente en el paso 6).
3. Crear un certificado en el equipo cliente que va ejecuta la aplicación.
4. Agregar el certificado como una clave para la aplicación.

Cadena de conexión de ejemplo:

```csharp
string ConnectionString =@"Data Source=n9lxnyuzhv.database.windows.net; Initial Catalog=testdb;"
SqlConnection conn = new SqlConnection(ConnectionString);
conn.AccessToken = "Your JWT token"
conn.Open();
```

Para más información, consulte el [blog de seguridad de SQL Server](https://blogs.msdn.microsoft.com/sqlsecurity/20../../token-based-authentication-support-for-azure-sql-db-using-azure-ad-auth/). Para obtener información acerca de cómo agregar un certificado, consulte [Introducción a la autenticación basada en certificados de Azure Active Directory](../../active-directory/authentication/active-directory-certificate-based-authentication-get-started.md).

### <a name="sqlcmd"></a>sqlcmd

Las instrucciones siguientes permiten conectar con la versión 13.1 de sqlcmd, que está disponible en el [Centro de descarga](https://www.microsoft.com/download/details.aspx?id=53591).

> [!NOTE]
> `sqlcmd` con el comando `-G` no funciona con identidades del sistema y requiere un inicio de sesión de entidad de seguridad de usuario.

```cmd
sqlcmd -S Target_DB_or_DW.testsrv.database.windows.net -G  
sqlcmd -S Target_DB_or_DW.testsrv.database.windows.net -U bob@contoso.com -P MyAADPassword -G -l 30
```

## <a name="troubleshoot-azure-ad-authentication"></a>Solución de problemas con la autenticación de Azure AD

Puede encontrar instrucciones para la solución de problemas con la autenticación de Azure AD en el siguiente blog: <https://techcommunity.microsoft.com/t5/azure-sql-database/troubleshooting-problems-related-to-azure-ad-authentication-with/ba-p/1062991>.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general de los inicios de sesión, usuarios, roles de base de datos y permisos de SQL Database, consulte [Inicios de sesión, usuarios, roles de base de datos y cuentas de usuario](logins-create-manage.md).
- Para más información acerca de las entidades de seguridad de bases de datos, consulte [Entidades de seguridad](https://msdn.microsoft.com/library/ms181127.aspx).
- Para más información acerca de los roles de base de datos, consulte [Roles de nivel de base de datos](https://msdn.microsoft.com/library/ms189121.aspx).
- Para más información general acerca de las reglas de firewall de SQL Database, consulte [Introducción a las reglas de firewall de Azure SQL Database](firewall-configure.md).
- Para más información acerca de cómo configurar un usuario invitado de Azure AD como administrador de Azure AD, consulte [Crear usuarios invitados de Azure AD y establecerlos como administradores de Azure AD](authentication-aad-guest-users.md).
- Para más información sobre el uso de entidades de servicio con Azure SQL, consulte [Creación de usuarios de Azure AD mediante aplicaciones de Azure AD](authentication-aad-service-principal-tutorial.md).

<!--Image references-->

[11]: ./media/authentication-aad-configure/active-directory-integrated.png
[12]: ./media/authentication-aad-configure/12connect-using-pw-auth2.png
[13]: ./media/authentication-aad-configure/13connect-to-db2.png
