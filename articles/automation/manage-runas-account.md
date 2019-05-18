---
title: Administración de cuentas de ejecución de Azure Automation
description: En este artículo se describe cómo administrar las cuenta de ejecución con PowerShell o desde el portal.
services: automation
ms.service: automation
ms.subservice: shared-capabilities
author: georgewallace
ms.author: gwallace
ms.date: 05/03/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 2a104c9e41ef1843c377b0406b499ffae504db97
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65595682"
---
# <a name="manage-azure-automation-run-as-accounts"></a>Administración de cuentas de ejecución de Azure Automation

Las cuentas de ejecución en Azure Automation se usan para proporcionar autenticación para administrar los recursos de Azure con los cmdlets de Azure.

Cuando se crea una cuenta de ejecución, crea un nuevo usuario de la entidad de servicio en Azure Active Directory y asigna el rol Colaborador a este usuario en el nivel de suscripción. Para los runbooks que utilizan instancias de Hybrid Runbook Worker en máquinas virtuales de Azure, puede utilizar [Managed Identities for Azure Resources](automation-hrw-run-runbooks.md#managed-identities-for-azure-resources) en lugar de cuentas de ejecución para autenticarse en sus recursos de Azure.

Existen dos tipos de cuentas de ejecución:

* **Cuenta de ejecución de Azure**: esta cuenta se usa para administrar recursos del modelo de implementación de Resource Manager.
  * Crea una aplicación de Azure AD con un certificado autofirmado, crea una cuenta de entidad servicio para la aplicación en Azure AD y asigna el rol Colaborador para esta cuenta en la suscripción actual. Puede cambiar esta configuración a Propietario o cualquier otro rol. Para más información, consulte [Control de acceso basado en rol en Azure Automation](automation-role-based-access-control.md).
  * Crea un recurso de certificado de Automation llamado *AzureRunAsCertificate* en la cuenta de Automation especificada. El recurso de certificado contiene la clave privada del certificado que usa la aplicación de Azure AD.
  * Crea un recurso de conexión de Automation llamado *AzureRunAsConnection* en la cuenta de Automation especificada. El recurso de conexión contiene el id. de aplicación, el id. de inquilino, el id. de suscripción y la huella digital de certificado.

* **Cuenta de ejecución de Azure clásico**: esta cuenta se usa para administrar recursos del modelo de implementación clásico.
  * Crea un certificado de administración en la suscripción
  * Crea un recurso de certificado de Automation llamado *AzureClassicRunAsCertificate* en la cuenta de Automation especificada. El recurso de certificado contiene la clave privada del certificado que usa el certificado de administración.
  * Crea un recurso de conexión de Automation llamado *AzureClassicRunAsConnection* en la cuenta de Automation especificada. El recurso de conexión contiene el nombre de la suscripción, el id. de suscripción y el nombre del recurso de certificado.
  * Debe ser Coadministrador en la suscripción para crear o renovar
  
  > [!NOTE]
  > Las suscripciones de Proveedor de soluciones en la nube de Azure (Azure CSP) solo admiten el modelo de Azure Resource Manager, los servicios que no sean de Azure Resource Manager no están disponibles en el programa. Cuando se usa una suscripción a CSP la Cuenta de ejecución de Azure clásico no se crea. Se sigue creando la Cuenta de ejecución de Azure. Para más información acerca de las suscripciones de CSP, consulte [Servicios disponibles en las suscripciones de CSP](https://docs.microsoft.com/azure/cloud-solution-provider/overview/azure-csp-available-services#comments).

  > [!NOTE]
  > La entidad de servicio para una cuenta de ejecución no tiene permisos para leer Azure Active Directory de forma predeterminada. Si desea agregar permisos para leer o administrar Azure Active directory, deberá conceder ese permiso en el servicio principal en **permisos de API**. Para obtener más información, consulte [agregar permisos para tener acceso a las API web](../active-directory/develop/quickstart-configure-app-access-web-apis.md#add-permissions-to-access-web-apis).

## <a name="permissions"></a>Permisos para configurar cuentas de ejecución

Para crear o actualizar una cuenta de ejecución, debe tener los permisos y privilegios específicos. Un administrador global o coadministrador pueden completar todas las tareas. En una situación en la que tenga separación de tareas, la siguiente tabla muestra una lista de las tareas, el cmdlet equivalente y los permisos necesarios:

|Tarea|Cmdlet  |Permisos mínimos  |Donde se establecen los permisos|
|---|---------|---------|---|
|Crear una aplicación de Azure AD|[New-AzureRmADApplication](/powershell/module/azurerm.resources/new-azurermadapplication)     | Rol de desarrollador de aplicaciones<sup>1</sup>        |[Azure Active Directory](../active-directory/develop/howto-create-service-principal-portal.md#required-permissions)</br>Inicio > Azure Active Directory > Registros de aplicaciones |
|Agregar una credencial a la aplicación.|[New-AzureRmADAppCredential](/powershell/module/AzureRM.Resources/New-AzureRmADAppCredential)     | Administrador de la aplicación o ADMINISTRADOR GLOBAL<sup>1</sup>         |[Azure Active Directory](../active-directory/develop/howto-create-service-principal-portal.md#required-permissions)</br>Inicio > Azure Active Directory > Registros de aplicaciones|
|Crear y obtener una entidad de servicio de Azure AD|[New-AzureRMADServicePrincipal](/powershell/module/AzureRM.Resources/New-AzureRmADServicePrincipal)</br>[Get-AzureRmADServicePrincipal](/powershell/module/AzureRM.Resources/Get-AzureRmADServicePrincipal)     | Administrador de la aplicación o administrador global        |[Azure Active Directory](../active-directory/develop/howto-create-service-principal-portal.md#required-permissions)</br>Inicio > Azure Active Directory > Registros de aplicaciones|
|Asignar u obtener el rol de RBAC para la entidad de seguridad especificada|[New-AzureRMRoleAssignment](/powershell/module/AzureRM.Resources/New-AzureRmRoleAssignment)</br>[Get-AzureRMRoleAssignment](/powershell/module/AzureRM.Resources/Get-AzureRmRoleAssignment)      | Administrador de acceso de usuario o propietario        | [Suscripción](../role-based-access-control/role-assignments-portal.md)</br>Inicio > Suscripciones > \<nombre de la suscripción\> -Control de acceso (IAM)|
|Crear o quitar un certificado de Automation|[New-AzureRmAutomationCertificate](/powershell/module/AzureRM.Automation/New-AzureRmAutomationCertificate)</br>[Remove-AzureRmAutomationCertificate](/powershell/module/AzureRM.Automation/Remove-AzureRmAutomationCertificate)     | Colaborador en el grupo de recursos         |Grupo de recursos de la cuenta de Automation|
|Crear o quitar una conexión de Automation|[New-AzureRmAutomationConnection](/powershell/module/AzureRM.Automation/New-AzureRmAutomationConnection)</br>[Remove-AzureRmAutomationConnection](/powershell/module/AzureRM.Automation/Remove-AzureRmAutomationConnection)|Colaborador en el grupo de recursos |Grupo de recursos de la cuenta de Automation|

<sup>1</sup> Los usuarios que no son administradores en el inquilino de Azure AD pueden [registrar aplicaciones de AD](../active-directory/develop/howto-create-service-principal-portal.md#required-permissions) si la opción **Los usuarios pueden registrar aplicaciones** del inquilino de Azure AD en la página **Configuración de usuario** está establecida en **Sí**. Si se establecen en **No**, el usuario que realice esta acción debe ser administrador global de Azure AD.

Si no es miembro de la instancia de Active Directory de la suscripción antes de que se le agregue al rol de administrador global o coadministrador de esta última, se le agregará como invitado. En este caso, recibirá una advertencia `You do not have permissions to create…` en la página **Agregar cuenta de Automation**. Los usuarios que primero se agregaron al rol de administrador global/coadministrador se pueden quitar de la instancia de Active Directory de la suscripción y volverse a agregar para convertirlos en usuarios completos en Active Directory. Para comprobar esta situación, en el panel de **Azure Active Directory** de Azure Portal, seleccione **Usuarios y grupos**, **All Users** y, después de seleccionar el usuario específico, seleccione **Perfil**. El valor del atributo **Tipo de usuario** del perfil de los usuarios no debería ser **Invitado**.

## <a name="permissions-classic"></a>Permisos para configurar las cuentas de identificación clásica

Para configurar o renovar las cuentas de identificación clásica, debe tener la **Coadministrador** rol en el nivel de suscripción. Para más información acerca de los permisos del modelo clásico, consulte [los administradores de suscripción de Azure clásico](../role-based-access-control/classic-administrators.md#add-a-co-administrator).

## <a name="create-a-run-as-account-in-the-portal"></a>Creación de una cuenta de ejecución en el portal

En esta sección, realizará los pasos que se describen a continuación para actualizar su cuenta de Azure Automation en Azure Portal. Las cuentas de ejecución y de ejecución clásica se crean de forma individual. Si no es necesario administrar los recursos clásicos, basta con crear la cuenta de ejecución de Azure.  

1. Inicie sesión en Azure Portal con una cuenta que sea miembro del rol Administradores de suscripciones y coadministrador de la suscripción.
2. En Azure Portal, haga clic en **Todos los servicios**. En la lista de recursos, escriba **Automation**. Cuando comience a escribir, la lista se filtrará en función de la entrada. Seleccione **Cuentas de Automation**.
3. En la página **Cuentas de Automation**, seleccione su cuenta de Automation en la lista Cuentas de Automation.
4. En el panel izquierdo, seleccione **Cuentas de ejecución** en la sección **Configuración de la cuenta**.  
5. En función de la cuenta que necesite, seleccione **Cuenta de ejecución de Azure** o **Cuenta de ejecución de Azure clásica**. Después de seleccionar **Agregar cuenta de ejecución de Azure** o **Agregar cuenta de ejecución de Azure clásico**, aparece el panel. Revise la información general y haga clic en **Crear** para continuar con la creación de la cuenta de ejecución.  
6. Mientras Azure crea la cuenta de ejecución, se puede seguir el progreso en **Notificaciones** en el menú. También se muestra un banner que indica que se está creando la cuenta. Este proceso puede tardar unos minutos en completarse.  

## <a name="create-run-as-account-using-powershell"></a>Creación de una cuenta de ejecución con PowerShell

## <a name="prerequisites"></a>Requisitos previos

En la lista siguiente se proporcionan los requisitos para crear una cuenta de ejecución en PowerShell:

* Windows 10 o Windows Server 2016 con módulos de Azure Resource Manager 3.4.1 y versiones posteriores. El script de PowerShell no admite versiones anteriores de Windows.
* Azure PowerShell 1.0 y versiones superiores. Para más información sobre la versión 1.0 de PowerShell, consulte [Instalación y configuración de Azure PowerShell](/powershell/azureps-cmdlets-docs).
* Una cuenta de Automation, a la que se hace referencia como el valor de los parámetros *–AutomationAccountName* y *-ApplicationDisplayName*.
* Permisos equivalentes a lo que se muestra en [Permisos para configurar cuentas de ejecución](#permissions)

Para obtener los valores de *SubscriptionID*, *ResourceGroup* y *AutomationAccountName*, que son parámetros necesarios para el script, siga estos pasos:

1. En Azure Portal, haga clic en **Todos los servicios**. En la lista de recursos, escriba **Automation**. Cuando comience a escribir, la lista se filtrará en función de la entrada. Seleccione **Cuentas de Automation**.
1. En la página Cuenta de Automation, seleccione su cuenta de Automation y, en **Configuración de la cuenta**, seleccione **Propiedades**.  
1. Anote los valores de **Id. de suscripción**, **Nombre** y **Grupo de recursos** en la página **Propiedades**.

   ![Página "Propiedades" de la cuenta de Automation](media/manage-runas-account/automation-account-properties.png)

Este script de PowerShell incluye compatibilidad con las siguientes configuraciones:

* Creación de una cuenta de ejecución mediante un certificado autofirmado.
* Creación de una cuenta de ejecución y una cuenta de ejecución clásica mediante un certificado autofirmado.
* Cree una cuenta de ejecución y una cuenta de ejecución clásica mediante un certificado emitido por su entidad de certificación (CA) empresarial.
* Creación de una cuenta de ejecución y una cuenta de ejecución clásica mediante un certificado autofirmado en la nube de Azure Government.

>[!NOTE]
> Si selecciona cualquiera de las opciones para crear una cuenta de ejecución clásica, cargue el certificado público (extensión de nombre del archivo .cer) en el almacén de administración para la suscripción en la que se creó la cuenta de Automation.

1. Guarde el script siguiente en el equipo. En este ejemplo, guárdelo con el nombre *New-RunAsAccount.ps1*.

   El script usa varios cmdlets de Azure Resource Manager para crear recursos. La siguiente tabla muestra los cmdlets y los permisos necesarios.

    ```powershell
    #Requires -RunAsAdministrator
    Param (
        [Parameter(Mandatory = $true)]
        [String] $ResourceGroup,

        [Parameter(Mandatory = $true)]
        [String] $AutomationAccountName,

        [Parameter(Mandatory = $true)]
        [String] $ApplicationDisplayName,

        [Parameter(Mandatory = $true)]
        [String] $SubscriptionId,

        [Parameter(Mandatory = $true)]
        [Boolean] $CreateClassicRunAsAccount,

        [Parameter(Mandatory = $true)]
        [String] $SelfSignedCertPlainPassword,

        [Parameter(Mandatory = $false)]
        [string] $EnterpriseCertPathForRunAsAccount,

        [Parameter(Mandatory = $false)]
        [String] $EnterpriseCertPlainPasswordForRunAsAccount,

        [Parameter(Mandatory = $false)]
        [String] $EnterpriseCertPathForClassicRunAsAccount,

        [Parameter(Mandatory = $false)]
        [String] $EnterpriseCertPlainPasswordForClassicRunAsAccount,

        [Parameter(Mandatory = $false)]
        [ValidateSet("AzureCloud", "AzureUSGovernment")]
        [string]$EnvironmentName = "AzureCloud",

        [Parameter(Mandatory = $false)]
        [int] $SelfSignedCertNoOfMonthsUntilExpired = 12
    )

    function CreateSelfSignedCertificate([string] $certificateName, [string] $selfSignedCertPlainPassword,
        [string] $certPath, [string] $certPathCer, [string] $selfSignedCertNoOfMonthsUntilExpired ) {
        $Cert = New-SelfSignedCertificate -DnsName $certificateName -CertStoreLocation cert:\LocalMachine\My `
            -KeyExportPolicy Exportable -Provider "Microsoft Enhanced RSA and AES Cryptographic Provider" `
            -NotAfter (Get-Date).AddMonths($selfSignedCertNoOfMonthsUntilExpired) -HashAlgorithm SHA256

        $CertPassword = ConvertTo-SecureString $selfSignedCertPlainPassword -AsPlainText -Force
        Export-PfxCertificate -Cert ("Cert:\localmachine\my\" + $Cert.Thumbprint) -FilePath $certPath -Password $CertPassword -Force | Write-Verbose
        Export-Certificate -Cert ("Cert:\localmachine\my\" + $Cert.Thumbprint) -FilePath $certPathCer -Type CERT | Write-Verbose
    }

    function CreateServicePrincipal([System.Security.Cryptography.X509Certificates.X509Certificate2] $PfxCert, [string] $applicationDisplayName) {  
        $keyValue = [System.Convert]::ToBase64String($PfxCert.GetRawCertData())
        $keyId = (New-Guid).Guid

        # Create an Azure AD application, AD App Credential, AD ServicePrincipal

        # Requires Application Developer Role, but works with Application administrator or GLOBAL ADMIN
        $Application = New-AzureRmADApplication -DisplayName $ApplicationDisplayName -HomePage ("http://" + $applicationDisplayName) -IdentifierUris ("http://" + $keyId) 
        # Requires Application administrator or GLOBAL ADMIN
        $ApplicationCredential = New-AzureRmADAppCredential -ApplicationId $Application.ApplicationId -CertValue $keyValue -StartDate $PfxCert.NotBefore -EndDate $PfxCert.NotAfter
        # Requires Application administrator or GLOBAL ADMIN
        $ServicePrincipal = New-AzureRMADServicePrincipal -ApplicationId $Application.ApplicationId 
        $GetServicePrincipal = Get-AzureRmADServicePrincipal -ObjectId $ServicePrincipal.Id

        # Sleep here for a few seconds to allow the service principal application to become active (ordinarily takes a few seconds)
        Sleep -s 15
        # Requires User Access Administrator or Owner.
        $NewRole = New-AzureRMRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $Application.ApplicationId -ErrorAction SilentlyContinue
        $Retries = 0;
        While ($NewRole -eq $null -and $Retries -le 6) {
            Sleep -s 10
            New-AzureRMRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $Application.ApplicationId | Write-Verbose -ErrorAction SilentlyContinue
            $NewRole = Get-AzureRMRoleAssignment -ServicePrincipalName $Application.ApplicationId -ErrorAction SilentlyContinue
            $Retries++;
        }
        return $Application.ApplicationId.ToString();
    }

    function CreateAutomationCertificateAsset ([string] $resourceGroup, [string] $automationAccountName, [string] $certifcateAssetName, [string] $certPath, [string] $certPlainPassword, [Boolean] $Exportable) {
        $CertPassword = ConvertTo-SecureString $certPlainPassword -AsPlainText -Force   
        Remove-AzureRmAutomationCertificate -ResourceGroupName $resourceGroup -AutomationAccountName $automationAccountName -Name $certifcateAssetName -ErrorAction SilentlyContinue
        New-AzureRmAutomationCertificate -ResourceGroupName $resourceGroup -AutomationAccountName $automationAccountName -Path $certPath -Name $certifcateAssetName -Password $CertPassword -Exportable:$Exportable  | write-verbose
    }

    function CreateAutomationConnectionAsset ([string] $resourceGroup, [string] $automationAccountName, [string] $connectionAssetName, [string] $connectionTypeName, [System.Collections.Hashtable] $connectionFieldValues ) {
        Remove-AzureRmAutomationConnection -ResourceGroupName $resourceGroup -AutomationAccountName $automationAccountName -Name $connectionAssetName -Force -ErrorAction SilentlyContinue
        New-AzureRmAutomationConnection -ResourceGroupName $ResourceGroup -AutomationAccountName $automationAccountName -Name $connectionAssetName -ConnectionTypeName $connectionTypeName -ConnectionFieldValues $connectionFieldValues
    }

    Import-Module AzureRM.Profile
    Import-Module AzureRM.Resources

    $AzureRMProfileVersion = (Get-Module AzureRM.Profile).Version
    if (!(($AzureRMProfileVersion.Major -ge 3 -and $AzureRMProfileVersion.Minor -ge 4) -or ($AzureRMProfileVersion.Major -gt 3))) {
        Write-Error -Message "Please install the latest Azure PowerShell and retry. Relevant doc url : https://docs.microsoft.com/powershell/azureps-cmdlets-docs/ "
        return
    }

    # To use the new Az modules to create your Run As accounts please uncomment the following lines and ensure you comment out the previous 8 lines that import the AzureRM modules to avoid any issues. To learn about about using Az modules in your Automation Account see https://docs.microsoft.com/azure/automation/az-modules

    # Import-Module Az.Automation
    # Enable-AzureRmAlias


    Connect-AzureRmAccount -Environment $EnvironmentName 
    $Subscription = Select-AzureRmSubscription -SubscriptionId $SubscriptionId

    # Create a Run As account by using a service principal
    $CertifcateAssetName = "AzureRunAsCertificate"
    $ConnectionAssetName = "AzureRunAsConnection"
    $ConnectionTypeName = "AzureServicePrincipal"

    if ($EnterpriseCertPathForRunAsAccount -and $EnterpriseCertPlainPasswordForRunAsAccount) {
        $PfxCertPathForRunAsAccount = $EnterpriseCertPathForRunAsAccount
        $PfxCertPlainPasswordForRunAsAccount = $EnterpriseCertPlainPasswordForRunAsAccount
    }
    else {
        $CertificateName = $AutomationAccountName + $CertifcateAssetName
        $PfxCertPathForRunAsAccount = Join-Path $env:TEMP ($CertificateName + ".pfx")
        $PfxCertPlainPasswordForRunAsAccount = $SelfSignedCertPlainPassword
        $CerCertPathForRunAsAccount = Join-Path $env:TEMP ($CertificateName + ".cer")
        CreateSelfSignedCertificate $CertificateName $PfxCertPlainPasswordForRunAsAccount $PfxCertPathForRunAsAccount $CerCertPathForRunAsAccount $SelfSignedCertNoOfMonthsUntilExpired
    }

    # Create a service principal
    $PfxCert = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList @($PfxCertPathForRunAsAccount, $PfxCertPlainPasswordForRunAsAccount)
    $ApplicationId = CreateServicePrincipal $PfxCert $ApplicationDisplayName

    # Create the Automation certificate asset
    CreateAutomationCertificateAsset $ResourceGroup $AutomationAccountName $CertifcateAssetName $PfxCertPathForRunAsAccount $PfxCertPlainPasswordForRunAsAccount $true

    # Populate the ConnectionFieldValues
    $SubscriptionInfo = Get-AzureRmSubscription -SubscriptionId $SubscriptionId
    $TenantID = $SubscriptionInfo | Select TenantId -First 1
    $Thumbprint = $PfxCert.Thumbprint
    $ConnectionFieldValues = @{"ApplicationId" = $ApplicationId; "TenantId" = $TenantID.TenantId; "CertificateThumbprint" = $Thumbprint; "SubscriptionId" = $SubscriptionId}

    # Create an Automation connection asset named AzureRunAsConnection in the Automation account. This connection uses the service principal.
    CreateAutomationConnectionAsset $ResourceGroup $AutomationAccountName $ConnectionAssetName $ConnectionTypeName $ConnectionFieldValues

    if ($CreateClassicRunAsAccount) {
        # Create a Run As account by using a service principal
        $ClassicRunAsAccountCertifcateAssetName = "AzureClassicRunAsCertificate"
        $ClassicRunAsAccountConnectionAssetName = "AzureClassicRunAsConnection"
        $ClassicRunAsAccountConnectionTypeName = "AzureClassicCertificate "
        $UploadMessage = "Please upload the .cer format of #CERT# to the Management store by following the steps below." + [Environment]::NewLine +
        "Log in to the Microsoft Azure portal (https://portal.azure.com) and select Subscriptions -> Management Certificates." + [Environment]::NewLine +
        "Then click Upload and upload the .cer format of #CERT#"

        if ($EnterpriseCertPathForClassicRunAsAccount -and $EnterpriseCertPlainPasswordForClassicRunAsAccount ) {
            $PfxCertPathForClassicRunAsAccount = $EnterpriseCertPathForClassicRunAsAccount
            $PfxCertPlainPasswordForClassicRunAsAccount = $EnterpriseCertPlainPasswordForClassicRunAsAccount
            $UploadMessage = $UploadMessage.Replace("#CERT#", $PfxCertPathForClassicRunAsAccount)
        }
        else {
            $ClassicRunAsAccountCertificateName = $AutomationAccountName + $ClassicRunAsAccountCertifcateAssetName
            $PfxCertPathForClassicRunAsAccount = Join-Path $env:TEMP ($ClassicRunAsAccountCertificateName + ".pfx")
            $PfxCertPlainPasswordForClassicRunAsAccount = $SelfSignedCertPlainPassword
            $CerCertPathForClassicRunAsAccount = Join-Path $env:TEMP ($ClassicRunAsAccountCertificateName + ".cer")
            $UploadMessage = $UploadMessage.Replace("#CERT#", $CerCertPathForClassicRunAsAccount)
            CreateSelfSignedCertificate $ClassicRunAsAccountCertificateName $PfxCertPlainPasswordForClassicRunAsAccount $PfxCertPathForClassicRunAsAccount $CerCertPathForClassicRunAsAccount $SelfSignedCertNoOfMonthsUntilExpired
        }

        # Create the Automation certificate asset
        CreateAutomationCertificateAsset $ResourceGroup $AutomationAccountName $ClassicRunAsAccountCertifcateAssetName $PfxCertPathForClassicRunAsAccount $PfxCertPlainPasswordForClassicRunAsAccount $false

        # Populate the ConnectionFieldValues
        $SubscriptionName = $subscription.Subscription.Name
        $ClassicRunAsAccountConnectionFieldValues = @{"SubscriptionName" = $SubscriptionName; "SubscriptionId" = $SubscriptionId; "CertificateAssetName" = $ClassicRunAsAccountCertifcateAssetName}

        # Create an Automation connection asset named AzureRunAsConnection in the Automation account. This connection uses the service principal.
        CreateAutomationConnectionAsset $ResourceGroup $AutomationAccountName $ClassicRunAsAccountConnectionAssetName $ClassicRunAsAccountConnectionTypeName   $ClassicRunAsAccountConnectionFieldValues

        Write-Host -ForegroundColor red       $UploadMessage
    }
    ```

    > [!IMPORTANT]
    > **Add-AzureRmAccount** es ahora un alias de **Connect-AzureRMAccount**. Al buscar elementos de biblioteca, si no ve **Connect-AzureRMAccount**, puede usar **Add-AzureRmAccount** o [actualizar los módulos](automation-update-azure-modules.md) en su cuenta de Automation.

1. En el equipo, inicie **Windows PowerShell** desde la pantalla **Inicio** con permisos de usuario elevados.
1. Desde el shell de línea de comandos con privilegios elevados, vaya a la carpeta que contiene el script que creó en el paso 1.  
1. Ejecute el script mediante los valores de parámetro de la configuración que necesita.

    **Creación de una cuenta de ejecución mediante un certificado autofirmado**  

    ```powershell
    .\New-RunAsAccount.ps1 -ResourceGroup <ResourceGroupName> -AutomationAccountName <NameofAutomationAccount> -SubscriptionId <SubscriptionId> -ApplicationDisplayName <DisplayNameofAADApplication> -SelfSignedCertPlainPassword <StrongPassword> -CreateClassicRunAsAccount $false
    ```

    **Creación de una cuenta de ejecución y una cuenta de ejecución clásica mediante un certificado autofirmado**  

    ```powershell
    .\New-RunAsAccount.ps1 -ResourceGroup <ResourceGroupName> -AutomationAccountName <NameofAutomationAccount> -SubscriptionId <SubscriptionId> -ApplicationDisplayName <DisplayNameofAADApplication> -SelfSignedCertPlainPassword <StrongPassword> -CreateClassicRunAsAccount $true
    ```

    **Creación de una cuenta de ejecución y una cuenta de ejecución clásica mediante un certificado de empresa**  

    ```powershell
    .\New-RunAsAccount.ps1 -ResourceGroup <ResourceGroupName> -AutomationAccountName <NameofAutomationAccount> -SubscriptionId <SubscriptionId> -ApplicationDisplayName <DisplayNameofAADApplication>  -SelfSignedCertPlainPassword <StrongPassword> -CreateClassicRunAsAccount $true -EnterpriseCertPathForRunAsAccount <EnterpriseCertPfxPathForRunAsAccount> -EnterpriseCertPlainPasswordForRunAsAccount <StrongPassword> -EnterpriseCertPathForClassicRunAsAccount <EnterpriseCertPfxPathForClassicRunAsAccount> -EnterpriseCertPlainPasswordForClassicRunAsAccount <StrongPassword>
    ```

    **Creación de una cuenta de ejecución y una cuenta de ejecución clásica mediante un certificado autofirmado en la nube de Azure Government**
  
    ```powershell
    .\New-RunAsAccount.ps1 -ResourceGroup <ResourceGroupName> -AutomationAccountName <NameofAutomationAccount> -SubscriptionId <SubscriptionId> -ApplicationDisplayName <DisplayNameofAADApplication> -SelfSignedCertPlainPassword <StrongPassword> -CreateClassicRunAsAccount $true  -EnvironmentName AzureUSGovernment
    ```

    > [!NOTE]
    > Después de ejecutar el script, se le pedirá que se autentique en Azure. Inicie sesión con una cuenta que sea miembro del rol Administradores de suscripciones y coadministrador de la suscripción.

Una vez que el script se ejecuta correctamente, observe lo siguiente:

* Si creó una cuenta de ejecución clásica con un certificado público autofirmado (formato .cer), el script lo crea y lo guarda en la carpeta de archivos temporales del equipo con el perfil de usuario *%USERPROFILE%\AppData\Local\Temp*, que se usa para ejecutar la sesión de PowerShell.

* Si creó una cuenta de identificación clásica con un certificado público de empresa (archivo .cer) , use este certificado. Siga las instrucciones para [cargar un certificado de Management API en Azure Portal](../azure-api-management-certs.md).

## <a name="delete-a-run-as-or-classic-run-as-account"></a>Eliminación de una cuenta de ejecución o de ejecución clásica

En esta sección se describe cómo eliminar y volver a crear una cuenta de ejecución o de ejecución clásica. Al realizar esta acción, la cuenta de Automation se conserva. Después de eliminar una cuenta de ejecución o de ejecución clásica, puede volver a crearla en el portal de Azure.

1. Abra la cuenta de Automation en Azure Portal.

2. En la página **Cuenta de Automation**, seleccione **Cuentas de ejecución**.

3. En la página de propiedades **Cuentas de ejecución**, seleccione la cuenta de ejecución o la cuenta de ejecución clásica que quiere eliminar. A continuación, en el panel **Propiedades** de la cuenta seleccionada, haga clic en **Eliminar**.

   ![Eliminación de una cuenta de ejecución](media/manage-runas-account/automation-account-delete-runas.png)

1. Mientras se está eliminando la cuenta, puede seguir el progreso desde el menú, en **Notificaciones**.

1. Después de eliminar la cuenta, puede volver a crearla en la página de propiedades **Cuentas de ejecución** seleccionando la opción de creación **Cuenta de ejecución de Azure**.

   ![Nueva creación de la cuenta de ejecución de Automation](media/manage-runas-account/automation-account-create-runas.png)

## <a name="cert-renewal"></a>Renovación de certificado autofirmado

En algún momento antes de que expire su cuenta de ejecución, debe renovar el certificado. Si cree que se ha puesto en peligro la cuenta de ejecución, puede eliminarla y volver a crearla. En esta sección se describe cómo realizar estas operaciones.

El certificado autofirmado que creó para la cuenta de ejecución expira un año a partir de la fecha de creación. Se puede renovar en cualquier momento antes de que expire. Cuando se renueva, se conserva el certificado válido actual para tener la seguridad de que los runbooks que se ponen en la cola o que se ejecutan activamente, y que se autentican con la cuenta de ejecución, no resultan afectados negativamente. El certificado es válido hasta la fecha de expiración.

> [!NOTE]
> Si ha configurado la cuenta de ejecución de Automation para usar un certificado emitido por una entidad de certificación de empresa y usa esta opción, ese certificado se reemplaza por otro autofirmado.

Para renovar el certificado, realice estos pasos:

1. Abra la cuenta de Automation en Azure Portal.

1. Seleccione **Cuentas de ejecución** en **Configuración de la cuenta**.

    ![Panel de propiedades de la cuenta de Automation](media/manage-runas-account/automation-account-properties-pane.png)

1. En la página de propiedades **Cuentas de ejecución**, seleccione la cuenta de ejecución o la cuenta de ejecución clásica para la que quiere renovar el certificado.

1. En el panel **Propiedades** de la cuenta seleccionada, haga clic en **Renovar certificado**.

    ![Renovación del certificado para una cuenta de ejecución](media/manage-runas-account/automation-account-renew-runas-certificate.png)

1. Mientras se está renovando el certificado, puede seguir el progreso desde el menú, en **Notificaciones**.

## <a name="limiting-run-as-account-permissions"></a>Limitación de los permisos de las cuentas de ejecución

Para controlar el destino de la automatización en los recursos de Azure Automation, a la cuenta de ejecución se le concede de forma predeterminada derechos de colaborador en la suscripción. Si tiene que restringir lo que puede hacer la entidad de servicio de ejecución, puede quitar la cuenta del rol de colaborador de la suscripción y agregarla como colaborador a los grupos de recursos que quiere especificar.

En Azure Portal, seleccione **Suscripciones** y elija la suscripción de su cuenta de Automation. Seleccione **Control de acceso (IAM)** y después la pestaña **Asignaciones de roles**. Busque la entidad de servicio de su cuenta de Automation (se parece a \<AutomationAccountName\>_unique identifier). Seleccione la cuenta y haga clic en **Quitar** para quitarla de la suscripción.

![Colaboradores de suscripción](media/manage-runas-account/automation-account-remove-subscription.png)

Para agregar la entidad de servicio a un grupo de recursos, seleccione el grupo de recursos en Azure Portal y elija **Control de acceso (IAM)**. Seleccione **Agregar asignación de roles** para abrir la página **Agregar asignación de roles**. En **Rol**, seleccione **Colaborador**. En el cuadro de texto **Seleccionar**, escriba el nombre de la entidad de servicio de la cuenta de ejecución y selecciónela de la lista. Haga clic en **Guardar** para guardar los cambios. Siga estos pasos con todos los grupos de recursos a los que desee dar acceso a la entidad de servicio de ejecución de Azure Automation.

## <a name="misconfiguration"></a>Error de configuración

Puede que alguno de los elementos de configuración necesarios para que la cuenta de ejecución o la cuenta de ejecución clásica funcionen correctamente se haya eliminado o no se haya creado correctamente durante la configuración inicial. Estos elementos son:

* Recurso de certificado
* Recurso de conexión
* La cuenta de ejecución se ha quitado del rol de colaborador
* Entidad de servicio o aplicación en Azure AD

En los casos anteriores y en otros casos de errores de configuración, la cuenta de Automation detecta los cambios y muestra el estado *Incompleto* en el panel de propiedades **Cuentas de ejecución** de la cuenta.

![Estado de configuración incompleta de la cuenta de ejecución](media/manage-runas-account/automation-account-runas-incomplete-config.png)

Al seleccionar la cuenta de ejecución, el panel **Propiedades** muestra el mensaje de error siguiente:

```text
The Run As account is incomplete. Either one of these was deleted or not created - Azure Active Directory Application, Service Principal, Role, Automation Certificate asset, Automation Connect asset - or the Thumbprint is not identical between Certificate and Connection. Please delete and then re-create the Run As Account.
```

Rápidamente puede resolver estos problemas de la cuenta de ejecución con solo eliminarla cuenta y volver a crearla.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre las entidades de servicio, consulte [Objetos de aplicación y de entidad de servicio](../active-directory/develop/app-objects-and-service-principals.md).
* Para más información sobre los certificados y los servicios de Azure, consulte [Introducción a los certificados para Azure Cloud Services](../cloud-services/cloud-services-certs-create.md).