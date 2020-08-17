---
title: Inicio de máquinas mediante runbooks de Automation en Azure DevTest Labs
description: Aprenda a iniciar máquinas virtuales en un laboratorio en Azure DevTest Labs mediante runbooks de Azure Automation.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 231e79d594aab7c59fa21f9ee512abaa9ac67043
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87282269"
---
# <a name="start-virtual-machines-in-a-lab-in-order-by-using-azure-automation-runbooks"></a>Iniciar máquinas virtuales en un laboratorio en orden mediante runbooks de Azure Automation
La característica de [inicio automático](devtest-lab-set-lab-policy.md#set-autostart) de DevTest Labs le permite configurar las VM para que se inicien automáticamente a una hora determinada. Sin embargo, esta característica no es compatible con máquinas que se inician en un orden específico. Existen varios escenarios donde sería útil este tipo de automatización.  Un escenario es donde una VM de Jumpbox en un laboratorio debe iniciarse en primer lugar, antes que las otras VM, ya que Jumpbox se utiliza como punto de acceso a las otras VM.  En este artículo se muestra cómo configurar una cuenta de Azure Automation con un runbook de PowerShell que ejecuta un script. El script utiliza etiquetas en las VM del laboratorio para permitirle controlar el orden de inicio sin tener que cambiar el script.

## <a name="setup"></a>Configurar
En este ejemplo, las VM del laboratorio deben tener la etiqueta **StartupOrder** agregada con el valor apropiado (0, 1, 2, etcetera.). Designe cualquier máquina que no tenga que iniciarse como -1.

## <a name="create-an-azure-automation-account"></a>Creación de una cuenta de Azure Automation
Cree una cuenta de Azure Automation con las instrucciones que aparecen en [este artículo](../automation/automation-create-standalone-account.md). Elija la opción **Cuentas de ejecución** al crear la cuenta. Una vez creada la cuenta de automation, abra la página **Módulos** y seleccione **Actualizar módulos de Azure** en la barra de menús. Los módulos predeterminados son de varias versiones anteriores y, sin la actualización del script, es posible que no funcionen.

## <a name="add-a-runbook"></a>Agregar un runbook
Ahora, para agregar un runbook a la cuenta de automation, seleccione **Runbooks** en el menú de la izquierda. Seleccione **Agregar un runbook** en el menú y siga las instrucciones para [crear un runbook de PowerShell](../automation/learn/automation-tutorial-runbook-textual-powershell.md).

## <a name="powershell-script"></a>Script de PowerShell
El script siguiente toma el nombre de la suscripción y el nombre del laboratorio como parámetros. El flujo de script es obtener todas las VM del laboratorio y, a continuación, analizar la información de etiquetas para crear una lista de los nombres de VM y su orden de inicio. El script recorre las VM en orden y las inicia. Si existen varias VM en un número de orden específico, se inician de forma asincrónica mediante trabajos de PowerShell. Para esas VM que no tienen una etiqueta, establezca el valor de inicio en el último (10) y se iniciarán las últimas de forma predeterminada.  Si el laboratorio no quiere que la VM se inicie automáticamente, establezca el valor de la etiqueta en 11 y se omitirá.

```powershell
#Requires -Version 3.0
#Requires -Module AzureRM.Resources

param
(
    [Parameter(Mandatory=$false, HelpMessage="Name of the subscription that has the lab")]
    [string] $SubscriptionName,

    [Parameter(Mandatory=$false, HelpMessage="Lab name")]
    [string] $LabName
)

# Connect and add the appropriate subscription
$Conn = Get-AutomationConnection -Name AzureRunAsConnection

Add-AzureRMAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationID $Conn.ApplicationId -Subscription $SubscriptionName -CertificateThumbprint $Conn.CertificateThumbprint

# Find the lab
$dtLab = Find-AzResource -ResourceType 'Microsoft.DevTestLab/labs' -ResourceNameEquals $LabName

# Get the VMs
$dtlAllVms = New-Object System.Collections.ArrayList
$AllVMs = Get-AzResource -ResourceId "$($dtLab.ResourceId)/virtualmachines" -ApiVersion 2016-05-15

# Get the StartupOrder tag, if missing set to be run last (10)
ForEach ($vm in $AllVMs) {
    if ($vm.Tags) {
        if ($vm.Tags['StartupOrder']) {
            $startupValue = $vm.Tags['StartupOrder']
        } else {
            $startupValue = 10
        }
        } else {
            $startupValue = 10
        }
        $dtlAllVms.Add(@{$vm.Name = $startupValue}) > $null
}

# Setup for the async multiple vm start

# Save profile
$profilePath = Join-Path $env:Temp "profile.json"
If (Test-Path $profilePath){
    Remove-Item $profilePath
}
Save-AzContext -Path $profilePath

# Job to start VMs asynch
$startVMBlock = {
    Param($devTestLab,$vmToStart,$profilePath)
    Import-AzContext -Path ($profilePath)
    Invoke-AzResourceAction `
        -ResourceId "$($devTestLab.ResourceId)/virtualmachines/$vmToStart" `
        -Action Start `
        -Force
    Write-Output "Started: $vmToStart"
}

$current = 0
# Start in order from 0 to 10

While ($current -le 10) {
# Get the VMs in the current stage
    $tobeStarted = $null
    $tobeStarted = $dtlAllVms | Where-Object { $_.Values -eq $current}
    if ($tobeStarted.Count -eq 1) {
        # Run sync – jobs not necessary for a single VM
        $returnStatus = Invoke-AzResourceAction `
                -ResourceId "$($dtLab.ResourceId)/virtualmachines/$($tobeStarted.Keys)" `
                -Action Start `
                -Force
        Write-Output "$($tobeStarted.Keys) status: $($returnStatus.status)"
    } elseif ($tobeStarted.Count -gt 1) {
        # Start multiple VMs async
        $jobs = @()
        Write-Output "Start Jobs start: $(Get-Date)"
        
        # Jobs
        $jobs += Start-Job -ScriptBlock $startVMBlock -ArgumentList $dtLab, $($singlevm.Keys), $profilePath
        Write-Output "Start Jobs end: $(Get-Date)"
    }

    # Get results from all jobs
    if($jobs.Count -ne 0) {
        Write-Output "Receive Jobs start: $(Get-Date)"
        foreach ($job in $jobs){
            $jobResult = Receive-Job -Job $job -Wait | Write-Output
        }
        Remove-Job -Job $jobs -Force
    }
    else
    {
        Write-Output "Information: No jobs available"
    }
}
```

## <a name="create-a-schedule"></a>Crear una programación
Para que este script se ejecute diariamente, [cree una programación](../automation/shared-resources/schedules.md#create-a-schedule) en la cuenta de automation. Una vez creada la programación, [vincúlela al runbook](../automation/shared-resources/schedules.md#link-a-schedule-to-a-runbook). 

En una situación a gran escala donde hay varias suscripciones con varios laboratorios, almacene la información de parámetros en un archivo para distintos laboratorios y pase el archivo al script en lugar de parámetros individuales. El script deberá modificarse, pero la ejecución principal será la misma. Aunque este ejemplo Azure Automation para ejecutar el script de PowerShell, existen otras opciones, como usar una tarea en una canalización de compilación o versión.

## <a name="next-steps"></a>Pasos siguientes
Para más información acerca de Azure Automation, consulte el siguiente artículo: [Introducción a Azure Automation](../automation/automation-intro.md).
