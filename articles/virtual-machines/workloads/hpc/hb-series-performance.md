---
title: Rendimiento del espacio de las máquinas virtuales de la serie HB (Azure Virtual Machines)| Microsoft Docs
description: Conozca los resultado de la prueba de rendimiento de los distintos tamaños de las máquinas virtuales de la serie HB en Azure.
services: virtual-machines
documentationcenter: ''
author: vermagit
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: article
ms.date: 05/15/2019
ms.author: amverma
ms.openlocfilehash: 4745fda5417ea937c367a116f999070bd62cf8cd
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87077414"
---
# <a name="hb-series-virtual-machine-sizes"></a>Tamaños de las máquinas virtuales de la serie HB

Se han realizado varias pruebas de rendimiento en los distintos tamaños de la serie HB. Estos son algunos de los resultados de esta prueba.


| Carga de trabajo                                        | HB                    |
|-------------------------------------------------|-----------------------|
| STREAM Triad                                    | ~ 260 GB/s (32 - 33 GB/s por CCX)  |
| High-Performance Linpack (HPL)                  | ~ 1000 GigaFLOPS (Rpeak), ~ 860 GigaFLOPS (Rmax) |
| Latencia y ancho de banda de RDMA                        | 2,35usec; 96,5 Gb/s   |
| FIO en SSD NVMe local                           | ~1,7 GB/s en lecturas; ~1,0 GB/s en escrituras      |  
| IOR en SSD Premium 4 * Azure (P30 Managed Disks, RAID0) **  | ~725 MB/s en lecturas, ~780 MB/escrituras   |



## <a name="infiniband-send-latency"></a>Latencia de envío de InfiniBand
Mellanox Perftest.

```console
numactl --physcpubind=[INSERT CORE #]  ib_send_lat -a
```


|  #bytes         | #iterations     | t_min [microsegundos]     | t_max [microsegundos]     | t_typical [microsegundos] | t_avg [microsegundos]     | t_stdev [microsegundos]   |
|-----------------|-----------------|-----------------|-----------------|-----------------|-----------------|
| 2               | 1000            | 2,35            | 12,63           | 2,38            | 2,42            | 0,33            |
| 4               | 1000            | 2,35            | 18,53           | 2,38            | 2.4             | 0,21            |
| 8               | 1000            | 2,36            | 6,06            | 2,39            | 2.41            | 0,22            |
| 16              | 1000            | 2,36            | 6,05            | 2,39            | 2.41            | 0,21            |
| 32              | 1000            | 2,37            | 18,93           | 2.4             | 2,42            | 0,25            |
| 64              | 1000            | 2,39            | 17,98           | 2,43            | 2.45            | 0,18            |
| 128             | 1000            | 2.44            | 19,4            | 2.76            | 2.65            | 0,29            |
| 256             | 1000            | 3,06            | 18,31           | 3.1             | 3,12            | 0,27            |
| 512             | 1000            | 3,15            | 7,89            | 3.2             | 3,23            | 0,31            |
| 1024            | 1000            | 3,27            | 17,62           | 3,31            | 3,33            | 0,22            |
| 2048            | 1000            | 3.48            | 7,94            | 3.52            | 3.55            | 0.26            |
| 4096            | 1000            | 3,91            | 7,7             | 3,96            | 3,98            | 0,21            |


## <a name="osu-mpi-latency-test"></a>Prueba de latencia de MPI de OSU

Prueba de latencia de MPI de OSU, versión 5.4.3.

```azure-cli
./bin/mpirun_rsh -np 2 -hostfile ~/hostfile MV2_CPU_MAPPING=[INSERT CORE #] ./osu_latency 
```


| #bytes  | Latencia [microsegundos] (MPICH 3.3 + CH4) | Latencia [microsegundos] (OpenMPI 4.0.0) | Latencia [microsegundos] (MVAPICH2 2.3) | Latencia [microsegundos] (Intel MPI 2019) |
|------|----------|----------|----------|----------|
| 2    | 2.44     | 2.52     | 2,84     | 2.76     |
| 4    | 2.44     | 2.53     | 2,84     | 2.76     |
| 8    | 2.44     | 2.53     | 2,83     | 2.76     |
| 16   | 2.45     | 2.53     | 2.87     | 2.77     |
| 32   | 2.62     | 2.69     | 2,89     | 2.78     |
| 64   | 2.72     | 2.79     | 2,93     | 2,85     |
| 128  | 2.76     | 2,88     | 3,06     | 2,91     |
| 256  | 3,53     | 3.65     | 3.73     | 3.57     |
| 512  | 3.68     | 3,78     | 3,81     | 3,70     |
| 1024 | 3,86     | 3,97     | 3,95     | 3,93     |
| 2048 | 4,12     | 4.5.      | 4.24     | 4,22     |
| 4096 | 4,79     | 5.28     | 6,33     | 4,91     |


## <a name="mpi-bandwidth"></a>Ancho de banda de MPI

Prueba de ancho de banda de MPI de OSU, versión 5.4.3.

```azure-cli
./mvapich2-2.3.install/bin/mpirun_rsh -np 2 -hostfile ~/hostfile MV2_CPU_MAPPING=[INSERT CORE #] ./mvapich2-2.3/osu_benchmarks/mpi/pt2pt/osu_bw
```

| #Size            | Ancho de banda (MB/s) | Ancho de banda (Gb/s) |
|------------------|------------------|------------------|
| 2                | 4,03             | 0,03             |
| 4                | 8,2              | 0,07             |
| 8                | 16,15            | 0,13             |
| 16               | 32,33            | 0.26             |
| 32               | 64,36            | 0,51             |
| 64               | 126,29           | 1.01             |
| 128              | 234,14           | 1,87             |
| 256              | 486,89           | 3,90             |
| 512              | 874,24           | 6,99             |
| 1024             | 1538,47          | 12,31            |
| 2048             | 2743,98          | 21,95            |
| 4096             | 4194,69          | 33,56            |
| 8192             | 5657,67          | 45,26            |
| 16384            | 7618,96          | 60,95            |
| 32 768            | 10333,76         | 82,67            |
| 65536            | 11 171,06         | 89,37            |
| 131 072           | 11 539,64         | 92,32            |
| 262 144           | 11 768,43         | 94,15            |
| 524 288           | 11 908,59         | 95,27            |
| 1 048 576          | 12 012,8          | 96,10            |
| 2 097 152          | 12 049,38         | 96,40            |
| 4 194 304          | 12 061,33         | 96,49            |


## <a name="next-steps"></a>Pasos siguientes

Más información sobre la [informática de alto rendimiento](/azure/architecture/topics/high-performance-computing/) en Azure.
