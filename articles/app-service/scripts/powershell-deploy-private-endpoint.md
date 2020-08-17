---
title: 'PowerShell: Implementación de un punto de conexión privado para una aplicación web con PowerShell'
description: Aprenda a usar PowerShell para implementar un punto de conexión privado para la aplicación web
author: ericgre
ms.assetid: e1cc08d5-91cf-49d7-8d0a-c0e7bd2046ac
ms.topic: sample
ms.date: 07/07/2020
ms.author: ericg
ms.service: app-service
ms.workload: web
ms.openlocfilehash: 97ded67b9f9daa2652b2740a21a4b7d6a0ac2bdd
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87068262"
---
# <a name="create-an-app-service-app-and-deploy-private-endpoint-using-powershell"></a>Creación de una aplicación de App Service e implementación de un punto de conexión privado mediante PowerShell

Este script de ejemplo crea una aplicación en App Service con sus recursos relacionados y, después, implementa un punto de conexión privado.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]


## <a name="sample-script"></a>Script de ejemplo

```azurepowershell-interactive
# Parameters
$sitename="mywebapp-$(Get-Random)"
$appserviceplanname="ASP-"+$sitename
$resourcegroupname="RG-"+$sitename
$VNetname="VNet-$(Get-Random)"
$location="francecentral"
$privateendpointsubnetname = "privateEndpointSubnet"

# Create a resource group.
New-AzResourceGroup -Name $resourcegroupname -Location $location

# Create an App Service plan in PremiumV2 tier.
$asp = New-AzAppServicePlan -Name $appserviceplanname `
-Location $location `
-ResourceGroupName $resourcegroupname `
-Tier PremiumV2 `
-NumberofWorkers 1 `
-WorkerSize Small

# Create a web app.
$webApp = New-AzWebApp -Name $sitename `
-Location $location `
-AppServicePlan $appserviceplanname `
-ResourceGroupName $resourcegroupname

# Create a Virtual Network with two subnets
$integrationsubnet = New-AzVirtualNetworkSubnetConfig -Name "integrationSubnet" `
-AddressPrefix "10.8.1.0/24"

$privateendpointsubnet = New-AzVirtualNetworkSubnetConfig -Name $privateendpointsubnetname `
-AddressPrefix "10.8.2.0/24" `
-PrivateEndpointNetworkPoliciesFlag Disabled

$virtualNetwork = New-AzVirtualNetwork -Name $VNetname `
-ResourceGroupName $resourcegroupname `
-Location $location -AddressPrefix "10.8.0.0/16" `
-Subnet $integrationsubnet,$privateendpointsubnet

# Configure the Private Endpoint
$privateEndPointConnection = New-AzPrivateLinkServiceConnection -Name "myPrivateEndpointconnection" `
-PrivateLinkServiceID $webApp.Id `
-GroupId sites

$subnet = $virtualNetwork | select -ExpandProperty subnets | Where-Object {$_.Name -eq $privateendpointsubnetname}

$privateEndpoint = New-AzPrivateEndpoint -Name "myPrivateEndpoint" `
-ResourceGroupName $resourcegroupname `
-Location $location `
-Subnet $subnet `
-PrivateLinkServiceConnection $privateEndPointConnection

# Configure the Private DNS zone
$dnsZone = New-AzPrivateDnsZone -Name "privatelink.azurewebsites.net" `
-ResourceGroupName $resourcegroupname

$dnsLink = New-AzPrivateDnsVirtualNetworkLink -Name "myDNSLink" `
-ResourceGroupName $resourcegroupname `
-ZoneName "privatelink.azurewebsites.net" `
-VirtualNetworkId $virtualNetwork.Id

$dnsConfig = New-AzPrivateDnsZoneConfig -Name "privatelink.azurewebsites.net" `
-PrivateDnsZoneId $dnsZone.ResourceId

$dnsZoneGroup = New-AzPrivateDnsZoneGroup -Name "myZoneGroup" `
-ResourceGroupName $resourcegroupname `
-PrivateEndpointName $privateEndpoint.Name `
-PrivateDnsZoneConfig $dnsConfig
```


## <a name="clean-up-deployment"></a>Limpieza de la implementación 

Después de ejecutar el script de ejemplo, se puede usar el comando siguiente para quitar el grupo de recursos, la aplicación web y todos los recursos relacionados.

```powershell
Remove-AzResourceGroup -Name myResourceGroup -Force
```


## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [New-AzAppServicePlan](/powershell/module/az.websites/new-azappserviceplan) | Crea un plan de App Service, |
| [New-AzWebApp](/powershell/module/az.websites/new-azwebapp) | Crea una aplicación web. |
| [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) | Crea una configuración de subred de una red virtual. |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Crea una red virtual. |
| [New-AzPrivateLinkServiceConnection](/powershell/module/az.network/new-azprivatelinkserviceconnection) | Crea una configuración de conexión del servicio Private Link. |
| [New-AzPrivateEndpoint](/powershell/module/az.network/new-azprivateendpoint) | Crea un punto de conexión privado. |
| [New-AzPrivateDnsZone](/powershell/module/az.privatedns/new-azprivatednszone) | Crea una nueva zona DNS privada. |
| [New-AzPrivateDnsVirtualNetworkLink](/powershell/module/az.privatedns/new-azprivatednsvirtualnetworklink) | Crea un nuevo vínculo de red virtual DNS privado. |
| [New-AzPrivateDnsZoneConfig](/powershell/module/az.network/new-azprivatednszoneconfig) | Crea la configuración de zona DNS del grupo de zonas DNS privadas. |
| [New-AzPrivateDnsZoneGroup](/powershell/module/az.network/new-azprivatednszonegroup) | Crea un grupo de zonas DNS privadas en el punto de conexión privado especificado. |

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).
- Puede encontrar más ejemplos de Azure PowerShell para Azure App Service Web Apps en los [ejemplos de PowerShell](../samples-powershell.md).
