---
title: 'Serie Dv2 y Dsv2: máquinas virtuales de Azure'
description: Especificaciones de las máquinas virtuales de la serie Dv2 y Dsv2.
author: joelpelley
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 02/03/2020
ms.author: jushiman
ms.openlocfilehash: dfb10e3f8386b4ab022d419d356f841a9ea7ea66
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87825674"
---
# <a name="dv2-and-dsv2-series"></a>Series Dv2 y DSv2

Las series Dv2 y Dsv2, una continuación de la serie D original, presentan una CPU más potente y una configuración óptima de la CPU a la memoria, lo que las hace adecuadas para la mayoría de las cargas de trabajo de producción. La serie Dv2 es un 35 % aproximadamente más rápida que la serie D. La serie Dv2 se ejecuta en procesadores Intel® Xeon® Platinum 8272CL (Cascade Lake), Intel® Xeon® 8171M 2,1 GHz (Skylake), Intel® Xeon® E5-2673 v4 2,3 GHz (Broadwell) o Intel® Xeon® E5-2673 v3 2,4 GHz (Haswell) con la tecnología Intel Turbo Boost 2.0. La serie Dv2 tiene las mismas configuraciones de disco y memoria que la serie D.

## <a name="dv2-series"></a>Serie Dv2

Los tamaños de la serie Dv2 se ejecutan en procesadores Intel® Xeon® Platinum 8272CL (Cascade Lake), Intel® Xeon® 8171M 2,1 GHz (Skylake), Intel® Xeon® E5-2673 v4 2,3 GHz (Broadwell) o Intel® Xeon® E5-2673 v3 2,4 GHz (Haswell) con la tecnología Intel Turbo Boost 2.0.

ACU: 210-250

Premium Storage:  No compatible

Almacenamiento en caché de Premium Storage:  No compatible

Migración en vivo: Compatible

Actualizaciones con conservación de memoria: Compatible

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Rendimiento máximo de almacenamiento temporal: IOPS/MBps de lectura/MBps de escritura | Discos de datos máx. | Rendimiento: E/S | Nº máx. NIC | Ancho de banda de red esperado (Mbps) |
|---|---|---|---|---|---|---|---|---|
| Standard_D1_v2 | 1  | 3,5 | 50  | 3000/46/23    | 4  | 4x500  | 2|750   |
| Standard_D2_v2 | 2  | 7   | 100 | 6000/93/46    | 8  | 8x500  | 2|1.500  |
| Standard_D3_v2 | 4  | 14  | 200 | 12000/187/93  | 16 | 16x500 | 4|3000  |
| Standard_D4_v2 | 8  | 28  | 400 | 24000/375/187 | 32 | 32x500 | 8|6000  |
| Standard_D5_v2 | 16 | 56  | 800 | 48000/750/375 | 64 | 64x500 | 8|12000 |

## <a name="dsv2-series"></a>DSv2-series

Los tamaños de la serie DSv2 se ejecutan en procesadores Intel® Xeon® Platinum 8272CL (Cascade Lake), Intel® Xeon® 8171M 2,1 GHz (Skylake), Intel® Xeon® E5-2673 v4 2,3 GHz (Broadwell) o Intel® Xeon® E5-2673 v3 2,4 GHz (Haswell) con la tecnología Intel Turbo Boost 2.0 y usan Premium Storage.

ACU: 210-250

Premium Storage:  Compatible

Almacenamiento en caché de Premium Storage:  Compatible

Migración en vivo: Compatible

Actualizaciones con conservación de memoria: Compatible

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Discos de datos máx. | Rendimiento máximo de almacenamiento temporal y en caché: IOPS/Mbps (tamaño de caché en GiB) | Rendimiento máximo del disco sin almacenamiento en la caché: IOPS/Mbps | Nº máx. NIC|Ancho de banda de red esperado (Mbps) |
|---|---|---|---|---|---|---|---|---|
| Standard_DS1_v2 | 1  | 3,5 | 7   | 4  | 4000/32 (43)    | 3200/48   | 2|750   |
| Standard_DS2_v2 | 2  | 7   | 14  | 8  | 8000/64 (86)    | 6400/96   | 2|1.500  |
| Standard_DS3_v2 | 4  | 14  | 28  | 16 | 16000/128 (172) | 12800/192 | 4|3000  |
| Standard_DS4_v2 | 8  | 28  | 56  | 32 | 32000/256 (344) | 25600/384 | 8|6000  |
| Standard_DS5_v2 | 16 | 56  | 112 | 64 | 64000/512 (688) | 51200/768 | 8|12000 |

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="other-sizes-and-information"></a>Otros tamaños e información

- [Uso general](sizes-general.md)
- [Memoria optimizada](sizes-memory.md)
- [Almacenamiento optimizado](sizes-storage.md)
- [GPU optimizada](sizes-gpu.md)
- [Proceso de alto rendimiento](sizes-hpc.md)
- [Generaciones anteriores](sizes-previous-gen.md)

Calculadora de precios: [Calculadora de precios](https://azure.microsoft.com/pricing/calculator/)

Más información sobre los tipos de disco: [Tipos de disco](./linux/disks-types.md#ultra-disk)

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cómo las [unidades de proceso de Azure (ACU)](acu.md) pueden ayudarlo a comparar el rendimiento en los distintos SKU de Azure.