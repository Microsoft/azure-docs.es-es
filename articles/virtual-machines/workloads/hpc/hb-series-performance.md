---
title: Rendimiento del tamaño de máquina virtual de la serie HB
description: Conozca los resultado de la prueba de rendimiento de los distintos tamaños de las máquinas virtuales de la serie HB en Azure.
author: vermagit
manager: gwallace
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: article
ms.date: 09/09/2020
ms.author: amverma
ms.openlocfilehash: 34e9ef3ab46f2ce11500aa87db9676635d3e9b4f
ms.sourcegitcommit: 43558caf1f3917f0c535ae0bf7ce7fe4723391f9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90016289"
---
# <a name="hb-series-virtual-machine-sizes"></a>Tamaños de las máquinas virtuales de la serie HB

Se han realizado varias pruebas de rendimiento en los distintos tamaños de la serie HB. Estos son algunos de los resultados de esta prueba.

| Carga de trabajo                                        | HB                    |
|-------------------------------------------------|-----------------------|
| STREAM Triad                                    | 260 GB/s (32-33 GB/s por CCX)  |
| High-Performance Linpack (HPL)                  | 1000 GigaFLOPS (Rpeak), 860 GigaFLOPS (Rmax) |
| Latencia y ancho de banda de RDMA                        | 1,27 microsegundos; 99,1 Gb/s   |
| FIO en SSD NVMe local                           | 1,7 GB/s en lecturas; 1,0 GB/s en escrituras      |  
| IOR en SSD Premium 4 * Azure (P30 Managed Disks, RAID0) **  | 725 MB/s en lecturas, 780 MB/s en escrituras   |


## <a name="mpi-latency"></a>Latencia de MPI

Se ejecuta la prueba de latencia de MPI del conjunto de pruebas OSU de microrealización. Los scripts de ejemplo se encuentran en [GitHub](https://github.com/Azure/azhpc-images/blob/04ddb645314a6b2b02e9edb1ea52f079241f1297/tests/run-tests.sh)

```bash
./bin/mpirun_rsh -np 2 -hostfile ~/hostfile MV2_CPU_MAPPING=[INSERT CORE #] ./osu_latency 
```

:::image type="content" source="./media/latency-hb.png" alt-text="Latencia de MPI en HB de Azure.":::

## <a name="mpi-bandwidth"></a>Ancho de banda de MPI

Se ejecuta la prueba de ancho de banda de MPI del conjunto de pruebas OSU de microrealización. Los scripts de ejemplo se encuentran en [GitHub](https://github.com/Azure/azhpc-images/blob/04ddb645314a6b2b02e9edb1ea52f079241f1297/tests/run-tests.sh)

```bash
./mvapich2-2.3.install/bin/mpirun_rsh -np 2 -hostfile ~/hostfile MV2_CPU_MAPPING=[INSERT CORE #] ./mvapich2-2.3/osu_benchmarks/mpi/pt2pt/osu_bw
```

:::image type="content" source="./media/bandwidth-hb.png" alt-text="Latencia de MPI en HB de Azure.":::


## <a name="mellanox-perftest"></a>Mellanox Perftest

El [paquete Mellanox Perftest](https://community.mellanox.com/s/article/perftest-package) tiene muchas pruebas de InfiniBand, como la de latencia (ib_send_lat) y ancho de banda (ib_send_bw). El siguiente es un ejemplo de comando.

```console
numactl --physcpubind=[INSERT CORE #]  ib_send_lat -a
```

## <a name="next-steps"></a>Pasos siguientes

- En los [blogs de la comunidad de Azure Compute Tech](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute), encontrará los anuncios más recientes y algunos ejemplos y resultados de informática de alto rendimiento (HPC).
- Si quiere una visión general de la arquitectura de la ejecución de cargas de trabajo de HPC, consulte [Informática de alto rendimiento (HPC) en Azure](/azure/architecture/topics/high-performance-computing/).
