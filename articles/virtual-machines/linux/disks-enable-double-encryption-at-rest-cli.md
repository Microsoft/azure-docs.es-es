---
title: 'Habilitación del cifrado doble en reposo para discos administrados: CLI de Azure'
description: Habilite el cifrado doble en reposo para los datos de discos administrados mediante la CLI de Azure.
author: roygara
ms.date: 08/24/2020
ms.topic: how-to
ms.author: rogarana
ms.service: virtual-machines-linux
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: 76e5ab49e354faac085be533fed1ff16c0315a5e
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88816837"
---
# <a name="use-the-azure-cli-to-enable-double-encryption-at-rest-for-managed-disks"></a>Uso de la CLI de Azure para habilitar el cifrado doble en reposo para discos administrados

Azure Disk Storage admite el cifrado doble en reposo para los discos administrados. Para obtener información conceptual sobre el cifrado doble en reposo, así como otros tipos de cifrado de discos administrados, consulte la sección [Doble cifrado en reposo](disk-encryption.md#double-encryption-at-rest) de nuestro artículo sobre el cifrado de discos.

## <a name="supported-regions"></a>Regiones admitidas

[!INCLUDE [virtual-machines-disks-double-encryption-at-rest-regions](../../../includes/virtual-machines-disks-double-encryption-at-rest-regions.md)]

## <a name="prerequisites"></a>Requisitos previos

Instale la última versión de la [CLI de Azure](/cli/azure/install-az-cli2) e inicie sesión en una cuenta de Azure con [az login](/cli/azure/reference-index).

## <a name="getting-started"></a>Introducción

1. Cree una instancia de Azure Key Vault y la clave de cifrado.

    Al crear la instancia de Key Vault, debe habilitar la eliminación temporal y la protección de purgas. La eliminación temporal garantiza que la instancia de Key Vault conserva una clave eliminada durante un período de retención determinado (valor predeterminado de 90 días). La protección de purgas garantiza que una clave eliminada no se puede eliminar permanentemente hasta que transcurra el período de retención. Esta configuración le protege contra la pérdida de datos debido a la eliminación accidental. Estos valores son obligatorios cuando se usa una instancia de Key Vault para cifrar discos administrados.

    
    ```azurecli
    subscriptionId=yourSubscriptionID
    rgName=yourResourceGroupName
    location=westcentralus
    keyVaultName=yourKeyVaultName
    keyName=yourKeyName
    diskEncryptionSetName=yourDiskEncryptionSetName
    diskName=yourDiskName

    az account set --subscription $subscriptionId

    az keyvault create -n $keyVaultName -g $rgName -l $location --enable-purge-protection true --enable-soft-delete true

    az keyvault key create --vault-name $keyVaultName -n $keyName --protection software
    ```

1.    Cree un elemento DiskEncryptionSet con encryptionType establecido en EncryptionAtRestWithPlatformAndCustomerKeys. Use la versión de API **2020-05-01** en la plantilla de Azure Resource Manager (ARM). 
    
        ```azurecli
        az group deployment create -g $rgName \
       --template-uri "https://raw.githubusercontent.com/Azure-Samples/managed-disks-powershell-getting-started/master/DoubleEncryption/CreateDiskEncryptionSetForDoubleEncryption.json" \
        --parameters "diskEncryptionSetName=$diskEncryptionSetName" "encryptionType=EncryptionAtRestWithPlatformAndCustomerKeys" "keyVaultId=$keyVaultId" "keyVaultKeyUrl=$keyVaultKeyUrl" "region=$location"
        ```

1.    Conceda al recurso DiskEncryptionSet acceso al almacén de claves. 

        > [!NOTE]
        > Azure puede tardar unos minutos en crear la identidad del elemento DiskEncryptionSet en Azure Active Directory. Si recibe un error similar a "No se encuentra el objeto en Active Directory" al ejecutar el siguiente comando, espere unos minutos y vuelva a intentarlo.

        ```azurecli
        desIdentity=$(az disk-encryption-set show -n $diskEncryptionSetName -g $rgName --query [identity.principalId] -o tsv)

        az keyvault set-policy -n $keyVaultName -g $rgName --object-id $desIdentity --key-permissions wrapkey unwrapkey get
        ```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha creado y configurado estos recursos, puede usarlos para proteger los discos administrados. En los vínculos siguientes encontrará scripts de ejemplo, cada uno con un escenario correspondiente, que puede usar para proteger sus discos administrados.

- [Ejemplos de plantillas de Azure Resource Manager](https://github.com/Azure-Samples/managed-disks-powershell-getting-started/tree/master/DoubleEncryption)
- [Habilitación de claves administradas por el cliente con el cifrado del lado servidor: Ejemplos](disks-enable-customer-managed-keys-cli.md#examples)