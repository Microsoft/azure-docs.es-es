---
title: Automatización de tareas de Azure Analysis Services con entidades de servicio | Microsoft Docs
description: Aprenda a crear entidades de servicio para automatizar las tareas de Azure Analysis Services.
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 04/23/2019
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: c034ed7164e67183b9a848d5210dcaf377476c6a
ms.sourcegitcommit: 17411cbf03c3fa3602e624e641099196769d718b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65518161"
---
# <a name="automation-with-service-principals"></a>Automatización con entidades de servicio

Las entidades de servicio son un recurso de aplicación de Azure Active Directory que se crean en el inquilino para realizar operaciones desatendidas de recursos y a nivel de servicio. Son un tipo único de *identidad de usuario* con un identificador de la aplicación y una contraseña o un certificado. Una entidad de servicio tiene los permisos necesarios para realizar tareas definidas por los roles y permisos para los que se asigna. 

En Analysis Services, las entidades de servicio se usan con Azure Automation, el modo desatendido de PowerShell, las aplicaciones cliente personalizadas y las aplicaciones web para automatizar tareas comunes. Por ejemplo, las tareas de aprovisionamiento de servidores, implementación de modelos, actualización de datos, escalado vertical y reducción vertical y pausa y reanudación pueden automatizarse con el uso de las entidades de servicio. Los permisos se asignan a las entidades de servicio con la pertenencia a roles, como las cuentas de UPN de Azure AD regulares.

Analysis Services también admite las operaciones realizadas por las identidades administradas mediante entidades de servicio. Para obtener más información, consulte [administra las identidades de los recursos de Azure](../active-directory/managed-identities-azure-resources/overview.md) y [que Azure AD admiten la autenticación de servicios de Azure](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-analysis-services).

## <a name="create-service-principals"></a>Creación de entidades de servicio
 
Las entidades de servicio se pueden crear en Azure Portal o con PowerShell. Para obtener más información, consulte:

[Creación de una entidad de servicio: Azure Portal](../active-directory/develop/howto-create-service-principal-portal.md)   
[Creación de una entidad de servicio: PowerShell](../active-directory/develop/howto-authenticate-service-principal-powershell.md)

## <a name="store-credential-and-certificate-assets-in-azure-automation"></a>Almacenamiento de recursos de credenciales y certificados en Azure Automation

Las credenciales y los certificados de las entidades de servicio se pueden almacenar de forma segura en Azure Automation para las operaciones de runbook. Para obtener más información, consulte:

[Recursos de credenciales en Azure Automation](../automation/automation-credentials.md)   
[Recursos de certificados en Azure Automation](../automation/automation-certificates.md)

## <a name="add-service-principals-to-server-admin-role"></a>Incorporación de entidades de servicio al rol de administrador del servidor

Antes de poder usar una entidad de servicio para las operaciones de administración del servidor de Analysis Services, debe agregarla al rol de administradores del servidor. Para más información, vea [Incorporación de una entidad de servicio al rol de administrador del servidor](analysis-services-addservprinc-admins.md).

## <a name="service-principals-in-connection-strings"></a>Entidades de servicio en las cadenas de conexión

El identificador de aplicación y la contraseña o el certificado de las entidades de servicio se pueden usar en las cadenas de conexión igual que un UPN.

### <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

#### <a name="a-nameazmodule-using-azanalysisservices-module"></a><a name="azmodule" />Uso de módulo Az.AnalysisServices

Cuando se usa una entidad de servicio para las operaciones de administración de recursos con el [Az.AnalysisServices](/powershell/module/az.analysisservices) módulo, use `Connect-AzAccount` cmdlet. 

En el ejemplo siguiente, se usan appID y una contraseña para realizar operaciones de plano de control para la sincronización de réplicas de solo lectura y escalar vertical u horizontal:

```powershell
Param (
        [Parameter(Mandatory=$true)] [String] $AppId,
        [Parameter(Mandatory=$true)] [String] $PlainPWord,
        [Parameter(Mandatory=$true)] [String] $TenantId
       )
$PWord = ConvertTo-SecureString -String $PlainPWord -AsPlainText -Force
$Credential = New-Object -TypeName "System.Management.Automation.PSCredential" -ArgumentList $AppId, $PWord

# Connect using Az module
Connect-AzAccount -Credential $Credential -SubscriptionId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxx"

# Syncronize a database for query scale out
Sync-AzAnalysisServicesInstance -Instance "asazure://westus.asazure.windows.net/testsvr" -Database "testdb"

# Scale up the server to an S1, set 2 read-only replicas, and remove the primary from the query pool. The new replicas will hydrate from the synchronized data.
Set-AzAnalysisServicesServer -Name "testsvr" -ResourceGroupName "testRG" -Sku "S1" -ReadonlyReplicaCount 2 -DefaultConnectionMode Readonly
```

#### <a name="using-sqlserver-module"></a>Usar el módulo SQLServer

En el ejemplo siguiente, se usa el identificador de aplicación y una contraseña para realizar una operación de actualización de la base de datos modelo.

```powershell
Param (
        [Parameter(Mandatory=$true)] [String] $AppId,
        [Parameter(Mandatory=$true)] [String] $PlainPWord,
        [Parameter(Mandatory=$true)] [String] $TenantId
       )
$PWord = ConvertTo-SecureString -String $PlainPWord -AsPlainText -Force

$Credential = New-Object -TypeName "System.Management.Automation.PSCredential" -ArgumentList $AppId, $PWord

Invoke-ProcessTable -Server "asazure://westcentralus.asazure.windows.net/myserver" -TableName "MyTable" -Database "MyDb" -RefreshType "Full" -ServicePrincipal -ApplicationId $AppId -TenantId $TenantId -Credential $Credential
```

### <a name="amo-and-adomd"></a>AMO y ADOMD 

Al conectarse con aplicaciones cliente y aplicaciones web, los paquetes instalables de la versión 15.0.2 y versiones posteriores de las [bibliotecas cliente de AMO y ADOMD](analysis-services-data-providers.md) de NuGet admiten entidades de servicio en las cadenas de conexión con el uso de la siguiente sintaxis: `app:AppID` y la contraseña o `cert:thumbprint`. 

En el ejemplo siguiente, se usan un `appID` y una `password` para realizar una operación de actualización de la base de datos modelo:

```C#
string appId = "xxx";
string authKey = "yyy";
string connString = $"Provider=MSOLAP;Data Source=asazure://westus.asazure.windows.net/<servername>;User ID=app:{appId};Password={authKey};";
Server server = new Server();
server.Connect(connString);
Database db = server.Databases.FindByName("adventureworks");
Table tbl = db.Model.Tables.Find("DimDate");
tbl.RequestRefresh(RefreshType.Full);
db.Model.SaveChanges();
```

## <a name="next-steps"></a>Pasos siguientes
[Inicio de sesión con Azure PowerShell](https://docs.microsoft.com/powershell/azure/authenticate-azureps)   
[Incorporación de una entidad de servicio al rol de administrador del servidor](analysis-services-addservprinc-admins.md)   
