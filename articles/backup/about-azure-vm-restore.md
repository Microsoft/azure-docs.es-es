---
title: Acerca del proceso de restauración de máquinas virtuales de Azure
description: Obtenga información sobre cómo el servicio Azure Backup restaura máquinas virtuales de Azure
ms.topic: conceptual
ms.date: 05/20/2020
ms.openlocfilehash: 5458d02e241860a98d1ab5f64df141132813f8dd
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89011963"
---
# <a name="about-azure-vm-restore"></a>Acerca de la restauración de máquinas virtuales de Azure

En este artículo se describe cómo el [servicio Azure Backup](./backup-overview.md) restaura máquinas virtuales (VM) de Azure. Hay una serie de opciones de restauración. Trataremos los distintos escenarios que admiten.

## <a name="concepts"></a>Conceptos

- **Punto de recuperación** (también conocido como **punto de restauración**): Un punto de recuperación es una copia de los datos originales de los que se hará una copia de seguridad.

- **Nivel (instantánea frente a almacén)** :  La copia de seguridad de máquina virtual de Azure se produce en dos fases:

  - En la fase 1, la instantánea tomada se almacena junto con el disco. Esto se denomina **nivel de instantánea**. Las restauraciones de nivel de instantánea son más rápidas (que la restauración desde el almacén) porque eliminan el tiempo de espera para que las instantáneas se copien en el almacén antes de desencadenar la restauración. Por lo tanto, la restauración desde el nivel de instantánea también se conoce como [restauración instantánea](./backup-instant-restore-capability.md).
  - En la fase 2, la instantánea se transfiere y se guarda en el almacén administrado por el servicio Azure Backup. Esto se denomina **nivel de almacén**.

- **Recuperación de ubicación original (OLR)** : Recuperación realizada desde el punto de restauración a la máquina virtual de Azure de origen desde la que se realizaron las copias de seguridad, reemplazándolo por el estado almacenado en el punto de recuperación. Esto reemplaza el disco del sistema operativo y los discos de datos de la máquina virtual de origen.

- **Recuperación de ubicación alternativa (ALR)** : Recuperación realizada desde el punto de recuperación a un servidor distinto del original, de donde proceden las copias de seguridad.

- **Restauración en el nivel de elemento (ILR):** Restauración de archivos o carpetas individuales del interior de la máquina virtual desde el punto de recuperación

- **Disponibilidad (tipos de replicación)** : Azure Backup ofrece dos tipos de replicación para mantener la alta disponibilidad de los datos o del almacenamiento:
  - El [almacenamiento con redundancia local (LRS)](../storage/common/storage-redundancy.md) replica los datos tres veces (crea tres copias de los datos) en una unidad de escalado de almacenamiento de un centro de datos. Todas las copias de los datos se encuentran en la misma región. LRS es una opción de bajo costo para proteger los datos contra errores de hardware local.
  - El [almacenamiento con redundancia geográfica (GRS)](../storage/common/storage-redundancy.md) es la opción de replicación predeterminada y recomendada. GRS replica los datos en una región secundaria (a cientos de kilómetros de la ubicación principal de los datos de origen). GRS cuesta más que LRS, pero proporciona un mayor nivel de durabilidad de los datos, aunque se produzca una interrupción regional.

- **Restauración entre regiones (CRR)** : Restauración entre regiones (CRR), una de las [opciones de restauración](./backup-azure-arm-restore-vms.md#restore-options), le permite restaurar máquinas virtuales de Azure en una región secundaria, que es una [región emparejada de Azure](../best-practices-availability-paired-regions.md#what-are-paired-regions).

## <a name="restore-scenarios"></a>Escenarios de restauración

![Escenarios de restauración ](./media/about-azure-vm-restore/recovery-scenarios.png)

| **Escenario**                                                 | **Qué se debe hacer**                                             | **Cuándo se deben usar**                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [Restauración para crear una máquina virtual](./backup-azure-arm-restore-vms.md) | Restaura la máquina virtual completa a OLR (si la máquina virtual de origen sigue existiendo) o ALR | <li> Si la máquina virtual de origen se pierde o está dañada, puede restaurarla por completo  <li> Puede crear una copia de la VM  <li> Puede realizar una restauración detallada para la auditoría o el cumplimiento  <li> Esta opción no funcionará para las máquinas virtuales de Azure creadas a partir de imágenes de Marketplace (es decir, si no están disponibles porque la licencia ha expirado). |
| [Restauración de discos de la máquina virtual](./backup-azure-arm-restore-vms.md#restore-disks) | Restauración de los discos conectados a la VM                             |  Todos los discos: Esta opción crea la plantilla y restaura el disco. Puede editar esta plantilla con configuraciones especiales (por ejemplo, conjuntos de disponibilidad) para cumplir sus requisitos y, a continuación, usar la plantilla y restaurar el disco para volver a crear la máquina virtual. |
| [Restauración de archivos específicos de la VM](./backup-azure-restore-files-from-vm.md) | Elija punto de restauración, navegue, seleccione archivos y restáurelos en el mismo sistema operativo (o compatible) que la VM de la que se realizó la copia de seguridad. |  Si sabe qué archivos específicos desea restaurar, use esta opción en lugar de restaurar toda la máquina virtual. |
| [Restauración de una máquina virtual cifrada](./backup-azure-vms-encryption.md) | En el portal, restaure los discos y, después, use PowerShell para crear la máquina virtual | <li> [Máquina virtual cifrada con Azure Active Directory (AAD)](../virtual-machines/windows/disk-encryption-windows-aad.md)  <li> [Máquina virtual cifrada sin AAD](../virtual-machines/windows/disk-encryption-windows.md) <li> [Máquina virtual cifrada *con  AAD* migrado a *sin AAD*](../virtual-machines/windows/disk-encryption-faq.md#can-i-migrate-vms-that-were-encrypted-with-an-azure-ad-app-to-encryption-without-an-azure-ad-app) |
| [Restauración entre regiones](./backup-azure-arm-restore-vms.md#cross-region-restore) | Creación de una máquina virtual o restauración de discos en una región secundaria (región emparejada de Azure) | <li> **Interrupción completa**:  Con la característica de restauración entre regiones, no hay tiempo de espera para recuperar los datos de la región secundaria. Puede iniciar restauraciones en la región secundaria incluso antes de que Azure declare una interrupción. <li> **Interrupción parcial**: Puede producirse un tiempo de inactividad en los clústeres de almacenamiento específicos en los que Azure Backup almacene los datos de copia de seguridad o incluso en la red, la conexión de Azure Backup y los clústeres de almacenamiento asociados a los datos de los que se hace copia de seguridad. Con la restauración entre regiones, puede realizar una restauración en la región secundaria mediante una réplica de datos de copia de seguridad en la región secundaria. <li> **Sin interrupción**: Puede realizar simulacros de continuidad empresarial y recuperación ante desastres (BCDR) para auditorías o conseguir el cumplimiento con los datos de la región secundaria. Esto le permite realizar una restauración de los datos de copia de seguridad en la región secundaria aunque no haya una interrupción completa o parcial en la región primaria para los simulacros de continuidad empresarial y recuperación ante desastres.  |

## <a name="next-steps"></a>Pasos siguientes

- [Preguntas frecuentes sobre la restauración de la máquina virtual](./backup-azure-vm-backup-faq.md#restore)
- [Métodos de restauración admitidos](./backup-support-matrix-iaas.md#supported-restore-methods)
- [Solución de problemas de restauración](./backup-azure-vms-troubleshoot.md#restore)
