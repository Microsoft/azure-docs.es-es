---
title: PowerShell con alias DNS en Azure SQL | Microsoft Docs
description: Cmdlets de PowerShell como New-AzSqlServerDNSAlias le permiten redirigir las nuevas conexiones de cliente a un servidor de base de datos de SQL de Azure diferentes, sin tener que tocar cualquier configuración de cliente.
keywords: dns sql database
services: sql-database
ms.service: sql-database
ms.subservice: operations
ms.devlang: PowerShell
ms.topic: conceptual
author: oslake
ms.author: moslake
ms.reviewer: genemi,amagarwa,maboja, jrasnick
manager: craigg
ms.date: 05/14/2019
ms.openlocfilehash: 4318e6557dc72dff7200beb8783575131659b77f
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65797701"
---
# <a name="powershell-for-dns-alias-to-azure-sql-database"></a>PowerShell con alias DNS para Azure SQL Database

En este artículo se proporciona un script de PowerShell que muestra cómo puede administrar un alias DNS para Azure SQL Database. El script ejecuta los siguientes cmdlets que realizan las siguientes acciones:

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Los cmdlets usados en el ejemplo de código son los siguientes:

- [New-AzSqlServerDNSAlias](https://docs.microsoft.com/powershell/module/az.Sql/New-azSqlServerDnsAlias): crea un nuevo alias DNS en el sistema del servicio Azure SQL Database. El alias hace referencia al servidor 1 de Azure SQL Database.
- [Get-AzSqlServerDNSAlias](https://docs.microsoft.com/powershell/module/az.Sql/Get-azSqlServerDnsAlias): obtiene y enumera todos los alias DNS asignados al servidor 1 de SQL DB.
- [Set-AzSqlServerDNSAlias](https://docs.microsoft.com/powershell/module/az.Sql/Set-azSqlServerDnsAlias): modifica el nombre del servidor con el que el alias está configurado para hacer referencia, de servidor 1 a servidor 2 de SQL DB.
- [Remove-AzSqlServerDNSAlias](https://docs.microsoft.com/powershell/module/az.Sql/Remove-azSqlServerDnsAlias): quita el alias DNS del servidor 2 de SQL DB mediante el nombre del alias.

## <a name="dns-alias-in-connection-string"></a>Alias DNS en la cadena de conexión

Para conectarse a un servidor determinado de Azure SQL Database, un cliente como SQL Server Management Studio (SSMS) puede proporcionar el nombre del alias DNS en lugar del nombre verdadero del servidor. En la siguiente cadena de servidor de ejemplo, el alias *any-unique-alias-name* reemplaza al primer nodo delimitado por puntos en la cadena de servidor de cuatro nodos:

- Ejemplo de cadena de servidor: `any-unique-alias-name.database.windows.net`.

## <a name="prerequisites"></a>Requisitos previos

Si quiere ejecutar el script de PowerShell dado en este artículo, se aplican los siguientes requisitos previos:

- Una suscripción y una cuenta de Azure. Para obtener una evaluación gratuita, haga clic en [https://azure.microsoft.com/free/][https://azure.microsoft.com/free/].
- Módulo de PowerShell de Azure, con el cmdlet **New AzSqlServerDNSAlias**.
  - Si necesita instalarlo o actualizarlo, consulte [Instalación del módulo de Azure PowerShell][install-Az-ps-84p].
  - Ejecute `Get-Module -ListAvailable Az;` en powershell\_ise.exe, para encontrar la versión.
- Dos servidores de Azure SQL Database.

## <a name="code-example"></a>Ejemplo de código

El siguiente ejemplo de código de PowerShell se inicia mediante la asignación de valores literales a varias variables. Para ejecutar el código, primero debe modificar todos los valores de marcador de posición para que coincidan con los valores reales en el sistema. O bien, simplemente puede estudiar el código. También se proporciona la salida de consola del código.

```powershell
################################################################
###    Assign prerequisites.                                 ###
################################################################
cls;

$SubscriptionName             = '<EDIT-your-subscription-name>';
[string]$SubscriptionGuid_Get = '?'; # The script assigns this value, not you.

$SqlServerDnsAliasName = '<EDIT-any-unique-alias-name>';

$1ResourceGroupName = '<EDIT-rg-1>';  # Can be same or different than $2ResourceGroupName.
$1SqlServerName     = '<EDIT-sql-1>'; # Must differ from $2SqlServerName.

$2ResourceGroupName = '<EDIT-rg-2>';
$2SqlServerName     = '<EDIT-sql-2>';

# Login to your Azure subscription, first time per session.
Write-Host "You must log into Azure once per powershell_ise.exe session,";
Write-Host "  thus type 'yes' only the first time.";
Write-Host " ";
$yesno = Read-Host '[yes/no]  Do you need to log into Azure now?';
if ('yes' -eq $yesno)
{
    Connect-AzAccount -SubscriptionName $SubscriptionName;
}

$SubscriptionGuid_Get = Get-AzSubscription `
    -SubscriptionName $SubscriptionName;

################################################################
###    Working with DNS aliasing for Azure SQL DB server.    ###
################################################################

Write-Host '[1] Assign a DNS alias to SQL DB server 1.';
New-AzSqlServerDNSAlias `
    –ResourceGroupName  $1ResourceGroupName `
    -ServerName         $1SqlServerName `
    -ServerDNSAliasName $SqlServerDnsAliasName;

Write-Host '[2] Get and list all the DNS aliases that are assigned to SQL DB server 1.';
Get-AzSqlServerDNSAlias `
    –ResourceGroupName $1ResourceGroupName `
    -ServerName        $1SqlServerName;

Write-Host '[3] Move the DNS alias from 1 to SQL DB server 2.';
Set-AzSqlServerDNSAlias `
    –ResourceGroupName  $2ResourceGroupName `
    -NewServerName      $2SqlServerName `
    -ServerDNSAliasName $SqlServerDnsAliasName `
    -OldServerResourceGroup  $1ResourceGroupName `
    -OldServerName           $1SqlServerName `
    -OldServerSubscriptionId $SubscriptionGuid_Get;

# Here your client, such as SSMS, can connect to your "$2SqlServerName"
# by using "$SqlServerDnsAliasName" in the server name.
# For example, server:  "any-unique-alias-name.database.windows.net".

# Remove-AzSqlServerDNSAlias  - would fail here for SQL DB server 1.

Write-Host '[4] Remove the DNS alias from SQL DB server 2.';
Remove-AzSqlServerDNSAlias `
    –ResourceGroupName  $2ResourceGroupName `
    -ServerName         $2SqlServerName `
    -ServerDNSAliasName $SqlServerDnsAliasName;
```

### <a name="actual-console-output-from-the-powershell-example"></a>Salida de consola real del ejemplo de PowerShell

La siguiente salida de consola se copió y pegó de una ejecución real.

```powershell
You must log into Azure once per powershell_ise.exe session,
  thus type 'yes' only the first time.

[yes/no]  Do you need to log into Azure now?: yes


Environment           : AzureCloud
Account               : gm@acorporation.com
TenantId              : 72f988bf-1111-1111-1111-111111111111
SubscriptionId        : 45651c69-2222-2222-2222-222222222222
SubscriptionName      : mysubscriptionname
CurrentStorageAccount :

[1] Assign a DNS alias to SQL DB server 1.
[2] Get the DNS alias that is assigned to SQL DB server 1.
[3] Move the DNS alias from 1 to SQL DB server 2.
[4] Remove the DNS alias from SQL DB server 2.
ResourceGroupName ServerName         ServerDNSAliasName
----------------- ----------         ------------------
gm-rg-dns-1       gm-sqldb-dns-1     unique-alias-name-food
gm-rg-dns-1       gm-sqldb-dns-1     unique-alias-name-food
gm-rg-dns-2       gm-sqldb-dns-2     unique-alias-name-food


[C:\windows\system32\]
>>
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener una explicación completa de la característica de alias DNS para SQL Database, consulte [Alias DNS para Azure SQL Database][dns-alias-overview-37v].

<!-- Article links. -->

[https://azure.microsoft.com/free/]: https://azure.microsoft.com/free/

[install-Az-ps-84p]: https://docs.microsoft.com/powershell/azure/install-az-ps

[dns-alias-overview-37v]: dns-alias-overview.md
