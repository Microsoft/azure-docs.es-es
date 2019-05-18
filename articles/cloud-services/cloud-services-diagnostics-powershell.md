---
title: Habilitar el diagnóstico en Azure Cloud Services mediante PowerShell | Microsoft Docs
description: Obtener información sobre cómo habilitar el diagnóstico en servicios en la nube mediante PowerShell
services: cloud-services
documentationcenter: .net
author: jpconnock
manager: timlt
editor: ''
ms.assetid: 66e08754-8639-4022-ae18-4237749ba17d
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 09/06/2016
ms.author: jeconnoc
ms.openlocfilehash: 13a855c5770281e2578523bfc1813b2e03df6651
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65539245"
---
# <a name="enable-diagnostics-in-azure-cloud-services-using-powershell"></a>Habilitar el diagnóstico en Azure Cloud Services mediante PowerShell
Puede recopilar datos de diagnóstico como registros de aplicaciones, contadores de rendimiento, etc., de un servicio en la nube mediante la extensión de Diagnósticos de Azure. En este artículo se describe cómo habilitar la extensión de Diagnósticos de Azure para un servicio en la nube mediante PowerShell.  Para conocer los requisitos previos necesarios para este artículo, consulte [Cómo instalar y configurar Azure PowerShell](/powershell/azure/overview) .

## <a name="enable-diagnostics-extension-as-part-of-deploying-a-cloud-service"></a>Habilitar la extensión de diagnósticos como parte de la implementación de un servicio en la nube
Este enfoque es aplicable para el tipo de integración continua de escenarios en los que se puede habilitar la extensión de diagnóstico como parte de la implementación del servicio en la nube. Al crear una nueva implementación de servicio en la nube, puede habilitar la extensión diagnostics pasando el *ExtensionConfiguration* parámetro para el [New-AzureDeployment](/powershell/module/servicemanagement/azure/new-azuredeployment?view=azuresmps-3.7.0) cmdlet. El parámetro *ExtensionConfiguration* toma una matriz de configuraciones de diagnóstico que se pueden crear mediante el cmdlet [New-AzureServiceDiagnosticsExtensionConfig](/powershell/module/servicemanagement/azure/new-azureservicediagnosticsextensionconfig?view=azuresmps-3.7.0) .

En el ejemplo siguiente se muestra cómo habilitar el diagnóstico de un servicio en la nube con un WebRole y WorkerRole, cada uno con una configuración de diagnóstico diferente.

```powershell
$service_name = "MyService"
$service_package = "CloudService.cspkg"
$service_config = "ServiceConfiguration.Cloud.cscfg"
$webrole_diagconfigpath = "MyService.WebRole.PubConfig.xml"
$workerrole_diagconfigpath = "MyService.WorkerRole.PubConfig.xml"

$webrole_diagconfig = New-AzureServiceDiagnosticsExtensionConfig -Role "WebRole" -DiagnosticsConfigurationPath $webrole_diagconfigpath
$workerrole_diagconfig = New-AzureServiceDiagnosticsExtensionConfig -Role "WorkerRole" -DiagnosticsConfigurationPath $workerrole_diagconfigpath

New-AzureDeployment -ServiceName $service_name -Slot Production -Package $service_package -Configuration $service_config -ExtensionConfiguration @($webrole_diagconfig,$workerrole_diagconfig)
```

Si el archivo de configuración de diagnóstico especifica un elemento `StorageAccount` con un nombre de cuenta de almacenamiento, el cmdlet `New-AzureServiceDiagnosticsExtensionConfig` usará automáticamente esa cuenta de almacenamiento. Para que esto funcione, la cuenta de almacenamiento debe estar en la misma suscripción que el servicio en la nube que se está implementando.

A partir de Azure SDK 2.6, los archivos de configuración de extensión generados por la salida de destino de publicación de MSBuild incluirán el nombre de la cuenta de almacenamiento en función de la cadena de configuración de diagnóstico que se haya especificado en el archivo de configuración de servicio (.cscfg). El script siguiente muestra cómo analizar los archivos de configuración de extensión desde la salida de destino de publicación y cómo configurar la extensión de diagnóstico de cada rol al implementar el servicio en la nube.

```powershell
$service_name = "MyService"
$service_package = "C:\build\output\CloudService.cspkg"
$service_config = "C:\build\output\ServiceConfiguration.Cloud.cscfg"

#Find the Extensions path based on service configuration file
$extensionsSearchPath = Join-Path -Path (Split-Path -Parent $service_config) -ChildPath "Extensions"

$diagnosticsExtensions = Get-ChildItem -Path $extensionsSearchPath -Filter "PaaSDiagnostics.*.PubConfig.xml"
$diagnosticsConfigurations = @()
foreach ($extPath in $diagnosticsExtensions)
{
    #Find the RoleName based on file naming convention PaaSDiagnostics.<RoleName>.PubConfig.xml
    $roleName = ""
    $roles = $extPath -split ".",0,"simplematch"
    if ($roles -is [system.array] -and $roles.Length -gt 1)
    {
        $roleName = $roles[1]
        $x = 2
        while ($x -le $roles.Length)
            {
               if ($roles[$x] -ne "PubConfig")
                {
                    $roleName = $roleName + "." + $roles[$x]
                }
                else
                {
                    break
                }
                $x++
            }
        $fullExtPath = Join-Path -path $extensionsSearchPath -ChildPath $extPath
        $diagnosticsconfig = New-AzureServiceDiagnosticsExtensionConfig -Role $roleName -DiagnosticsConfigurationPath $fullExtPath
        $diagnosticsConfigurations += $diagnosticsconfig
    }
}
New-AzureDeployment -ServiceName $service_name -Slot Production -Package $service_package -Configuration $service_config -ExtensionConfiguration $diagnosticsConfigurations
```

Visual Studio Online usa un enfoque similar para las implementaciones automatizadas de Cloud Services con la extensión de diagnóstico. Consulte [Publish-AzureCloudDeployment.ps1](https://github.com/Microsoft/azure-pipelines-tasks/blob/master/Tasks/AzureCloudPowerShellDeploymentV1/Publish-AzureCloudDeployment.ps1) para obtener un ejemplo completo.

Si no se ha especificado el parámetro `StorageAccount` en la configuración de diagnóstico, es necesario pasar el parámetro *StorageAccountName* al cmdlet. Si no se ha especificado el parámetro *StorageAccountName* , el cmdlet siempre usará la cuenta de almacenamiento especificada en el parámetro y no la especificada en el archivo de configuración de diagnóstico.

Si la cuenta de almacenamiento de diagnóstico está en una suscripción distinta que el servicio en la nube, es necesario pasar explícitamente los parámetros *StorageAccountName* y *StorageAccountKey* al cmdlet. El parámetro *StorageAccountKey* no es necesario cuando la cuenta de almacenamiento de diagnóstico está en la misma suscripción, ya que el cmdlet puede consultar automáticamente y establecer el valor de clave cuando se habilita la extensión de diagnóstico. Sin embargo, si la cuenta de almacenamiento de diagnóstico está en una suscripción diferente, es posible que el cmdlet no pueda obtener la clave automáticamente y que sea necesario especificar explícitamente la clave mediante el parámetro *StorageAccountKey* .

```powershell
$webrole_diagconfig = New-AzureServiceDiagnosticsExtensionConfig -Role "WebRole" -DiagnosticsConfigurationPath $webrole_diagconfigpath -StorageAccountName $diagnosticsstorage_name -StorageAccountKey $diagnosticsstorage_key
$workerrole_diagconfig = New-AzureServiceDiagnosticsExtensionConfig -Role "WorkerRole" -DiagnosticsConfigurationPath $workerrole_diagconfigpath -StorageAccountName $diagnosticsstorage_name -StorageAccountKey $diagnosticsstorage_key
```

## <a name="enable-diagnostics-extension-on-an-existing-cloud-service"></a>Habilitar la extensión de diagnóstico en un servicio en la nube existente
Puede usar el cmdlet [Set-AzureServiceDiagnosticsExtension](/powershell/module/servicemanagement/azure/set-azureservicediagnosticsextension?view=azuresmps-3.7.0) para habilitar o actualizar la configuración de diagnóstico en un servicio en la nube que ya se esté ejecutando.

[!INCLUDE [cloud-services-wad-warning](../../includes/cloud-services-wad-warning.md)]

```powershell
$service_name = "MyService"
$webrole_diagconfigpath = "MyService.WebRole.PubConfig.xml"
$workerrole_diagconfigpath = "MyService.WorkerRole.PubConfig.xml"

$webrole_diagconfig = New-AzureServiceDiagnosticsExtensionConfig -Role "WebRole" -DiagnosticsConfigurationPath $webrole_diagconfigpath
$workerrole_diagconfig = New-AzureServiceDiagnosticsExtensionConfig -Role "WorkerRole" -DiagnosticsConfigurationPath $workerrole_diagconfigpath

Set-AzureServiceDiagnosticsExtension -DiagnosticsConfiguration @($webrole_diagconfig,$workerrole_diagconfig) -ServiceName $service_name
```

## <a name="get-current-diagnostics-extension-configuration"></a>Obtener la configuración actual de la extensión de diagnósticos
Use el cmdlet [Get-AzureServiceDiagnosticsExtension](/powershell/module/servicemanagement/azure/get-azureservicediagnosticsextension?view=azuresmps-3.7.0) para obtener la configuración actual de diagnósticos para un servicio en la nube.

```powershell
Get-AzureServiceDiagnosticsExtension -ServiceName "MyService"
```

## <a name="remove-diagnostics-extension"></a>Eliminar la extensión de diagnósticos
Para desactivar los diagnósticos en un servicio en la nube, puede usar el [Remove-AzureServiceDiagnosticsExtension](/powershell/module/servicemanagement/azure/remove-azureservicediagnosticsextension?view=azuresmps-3.7.0) cmdlet.

```powershell
Remove-AzureServiceDiagnosticsExtension -ServiceName "MyService"
```

Si ha habilitado la extensión de diagnósticos mediante *Set-AzureServiceDiagnosticsExtension* o *New-AzureServiceDiagnosticsExtensionConfig* sin el *rol*parámetro, a continuación, se puede quitar la extensión mediante *Remove-AzureServiceDiagnosticsExtension* sin el *rol* parámetro. Si el *rol* parámetro se utiliza cuando se habilita la extensión y, después, también debe usarse al quitar la extensión.

Para quitar la extensión de diagnóstico de cada rol individual:

```powershell
Remove-AzureServiceDiagnosticsExtension -ServiceName "MyService" -Role "WebRole"
```

## <a name="next-steps"></a>Pasos siguientes
* Para obtener orientación adicional sobre el uso de diagnósticos de Azure y otras técnicas para solucionar problemas, consulte [Habilitación de diagnósticos en Azure Cloud Services y Virtual Machines](cloud-services-dotnet-diagnostics.md).
* En el [Esquema de configuración de diagnósticos](/azure/azure-monitor/platform/diagnostics-extension-schema-1dot2) se explican las distintas opciones de configuración xml para la extensión de diagnósticos.
* Para obtener información sobre cómo habilitar la extensión de diagnósticos para las máquinas virtuales, consulte [Crear una máquina virtual de Windows con supervisión y diagnóstico mediante la plantilla de Azure Resource Manager](../virtual-machines/windows/extensions-diagnostics-template.md)
