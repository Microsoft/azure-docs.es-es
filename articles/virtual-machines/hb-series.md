---
title: Serie HB
description: Especificaciones de las máquinas virtuales de la serie HB.
author: ju-shim
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 08/06/2020
ms.author: amverma
ms.reviewer: jushiman
ms.openlocfilehash: e735b389b5e223e558736f5eaa3c393f8bc29004
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87926270"
---
# <a name="hb-series"></a>Serie HB

Las máquinas virtuales de la serie HB están optimizadas para aplicaciones que funcionan con ancho de banda de la memoria, como la dinámica de fluidos, el análisis explícito de elementos finitos y los modelos de clima. Las máquinas virtuales HB cuentan con 60 núcleos de procesador AMD EPYC 7551, 4 GB de RAM por núcleo de CPU y ningún multithreading simultáneo. Una máquina virtual HB proporciona hasta 260 GB/s de ancho de banda de memoria.

ACU: 199-216

Premium Storage: Compatible

Almacenamiento en caché de Premium Storage: Compatible

Migración en vivo: No compatible

Actualizaciones con conservación de memoria: No compatible

| Size | vCPU | Procesador | Memoria (GB) | Ancho de banda de memoria, en GB/s | Frecuencia de CPU base (GHz) | Frecuencia de todos los núcleos (GHz, pico) | Frecuencia de cada núcleo (GHz, pico) | Rendimiento de RDMA (GB/s) | Compatibilidad con MPI | Almacenamiento temporal (GB) | Discos de datos máx. | NIC Ethernet máx. |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_HB60rs | 60 | AMD EPYC 7551 | 240 | 263 | 2.0 | 2.55 | 2.55 | 100 | All | 700 | 4 | 1 |

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="other-sizes"></a>Otros tamaños

- [Uso general](sizes-general.md)
- [Memoria optimizada](sizes-memory.md)
- [Almacenamiento optimizado](sizes-storage.md)
- [GPU optimizada](sizes-gpu.md)
- [Proceso de alto rendimiento](sizes-hpc.md)
- [Generaciones anteriores](sizes-previous-gen.md)

## <a name="next-steps"></a>Pasos siguientes
- Obtenga más información sobre la [configuración de máquinas virtuales](./workloads/hpc/configure.md), la [habilitación de InfiniBand](./workloads/hpc/enable-infiniband.md), la [configuración de MPI](./workloads/hpc/setup-mpi.md) y la optimización de las aplicaciones HPC para Azure en el archivo sobre [cargas de trabajo de HPC](./workloads/hpc/overview.md).
- En los [blogs de la comunidad de Azure Compute Tech](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute), encontrará los anuncios más recientes y algunos ejemplos y resultados de HPC.
- Si desea una visión general de la arquitectura de la ejecución de cargas de trabajo de HPC, consulte [Informática de alto rendimiento (HPC) en Azure](/azure/architecture/topics/high-performance-computing/).
- Obtenga más información sobre cómo las [unidades de proceso de Azure (ACU)](acu.md) pueden ayudarlo a comparar el rendimiento en los distintos SKU de Azure.
