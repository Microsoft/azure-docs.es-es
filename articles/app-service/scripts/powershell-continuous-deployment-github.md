---
title: 'PowerShell: Implementación continua desde GitHub'
description: Aprenda a usar Azure PowerShell para automatizar la implementación y administración de App Service. En este ejemplo se muestra cómo crear una aplicación con CI/CD desde GitHub.
tags: azure-service-management
ms.assetid: 42f901f8-02f7-4869-b22d-d99ef59f874c
ms.topic: sample
ms.date: 03/20/2017
ms.custom: mvc
ms.openlocfilehash: e3193180165ec63f3716a9b875a8244ec7288eb8
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87083300"
---
# <a name="create-a-web-app-with-continuous-deployment-from-github"></a>Creación de una aplicación web con implementación continua desde GitHub

Este script de ejemplo crea una aplicación web en App Service con sus recursos relacionados y, después, configura la [implementación continua](../deploy-continuous-deployment.md) desde un repositorio de GitHub. Para la implementación de GitHub sin una implementación continua, consulte [Creación de una aplicación web e implementación de código desde GitHub](powershell-deploy-github.md).

Si es necesario, instale Azure PowerShell con la instrucción que se encuentra en la [Guía de Azure PowerShell](/powershell/azure/) y, luego, ejecute `Connect-AzAccount` para crear una conexión con Azure. Asegúrese de lo siguiente:

- El código de la aplicación está en un repositorio de GitHub público o privado del que usted es el propietario. Para obtener compilaciones automáticas, estructure el repositorio según la tabla que encontrará en la sección [Preparación del repositorio](../deploy-continuous-deployment.md#prepare-your-repository).
- Ha [creado un token de acceso personal en su cuenta de GitHub](https://help.github.com/en/articles/creating-a-personal-access-token-for-the-command-line).

## <a name="sample-script"></a>Script de ejemplo

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!code-azurepowershell-interactive[main](../../../powershell_scripts/app-service/deploy-github-continuous/deploy-github-continuous.ps1?highlight=1-2 "Create a web app with continuous deployment from GitHub")]

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
