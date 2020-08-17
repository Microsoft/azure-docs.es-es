---
title: Series Dav4 y Dasv4
description: Especificaciones de las máquinas virtuales de las series Dav4 y Dasv4.
author: migerdes
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 02/03/2020
ms.author: jushiman
ms.openlocfilehash: 570e2798cf2ae5a20d55973622cca9344ea4a4a3
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87837149"
---
# <a name="dav4-and-dasv4-series"></a>Series Dav4 y Dasv4

Las series Dav4 y Dasv4 son tamaños nuevos que utilizan el procesador EPYC<sup>TM</sup> 7452 de 2,35 GHz de AMD en una configuración de varios subprocesos con una caché L3 de hasta 256 MB que dedica 8 MB de esa caché L3 a cada 8 núcleos. De este modo, aumentan las opciones que tiene el cliente para ejecutar sus cargas de trabajo de uso general. Las series Dav4 y Dasv4 tienen las mismas configuraciones de memoria y disco que las series D y Dsv3.

## <a name="dav4-series"></a>Serie Dav4

ACU: 230-260

Premium Storage: No compatible

Almacenamiento en caché de Premium Storage: No compatible

Migración en vivo: Compatible

Actualizaciones con conservación de memoria: Compatible

Los tamaños de la serie Dav4 se basan en el procesador EPYC<sup>TM</sup> 7452 de AMD de 2,35 Ghz que pueden alcanzar una frecuencia máxima incrementada de 3,35 Ghz. Los tamaños de la serie Dav4 ofrecen una combinación de vCPU, memoria y almacenamiento local adecuados para la mayoría de las cargas de trabajo de producción. El almacenamiento en disco de datos se factura de forma independiente a las máquinas virtuales. Para usar SSD Premium, use los tamaños de Dasv4. El precio y los medidores de facturación para los tamaños Dasv4 son los mismos que para la serie Dav4.

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Discos de datos máx. | Rendimiento máximo de almacenamiento temporal: IOPS / MBps de lectura / MBps de escritura | Nº máx. NIC | Ancho de banda de red esperado (Mbps) |
|-----|-----|-----|-----|-----|-----|-----|-----|
| Standard_D2a_v4 |  2  | 8  | 50  | 4  | 3000 / 46 / 23   | 2 | 1000 |
| Standard_D4a_v4 |  4  | 16 | 100 | 8  | 6000 / 93 / 46   | 2 | 2000 |
| Standard_D8a_v4 |  8  | 32 | 200 | 16 | 12000 / 187 / 93 | 4 | 4000 |
| Standard_D16a_v4|  16 | 64 | 400 |32  | 24000 / 375 / 187 |8 | 8000 |
| Standard_D32a_v4|  32 | 128| 800 | 32 | 48000 / 750 / 375 |8 | 16000 |
| Standard_D48a_v4| 48 | 192| 1200 | 32 | 96000 / 1000 / 500 | 8 | 24000 |
| Standard_D64a_v4| 64 | 256 | 1600 | 32 | 96000 / 1000 / 500 | 8 | 30000 |
| Standard_D96a_v4| 96 | 384 | 2400 | 32 | 96000 / 1000 / 500 | 8 | 30000 |

## <a name="dasv4-series"></a>Serie Dasv4

ACU:  230-260

Premium Storage: Compatible

Almacenamiento en caché de Premium Storage: Compatible

Migración en vivo: Compatible

Actualizaciones con conservación de memoria: Compatible

Los tamaños de la serie Dasv4 se basan en el procesador EPYC<sup>TM</sup> 7452 de AMD de 2,35 Ghz que pueden alcanzar una frecuencia máxima incrementada de 3,35 Ghz y usar SSD Premium. Los tamaños de la serie Dasv4 ofrecen una combinación de vCPU, memoria y almacenamiento local adecuados para la mayoría de las cargas de trabajo de producción.

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Discos de datos máx. | Rendimiento máximo de almacenamiento temporal y en caché: IOPS / MBps (tamaño de caché en GiB) | Rendimiento máximo del disco sin almacenamiento en la caché: IOPS / MBps | Nº máx. NIC | Ancho de banda de red esperado (Mbps) |
|-----|-----|-----|-----|-----|-----|-----|-----|-----|
| Standard_D2as_v4|2|8|16|4|4000/32 (50)|3200/48|2 | 1000 |
| Standard_D4as_v4|4|16|32|8|8000/64 (100)|6400/96|2 | 2000 |
| Standard_D8as_v4|8|32|64|16|16000/128 (200)|12800/192|4 | 4000 |
| Standard_D16as_v4|16|64|128|32|32 000 / 255 (400)|25600/384|8 | 8000 |
| Standard_D32as_v4|32|128|256|32|64 000 / 510 (800)|51200/768|8 | 16000 |
| Standard_D48as_v4|48|192|384|32|96000/1020 (1200)|76800/1148|8 | 24000 |
| Standard_D64as_v4|64|256|512|32|128000/1020 (1600)|80000/1200|8 | 30000 | 
| Standard_D96as_v4|96|384|768|32|192000/1020 (2400)|80000/1200|8 | 30000 |

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