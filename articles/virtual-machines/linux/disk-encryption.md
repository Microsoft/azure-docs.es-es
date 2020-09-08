---
title: Cifrado del lado servidor de Azure Managed Disks - CLI de Azure
description: Azure Storage protege los datos mediante su cifrado en reposo antes de guardarlos en los clústeres de Storage. Puede usar las claves administradas por el cliente para administrar el cifrado con sus propias claves, o bien puede utilizar las claves administradas por Microsoft para el cifrado de los discos administrados.
author: roygara
ms.date: 07/10/2020
ms.topic: conceptual
ms.author: rogarana
ms.service: virtual-machines-linux
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: e541fd90f0cb9c8f0e2763f16541824c5a7b3000
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88816905"
---
# <a name="server-side-encryption-of-azure-disk-storage"></a>Cifrado del lado servidor de Azure Disk Storage

El cifrado del lado servidor (SSE) protege los datos y le ayuda a cumplir los compromisos de cumplimiento y seguridad de la organización. SSE cifra automáticamente los datos almacenados en discos administrados de Azure (discos de datos y SO) en reposo de forma predeterminada cuando los conserva en la nube. 

Los datos de los discos administrados de Azure se cifran de forma transparente mediante [cifrado AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) de 256 bits, uno de los cifrados de bloques más sólidos que hay disponibles, y son compatibles con FIPS 140-2. Para más información sobre de los módulos criptográficos subyacentes en los discos administrados de Azure, consulte [Cryptography API: Next Generation](/windows/desktop/seccng/cng-portal)

El cifrado del lado servidor no afecta al rendimiento de los discos administrados y no implica ningún costo adicional. 

## <a name="about-encryption-key-management"></a>Información sobre la administración de claves de cifrado

Puede confiar en las claves administradas por la plataforma para el cifrado del disco administrado o puede administrar el cifrado con sus propias claves. Si opta por administrar el cifrado con sus propias claves, puede especificar una *clave administrada por el cliente* que se usará para cifrar y descifrar todos los datos de discos administrados. 

En las secciones siguientes se describe cada una de las opciones de administración de claves con mayor detalle.

### <a name="platform-managed-keys"></a>Claves administradas por la plataforma

De forma predeterminada, los discos administrados usan claves de cifrado administradas por la plataforma. Todos los discos administrados, instantáneas e imágenes, así como los datos nuevos que se escriban en discos administrados existentes, se cifran automáticamente en reposo con claves administradas por la plataforma.

### <a name="customer-managed-keys"></a>Claves administradas por el cliente

[!INCLUDE [virtual-machines-managed-disks-description-customer-managed-keys](../../../includes/virtual-machines-managed-disks-description-customer-managed-keys.md)]

#### <a name="restrictions"></a>Restricciones

Por ahora, las claves administradas por el cliente tienen las siguientes restricciones:

- Si esta característica está habilitada para el disco, no puede deshabilitarla.
    Si necesita encontrar una solución alternativa, debe [copiar todos los datos](disks-upload-vhd-to-managed-disk-cli.md#copy-a-managed-disk) en un disco administrado totalmente diferente que no use claves administradas por el cliente.
[!INCLUDE [virtual-machines-managed-disks-customer-managed-keys-restrictions](../../../includes/virtual-machines-managed-disks-customer-managed-keys-restrictions.md)]

> [!IMPORTANT]
> Las claves administradas por el cliente dependen de identidades administradas para los recursos de Azure, una característica de Azure Active Directory (Azure AD). Al configurar claves administradas por el cliente, se asigna automáticamente una identidad administrada a los recursos en segundo plano. Si posteriormente mueve la suscripción, el grupo de recursos o el disco administrado de un directorio de Azure AD a otro, la identidad administrada asociada a los discos administrados no se transfiere al nuevo inquilino, por lo que es posible que las claves administradas por el cliente dejen de funcionar. Para obtener más información, consulte [Transferencia de una suscripción entre directorios de Azure AD](../../active-directory/managed-identities-azure-resources/known-issues.md#transferring-a-subscription-between-azure-ad-directories).

## <a name="encryption-at-host---end-to-end-encryption-for-your-vm-data"></a>Cifrado en el host: cifrado de un extremo a otro de los datos de la máquina virtual

Al habilitar el cifrado en el host, dicho cifrado se inicia en el host de VM mismo, el servidor de Azure al que está asignada la máquina virtual. Los datos de las caché de disco temporal y de disco de datos/SO se almacenan en el host de VM. Después de habilitar el cifrado en el host, todos estos datos se cifran en reposo y los flujos se cifran en el servicio de almacenamiento, donde se conservan. Esencialmente, el cifrado en el host cifra los datos de un extremo a otro. El cifrado en el host no usa la CPU de la VM y no afecta el rendimiento de la VM. 

Los discos temporales se cifran en reposo con claves administradas por la plataforma al habilitar el cifrado de un extremo a otro. Las cachés del disco de datos y SO se cifran en reposo con claves administradas por el cliente o por la plataforma, según el tipo de cifrado de disco seleccionado. Por ejemplo, si un disco se cifra con claves administradas por el cliente, la caché del disco se cifra con claves administradas por el cliente, y si un disco se cifra con claves administradas por la plataforma, la caché del disco se cifra con claves administradas por la plataforma.

### <a name="restrictions"></a>Restricciones

[!INCLUDE [virtual-machines-disks-encryption-at-host-restrictions](../../../includes/virtual-machines-disks-encryption-at-host-restrictions.md)]

#### <a name="supported-regions"></a>Regiones admitidas

[!INCLUDE [virtual-machines-disks-encryption-at-host-regions](../../../includes/virtual-machines-disks-encryption-at-host-regions.md)]

#### <a name="supported-vm-sizes"></a>Tamaños de máquinas virtuales que se admiten

[!INCLUDE [virtual-machines-disks-encryption-at-host-suported-sizes](../../../includes/virtual-machines-disks-encryption-at-host-suported-sizes.md)]

## <a name="double-encryption-at-rest"></a>Cifrado doble en reposo

Los clientes confidenciales de alto nivel de seguridad que están preocupados por el riesgo asociado a cualquier algoritmo de cifrado, implementación o clave en peligro concretos pueden optar por una capa adicional de cifrado con un algoritmo o modo de cifrado diferente en el nivel de infraestructura mediante claves de cifrado administradas por la plataforma. Esta capa nueva se puede aplicar a discos, instantáneas e imágenes, los cuales se cifrarán en reposo con el cifrado doble.

### <a name="supported-regions"></a>Regiones admitidas

[!INCLUDE [virtual-machines-disks-double-encryption-at-rest-regions](../../../includes/virtual-machines-disks-double-encryption-at-rest-regions.md)]

## <a name="server-side-encryption-versus-azure-disk-encryption"></a>Cifrado del lado servidor frente a Azure Disk Encryption

[Azure Disk Encryption](../../security/fundamentals/azure-disk-encryption-vms-vmss.md) aprovecha la característica [DM-Crypt](https://en.wikipedia.org/wiki/Dm-crypt) de Linux para cifrar los discos administrados con claves administradas por el cliente dentro de la máquina virtual invitada.  El cifrado del lado servidor con claves administradas por el cliente mejora en ADE al permitir el uso de cualquier tipo de sistema operativo y de imágenes para las máquinas virtuales mediante el cifrado de datos en el servicio Storage.

## <a name="next-steps"></a>Pasos siguientes

- Habilite el cifrado de un extremo a otro mediante el cifrado en el host con la [CLI](disks-enable-host-based-encryption-cli.md) o [Azure Portal](../disks-enable-host-based-encryption-portal.md).
- Habilite el cifrado doble en reposo para discos administrados con la [CLI](disks-enable-double-encryption-at-rest-cli.md) o [Azure Portal](../disks-enable-double-encryption-at-rest-portal.md).
- Habilite las claves administradas por el cliente para discos administrados con la [CLI](disks-enable-customer-managed-keys-cli.md) o [Azure Portal](../disks-enable-customer-managed-keys-portal.md).
- [¿Qué es Azure Key Vault?](../../key-vault/general/overview.md)
