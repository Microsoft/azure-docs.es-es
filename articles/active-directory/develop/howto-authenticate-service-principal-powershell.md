---
title: Creación de identidad para aplicaciones de Azure con PowerShell | Microsoft Docs
description: Se describe cómo usar Azure PowerShell para crear una entidad de servicio y una aplicación de Azure Active Directory, además de cómo concederle acceso a recursos mediante el control de acceso basado en rol. Muestra cómo autenticar la aplicación con un certificado.
services: active-directory
documentationcenter: na
author: rwike77
manager: CelesteDG
ms.assetid: d2caf121-9fbe-4f00-bf9d-8f3d1f00a6ff
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 10/24/2018
ms.author: ryanwi
ms.reviewer: tomfitz
ms.collection: M365-identity-device-management
ms.openlocfilehash: d6d6de5186b1906d56b5a43317d9c36ad1cc6aad
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540412"
---
# <a name="how-to-use-azure-powershell-to-create-a-service-principal-with-a-certificate"></a>Procedimientos para: Uso de Azure PowerShell para crear una entidad de servicio con un certificado

Cuando haya una aplicación o un script que necesite acceder a recursos, puede configurar una identidad para la aplicación y autenticarla con sus propias credenciales. Esta identidad se conoce como una entidad de servicio. Este enfoque le permite:

* Asignar permisos a la identidad de la aplicación que sean diferentes a los suyos propios. Normalmente, estos permisos están restringidos a exactamente aquello que la aplicación debe hacer.
* Usar un certificado para la autenticación al ejecutar un script desatendido.

> [!IMPORTANT]
> En lugar de crear una entidad de servicio, considere el uso de identidades administradas para recursos de Azure para la identidad de la aplicación. Si el código se ejecuta en un servicio que admite identidades administradas y tiene acceso a recursos que admiten la autenticación de Azure Active Directory (Azure AD), las identidades administradas son la opción ideal para usted. Para obtener más información sobre las identidades administradas para recursos de Azure, incluidos los servicios que actualmente lo admiten, consulte [¿Qué es Managed Identities for Azure Resources?](../managed-identities-azure-resources/overview.md).

En este artículo se muestra cómo crear una entidad de servicio que se autentica con un certificado. Para configurar una entidad de servicio con contraseña, consulte [Creación de una entidad de servicio de Azure con Azure PowerShell](/powershell/azure/create-azure-service-principal-azureps).

Debe tener la [versión más reciente](/powershell/azure/install-az-ps) de PowerShell para este artículo.

[!INCLUDE [az-powershell-update](../../../includes/updated-for-az.md)]

## <a name="required-permissions"></a>Permisos necesarios

Para completar este artículo, debe tener permisos suficientes tanto en su suscripción de Azure como en Azure AD. En concreto, debe poder crear una aplicación en Azure AD y asignar la entidad de servicio a un rol.

El portal representa la forma más sencilla de comprobar si su cuenta tiene los permisos adecuados. Consulte el [artículo que explica cómo comprobar el permiso requerido](howto-create-service-principal-portal.md#required-permissions).

## <a name="create-service-principal-with-self-signed-certificate"></a>Creación de una entidad de servicio con un certificado autofirmado

En el ejemplo siguiente se trata un escenario sencillo. En él se usa [New-AzADServicePrincipal](/powershell/module/az.resources/new-azadserviceprincipal) para crear una entidad de servicio con un certificado autofirmado, y se utiliza [New-AzureRmRoleAssignment](/powershell/module/az.resources/new-azroleassignment) para asignar el rol [Colaborador](../../role-based-access-control/built-in-roles.md#contributor) a la entidad de servicio. La asignación de roles se limita a su suscripción de Azure seleccionada actualmente. Para seleccionar una suscripción diferente, use [Set-AzContext](/powershell/module/Az.Accounts/Set-AzContext).

```powershell
$cert = New-SelfSignedCertificate -CertStoreLocation "cert:\CurrentUser\My" `
  -Subject "CN=exampleappScriptCert" `
  -KeySpec KeyExchange
$keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData())

$sp = New-AzADServicePrincipal -DisplayName exampleapp `
  -CertValue $keyValue `
  -EndDate $cert.NotAfter `
  -StartDate $cert.NotBefore
Sleep 20
New-AzRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $sp.ApplicationId
```

El ejemplo entra en suspensión durante 20 segundos para dar tiempo a que la nueva entidad de servicio se propague por Azure AD. Si el script no espera el tiempo suficiente, verá un error que indica: "La entidad de seguridad {ID} no existe en el directorio {DIR-ID}". Para resolver este error, espere unos instantes y, a continuación, ejecute de nuevo el comando **New-AzRoleAssignment**.

Puede restringir el ámbito de la asignación de roles a un grupo de recursos específico mediante el parámetro **ResourceGroupName**. Puede restringir el ámbito a un recurso concreto si usa los parámetros **ResourceType** y **ResourceName**. 

Si **no tiene Windows 10 o Windows Server 2016**, descargue el [generador de certificados autofirmados](https://gallery.technet.microsoft.com/scriptcenter/Self-signed-certificate-5920a7c6/) del sitio Microsoft Script Center. Extraiga el contenido e importe el cmdlet que necesita.

```powershell
# Only run if you could not use New-SelfSignedCertificate
Import-Module -Name c:\ExtractedModule\New-SelfSignedCertificateEx.ps1
```

En el script, sustituya las dos líneas siguientes para generar el certificado.

```powershell
New-SelfSignedCertificateEx -StoreLocation CurrentUser `
  -Subject "CN=exampleapp" `
  -KeySpec "Exchange" `
  -FriendlyName "exampleapp"
$cert = Get-ChildItem -path Cert:\CurrentUser\my | where {$PSitem.Subject -eq 'CN=exampleapp' }
```

### <a name="provide-certificate-through-automated-powershell-script"></a>Proporcionar un certificado a través de un script automatizado de PowerShell

Cada vez que inicie sesión como entidad de servicio, debe proporcionar el id. del inquilino del directorio para la aplicación de AD. Un inquilino es una instancia de Azure AD.

```powershell
$TenantId = (Get-AzSubscription -SubscriptionName "Contoso Default").TenantId
$ApplicationId = (Get-AzADApplication -DisplayNameStartWith exampleapp).ApplicationId

 $Thumbprint = (Get-ChildItem cert:\CurrentUser\My\ | Where-Object {$_.Subject -eq "CN=exampleappScriptCert" }).Thumbprint
 Connect-AzAccount -ServicePrincipal `
  -CertificateThumbprint $Thumbprint `
  -ApplicationId $ApplicationId `
  -TenantId $TenantId
```

## <a name="create-service-principal-with-certificate-from-certificate-authority"></a>Creación de una entidad de servicio con un certificado de una entidad de certificación

En el ejemplo siguiente se utiliza un certificado emitido por una entidad de certificación para crear una entidad de servicio. La asignación se limita a la suscripción de Azure especificada. Se agrega la entidad de servicio al rol [Colaborador](../../role-based-access-control/built-in-roles.md#contributor). Si se produce un error durante la asignación de roles, vuelva a intentar la asignación.

```powershell
Param (
 [Parameter(Mandatory=$true)]
 [String] $ApplicationDisplayName,

 [Parameter(Mandatory=$true)]
 [String] $SubscriptionId,

 [Parameter(Mandatory=$true)]
 [String] $CertPath,

 [Parameter(Mandatory=$true)]
 [String] $CertPlainPassword
 )

 Connect-AzAccount
 Import-Module Az.Resources
 Set-AzContext -Subscription $SubscriptionId
 
 $CertPassword = ConvertTo-SecureString $CertPlainPassword -AsPlainText -Force

 $PFXCert = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList @($CertPath, $CertPassword)
 $KeyValue = [System.Convert]::ToBase64String($PFXCert.GetRawCertData())

 $ServicePrincipal = New-AzADServicePrincipal -DisplayName $ApplicationDisplayName
 New-AzADSpCredential -ObjectId $ServicePrincipal.Id -CertValue $KeyValue -StartDate $PFXCert.NotBefore -EndDate $PFXCert.NotAfter
 Get-AzADServicePrincipal -ObjectId $ServicePrincipal.Id 

 $NewRole = $null
 $Retries = 0;
 While ($NewRole -eq $null -and $Retries -le 6)
 {
    # Sleep here for a few seconds to allow the service principal application to become active (should only take a couple of seconds normally)
    Sleep 15
    New-AzRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $ServicePrincipal.ApplicationId | Write-Verbose -ErrorAction SilentlyContinue
    $NewRole = Get-AzRoleAssignment -ObjectId $ServicePrincipal.Id -ErrorAction SilentlyContinue
    $Retries++;
 }
 
 $NewRole
```

### <a name="provide-certificate-through-automated-powershell-script"></a>Proporcionar un certificado a través de un script automatizado de PowerShell
Cada vez que inicie sesión como entidad de servicio, debe proporcionar el id. del inquilino del directorio para la aplicación de AD. Un inquilino es una instancia de Azure AD.

```powershell
Param (
 
 [Parameter(Mandatory=$true)]
 [String] $CertPath,

 [Parameter(Mandatory=$true)]
 [String] $CertPlainPassword,
 
 [Parameter(Mandatory=$true)]
 [String] $ApplicationId,

 [Parameter(Mandatory=$true)]
 [String] $TenantId
 )

 $CertPassword = ConvertTo-SecureString $CertPlainPassword -AsPlainText -Force
 $PFXCert = New-Object `
  -TypeName System.Security.Cryptography.X509Certificates.X509Certificate2 `
  -ArgumentList @($CertPath, $CertPassword)
 $Thumbprint = $PFXCert.Thumbprint

 Connect-AzAccount -ServicePrincipal `
  -CertificateThumbprint $Thumbprint `
  -ApplicationId $ApplicationId `
  -TenantId $TenantId
```

Ni el identificador de la aplicación ni el identificador del inquilino son confidenciales, por lo que se pueden integrar directamente en el script. Si necesita recuperar el identificador del inquilino, use:

```powershell
(Get-AzSubscription -SubscriptionName "Contoso Default").TenantId
```

Si necesita recuperar el identificador de la aplicación, use:

```powershell
(Get-AzADApplication -DisplayNameStartWith {display-name}).ApplicationId
```

## <a name="change-credentials"></a>Cambio de credenciales

Para cambiar las credenciales de una aplicación de AD, ya sea debido a un riesgo de seguridad o a la expiración de una credencial, use los cmdlets [Remove-AzADAppCredential](/powershell/module/az.resources/remove-azadappcredential) y [New-AzADAppCredential](/powershell/module/az.resources/new-azadappcredential).

Para quitar todas las credenciales de una aplicación, use:

```powershell
Get-AzADApplication -DisplayName exampleapp | Remove-AzADAppCredential
```

Para agregar un valor de certificado, cree un certificado autofirmado, como se muestra en este artículo. A continuación, use:

```powershell
Get-AzADApplication -DisplayName exampleapp | New-AzADAppCredential `
  -CertValue $keyValue `
  -EndDate $cert.NotAfter `
  -StartDate $cert.NotBefore
```

## <a name="debug"></a>Depurar

Durante la creación de una entidad de servicio pueden producirse los siguientes errores:

* **"Authentication_Unauthorized"** o **"No subscription found in the context"** (No se encuentra una suscripción en el contexto). - Este error se ve cuando la cuenta no tiene los [permisos necesarios](#required-permissions) en Azure AD para registrar una aplicación. Por lo general, este error aparece cuando los usuarios administradores de Azure Active Directory son los únicos que pueden registrar las aplicaciones y la cuenta no es de un administrador. Pida al administrador que le asigne un rol de administrador o que permita a los usuarios registrar aplicaciones.

* Su cuenta **"no tiene autorización para realizar la acción 'Microsoft.Authorization/roleAssignments/write' en el ámbito '/ subscriptions / {guid}'."**  - Este error aparece cuando la cuenta no tiene permisos suficientes para asignar un rol a una identidad. Pida al administrador de suscripciones que le agregue al rol de administrador de acceso de usuario.

## <a name="next-steps"></a>Pasos siguientes

* Para configurar una entidad de servicio con contraseña, consulte [Creación de una entidad de servicio de Azure con Azure PowerShell](/powershell/azure/create-azure-service-principal-azureps).
* Si desea conocer los pasos detallados de la integración de una aplicación en Azure para administrar recursos, consulte [Guía del desarrollador para la autorización con la API de Azure Resource Manager](../../azure-resource-manager/resource-manager-api-authentication.md).
* Para obtener una explicación más detallada de las aplicaciones y entidades de servicio, consulte [Objetos de aplicación y de entidad de servicio](app-objects-and-service-principals.md).
* Para más información sobre la autenticación de Azure AD, vea [Escenarios de autenticación para Azure AD](authentication-scenarios.md).
