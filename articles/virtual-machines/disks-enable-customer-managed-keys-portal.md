---
title: 'Azure Portal: Habilitación de claves administradas por el cliente con SSE para discos administrados'
description: Habilite las claves administradas por el cliente en los discos administrados con Azure Portal.
author: roygara
ms.date: 08/24/2020
ms.topic: how-to
ms.author: rogarana
ms.service: virtual-machines
ms.subservice: disks
ms.openlocfilehash: 1b40eef0a4cae8fa68631426ff72003db43d7530
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88817213"
---
# <a name="use-the-azure-portal-to-enable-server-side-encryption-with-customer-managed-keys-for-managed-disks"></a>Uso de Azure Portal para habilitar el cifrado del lado servidor con claves administradas por el cliente para los discos administrados

Azure Disk Storage le permite administrar sus propias claves al usar el cifrado del lado servidor (SSE) para discos administrados, si así lo decide. Para obtener información conceptual sobre SSE con claves administradas por el cliente, así como otros tipos de cifrado de discos administrados, consulte la sección **Claves administradas por el cliente** de nuestro artículo sobre el cifrado de discos:

- Para Linux: [Claves administradas por el cliente](./linux/disk-encryption.md#customer-managed-keys)
- Para Windows: [Claves administradas por el cliente](./windows/disk-encryption.md#customer-managed-keys)

## <a name="restrictions"></a>Restricciones

Por ahora, las claves administradas por el cliente tienen las siguientes restricciones:

- Si esta característica está habilitada para el disco, no puede deshabilitarla.
    Si necesita encontrar una solución alternativa, debe copiar todos los datos en un disco administrado totalmente diferente que no use claves administradas por el cliente:

    - Para Linux: [Copia de un disco administrado](./linux/disks-upload-vhd-to-managed-disk-cli.md#copy-a-managed-disk)

    - Para Windows: [Copia de un disco administrado](./windows/disks-upload-vhd-to-managed-disk-powershell.md#copy-a-managed-disk)

- Solo se admiten [software y claves RSA de HSM](../key-vault/keys/about-keys.md) con un tamaño de 2048 bits, 3072 bits y 4096 bits, ninguna otra clave o tamaño.
    - Las claves de [HSM](../key-vault/keys/hsm-protected-keys.md) requieren el nivel **premium** de los almacenes Azure Key Vault.
[!INCLUDE [virtual-machines-managed-disks-customer-managed-keys-restrictions](../../includes/virtual-machines-managed-disks-customer-managed-keys-restrictions.md)]

En las secciones siguientes se explica cómo habilitar y usar las claves administradas por el cliente para discos administrados:

[!INCLUDE [virtual-machines-disks-encryption-create-key-vault-portal](../../includes/virtual-machines-disks-encryption-create-key-vault-portal.md)]

## <a name="deploy-a-vm"></a>Implementación de una máquina virtual

Ahora que ha creado y configurado el almacén de claves y el conjunto de cifrado de disco, puede implementar una VM mediante el cifrado.
El proceso de implementación de VM es similar al proceso de implementación estándar, las únicas diferencias son que debe implementar la VM en la misma región que los demás recursos y optar por usar una clave administrada por el cliente.

1. Busque **Máquinas virtuales** y seleccione **+Agregar** para crear una VM.
1. En la hoja **Básico**, seleccione la misma región que el conjunto de cifrado de disco y Azure Key Vault.
1. Rellene los demás valores en la hoja **Básico** como desee.

    ![Captura de pantalla de la experiencia de creación de la máquina virtual, con el valor de región resaltado.](media/virtual-machines-disk-encryption-portal/server-side-encryption-create-a-vm-region.png)

1. En la hoja **Discos**, seleccione **Cifrado en reposo con una clave administrada por el cliente**.
1. Seleccione el conjunto de cifrado de disco en el menú desplegable **Conjunto de cifrado de disco**.
1. Realice las selecciones restantes como desee.

    ![Captura de pantalla de la experiencia de creación de la máquina virtual, con la de hoja discos. Con la lista desplegable conjunto de cifrado de disco resaltada.](media/virtual-machines-disk-encryption-portal/server-side-encryption-create-vm-select-customer-managed-key-disk-encryption-set.png)

## <a name="enable-on-an-existing-disk"></a>Habilitación en un disco existente

> [!CAUTION]
> Para habilitar el cifrado de disco en todos los discos conectados a una VM, será necesario detener la VM.
    
1. Vaya a una VM que esté en la misma región que uno de los conjuntos de cifrado de disco.
1. Abra la VM y seleccione **Detener**.

    ![Captura de pantalla de la superposición principal de la máquina virtual de ejemplo, con el botón Detener resaltado.](media/virtual-machines-disk-encryption-portal/server-side-encryption-stop-vm-to-encrypt-disk-fix.png)

1. Una vez finalizada la detención de la VM, seleccione **Discos** y, después, seleccione el disco que desea cifrar.

    ![Captura de pantalla de la máquina virtual de ejemplo, con la hoja de discos abierta. El disco del sistema operativo está resaltado, como un disco de ejemplo para que lo seleccione.](media/virtual-machines-disk-encryption-portal/server-side-encryption-existing-disk-select.png)

1. Seleccione **Cifrado** y seleccione **Cifrado en reposo con una clave administrada por el cliente** y, a continuación, seleccione el conjunto de cifrado de disco en la lista desplegable.
1. Seleccione **Guardar**.

    ![Captura de pantalla del disco del sistema operativo de ejemplo. La hoja cifrado está abierta, el cifrado en reposo con una clave administrada por el cliente está resaltada, así como la instancia de Azure Key Vault de ejemplo. Después de efectuar esas selecciones, se selecciona el botón Guardar.](media/virtual-machines-disk-encryption-portal/server-side-encryption-encrypt-existing-disk-customer-managed-key.png)

1. Repita este proceso para todos los demás discos conectados a la VM que desea cifrar.
1. Cuando los discos terminen de cambiar a las claves administradas por el cliente, si no hay ningún otro disco conectado que quiera cifrar, puede iniciar la VM.

> [!IMPORTANT]
> Las claves administradas por el cliente dependen de identidades administradas para los recursos de Azure, una característica de Azure Active Directory (Azure AD). Al configurar claves administradas por el cliente, se asigna automáticamente una identidad administrada a los recursos en segundo plano. Si posteriormente mueve la suscripción, el grupo de recursos o el disco administrado de un directorio de Azure AD a otro, la identidad administrada asociada a los discos administrados no se transfiere al nuevo inquilino, por lo que es posible que las claves administradas por el cliente dejen de funcionar. Para obtener más información, consulte [Transferencia de una suscripción entre directorios de Azure AD](../active-directory/managed-identities-azure-resources/known-issues.md#transferring-a-subscription-between-azure-ad-directories).

## <a name="next-steps"></a>Pasos siguientes

- [Explore las plantillas de Azure Resource Manager para crear discos cifrados con claves administradas por el cliente](https://github.com/ramankumarlive/manageddiskscmkpreview)
- [¿Qué es Azure Key Vault?](../key-vault/general/overview.md)
- [Replicación de máquinas con discos habilitados para claves administradas por el cliente](../site-recovery/azure-to-azure-how-to-enable-replication-cmk-disks.md)
- [Configuración de la recuperación ante desastres de máquinas virtuales de VMware en Azure con PowerShell](../site-recovery/vmware-azure-disaster-recovery-powershell.md#replicate-vmware-vms)
- [Configuración de la recuperación ante desastres en Azure para máquinas virtuales de Hyper-V mediante PowerShell y Azure Resource Manager](../site-recovery/hyper-v-azure-powershell-resource-manager.md#step-7-enable-vm-protection)
