---
title: Habilitación del cifrado doble en reposo con discos administrados - Azure Portal
description: Habilite el cifrado doble en reposo para los datos de discos administrados mediante Azure Portal.
author: roygara
ms.date: 08/24/2020
ms.topic: how-to
ms.author: rogarana
ms.service: virtual-machines-linux
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: bf2531536796965f145a9ac3e6a23cbb6634852a
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88817236"
---
# <a name="use-the-azure-portal-to-enable-double-encryption-at-rest-for-managed-disks"></a>Uso de Azure Portal para habilitar el cifrado doble en reposo para discos administrados

Azure Disk Storage admite el cifrado doble en reposo para los discos administrados. Para obtener información conceptual sobre el cifrado doble en reposo, así como otros tipos de cifrado de discos administrados, consulte la sección **Doble cifrado en reposo** de nuestro artículo sobre el cifrado de discos:

- Para Linux: [Cifrado doble en reposo](./linux/disk-encryption.md#double-encryption-at-rest)
- Para Windows: [Cifrado doble en reposo](./windows/disk-encryption.md#double-encryption-at-rest)

## <a name="supported-regions"></a>Regiones admitidas

[!INCLUDE [virtual-machines-disks-double-encryption-at-rest-regions](../../includes/virtual-machines-disks-double-encryption-at-rest-regions.md)]

## <a name="getting-started"></a>Introducción

1. Inicie sesión en [Azure Portal](https://aka.ms/diskencryptionupdates).

    > [!IMPORTANT]
    > Debe usar el [vínculo proporcionado](https://aka.ms/diskencryptionupdates) para tener acceso a Azure Portal. El cifrado doble en reposo no está visible actualmente en Azure Portal público sin usar el vínculo.

1. Busque y seleccione **Conjuntos de cifrado de disco**.

    :::image type="content" source="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-disk-encryption-sets-search.png" alt-text="Captura de pantalla de Azure Portal principal, Conjuntos de cifrado de disco está resaltado en la barra de búsqueda.":::

1. Seleccione **+Agregar**.

    :::image type="content" source="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-add-disk-encryption-set.png" alt-text="Captura de pantalla de la hoja Conjunto de cifrado de disco, + Agregar está resaltado.":::

1. Seleccione alguna de las regiones admitidas.
1. Para **Tipo de cifrado**, seleccione **Cifrado doble con claves administradas por el cliente y la plataforma**.

    > [!NOTE]
    > Una vez que cree un conjunto de cifrado de disco con un tipo de cifrado en particular, no se puede cambiar. Si desea usar otro tipo de cifrado, debe crear un nuevo conjunto de cifrado de disco.

1. Rellene la información restante.

    :::image type="content" source="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-create-disk-encryption-set-blade.png" alt-text="Captura de pantalla de la hoja de creación de conjunto de cifrado de disco, la región y el cifrado doble con claves administradas por el cliente y la plataforma están resaltados.":::

1. Seleccione una instancia de Azure Key Vault y una clave, o cree una nueva si es necesario.

    > [!NOTE]
    > Si crea una instancia de Key Vault, debe habilitar la eliminación temporal y la protección de purgas. Esta configuración es obligatoria al usar una instancia de Key Vault para cifrar discos administrados, y le protege contra la pérdida de datos debido a la eliminación accidental.

    :::image type="content" source="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-select-key-vault.png" alt-text="Captura de pantalla de la hoja de creación de Key Vault.":::

1. Seleccione **Crear**.
1. Navegue hasta el conjunto de cifrado de disco que creó y seleccione el error que se muestra. Esto configurará el conjunto de cifrado de disco para que funcione.

    :::image type="content" source="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-disk-set-error.png" alt-text="Captura de pantalla del error del conjunto de cifrado de disco, el texto del error muestra: Para asociar un disco, una imagen o una instantánea a este conjunto de cifrado de disco, debe conceder permisos al almacén de claves.":::

    Una notificación debería aparecer y completarse correctamente. Esto le permitirá usar el conjunto de cifrado de disco con el almacén de claves.
    
    ![Captura de pantalla de la asignación de roles y permisos correctos para el almacén de claves.](media/virtual-machines-disks-double-encryption-at-rest-portal/disk-encryption-notification-success.png)

1. Vaya al disco.
1. Seleccione **Cifrado**.
1. Para **Tipo de cifrado**, seleccione **Cifrado doble con claves administradas por el cliente y la plataforma**.
1. Seleccione el conjunto de cifrado de disco.
1. Seleccione **Guardar**.

    :::image type="content" source="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-enable-disk-blade.png" alt-text="Captura de pantalla de la hoja cifrado del disco administrado, el tipo de cifrado mencionado está resaltado.":::

Ha habilitado el cifrado doble en reposo en el disco administrado.


## <a name="next-steps"></a>Pasos siguientes

- [Azure PowerShell: habilitación de claves administradas por el cliente con el cifrado del lado servidor para discos administrados](./windows/disks-enable-customer-managed-keys-powershell.md)
- [Ejemplos de plantillas de Azure Resource Manager](https://github.com/Azure-Samples/managed-disks-powershell-getting-started/tree/master/DoubleEncryption)
- [Habilitación de claves administradas por el cliente con el cifrado del lado servidor: ejemplos](./linux/disks-enable-customer-managed-keys-cli.md#examples)