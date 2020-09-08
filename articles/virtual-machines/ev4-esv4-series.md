---
title: 'Serie Ev4 y Esv4: Azure Virtual Machines'
description: Especificaciones para las VM de las series Ev4 y Esv4.
author: brbell
ms.author: brbell
ms.reviewer: cynthn
ms.custom: mimckitt
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 6/8/2020
ms.openlocfilehash: b8a6e1013c131778bacb11bd885b9280fb6c6077
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89182180"
---
# <a name="ev4-and-esv4-series"></a>Series Ev4 y Esv4

Las series Ev4 y Esv4 se ejecuta en los procesadores Intel&reg; Xeon&reg; Platinum 8272CL (lago en cascada) en una configuración con Hyper-Threading. Son ideales para las diversas aplicaciones empresariales de uso intensivo de memoria y hasta 504 GiB de RAM. Cuenta con una velocidad sostenida de reloj en todos los núcleos de 3,4 GHz.

> [!NOTE]
> Para consultar las preguntas más frecuentes, consulte [tamaños de VM de Azure sin disco temporal local](azure-vms-no-temp-disk.md).

## <a name="ev4-series"></a>Serie Ev4

Los tamaños de la serie Ev4 se ejecutan en Intel Xeon&reg; Platinum 8272CL (Cascade Lake). Las instancias de la serie Ev4 son ideales para aplicaciones empresariales de uso intensivo de memoria. Las VM de la serie Ev4 cuentan con la tecnología Hyper-Threading de Intel&reg;.

El almacenamiento en disco de datos remotos se factura de forma independiente a las máquinas virtuales. Para usar discos de Premium Storage, utilice los tamaños de la serie Esv4. Los precios y los medidores de facturación para los tamaños de la serie Esv4 son los mismos que para la serie Ev4.

ACU: 195 - 210

Premium Storage:  No compatible

Almacenamiento en caché de Premium Storage:  No compatible

Migración en vivo: Compatible

Actualizaciones con conservación de memoria: Compatible

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Discos de datos máx. | Nº máx. NIC|Ancho de banda de red esperado (Mbps) |
|---|---|---|---|---|---|---|
| Standard_E2_v4  | 2 | 16   | Solo almacenamiento remoto | 4 | 2|1000  |
| Standard_E4_v4  | 4 | 32  | Solo almacenamiento remoto | 8 | 2|2000  |
| Standard_E8_v4  | 8 | 64 | Solo almacenamiento remoto | 16 | 4|4000 |
| Standard_E16_v4 | 16 | 128 | Solo almacenamiento remoto | 32 | 8|8000 |
| Standard_E20_v4 | 20 | 160 | Solo almacenamiento remoto | 32 | 8|10000 |
| Standard_E32_v4 | 32 | 256 | Solo almacenamiento remoto | 32 | 8|16000 |
| Standard_E48_v4 | 48 | 384 | Solo almacenamiento remoto | 32 | 8|24000 |
| Standard_E64_v4 | 64 | 504 | Solo almacenamiento remoto | 32| 8|30000 |


## <a name="esv4-series"></a>Serie Esv4

Los tamaños de la serie Esv4 se ejecutan en Intel&reg; Xeon&reg; Platinum 8272CL (Cascade Lake). Las instancias de la serie Esv4 son ideales para aplicaciones empresariales de uso intensivo de memoria. Las VM de la serie Evs4 cuentan con la tecnología Hyper-Threading de Intel&reg;. El almacenamiento en disco de datos remotos se factura de forma independiente a las máquinas virtuales.

ACU: 195-210

Premium Storage:  Compatible

Almacenamiento en caché de Premium Storage:  Compatible

Migración en vivo: Compatible

Actualizaciones con conservación de memoria: Compatible

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Discos de datos máx. | Rendimiento máximo del disco sin almacenamiento en la caché: IOPS/Mbps | Nº máx. NIC|Ancho de banda de red esperado (Mbps) |
|---|---|---|---|---|---|---|---|
| Standard_E2s_v4  | 2 | 16  | Solo almacenamiento remoto | 4 | 3200/48 | 2|1000  |
| Standard_E4s_v4  | 4 | 32  | Solo almacenamiento remoto | 8 | 6400/96 | 2|2000  |
| Standard_E8s_v4  | 8 | 64  | Solo almacenamiento remoto | 16 | 12800/192 | 4|4000 |
| Standard_E16s_v4 | 16 | 128 | Solo almacenamiento remoto | 32 | 25600/384 | 8|8000 |
| Standard_E20s_v4 | 20 | 160 | Solo almacenamiento remoto | 32 | 32000/480  | 8|10000 |
| Standard_E32s_v4 | 32 | 256 | Solo almacenamiento remoto | 32 | 51200/768  | 8|16000 |
| Standard_E48s_v4 | 48 | 384 | Solo almacenamiento remoto | 32 | 76800/1152 | 8|24000 |
| Standard_E64s_v4 <sup>1</sup> | 64 | 504| Solo almacenamiento remoto | 32 | 80000/1200 | 8|30000 |

<sup>1</sup> [Tamaños de núcleos restringidos disponibles](./constrained-vcpu.md).

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="other-sizes-and-information"></a>Otros tamaños e información

- [Uso general](sizes-general.md)
- [Memoria optimizada](sizes-memory.md)
- [Almacenamiento optimizado](sizes-storage.md)
- [GPU optimizada](sizes-gpu.md)
- [Proceso de alto rendimiento](sizes-hpc.md)
- [Generaciones anteriores](sizes-previous-gen.md)

Calculadora de precios: [Calculadora de precios](https://azure.microsoft.com/pricing/calculator/)

Más información sobre los tipos de disco: [Tipos de disco](./disks-types.md#ultra-disk)


## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cómo las [unidades de proceso de Azure (ACU)](acu.md) pueden ayudarlo a comparar el rendimiento en los distintos SKU de Azure.
