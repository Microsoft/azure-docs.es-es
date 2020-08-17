---
title: Uso de Azure PowerShell para habilitar los diagnósticos en una máquina virtual Windows
services: virtual-machines-windows
documentationcenter: ''
description: Aprenda a usar PowerShell para habilitar Diagnósticos de Azure en una máquina virtual con Windows
author: mimckitt
manager: gwallace
editor: ''
ms.assetid: 2e6d88f2-1980-4a24-827e-a81616a0d247
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: article
ms.date: 12/15/2015
ms.author: mimckitt
ms.openlocfilehash: 3b46ae07d4bbbcd02b9880a394b72c790553f95c
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87837030"
---
# <a name="use-powershell-to-enable-azure-diagnostics-in-a-virtual-machine-running-windows"></a>Uso de PowerShell para habilitar Diagnósticos de Azure en una máquina virtual con Windows

Diagnósticos de Azure es la funcionalidad de Azure que habilita la recopilación de datos de diagnóstico en una aplicación implementada. Puede usar la extensión de diagnósticos para recopilar datos de diagnóstico como registros de aplicaciones o contadores de rendimiento de una máquina virtual (VM) de Azure con Windows. 

 

## <a name="enable-the-diagnostics-extension-if-you-use-the-resource-manager-deployment-model"></a>Habilite la extensión de diagnósticos si usa el modelo de implementación del Administrador de recursos
Puede habilitar la extensión de diagnósticos al crear una VM de Windows a través del modelo de implementación del Administrador de recursos de Azure añadiendo la configuración de extensiones a la plantilla del Administrador de recursos. Vea [Creación de una máquina virtual de Windows con supervisión y diagnóstico mediante la plantilla de Azure Resource Manager](diagnostics-template.md).

Para habilitar la extensión de diagnósticos en una VM existente creada mediante el modelo de implementación de Resource Manager, puede usar el cmdlet de PowerShell [Set-AzVMDiagnosticsExtension](/powershell/module/az.compute/set-azvmdiagnosticsextension) tal como se muestra a continuación.

```azurepowershell
$vm_resourcegroup = "myvmresourcegroup"
$vm_name = "myvm"
$diagnosticsconfig_path = "DiagnosticsPubConfig.xml"

Set-AzVMDiagnosticsExtension -ResourceGroupName $vm_resourcegroup -VMName $vm_name -DiagnosticsConfigurationPath $diagnosticsconfig_path
```


*$diagnosticsconfig_path* es la ruta de acceso del archivo que contiene la configuración de diagnóstico en XML como se describe en el [ejemplo](#sample-diagnostics-configuration) a continuación.  

Si especifica el archivo de configuración de diagnósticos de un elemento **StorageAccount** con un nombre de cuenta de almacenamiento, el script *Set-AzVMDiagnosticsExtension* establecerá automáticamente la extensión de diagnósticos para enviar datos de diagnóstico a esa cuenta de almacenamiento. Para que esto funcione, la cuenta de almacenamiento necesita estar en la misma suscripción que la VM.

Si no se ha especificado el parámetro **StorageAccount** en la configuración de diagnóstico, es necesario pasar el parámetro *StorageAccountName* al cmdlet. Si no se ha especificado el parámetro *StorageAccountName* , el cmdlet siempre usará la cuenta de almacenamiento especificada en el parámetro y no la especificada en el archivo de configuración de diagnóstico.

Si la cuenta de almacenamiento de diagnóstico está en una suscripción distinta que la VM, es necesario pasar explícitamente los parámetros *StorageAccountName* y *StorageAccountKey* al cmdlet. El parámetro *StorageAccountKey* no es necesario cuando la cuenta de almacenamiento de diagnóstico está en la misma suscripción, ya que el cmdlet puede consultar automáticamente y establecer el valor de clave cuando se habilita la extensión de diagnóstico. Sin embargo, si la cuenta de almacenamiento de diagnóstico está en una suscripción diferente, es posible que el cmdlet no pueda obtener la clave automáticamente y que sea necesario especificar explícitamente la clave mediante el parámetro *StorageAccountKey* .  

```azurepowershell
Set-AzVMDiagnosticsExtension -ResourceGroupName $vm_resourcegroup -VMName $vm_name -DiagnosticsConfigurationPath $diagnosticsconfig_path -StorageAccountName $diagnosticsstorage_name -StorageAccountKey $diagnosticsstorage_key
```

Una vez habilitada la extensión de diagnósticos en una VM, puede obtener la configuración actual mediante el cmdlet [Get-AzmDiagnosticsExtension Get](/powershell/module/az.compute/get-azvmdiagnosticsextension).

```azurepowershell
Get-AzVMDiagnosticsExtension -ResourceGroupName $vm_resourcegroup -VMName $vm_name
```

El cmdlet devuelve *PublicSettings*, que contiene la configuración de diagnóstico. Hay dos tipos de configuraciones compatibles: WadCfg y xmlCfg. WadCfg es la configuración JSON y xmlCfg es la configuración XML en formato con codificación Base64. Para leer el archivo XML, es necesario descodificarlo.

```azurepowershell
$publicsettings = (Get-AzVMDiagnosticsExtension -ResourceGroupName $vm_resourcegroup -VMName $vm_name).PublicSettings
$encodedconfig = (ConvertFrom-Json -InputObject $publicsettings).xmlCfg
$xmlconfig = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($encodedconfig))
Write-Host $xmlconfig
```

El cmdlet [Remove-AzVmDiagnosticsExtension](/powershell/module/az.compute/remove-azvmdiagnosticsextension) puede usarse para quitar la extensión de diagnósticos de la VM.  

## <a name="enable-the-diagnostics-extension-if-you-use-the-classic-deployment-model"></a>Habilite la extensión de diagnósticos si usa el modelo de implementación clásico

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

Puede usar el cmdlet [Set-AzureVMDiagnosticsExtension](/powershell/module/servicemanagement/azure.service/set-azurevmdiagnosticsextension) para habilitar la extensión de diagnósticos en una VM creada mediante el modelo de implementación clásica. En el ejemplo siguiente se muestra cómo crear una nueva VM mediante el modelo de implementación clásica con la extensión de diagnósticos habilitada.

```azurepowershell
$VM = New-AzureVMConfig -Name $VM -InstanceSize Small -ImageName $VMImage
$VM = Add-AzureProvisioningConfig -VM $VM -AdminUsername $Username -Password $Password -Windows
$VM = Set-AzureVMDiagnosticsExtension -DiagnosticsConfigurationPath $Config_Path -VM $VM -StorageContext $Storage_Context
New-AzVM -Location $Location -ServiceName $Service_Name -VM $VM
```

Para habilitar la extensión de diagnósticos en una VM existente creada mediante el modelo de implementación clásica, use el cmdlet [Get-AzureVM](/powershell/module/servicemanagement/azure.service/get-azurevm) en primer lugar para obtener la configuración de VM. A continuación, actualice la configuración de VM para incluir la extensión de diagnósticos mediante el cmdlet [Set-AzureVMDiagnosticsExtension](/powershell/module/servicemanagement/azure.service/set-azurevmdiagnosticsextension) . Por último, aplique la configuración actualizada a la VM mediante [Update-AzureVM](/powershell/module/servicemanagement/azure.service/update-azurevm).

```azurepowershell
$VM = Get-AzureVM -ServiceName $Service_Name -Name $VM_Name
$VM_Update = Set-AzureVMDiagnosticsExtension  -DiagnosticsConfigurationPath $Config_Path -VM $VM -StorageContext $Storage_Context
Update-AzureVM -ServiceName $Service_Name -Name $VM_Name -VM $VM_Update.VM
```

## <a name="sample-diagnostics-configuration"></a>Configuración de diagnósticos de ejemplo
Se puede utilizar el siguiente XML para la configuración pública de diagnósticos con los scripts anteriores. Esta configuración de ejemplo transferirá diversos contadores de rendimiento a la cuenta de almacenamiento de información de diagnósticos junto con los errores de la aplicación, seguridad y canales del sistema en los registros de eventos de Windows y los errores de los registros de infraestructura de diagnóstico.

La configuración debe actualizarse para incluir lo siguiente:

* El atributo *resourceID* del elemento **Métricas** tiene que actualizarse con el identificador de recurso de la VM.
  
  * El identificador de recursos puede crearse mediante el siguiente patrón: "/subscriptions/{*Identificador de suscripción para la suscripción con la VM*}/resourceGroups/{*Nombre del grupo de recursos para la VM*}/providers/Microsoft.Compute/virtualMachines/{*Nombre de la VM*}".
  * Por ejemplo, si el identificador de suscripción para la suscripción donde se está ejecutando la VM es **11111111-1111-1111-1111-111111111111**, el nombre del grupo de recursos para el grupo de recursos es **MyResourceGroup** y el nombre de la VM es**MyWindowsVM**, el valor de *resourceID* sería:
    
      ```xml
      <Metrics resourceId="/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/virtualMachines/MyWindowsVM" >
      ```
  * Para obtener más información sobre cómo se generan las métricas según la configuración de las métricas y los contadores de rendimiento, consulte la [tabla de métricas de Diagnósticos de Azure de almacenamiento](diagnostics-template.md#wadmetrics-tables-in-storage).
* El elemento **StorageAccount** tiene que actualizarse con el nombre de la cuenta de almacenamiento de diagnósticos.
  
    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
        <WadCfg>
          <DiagnosticMonitorConfiguration overallQuotaInMB="4096">
            <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter="Error"/>
            <PerformanceCounters scheduledTransferPeriod="PT1M">
          <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT15S" unit="Percent">
            <annotation displayName="CPU utilization" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Privileged Time" sampleRate="PT15S" unit="Percent">
            <annotation displayName="CPU privileged time" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% User Time" sampleRate="PT15S" unit="Percent">
            <annotation displayName="CPU user time" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\Processor Information(_Total)\Processor Frequency" sampleRate="PT15S" unit="Count">
            <annotation displayName="CPU frequency" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\System\Processes" sampleRate="PT15S" unit="Count">
            <annotation displayName="Processes" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\Process(_Total)\Thread Count" sampleRate="PT15S" unit="Count">
            <annotation displayName="Threads" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\Process(_Total)\Handle Count" sampleRate="PT15S" unit="Count">
            <annotation displayName="Handles" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\Memory\% Committed Bytes In Use" sampleRate="PT15S" unit="Percent">
            <annotation displayName="Memory usage" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Available Bytes" sampleRate="PT15S" unit="Bytes">
            <annotation displayName="Memory available" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Committed Bytes" sampleRate="PT15S" unit="Bytes">
            <annotation displayName="Memory committed" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Commit Limit" sampleRate="PT15S" unit="Bytes">
            <annotation displayName="Memory commit limit" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Pool Paged Bytes" sampleRate="PT15S" unit="Bytes">
            <annotation displayName="Memory paged pool" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Pool Nonpaged Bytes" sampleRate="PT15S" unit="Bytes">
            <annotation displayName="Memory non-paged pool" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\% Disk Time" sampleRate="PT15S" unit="Percent">
            <annotation displayName="Disk active time" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\% Disk Read Time" sampleRate="PT15S" unit="Percent">
            <annotation displayName="Disk active read time" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\% Disk Write Time" sampleRate="PT15S" unit="Percent">
            <annotation displayName="Disk active write time" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Transfers/sec" sampleRate="PT15S" unit="CountPerSecond">
            <annotation displayName="Disk operations" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Reads/sec" sampleRate="PT15S" unit="CountPerSecond">
            <annotation displayName="Disk read operations" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Writes/sec" sampleRate="PT15S" unit="CountPerSecond">
            <annotation displayName="Disk write operations" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Bytes/sec" sampleRate="PT15S" unit="BytesPerSecond">
            <annotation displayName="Disk speed" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Read Bytes/sec" sampleRate="PT15S" unit="BytesPerSecond">
            <annotation displayName="Disk read speed" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Write Bytes/sec" sampleRate="PT15S" unit="BytesPerSecond">
            <annotation displayName="Disk write speed" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Avg. Disk Queue Length" sampleRate="PT15S" unit="Count">
            <annotation displayName="Disk average queue length" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Avg. Disk Read Queue Length" sampleRate="PT15S" unit="Count">
            <annotation displayName="Disk average read queue length" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Avg. Disk Write Queue Length" sampleRate="PT15S" unit="Count">
            <annotation displayName="Disk average write queue length" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\LogicalDisk(_Total)\% Free Space" sampleRate="PT15S" unit="Percent">
            <annotation displayName="Disk free space (percentage)" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\LogicalDisk(_Total)\Free Megabytes" sampleRate="PT15S" unit="Count">
            <annotation displayName="Disk free space (MB)" locale="en-us"/>
          </PerformanceCounterConfiguration>
        </PerformanceCounters>
        <Metrics resourceId="(Update with resource ID for the VM)" >
            <MetricAggregation scheduledTransferPeriod="PT1H"/>
            <MetricAggregation scheduledTransferPeriod="PT1M"/>
        </Metrics>
        <WindowsEventLog scheduledTransferPeriod="PT1M">
          <DataSource name="Application!*[System[(Level = 1 or Level = 2)]]"/>
          <DataSource name="Security!*[System[(Level = 1 or Level = 2)]"/>
          <DataSource name="System!*[System[(Level = 1 or Level = 2)]]"/>
        </WindowsEventLog>
          </DiagnosticMonitorConfiguration>
        </WadCfg>
        <StorageAccount>(Update with diagnostics storage account name)</StorageAccount>
    </PublicConfig>
    ```

## <a name="next-steps"></a>Pasos siguientes
* Para obtener orientación adicional sobre el uso de la funcionalidad Diagnósticos de Azure y otras técnicas para solucionar problemas, consulte [Habilitación de diagnósticos en Azure Cloud Services y Virtual Machines](../../cloud-services/cloud-services-dotnet-diagnostics.md).
* [Esquema de configuración de diagnósticos](../../azure-monitor/platform/diagnostics-extension-versions.md) se explican las distintas opciones de configuración XML para la extensión de diagnósticos.