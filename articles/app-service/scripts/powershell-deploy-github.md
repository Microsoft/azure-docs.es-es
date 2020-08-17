---
title: 'PowerShell: Implementación de código desde GitHub'
description: Aprenda a usar Azure PowerShell para automatizar la implementación y administración de App Service. En este ejemplo se muestra cómo implementar el código desde GitHub.
tags: azure-service-management
ms.assetid: 0f9c8bc5-3789-4eb3-8deb-ae6e2200795a
ms.topic: sample
ms.date: 03/20/2017
ms.custom: mvc
ms.openlocfilehash: 28a022c21f102a7959ba7c317a435e828efd7d4e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87083283"
---
# <a name="create-a-web-app-and-deploy-code-from-github"></a>Creación de una aplicación web e implementación de código desde GitHub

Este script de ejemplo crea una aplicación web en App Service con sus recursos relacionados y, después, implementa el código de la aplicación web desde un repositorio de GitHub (sin una implementación continua). Para la implementación de GitHub con una implementación continua, consulte [Creación de una aplicación web con implementación continua desde GitHub](powershell-continuous-deployment-github.md).

Si es necesario, instale Azure PowerShell con la instrucción que se encuentra en la [Guía de Azure PowerShell](/powershell/azure/) y, luego, ejecute `Connect-AzAccount` para crear una conexión con Azure. Además, necesita un vínculo al repositorio de GitHub que contiene el código de aplicación web.

## <a name="sample-script"></a>Script de ejemplo

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!code-azurepowershell-interactive[main](../../../powershell_scripts/app-service/deploy-github/deploy-github.ps1?highlight=1-2 "Create a web app and deploy code from GitHub")]

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
| [Set-AzResource](/powershell/module/az.resources/set-azresource) | Modifica un recurso en un grupo de recursos. |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Puede encontrar ejemplos de Azure PowerShell para Azure App Service Web Apps en los [ejemplos de PowerShell](../samples-powershell.md).
