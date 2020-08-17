---
title: 'Ejemplo de script de Azure PowerShell: cifrado de una máquina virtual Windows'
description: 'Ejemplo de script de Azure PowerShell: cifrado de una máquina virtual Windows'
services: virtual-machines-windows
documentationcenter: virtual-machines
author: cynthn
manager: gwallace
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.topic: sample
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 12/12/2017
ms.author: cynthn
ms.openlocfilehash: 395adb0e4e9a01283a8b5bb47fa9cdd4ec55f66f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87058899"
---
# <a name="encrypt-a-windows-virtual-machine-with-azure-powershell"></a>Cifrado de una máquina virtual Windows con Azure PowerShell

Este script crea un almacén Azure Key Vault seguro, claves de cifrado, una entidad de servicio de Azure Active Directory y una máquina virtual (VM) Windows. La máquina virtual, a continuación, se cifra mediante la clave de cifrado del almacén de claves y las credenciales de la entidad de servicio.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

 

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell[main](../../../powershell_scripts/virtual-machine/encrypt-vm/encrypt-windows-vm.ps1 "Encrypt VM disks")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Ejecute el siguiente comando para quitar el grupo de recursos, la máquina virtual y todos los recursos relacionados.

```powershell
Remove-AzResourceGroup -Name myResourceGroup
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear la implementación. Cada elemento de la tabla incluye vínculos a la documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [New-AzKeyVault](/powershell/module/az.keyvault/new-azkeyvault) | Crea un almacén Azure Key Vault para almacenar los datos seguros, como las claves de cifrado. |
| [Add-AzKeyVaultKey](/powershell/module/az.keyvault/add-azkeyvaultkey) | Crea una clave de cifrado en el almacén Key Vault. |
| [New-AzADServicePrincipal](/powershell/module/az.resources/new-azadserviceprincipal) | Crea una entidad de servicio de Azure Active Directory para autenticar y controlar el acceso a las claves de cifrado de forma segura. |
| [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy) | Establece permisos en el almacén Key Vault para conceder acceso a la entidad de servicio a las claves de cifrado. |
| [New-AzVM](/powershell/module/az.compute/new-azvm) | Crea la máquina virtual y la conecta con la tarjeta de red, la red virtual, la subred y el grupo de seguridad de red. Este comando también abre el puerto 80 y establece las credenciales administrativas. |
| [Get-AzKeyVault](/powershell/module/az.keyvault/get-azkeyvault) | Obtiene la información necesaria del almacén de claves |
| [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) | Habilita el cifrado en una máquina virtual usando las credenciales de la entidad de servicio y la clave de cifrado. |
| [Get-AzVmDiskEncryptionStatus](/powershell/module/az.compute/get-azvmdiskencryptionstatus) | Muestra el estado del proceso de cifrado de la máquina virtual. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Quita un grupo de recursos y todos los recursos incluidos en él. |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Encontrará más ejemplos de scripts de Azure PowerShell de máquina virtual en la [documentación sobre máquinas virtuales Windows de Azure](../windows/powershell-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
