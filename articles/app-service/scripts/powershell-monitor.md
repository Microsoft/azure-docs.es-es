---
title: 'PowerShell: Supervisión con los registros del servidor web'
description: Aprenda a usar Azure PowerShell para automatizar la implementación y administración de App Service. En este ejemplo se muestra cómo supervisar una aplicación con registros de servidor web.
tags: azure-service-management
ms.assetid: 5805d7cd-9e56-4eba-bd85-75b013690ff5
ms.topic: sample
ms.date: 03/20/2017
ms.custom: mvc
ms.openlocfilehash: dbe1e0decf48e8300010d531998fa1fbafccf61b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87033962"
---
# <a name="monitor-a-web-appwith-web-server-logs"></a>Supervisión de una aplicación web con registros de servidor web

En este escenario, creará un grupo de recursos, un plan de App Service y una aplicación web, además de configurar la aplicación web para habilitar los registros del servidor web. A continuación, descargará los archivos de registro para revisarlos.

Si es necesario, instale Azure PowerShell con la instrucción que se encuentra en la [Guía de Azure PowerShell](/powershell/azure/) y, luego, ejecute `Connect-AzAccount` para crear una conexión con Azure.

## <a name="sample-script"></a>Script de ejemplo

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!code-azurepowershell-interactive[main](../../../powershell_scripts/app-service/monitor-with-logs/monitor-with-logs.ps1 "Monitor a web app with web server logs")]

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
| [Set-AzWebApp](/powershell/module/az.websites/set-azwebapp) | Modifica la configuración de una aplicación web. |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Puede encontrar ejemplos de Azure PowerShell para Azure App Service Web Apps en los [ejemplos de PowerShell](../samples-powershell.md).
