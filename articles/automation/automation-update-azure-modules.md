---
title: Actualización de módulos de Azure en Azure Automation
description: En este artículo se describe cómo puede actualizar ahora módulos comunes de Azure PowerShell proporcionados de forma predeterminada en Azure Automation.
services: automation
ms.service: automation
ms.subservice: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 05/13/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: a2f55891ddd383ea15da499495909b56ffb0e06d
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65786145"
---
# <a name="how-to-update-azure-powershell-modules-in-azure-automation"></a>Actualización de módulos de Azure PowerShell en Azure Automation

Para actualizar los módulos de Azure en su cuenta de Automation, deberá usar el [Update Azure modules runbook](https://github.com/Microsoft/AzureAutomation-Account-Modules-Update), que está disponible como código abierto. Para empezar a usar el runbook **Update-AutomationAzureModulesForAccount** para actualizar los módulos de Azure, descárguelo desde el [repositorio del runbook Actualizar módulos de Azure](https://github.com/Microsoft/AzureAutomation-Account-Modules-Update) en GitHub. A continuación, puede importarlo en su cuenta de Automation o ejecutarlo como un script. Las instrucciones sobre cómo hacer esto se encuentran en el [repositorio del runbook Actualizar módulos de Azure](https://github.com/Microsoft/AzureAutomation-Account-Modules-Update).

Los módulos de AzureRM PowerShell más comunes se proporcionan de forma predeterminada en cada cuenta de Automation. El equipo de Azure actualiza los módulos de Azure con regularidad, por lo tanto, para mantenerse al día querrá usar la [actualización AutomationAzureModulesForAccount](https://github.com/Microsoft/AzureAutomation-Account-Modules-Update) runbook para actualizar los módulos de sus cuentas de Automation.

Dado que los módulos se actualizan periódicamente por el grupo de productos, pueden producirse cambios con los cmdlets incluidos. Esta acción puede afectar negativamente los runbooks según el tipo de cambio, como cambiar el nombre de un parámetro o dejar de usar un cmdlet por completo.

Para evitar afectar los runbooks y los procesos que automatizan, pruébelos y valídelos antes de continuar. Si no tiene una cuenta de Automation dedicada pensada para este propósito, considere la posibilidad de crear una para que pueda probar muchos escenarios diferentes durante el desarrollo de los runbooks. Estas pruebas deben incluir cambios iterativos, como la actualización de los módulos de PowerShell.

Si desarrolla sus scripts de forma local, se recomienda tener localmente las mismas versiones del módulo que tiene en la cuenta de Automation al realizar pruebas, a fin de asegurarse de que obtendrá los mismos resultados. Una vez validados los resultados y aplicados los cambios necesarios, puede mover los cambios a producción.

> [!NOTE]
> Es posible que una cuenta de Automation nueva no contenga los módulos más recientes.

## <a name="considerations"></a>Consideraciones

Éstas son algunas consideraciones a tener en cuenta al usar este proceso para actualizar los módulos de Azure:

* Este runbook admite la actualización solo la **Azure** y **AzureRm** módulos actualmente. [Módulos de Azure PowerShell Az](/powershell/azure/new-azureps-module-az) se admiten en las cuentas de Automation, pero no se puede actualizar con este runbook. Hay factores importantes que debe tener en cuenta cuando se usa el `Az` módulos en su cuenta de automatización, para obtener más información, consulte [módulos Az Using en la cuenta de Automation](az-modules.md).

* Evite iniciar este runbook en cuentas de Automation que contengan módulos Az.

* Antes de iniciar este runbook, asegúrese de que la cuenta de Automation tiene una [credencial de cuenta de ejecución de Azure](manage-runas-account.md) creada.

* Puede utilizar este código como un script de PowerShell regular en lugar de un runbook: simplemente inicie sesión en Azure mediante el [Connect-AzureRmAccount](/powershell/module/azurerm.profile/connect-azurermaccount) comando primero y luego transmitir `-Login $false` a la secuencia de comandos.

* Para usar este runbook en las nubes soberanas, use el parámetro `AzureRmEnvironment` para pasar el entorno correcto al runbook.  Los valores permitidos son **AzureCloud**, **AzureChinaCloud**, **AzureGermanCloud** y **AzureUSGovernment**. Estos valores se pueden obtener si se usa `Get-AzureRmEnvironment | select Name`. Si no pasa un valor a este parámetro, el runbook de manera predeterminada será la nube pública de Azure **AzureCloud**.

* Si desea utilizar una versión específica del módulo Azure PowerShell en lugar de la última disponible en la Galería de PowerShell, pase estas versiones al parámetro opcional `ModuleVersionOverrides` del runbook **Update-AutomationAzureModulesForAccount**. Para obtener ejemplos, consulte el runbook [Update-AutomationAzureModulesForAccount.ps1](https://github.com/Microsoft/AzureAutomation-Account-Modules-Update/blob/master/Update-AutomationAzureModulesForAccount.ps1
). Los módulos de Azure PowerShell que no se mencionan en el parámetro `ModuleVersionOverrides` se actualizan con las últimas versiones de los módulos en la Galería de PowerShell. Si no se pasa nada al parámetro `ModuleVersionOverrides`, todos los módulos se actualizan con las versiones de módulo más recientes de la Galería de PowerShell. Se trata del mismo comportamiento que tiene el botón **Actualizar módulos de Azure**.

## <a name="known-issues"></a>Problemas conocidos

Hay un problema conocido con la actualización de los módulos de AzureRM de una cuenta de Automation que se encuentra en un grupo de recursos con un nombre numérico que empieza por 0. Para actualizar los módulos de Azure en su cuenta de Automation, debe ser en un grupo de recursos que tiene un nombre alfanumérico. Grupos de recursos con nombres numéricos empezando por 0 son no se puede actualizar los módulos de AzureRM en este momento.

## <a name="next-steps"></a>Pasos siguientes

Visite el [runbook Actualizar módulos de Azure](https://github.com/Microsoft/AzureAutomation-Account-Modules-Update) de código abierto para más información.
