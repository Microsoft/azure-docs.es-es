---
title: Generación de perfiles de aplicaciones web en una VM de Azure - Application Insights Profiler
description: Generación de perfiles de aplicaciones web en una VM de Azure mediante Application Insights Profiler.
ms.topic: conceptual
author: cweining
ms.author: cweining
ms.date: 11/08/2019
ms.reviewer: mbullwin
ms.openlocfilehash: f514dd7b54ac091535aeab43a8a7d2a645b50a09
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87315842"
---
# <a name="profile-web-apps-running-on-an-azure-virtual-machine-or-a-virtual-machine-scale-set-by-using-application-insights-profiler"></a>Generación de perfiles de aplicaciones web en ejecución en una máquina virtual o un conjunto de escalado de máquinas virtuales de Azure mediante Application Insights Profiler

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

También puede implementar Azure Application Insights Profiler en estos servicios:
* [Azure App Service](./profiler.md?toc=%2fazure%2fazure-monitor%2ftoc.json)
* [Azure Cloud Services](profiler-cloudservice.md?toc=/azure/azure-monitor/toc.json)
* [Azure Service Fabric](?toc=%2fazure%2fazure-monitor%2ftoc.json)

## <a name="deploy-profiler-on-a-virtual-machine-or-a-virtual-machine-scale-set"></a>Implementación de Profiler en una máquina virtual o un conjunto de escalado de máquinas virtuales
En este artículo se muestran los pasos necesarios para que Application Insights Profiler se ejecute en una máquina virtual (VM) de Azure o un conjunto de escalado de máquinas virtuales de Azure. Profiler se instala con la extensión de Azure Diagnostics para VM. Configure la extensión para ejecutar Profiler y compile el SDK de Application Insights en la aplicación.

1. Agregue el SDK de Application Insights a la [aplicación de ASP.NET](./asp-net.md).

   Para ver los perfiles para las solicitudes, debe enviar telemetría de solicitudes a Application Insights.

1. Instale la extensión de Azure Diagnostics en la VM. Para consultar ejemplos de plantillas de Azure Resource Manager completas:  
   * [Máquina virtual](https://github.com/Azure/azure-docs-json-samples/blob/master/application-insights/WindowsVirtualMachine.json)
   * [Conjunto de escalado de máquinas virtuales](https://github.com/Azure/azure-docs-json-samples/blob/master/application-insights/WindowsVirtualMachineScaleSet.json)
    
     El elemento clave es ApplicationInsightsProfilerSink en WadCfg. Para indicar a Azure Diagnostics que habilite Profiler para enviar datos a la clave iKey, agregue otro receptor a esta sección.
    
     ```json
     "SinksConfig": {
       "Sink": [
         {
           "name": "ApplicationInsightsSink",
           "ApplicationInsights": "85f73556-b1ba-46de-9534-606e08c6120f"
         },
         {
           "name": "MyApplicationInsightsProfilerSink",
           "ApplicationInsightsProfiler": "85f73556-b1ba-46de-9534-606e08c6120f"
         }
       ]
     },
     ```

1. Implemente la definición de implementación de entorno modificada.  

   Al aplicar las modificaciones, generalmente se realiza una implementación de plantilla completa o una publicación basada en el servicio en la nube mediante cmdlets de PowerShell o Visual Studio.  

   Los siguientes comandos de PowerShell se pueden usar como un enfoque alternativo para las máquinas virtuales existentes que solo afecta a la extensión de Azure Diagnostics. Agregue el valor ProfilerSink que se mencionó anteriormente a la configuración que devuelve el comando Get-AzVMDiagnosticsExtension. Luego, pase la configuración actualizada al comando Set-AzVMDiagnosticsExtension.

    ```powershell
    $ConfigFilePath = [IO.Path]::GetTempFileName()
    # After you export the currently deployed Diagnostics config to a file, edit it to include the ApplicationInsightsProfiler sink.
    (Get-AzVMDiagnosticsExtension -ResourceGroupName "MyRG" -VMName "MyVM").PublicSettings | Out-File -Verbose $ConfigFilePath
    # Set-AzVMDiagnosticsExtension might require the -StorageAccountName argument
    # If your original diagnostics configuration had the storageAccountName property in the protectedSettings section (which is not downloadable), be sure to pass the same original value you had in this cmdlet call.
    Set-AzVMDiagnosticsExtension -ResourceGroupName "MyRG" -VMName "MyVM" -DiagnosticsConfigurationPath $ConfigFilePath
    ```

1. Si la aplicación deseada se ejecuta mediante [IIS](https://www.microsoft.com/web/downloads/platform.aspx), habilite la característica `IIS Http Tracing` de Windows.

   a. Establezca el acceso remoto al entorno y, a continuación, use la ventana [Agregar características de Windows](/iis/configuration/system.webserver/tracing/). También puede ejecutar el siguiente comando en PowerShell (como administrador):  

    ```powershell
    Enable-WindowsOptionalFeature -FeatureName IIS-HttpTracing -Online -All
    ```  
   b. Si el establecimiento del acceso remoto supone un problema, puede usar la [CLI de Azure](/cli/azure/get-started-with-azure-cli) para ejecutar el siguiente comando:  

    ```powershell
    az vm run-command invoke -g MyResourceGroupName -n MyVirtualMachineName --command-id RunPowerShellScript --scripts "Enable-WindowsOptionalFeature -FeatureName IIS-HttpTracing -Online -All"
    ```

1. Implementación de aplicación.

## <a name="set-profiler-sink-using-azure-resource-explorer"></a>Establecimiento del receptor de Profiler con Azure Resource Explorer
Todavía no tenemos forma de establecer el receptor de Application Insights Profiler desde el portal. En lugar de usar PowerShell como se describió anteriormente, puede usar Azure Resource Explorer para establecer el receptor. Pero tenga en cuenta que, si vuelve a implementar la máquina virtual, el receptor se perderá. Deberá actualizar la configuración que usa al implementar la VM para conservar este ajuste.

1. Compruebe que la extensión Windows Azure Diagnostics esté instalada. Para ello, revise las extensiones instaladas para la máquina virtual.  

    ![Comprobación de la instalación de la extensión WAD.][wadextension]

2. Busque la extensión de diagnóstico de VM para su máquina virtual. Vaya a [https://resources.azure.com](https://resources.azure.com). Expanda el grupo de recursos, Microsoft.Compute virtualMachines, el nombre de la máquina virtual y las extensiones.  

    ![Vaya a la configuración de WAD en Azure Resource Explorer.][azureresourceexplorer]

3. Agregue el receptor de Application Insights Profiler al nodo SinksConfig en WadCfg. Si todavía no tiene una sección SinksConfig, es posible que tenga que agregar una. Asegúrese de especificar la iKey de Application Insights correspondiente en la configuración. Deberá cambiar el modo de los exploradores a Lectura/escritura en la esquina superior derecha y presionar el botón "Editar" azul.

    ![Incorporación del receptor de Application Insights Profiler][resourceexplorersinksconfig]

4. Cuando haya terminado de editar la configuración, presione "Put". Si la operación Put se realiza correctamente, aparecerá una marca verde en medio de la pantalla.

    ![Envío de una solicitud Put para aplicar los cambios][resourceexplorerput]






## <a name="can-profiler-run-on-on-premises-servers"></a>¿Puede ejecutarse Profiler en servidores locales?
No tenemos planes para que Application Insights Profiler sea compatible con servidores locales.

## <a name="next-steps"></a>Pasos siguientes

- Genere tráfico para su aplicación (por ejemplo, inicie una [prueba de disponibilidad](monitor-web-app-availability.md)). A continuación, espere de 10 a 15 minutos para que se empiecen a enviar seguimientos a la instancia de Application Insights.
- Consulte [Seguimientos de Profiler](profiler-overview.md?toc=/azure/azure-monitor/toc.json) en Azure Portal.
- Para obtener ayuda para solucionar problemas de Profiler, consulte la sección [Solución de problemas de Profiler](profiler-troubleshooting.md?toc=/azure/azure-monitor/toc.json).

[azureresourceexplorer]: ./media/profiler-vm/azure-resource-explorer.png
[resourceexplorerput]: ./media/profiler-vm/resource-explorer-put.png
[resourceexplorersinksconfig]: ./media/profiler-vm/resource-explorer-sinks-config.png
[wadextension]: ./media/profiler-vm/wad-extension.png

