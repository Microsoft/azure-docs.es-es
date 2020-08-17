---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 03/23/2020
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 950fd82e14902c3a7a94fddb44d50ac372ebe119
ms.sourcegitcommit: d7bd8f23ff51244636e31240dc7e689f138c31f0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87177060"
---
[!INCLUDE [virtual-machines-disks-encryption-create-key-vault-portal](virtual-machines-disks-encryption-create-key-vault-portal.md)]

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
