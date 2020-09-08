---
title: 'Serie H: Azure Virtual Machines'
description: Especificaciones de las máquinas virtuales de la serie H.
author: ju-shim
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 08/06/2020
ms.author: amverma
ms.reviewer: jushiman
ms.openlocfilehash: e9f876f3d20af01867283f550590b3af23dec662
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87926627"
---
# <a name="h-series"></a>Serie H

Las máquinas virtuales de la serie H están optimizadas para aplicaciones basadas en frecuencias alta de CPU o requisitos de gran cantidad de memoria por núcleo. Las máquinas virtuales de la serie H cuentan con 8 o 16 núcleos de procesador Intel Xeon E5 2667 v3, hasta 14 GB de RAM por núcleo de CPU y no tienen hyperthreading. Las máquinas de la serie H cuentan con una interconexión Mellanox FDR InfiniBand de 56 Gb/s en una configuración de árbol de FAT sin bloqueo para proporcionar un rendimiento de RDMA consistente. Las máquinas virtuales de la serie H admiten Intel MPI 5.x y MS-MPI.

ACU: 290-300

Premium Storage:  No compatible

Almacenamiento en caché de Premium Storage:  No compatible

Migración en vivo: No compatible

Actualizaciones con conservación de memoria: No compatible

| Size | vCPU | Procesador | Memoria (GB) | Ancho de banda de memoria, en GB/s | Frecuencia de CPU base (GHz) | Frecuencia de todos los núcleos (GHz, pico) | Frecuencia de cada núcleo (GHz, pico) | Rendimiento de RDMA (GB/s) | Compatibilidad con MPI | Almacenamiento temporal (GB) | Discos de datos máx. | Rendimiento máximo de discos E/S | NIC Ethernet máx. |
| --- | --- |--- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_H8   | 8  | Intel Xeon E5 2667 v3 | 56 | 40 | 3.2 | 3.3 | 3.6 | - | Intel 5.x, MS-MPI | 1000 | 32 | 32 x 500 | 2 |
| Standard_H16  | 16 | Intel Xeon E5 2667 v3 | 112 | 80 | 3.2 | 3.3 | 3.6 | - | Intel 5.x, MS-MPI | 2000 | 64 | 64 x 500 | 4 |
| Standard_H8m  | 8  | Intel Xeon E5 2667 v3 | 112 | 40 | 3.2 | 3.3 | 3.6 | - | Intel 5.x, MS-MPI | 1000 | 32 | 32 x 500 | 2 |
| Standard_H16m | 16 | Intel Xeon E5 2667 v3 | 224 | 80 | 3.2 | 3.3 | 3.6 | - | Intel 5.x, MS-MPI | 2000 | 64 | 64 x 500 | 4 |
| Standard_H16r <sup>1</sup>  | 16 | Intel Xeon E5 2667 v3 | 112 | 80 | 3.2 | 3.3 | 3.6 | 56 | Intel 5.x, MS-MPI | 2000 | 64 | 64 x 500 | 4 |
| Standard_H16mr <sup>1</sup> | 16 | Intel Xeon E5 2667 v3 | 224 | 80 | 3.2 | 3.3 | 3.6 | 56 | Intel 5.x, MS-MPI | 2000 | 64 | 64 x 500 | 4 |

<sup>1</sup> Para las aplicaciones MPI, la red de back-end RDMA dedicada está habilitada por la red InfiniBand FDR.

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

> [!NOTE]
> Entre las [máquinas virtuales compatibles con RDMA](sizes-hpc.md#rdma-capable-instances), la serie H no está habilitada para SR-IOV. Por lo tanto, las imágenes de máquina virtual [admitidas](./workloads/hpc/configure.md#vm-images), los requisitos del [controlador de InfiniBand](./workloads/hpc/enable-infiniband.md) y las [bibliotecas MPI](./workloads/hpc/setup-mpi.md) admitidas son diferentes de las máquinas virtuales habilitadas para SR-IOV.

## <a name="other-sizes"></a>Otros tamaños

- [Uso general](sizes-general.md)
- [Memoria optimizada](sizes-memory.md)
- [Almacenamiento optimizado](sizes-storage.md)
- [GPU optimizada](sizes-gpu.md)
- [Proceso de alto rendimiento](sizes-hpc.md)
- [Generaciones anteriores](sizes-previous-gen.md)

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre la [configuración de máquinas virtuales](./workloads/hpc/configure.md), la [habilitación de InfiniBand](./workloads/hpc/enable-infiniband.md), la [configuración de MPI](./workloads/hpc/setup-mpi.md) y la optimización de las aplicaciones HPC para Azure en [cargas de trabajo de HPC](./workloads/hpc/overview.md).
- En los [blogs de la comunidad de Azure Compute Tech](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute), encontrará los anuncios más recientes y algunos ejemplos y resultados de HPC.
- Si desea una visión general de la arquitectura de la ejecución de cargas de trabajo de HPC, consulte [Informática de alto rendimiento (HPC) en Azure](/azure/architecture/topics/high-performance-computing/).
- Obtenga más información sobre cómo las [unidades de proceso de Azure (ACU)](acu.md) pueden ayudarlo a comparar el rendimiento en los distintos SKU de Azure.
