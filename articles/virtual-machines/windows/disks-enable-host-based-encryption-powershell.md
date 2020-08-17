---
title: 'Azure PowerShell: habilitación del cifrado de un extremo a otro en el host de máquina virtual'
description: Habilitación del cifrado de un extremo a otro para las máquinas virtuales de Azure mediante el cifrado en el host.
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 07/10/2020
ms.author: rogarana
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: 6cb6235c5c1a34cb3f48d315adee565591bb72c4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87088468"
---
# <a name="enable-end-to-end-encryption-using-encryption-at-host---azure-powershell"></a>Habilitación del cifrado de un extremo a otro mediante cifrado en el host - Azure PowerShell

Cuando se habilita el cifrado en el host, los datos almacenados en el host de máquina virtual se cifran en reposo y se transmiten cifrados al servido Storage. Para obtener información conceptual sobre el cifrado en el host, así como otros tipos de cifrado de disco administrado, consulte [Cifrado en host: cifrado de un extremo a otro para los datos de la VM](disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data).

## <a name="restrictions"></a>Restricciones

[!INCLUDE [virtual-machines-disks-encryption-at-host-restrictions](../../../includes/virtual-machines-disks-encryption-at-host-restrictions.md)]

### <a name="supported-regions"></a>Regiones admitidas

[!INCLUDE [virtual-machines-disks-encryption-at-host-regions](../../../includes/virtual-machines-disks-encryption-at-host-regions.md)]

### <a name="supported-vm-sizes"></a>Tamaños de máquinas virtuales que se admiten

[!INCLUDE [virtual-machines-disks-encryption-at-host-suported-sizes](../../../includes/virtual-machines-disks-encryption-at-host-suported-sizes.md)]

También puede encontrar los tamaños de máquina virtual mediante programación. Para obtener información sobre cómo recuperarlos mediante programación, consulte la sección [Búsqueda de tamaños de máquinas virtuales admitidos](#finding-supported-vm-sizes).

## <a name="prerequisites"></a>Requisitos previos

Para poder usar el cifrado en el host para las máquinas virtuales o los conjuntos de escalado de máquinas virtuales, debe habilitar la característica en la suscripción. Envíe un correo electrónico a encryptionAtHost@microsoft.com con los identificadores de suscripción para que la característica se habilite para sus suscripciones.

### <a name="create-an-azure-key-vault-and-diskencryptionset"></a>Creación de una instancia de Azure Key Vault y DiskEncryptionSet

Una vez habilitada la característica, deberá configurar un almacén de Azure Key Vault y un elemento DiskEncryptionSet, si no lo ha hecho aún.

[!INCLUDE [virtual-machines-disks-encryption-create-key-vault-powershell](../../../includes/virtual-machines-disks-encryption-create-key-vault-powershell.md)]

## <a name="enable-encryption-at-host-for-disks-attached-to-vm-and-virtual-machine-scale-sets"></a>Habilitación del cifrado en el host para discos conectados a máquinas virtuales y conjuntos de escalado de máquinas virtuales

Puede habilitar el cifrado en el host estableciendo una nueva propiedad EncryptionAtHost en securityProfile de máquinas virtuales o en conjuntos de escalado de máquinas virtuales con la versión de API **2020-06-01** y posteriores.

`"securityProfile": { "encryptionAtHost": "true" }`

## <a name="example-scripts"></a>Scripts de ejemplo

### <a name="enable-encryption-at-host-for-disks-attached-to-a-vm-with-customer-managed-keys"></a>Habilitación del cifrado en el host para los discos conectados a una máquina virtual con claves administradas por el cliente

Cree una máquina virtual con discos administrados mediante el URI de recurso del elemento DiskEncryptionSet creado anteriormente.

Reemplace `<yourPassword>`, `<yourVMName>`, `<yourVMSize>`, `<yourDESName>`, `<yoursubscriptionID>`, `<yourResourceGroupName>` y `<yourRegion>` y, a continuación, ejecute el script.

```PowerShell
$password=ConvertTo-SecureString -String "<yourPassword>" -AsPlainText -Force
New-AzResourceGroupDeployment -ResourceGroupName <yourResourceGroupName> `
-TemplateUri "https://raw.githubusercontent.com/Azure-Samples/managed-disks-powershell-getting-started/master/EncryptionAtHost/CreateVMWithDisksEncryptedAtHostWithCMK.json" `
-virtualMachineName "<yourVMName>" `
-adminPassword $password `
-vmSize "<yourVMSize>" `
-diskEncryptionSetId "/subscriptions/<yoursubscriptionID>/resourceGroups/<yourResourceGroupName>/providers/Microsoft.Compute/diskEncryptionSets/<yourDESName>" `
-region "<yourRegion>"
```

### <a name="enable-encryption-at-host-for-disks-attached-to-a-vm-with-platform-managed-keys"></a>Habilitación del cifrado en el host para los discos conectados a una máquina virtual con claves administradas por la plataforma

Reemplace `<yourPassword>`, `<yourVMName>`, `<yourVMSize>`, `<yourResourceGroupName>` y `<yourRegion>` y, a continuación, ejecute el script.

```PowerShell
$password=ConvertTo-SecureString -String "<yourPassword>" -AsPlainText -Force
New-AzResourceGroupDeployment -ResourceGroupName <yourResourceGroupName> `
-TemplateUri "https://raw.githubusercontent.com/Azure-Samples/managed-disks-powershell-getting-started/master/EncryptionAtHost/CreateVMWithDisksEncryptedAtHostWithPMK.json" `
-virtualMachineName "<yourVMName>" `
-adminPassword $password `
-vmSize "<yourVMSize>" `
-region "<yourRegion>"
```

## <a name="finding-supported-vm-sizes"></a>Búsqueda de tamaños de máquinas virtuales admitidos

No se admiten los tamaños de máquina virtual heredados. Para consultar la lista de tamaños de máquina virtual admitidos, puede hacer lo siguiente:

Llamar a la [API Resource Skus](/rest/api/compute/resourceskus/list) y comprobar que la capacidad `EncryptionAtHostSupported` está configurada en **True**.

```json
    {
        "resourceType": "virtualMachines",
        "name": "Standard_DS1_v2",
        "tier": "Standard",
        "size": "DS1_v2",
        "family": "standardDSv2Family",
        "locations": [
        "CentralUSEUAP"
        ],
        "capabilities": [
        {
            "name": "EncryptionAtHostSupported",
            "value": "True"
        }
        ]
    }
```

O bien, llamar al cmdlet de PowerShell [Get-AzComputeResourceSku](/powershell/module/az.compute/get-azcomputeresourcesku?view=azps-3.8.0).

```powershell
$vmSizes=Get-AzComputeResourceSku | where{$_.ResourceType -eq 'virtualMachines' -and $_.Locations.Contains('CentralUSEUAP')} 

foreach($vmSize in $vmSizes)
{
    foreach($capability in $vmSize.capabilities)
    {
        if($capability.Name -eq 'EncryptionAtHostSupported' -and $capability.Value -eq 'true')
        {
            $vmSize

        }

    }
}
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha creado y configurado estos recursos, puede usarlos para proteger los discos administrados. En los vínculos siguientes encontrará scripts de ejemplo, cada uno con un escenario correspondiente, que puede usar para proteger sus discos administrados.

[Ejemplos de plantillas de Azure Resource Manager](https://github.com/Azure-Samples/managed-disks-powershell-getting-started/tree/master/EncryptionAtHost)
