---
title: 'Script de PowerShell: buscar el almacén de la cuenta de almacenamiento'
description: Aprenda a usar un script de Azure PowerShell para buscar el almacén de Recovery Services en el que está registrada la cuenta de almacenamiento.
ms.topic: sample
ms.date: 1/28/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 40859c1ea05210d27fcdcf33ba9d4f961965ea22
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89075705"
---
# <a name="powershell-script-to-find-the-recovery-services-vault-where-a-storage-account-is-registered"></a>Script de PowerShell para buscar el almacén de Recovery Services donde está registrada una cuenta de almacenamiento

Este script le ayudará a encontrar el almacén de Recovery Services donde está registrada la cuenta de almacenamiento.

## <a name="sample-script"></a>Script de ejemplo

```powershell
Param(
        [Parameter(Mandatory=$True)][System.String] $ResourceGroupName,
        [Parameter(Mandatory=$True)][System.String] $StorageAccountName,
        [Parameter(Mandatory=$True)][System.String] $SubscriptionId
    )

Connect-AzAccount
Select-AzSubscription -Subscription $SubscriptionId
$vaults = Get-AzRecoveryServicesVault
$found = $false
foreach($vault in $vaults)
{
  Write-Verbose "Checking vault: $($vault.Id)" -Verbose
  
  $containers = Get-AzRecoveryServicesBackupContainer -ContainerType AzureStorage -FriendlyName $StorageAccountName -ResourceGroupName $ResourceGroupName -VaultId $vault.Id -Status Registered
  
  if($containers -ne $null)
  {
    $found = $True
    Write-Information "Found Storage account $StorageAccountName registered in vault: $($vault.Id)" -InformationAction Continue
    break;
  }
}

if(!$found)
{
     Write-Information "Storage account: $StorageAccountName is not registered in any vault of this subscription" -InformationAction Continue
}
```

## <a name="how-to-execute-the-script"></a>Cómo ejecutar el script

1. Guarde el script anterior en el equipo con el nombre que prefiera. En este ejemplo, lo hemos guardado como *FindRegisteredStorageAccount.ps1*.
2. Ejecute el script con los parámetros siguientes:

    * **-ResourceGroupName**: grupo de recursos de la cuenta de almacenamiento.
    * **-StorageAccountName**: nombre de la cuenta de almacenamiento.
    * **-SubscriptionID**: identificador de la suscripción donde se encuentra la cuenta de almacenamiento.

En el ejemplo siguiente se intenta encontrar el almacén de Recovery Services en el que se ha registrado la cuenta de almacenamiento *afsaccount*:

```powershell
.\FindRegisteredStorageAccount.ps1 -ResourceGroupName AzureFiles -StorageAccountName afsaccount -SubscriptionId ef4ad5a7-c2c0-4304-af80-af49f49af3d1
```

## <a name="output"></a>Output

La salida mostrará la ruta de acceso completa del almacén de Recovery Services en el que está registrada la cuenta de almacenamiento. Este es una salida de ejemplo:

```output
Found Storage account afsaccount registered in vault: /subscriptions/ ef4ad5a7-c2c0-4304-af80-af49f49af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault123
```

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo [hacer copias de seguridad de los recursos compartidos de archivos de Azure desde Azure Portal](../backup-afs.md).
