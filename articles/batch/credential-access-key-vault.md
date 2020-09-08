---
title: Acceso seguro a Key Vault con Batch
description: Obtenga información sobre cómo tener acceso mediante programación a las credenciales de Key Vault mediante Azure Batch.
ms.topic: how-to
ms.date: 02/13/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 6938d0fcd2357efcf03053b0c9b2bde3954270b7
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89079445"
---
# <a name="securely-access-key-vault-with-batch"></a>Acceso seguro a Key Vault con Batch

En este artículo, aprenderá a configurar nodos de Batch para acceder de forma segura a las credenciales almacenadas en Azure Key Vault. No tiene sentido poner las credenciales de administrador en Key Vault y, a continuación, codificar de forma rígida las credenciales para tener acceso a Key Vault desde un script. La solución consiste en usar un certificado que conceda a los nodos de Batch acceso a Key Vault. Con unos pocos pasos, podemos implementar el almacenamiento de claves seguro para Batch.

Para autenticarse en Azure Key Vault desde un nodo de Batch, necesita:

- Una credencial de Azure Active Directory (Azure AD)
- Un certificado
- Una cuenta de Batch
- Un grupo de Batch con al menos un nodo

## <a name="obtain-a-certificate"></a>Obtención de un certificado

Si aún no tiene un certificado, la forma más sencilla de obtenerlo es generar un certificado autofirmado mediante la herramienta de línea de comandos `makecert`.

Normalmente, puede encontrar `makecert` en esta ruta de acceso: `C:\Program Files (x86)\Windows Kits\10\bin\<arch>`. Abra un símbolo del sistema como administrador y vaya a `makecert` mediante el ejemplo siguiente.

```console
cd C:\Program Files (x86)\Windows Kits\10\bin\x64
```

A continuación, use la herramienta `makecert` para crear archivos de certificado autofirmados denominados `batchcertificate.cer` y `batchcertificate.pvk`. El nombre común (cn) utilizado no es importante para esta aplicación, pero resulta útil si le indica en qué se usa el certificado.

```console
makecert -sv batchcertificate.pvk -n "cn=batch.cert.mydomain.org" batchcertificate.cer -b 09/23/2019 -e 09/23/2019 -r -pe -a sha256 -len 2048
```

Batch requiere un archivo de `.pfx`. Use la herramienta [pvk2pfx](/windows-hardware/drivers/devtest/pvk2pfx) para convertir los archivos `.cer` y `.pvk` creados por `makecert` en un solo archivo `.pfx`.

```console
pvk2pfx -pvk batchcertificate.pvk -spc batchcertificate.cer -pfx batchcertificate.pfx -po
```

## <a name="create-a-service-principal"></a>Creación de una entidad de servicio

El acceso a Key Vault se concede a un **usuario** o a una **entidad de servicio**. Para acceder a Key Vault mediante programación, use una entidad de servicio con el certificado que se creó en el paso anterior.

Para más información sobre las entidades de servicio de Azure, consulte [Objetos de aplicación y de entidad de servicio de Azure Active Directory](../active-directory/develop/app-objects-and-service-principals.md).

> [!NOTE]
> La entidad de servicio debe estar en el mismo inquilino de Azure AD que Key Vault.

```powershell
$now = [System.DateTime]::Parse("2020-02-10")
# Set this to the expiration date of the certificate
$expirationDate = [System.DateTime]::Parse("2021-02-10")
# Point the script at the cer file you created $cerCertificateFilePath = 'c:\temp\batchcertificate.cer'
$cer = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2
$cer.Import($cerCertificateFilePath)
# Load the certificate into memory
$credValue = [System.Convert]::ToBase64String($cer.GetRawCertData())
# Create a new AAD application that uses this certificate
$newADApplication = New-AzureRmADApplication -DisplayName "Batch Key Vault Access" -HomePage "https://batch.mydomain.com" -IdentifierUris "https://batch.mydomain.com" -certValue $credValue -StartDate $now -EndDate $expirationDate
# Create new AAD service principal that uses this application
$newAzureAdPrincipal = New-AzureRmADServicePrincipal -ApplicationId $newADApplication.ApplicationId
```

Las direcciones URL de la aplicación no son importantes, ya que solo se usan para el acceso a Key Vault.

## <a name="grant-rights-to-key-vault"></a>Concesión de derechos a Key Vault

La entidad de servicio creada en el paso anterior necesita permiso para recuperar los secretos de Key Vault. Los permisos se pueden conceder mediante Azure Portal o con el siguiente comando de PowerShell.

```powershell
Set-AzureRmKeyVaultAccessPolicy -VaultName 'BatchVault' -ServicePrincipalName '"https://batch.mydomain.com' -PermissionsToSecrets 'Get'
```

## <a name="assign-a-certificate-to-a-batch-account"></a>Asignación de un certificado a una cuenta de Batch

Cree un grupo de Batch y, a continuación, vaya a la pestaña Certificado del grupo y asigne el certificado que creó. El certificado está ahora en todos los nodos de Batch.

A continuación, es necesario asignar el certificado a la cuenta de Batch. La asignación del certificado a la cuenta nos permite asignarlo a los grupos y, a continuación, a los nodos. La forma más fácil de hacerlo es ir a la cuenta de Batch en el portal, ir a **Certificados** y seleccionar **Agregar**. Cargue el archivo `.pfx` generado en el paso [Obtención de un certificado](#obtain-a-certificate) y proporcione la contraseña. Una vez completado, el certificado se agrega a la lista y puede comprobar la huella digital.

Ahora, cuando cree un grupo de Batch, puede ir a **Certificados** dentro del grupo y asignar el certificado que creó a ese grupo. Al hacerlo, asegúrese de seleccionar **LocalMachine** para la ubicación del almacén. El certificado se carga en todos los nodos de Batch del grupo.

## <a name="install-azure-powershell"></a>Instalar Azure Powershell

Si planea acceder a Key Vault mediante scripts de PowerShell en los nodos, deberá tener instalada la biblioteca de Azure PowerShell. Hay varias maneras de hacerlo; si los nodos tienen instalado Windows Management Framework (WMF) 5, puede usar el comando install-module para descargarlo. Si usa nodos que no tienen WMF 5, la forma más fácil de instalarlo es agrupar el archivo `.msi` de Azure PowerShell con los archivos de Batch y, a continuación, llamar al instalador como la primera parte del script de inicio de Batch. Para más información, vea este ejemplo:

```powershell
$psModuleCheck=Get-Module -ListAvailable -Name Azure -Refresh
if($psModuleCheck.count -eq 0) {
    $psInstallerPath = Join-Path $downloadPath "azure-powershell.3.4.0.msi" Start-Process msiexec.exe -ArgumentList /i, $psInstallerPath, /quiet -wait
}
```

## <a name="access-key-vault"></a>Acceda a Key Vault.

Ahora ya está todo listo para acceder a Key Vault en scripts que se ejecutan en nodos de Batch. Para acceder a Key Vault desde un script, lo único que necesita es que el script se autentique en Azure AD mediante el certificado. Para hacer esto en PowerShell, use los siguientes comandos de ejemplo. Especifique el GUID adecuado para **Huella digital**, **Id. de aplicación** (el identificador de la entidad de servicio) e **Id. del inquilino** (el inquilino en el que existe la entidad de servicio).

```powershell
Add-AzureRmAccount -ServicePrincipal -CertificateThumbprint -ApplicationId
```

Una vez autenticado, acceda a KeyVault como lo haría normalmente.

```powershell
$adminPassword=Get-AzureKeyVaultSecret -VaultName BatchVault -Name batchAdminPass
```

Estas son las credenciales que se usan en el script.
