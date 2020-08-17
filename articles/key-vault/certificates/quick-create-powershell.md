---
title: 'Inicio rápido: Establecimiento y visualización de los certificados de Azure Key Vault: Azure PowerShell'
description: Inicio rápido en el que se muestra cómo establecer y recuperar un certificado de Azure Key Vault mediante Azure PowerShell
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: quickstart
ms.custom: mvc, seo-javascript-september2019, seo-javascript-october2019
ms.date: 09/03/2019
ms.author: mbaldwin
ms.openlocfilehash: efd73907a8ca691ceb451391477c9c8e2239c250
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87043906"
---
# <a name="quickstart-set-and-retrieve-a-certificate-from-azure-key-vault-using-azure-powershell"></a>Inicio rápido: Establecimiento y recuperación de un certificado de Azure Key Vault mediante Azure PowerShell

En este inicio rápido, creará un almacén de claves en Azure Key Vault con Azure PowerShell. Azure Key Vault es un servicio en la nube que funciona como un almacén de secretos seguro. Puede almacenar de forma segura claves, contraseñas, certificados y otros secretos. Para más información sobre Key Vault, puede consultar esta [introducción](../general/overview.md). Azure PowerShell se usa para crear y administrar recursos de Azure mediante comandos o scripts. Una vez que haya terminado, almacenará un certificado.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.


[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de forma local, en este tutorial necesitará la versión 1.0.0 del módulo de Azure PowerShell, o cualquier versión posterior. Escriba `$PSVersionTable.PSVersion` para encontrar la versión. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Login-AzAccount` para crear una conexión con Azure.

```azurepowershell-interactive
Login-AzAccount
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un grupo de recursos de Azure con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. 

```azurepowershell-interactive
New-AzResourceGroup -Name ContosoResourceGroup -Location EastUS
```

## <a name="create-a-key-vault"></a>Creación de un almacén de claves

A continuación, puede crear una instancia de Key Vault. Cuando se realiza este paso, se necesita cierta información:

Aunque aquí hemos usado "Contoso KeyVault2" como el nombre de la instancia de Key Vault a lo largo de este inicio rápido, usted tiene que utilizar un nombre único.

- **Nombre del almacén**: Contoso-Vault2.
- **Nombre del grupo de recursos**: ContosoResourceGroup.
- **Ubicación**: Este de EE. UU.

```azurepowershell-interactive
New-AzKeyVault -Name 'Contoso-Vault2' -ResourceGroupName 'ContosoResourceGroup' -Location 'East US'
```

La salida de este cmdlet muestra las propiedades del almacén de claves que acaba de crear. Tome nota de las dos propiedades siguientes:

* **Nombre del almacén**: en este ejemplo es **Contoso-Vault2**. Utilizará este nombre para otros cmdlets de Key Vault.
* **URI de almacén**: en este ejemplo, es https://Contoso-Vault2.vault.azure.net/. Las aplicaciones que utilizan el almacén a través de su API de REST deben usar este identificador URI.

Después de la creación del almacén, la cuenta de Azure es la única cuenta a la que se permite realizar cualquier acción en este nuevo almacén.

## <a name="add-a-certificate-to-key-vault"></a>Incorporación de un certificado a Key Vault

Para agregar un certificado al almacén, no tiene más que realizar un par de pasos adicionales. Este certificado puede ser utilizado por una aplicación. 

Escriba los siguientes comandos para crear un certificado autofirmado con una directiva denominada **ExampleCertificate**:

```azurepowershell-interactive
$Policy = New-AzKeyVaultCertificatePolicy -SecretContentType "application/x-pkcs12" -SubjectName "CN=contoso.com" -IssuerName "Self" -ValidityInMonths 6 -ReuseKeyOnRenewal
Add-AzKeyVaultCertificate -VaultName "Contoso-Vault2" -Name "ExampleCertificate" -CertificatePolicy $Policy
```

Ahora puede hacer referencia a este certificado que ha agregado a Azure Key Vault utilizando su URI. Use **https://Contoso-Vault2.vault.azure.net/certificates/ExampleCertificate** para obtener la versión actual. 

Para ver el certificado que se ha almacenado previamente:

```azurepowershell-interactive
Get-AzKeyVaultCertificate -VaultName "Contoso-Vault2" -Name "ExampleCertificate"
```

Ya ha creado una instancia de Key Vault, ha almacenado un certificado y, posteriormente, lo ha recuperado.

## <a name="clean-up-resources"></a>Limpieza de recursos

Otras guías de inicio rápido y tutoriales de esta colección se basan en los valores de esta. Si tiene pensado seguir trabajando en otras guías de inicio rápido y tutoriales, considere la posibilidad de dejar estos recursos activos.
Cuando ya no los necesite, puede usar el comando [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos y todos los recursos relacionados. Puede eliminar los recursos como se indica a continuación:

```azurepowershell-interactive
Remove-AzResourceGroup -Name ContosoResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha creado una instancia de Key Vault y ha almacenado un certificado en ella. Para más información sobre Key Vault y cómo integrarlo con las aplicaciones, continúe con los artículos siguientes.

- Lea una [introducción a Azure Key Vault](../general/overview.md).
- Consulte la referencia para los [cmdlets de Key Vault de Azure PowerShell](/powershell/module/az.keyvault/).
- Consulte los [procedimientos recomendados de Azure Key Vault](../general/best-practices.md).
