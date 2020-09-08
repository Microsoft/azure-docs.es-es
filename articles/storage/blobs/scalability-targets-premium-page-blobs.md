---
title: Objetivos de escalabilidad de las cuentas de almacenamiento de blob en páginas Premium
titleSuffix: Azure Storage
description: Una cuenta de almacenamiento de blobs en páginas de rendimiento prémium está optimizada para las operaciones de lectura y escritura. Este tipo de cuenta de almacenamiento hace una copia de seguridad de un disco no administrado para una máquina virtual de Azure.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 12/18/2019
ms.author: tamram
ms.subservice: common
ms.openlocfilehash: 2c238bf3911283db5d09fdd5679d784c5e5401f8
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88654810"
---
# <a name="scalability-and-performance-targets-for-premium-page-blob-storage-accounts"></a>Objetivos de escalabilidad y rendimiento de las cuentas de almacenamiento de blob en páginas Premium

[!INCLUDE [storage-scalability-intro-include](../../../includes/storage-scalability-intro-include.md)]

## <a name="scale-targets-for-premium-page-blob-accounts"></a>Objetivos de escalado para cuentas de blob en páginas Premium

Una cuenta de almacenamiento de blobs en páginas de rendimiento Premium está optimizada para las operaciones de lectura y escritura. Este tipo de cuenta de almacenamiento hace una copia de seguridad de un disco no administrado para una máquina virtual de Azure.

> [!NOTE]
> Microsoft recomienda usar discos administrados con máquinas virtuales de Azure (VM), de ser posible. Para obtener más información sobre los discos administrados, consulte [Introducción a Azure Disk Storage para máquinas virtuales](../../virtual-machines/managed-disks-overview.md).

Las cuentas de almacenamiento de blobs en páginas Premium tienen los siguientes objetivos de escalabilidad:

| Capacidad total de la cuenta                            | Ancho de banda total de una cuenta de almacenamiento con redundancia local                     |
| ------------------------------------------------- | --------------------------------------------------------------------------- |
| Capacidad de disco: 4 TB (disco individual)/35 TB (total acumulado de todos los discos) <br>Capacidad de instantánea: 10 TB | Hasta 50 gigabits por segundo de entrada<sup>1</sup> y salida<sup>2</sup> |

<sup>1</sup> Todos los datos (solicitudes) que se envían a una cuenta de almacenamiento

<sup>1</sup> Todos los datos (respuestas) que se reciben desde una cuenta de almacenamiento

Una cuenta de blob en páginas Premium es una cuenta de uso general configurada para tener rendimiento Premium. Se recomiendan las cuentas de almacenamiento de uso general v2.

Si usa cuentas de almacenamiento de blobs en páginas Premium para los discos no administrados y la aplicación supera los objetivos de escalabilidad de una cuenta de almacenamiento individual, Microsoft recomienda migrar a discos administrados. Para obtener más información sobre los discos administrados, consulte [Introducción a Azure Disk Storage para máquinas virtuales](../../virtual-machines/managed-disks-overview.md).

Si no puede migrar a los discos administrados, compile la aplicación para que use varias cuentas de almacenamiento y crear particiones de los datos en dichas cuentas. Por ejemplo, si desea asociar discos de 51 TB entre varias VM, distribúyalos entre dos cuentas de almacenamiento. 35 TB es el límite de una cuenta de Premium Storage única. Asegúrese de que una sola cuenta de almacenamiento de rendimiento prémium nunca tenga más de 35 TB de discos aprovisionados.

## <a name="see-also"></a>Consulte también

- [Destinos de escalabilidad y rendimiento para cuentas de almacenamiento estándar](../common/scalability-targets-standard-account.md)
- [Objetivos de escalabilidad de las cuentas de almacenamiento de blob en bloques Premium](../blobs/scalability-targets-premium-block-blobs.md)
- [Límites y cuotas de suscripción de Azure](../../azure-resource-manager/management/azure-subscription-service-limits.md)
