---
title: Scripts de ejemplo de Azure Disk Encryption
description: Este artículo es el apéndice de Microsoft Azure Disk Encryption para máquinas virtuales Windows.
author: msmbaldwin
ms.service: virtual-machines-windows
ms.subservice: security
ms.topic: how-to
ms.author: mbaldwin
ms.date: 08/06/2019
ms.custom: seodec18
ms.openlocfilehash: e9dc6acf33208de44eec2b5b9706b9f0b176f0d7
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87284479"
---
# <a name="azure-disk-encryption-sample-scripts"></a>Scripts de ejemplo de Azure Disk Encryption 

En este artículo se proporcionan scripts de ejemplo tanto para preparar los discos duros virtuales previamente cifrados como para realizar otras tareas.

 

## <a name="list-vms-and-secrets"></a>Enumeración de máquinas virtuales y secretos

Enumeración de todas las máquinas virtuales cifradas de una suscripción:

```azurepowershell-interactive
$osVolEncrypted = {(Get-AzVMDiskEncryptionStatus -ResourceGroupName $_.ResourceGroupName -VMName $_.Name).OsVolumeEncrypted}
$dataVolEncrypted= {(Get-AzVMDiskEncryptionStatus -ResourceGroupName $_.ResourceGroupName -VMName $_.Name).DataVolumesEncrypted}
Get-AzVm | Format-Table @{Label="MachineName"; Expression={$_.Name}}, @{Label="OsVolumeEncrypted"; Expression=$osVolEncrypted}, @{Label="DataVolumesEncrypted"; Expression=$dataVolEncrypted}
```
Enumeración de todos los secretos de cifrado de disco usados para cifrar las máquinas virtuales de un almacén de claves:

```azurepowershell-interactive
Get-AzKeyVaultSecret -VaultName $KeyVaultName | where {$_.Tags.ContainsKey('DiskEncryptionKeyFileName')} | format-table @{Label="MachineName"; Expression={$_.Tags['MachineName']}}, @{Label="VolumeLetter"; Expression={$_.Tags['VolumeLetter']}}, @{Label="EncryptionKeyURL"; Expression={$_.Id}}
```

## <a name="the-azure-disk-encryption-prerequisites-scripts"></a>Scripts de requisitos previos de Azure Disk Encryption
Si ya está familiarizado con los requisitos previos para Azure Disk Encryption, puede usar el [script de PowerShell de requisitos previos de Azure Disk Encryption](https://raw.githubusercontent.com/Azure/azure-powershell/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts/AzureDiskEncryptionPreRequisiteSetup.ps1 ). Para un ejemplo de uso de este script de PowerShell, consulte la [guía de inicio rápido para el cifrado de máquinas virtuales](disk-encryption-powershell-quickstart.md). Puede quitar los comentarios de una sección del script, a partir de la línea 211, para cifrar todos los discos de las máquinas virtuales de un grupo de recursos existente. 

En la siguiente tabla se muestran los parámetros que se pueden usar en el script de PowerShell: 

|Parámetro|Descripción|¿Obligatorio?|
|------|------|------|
|$resourceGroupName| Nombre del grupo de recursos al que pertenece la instancia de KeyVault.  Si no existe, se creará un grupo de recursos con este nombre.| True|
|$keyVaultName|Nombre de la instancia de KeyVault en donde se colocarán las claves de cifrado. Si no existe, se creará un almacén con este nombre.| True|
|$location|Ubicación de la instancia de KeyVault. Asegúrese de que la instancia de KeyVault y las máquinas virtuales que se van a cifrar están en la misma ubicación. Obtenga una lista de ubicaciones con `Get-AzLocation`.|True|
|$subscriptionId|Identificador de la suscripción de Azure que se va a utilizar.  El identificador de la suscripción se puede obtener con `Get-AzSubscription`.|True|
|$aadAppName|Nombre de la aplicación de Azure AD que se va a utilizar para escribir secretos en KeyVault. Si no existe, se creará una aplicación con este nombre. Si esta aplicación ya existe, pasará el parámetro aadClientSecret al script.|False|
|$aadClientSecret|Secreto de cliente de la aplicación de Azure AD que se creó anteriormente.|False|
|$keyEncryptionKeyName|Nombre de la clave de cifrado de clave opcional en KeyVault. Si no existe, se creará una clave con este nombre.|False|

## <a name="resource-manager-templates"></a>Plantillas de Resource Manager

### <a name="encrypt-or-decrypt-vms-without-an-azure-ad-app"></a>Cifrado o descifrado de máquinas virtuales sin aplicación de Azure AD

- [Habilitación del cifrado de disco en una máquina virtual Windows existente o en ejecución](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-windows-vm-without-aad)  
- [Deshabilitación del cifrado en una máquina virtual Windows en ejecución](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-windows-vm-without-aad) 

### <a name="encrypt-or-decrypt-vms-with-an-azure-ad-app-previous-release"></a>Cifrado o descifrado de máquinas virtuales con aplicación de Azure AD (versión anterior) 
 
- [Habilitación del cifrado de disco en una máquina virtual Windows existente o en ejecución](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-windows-vm)    
- [Deshabilitación del cifrado en una máquina virtual Windows en ejecución](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-windows-vm) 
- [Crear un disco administrado cifrado a partir de un blob de almacenamiento o disco duro virtual previamente cifrado](https://github.com/Azure/azure-quickstart-templates/tree/master/201-create-encrypted-managed-disk)
    - Crea un disco administrado cifrado a partir de un disco duro virtual previamente cifrado y su correspondiente configuración de cifrado.

## <a name="prepare-a-pre-encrypted-windows-vhd"></a>Preparación de un VHD con Windows precifrado
Las secciones siguientes son necesarias para preparar un VHD con Windows precifrado para implementarlo como VHD cifrado en IaaS de Azure. Utilice la información para preparar y arrancar una máquina virtual (VHD) con Windows nueva en Azure Site Recovery o Azure. Para más información sobre cómo preparar y cargar un disco duro virtual, consulte [Carga de un VHD generalizado y su uso para crear máquinas virtuales nuevas en Azure](upload-generalized-managed.md).

### <a name="update-group-policy-to-allow-non-tpm-for-os-protection"></a>Actualización de una directiva de grupo para permitir un módulo no TPM para la protección del sistema operativo
Configure la opción de la directiva de grupos de BitLocker denominada **Cifrado de unidad BitLocker**, que se encuentra en **Directiva de equipo local** > **Configuración del equipo** > **Plantillas administrativas** > **Componentes de Windows**. Cambie esta opción a: **Unidades del sistema operativo** > **Requerir autenticación adicional al iniciar** > **Permitir BitLocker sin un TPM compatible**, como se muestra en la ilustración siguiente:

![Microsoft Antimalware en Azure](../media/disk-encryption/disk-encryption-fig8.png)

### <a name="install-bitlocker-feature-components"></a>Instalación de componentes de características de BitLocker
Para Windows Server 2012 y versiones posteriores, utilice el siguiente comando:

```console
dism /online /Enable-Feature /all /FeatureName:BitLocker /quiet /norestart
```

Para Windows Server 2008 R2, utilice el siguiente comando:

```console
ServerManagerCmd -install BitLockers
```

### <a name="prepare-the-os-volume-for-bitlocker-by-using-bdehdcfg"></a>Preparación del volumen del sistema operativo para BitLocker mediante `bdehdcfg`
Si es necesario, ejecute el comando [bdehdcfg](/windows/security/information-protection/bitlocker/bitlocker-basic-deployment) para comprimir la partición del sistema operativo y preparar la máquina para BitLocker:

```console
bdehdcfg -target c: shrink -quiet 
```

### <a name="protect-the-os-volume-by-using-bitlocker"></a>Protección del volumen del sistema operativo mediante BitLocker
Use el comando [`manage-bde`](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/ff829849(v=ws.11)) para habilitar el cifrado en el volumen de arranque mediante un protector de clave externo. También puede colocar la clave externa (archivo .bek) en el disco externo o el volumen. El cifrado se habilita en el volumen de sistema o de arranque la próxima vez que se reinicie el equipo.

```console
manage-bde -on %systemdrive% -sk [ExternalDriveOrVolume]
reboot
```

> [!NOTE]
> Prepare la máquina virtual con un VHD de datos o recursos separado para obtener la clave externa mediante BitLocker.

## <a name="upload-encrypted-vhd-to-an-azure-storage-account"></a>Carga de un VHD cifrado a una cuenta de almacenamiento de Azure
Una vez que se cifre DM-Crypt, el disco duro virtual cifrado local debe cargarse en la cuenta de almacenamiento.
```powershell
    Add-AzVhd [-Destination] <Uri> [-LocalFilePath] <FileInfo> [[-NumberOfUploaderThreads] <Int32> ] [[-BaseImageUriToPatch] <Uri> ] [[-OverWrite]] [ <CommonParameters>]
```

## <a name="upload-the-secret-for-the-pre-encrypted-vm-to-your-key-vault"></a>Carga del secreto de la máquina virtual previamente cifrada en el almacén de claves
El secreto de cifrado del disco que obtuvo con anterioridad se debe cargar como un secreto en el almacén de claves.  Para ello, es preciso conceder el permiso para establecer secretos como el permiso para encapsular claves a la cuenta que cargará los secretos.

```powershell 
# Typically, account Id is the user principal name (in user@domain.com format)
$upn = (Get-AzureRmContext).Account.Id
Set-AzKeyVaultAccessPolicy -VaultName $kvname -UserPrincipalName $acctid -PermissionsToKeys wrapKey -PermissionsToSecrets set

# In cloud shell, the account ID is a managed service identity, so specify the username directly 
# $upn = "user@domain.com" 
# Set-AzKeyVaultAccessPolicy -VaultName $kvname -UserPrincipalName $acctid -PermissionsToKeys wrapKey -PermissionsToSecrets set

# When running as a service principal, retrieve the service principal ID from the account ID, and set access policy to that 
# $acctid = (Get-AzureRmContext).Account.Id
# $spoid = (Get-AzureRmADServicePrincipal -ServicePrincipalName $acctid).Id
# Set-AzKeyVaultAccessPolicy -VaultName $kvname -ObjectId $spoid -BypassObjectIdValidation -PermissionsToKeys wrapKey -PermissionsToSecrets set

```

### <a name="disk-encryption-secret-not-encrypted-with-a-kek"></a>Secreto del cifrado de disco no cifrado con una KEK
Para configurar el secreto en el almacén de claves, use [Set-AzKeyVaultSecret](/powershell/module/az.keyvault/set-azkeyvaultsecret). La frase de contraseña se codifica en forma de cadena base64 y, después, se carga en el almacén de claves. Además, asegúrese de que se establecen las siguientes etiquetas al crear el secreto en el almacén de claves.

```powershell

 # This is the passphrase that was provided for encryption during the distribution installation
 $passphrase = "contoso-password"

 $tags = @{"DiskEncryptionKeyEncryptionAlgorithm" = "RSA-OAEP"; "DiskEncryptionKeyFileName" = "LinuxPassPhraseFileName"}
 $secretName = [guid]::NewGuid().ToString()
 $secretValue = [Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes($passphrase))
 $secureSecretValue = ConvertTo-SecureString $secretValue -AsPlainText -Force

 $secret = Set-AzKeyVaultSecret -VaultName $KeyVaultName -Name $secretName -SecretValue $secureSecretValue -tags $tags
 $secretUrl = $secret.Id
```


Use `$secretUrl` en el paso siguiente para [conectar el disco del sistema operativo sin usar KEK](#without-using-a-kek).

### <a name="disk-encryption-secret-encrypted-with-a-kek"></a>Secreto del cifrado de disco cifrado con una KEK
Antes de cargar el secreto en el almacén de claves, puede cifrarlo si lo desea mediante una clave de cifrado de claves. Utilice la [API](/rest/api/keyvault/wrapkey) de encapsulamiento para cifrar por primera vez el secreto mediante la clave de cifrado de claves. El resultado de esta operación de encapsulamiento es una cadena codificada en URL como base64 que luego se carga como secreto con el cmdlet [`Set-AzKeyVaultSecret`](/powershell/module/az.keyvault/set-azkeyvaultsecret).

```powershell
    # This is the passphrase that was provided for encryption during the distribution installation
    $passphrase = "contoso-password"

    Add-AzKeyVaultKey -VaultName $KeyVaultName -Name "keyencryptionkey" -Destination Software
    $KeyEncryptionKey = Get-AzKeyVaultKey -VaultName $KeyVault.OriginalVault.Name -Name "keyencryptionkey"

    $apiversion = "2015-06-01"

    ##############################
    # Get Auth URI
    ##############################

    $uri = $KeyVault.VaultUri + "/keys"
    $headers = @{}

    $response = try { Invoke-RestMethod -Method GET -Uri $uri -Headers $headers } catch { $_.Exception.Response }

    $authHeader = $response.Headers["www-authenticate"]
    $authUri = [regex]::match($authHeader, 'authorization="(.*?)"').Groups[1].Value

    Write-Host "Got Auth URI successfully"

    ##############################
    # Get Auth Token
    ##############################

    $uri = $authUri + "/oauth2/token"
    $body = "grant_type=client_credentials"
    $body += "&client_id=" + $AadClientId
    $body += "&client_secret=" + [Uri]::EscapeDataString($AadClientSecret)
    $body += "&resource=" + [Uri]::EscapeDataString("https://vault.azure.net")
    $headers = @{}

    $response = Invoke-RestMethod -Method POST -Uri $uri -Headers $headers -Body $body

    $access_token = $response.access_token

    Write-Host "Got Auth Token successfully"

    ##############################
    # Get KEK info
    ##############################

    $uri = $KeyEncryptionKey.Id + "?api-version=" + $apiversion
    $headers = @{"Authorization" = "Bearer " + $access_token}

    $response = Invoke-RestMethod -Method GET -Uri $uri -Headers $headers

    $keyid = $response.key.kid

    Write-Host "Got KEK info successfully"

    ##############################
    # Encrypt passphrase using KEK
    ##############################

    $passphraseB64 = [Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes($Passphrase))
    $uri = $keyid + "/encrypt?api-version=" + $apiversion
    $headers = @{"Authorization" = "Bearer " + $access_token; "Content-Type" = "application/json"}
    $bodyObj = @{"alg" = "RSA-OAEP"; "value" = $passphraseB64}
    $body = $bodyObj | ConvertTo-Json

    $response = Invoke-RestMethod -Method POST -Uri $uri -Headers $headers -Body $body

    $wrappedSecret = $response.value

    Write-Host "Encrypted passphrase successfully"

    ##############################
    # Store secret
    ##############################

    $secretName = [guid]::NewGuid().ToString()
    $uri = $KeyVault.VaultUri + "/secrets/" + $secretName + "?api-version=" + $apiversion
    $secretAttributes = @{"enabled" = $true}
    $secretTags = @{"DiskEncryptionKeyEncryptionAlgorithm" = "RSA-OAEP"; "DiskEncryptionKeyFileName" = "LinuxPassPhraseFileName"}
    $headers = @{"Authorization" = "Bearer " + $access_token; "Content-Type" = "application/json"}
    $bodyObj = @{"value" = $wrappedSecret; "attributes" = $secretAttributes; "tags" = $secretTags}
    $body = $bodyObj | ConvertTo-Json

    $response = Invoke-RestMethod -Method PUT -Uri $uri -Headers $headers -Body $body

    Write-Host "Stored secret successfully"

    $secretUrl = $response.id
```

Use `$KeyEncryptionKey` y `$secretUrl` en el paso siguiente para [conectar el disco del sistema operativo usando KEK](#using-a-kek).

##  <a name="specify-a-secret-url-when-you-attach-an-os-disk"></a>Especificación de una dirección URL de secreto al adjuntar un disco del sistema operativo

###  <a name="without-using-a-kek"></a>Sin utilizar una KEK
Mientras esté adjuntando el disco del sistema operativo, tiene que pasar `$secretUrl`. La dirección URL se generó en la sección "Secreto de cifrado de disco no cifrado con una KEK".
```powershell
    Set-AzVMOSDisk `
            -VM $VirtualMachine `
            -Name $OSDiskName `
            -SourceImageUri $VhdUri `
            -VhdUri $OSDiskUri `
            -Windows `
            -CreateOption FromImage `
            -DiskEncryptionKeyVaultId $KeyVault.ResourceId `
            -DiskEncryptionKeyUrl $SecretUrl
```
### <a name="using-a-kek"></a>Uso de una KEK
Cuando conecte un disco del sistema operativo, pase `$KeyEncryptionKey` y `$secretUrl`. La dirección URL se generó en la sección "Secreto del cifrado de disco cifrado con KEK".
```powershell
    Set-AzVMOSDisk `
            -VM $VirtualMachine `
            -Name $OSDiskName `
            -SourceImageUri $CopiedTemplateBlobUri `
            -VhdUri $OSDiskUri `
            -Windows `
            -CreateOption FromImage `
            -DiskEncryptionKeyVaultId $KeyVault.ResourceId `
            -DiskEncryptionKeyUrl $SecretUrl `
            -KeyEncryptionKeyVaultId $KeyVault.ResourceId `
            -KeyEncryptionKeyURL $KeyEncryptionKey.Id
```
