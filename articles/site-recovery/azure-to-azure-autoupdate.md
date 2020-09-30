---
title: Actualización automática de Mobility Service en Azure Site Recovery
description: Descripción general de la actualización automática de Mobility Service al replicar VM de Azure mediante Azure Site Recovery.
services: site-recovery
author: sideeksh
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 04/02/2020
ms.author: sideeksh
ms.openlocfilehash: 53c5dc4920b6c50ee3c900db9626f4d283f7b846
ms.sourcegitcommit: ac5cbef0706d9910a76e4c0841fdac3ef8ed2e82
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89426425"
---
# <a name="automatic-update-of-the-mobility-service-in-azure-to-azure-replication"></a>Actualización automática de Mobility Service en Azure para la replicación de Azure

Azure Site Recovery usa una cadencia de lanzamiento mensual para solucionar cualquier problema y mejorar las funciones existentes o agregar otras nuevas. Para mantenerse actualizado con el servicio, debe planear la implementación de revisiones cada mes. Para evitar la sobrecarga asociada con cada actualización, puede permitir que Site Recovery administre las actualizaciones de los componentes.

Tal como se menciona en la [arquitectura de recuperación ante desastres de Azure a Azure](azure-to-azure-architecture.md), el servicio Mobility se instala en todas las máquinas virtuales de Azure (VM) que tienen habilitada la replicación desde una región de Azure a otra. Cuando usa actualizaciones automáticas, cada nueva versión actualiza la extensión de Mobility Service.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="how-automatic-updates-work"></a>Cómo funcionan las actualizaciones automáticas

Cuando usa Site Recovery para administrar las actualizaciones, se implementa un runbook global (que usan los servicios de Azure) a través de una cuenta de automatización creada en la misma suscripción que el almacén. Cada almacén usa una cuenta de automatización. Para cada máquina virtual de un almacén, el runbook comprueba si hay actualizaciones automáticas activas. Si hay disponible una versión más reciente de la extensión del servicio Mobility, se instalará la actualización.

La programación predeterminada del runbook se produce diariamente a las 12:00 a. m. en la zona horaria de la geografía de la máquina virtual replicada. También puede cambiar el horario del runbook a través de la cuenta de automatización.

> [!NOTE]
> Si comienza con el [paquete acumulativo de actualizaciones 35](site-recovery-whats-new.md#updates-march-2019), puede elegir una cuenta de automatización existente que puede usar para las actualizaciones. Antes del paquete acumulativo de actualizaciones 35, Site Recovery creó la cuenta de automatización de forma predeterminada. Solo puede seleccionar esta opción al habilitar la replicación para una máquina virtual. No está disponible para una máquina virtual que ya tenga habilitada la replicación. Los valores que seleccione se aplicarán a todas las máquinas virtuales de Azure protegidas en el mismo almacén.

La activación de las actualizaciones automáticas no requiere reiniciar las VM de Azure ni afecta a la replicación en curso.

La facturación de trabajos en la cuenta de automatización se basa en la cantidad de minutos de runtime de los trabajos usados en un mes. La ejecución del trabajo toma entre unos segundos y un minuto cada día aproximadamente, y se establece como unidades gratuitas. De forma predeterminada, se incluyen 500 minutos como unidades gratuitas en una cuenta de automatización, como se muestra en la tabla siguiente:

| Unidades gratuitas incluidas (cada mes) | Price |
|---|---|
| Tiempo de trabajo de 500 minutos | ₹0,14/minuto

## <a name="enable-automatic-updates"></a>Habilitar las actualizaciones automáticas

Site Recovery puede administrar las actualizaciones de la extensión de varias maneras:

- [Administrar como parte del paso para habilitar la replicación](#manage-as-part-of-the-enable-replication-step)
- [Alternar la configuración de actualización de la extensión dentro del almacén.](#toggle-the-extension-update-settings-inside-the-vault)
- [Administrar actualizaciones manualmente](#manage-updates-manually)

### <a name="manage-as-part-of-the-enable-replication-step"></a>Administrar como parte del paso para habilitar la replicación

Cuando habilita la replicación para una VM iniciándola [desde la vista de la VM](azure-to-azure-quickstart.md) o [desde el almacén de Recovery Services](azure-to-azure-how-to-enable-replication.md), puede permitir que Site Recovery administre las actualizaciones para la extensión de Site Recovery o puede administrarlas manualmente.

:::image type="content" source="./media/azure-to-azure-autoupdate/enable-rep.png" alt-text="Configuración de extensiones":::

### <a name="toggle-the-extension-update-settings-inside-the-vault"></a>Alternar la configuración de actualización de la extensión dentro del almacén

1. En el almacén de Recovery Services, vaya a **Administrar** > **Infraestructura de Site Recovery**.
1. En **Para las máquinas virtuales de Azure** > **Configuración de actualización de extensiones** > **Permitir la administración por parte de Site Recovery**, seleccione **Activado**.

   Para administrar la extensión manualmente, seleccione **Desactivado**.

1. Seleccione **Guardar**.

:::image type="content" source="./media/azure-to-azure-autoupdate/vault-toggle.png" alt-text="Configuración de actualización de extensiones":::

> [!IMPORTANT]
> Si elige la opción **Permitir la administración por parte de Site Recovery**, la configuración se aplica a todas las máquinas virtuales del almacén.

> [!NOTE]
> Cualquiera de las opciones le ofrece información de la cuenta de automatización que se usa para administrar las actualizaciones. Si está usando esta función en un almacén por primera vez, se creará una nueva cuenta de automatización de forma predeterminada. Alternativamente, puede personalizar la configuración y elegir una cuenta de automatización ya existente. Todas las tareas posteriores para habilitar la replicación en el mismo almacén usarán la cuenta de automatización que se creó anteriormente. Actualmente, en el menú desplegable solo se muestran las cuentas de automatización que se encuentran en el mismo grupo de recursos que el almacén.

> [!IMPORTANT]
> El siguiente script debe ejecutarse en el contexto de una cuenta de automatización.
Para obtener una cuenta de automatización personalizada, use el siguiente script:

```azurepowershell
param(
    [Parameter(Mandatory=$true)]
    [String] $VaultResourceId,
    [Parameter(Mandatory=$true)]
    [ValidateSet("Enabled",'Disabled')]
    [Alias("Enabled or Disabled")]
    [String] $AutoUpdateAction,
    [Parameter(Mandatory=$false)]
    [String] $AutomationAccountArmId
)
$SiteRecoveryRunbookName = "Modify-AutoUpdateForVaultForPatner"
$TaskId = [guid]::NewGuid().ToString()
$SubscriptionId = "00000000-0000-0000-0000-000000000000"
$AsrApiVersion = "2018-01-10"
$RunAsConnectionName = "AzureRunAsConnection"
$ArmEndPoint = "https://management.azure.com"
$AadAuthority = "https://login.windows.net/"
$AadAudience = "https://management.core.windows.net/"
$AzureEnvironment = "AzureCloud"
$Timeout = "160"
function Throw-TerminatingErrorMessage
{
        Param
    (
        [Parameter(Mandatory=$true)]
        [String]
        $Message
        )
    throw ("Message: {0}, TaskId: {1}.") -f $Message, $TaskId
}
function Write-Tracing
{
        Param
    (
        [Parameter(Mandatory=$true)]
        [ValidateSet("Informational", "Warning", "ErrorLevel", "Succeeded", IgnoreCase = $true)]
                [String]
        $Level,
        [Parameter(Mandatory=$true)]
        [String]
        $Message,
            [Switch]
        $DisplayMessageToUser
        )
    Write-Output $Message
}
function Write-InformationTracing
{
        Param
    (
        [Parameter(Mandatory=$true)]
        [String]
        $Message
        )
    Write-Tracing -Message $Message -Level Informational -DisplayMessageToUser
}
function ValidateInput()
{
    try
    {
        if(!$VaultResourceId.StartsWith("/subscriptions", [System.StringComparison]::OrdinalIgnoreCase))
        {
            $ErrorMessage = "The vault resource id should start with /subscriptions."
            throw $ErrorMessage
        }
        $Tokens = $VaultResourceId.SubString(1).Split("/")
        if(!($Tokens.Count % 2 -eq 0))
        {
            $ErrorMessage = ("Odd Number of tokens: {0}." -f $Tokens.Count)
            throw $ErrorMessage
        }
        if(!($Tokens.Count/2 -eq 4))
        {
            $ErrorMessage = ("Invalid number of resource in vault ARM id expected:4, actual:{0}." -f ($Tokens.Count/2))
            throw $ErrorMessage
        }
        if($AutoUpdateAction -ieq "Enabled" -and [string]::IsNullOrEmpty($AutomationAccountArmId))
        {
            $ErrorMessage = ("The automation account ARM id should not be null or empty when AutoUpdateAction is enabled.")
            throw $ErrorMessage
        }
    }
    catch
    {
        $ErrorMessage = ("ValidateInput failed with [Exception: {0}]." -f $_.Exception)
        Write-Tracing -Level ErrorLevel -Message $ErrorMessage -DisplayMessageToUser
        Throw-TerminatingErrorMessage -Message $ErrorMessage
    }
}
function Initialize-SubscriptionId()
{
    try
    {
        $Tokens = $VaultResourceId.SubString(1).Split("/")
        $Count = 0
                $ArmResources = @{}
        while($Count -lt $Tokens.Count)
        {
            $ArmResources[$Tokens[$Count]] = $Tokens[$Count+1]
            $Count = $Count + 2
        }
                return $ArmResources["subscriptions"]
    }
    catch
    {
        Write-Tracing -Level ErrorLevel -Message ("Initialize-SubscriptionId: failed with [Exception: {0}]." -f $_.Exception) -DisplayMessageToUser
        throw
    }
}
function Invoke-InternalRestMethod($Uri, $Headers, [ref]$Result)
{
    $RetryCount = 0
    $MaxRetry = 3
    do
    {
        try
        {
            $ResultObject = Invoke-RestMethod -Uri $Uri -Headers $Headers
            ($Result.Value) += ($ResultObject)
            break
        }
        catch
        {
            Write-InformationTracing ("Retry Count: {0}, Exception: {1}." -f $RetryCount, $_.Exception)
            $RetryCount++
            if(!($RetryCount -le $MaxRetry))
            {
                throw
            }
            Start-Sleep -Milliseconds 2000
        }
    }while($true)
}
function Invoke-InternalWebRequest($Uri, $Headers, $Method, $Body, $ContentType, [ref]$Result)
{
    $RetryCount = 0
    $MaxRetry = 3
    do
    {
        try
        {
            $ResultObject = Invoke-WebRequest -Uri $UpdateUrl -Headers $Header -Method 'PATCH' `
                -Body $InputJson  -ContentType "application/json" -UseBasicParsing
            ($Result.Value) += ($ResultObject)
            break
        }
        catch
        {
            Write-InformationTracing ("Retry Count: {0}, Exception: {1}." -f $RetryCount, $_.Exception)
            $RetryCount++
            if(!($RetryCount -le $MaxRetry))
            {
                throw
            }
            Start-Sleep -Milliseconds 2000
        }
    }while($true)
}
function Get-Header([ref]$Header, $AadAudience, $AadAuthority, $RunAsConnectionName){
    try
    {
        $RunAsConnection = Get-AutomationConnection -Name $RunAsConnectionName
        $TenantId = $RunAsConnection.TenantId
        $ApplicationId = $RunAsConnection.ApplicationId
        $CertificateThumbprint = $RunAsConnection.CertificateThumbprint
        $Path = "cert:\CurrentUser\My\{0}" -f $CertificateThumbprint
        $Secret = Get-ChildItem -Path $Path
        $ClientCredential = New-Object Microsoft.IdentityModel.Clients.ActiveDirectory.ClientAssertionCertificate(
                $ApplicationId,
                $Secret)
        # Trim the forward slash from the AadAuthority if it exist.
        $AadAuthority = $AadAuthority.TrimEnd("/")
        $AuthContext = New-Object Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext(
            "{0}/{1}" -f $AadAuthority, $TenantId )
        $AuthenticationResult = $authContext.AcquireToken($AadAudience, $Clientcredential)
        $Header.Value['Content-Type'] = 'application\json'
        $Header.Value['Authorization'] = $AuthenticationResult.CreateAuthorizationHeader()
        $Header.Value["x-ms-client-request-id"] = $TaskId + "/" + (New-Guid).ToString() + "-" + (Get-Date).ToString("u")
    }
    catch
    {
        $ErrorMessage = ("Get-BearerToken: failed with [Exception: {0}]." -f $_.Exception)
        Write-Tracing -Level ErrorLevel -Message $ErrorMessage -DisplayMessageToUser
        Throw-TerminatingErrorMessage -Message $ErrorMessage
    }
}
function Get-ProtectionContainerToBeModified([ref] $ContainerMappingList)
{
    try
    {
        Write-InformationTracing ("Get protection container mappings : {0}." -f $VaultResourceId)
        $ContainerMappingListUrl = $ArmEndPoint + $VaultResourceId + "/replicationProtectionContainerMappings" + "?api-version=" + $AsrApiVersion
        Write-InformationTracing ("Getting the bearer token and the header.")
        Get-Header ([ref]$Header) $AadAudience $AadAuthority $RunAsConnectionName
        $Result = @()
        Invoke-InternalRestMethod -Uri $ContainerMappingListUrl -Headers $header -Result ([ref]$Result)
        $ContainerMappings = $Result[0]
        Write-InformationTracing ("Total retrieved container mappings: {0}." -f $ContainerMappings.Value.Count)
        foreach($Mapping in $ContainerMappings.Value)
        {
            if(($Mapping.properties.providerSpecificDetails -eq $null) -or ($Mapping.properties.providerSpecificDetails.instanceType -ine "A2A"))
            {
                Write-InformationTracing ("Mapping properties: {0}." -f ($Mapping.properties))
                Write-InformationTracing ("Ignoring container mapping: {0} as the provider does not match." -f ($Mapping.Id))
                continue;
            }
            if($Mapping.Properties.State -ine "Paired")
            {
                Write-InformationTracing ("Ignoring container mapping: {0} as the state is not paired." -f ($Mapping.Id))
                continue;
            }
            Write-InformationTracing ("Provider specific details {0}." -f ($Mapping.properties.providerSpecificDetails))
            $MappingAutoUpdateStatus = $Mapping.properties.providerSpecificDetails.agentAutoUpdateStatus
            $MappingAutomationAccountArmId = $Mapping.properties.providerSpecificDetails.automationAccountArmId
            $MappingHealthErrorCount = $Mapping.properties.HealthErrorDetails.Count
            if($AutoUpdateAction -ieq "Enabled" -and
                ($MappingAutoUpdateStatus -ieq "Enabled") -and
                ($MappingAutomationAccountArmId -ieq $AutomationAccountArmId) -and
                ($MappingHealthErrorCount -eq 0))
            {
                Write-InformationTracing ("Provider specific details {0}." -f ($Mapping.properties))
                Write-InformationTracing ("Ignoring container mapping: {0} as the auto update is already enabled and is healthy." -f ($Mapping.Id))
                continue;
            }
            ($ContainerMappingList.Value).Add($Mapping.id)
        }
    }
    catch
    {
        $ErrorMessage = ("Get-ProtectionContainerToBeModified: failed with [Exception: {0}]." -f $_.Exception)
        Write-Tracing -Level ErrorLevel -Message $ErrorMessage -DisplayMessageToUser
        Throw-TerminatingErrorMessage -Message $ErrorMessage
    }
}
$OperationStartTime = Get-Date
$ContainerMappingList = New-Object System.Collections.Generic.List[System.String]
$JobsInProgressList = @()
$JobsCompletedSuccessList = @()
$JobsCompletedFailedList = @()
$JobsFailedToStart = 0
$JobsTimedOut = 0
$Header = @{}
$AzureRMProfile = Get-Module -ListAvailable -Name AzureRM.Profile | Select Name, Version, Path
$AzureRmProfileModulePath = Split-Path -Parent $AzureRMProfile.Path
Add-Type -Path (Join-Path $AzureRmProfileModulePath "Microsoft.IdentityModel.Clients.ActiveDirectory.dll")
$Inputs = ("Tracing inputs VaultResourceId: {0}, Timeout: {1}, AutoUpdateAction: {2}, AutomationAccountArmId: {3}." -f $VaultResourceId, $Timeout, $AutoUpdateAction, $AutomationAccountArmId)
Write-Tracing -Message $Inputs -Level Informational -DisplayMessageToUser
$CloudConfig = ("Tracing cloud configuration ArmEndPoint: {0}, AadAuthority: {1}, AadAudience: {2}." -f $ArmEndPoint, $AadAuthority, $AadAudience)
Write-Tracing -Message $CloudConfig -Level Informational -DisplayMessageToUser
$AutomationConfig = ("Tracing automation configuration RunAsConnectionName: {0}." -f $RunAsConnectionName)
Write-Tracing -Message $AutomationConfig -Level Informational -DisplayMessageToUser
ValidateInput
$SubscriptionId = Initialize-SubscriptionId
Get-ProtectionContainerToBeModified ([ref]$ContainerMappingList)
$Input = @{
  "properties"= @{
    "providerSpecificInput"= @{
        "instanceType" = "A2A"
        "agentAutoUpdateStatus" = $AutoUpdateAction
        "automationAccountArmId" = $AutomationAccountArmId
    }
  }
}
$InputJson = $Input |  ConvertTo-Json
if ($ContainerMappingList.Count -eq 0)
{
    Write-Tracing -Level Succeeded -Message ("Exiting as there are no container mappings to be modified.") -DisplayMessageToUser
    exit
}
Write-InformationTracing ("Container mappings to be updated has been retrieved with count: {0}." -f $ContainerMappingList.Count)
try
{
    Write-InformationTracing ("Start the modify container mapping jobs.")
    ForEach($Mapping in $ContainerMappingList)
    {
    try {
            $UpdateUrl = $ArmEndPoint + $Mapping + "?api-version=" + $AsrApiVersion
            Get-Header ([ref]$Header) $AadAudience $AadAuthority $RunAsConnectionName
            $Result = @()
            Invoke-InternalWebRequest -Uri $UpdateUrl -Headers $Header -Method 'PATCH' `
                -Body $InputJson  -ContentType "application/json" -Result ([ref]$Result)
            $Result = $Result[0]
            $JobAsyncUrl = $Result.Headers['Azure-AsyncOperation']
            Write-InformationTracing ("The modify container mapping job invoked with async url: {0}." -f $JobAsyncUrl)
            $JobsInProgressList += $JobAsyncUrl;
            # Rate controlling the set calls to maximum 60 calls per minute.
            # ASR throttling for set calls is 200 in 1 minute.
            Start-Sleep -Milliseconds 1000
        }
        catch{
            Write-InformationTracing ("The modify container mappings job creation failed for: {0}." -f $Ru)
            Write-InformationTracing $_
            $JobsFailedToStart++
        }
    }
    Write-InformationTracing ("Total modify container mappings has been initiated: {0}." -f $JobsInProgressList.Count)
}
catch
{
    $ErrorMessage = ("Modify container mapping jobs failed with [Exception: {0}]." -f $_.Exception)
    Write-Tracing -Level ErrorLevel -Message $ErrorMessage -DisplayMessageToUser
    Throw-TerminatingErrorMessage -Message $ErrorMessage
}
try
{
    while($JobsInProgressList.Count -ne 0)
    {
        Sleep -Seconds 30
        $JobsInProgressListInternal = @()
        ForEach($JobAsyncUrl in $JobsInProgressList)
        {
            try
            {
                Get-Header ([ref]$Header) $AadAudience $AadAuthority $RunAsConnectionName
                $Result = Invoke-RestMethod -Uri $JobAsyncUrl -Headers $header
                $JobState = $Result.Status
                if($JobState -ieq "InProgress")
                {
                    $JobsInProgressListInternal += $JobAsyncUrl
                }
                elseif($JobState -ieq "Succeeded" -or `
                    $JobState -ieq "PartiallySucceeded" -or `
                    $JobState -ieq "CompletedWithInformation")
                {
                    Write-InformationTracing ("Jobs succeeded with state: {0}." -f $JobState)
                    $JobsCompletedSuccessList += $JobAsyncUrl
                }
                else
                {
                    Write-InformationTracing ("Jobs failed with state: {0}." -f $JobState)
                    $JobsCompletedFailedList += $JobAsyncUrl
                }
            }
            catch
            {
                Write-InformationTracing ("The get job failed with: {0}. Ignoring the exception and retrying the next job." -f $_.Exception)
                # The job on which the tracking failed, will be considered in progress and tried again later.
                $JobsInProgressListInternal += $JobAsyncUrl
            }
            # Rate controlling the get calls to maximum 120 calls each minute.
            # ASR throttling for get calls is 10000 in 60 minutes.
            Start-Sleep -Milliseconds 500
        }
        Write-InformationTracing ("Jobs remaining {0}." -f $JobsInProgressListInternal.Count)
        $CurrentTime = Get-Date
        if($CurrentTime -gt $OperationStartTime.AddMinutes($Timeout))
        {
            Write-InformationTracing ("Tracing modify cloud pairing jobs has timed out.")
            $JobsTimedOut = $JobsInProgressListInternal.Count
            $JobsInProgressListInternal = @()
        }
        $JobsInProgressList = $JobsInProgressListInternal
    }
}
catch
{
    $ErrorMessage = ("Tracking modify cloud pairing jobs failed with [Exception: {0}]." -f $_.Exception)
    Write-Tracing -Level ErrorLevel -Message $ErrorMessage  -DisplayMessageToUser
    Throw-TerminatingErrorMessage -Message $ErrorMessage
}
Write-InformationTracing ("Tracking modify cloud pairing jobs completed.")
Write-InformationTracing ("Modify cloud pairing jobs success: {0}." -f $JobsCompletedSuccessList.Count)
Write-InformationTracing ("Modify cloud pairing jobs failed: {0}." -f $JobsCompletedFailedList.Count)
Write-InformationTracing ("Modify cloud pairing jobs failed to start: {0}." -f $JobsFailedToStart)
Write-InformationTracing ("Modify cloud pairing jobs timedout: {0}." -f $JobsTimedOut)
if($JobsTimedOut -gt  0)
{
    $ErrorMessage = "One or more modify cloud pairing jobs has timedout."
    Write-Tracing -Level ErrorLevel -Message ($ErrorMessage)
    Throw-TerminatingErrorMessage -Message $ErrorMessage
}
elseif($JobsCompletedSuccessList.Count -ne $ContainerMappingList.Count)
{
    $ErrorMessage = "One or more modify cloud pairing jobs failed."
    Write-Tracing -Level ErrorLevel -Message ($ErrorMessage)
    Throw-TerminatingErrorMessage -Message $ErrorMessage
}
Write-Tracing -Level Succeeded -Message ("Modify cloud pairing completed.") -DisplayMessageToUser
```

### <a name="manage-updates-manually"></a>Administrar actualizaciones manualmente

1. Si hay nuevas actualizaciones para la instancia de Mobility Service instalada en sus VM, verá la siguiente notificación: **Hay una nueva actualización del agente de replicación de Site Recovery disponible. Haga clic para instalarla.**

   :::image type="content" source="./media/vmware-azure-install-mobility-service/replicated-item-notif.png" alt-text="Ventana Elementos replicados":::

1. Seleccione la notificación para abrir la página de selección de VM.
1. Elija las VM que quiera actualizar y luego seleccione **Aceptar**. El servicio de actualización de Mobility Service se iniciará por cada VM seleccionada.

   :::image type="content" source="./media/vmware-azure-install-mobility-service/update-okpng.png" alt-text="Lista de máquinas virtuales de elementos replicados":::

## <a name="common-issues-and-troubleshooting"></a>Problemas habituales y soluciones

Si hay un problema con las actualizaciones automáticas, verá una notificación de error en la opción de **problemas de configuración** del panel del almacén.

Si no puede habilitar las actualizaciones automáticas, consulte los siguientes errores comunes y acciones recomendadas:

- **Error**: No tiene permisos para crear una cuenta de ejecución de Azure (entidad de servicio) ni para conceder el rol Colaborador a la entidad de servicio.

  **Acción recomendada**: Asegúrese de que la cuenta de inicio de sesión esté asignada como Colaborador e inténtelo nuevamente. Para más información acerca de la asignación de permisos, consulte la sección sobre permisos necesarios de [Procedimientos: portal para crear una aplicación de Azure AD y una entidad de servicio que puedan acceder a los recursos](../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app).

  Para solucionar la mayoría de los problemas después de habilitar las actualizaciones automáticas, seleccione **Reparar**. Si el botón de reparación no está disponible, consulte el mensaje de error que se muestra en el panel de configuración de la actualización de la extensión.

  :::image type="content" source="./media/azure-to-azure-autoupdate/repair.png" alt-text="Botón de reparación del servicio de Site Recovery en la configuración de actualización de la extensión":::

- **Error**: la cuenta de ejecución no tiene permiso para obtener acceso al recurso de Recovery Services.

  **Acción recomendada**: elimine y [vuelva a crear la cuenta de ejecución](../automation/manage-runas-account.md). O bien, asegúrese de que la aplicación de Azure Active Directory de la cuenta de ejecución de Automation puede acceder al recurso de Recovery Services.

- **Error**: no se encuentra la cuenta de ejecución. Es posible que la huella digital del certificado y la conexión no sea idéntica, o que alguno de los elementos siguientes se haya eliminado o no se haya creado: la aplicación Azure Active Directory, la entidad de servicio, el rol, el recurso de certificado de Automation o el recurso de conexión de Automation.

  **Acción recomendada**: elimine y [vuelva a crear la cuenta de ejecución](../automation/manage-runas-account.md).

- **Error**: La ejecución de Azure como un certificado que usa la cuenta de automatización está a punto de caducar.

  El certificado autofirmado que creó para la cuenta de ejecución expira en un año a partir de la fecha de creación. Se puede renovar en cualquier momento antes de que expire. Si se ha registrado para recibir notificaciones por correo electrónico, también recibirá correos electrónicos cuando deba realizar cualquier una acción. Este error se mostrará dos meses antes de la fecha de expiración y cambiará a un error crítico si el certificado ha expirado. Una vez que el certificado haya expirado, la actualización automática no funcionará hasta que renueve el mismo.

  **Acción recomendada**: Haga clic en **Reparar** y luego en **Renovar certificado** para resolver este problema.

  :::image type="content" source="./media/azure-to-azure-autoupdate/automation-account-renew-runas-certificate.PNG" alt-text="renew-cert":::

  > [!NOTE]
  > Después de renovar el certificado, actualice la página para mostrar el estado actual.
