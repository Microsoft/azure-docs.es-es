---
title: Configuración de Azure Attestation con Azure PowerShell
description: Instalación y configuración de un proveedor de atestación mediante Azure PowerShell.
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: overview
ms.date: 08/31/2020
ms.author: mbaldwin
ms.openlocfilehash: 70e86e01a9d37a27620d451bcd5d035dfcb4573d
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89237121"
---
# <a name="quickstart-set-up-azure-attestation-with-azure-powershell"></a>Inicio rápido: Configuración de Azure Attestation con Azure PowerShell

Siga los pasos que se indican a continuación para crear y configurar un proveedor de atestación mediante Azure PowerShell. Consulte [Introducción a Azure PowerShell](/powershell/azure/?view=azps-2.8.0&viewFallbackFrom=azps-2.4.0) para obtener información sobre cómo instalar y ejecutar Azure PowerShell.

Tenga en cuenta que la Galería de PowerShell ha dejado de usar las versiones 1.0 y 1.1 de Seguridad de la capa de transporte (TLS). Se recomienda usar TLS 1.2 o una versión posterior. Puede que aparezcan los siguientes mensajes de error:

- ADVERTENCIA: No se puede resolver el origen del paquete 'https://www.powershellgallery.com/api/v2 '
- PackageManagement\Install-Package: No match was found for the specified search criteria and module name (PackageManagement\Install-Package: No se encontraron coincidencias para los criterios de búsqueda y el nombre de módulo especificados) 

Para seguir interactuando con la Galería de PowerShell, ejecute el siguiente comando antes que los comandos Install-Module:

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12 
```

## <a name="install-azattestation-powershell-module"></a>Instalación del módulo Az.Atestation de PowerShell

En la máquina con Azure PowerShell, instale el módulo Az.Atestation de PowerShell, que contiene cmdlets para Azure Attestation.  

### <a name="initial-installation"></a>Instalación inicial

Finalice todas las ventanas de PowerShell existentes.

Para instalar para el "usuario actual", inicie una ventana de PowerShell sin privilegios elevados y ejecute:

```powershell
Install-Module -Name Az.Attestation -AllowClobber -Scope CurrentUser
```

Para instalar para "todos los usuarios", inicie una ventana de PowerShell con privilegios elevados y ejecute:

```powershell
Install-Module -Name Az.Attestation -AllowClobber -Scope AllUsers
```

Cierre la consola de PowerShell con privilegios elevados.

### <a name="update-the-installation"></a>Actualización de la instalación

Finalice todas las ventanas de PowerShell existentes.

Para actualizar para el "usuario actual", inicie una ventana de PowerShell sin privilegios elevados y ejecute:

```powershell
Update-Module -Name Az.Attestation
```

Para actualizar para "todos los usuarios", inicie una ventana de PowerShell con privilegios elevados y ejecute:

```powershell
Update-Module -Name Az.Attestation
```

Cierre la consola de PowerShell con privilegios elevados.

### <a name="get-installed-modules"></a>Obtener los módulos instalados

Versión mínima de los módulos Az necesarios para admitir las operaciones de atestación:
- Az 4.5.0
- Az.Accounts 1.9.2
- Az.Attestation 0.1.8

Ejecute el siguiente comando para comprobar la versión instalada de todos los módulos Az: 

```powershell
Get-InstalledModule
```
Si las versiones no coinciden con el requisito mínimo, ejecute comandos Update-Module.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure en la consola de PowerShell (sin privilegios de acceso elevados).

```powershell
Connect-AzAccount
```

Si es necesario, cambie a la suscripción que se va a usar para Azure Attestation.

```powershell
Set-AzContext -Subscription <subscription id>  
```

## <a name="register-microsoftattestation-resource-provider"></a>Registro del proveedor de recursos Microsoft.Atestation

Registre el proveedor de recursos Microsoft.Atestation en la suscripción. Para obtener más información sobre los proveedores de recursos de Azure y cómo configurar y administrar los proveedores de recursos, consulte [Tipos y proveedores de recursos de Azure](../azure-resource-manager/management/resource-providers-and-types.md). Tenga en cuenta que el registro de un proveedor de recursos solo es necesario una vez para una suscripción.

```powershell
Register-AzResourceProvider -ProviderNamespace Microsoft.Attestation
```
## <a name="regional-availability-of-azure-attestation"></a>Disponibilidad regional de Azure Attestation

```powershell
(Get-AzResourceProvider -ProviderNamespace Microsoft.Attestation)[0].Locations
```

## <a name="create-an-azure-resource-group"></a>Creación de un grupo de recursos de Azure

Cree un grupo de recursos para el proveedor de atestación. Tenga en cuenta que en este mismo grupo se pueden colocar otros recursos de Azure, incluida una máquina virtual con una instancia de la aplicación cliente.

```powershell
$location = "uksouth" 
$attestationResourceGroup = "<attestation provider resource group name>"
New-AzResourceGroup -Name $attestationResourceGroup -Location $location 
```

## <a name="create-and-manage-an-attestation-provider"></a>Creación y administración de un proveedor de atestación

New-AzAttestation crea un proveedor de atestación.

```powershell
$attestationProvider = "<attestation provider name>" 
New-AzAttestation -Name $attestationProvider -ResourceGroupName $attestationResourceGroup -Location $location
```

PolicySignerCertificateFile es un archivo que especifica un conjunto de claves de firma de confianza. Si se especifica un nombre de archivo en el parámetro PolicySignerCertificateFile, el proveedor de atestación solo se puede configurar mediante directivas con formato JWT firmado. En caso contrario, la directiva se puede configurar con un formato de texto o un formato JWT sin firma.

```powershell
New-AzAttestation -Name $attestationProvider -ResourceGroupName $attestationResourceGroup -Location $location -PolicySignersCertificateFile "C:\test\policySignersCertificates.pem"
```

Puede consultar un ejemplo de PolicySignersCertificateFile en [Ejemplos de un certificado de firmante de directivas de atestación](policy-signer-examples.md).

Get-AzAttestation recupera las propiedades del proveedor de atestación, como status y AttestURI. Tome nota del valor de AttestURI, ya que se necesitará más adelante.

```azurepowershell
Get-AzAttestation -Name $attestationProvider -ResourceGroupName $attestationResourceGroup  
```

El comando anterior debe generar una salida como la siguiente: 

```
Id:/subscriptions/MySubscriptionID/resourceGroups/MyResourceGroup/providers/Microsoft.Attestation/attestationProviders/MyAttestationProvider
Location: MyLocation
ResourceGroupName: MyResourceGroup
Name: MyAttestationProvider
Status: Ready
TrustModel: AAD
AttestUri: https://MyAttestationProvider.us.attest.azure.net 
Tags: 
TagsTable: 
```

Los proveedores de atestación se pueden eliminar con el cmdlet Remove-AzAttestation.  

``azurepowershell Remove-AzAttestation -Name $attestationProvider -ResourceGroupName $attestationResourceGroup
```

## Policy management

In order to manage policies, an Azure AD user requires the following permissions for "Actions":
- Microsoft.Attestation/attestationProviders/attestation/read
- Microsoft.Attestation/attestationProviders/attestation/write
- Microsoft.Attestation/attestationProviders/attestation/delete

These permissions can be assigned to an AD user through a role such as "Owner" (wildcard permissions), "Contributor" (wildcard permissions) or "Attestation Contributor" (specific permissions for Azure Attestation only).  

In order to read policies, an Azure AD user requires the following permission for "Actions":
- Microsoft.Attestation/attestationProviders/attestation/read

This permission can be assigned to an AD user through a role such as "Reader" (wildcard permissions) or "Attestation Reader" (specific permissions for Azure Attestation only).

Below PowerShell cmdlets provide policy management for an attestation provider (one TEE at a time).

Get-AzAttestationPolicy returns the current policy for the specified TEE. The cmdlet displays policy in both text and JWT format of the policy.

```powershell
$teeType = "<tee Type>"
Get-AzAttestationPolicy   -Name $attestationProvider -ResourceGroupName $attestationResourceGroup -Tee $teeType 
```

Los tipos de TEE admitidos son "sgxenclave" y "vbsenclave".

Set-AttestationPolicy establece una nueva directiva para el TEE especificado. El cmdlet acepta la directiva con formato de texto o JWT y se controla mediante el parámetro PolicyFormat. El valor predeterminado de PolicyFormat es "Text". 

```powershell
$policyFormat = "<policy format>"
$policy=Get-Content -path "C:\test\policy.txt" -Raw
Set-AzAttestationPolicy   -Name $attestationProvider -ResourceGroupName $attestationResourceGroup -Tee $teeType -Policy $policy -PolicyFormat $policyFormat 
```

Si se proporciona un archivo con PolicySignerCertificateFile durante la creación de un proveedor de atestación, las directivas solo se pueden configurar con formato JWT firmado. En caso contrario, la directiva se puede configurar con un formato de texto o un formato JWT sin firma.

La directiva de atestación en formato JWT debe contener una notificación denominada "AttestationPolicy". En el caso de la directiva firmada, el JWT debe estar firmado con una clave privada que se corresponda con cualquiera de los certificados de firmante de directivas existentes.

Para obtener ejemplos de directivas, consulte [Ejemplos de una directiva de atestación](policy-examples.md).

Reset-AzAttestationPolicy restablece el valor predeterminado de la directiva en el TEE especificado.

```powershell
Reset-AzAttestationPolicy -Name $attestationProvider -ResourceGroupName $attestationResourceGroup -Tee $teeType 
```

## <a name="policy-signer-certificates-management"></a>Administración de certificados de firmante de directivas

Los siguientes cmdlets de PowerShell proporcionan la administración de certificados de firmante de directivas para un proveedor de atestación:

```powershell
Get-AzAttestationPolicySigners -Name $attestationProvider -ResourceGroupName $attestationResourceGroup

Add-AzAttestationPolicySigner -Name $attestationProvider -ResourceGroupName $attestationResourceGroup -Signer <signer>

Remove-AzAttestationPolicySigner -Name $attestationProvider -ResourceGroupName $attestationResourceGroup -Signer <signer>
```

El certificado de firmante de directivas es un JWT firmado con la notificación "maa-policyCertificate". El valor de la notificación es un JWK que contiene la clave de firma de confianza que se va a agregar. El JWT debe estar firmado con una clave privada que se corresponda con cualquiera de los certificados de firmante de directivas existentes.

Tenga en cuenta que toda la manipulación semántica del certificado del firmante de directivas debe realizarse fuera de PowerShell. En lo que se refiere a PowerShell, es una cadena simple.

Puede consultar un ejemplo de certificado de firmante de directivas en [Ejemplos de un certificado de firmante de directivas de atestación](policy-signer-examples.md).

Para obtener más información sobre los cmdlets y sus parámetros, consulte los [cmdlets de PowerShell de Azure Attestation](/powershell/module/az.attestation/?view=azps-4.3.0#attestation). 

## <a name="next-steps"></a>Pasos siguientes

- [Creación y firma de una directiva de atestación](author-sign-policy.md)
- [Atestación de un enclave SGX con ejemplos de código](https://docs.microsoft.com/samples/browse/?expanded=azure&terms=attestation)
