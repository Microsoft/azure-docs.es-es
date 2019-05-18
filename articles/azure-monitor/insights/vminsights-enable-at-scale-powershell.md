---
title: Habilitar Monitor de Azure para máquinas virtuales (versión preliminar) con plantilla de Resource Manager o Azure PowerShell | Microsoft Docs
description: En este artículo se describe cómo habilitar a Azure Monitor para las máquinas virtuales para uno o más máquinas virtuales o conjuntos de escalado de máquinas virtuales con plantillas de Azure PowerShell o Azure Resource Manager.
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/09/2019
ms.author: magoedte
ms.openlocfilehash: a22bc88fb066d9b845f7fdf1592e2194a03915bc
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65524136"
---
# <a name="enable-azure-monitor-for-vms-preview-using-azure-powershell-or-resource-manager-template"></a>Habilitar a Monitor de Azure para máquinas virtuales (versión preliminar) con plantilla de Resource Manager o Azure PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

En este artículo se explica cómo habilitar a Azure Monitor para las máquinas virtuales (versión preliminar) para máquinas virtuales o conjuntos de escalado de máquinas virtuales con plantillas de Azure PowerShell o Azure Resource Manager. Al final de este proceso que habrá ha confirmado correctamente la supervisión todas las máquinas virtuales y obtenga información sobre si alguno está experimentando problemas de rendimiento o disponibilidad. 

## <a name="set-up-a-log-analytics-workspace"></a>Configure un área de trabajo de Log Analytics 

Si no tiene un área de trabajo de Log Analytics, crear uno mediante la revisión de los métodos que se sugieren en el [requisitos previos](vminsights-enable-overview.md#log-analytics) sección antes de continuar con los pasos para configurarlo para poder completar la implementación de Azure Monitor para Máquinas virtuales mediante el método de plantilla de Azure Resource Manager.

### <a name="enable-performance-counters"></a>Activar los contadores de rendimiento

Si el área de trabajo de Log Analytics a la que la solución hace referencia no está todavía configurada para recopilar los contadores de rendimiento que la solución requiere, debe habilitarlos. Puede hacerlo de dos maneras:
* Manualmente, según se describe en [Orígenes de datos de rendimiento de Windows y Linux en Log Analytics](../../azure-monitor/platform/data-sources-performance-counters.md)
* Descargando y ejecutando un script de PowerShell que está disponible en [Galería de Azure PowerShell](https://www.powershellgallery.com/packages/Enable-VMInsightsPerfCounters/1.1)

### <a name="install-the-servicemap-and-infrastructureinsights-solutions"></a>Instalación de las soluciones ServiceMap e InfrastructureInsights
Este método incluye una plantilla JSON que especifica la configuración para habilitar los componentes de la solución en el área de trabajo de Log Analytics.

Si no sabe cómo implementar los recursos con una plantilla, consulte:
* [Implementación de recursos con las plantillas de Resource Manager y Azure PowerShell](../../azure-resource-manager/resource-group-template-deploy.md)
* [Implementación de recursos con plantillas de Resource Manager y la CLI de Azure](../../azure-resource-manager/resource-group-template-deploy-cli.md)

Si decide usar la CLI de Azure, primero debe instalar y usar la CLI localmente. Debe ejecuta la versión 2.0.27 de la CLI de Azure, o cualquier versión posterior. Para identificar la versión, ejecute `az --version`. Si necesita instalar o actualizar la CLI de Azure, consulte [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli).

1. Copie y pegue la siguiente sintaxis JSON en el archivo:

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "WorkspaceName": {
                "type": "string"
            },
            "WorkspaceLocation": {
                "type": "string"
            }
        },
        "resources": [
            {
                "apiVersion": "2017-03-15-preview",
                "type": "Microsoft.OperationalInsights/workspaces",
                "name": "[parameters('WorkspaceName')]",
                "location": "[parameters('WorkspaceLocation')]",
                "resources": [
                    {
                        "apiVersion": "2015-11-01-preview",
                        "location": "[parameters('WorkspaceLocation')]",
                        "name": "[concat('ServiceMap', '(', parameters('WorkspaceName'),')')]",
                        "type": "Microsoft.OperationsManagement/solutions",
                        "dependsOn": [
                            "[concat('Microsoft.OperationalInsights/workspaces/', parameters('WorkspaceName'))]"
                        ],
                        "properties": {
                            "workspaceResourceId": "[resourceId('Microsoft.OperationalInsights/workspaces/', parameters('WorkspaceName'))]"
                        },

                        "plan": {
                            "name": "[concat('ServiceMap', '(', parameters('WorkspaceName'),')')]",
                            "publisher": "Microsoft",
                            "product": "[Concat('OMSGallery/', 'ServiceMap')]",
                            "promotionCode": ""
                        }
                    },
                    {
                        "apiVersion": "2015-11-01-preview",
                        "location": "[parameters('WorkspaceLocation')]",
                        "name": "[concat('InfrastructureInsights', '(', parameters('WorkspaceName'),')')]",
                        "type": "Microsoft.OperationsManagement/solutions",
                        "dependsOn": [
                            "[concat('Microsoft.OperationalInsights/workspaces/', parameters('WorkspaceName'))]"
                        ],
                        "properties": {
                            "workspaceResourceId": "[resourceId('Microsoft.OperationalInsights/workspaces/', parameters('WorkspaceName'))]"
                        },
                        "plan": {
                            "name": "[concat('InfrastructureInsights', '(', parameters('WorkspaceName'),')')]",
                            "publisher": "Microsoft",
                            "product": "[Concat('OMSGallery/', 'InfrastructureInsights')]",
                            "promotionCode": ""
                        }
                    }
                ]
            }
        ]
    }
    ```

1. Guarde este archivo como *installsolutionsforvminsights.json* en una carpeta local.

1. Capture los valores de *WorkspaceName*, *ResourceGroupName* y *WorkspaceLocation*. El valor de *WorkspaceName* es el nombre del área de trabajo de Log Analytics. El valor de *WorkspaceLocation* es la región en la que el área de trabajo está definida.

1. Ya está listo para implementar esta plantilla.
 
    * Use los siguientes comandos de PowerShell en la carpeta que contenga la plantilla:

        ```powershell
        New-AzResourceGroupDeployment -Name DeploySolutions -TemplateFile InstallSolutionsForVMInsights.json -ResourceGroupName <ResourceGroupName> -WorkspaceName <WorkspaceName> -WorkspaceLocation <WorkspaceLocation - example: eastus>
        ```

        El cambio de configuración puede tardar unos minutos en completarse. Cuando se completa, se muestra un mensaje que incluye el resultado y que es similar al siguiente:

        ```powershell
        provisioningState       : Succeeded
        ```

    * Para ejecutar el comando siguiente mediante la CLI de Azure:
    
        ```azurecli
        az login
        az account set --subscription "Subscription Name"
        az group deployment create --name DeploySolutions --resource-group <ResourceGroupName> --template-file InstallSolutionsForVMInsights.json --parameters WorkspaceName=<workspaceName> WorkspaceLocation=<WorkspaceLocation - example: eastus>
        ```

El cambio de configuración puede tardar unos minutos en completarse. Cuando se completa, se muestra un mensaje que incluye el resultado y que es similar al siguiente:

```azurecli
provisioningState       : Succeeded
```

## <a name="enable-with-azure-resource-manager-template"></a>Habilitar la plantilla de Azure Resource Manager
Hemos creado las plantillas de Azure Resource Manager de ejemplo para la incorporación de las máquinas virtuales y conjuntos de escalado de máquinas virtuales. Estas plantillas incluyen escenarios para habilitar la supervisión en un recurso existente y para crear un nuevo recurso que tendrá la supervisión habilitada.

>[!NOTE]
>La plantilla debe implementarse en el mismo grupo de recursos que el recurso esté incorporado.

Si no conoce el concepto de implementación de recursos mediante una plantilla, consulte:
* [Implementación de recursos con las plantillas de Resource Manager y Azure PowerShell](../../azure-resource-manager/resource-group-template-deploy.md)
* [Implementación de recursos con plantillas de Resource Manager y la CLI de Azure](../../azure-resource-manager/resource-group-template-deploy-cli.md)

Si decide usar la CLI de Azure, primero debe instalar y usar la CLI localmente. Debe ejecuta la versión 2.0.27 de la CLI de Azure, o cualquier versión posterior. Para identificar la versión, ejecute `az --version`. Si necesita instalar o actualizar la CLI de Azure, consulte [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli). 

### <a name="download-templates"></a>Descargue las plantillas

Se proporcionan las plantillas de Azure Resource Manager en un archivo de almacenamiento (.zip) que puede [descargar](https://aka.ms/VmInsightsARMTemplates) desde nuestro repositorio de GitHub. Contenido del archivo incluye las carpetas que representa cada escenario de implementación con un archivo de plantilla y parámetro. Antes de ejecutarse, modifique el archivo de parámetros y especifique los valores necesarios. No modifique el archivo de plantilla a menos que necesite personalizar para admitir requisitos determinados. Después de haber modificado el archivo de parámetros puede implementarla mediante los métodos siguientes que se describe más adelante en este artículo. 

El archivo de descarga contiene las siguientes plantillas para distintos escenarios:

- **ExistingVmOnboarding** plantilla habilita Azure Monitor para las máquinas virtuales si las máquinas virtuales ya existe.
- **NewVmOnboarding** plantilla crea una máquina virtual y habilita el Monitor de Azure para máquinas virtuales para supervisarlo.
- **ExistingVmssOnboarding** plantilla permite a Azure Monitor para las máquinas virtuales si la escala de máquina virtual ya existe.
- **NewVmssOnboarding** plantilla crea conjuntos de escalado de máquinas virtuales y habilita el Monitor de Azure para máquinas virtuales supervisarlos.
- **ConfigureWorksapce*** plantilla configura el área de trabajo de Log Analytics para admitir Azure Monitor para las máquinas virtuales mediante la habilitación de la colección de contadores de rendimiento del sistema operativo Windows y Linux y soluciones.

>[!NOTE]
>Si ya estaban presentes conjuntos de escalado de máquinas virtuales y la directiva de actualización se establece en **Manual**, Azure Monitor para las máquinas virtuales no se habilitarán para las instancias de forma predeterminada después de ejecutar el **ExistingVmssOnboarding** Plantilla de Azure Resource Manager. Tendrá que actualizar manualmente las instancias.

### <a name="deploy-using-azure-powershell"></a>Implementación mediante Azure PowerShell

El paso siguiente habilita la supervisión mediante Azure PowerShell.

```powershell
New-AzResourceGroupDeployment -Name OnboardCluster -ResourceGroupName <ResourceGroupName> -TemplateFile <Template.json> -TemplateParameterFile <Parameters.json>
```
El cambio de configuración puede tardar unos minutos en completarse. Cuando se completa, se muestra un mensaje que incluye el resultado y que es similar al siguiente:

```powershell
provisioningState       : Succeeded
```
### <a name="deploy-using-azure-cli"></a>Implementación con la CLI de Azure

El paso siguiente habilita la supervisión mediante la CLI de Azure.   

```azurecli
az login
az account set --subscription "Subscription Name"
az group deployment create --resource-group <ResourceGroupName> --template-file <Template.json> --parameters <Parameters.json>
```

La salida será similar a la siguiente:

```azurecli
provisioningState       : Succeeded
```

## <a name="enable-with-powershell"></a>Habilitar con PowerShell

Para habilitar Azure Monitor para VM en varias máquinas virtuales o conjuntos de escalado de máquinas virtuales, puede usar el script de PowerShell [Install-VMInsights.ps1](https://www.powershellgallery.com/packages/Install-VMInsights/1.0), disponible en la Galería de Azure PowerShell. Este script se repite en todas las máquinas virtuales y conjuntos de escalado de máquinas virtuales de la suscripción, en el grupo de recursos con ámbito que especifica *ResourceGroup* o en una sola máquina virtual o en un conjunto de escalado de máquinas virtuales especificado por *Nombre*. En cada máquina virtual o conjunto de escalado de máquinas virtuales, el script comprueba si la extensión de la máquina virtual ya está instalada. Si no lo está, el script vuelve a intentar instalarla. Si la extensión de máquina virtual está instalada, el script instala las extensiones de máquina virtual del agente de Log Analytics y de Dependency Agent.

Este script requiere Azure PowerShell versión del módulo de Az 1.0.0 o versiones posteriores. Ejecute `Get-Module -ListAvailable Az` para encontrar la versión. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

Para obtener una lista de los detalles de los argumentos del script y un ejemplo de uso, ejecute `Get-Help`.

```powershell
Get-Help .\Install-VMInsights.ps1 -Detailed

SYNOPSIS
    This script installs VM extensions for Log Analytics and the Dependency agent as needed for VM Insights.


SYNTAX
    .\Install-VMInsights.ps1 [-WorkspaceId] <String> [-WorkspaceKey] <String> [-SubscriptionId] <String> [[-ResourceGroup]
    <String>] [[-Name] <String>] [[-PolicyAssignmentName] <String>] [-ReInstall] [-TriggerVmssManualVMUpdate] [-Approve] [-WorkspaceRegion] <String>
    [-WhatIf] [-Confirm] [<CommonParameters>]


DESCRIPTION
    This script installs or re-configures following on VMs and VM Scale Sets:
    - Log Analytics VM Extension configured to supplied Log Analytics Workspace
    - Dependency Agent VM Extension

    Can be applied to:
    - Subscription
    - Resource Group in a Subscription
    - Specific VM/VM Scale Set
    - Compliance results of a policy for a VM or VM Extension

    Script will show you list of VMs/VM Scale Sets that will apply to and let you confirm to continue.
    Use -Approve switch to run without prompting, if all required parameters are provided.

    If the extensions are already installed will not install again.
    Use -ReInstall switch if you need to for example update the workspace.

    Use -WhatIf if you would like to see what would happen in terms of installs, what workspace configured to, and status of the extension.


PARAMETERS
    -WorkspaceId <String>
        Log Analytics WorkspaceID (GUID) for the data to be sent to

    -WorkspaceKey <String>
        Log Analytics Workspace primary or secondary key

    -SubscriptionId <String>
        SubscriptionId for the VMs/VM Scale Sets
        If using PolicyAssignmentName parameter, subscription that VMs are in

    -ResourceGroup <String>
        <Optional> Resource Group to which the VMs or VM Scale Sets belong

    -Name <String>
        <Optional> To install to a single VM/VM Scale Set

    -PolicyAssignmentName <String>
        <Optional> Take the input VMs to operate on as the Compliance results from this Assignment
        If specified will only take from this source.

    -ReInstall [<SwitchParameter>]
        <Optional> If VM/VM Scale Set is already configured for a different workspace, set this to change to the new workspace

    -TriggerVmssManualVMUpdate [<SwitchParameter>]
        <Optional> Set this flag to trigger update of VM instances in a scale set whose upgrade policy is set to Manual

    -Approve [<SwitchParameter>]
        <Optional> Gives the approval for the installation to start with no confirmation prompt for the listed VMs/VM Scale Sets

    -WorkspaceRegion <String>
        Region the Log Analytics Workspace is in
        Supported values: "East US","eastus","Southeast Asia","southeastasia","West Central US","westcentralus","West Europe","westeurope"
        For Health supported is: "East US","eastus","West Central US","westcentralus"

    -WhatIf [<SwitchParameter>]
        <Optional> See what would happen in terms of installs.
        If extension is already installed will show what workspace is currently configured, and status of the VM extension

    -Confirm [<SwitchParameter>]
        <Optional> Confirm every action

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (https:/go.microsoft.com/fwlink/?LinkID=113216).

    -------------------------- EXAMPLE 1 --------------------------
    .\Install-VMInsights.ps1 -WorkspaceRegion eastus -WorkspaceId <WorkspaceId>-WorkspaceKey <WorkspaceKey> -SubscriptionId <SubscriptionId>
    -ResourceGroup <ResourceGroup>

    Install for all VMs in a Resource Group in a subscription

    -------------------------- EXAMPLE 2 --------------------------
    .\Install-VMInsights.ps1 -WorkspaceRegion eastus -WorkspaceId <WorkspaceId>-WorkspaceKey <WorkspaceKey> -SubscriptionId <SubscriptionId>
    -ResourceGroup <ResourceGroup> -ReInstall

    Specify to reinstall extensions even if already installed, for example to update to a different workspace

    -------------------------- EXAMPLE 3 --------------------------
    .\Install-VMInsights.ps1 -WorkspaceRegion eastus -WorkspaceId <WorkspaceId>-WorkspaceKey <WorkspaceKey> -SubscriptionId <SubscriptionId>
    -PolicyAssignmentName a4f79f8ce891455198c08736 -ReInstall

    Specify to use a PolicyAssignmentName for source, and to reinstall (move to a new workspace)
```

En el ejemplo siguiente se muestra cómo utilizar los comandos de PowerShell en la carpeta para habilitar Azure Monitor para VM y comprender el resultado esperado:

```powershell
$WorkspaceId = "<GUID>"
$WorkspaceKey = "<Key>"
$SubscriptionId = "<GUID>"
.\Install-VMInsights.ps1 -WorkspaceId $WorkspaceId -WorkspaceKey $WorkspaceKey -SubscriptionId $SubscriptionId -WorkspaceRegion eastus

Getting list of VMs or VM ScaleSets matching criteria specified

VMs or VM ScaleSets matching criteria:

db-ws-1 VM running
db-ws2012 VM running

This operation will install the Log Analytics and Dependency agent extensions on above 2 VMs or VM Scale Sets.
VMs in a non-running state will be skipped.
Extension will not be reinstalled if already installed. Use -ReInstall if desired, for example to update workspace

Confirm
Continue?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): y

db-ws-1 : Deploying DependencyAgentWindows with name DAExtension
db-ws-1 : Successfully deployed DependencyAgentWindows
db-ws-1 : Deploying MicrosoftMonitoringAgent with name MMAExtension
db-ws-1 : Successfully deployed MicrosoftMonitoringAgent
db-ws2012 : Deploying DependencyAgentWindows with name DAExtension
db-ws2012 : Successfully deployed DependencyAgentWindows
db-ws2012 : Deploying MicrosoftMonitoringAgent with name MMAExtension
db-ws2012 : Successfully deployed MicrosoftMonitoringAgent

Summary:

Already Onboarded: (0)

Succeeded: (4)
db-ws-1 : Successfully deployed DependencyAgentWindows
db-ws-1 : Successfully deployed MicrosoftMonitoringAgent
db-ws2012 : Successfully deployed DependencyAgentWindows
db-ws2012 : Successfully deployed MicrosoftMonitoringAgent

Connected to different workspace: (0)

Not running - start VM to configure: (0)

Failed: (0)
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que la supervisión está habilitada para las máquinas virtuales, esta información está disponible para su análisis con Azure Monitor para las máquinas virtuales. Para obtener información sobre cómo usar la característica de mantenimiento, consulte [Descripción del estado de las máquinas virtuales de Azure con Azure Monitor para VM (versión preliminar)](vminsights-health.md). Para ver las dependencias de las aplicaciones detectadas, consulte [Uso de la asignación de Azure Monitor para VM (versión preliminar) para conocer los componentes de una aplicación](vminsights-maps.md). Para identificar cuellos de botella y el uso general con el rendimiento de sus VM, consulte el artículo sobre cómo [ver el rendimiento de la VM de Azure](vminsights-performance.md), o bien, para ver las dependencias de la aplicación detectadas, consulte cómo [ver el Azure Monitor para la asignación de VM](vminsights-maps.md).