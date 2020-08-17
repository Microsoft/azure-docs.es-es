---
title: 'PowerShell: Adición de una instancia administrada a un grupo de conmutación por error'
titleSuffix: Azure SQL Managed Instance
description: Script de ejemplo de Azure PowerShell para crear una instancia administrada, agregarla a un grupo de conmutación por error automática y probar la conmutación por error.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: high-availability
ms.custom: sqldbrb=1
ms.devlang: PowerShell
ms.topic: sample
author: MashaMSFT
ms.author: mathoma
ms.reviewer: carlrab
ms.date: 07/16/2019
ms.openlocfilehash: 5b8ca2f578881b3e8bd557fa5aaac618bcab1599
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87073444"
---
# <a name="use-powershell-to-add-a-managed-instance-to-a-failover-group"></a>Uso de PowerShell para agregar una instancia administrada a un grupo de conmutación por error 

[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqlmi.md)]

Este script de ejemplo de PowerShell crea dos instancias administradas, las agrega a un grupo de conmutación por error y, a continuación, prueba la conmutación por error de la instancia administrada principal en la instancia administrada secundaria. 

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]
[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de manera local, en este tutorial se requiere la versión 1.4.0 o posterior de Azure PowerShell. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="sample-scripts"></a>Muestras de scripts

[!code-powershell-interactive[main](../../../../powershell_scripts/sql-database/failover-groups/add-managed-instance-to-failover-group-az-ps.ps1 "Add managed instance to a failover group")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Use el siguiente comando para quitar el grupo de recursos y todos los recursos que tenga asociados. Tendrá que quitar el grupo de recursos dos veces. Al quitar el grupo de recursos la primera vez, se quitarán la instancia administrada y los clústeres virtuales, pero se producirá el mensaje de error `Remove-AzResourceGroup : Long running operation failed with status 'Conflict'`. Ejecute el comando Remove-AzResourceGroup una segunda vez para quitar los recursos residuales y el grupo de recursos.

```powershell
Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos de Azure.  |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Crea una red virtual.  |
| [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) | Agrega una configuración de subred a una red virtual. | 
| [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) | Obtiene una red virtual en un grupo de recursos. | 
| [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) | Obtiene una subred en una red virtual. | 
| [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) | Crea un grupo de seguridad de red. | 
| [New-AzRouteTable](/powershell/module/az.network/new-azroutetable) | Crea una tabla de rutas. |
| [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig) | Actualiza una configuración de subred para una red virtual.  |
| [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) | Actualiza una red virtual.  |
| [Get-AzNetworkSecurityGroup](/powershell/module/az.network/get-aznetworksecuritygroup) | Obtiene un grupo de seguridad de red. |
| [Add-AzNetworkSecurityRuleConfig](/powershell/module/az.network/add-aznetworksecurityruleconfig)| Agrega una configuración de regla de seguridad de red a un grupo de seguridad de red. |
| [Set-AzNetworkSecurityGroup](/powershell/module/az.network/set-aznetworksecuritygroup) | Actualiza un grupo de seguridad de red.  | 
| [Add-AzRouteConfig](/powershell/module/az.network/add-azrouteconfig) | Agrega una ruta a una tabla de rutas. |
| [Set-AzRouteTable](/powershell/module/az.network/set-azroutetable) | Actualiza una tabla de rutas.  |
| [New-AzSqlInstance](/powershell/module/az.sql/new-azsqlinstance) | Crea una instancia administrada.  |
| [Get-AzSqlInstance](/powershell/module/az.sql/get-azsqlinstance)| Devuelve información sobre Instancia administrada de Azure SQL. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Crea una dirección IP pública.  | 
| [New-AzVirtualNetworkGatewayIpConfig](/powershell/module/az.network/new-azvirtualnetworkgatewayipconfig) | Crea una configuración de IP para una puerta de enlace de Virtual Network. |
| [New-AzVirtualNetworkGateway](/powershell/module/az.network/new-azvirtualnetworkgateway) | Crea una puerta de enlace de Virtual Network. |
| [New-AzVirtualNetworkGatewayConnection](/powershell/module/az.network/new-azvirtualnetworkgatewayconnection) | Crea una conexión entre las dos puertas de enlace de redes virtuales.   |
| [New-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/new-azsqldatabaseinstancefailovergroup)| Crea un nuevo grupo de conmutación por error de la instancia administrada de Azure SQL.  |
| [Get-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/get-azsqldatabaseinstancefailovergroup) | Obtiene o enumera los grupos de conmutación por error de la instancia administrada de SQL.| 
| [Switch-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/switch-azsqldatabaseinstancefailovergroup) | Ejecuta una conmutación por error de un grupo de conmutación por error de la instancia administrada de SQL. | 
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Elimina un grupo de recursos. | 

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Puede encontrar más ejemplos de scripts de PowerShell para Instancia administrada de SQL en [Scripts de PowerShell de Instancia administrada de Azure SQL](../../database/powershell-script-content-guide.md).
