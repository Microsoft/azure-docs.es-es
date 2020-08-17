---
title: 'Inicio rápido: Establecer y recuperar un secreto de Key Vault mediante PowerShell'
description: En esta guía de inicio rápido, aprenderá a crear, recuperar y eliminar secretos desde una instancia de Azure Key Vault mediante Azure PowerShell.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.custom: mvc
ms.date: 11/08/2019
ms.author: mbaldwin
ms.openlocfilehash: 9e3f5e37462b30b0201be5350dedc103a49bb39a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87039300"
---
# <a name="quickstart-set-and-retrieve-a-secret-from-azure-key-vault-using-powershell"></a>Inicio rápido: Establecimiento y recuperación de un secreto de Azure Key Vault mediante PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Azure Key Vault es un servicio en la nube que funciona como un almacén de secretos seguro. Puede almacenar de forma segura claves, contraseñas, certificados y otros secretos. Para más información sobre Key Vault, puede consultar esta [introducción](../general/overview.md). En esta guía de inicio rápido, va a usar PowerShell para crear un almacén de claves. Posteriormente, va a almacenar un secreto en el almacén recién creado.

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

## <a name="give-your-user-account-permissions-to-manage-secrets-in-key-vault"></a>Conceda a su cuenta de usuario los permisos necesarios para administrar secretos en Key Vault

Use el cmdlet Set-AzKeyVaultAccessPolicy de Azure PowerShell para actualizar la directiva de acceso de Key Vault y conceder permisos de secreto a su cuenta de usuario.
```azurepowershell-interactive
Set-AzKeyVaultAccessPolicy -VaultName 'Contoso-Vault2' -UserPrincipalName 'user@domain.com' -PermissionsToSecrets get,set,delete
```

## <a name="adding-a-secret-to-key-vault"></a>Incorporación de un secreto a Key Vault

Para agregar un secreto al almacén, solo tiene que realizar un par de pasos. En este caso, ha agregado una contraseña que una aplicación podría usar. La contraseña se denomina **ExamplePassword** y almacena el valor de **hVFkk965BuUv**.

Primero, para convertir el valor de **hVFkk965BuUv** en una cadena segura, escriba:

```azurepowershell-interactive
$secretvalue = ConvertTo-SecureString 'hVFkk965BuUv' -AsPlainText -Force
```

A continuación, escriba los siguientes comandos de PowerShell para crear un secreto en Key Vault denominado **ExamplePassword** con el valor **hVFkk965BuUv**:

```azurepowershell-interactive
$secret = Set-AzKeyVaultSecret -VaultName 'Contoso-Vault2' -Name 'ExamplePassword' -SecretValue $secretvalue
```

Para ver el valor contenido en el secreto como texto sin formato:

```azurepowershell-interactive
(Get-AzKeyVaultSecret -vaultName "Contoso-Vault2" -name "ExamplePassword").SecretValueText
```

Ya ha creado una instancia de Key Vault, ha almacenado un secreto y, posteriormente, lo ha recuperado.

## <a name="clean-up-resources"></a>Limpieza de recursos

 Otras guías de inicio rápido y tutoriales de esta colección se basan en los valores de esta. Si tiene pensado seguir trabajando en otras guías de inicio rápido y tutoriales, considere la posibilidad de dejar estos recursos activos.

Cuando ya no los necesite, puede usar el comando [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos, la instancia de Key Vault y todos los recursos relacionados.

```azurepowershell-interactive
Remove-AzResourceGroup -Name ContosoResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado una instancia de Key Vault y ha almacenado un secreto en ella. Para más información sobre Key Vault y cómo integrarlo con las aplicaciones, continúe con los artículos siguientes.

- Lea una [introducción a Azure Key Vault](../general/overview.md).
- Consulte la referencia para los [cmdlets de Key Vault de Azure PowerShell](/powershell/module/az.keyvault/?view=azps-2.6.0#key_vault).
- Consulte los [procedimientos recomendados de Azure Key Vault](../general/best-practices.md).
