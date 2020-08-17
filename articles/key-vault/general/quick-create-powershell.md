---
title: 'Guía de inicio rápido: creación de una instancia de Azure Key Vault con Azure PowerShell'
description: Guía de inicio rápido que muestra cómo crear una instancia de Azure Key Vault mediante Azure PowerShell
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
ms.openlocfilehash: e24f1e546de1ce01896e271dbc9155c47f6c2bb6
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87101327"
---
# <a name="quickstart-create-a-key-vault-using-powershell"></a>Inicio rápido: Creación de un almacén de claves mediante PowerShell

Azure Key Vault es un servicio de almacenamiento seguro en la nube para [claves](../keys/index.yml), [secretos](../secrets/index.yml) y [certificados](../certificates/index.yml). Para obtener más información sobre Key Vault, consulte el artículo [Acerca de Azure Key Vault](overview.md); para obtener más información sobre lo que se puede almacenar en un almacén de claves, consulte el artículo [Acerca de las claves, secretos y certificados](about-keys-secrets-certificates.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

En esta guía de inicio rápido, creará un almacén de claves con [Azure PowerShell](/powershell/azure/). Si decide instalar y usar PowerShell de forma local, en este tutorial necesitará la versión 1.0.0 del módulo de Azure PowerShell, o cualquier versión posterior. Escriba `$PSVersionTable.PSVersion` para encontrar la versión. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Login-AzAccount` para crear una conexión con Azure.

```azurepowershell-interactive
Login-AzAccount
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un grupo de recursos de Azure con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. 

```azurepowershell-interactive
New-AzResourceGroup -Name 'myResourceGroup" -Location "EastUS"
```

## <a name="create-a-key-vault"></a>Creación de un Almacén de claves

Cree una instancia de Key Vault en el grupo de recursos del paso anterior. Tendrá que proporcionar algo de información:

- Nombre del almacén de claves: cadena de entre 3 y 24 caracteres que puede contener solo números (0-9), letras (a-z, A-Z) y guiones (-).

  > [!Important]
  > Cada almacén de claves debe tener un nombre único. Reemplace <nombre-almacén de claves-único> por el nombre del almacén de claves en los ejemplos siguientes.

- Nombre del grupo de recursos: **myResourceGroup**.
- Ubicación: **EastUS**.

```azurepowershell-interactive
New-AzKeyVault -Name "&lt;your-unique-key-vault-name&gt; -ResourceGroupName "myResourceGroup" -Location "East US"
```

La salida de este cmdlet muestra las propiedades del almacén de claves que acaba de crear. Tome nota de las dos propiedades siguientes:

- **Nombre del almacén**: nombre que proporcionó al parámetro --name anterior.
- **URI de almacén**: en el ejemplo, es https://&lt;su-nombre-de-almacén-de-claves-único&gt;.vault.azure.net/. Las aplicaciones que utilizan el almacén a través de su API de REST deben usar este identificador URI.

En este momento, su cuenta de Azure es la única autorizada para realizar operaciones en este nuevo almacén.

## <a name="clean-up-resources"></a>Limpieza de recursos

Otras guías de inicio rápido y tutoriales de esta colección se basan en los valores de esta. Si tiene pensado seguir trabajando en otras guías de inicio rápido y tutoriales, considere la posibilidad de dejar estos recursos activos.

Cuando ya no los necesite, puede usar el comando [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) de Azure PowerShell para quitar el grupo de recursos y todos los recursos relacionados.

```azurepowershell-interactive
Remove-AzResourceGroup -Name "myResourceGroup"
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado una instancia de Key Vault y ha almacenado un secreto en ella. Para más información sobre Key Vault y cómo integrarlo con las aplicaciones, continúe con los artículos siguientes.

- Lea una [introducción a Azure Key Vault](overview.md).
- Consulte la referencia para los [cmdlets de Key Vault de Azure PowerShell](/powershell/module/az.keyvault/?view=azps-2.6.0#key_vault).
- Consulte los [procedimientos recomendados de Azure Key Vault](best-practices.md).
