---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 09/15/2018
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: f0832672cc848495f3d95d308071e0a8359ae4f1
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87375410"
---
## <a name="overview"></a>Información general
Azure Storage permite realizar instantáneas de blobs. Dichas instantáneas capturan el estado del blob en el preciso momento en el que se realicen. En este artículo, se describe un escenario para mantener copias de seguridad de discos de máquinas virtuales por medio de instantáneas. Podrá utilizar esta metodología cuando opte por no usar el servicio de recuperación y copia de seguridad de Azure y quiera crear una estrategia de copias de seguridad personalizada para los discos de sus máquinas virtuales. En el caso de las máquinas virtuales que ejecutan cargas de trabajo críticas o empresariales, se recomienda usar [Azure Backup](https://docs.microsoft.com/azure/backup/backup-azure-vms-introduction) como parte de la estrategia de copia de seguridad.  

Los discos de las máquinas virtuales de Azure se almacenan como blobs en páginas en Azure Storage. Como en este artículo se describe una estrategia de copias de seguridad para discos de máquinas virtuales, cuando hablemos de instantáneas, lo haremos en el contexto de blobs en páginas. Para obtener más información sobre las instantáneas, consulte [Creación de una instantánea a partir de un blob](https://docs.microsoft.com/rest/api/storageservices/Creating-a-Snapshot-of-a-Blob).

## <a name="what-is-a-snapshot"></a>¿Qué es una instantánea?
Una instantánea de blob es una versión de solo lectura de un blob que se ha capturado en un momento dado. Una vez se crea la instantánea, puede leerla, copiarla o eliminarla, pero no modificarla. Las instantáneas le ofrecen una oportunidad de realizar una copia de seguridad de un blob en el momento en que éste aparezca. Hasta la versión 2015-04-05 de REST podía copiar instantáneas completas. Con la versión 2015-07-08 de REST y posteriores, también puede copiar instantáneas incrementales.

## <a name="full-snapshot-copy"></a>Copia de instantáneas completas
Es posible copiar las instantáneas en otra cuenta de almacenamiento como un blob para guardar copias de seguridad del blob de base. También puede sobrescribir el blob de base con una instantánea suya, lo que equivale a restaurar el blob a una versión previa. Cuando una instantánea se copia de una cuenta de almacenamiento a otra, ocupa el mismo espacio que el blob en páginas de base. Por lo tanto, la copia de instantáneas completas de una cuenta de almacenamiento a otra es un proceso lento y consume mucho espacio en la de destino.

> [!NOTE]
> Si copia el blob de base en otro destino, sus instantáneas no se copian junto con él. Igualmente, si sobrescribe un blob de base con una copia, las instantáneas asociadas a él no se ven afectadas, sino que permanecen intactas con el nombre del blob de base.
> 
> 

### <a name="back-up-disks-using-snapshots"></a>Copia de seguridad de discos mediante instantáneas
Como una estrategia de copias de seguridad para los discos de sus máquinas virtuales, puede realizar instantáneas periódicas del disco o el blob en páginas y copiarlas en otra cuenta de almacenamiento mediante herramientas como la operación [Copy Blob](https://docs.microsoft.com/rest/api/storageservices/Copy-Blob) o [AzCopy](../articles/storage/common/storage-use-azcopy.md). Puede copiar una instantánea en un blob en páginas de destino con un nombre diferente. El blob en páginas de destino resultante es un blob en páginas en el que se puede escribir y no una instantánea. Más adelante en este artículo, se describen los pasos para realizar copias de seguridad de discos de máquinas virtuales mediante instantáneas.

### <a name="restore-disks-using-snapshots"></a>Restauración de discos mediante instantáneas
Cuando llegue el momento de restaurar el disco a una versión estable capturada previamente en una de las instantáneas de copia de seguridad, puede copiar una instantánea sobre el blob en páginas de base. Una vez que se convierta la instantánea en el blob en páginas de base, esta se conservará, pero el origen se sobrescribe con una copia de lectura y escritura. Más adelante en este artículo, se describen los pasos para restaurar una versión anterior de su disco a partir de su instantánea.

### <a name="implementing-full-snapshot-copy"></a>Implementación de una copia una instantánea completa
Puede implementar una copia de una instantánea completa siguiendo estos pasos:

* Primero, realice una instantánea del blob de base con la operación [Snapshot Blob](https://docs.microsoft.com/rest/api/storageservices/Snapshot-Blob) .
* Después, copie la instantánea en una cuenta de almacenamiento de destino mediante [Copy Blob](https://docs.microsoft.com/rest/api/storageservices/Copy-Blob).
* Repita este proceso para mantener copias de seguridad de su blob de base.

## <a name="incremental-snapshot-copy"></a>Copia de instantáneas incrementales
La nueva característica de la API [GetPageRanges](https://docs.microsoft.com/rest/api/storageservices/Get-Page-Ranges) ofrece un modo mucho mejor de realizar copias de seguridad de las instantáneas de los discos o blobs en páginas. La API devuelve la lista de cambios entre el blob de base y las instantáneas, lo que reduce la cantidad de espacio de almacenamiento utilizado en la cuenta de copia de seguridad. La API es compatible con blobs en páginas en Premium Storage, así como en su vertiente Standard. Gracias a esta API, puede crear soluciones de copia de seguridad más rápidas y eficaces para las máquinas virtuales de Azure. Esta API estará disponible con la versión 2015-07-08 de REST y posteriores.

La copia de instantáneas incrementales permite realizar copiar de una cuenta de almacenamiento a otra la diferencia entre:

* Un blob de base y su instantánea, o bien
* Dos instantáneas cualesquiera del blob de base

Siempre que se cumplan las condiciones siguientes:

* El blob se creó a partir del 1 de enero de 2016.
* El blob no se sobrescribió con [PutPage](https://docs.microsoft.com/rest/api/storageservices/Put-Page) o [Copy Blob](https://docs.microsoft.com/rest/api/storageservices/Copy-Blob) entre dos instantáneas.

>[!NOTE]
>Esta característica está disponible para blobs en páginas de Azure Premium y Standard.

Cuando se emplea una estrategia de copia de seguridad personalizada en la que se utilicen instantáneas, la copia de estas últimas de una cuenta de almacenamiento a otra puede ser lenta y consumir mucho espacio. En lugar de copiar toda la instantánea en una cuenta de almacenamiento de copia de seguridad, puede escribir la diferencia entre instantáneas consecutivas en un blob en páginas de copia de seguridad. De este modo, se reduce considerablemente el tiempo de copia y espacio necesario para almacenar las copias de seguridad.

### <a name="implementing-incremental-snapshot-copy"></a>Implementación de la copia de instantáneas incrementales
Puede implementar la copia de instantáneas incrementales siguiendo estos pasos:

* Realice una instantánea del blob de base mediante [Instantánea de blob](https://docs.microsoft.com/rest/api/storageservices/Snapshot-Blob).
* Copie la instantánea en una cuenta de almacenamiento de copia de seguridad de destino en la misma región de Azure (o en cualquier otra) mediante la opción [Copiar blob](https://docs.microsoft.com/rest/api/storageservices/Copy-Blob). Este es el blob en páginas de copia de seguridad. Realice una instantánea del blob en páginas de copia de seguridad y almacénela en la cuenta de copia de seguridad.
* Realice otra instantánea del blob de base mediante Instantánea de blob.
* Obtenga la diferencia entre la primera y la segunda instantánea del blob de base mediante [GetPageRanges](https://docs.microsoft.com/rest/api/storageservices/Get-Page-Ranges). Use el nuevo parámetro **prevsnapshot** para especificar el valor DateTime de la instantánea que quiere usar para obtener la diferencia. Cuando este parámetro está presente, la respuesta de REST solo incorpora las páginas que se cambiaron entre la instantánea de destino y la anterior, incluidas las borradas.
* Use [PutPage](https://docs.microsoft.com/rest/api/storageservices/Put-Page) para aplicar estos cambios al blob en páginas de copia de seguridad.
* Por último, realice una instantánea del blob en páginas de copia de seguridad y almacénela en la cuenta de almacenamiento de copia de seguridad.

En la siguiente sección, se describirá en mayor profundidad cómo puede mantener copias de seguridad de discos mediante la copia de instantáneas incrementales.

## <a name="scenario"></a>Escenario
En esta sección se describe un escenario en el que se utiliza una estrategia de copia de seguridad personalizada para discos de máquinas virtuales mediante instantáneas.

Considere la posibilidad de usar una VM de Azure de serie DS con un disco P30 de almacenamiento premium conectado. El disco P30 denominado *mypremiumdisk* se almacena en una cuenta de almacenamiento premium denominada *mypremiumaccount*. Se usa una cuenta de almacenamiento estándar denominada *mybackupstdaccount* para almacenar la copia de seguridad de *mypremiumdisk*. Nos gustaría mantener una instantánea de *mypremiumdisk* cada 12 horas.

Para obtener más información sobre la creación de una cuenta de almacenamiento, consulte [Creación de una cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account).

Para obtener información sobre la realización de copias de seguridad de máquinas virtuales de Azure, consulte [Plan Azure VM backups](../articles/backup/backup-azure-vms-introduction.md)(Planeación de copias de seguridad de máquinas virtuales de Azure).

## <a name="steps-to-maintain-backups-of-a-disk-using-incremental-snapshots"></a>Pasos para mantener copias de seguridad de un disco con instantáneas incrementales
En los pasos que se describen a continuación se realizan instantáneas de *mypremiumdisk* y las copias de seguridad se mantienen en *mybackupstdaccount*. La copia de seguridad es un blob en páginas estándar denominado *mybackupstdpageblob*. El blob en páginas de copia de seguridad siempre refleja el mismo estado que la última instantánea de *mypremiumdisk*.

1. Cree el blob en páginas de copia de seguridad para el disco de almacenamiento premium; para ello, realice una instantánea de *mypremiumdisk* denominada *mypremiumdisk_ss1*.
2. Copie esta instantánea en mybackupstdaccount como un blob en páginas denominado *mybackupstdpageblob*.
3. Realice una instantánea de *mybackupstdpageblob* denominada *mybackupstdpageblob_ss1* con [Instantánea de blob](https://docs.microsoft.com/rest/api/storageservices/Snapshot-Blob) y almacénela en *mybackupstdaccount*.
4. En la ventana de copia de seguridad, cree otra instantánea de *mypremiumdisk*, por ejemplo, *mypremiumdisk_ss2*, y almacénela en *mypremiumaccount*.
5. Obtenga los cambios incrementales entre las dos instantáneas, *mypremiumdisk_ss2* y *mypremiumdisk_ss1*, con [GetPageRanges](https://docs.microsoft.com/rest/api/storageservices/Get-Page-Ranges) en *mypremiumdisk_ss2* con el parámetro **prevsnapshot** establecido en la marca de tiempo de *mypremiumdisk_ss1*. Escriba estos cambios incrementales en el blob en páginas de copia de seguridad *mybackupstdpageblob* de *mybackupstdaccount*. Si hay intervalos eliminados en los cambios incrementales, deben borrarse desde el blob en páginas de copia de seguridad. Use [PutPage](https://docs.microsoft.com/rest/api/storageservices/Put-Page) para escribir los cambios incrementales en el blob en páginas de copia de seguridad.
6. Realice una instantánea del blob en páginas de copia de seguridad *mybackupstdpageblob* denominada *mybackupstdpageblob_ss2*. Elimine la instantánea anterior *mypremiumdisk_ss1* de la cuenta de almacenamiento premium.
7. Repita los pasos del 4 al 6 en cada ventana de copia de seguridad. De esta forma, puede mantener copias de seguridad de *mypremiumdisk* en una cuenta de almacenamiento estándar.

![Realización de copias de seguridad de discos usando instantáneas incrementales](../articles/virtual-machines/windows/media/incremental-snapshots/storage-incremental-snapshots-1.png)

## <a name="steps-to-restore-a-disk-from-snapshots"></a>Pasos para restaurar un disco de las instantáneas
En los pasos siguientes se describe cómo restaurar el disco premium, *mypremiumdisk*, a una instantánea anterior desde la cuenta de almacenamiento de copia de seguridad *mybackupstdaccount*.

1. Identifique el momento al que desea restaurar el disco premium. Supongamos que es la instantánea *mybackupstdpageblob_ss2*, que se almacena en la cuenta de almacenamiento de copia de seguridad *mybackupstdaccount*.
2. En mybackupstdaccount, promueva la instantánea *mybackupstdpageblob_ss2* como el nuevo blob en páginas de base de copia de seguridad *mybackupstdpageblobrestored*.
3. Realice una instantánea de este blob en páginas de copia de seguridad restaurado denominada *mybackupstdpageblobrestored_ss1*.
4. Copie el blob en páginas restaurado *mybackupstdpageblobrestored* desde *mybackupstdaccount* en *mypremiumaccount* como el nuevo disco premium *mypremiumdiskrestored*.
5. Realice una instantánea de *mypremiumdiskrestored* denominada *mypremiumdiskrestored_ss1* para realizar futuras copias de seguridad incrementales.
6. Seleccione la máquina virtual de serie DS en el disco restaurado *mypremiumdiskrestored* y desasocie el *mypremiumdisk* anterior de la máquina virtual.
7. Inicie el proceso de copia de seguridad que se describe en la sección anterior para el disco restaurado *mypremiumdiskrestored* con *mybackupstdpageblobrestored* como el blob en páginas de copia de seguridad.

![Restauración de discos usando instantáneas](../articles/virtual-machines/windows/media/incremental-snapshots/storage-incremental-snapshots-2.png)

## <a name="next-steps"></a>Pasos siguientes
Obtenga información sobre cómo crear instantáneas de un blob y planear la infraestructura de copia de seguridad de máquinas virtuales mediante los vínculos siguientes.

* [Crear una instantánea de un blob](https://docs.microsoft.com/rest/api/storageservices/Creating-a-Snapshot-of-a-Blob)
* [Planeación de la infraestructura de copia de seguridad de máquinas virtuales](../articles/backup/backup-azure-vms-introduction.md)

