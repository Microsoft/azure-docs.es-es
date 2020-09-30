---
title: Exportación de certificados de Azure Key Vault
description: Aprenda a exportar certificados de Azure Key Vault.
services: key-vault
author: sebansal
tags: azure-key-vault
ms.service: key-vault
ms.subservice: certificates
ms.topic: how-to
ms.custom: mvc, devx-track-azurecli
ms.date: 08/11/2020
ms.author: sebansal
ms.openlocfilehash: ae9891edc80aab5d3223686547dfef4273f8a921
ms.sourcegitcommit: 7f62a228b1eeab399d5a300ddb5305f09b80ee14
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/08/2020
ms.locfileid: "89514635"
---
# <a name="export-certificates-from-azure-key-vault"></a>Exportación de certificados de Azure Key Vault

Aprenda a exportar certificados de Azure Key Vault. Para exportar certificados se deben usar Azure Portal, Azure PowerShell o la CLI de Azure. Azure Portal también se puede usar para exportar certificados de Azure App Service.

## <a name="about-azure-key-vault-certificates"></a>Acerca de los certificados de Azure Key Vault

Azure Key Vault permite aprovisionar, administrar e implementar fácilmente certificados digitales en cualquier red. También habilita las comunicaciones seguras en las aplicaciones. Para más información, consulte [Certificados de Azure Key Vault](https://docs.microsoft.com/azure/key-vault/certificates/about-certificates).

### <a name="composition-of-a-certificate"></a>Composición de un certificado

Cuando se crea un certificado de Key Vault, se crean también una *clave* direccionable y un *secreto* que se llaman igual. La clave de Key Vault permite operaciones con clave. El secreto de Key Vault permite la recuperación del valor del certificado como un secreto. Un certificado de Key Vault también contiene metadatos del certificado X.509 público. Para más información, vaya a la sección [Composición de un certificado](https://docs.microsoft.com/azure/key-vault/certificates/about-certificates#composition-of-a-certificate).

### <a name="exportable-and-non-exportable-keys"></a>Claves exportables y no exportables

Tras crear un certificado de Key Vault, se puede recuperar desde el secreto direccionable con la clave privada. Recupere el certificado en formato PFX o PEM.

- **Exportable**: la directiva que se utiliza para crear el certificado indica que la clave se puede exportar.
- **No exportable**: la directiva que se utiliza para crear el certificado indica que la clave no se puede exportar. En este caso, la clave privada no forma parte del valor cuando se recupera como un secreto.

Tipos de clave permitidos: RSA, RSA-HSM, EC, EC-HSM, Oct (se enumeran [aquí](https://docs.microsoft.com/rest/api/keyvault/createcertificate/createcertificate#jsonwebkeytype)). Exportable solo se permite con RSA y EC. Las claves HSM serían no exportables.

Para más información, consulte [Acerca de los certificados de Azure Key Vault](https://docs.microsoft.com/azure/key-vault/certificates/about-certificates#exportable-or-non-exportable-key).

## <a name="export-stored-certificates"></a>Exportación de certificados almacenados

Mediante la CLI de Azure, Azure Portal o Azure PowerShell se pueden exportar certificados almacenados en Azure Key Vault.

> [!NOTE]
> Cuando el certificado se importa en un almacén de claves solo se requiere una contraseña de certificado. Key Vault no guarda la contraseña asociada. Al exportar el certificado, la contraseña está en blanco.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Use el siguiente comando de la CLI de Azure para descargar la **parte pública** de un certificado de Key Vault.

```azurecli
az keyvault certificate download --file
                                 [--encoding {DER, PEM}]
                                 [--id]
                                 [--name]
                                 [--subscription]
                                 [--vault-name]
                                 [--version]
```

Para más información, vea [ejemplos y definiciones de parámetros](https://docs.microsoft.com/cli/azure/keyvault/certificate?view=azure-cli-latest#az-keyvault-certificate-download).

Si desea descargar todo el certificado (tanto la parte pública como la parte privada de su composición), descargue el certificado como un secreto.

```azurecli
az keyvault secret download -–file {nameofcert.pfx}
                            [--encoding {ascii, base64, hex, utf-16be, utf-16le, utf-8}]
                            [--id]
                            [--name]
                            [--subscription]
                            [--vault-name]
                            [--version]
```

Para más información, vea las [definiciones de los parámetros](https://docs.microsoft.com/cli/azure/keyvault/secret?view=azure-cli-latest#az-keyvault-secret-download).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Use este comando en Azure PowerShell para obtener el certificado denominado **TestCert01** del almacén de claves denominado **ContosoKV01**. Para descargar el certificado en forma de archivo PFX, ejecute el siguiente comando. Estos comandos acceden a **SecretId** y, después, guardan el contenido en forma de archivo PFX.

```azurepowershell
$cert = Get-AzKeyVaultCertificate -VaultName "ContosoKV01" -Name "TestCert01"
$kvSecret = Get-AzKeyVaultSecret -VaultName "ContosoKV01" -Name $Cert.Name
$kvSecretBytes = [System.Convert]::FromBase64String($kvSecret.SecretValueText)
$certCollection = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2Collection
$certCollection.Import($kvSecretBytes,$null,[System.Security.Cryptography.X509Certificates.X509KeyStorageFlags]::Exportable)
$password = '******'
$protectedCertificateBytes = $certCollection.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pkcs12, $password)
$pfxPath = [Environment]::GetFolderPath("Desktop") + "\MyCert.pfx"
[System.IO.File]::WriteAllBytes($pfxPath, $protectedCertificateBytes)
```

Este comando exporta toda la cadena de certificados con clave privada. El certificado está protegido con contraseña.
Para más información sobre el comando **Get-AzKeyVaultCertificate** y sus parámetros, consulte [Get-AzKeyVaultCertificate: ejemplo 2](https://docs.microsoft.com/powershell/module/az.keyvault/Get-AzKeyVaultCertificate?view=azps-4.4.0).

# <a name="portal"></a>[Portal](#tab/azure-portal)

En Azure Portal, tras crear o importar un certificado en la hoja **Certificado**, se recibe la notificación de que se ha creado correctamente. Seleccione el certificado y la versión actual para ver la opción que debe descargar.

Para descargar el certificado, seleccione **Descargar en formato CER** o **Descargar en formato PFX/PEM**.

![Descarga del certificado](../media/certificates/quick-create-portal/current-version-shown.png)

**Exportación de certificados de Azure App Service**

Los certificados de Azure App Service son una forma cómoda de comprar certificados SSL. Se pueden asignar a aplicaciones de Azure desde el portal. Estos certificados también se pueden exportar desde el portal en forma de archivos PFX para usarse en otro lugar. Después de importarlos, los certificados de App Service se encuentran en **secrets**.

Para más información, consulte los pasos necesarios para [exportar los certificados de Azure App Service](https://social.technet.microsoft.com/wiki/contents/articles/37431.exporting-azure-app-service-certificates.aspx).

---

## <a name="read-more"></a>Más información
* [Diversos tipos y definiciones de archivos de certificado](https://docs.microsoft.com/archive/blogs/kaushal/various-ssltls-certificate-file-typesextensions)
