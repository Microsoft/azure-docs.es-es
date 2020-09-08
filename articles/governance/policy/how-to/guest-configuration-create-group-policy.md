---
title: Cómo crear definiciones de directivas de configuración de invitado a partir de la directiva de grupo línea de base para Windows
description: Obtenga información sobre cómo convertir una directiva de grupo de la línea base de seguridad de Windows Server 2019 en una definición de directiva.
ms.date: 08/17/2020
ms.topic: how-to
ms.openlocfilehash: 58fe4fa3e5056192fa5febe4883a1457d130871b
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88547775"
---
# <a name="how-to-create-guest-configuration-policy-definitions-from-group-policy-baseline-for-windows"></a>Cómo crear definiciones de directivas de configuración de invitado a partir de la directiva de grupo línea de base para Windows

Antes de crear definiciones de directivas personalizadas, es conveniente leer la información conceptual general en [Información sobre Guest Configuration de Azure Policy](../concepts/guest-configuration.md). Para obtener información sobre cómo crear definiciones de directivas de configuración de invitado personalizadas para Linux, consulte la página [Cómo crear una directiva de configuración de invitados para Linux](./guest-configuration-create-linux.md). Para obtener información sobre cómo crear definiciones de directivas de configuración de invitado personalizadas para Windows, consulte la página [Cómo crear una directiva de configuración de invitados para Windows](./guest-configuration-create.md).

Durante la auditoría en Windows, Configuración de invitado usa un módulo de recursos [Desired State Configuration](/powershell/scripting/dsc/overview/overview) (DSC) para crear el archivo de configuración. La configuración de DSC define la condición en la que debe estar la máquina. Si la evaluación de la configuración es **no compatible**, se desencadena el efecto *auditIfNotExists* de la directiva.
La [configuración de invitado de Azure Policy](../concepts/guest-configuration.md) solo realiza la auditoría de la configuración dentro de máquinas.

> [!IMPORTANT]
> Las definiciones de directivas personalizadas con la configuración de invitados son una característica en vista previa (GB).
>
> La extensión de configuración de invitado es necesaria para realizar auditorías en las máquinas virtuales de Azure. Para implementar la extensión a gran escala en todas las máquinas Windows, asigne las siguientes definiciones de directiva:
> - [Implemente los requisitos previos para habilitar la directiva de configuración de invitado en VM de Windows.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0ecd903d-91e7-4726-83d3-a229d7f2e293)

La comunidad de DSC ha publicado el [módulo BaselineManagement](https://github.com/microsoft/BaselineManagement) para convertir las plantillas exportadas de la directiva de grupo al formato DSC. Junto con el cmdlet GuestConfiguration, el módulo BaselineManagement crea el paquete de configuración de invitado de Azure Policy para Windows a partir del contenido de directiva de grupo. Para obtener más información sobre el uso del módulo BaselineManagement, consulte el artículo [Inicio rápido: Conversión de directiva de grupo en DSC](/powershell/scripting/dsc/quickstarts/gpo-quickstart).

En esta guía, recorremos el proceso de creación de un paquete de configuración de invitado de Azure Policy a partir de un objeto de directiva de grupo (GPO). Aunque en el tutorial se describe la conversión de la línea base de seguridad de Windows Server 2019, se puede aplicar el mismo proceso a otros GPO.  

## <a name="download-windows-server-2019-security-baseline-and-install-related-powershell-modules"></a>Descargue la línea de base de seguridad de Windows Server 2019 e instale los módulos de PowerShell relacionados

Para instalar el **DSC**, **GuestConfiguration**, **administración de línea de base** y los módulos de Azure relacionados en PowerShell:

1. En una secuencia de comandos de PowerShell, ejecute el siguiente comando:

   ```azurepowershell-interactive
   # Install the BaselineManagement module, Guest Configuration DSC resource module, and relevant Azure modules from PowerShell Gallery
   Install-Module az.resources, az.policyinsights, az.storage, guestconfiguration, gpregistrypolicyparser, securitypolicydsc, auditpolicydsc, baselinemanagement -scope currentuser -Repository psgallery -AllowClobber
   ```

1. Cree un directorio y descargue la línea base de seguridad de Windows Server 2019 desde el kit de herramientas de cumplimiento de seguridad de Windows.

   ```azurepowershell-interactive
   # Download the 2019 Baseline files from https://docs.microsoft.com/windows/security/threat-protection/security-compliance-toolkit-10
   New-Item -Path 'C:\git\policyfiles\downloads' -Type Directory
   Invoke-WebRequest -Uri 'https://download.microsoft.com/download/8/5/C/85C25433-A1B0-4FFA-9429-7E023E7DA8D8/Windows%2010%20Version%201909%20and%20Windows%20Server%20Version%201909%20Security%20Baseline.zip' -Out C:\git\policyfiles\downloads\Server2019Baseline.zip
   ```

1. Desbloquee y expanda la línea de base de la instancia de Server 2019 descargada.

   ```azurepowershell-interactive
   Unblock-File C:\git\policyfiles\downloads\Server2019Baseline.zip
   Expand-Archive -Path C:\git\policyfiles\downloads\Server2019Baseline.zip -DestinationPath C:\git\policyfiles\downloads\
   ```

1. Valide el contenido de línea de base de Server 2019 mediante **MapGuidsToGpoNames.ps1**.

   ```azurepowershell-interactive
   # Show content details of downloaded GPOs
   C:\git\policyfiles\downloads\Scripts\Tools\MapGuidsToGpoNames.ps1 -rootdir C:\git\policyfiles\downloads\GPOs\ -Verbose
   ```

## <a name="convert-from-group-policy-to-azure-policy-guest-configuration"></a>Conversión de directiva de grupo a configuración de invitado de Azure Policy

Después, convierta la línea de base de la instancia de Server 2019 descargada en un paquete de configuración de invitado mediante los módulos configuración de invitado y administración de línea de base.

1. Convierta la directiva de grupo a Desired State Configuration mediante el módulo de administración de línea de base.

   ```azurepowershell-interactive
   ConvertFrom-GPO -Path 'C:\git\policyfiles\downloads\GPOs\{3657C7A2-3FF3-4C21-9439-8FDF549F1D68}\' -OutputPath 'C:\git\policyfiles\' -OutputConfigurationScript -Verbose
   ```

1. Renombre, reformatee y ejecute los scripts convertidos antes de crear un paquete de contenido de directivas.

   ```azurepowershell-interactive
   Rename-Item -Path C:\git\policyfiles\DSCFromGPO.ps1 -NewName C:\git\policyfiles\Server2019Baseline.ps1
   (Get-Content -Path C:\git\policyfiles\Server2019Baseline.ps1).Replace('DSCFromGPO', 'Server2019Baseline') | Set-Content -Path C:\git\policyfiles\Server2019Baseline.ps1
   (Get-Content -Path C:\git\policyfiles\Server2019Baseline.ps1).Replace('PSDesiredStateConfiguration', 'PSDscResources') | Set-Content -Path C:\git\policyfiles\Server2019Baseline.ps1
   C:\git\policyfiles\Server2019Baseline.ps1
   ```

1. Cree un paquete de contenido de configuración de invitado de Azure Policy.

   ```azurepowershell-interactive
   New-GuestConfigurationPackage -Name Server2019Baseline -Configuration c:\git\policyfiles\localhost.mof -Verbose
   ```

## <a name="create-azure-policy-guest-configuration"></a>Creación de configuración de invitado de Azure Policy

El siguiente paso consiste en publicar el archivo en el almacenamiento de blobs. 

1. El script siguiente contiene una función que puede usar para automatizar esta tarea. Nota: Los comandos usados en la función `publish` requieren el módulo `Az.Storage`.

   ```azurepowershell-interactive
    function Publish-Configuration {
        param(
        [Parameter(Mandatory=$true)]
        $resourceGroup,
        [Parameter(Mandatory=$true)]
        $storageAccountName,
        [Parameter(Mandatory=$true)]
        $storageContainerName,
        [Parameter(Mandatory=$true)]
        $filePath,
        [Parameter(Mandatory=$true)]
        $blobName
        )

        # Get Storage Context
        $Context = Get-AzStorageAccount -ResourceGroupName $resourceGroup `
            -Name $storageAccountName | `
            ForEach-Object { $_.Context }

        # Upload file
        $Blob = Set-AzStorageBlobContent -Context $Context `
            -Container $storageContainerName `
            -File $filePath `
            -Blob $blobName `
            -Force

        # Get url with SAS token
        $StartTime = (Get-Date)
        $ExpiryTime = $StartTime.AddYears('3')  # THREE YEAR EXPIRATION
        $SAS = New-AzStorageBlobSASToken -Context $Context `
            -Container $storageContainerName `
            -Blob $blobName `
            -StartTime $StartTime `
            -ExpiryTime $ExpiryTime `
            -Permission rl `
            -FullUri

        # Output
        return $SAS
    }
   ```

1. Cree parámetros para definir el grupo de recursos único, la cuenta de almacenamiento y el contenedor. 
   
   ```azurepowershell-interactive
    # Replace the $resourceGroup, $storageAccount, and $storageContainer values below.
    $resourceGroup = 'rfc_customguestconfig'
    $storageAccount = 'guestconfiguration'
    $storageContainer = 'content'
    $path = 'c:\git\policyfiles\Server2019Baseline\Server2019Baseline.zip'
    $blob = 'Server2019Baseline.zip' 
    ```

1. Use la función Publish con los parámetros asignados para publicar el paquete de configuración de invitado en el almacenamiento de blobs público.


   ```azurepowershell-interactive
   $PublishConfigurationSplat = @{
       resourceGroup = $resourceGroup
       storageAccountName = $storageAccount
       storageContainerName = $storageContainer
       filePath = $path
       blobName = $blob
       FullUri = $true
   }
   $uri = Publish-Configuration @PublishConfigurationSplat
    ```
1. Una vez que se ha creado y cargado un paquete de directivas personalizadas de Configuración de invitado, cree la definición de la directiva de Configuración de invitado. Use el cmdlet `New-GuestConfigurationPolicy` para crear la configuración de invitado.

   ```azurepowershell-interactive
    $NewGuestConfigurationPolicySplat = @{
        ContentUri = $Uri 
        DisplayName = 'Server 2019 Configuration Baseline' 
        Description 'Validation of using a completely custom baseline configuration for Windows VMs' 
        Path = 'C:\git\policyfiles\policy'  
        Platform = Windows 
        }
   New-GuestConfigurationPolicy @NewGuestConfigurationPolicySplat
   ```
    
1. Publique las definiciones de directivas con el cmdlet `Publish-GuestConfigurationPolicy`. El cmdlet solo tiene el parámetro **Path** que apunta a la ubicación de los archivos JSON que creó `New-GuestConfigurationPolicy`. Para ejecutar el comando Publish, necesita acceso para crear definiciones de directivas en Azure. Los requisitos de autorización específicos se documentan en la página [Información general de Azure Policy](../overview.md#getting-started). El mejor rol integrado es **Colaborador de la directiva de recursos**.

   ```azurepowershell-interactive
   Publish-GuestConfigurationPolicy -Path C:\git\policyfiles\policy\ -Verbose
   ```

## <a name="assign-guest-configuration-policy-definition"></a>Asignación de la definición de directiva de configuración de invitado

Con la directiva creada en Azure, el último paso es asignar la iniciativa. Consulte cómo puede asignar la iniciativa con el [portal](../assign-policy-portal.md), la [CLI de Azure](../assign-policy-azurecli.md) y [Azure PowerShell](../assign-policy-powershell.md).

> [!IMPORTANT]
> Las definiciones de directivas de configuración de invitados **siempre** se deben asignar mediante la iniciativa que combina las directivas _AuditIfNotExists_ y _DeployIfNotExists_. Si solo se asigna la directiva _AuditIfNotExists_, los requisitos previos no se implementan y la directiva siempre muestra que los servidores "0" son compatibles.

La asignación de una definición de directiva con el efecto _DeployIfNotExists_ requiere un nivel de acceso adicional. Para conceder el privilegio mínimo, puede crear una definición de rol personalizada que amplíe **Colaborador de la directiva de recursos**. En el ejemplo siguiente se crea un rol denominado **Resource Policy Contributor DINE** con el permiso adicional _Microsoft.Authorization/roleAssignments/write_.

   ```azurepowershell-interactive
   $subscriptionid = '00000000-0000-0000-0000-000000000000'
   $role = Get-AzRoleDefinition "Resource Policy Contributor"
   $role.Id = $null
   $role.Name = "Resource Policy Contributor DINE"
   $role.Description = "Can assign Policies that require remediation."
   $role.Actions.Clear()
   $role.Actions.Add("Microsoft.Authorization/roleAssignments/write")
   $role.AssignableScopes.Clear()
   $role.AssignableScopes.Add("/subscriptions/$subscriptionid")
   New-AzRoleDefinition -Role $role
   ```

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre la auditoría de VM con la [configuración de invitados](../concepts/guest-configuration.md).
- Obtenga información acerca de cómo se pueden [crear directivas mediante programación](./programmatically-create.md).
- Obtenga información sobre cómo [obtener datos de cumplimiento](./get-compliance-data.md).
