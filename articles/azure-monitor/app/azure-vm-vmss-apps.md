---
title: Supervisión del rendimiento en las VM de Azure - Azure Application Insights
description: Supervisión del rendimiento de aplicaciones para máquinas virtuales de Azure y conjuntos de escalado de máquinas virtuales de Azure. Carga y tiempo de respuesta de gráfico, información de dependencia y establecer alertas en el rendimiento.
ms.topic: conceptual
ms.date: 08/26/2019
ms.openlocfilehash: 8b025c5196d65234a632bd1f939bc1116b72dce0
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87024639"
---
# <a name="deploy-the-azure-monitor-application-insights-agent-on-azure-virtual-machines-and-azure-virtual-machine-scale-sets"></a>Implementación de Azure Application Insights Agent en máquinas virtuales de Azure y conjuntos de escalado de máquinas virtuales de Azure

La habilitación de la supervisión en las aplicaciones web basadas en .NET que se ejecutan en [máquinas virtuales de Azure](https://azure.microsoft.com/services/virtual-machines/) y [conjuntos de escalado de máquinas virtuales](../../virtual-machine-scale-sets/index.yml) ahora es más fácil que nunca. Obtenga todas las ventajas de usar Application Insights sin modificar el código.

En este artículo se le guía a través de la habilitación de la supervisión de Application Insights mediante Application Insights Agent y se proporcionan instrucciones preliminares para automatizar el proceso para implementaciones a gran escala.

> [!IMPORTANT]
> Azure Application Insights Agent para .NET está actualmente en la versión preliminar pública.
> Esta versión preliminar se ofrece sin un Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="enable-application-insights"></a>Habilitación de Application Insights

Hay dos maneras de habilitar la supervisión de aplicaciones para las aplicaciones hospedadas en máquinas virtuales y conjuntos de escalado de máquinas virtuales de Azure:

* **Sin código** a través de Application Insights Agent
    * Este método es el más fácil de habilitar y no requiere ninguna configuración avanzada. A menudo se conoce como supervisión de "entorno en tiempo de ejecución".

    * En el caso de las máquinas virtuales de Azure y los conjuntos de escalado de máquinas virtuales de Azure, se recomienda habilitar, como mínimo, este nivel de supervisión. Después, en función del escenario, puede evaluar si la instrumentación manual es necesaria.

    * Application Insights Agent recopila automáticamente las mismas señales de dependencia que el SDK de NET. Consulte [Recopilación automática de dependencias](./auto-collect-dependencies.md#net) para más información.
        > [!NOTE]
        > En la actualidad, solo se admiten las aplicaciones .NET hospedadas en IIS. Use un SDK para instrumentar aplicaciones ASP.NET Core, Java y Node.js hospedadas en máquinas virtuales de Azure y conjuntos de escalado de máquinas virtuales.

* **Basado en código** mediante SDK

    * Este enfoque es mucho más personalizable, pero requiere la [incorporación de una dependencia en los paquetes NuGet del SDK de Application Insights](./asp-net.md). Este método también implica que el usuario tiene que administrar las actualizaciones a la versión más reciente de los paquetes.

    * Si necesita realizar llamadas de API personalizadas para supervisar eventos o dependencias no capturados de manera predeterminada con la supervisión basada en agentes, deberá usar este método. Consulte el [artículo API de Application Insights para eventos y métricas personalizados](./api-custom-events-metrics.md) para obtener más información.

> [!NOTE]
> Si se detecta tanto la supervisión basada en agentes como la instrumentación manual basada en SDK, solo se respetará la configuración de la instrumentación manual. Esto es para evitar que se envíen datos duplicados. Para más información sobre este tema, consulte la [sección Solución de problemas](#troubleshooting) a continuación.

## <a name="manage-application-insights-agent-for-net-applications-on-azure-virtual-machines-using-powershell"></a>Administración de Application Insights Agent para aplicaciones .NET en máquinas virtuales de Azure mediante PowerShell

> [!NOTE]
> Para instalar Application Insights Agent, necesitará una cadena de conexión. [Cree un nuevo recurso de Application Insights](./create-new-resource.md) o copie la cadena de conexión de un recurso de Application Insights existente.

> [!NOTE]
> ¿Es nuevo en PowerShell? Eche un vistazo a la [Guía de introducción](/powershell/azure/get-started-azureps?view=azps-2.5.0).

Instalación o actualización de Application Insights Agent como una extensión para las máquinas virtuales de Azure
```powershell
$publicCfgJsonString = '
{
  "redfieldConfiguration": {
    "instrumentationKeyMap": {
      "filters": [
        {
          "appFilter": ".*",
          "machineFilter": ".*",
          "virtualPathFilter": ".*",
          "instrumentationSettings" : {
            "connectionString": "InstrumentationKey=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
          }
        }
      ]
    }
  }
}
';
$privateCfgJsonString = '{}';

Set-AzVMExtension -ResourceGroupName "<myVmResourceGroup>" -VMName "<myVmName>" -Location "<myVmLocation>" -Name "ApplicationMonitoring" -Publisher "Microsoft.Azure.Diagnostics" -Type "ApplicationMonitoringWindows" -Version "2.8" -SettingString $publicCfgJsonString -ProtectedSettingString $privateCfgJsonString
```

> [!NOTE]
> Puede instalar o actualizar Application Insights Agent como una extensión en varias máquinas virtuales a escala mediante un bucle de PowerShell.

Desinstalación de la extensión de Agent de Application Insights desde una máquina virtual de Azure
```powershell
Remove-AzVMExtension -ResourceGroupName "<myVmResourceGroup>" -VMName "<myVmName>" -Name "ApplicationMonitoring"
```

Consulta del estado de la extensión Application Insights Agent para una máquina virtual de Azure
```powershell
Get-AzVMExtension -ResourceGroupName "<myVmResourceGroup>" -VMName "<myVmName>" -Name ApplicationMonitoring -Status
```

Obtención de la lista de extensiones instaladas para la máquina virtual de Azure
```powershell
Get-AzResource -ResourceId "/subscriptions/<mySubscriptionId>/resourceGroups/<myVmResourceGroup>/providers/Microsoft.Compute/virtualMachines/<myVmName>/extensions"

# Name              : ApplicationMonitoring
# ResourceGroupName : <myVmResourceGroup>
# ResourceType      : Microsoft.Compute/virtualMachines/extensions
# Location          : southcentralus
# ResourceId        : /subscriptions/<mySubscriptionId>/resourceGroups/<myVmResourceGroup>/providers/Microsoft.Compute/virtualMachines/<myVmName>/extensions/ApplicationMonitoring
```
También puede ver las extensiones instaladas en la [hoja de la máquina virtual de Azure](../../virtual-machines/extensions/overview.md) en el portal.

> [!NOTE]
> Para comprobar la instalación, haga clic en Live Metrics Stream en el recurso de Application Insights asociado a la cadena de conexión que usó para implementar la extensión de Application Insights Agent. Si va a enviar datos desde varias máquinas virtuales, seleccione las máquinas virtuales de Azure de destino en nombre del servidor. Los datos pueden tardar hasta un minuto en empezar a fluir.

## <a name="manage-application-insights-agent-for-net-applications-on-azure-virtual-machine-scale-sets-using-powershell"></a>Administración de Application Insights Agent para aplicaciones .NET en conjuntos de escalado de máquinas virtuales de Azure mediante PowerShell

Instalación o actualización de Application Insights Agent como una extensión para el conjunto de escalado de máquinas virtuales de Azure
```powershell
$publicCfgHashtable =
@{
  "redfieldConfiguration"= @{
    "instrumentationKeyMap"= @{
      "filters"= @(
        @{
          "appFilter"= ".*";
          "machineFilter"= ".*";
          "virtualPathFilter": ".*",
          "instrumentationSettings" : {
            "connectionString": "InstrumentationKey=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" # Application Insights connection string, create new Application Insights resource if you don't have one. https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/microsoft.insights%2Fcomponents
          }
        }
      )
    }
  }
};
$privateCfgHashtable = @{};

$vmss = Get-AzVmss -ResourceGroupName "<myResourceGroup>" -VMScaleSetName "<myVmssName>"

Add-AzVmssExtension -VirtualMachineScaleSet $vmss -Name "ApplicationMonitoring" -Publisher "Microsoft.Azure.Diagnostics" -Type "ApplicationMonitoringWindows" -TypeHandlerVersion "2.8" -Setting $publicCfgHashtable -ProtectedSetting $privateCfgHashtable

Update-AzVmss -ResourceGroupName $vmss.ResourceGroupName -Name $vmss.Name -VirtualMachineScaleSet $vmss

# Note: depending on your update policy, you might need to run Update-AzVmssInstance for each instance
```

Desinstalación de la extensión de supervisión de aplicaciones desde conjuntos de escalado de máquinas virtuales de Azure
```powershell
$vmss = Get-AzVmss -ResourceGroupName "<myResourceGroup>" -VMScaleSetName "<myVmssName>"

Remove-AzVmssExtension -VirtualMachineScaleSet $vmss -Name "ApplicationMonitoring"

Update-AzVmss -ResourceGroupName $vmss.ResourceGroupName -Name $vmss.Name -VirtualMachineScaleSet $vmss

# Note: depending on your update policy, you might need to run Update-AzVmssInstance for each instance
```

Estado de la extensión de supervisión de aplicaciones de consulta para conjuntos de escalado de máquinas virtuales de Azure
```powershell
# Not supported by extensions framework
```

Obtención de la lista de extensiones instaladas para conjuntos de escalado de máquinas virtuales de Azure
```powershell
Get-AzResource -ResourceId /subscriptions/<mySubscriptionId>/resourceGroups/<myResourceGroup>/providers/Microsoft.Compute/virtualMachineScaleSets/<myVmssName>/extensions

# Name              : ApplicationMonitoringWindows
# ResourceGroupName : <myResourceGroup>
# ResourceType      : Microsoft.Compute/virtualMachineScaleSets/extensions
# Location          :
# ResourceId        : /subscriptions/<mySubscriptionId>/resourceGroups/<myResourceGroup>/providers/Microsoft.Compute/virtualMachineScaleSets/<myVmssName>/extensions/ApplicationMonitoringWindows
```

## <a name="troubleshooting"></a>Solución de problemas

Busque sugerencias para la solución de problemas de la extensión de Agent de Application Insights para las aplicaciones .NET que se ejecutan en máquinas virtuales de Azure y en conjuntos de escalado de máquinas virtuales.

> [!NOTE]
> Las aplicaciones .NET Core, Java y Node.js solo se admiten en máquinas virtuales y conjuntos de escalado de máquinas virtuales de Azure a través de la instrumentación manual basada en SDK y, por tanto, los pasos siguientes no se aplican a estos escenarios.

El resultado de la ejecución de las extensiones se registra en los archivos que se encuentran en los siguientes directorios:
```Windows
C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.ApplicationMonitoringWindows\<version>\
```

## <a name="next-steps"></a>Pasos siguientes
* Obtenga información sobre cómo [implementar una aplicación en un conjunto de escalado de máquinas virtuales de Azure](../../virtual-machine-scale-sets/virtual-machine-scale-sets-deploy-app.md).
* [Configure pruebas web de disponibilidad](monitor-web-app-availability.md) para recibir una alerta si el punto de conexión deja de estar activo.
