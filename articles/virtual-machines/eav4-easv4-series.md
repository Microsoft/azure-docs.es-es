---
title: Series Eav4 y Easv4
description: Especificaciones de las máquinas virtuales de las series Eav4 y Easv4.
author: migerdes
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 02/03/2020
ms.author: jushiman
ms.openlocfilehash: 775b296185f9f0f0e967ada2688e24158f36360e
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88648367"
---
# <a name="eav4-and-easv4-series"></a>Series Eav4 y Easv4

Las series Eav4 y Easv4 emplean el procesador EPYC<sup>TM</sup> 7452 de 2,35 GHz de AMD en una configuración de varios subprocesos con una caché L3 de hasta 256 MB, lo que aumenta las opciones para ejecutar la mayoría de las cargas de trabajo optimizadas para memoria. Las series Eav4 y Easv4 tienen las mismas configuraciones de memoria y disco que las series Ev3 y Esv3.

## <a name="eav4-series"></a>Serie Eav4

ACU: 230 - 260

Premium Storage: No compatible

Almacenamiento en caché de Premium Storage: No compatible

Migración en vivo: Compatible

Actualizaciones con conservación de memoria: Compatible

Los tamaños de la serie Eav4 se basan en el procesador EPYC<sup>TM</sup> 7452 de AMD de 2,35 Ghz que pueden alcanzar una frecuencia máxima incrementada de 3,35 Ghz y usar SSD Premium. Los tamaños de la serie Eav4 son ideales para aplicaciones empresariales de uso intensivo de memoria. El almacenamiento en disco de datos se factura de forma independiente a las máquinas virtuales. Para usar SSD Premium, use los tamaños de la serie Easv4. El precio y los medidores de facturación para los tamaños Easv4 son los mismos que para la serie Eav3.

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Discos de datos máx. | Rendimiento máximo de almacenamiento temporal: IOPS / MBps de lectura / MBps de escritura | Nº máx. NIC | Ancho de banda de red esperado (Mbps) |
| -----|-----|-----|-----|-----|-----|-----|-----|
| Standard\_E2a\_v4|2|16|50|4|3000 / 46 / 23|2 | 1000 |
| Standard\_E4a\_v4|4|32|100|8|6000 / 93 / 46|2 | 2000 |
| Standard\_E8a\_v4|8|64|200|16|12000 / 187 / 93|4 | 4000 |
| Standard\_E16a\_v4|16|128|400|32|24000 / 375 / 187|8 | 8000 |
| Standard\_E20a\_v4|20|160|500|32|30000 / 468 / 234|8 | 10 000 |
| Standard\_E32a\_v4|32|256|800|32|48000 / 750 / 375|8 | 16000 |
| Standard\_E48a\_v4|48|384|1200|32|96000/1000 (500)|8 | 24000 |
| Standard\_E64a\_v4|64|512|1600|32|96000/1000 (500)|8 | 30000 |
| Standard\_E96a\_v4|96|672|2400|32|96000/1000 (500)|8 | 30000 |

## <a name="easv4-series"></a>Serie Easv4

ACU: 230 - 260

Premium Storage: Compatible

Almacenamiento en caché de Premium Storage: Compatible

Migración en vivo: Compatible

Actualizaciones con conservación de memoria: Compatible

Los tamaños de la serie Easv4 se basan en el procesador EPYC<sup>TM</sup> 7452 de AMD de 2,35 Ghz que pueden alcanzar una frecuencia máxima incrementada de 3,35 Ghz y usar SSD Premium. Los tamaños de la serie Easv4 son ideales para aplicaciones empresariales de uso intensivo de memoria.

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Discos de datos máx. | Rendimiento máximo de almacenamiento temporal y en caché: IOPS / MBps (tamaño de caché en GiB) | Rendimiento máximo del disco sin almacenamiento en la caché: IOPS / MBps | Nº máx. NIC | Ancho de banda de red esperado (MBps) |
|-----|-----|-----|-----|-----|-----|-----|-----|-----|
| Standard_E2as_v4|2|16|32|4|4000/32 (50)|3200/48|2 | 1000 |
| Standard_E4as_v4|4|32|64|8|8000/64 (100)|6400/96|2 | 2000 |
| Standard_E8as_v4|8|64|128|16|16000/128 (200)|12800/192|4 | 4000 |
| Standard_E16as_v4|16|128|256|32|32 000 / 255 (400)|25600/384|8 | 8000 |
| Standard_E20as_v4|20|160|320|32|40000 / 320 (500)|32000 / 480|8 | 10 000 |
| Standard_E32as_v4|32|256|512|32|64 000 / 510 (800)|51200/768|8 | 16000 |
| Standard_E48as_v4|48|384|768|32|96000/1020 (1200)|76800/1148|8 | 24000 |
| Standard_E64as_v4|64|512|1024|32|128000/1020 (1600)|80000/1200|8 | 30000 |
| Standard_E96as_v4|96|672|1344|32|192000/1020 (2400)|80000/1200|8 | 30000 |

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
