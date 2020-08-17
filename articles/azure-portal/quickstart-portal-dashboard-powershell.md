---
title: Creación de un panel en Azure Portal con PowerShell
description: Aprenda a crear un panel en Azure Portal mediante Azure PowerShell.
author: mgblythe
ms.service: azure-portal
ms.topic: quickstart
ms.custom: devx-track-azurepowershell
ms.author: mblythe
ms.date: 07/24/2020
ms.openlocfilehash: 6b7a4f6d4ad7f5e94d19b9d531992f54ff13fec0
ms.sourcegitcommit: 14bf4129a73de2b51a575c3a0a7a3b9c86387b2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87440728"
---
# <a name="quickstart-create-an-azure-portal-dashboard-with-powershell"></a>Inicio rápido: Creación de un panel en Azure Portal con PowerShell

En Azure Portal, un panel es una vista organizada de los recursos en la nube. Este artículo se centra en el proceso de utilizar el módulo Az.Portal de PowerShell para crear un panel.
El panel muestra el rendimiento de una máquina virtual y algunos vínculos e información estática.

## <a name="requirements"></a>Requisitos

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

Si decide usar PowerShell de forma local, para este artículo es preciso que instale el módulo Az PowerShell y que se conecte a su cuenta de Azure con el cmdlet [Connect-AzAccount](https://docs.microsoft.com/powershell/module/az.accounts/connect-azaccount). Para más información sobre cómo instalar el módulo Az PowerShell, consulte [Instalación de Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps).

> [!IMPORTANT]
> Aunque el módulo **Az.Portal** de PowerShell esté en versión, se debe instalar por separado desde el módulo Az PowerShell mediante el cmdlet `Install-Module`. Una vez que este módulo de PowerShell esté disponible con carácter general, formará parte de las futuras versiones del módulo Az PowerShell y estará disponible de forma nativa en Azure Cloud Shell.

```azurepowershell-interactive
Install-Module -Name Az.Portal
```

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

## <a name="choose-a-specific-azure-subscription"></a>Elección de una suscripción de Azure específica

Si tiene varias suscripciones a Azure, elija la suscripción adecuada en la que se debe facturar el recurso. Seleccione una suscripción específica con el cmdlet [Set-AzContext](https://docs.microsoft.com/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

## <a name="define-variables"></a>Definición de variables

Usará varias partes de la información de forma repetida. Cree variables para almacenar la información.

```azurepowershell-interactive
# Name of resource group used throughout this article
$resourceGroupName = 'myResourceGroup'

# Azure region
$location = 'centralus'

# Dashboard Title
$dashboardTitle = 'Simple VM Dashboard'

# Dashboard Name
$dashboardName = $dashboardTitle -replace '\s'

# Your Azure Subscription ID
$subscriptionID = (Get-AzContext).Subscription.Id

# Name of test VM
$vmName = 'SimpleWinVM'
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un [grupo de recursos de Azure](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) con el cmdlet [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup). Un grupo de recursos es un contenedor lógico en el que se implementan y se administran recursos de Azure como un grupo.

En el ejemplo siguiente se crea un grupo de recursos basado en el nombre de la variable `$resourceGroupName` en la región especificada en la variable `$location`.

```azurepowershell-interactive
New-AzResourceGroup -Name $resourceGroupName -Location $location
```

## <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

El panel que se crea en la siguiente parte de este inicio rápido requiere que haya una máquina virtual. Cree una máquina virtual siguiendo estos pasos.

Almacene las credenciales de inicio de sesión de la máquina virtual en una variable. La contraseña debe ser compleja. Se trata de un nombre de usuario y una contraseña nuevos; no es, por ejemplo, la cuenta que usa para iniciar sesión en Azure. Para más información, consulte los [requisitos de nombre de usuario](../virtual-machines/windows/faq.md#what-are-the-username-requirements-when-creating-a-vm) y los [requisitos de contraseña](../virtual-machines/windows/faq.md#what-are-the-password-requirements-when-creating-a-vm).

```azurepowershell-interactive
$Cred = Get-Credential
```

Cree la máquina virtual.

```azurepowershell-interactive
$AzVmParams = @{
  ResourceGroupName = $resourceGroupName
  Name = $vmName
  Location = $location
  Credential = $Cred
}
New-AzVm @AzVmParams
```

Ahora se inicia la implementación de la máquina virtual y normalmente tarda unos minutos en completarse. Una vez finalizada la implementación, pase a la sección siguiente.

## <a name="download-the-dashboard-template"></a>Descarga de la plantilla del panel

Como los paneles de Azure son recursos, se pueden representar como JSON. El código siguiente descarga una representación JSON de un panel de ejemplo. Para más información, consulte [La estructura de los paneles de Azure](/azure/azure-portal/azure-portal-dashboards-structure).

```azurepowershell-interactive
$myPortalDashboardTemplateUrl = 'https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/azure-portal/portal-dashboard-template-testvm.json'

$myPortalDashboardTemplatePath = "$env:TEMP\portal-dashboard-template-testvm.json"

Invoke-WebRequest -Uri $myPortalDashboardTemplateUrl -OutFile $myPortalDashboardTemplatePath -UseBasicParsing
```

## <a name="customize-the-template"></a>Personalización de la plantilla

Para personalizar la plantilla descargada ejecute el código siguiente.

```azurepowershell
$Content = Get-Content -Path $myPortalDashboardTemplatePath -Raw
$Content = $Content -replace '<subscriptionID>', $subscriptionID
$Content = $Content -replace '<rgName>', $resourceGroupName
$Content = $Content -replace '<vmName>', $vmName
$Content = $Content -replace '<dashboardTitle>', $dashboardTitle
$Content = $Content -replace '<location>', $location
$Content | Out-File -FilePath $myPortalDashboardTemplatePath -Force
```

Para más información, consulte el artículo sobre la [referencia de las plantillas de paneles del portal de Microsoft](/azure/templates/microsoft.portal/dashboards).

## <a name="deploy-the-dashboard-template"></a>Implementación de la plantilla del panel

Puede usar el cmdlet `New-AzPortalDashboard` que forma parte del Módulo Az.Portal para implementar la plantilla directamente desde PowerShell.

```azurepowershell
$DashboardParams = @{
  DashboardPath = $myPortalDashboardTemplatePath
  ResourceGroupName = $resourceGroupName
  DashboardName = $dashboardName
}
New-AzPortalDashboard @DashboardParams
```

## <a name="review-the-deployed-resources"></a>Examen de los recursos implementados

Compruebe que el panel se ha creado correctamente.

```azurepowershell
Get-AzPortalDashboard -Name $dashboardName -ResourceGroupName $resourceGroupName
```

Compruebe que puede ver los datos de la máquina virtual desde Azure Portal.

1. En Azure Portal, seleccione **Panel**.

   ![Navegación al panel en Azure Portal](media/quickstart-portal-dashboard-powershell/navigate-to-dashboards.png)

1. En la página del panel, seleccione **Simple VM Dashboard**.

   ![Navegación al panel Simple VM Dashboard](media/quickstart-portal-dashboard-powershell/select-simple-vm-dashboard.png)

1. Examine el panel. Puede ver que una parte de su contenido es estático, pero que también hay gráficos que muestran el rendimiento de la máquina virtual.

   ![Revisión del panel Simple VM Dashboard](media/quickstart-portal-dashboard-powershell/review-simple-vm-dashboard.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Para quitar la máquina virtual y el panel asociado, elimine el grupo de recursos que los contiene.

> [!CAUTION]
> En el ejemplo siguiente se elimina el grupo de recursos especificado y todos los recursos que contiene.
> Si los recursos que están fuera del ámbito de este artículo existen en el grupo de recursos especificado, también se eliminarán.

```azurepowershell-interactive
Remove-AzResourceGroup -Name $resourceGroupName
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre los cmdlets del módulo Az.Portal de PowerShell, consulte:

> [!div class="nextstepaction"]
> [Microsoft Azure PowerShell: cmdlets de los paneles del portal](https://docs.microsoft.com/powershell/module/Az.Portal/)
