---
title: 'Serie HBv2: Azure Virtual Machines'
description: Especificaciones de las máquinas virtuales de la serie HBv2.
author: vermagit
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 02/03/2020
ms.author: amverma
ms.openlocfilehash: f9717105c9241777d72a8943e87f33ab31c8038c
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87288482"
---
# <a name="hbv2-series"></a>Serie HBv2

Las máquinas virtuales de la serie HBv2 están optimizadas para aplicaciones impulsadas por el ancho de banda de la memoria, como la dinámica de fluidos, el análisis de elementos finitos y la simulación de yacimientos. Las máquinas virtuales HBv2 cuentan con 120 núcleos de procesador AMD EPYC 7742, 4 GB de RAM por núcleo de CPU y sin multithreading simultáneo. Cada máquina virtual de HBv2 proporciona hasta 340 GB/s de ancho de banda de memoria y hasta 4 teraFLOPS de proceso FP64.

Premium Storage: Compatible

Migración en vivo: No compatible

Actualizaciones con conservación de memoria: No compatible

| Size | vCPU | Procesador | Memoria (GB) | Ancho de banda de memoria, en GB/s | Frecuencia de CPU base (GHz) | Frecuencia de todos los núcleos (GHz, pico) | Frecuencia de cada núcleo (GHz, pico) | Rendimiento de RDMA (GB/s) | Compatibilidad con MPI | Almacenamiento temporal (GB) | Discos de datos máx. | NIC Ethernet máx. |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_HB120rs_v2 | 120 | AMD EPYC 7V12 | 480 | 350 | 2.45 | 3.1 | 3.3 | 200 | All | 480 + 960 | 8 | 1 |


[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="other-sizes"></a>Otros tamaños

- [Uso general](sizes-general.md)
- [Memoria optimizada](sizes-memory.md)
- [Almacenamiento optimizado](sizes-storage.md)
- [GPU optimizada](sizes-gpu.md)
- [Proceso de alto rendimiento](sizes-hpc.md)
- [Generaciones anteriores](sizes-previous-gen.md)

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cómo las [unidades de proceso de Azure (ACU)](acu.md) pueden ayudarlo a comparar el rendimiento en los distintos SKU de Azure.