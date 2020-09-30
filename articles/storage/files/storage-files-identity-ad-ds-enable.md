---
title: Habilitación de la autenticación de AD DS en recursos compartidos de archivos de Azure
description: Obtenga información sobre cómo habilitar la autenticación de Active Directory Domain Services en SMB para recursos compartidos de archivos de Azure. Después, las máquinas virtuales Windows unidas a un dominio podrán acceder a los recursos compartidos de archivos de Azure con las credenciales de AD DS.
author: roygara
ms.service: storage
ms.subservice: files
ms.topic: how-to
ms.date: 09/13/2020
ms.author: rogarana
ms.openlocfilehash: ce6325abf34813a9ca397f5bcbe2e774af3442d4
ms.sourcegitcommit: 51df05f27adb8f3ce67ad11d75cb0ee0b016dc5d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/14/2020
ms.locfileid: "90061485"
---
# <a name="part-one-enable-ad-ds-authentication-for-your-azure-file-shares"></a>Parte 1: Habilitación de la autenticación de AD DS para los recursos compartidos de archivos de Azure 

Antes de habilitar la autenticación de Active Directory Domain Services (AD DS), asegúrese de leer el [artículo de información general](storage-files-identity-auth-active-directory-enable.md) para comprender los escenarios y requisitos admitidos.

En este artículo se describe el proceso necesario para habilitar la autenticación de Active Directory Domain Services (AD DS) en la cuenta de almacenamiento. Después de habilitar la característica, tendrá que configurar la cuenta de almacenamiento y AD DS, con el fin de usar las credenciales de AD DS para autenticarse en el recurso compartido de archivos de Azure. Para habilitar la autenticación de AD DS en SMB para los recursos compartidos de archivos de Azure, debe registrar la cuenta de almacenamiento en AD DS y, luego, establecer las propiedades de dominio necesarias en la cuenta de almacenamiento.

Para registrar la cuenta de almacenamiento con AD DS, cree una cuenta que la represente en AD DS. Este proceso se puede considerar como si se tratara de la creación de una cuenta que represente un servidor de archivos de Windows local en AD DS. Cuando la característica está habilitada en la cuenta de almacenamiento, se aplica a todos los recursos compartidos de archivos nuevos y existentes de la cuenta.

## <a name="option-one-recommended-use-azfileshybrid-powershell-module"></a>Opción 1 (recomendada): Uso del módulo de PowerShell AzFilesHybrid

Los cmdlets del módulo AzFilesHybrid de PowerShell realizan las modificaciones necesarias y habilitan automáticamente la característica. Como algunas partes del cmdlet interactúan con la instancia local de AD DS, se explica lo que hacen los cmdlets, para que pueda determinar si los cambios se alinean con las directivas de cumplimiento y seguridad, y asegurarse de que tiene los permisos adecuados para ejecutar los cmdlets. Aunque se recomienda usar el módulo AzFilesHybrid, si no puede hacerlo, se proporcionan los pasos para que pueda seguirlos manualmente.

### <a name="download-azfileshybrid-module"></a>Descarga del módulo AzFilesHybrid

- [Descargue y descomprima el módulo AzFilesHybrid (módulo de disponibilidad general: v0.2.0+)](https://github.com/Azure-Samples/azure-files-samples/releases) Tenga en cuenta que el cifrado de Kerberos AES 256 es compatible con v0.2.2 o posterior. Si ha habilitado la característica con una versión de AzFilesHybrid inferior a v0.2.2 y quiere actualizarla para que admita el cifrado de Kerberos AES 256, consulte [este artículo](https://docs.microsoft.com/azure/storage/files/storage-troubleshoot-windows-file-connection-problems). 
- Instale y ejecute el módulo en un dispositivo que se haya unido a un dominio a AD DS local con credenciales de AD DS que tengan permisos para crear una cuenta de inicio de sesión de servicio o una cuenta de equipo en la instancia de destino.
-  Ejecute el script con una credencial de AD DS local que esté sincronizada con su instancia de Azure AD. La credencial de AD DS local debe tener los permisos de la cuenta de propietario o del rol de Azure de colaborador.

### <a name="run-join-azstorageaccountforauth"></a>Ejecución de Join-AzStorageAccountForAuth

El cmdlet `Join-AzStorageAccountForAuth` realiza la acción equivalente a la unión a un dominio sin conexión en nombre de la cuenta de almacenamiento especificada. El script usa el cmdlet para crear una [cuenta de equipo](https://docs.microsoft.com/windows/security/identity-protection/access-control/active-directory-accounts#manage-default-local-accounts-in-active-directory) en el dominio de AD. Si, por cualquier motivo, no puede usar una cuenta de equipo, puede modificar el script para crear una [cuenta de inicio de sesión de servicio](https://docs.microsoft.com/windows/win32/ad/about-service-logon-accounts) en su lugar. Si decide ejecutar el comando manualmente, debe seleccionar la cuenta que mejor se adapte al entorno.

La cuenta de AD DS que crea el cmdlet representa la cuenta de almacenamiento. Si la cuenta de AD DS se crea en una unidad organizativa (OU) que aplica la expiración de contraseñas, debe actualizar la contraseña antes de su vigencia máxima. Si no actualiza la contraseña de la cuenta antes de esa vigencia, se producirán errores de autenticación al acceder a los recursos compartidos de archivos de Azure. Para saber cómo actualizar la contraseña, consulte [Actualización de la contraseña de la cuenta de AD DS](storage-files-identity-ad-ds-update-password.md).

Reemplace los valores de marcador de posición por los suyos propios en los parámetros siguientes antes de ejecutar el script en PowerShell.
> [!IMPORTANT]
> El cmdlet de unión a un dominio creará una cuenta de AD para representar la cuenta de almacenamiento (recurso compartido de archivos) en AD. Puede optar por registrarla como una cuenta de equipo o una cuenta de inicio de sesión de servicio; consulte [Preguntas más frecuentes](https://docs.microsoft.com/azure/storage/files/storage-files-faq#security-authentication-and-access-control) para obtener más información. En el caso de las cuentas de equipo, se establece un tiempo de expiración de contraseña predeterminado en AD de 30 días. Del mismo modo, la cuenta de inicio de sesión de servicio puede tener establecido un tiempo de expiración de contraseña predeterminado en el dominio o la unidad organizativa (OU) de AD.
> Para ambos tipos de cuenta, se recomienda comprobar cuál es el tiempo de expiración de la contraseña configurado en el entorno de AD y planear la [actualización de la contraseña de la identidad de la cuenta de almacenamiento](storage-files-identity-ad-ds-update-password.md) de la cuenta de AD antes el tiempo máximo. Puede considerar la posibilidad de [crear una unidad organizativa (UO) de AD en AD](https://docs.microsoft.com/powershell/module/addsadministration/new-adorganizationalunit?view=win10-ps) y deshabilitar la directiva de expiración de contraseñas en las [cuentas de equipo](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj852252(v=ws.11)?redirectedfrom=MSDN) o las cuentas de inicio de sesión de servicio, según corresponda. 

```PowerShell
#Change the execution policy to unblock importing AzFilesHybrid.psm1 module
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser

# Navigate to where AzFilesHybrid is unzipped and stored and run to copy the files into your path
.\CopyToPSPath.ps1 

#Import AzFilesHybrid module
Import-Module -Name AzFilesHybrid

#Login with an Azure AD credential that has either storage account owner or contributer Azure role assignment
Connect-AzAccount

#Define parameters
$SubscriptionId = "<your-subscription-id-here>"
$ResourceGroupName = "<resource-group-name-here>"
$StorageAccountName = "<storage-account-name-here>"

#Select the target subscription for the current session
Select-AzSubscription -SubscriptionId $SubscriptionId 

# Register the target storage account with your active directory environment under the target OU (for example: specify the OU with Name as "UserAccounts" or DistinguishedName as "OU=UserAccounts,DC=CONTOSO,DC=COM"). 
# You can use to this PowerShell cmdlet: Get-ADOrganizationalUnit to find the Name and DistinguishedName of your target OU. If you are using the OU Name, specify it with -OrganizationalUnitName as shown below. If you are using the OU DistinguishedName, you can set it with -OrganizationalUnitDistinguishedName. You can choose to provide one of the two names to specify the target OU.
# You can choose to create the identity that represents the storage account as either a Service Logon Account or Computer Account (default parameter value), depends on the AD permission you have and preference. 
# Run Get-Help Join-AzStorageAccountForAuth for more details on this cmdlet.

Join-AzStorageAccountForAuth `
        -ResourceGroupName $ResourceGroupName `
        -StorageAccountName $StorageAccountName `
        -DomainAccountType "<ComputerAccount|ServiceLogonAccount>" `
        -OrganizationalUnitDistinguishedName "<ou-distinguishedname-here>" # If you don't provide the OU name as an input parameter, the AD identity that represents the storage account is created under the root directory.

#You can run the Debug-AzStorageAccountAuth cmdlet to conduct a set of basic checks on your AD configuration with the logged on AD user. This cmdlet is supported on AzFilesHybrid v0.1.2+ version. For more details on the checks performed in this cmdlet, see Azure Files Windows troubleshooting guide.
Debug-AzStorageAccountAuth -StorageAccountName $StorageAccountName -ResourceGroupName $ResourceGroupName -Verbose
```

## <a name="option-2-manually-perform-the-enablement-actions"></a>Opción 2: Realizar manualmente las acciones de habilitación

Si ya ha ejecutado correctamente el script `Join-AzStorageAccountForAuth` anterior, vaya a la sección [Confirmación de que la característica está habilitada](#confirm-the-feature-is-enabled). No tendrá que realizar los siguientes pasos manuales.

### <a name="checking-environment"></a>Comprobación del entorno

En primer lugar, debe comprobar el estado del entorno. En concreto, compruebe si [Active Directory PowerShell](https://docs.microsoft.com/powershell/module/addsadministration/?view=win10-ps) está instalado y si el shell se ejecuta con privilegios de administrador. Después, compruebe si está instalado el [módulo Az.Storage 2.0](https://www.powershellgallery.com/packages/Az.Storage/2.0.0) y, en caso negativo, instálelo. Después de completar esas comprobaciones, compruebe la instancia de AD DS para ver si hay una [cuenta de equipo](https://docs.microsoft.com/windows/security/identity-protection/access-control/active-directory-accounts#manage-default-local-accounts-in-active-directory) (valor predeterminado) o una [cuenta de inicio de sesión de servicio](https://docs.microsoft.com/windows/win32/ad/about-service-logon-accounts) que ya se haya creado con SPN/UPN como "cifs/nombre_de_la_cuenta_de_almacenamiento.file.core.windows.net". Si la cuenta no existe, cree una como se describe en la sección siguiente.

### <a name="creating-an-identity-representing-the-storage-account-in-your-ad-manually"></a>Creación de una identidad que representa la cuenta de almacenamiento en AD manualmente

Para crear esta cuenta manualmente, cree una clave Kerberos para la cuenta de almacenamiento. A continuación, use la clave Kerberos como contraseña para su cuenta con los cmdlets de PowerShell siguientes. Esta clave solo se usa durante la configuración y no se puede utilizar para las operaciones de control o de plano de datos en la cuenta de almacenamiento. 

```PowerShell
# Create the Kerberos key on the storage account and get the Kerb1 key as the password for the AD identity to represent the storage account
$ResourceGroupName = "<resource-group-name-here>"
$StorageAccountName = "<storage-account-name-here>"

New-AzStorageAccountKey -ResourceGroupName $ResourceGroupName -Name $StorageAccountName -KeyName kerb1
Get-AzStorageAccountKey -ResourceGroupName $ResourceGroupName -Name $StorageAccountName -ListKerbKey | where-object{$_.Keyname -contains "kerb1"}
```

Cuando tenga esa clave, cree una cuenta de servicio o de equipo en la unidad organizativa. Use la especificación siguiente (recuerde reemplazar el texto de ejemplo con el nombre de la cuenta de almacenamiento):

SPN: "cifs/your-storage-account-name-here.file.core.windows.net" Contraseña: clave Kerberos de la cuenta de almacenamiento.

Si la unidad organizativa impone la expiración de la contraseña, debe actualizar la contraseña antes de su vigencia máxima para evitar errores de autenticación al acceder a recursos compartidos de archivos de Azure. Vea [Actualización de la contraseña de la identidad de la cuenta de almacenamiento en AD](storage-files-identity-ad-ds-update-password.md) para obtener más información.

Mantenga el SID de la identidad recién creada; lo necesitará en el siguiente paso. No es preciso sincronizar con Azure AD la identidad que ha creado y que representa la cuenta de almacenamiento.

### <a name="enable-the-feature-on-your-storage-account"></a>Habilitación de la característica en la cuenta de almacenamiento

Ahora puede habilitar la característica en la cuenta de almacenamiento. Proporcione algunos detalles de configuración para las propiedades de dominio en el siguiente comando y, luego, ejecútelo. El SID de la cuenta de almacenamiento necesario en el siguiente comando es el de la identidad que creó en su AD DS en [la sección anterior](#creating-an-identity-representing-the-storage-account-in-your-ad-manually).

```PowerShell
# Set the feature flag on the target storage account and provide the required AD domain information
Set-AzStorageAccount `
        -ResourceGroupName "<your-resource-group-name-here>" `
        -Name "<your-storage-account-name-here>" `
        -EnableActiveDirectoryDomainServicesForFile $true `
        -ActiveDirectoryDomainName "<your-domain-name-here>" `
        -ActiveDirectoryNetBiosDomainName "<your-netbios-domain-name-here>" `
        -ActiveDirectoryForestName "<your-forest-name-here>" `
        -ActiveDirectoryDomainGuid "<your-guid-here>" `
        -ActiveDirectoryDomainsid "<your-domain-sid-here>" `
        -ActiveDirectoryAzureStorageSid "<your-storage-account-sid>"
```

### <a name="debugging"></a>Depuración

Puede ejecutar el cmdlet Debug-AzStorageAccountAuth para realizar un conjunto de comprobaciones básicas en la configuración de AD con el usuario de AD que ha iniciado sesión. Este cmdlet se admite en AzFilesHybrid v0.1.2 y versiones posteriores. Para obtener más detalles sobre las comprobaciones realizadas en este cmdlet, vea [No se puede montar Azure Files con las credenciales de AD](storage-troubleshoot-windows-file-connection-problems.md#unable-to-mount-azure-files-with-ad-credentials) en la guía de solución de problemas de Windows.

```PowerShell
Debug-AzStorageAccountAuth -StorageAccountName $StorageAccountName -ResourceGroupName $ResourceGroupName -Verbose
```

## <a name="confirm-the-feature-is-enabled"></a>Confirmación de que la característica está habilitada

Puede confirmar que la característica está habilitada en la cuenta de almacenamiento con el siguiente script:

```PowerShell
# Get the target storage account
$storageaccount = Get-AzStorageAccount `
        -ResourceGroupName "<your-resource-group-name-here>" `
        -Name "<your-storage-account-name-here>"

# List the directory service of the selected service account
$storageAccount.AzureFilesIdentityBasedAuth.DirectoryServiceOptions

# List the directory domain information if the storage account has enabled AD DS authentication for file shares
$storageAccount.AzureFilesIdentityBasedAuth.ActiveDirectoryProperties
```

## <a name="next-steps"></a>Pasos siguientes

Ahora ha habilitado correctamente la característica en la cuenta de almacenamiento. Para usar la característica, debe asignar permisos de nivel de recurso compartido. Continúe con la sección siguiente.

[Parte 2: Asignación de permisos de nivel de recurso compartido a una identidad](storage-files-identity-ad-ds-assign-permissions.md)
