---
title: Adición de una regla de grupo de seguridad de red en PowerShell
description: 'Ejemplo de script de Azure PowerShell: agrega un grupo de seguridad de red para permitir el tráfico entrante en un puerto específico.'
services: service-fabric
documentationcenter: ''
author: athinanthny
manager: chackdan
editor: ''
tags: azure-service-management
ms.assetid: ''
ms.service: service-fabric
ms.workload: multiple
ms.topic: sample
ms.date: 11/28/2017
ms.author: atsenthi
ms.custom: mvc
ms.openlocfilehash: 167fa74b04af7c086c2c95b0fdd56e5932483080
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87076212"
---
# <a name="add-an-inbound-network-security-group-rule"></a>Adición de una regla de grupo de seguridad de red entrante

Este script de ejemplo crea una regla de grupo de seguridad de red para permitir el tráfico entrante en el puerto 8081.  El script obtiene el grupo de seguridad de red, crea una nueva regla de configuración de seguridad de red y actualiza el grupo de seguridad de red. Personalice los parámetros según sea necesario.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Si es necesario, instale PowerShell con la instrucción que se encuentra en la [Guía de instalación de Azure PowerShell](/powershell/azure/). 

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell[main](../../../powershell_scripts/service-fabric/add-inbound-nsg-rule/add-inbound-nsg-rule.ps1 "Update the RDP port range values")]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [Get-AzResource](/powershell/module/az.resources/get-azresource) | Obtiene el recurso `Microsoft.Network/networkSecurityGroups`. |
|[Get-AzNetworkSecurityGroup](/powershell/module/az.network/get-aznetworksecuritygroup)| Obtiene el grupo de seguridad de red por nombre.|
|[Add-AzNetworkSecurityRuleConfig](/powershell/module/az.network/add-aznetworksecurityruleconfig)| Agrega una configuración de regla de seguridad de red a un grupo de seguridad de red. |
|[Set-AzNetworkSecurityGroup](/powershell/module/az.network/set-aznetworksecuritygroup)| Establece el estado de objetivo para un grupo de seguridad de red.|

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).
