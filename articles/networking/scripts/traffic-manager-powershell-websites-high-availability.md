---
title: 'Enrutamiento del tráfico para alta disponibilidad: Azure PowerShell'
description: 'Ejemplo de script de Azure PowerShell: enrutamiento del tráfico para la alta disponibilidad de las aplicaciones'
services: traffic-manager
documentationcenter: traffic-manager
author: asudbring
manager: KumudD
ms.service: traffic-manager
ms.devlang: powershell
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: traffic-manager
ms.date: 05/16/2017
ms.author: allensu
ms.openlocfilehash: 46d76a25018a68d43d11efcef162e5b4b6801572
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87030460"
---
# <a name="route-traffic-for-high-availability-of-applications---azure-powershell"></a>Enrutamiento del tráfico para la alta disponibilidad de las aplicaciones: Azure PowerShell

Este script crea un grupo de recursos, dos planes de App Service, dos aplicaciones web, un perfil de Traffic Manager y dos puntos de conexión de Traffic Manager. Traffic Manager dirige el tráfico a la aplicación en una región como la región primaria y a la región secundaria cuando no está disponible la aplicación en la región primaria. Antes de ejecutar el script, debe cambiar los valores de MyWebApp, MyWebAppL1 y MyWebAppL2 a valores únicos en todo Azure. Después de ejecutar el script, puede tener acceso a la aplicación en la región primaria con la dirección URL mywebapp.trafficmanager.net.

Si es necesario, instale Azure PowerShell con la instrucción que se encuentra en la [Guía de Azure PowerShell](https://docs.microsoft.com/powershell/azure/) y, luego, ejecute `Connect-AzAccount` para crear una conexión con Azure.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!code-powershell[main](../../../powershell_scripts/traffic-manager/direct-traffic-for-increased-application-availability/direct-traffic-for-increased-application-availability.ps1 "Route traffic for high availability")]


Ejecute el siguiente comando para quitar el grupo de recursos, la máquina virtual y todos los recursos relacionados.

```powershell
Remove-AzResourceGroup -Name myResourceGroup1
Remove-AzResourceGroup -Name myResourceGroup2
```


## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un grupo de recursos, una aplicación web, un perfil de Traffic Manager y todos los recursos relacionados. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup)  | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [New-AzAppServicePlan](/powershell/module/az.websites/new-azappserviceplan) | Crea un plan de App Service, que es como una granja de servidores para una aplicación web de Azure. |
| [New-AzWebApp](/powershell/module/az.websites/new-azwebapp) | Crea una aplicación web de Azure en el plan de App Service. |
| [Set-AzResource](/powershell/module/az.resources/new-azresource) | Crea una aplicación web de Azure en el plan de App Service. |
| [New-AzTrafficManagerProfile](/powershell/module/az.trafficmanager/new-aztrafficmanagerprofile) | Crea un perfil de Azure Traffic Manager. |
| [New-AzTrafficManagerEndpoint](/powershell/module/az.trafficmanager/new-aztrafficmanagerendpoint) | Agrega un punto de conexión a un perfil de Azure Traffic Manager. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](https://docs.microsoft.com/powershell/azure/).

En la [documentación de la información general de redes de Azure](../powershell-samples.md?toc=%2fazure%2fnetworking%2ftoc.json) puede encontrar ejemplos adicionales de script de PowerShell de redes.
