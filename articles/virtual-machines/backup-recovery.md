---
title: Introducción a las opciones de copia de seguridad para máquinas virtuales Linux
description: Información general de opciones de copia de seguridad para máquinas virtuales Linux en Azure.
author: cynthn
ms.service: virtual-machines
ms.topic: conceptual
ms.date: 8/03/2020
ms.author: cynthn
ms.openlocfilehash: bb1abb5b94653893e131c09af6de1625b0bc7998
ms.sourcegitcommit: 1b2d1755b2bf85f97b27e8fbec2ffc2fcd345120
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87554741"
---
# <a name="backup-and-restore-options-for-linux-virtual-machines-in-azure"></a>Opciones de copia de seguridad y restauración para máquinas virtuales Linux en Azure

Para proteger sus datos realice copias de seguridad a intervalos regulares. Hay varias opciones de copia de seguridad disponibles para máquinas virtuales, según el caso de uso.

## <a name="azure-backup"></a>Azure Backup

Para realizar copias de seguridad de máquinas virtuales de Azure que ejecutan cargas de trabajo de producción, use Azure Backup. Azure Backup admite las copias de seguridad coherentes con la aplicación para máquinas virtuales Windows y Linux. Azure Backup crea puntos de recuperación que se almacenan en almacenes de recuperación con redundancia geográfica. Cuando se realiza una restauración desde un punto de recuperación, se puede restaurar toda una máquina virtual o solo determinados archivos. 

Para obtener una introducción simple y práctica sobre Azure Backup para máquinas virtuales de Azure, consulte el tutorial "Copia de seguridad de máquinas virtuales de Azure" para [Linux](./linux/tutorial-backup-vms.md) o [Windows](./windows/tutorial-backup-vms.md).

Para más información sobre cómo funciona Azure Backup, consulte [Planeamiento de la infraestructura de copia de seguridad de máquinas virtuales en Azure](../backup/backup-azure-vms-introduction.md)


## <a name="azure-site-recovery"></a>Azure Site Recovery

Azure Site Recovery permite proteger las máquinas virtuales en un escenario grave de desastre en el que toda una región experimenta una interrupción debido a un desastre natural importante o a una interrupción del servicio generalizada. Puede configurar Azure Site Recovery para las VM de modo que pueda recuperar la aplicación con un solo clic en cuestión de minutos. Puede realizar replicaciones en la región de Azure que elija, no solo en regiones emparejadas. 

Puede ejecutar maniobras de recuperación ante desastres con conmutaciones por error de prueba a petición, sin que las cargas de trabajo de producción o la replicación en curso se vean afectadas. Cree planes de recuperación para orquestar la conmutación por error y la conmutación por recuperación de toda la aplicación que se ejecuta en varias máquinas virtuales. La característica de los planes de recuperación está integrada con runbooks de Azure Automation.

Puede empezar por [replicar las máquinas virtuales](https://aka.ms/a2a-getting-started). 

## <a name="managed-snapshots"></a>Instantáneas administradas 

En entornos de desarrollo y prueba, las instantáneas proporcionan una opción rápida y sencilla para realizar copias de seguridad de máquinas virtuales que usan Managed Disks. Una instantánea administrada es una copia completa de solo lectura de un disco administrado. Las instantáneas existen independientemente del disco de origen y se pueden usar para recompilar una máquina virtual. Se facturan según la porción usada del disco. Por ejemplo, si crea una instantánea de un disco administrado con capacidad aprovisionada de 64 GB y el tamaño de datos usado real es de 10 GB, solo se le cobrará por el tamaño de datos usado de 10 GB.  

Para más información sobre la creación de instantáneas, consulte:

* [Creación de una copia del disco duro virtual que se almacene como un disco administrado mediante instantáneas en Windows](./windows/snapshot-copy-managed-disk.md)
* [Creación de una copia del disco duro virtual que se almacene como un disco administrado mediante instantáneas en Linux](./linux/snapshot-copy-managed-disk.md)



## <a name="next-steps"></a>Pasos siguientes
Puede probar Azure Backup siguiendo el tutorial "Copia de seguridad de máquinas virtuales Windows" para [Linux](./linux/tutorial-backup-vms.md) o [Windows](./windows/tutorial-backup-vms.md).
