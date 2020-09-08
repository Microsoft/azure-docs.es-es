---
title: Introducción a las máquinas virtuales de la serie HC - Azure Virtual Machines| Microsoft Docs
description: Obtenga información sobre la compatibilidad en versión preliminar con el tamaño de las máquinas virtuales de la serie HC en Azure.
services: virtual-machines
documentationcenter: ''
author: vermagit
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: article
ms.date: 08/19/2020
ms.author: amverma
ms.reviewer: cynthn
ms.openlocfilehash: de6051e8880bbe3df42031a0d0d7b60abc27d2b0
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88689806"
---
# <a name="hc-series-virtual-machine-overview"></a>Introducción a las máquinas virtuales de la serie HC

Para poder maximizar el rendimiento de las aplicaciones HPC en procesadores escalables de Intel Xeon, se requiere un enfoque bien meditado a la hora de procesar la selección de la ubicación de esta nueva arquitectura. En este artículo, vamos a tratar de esbozar la implementación de estos procesadores en máquinas virtuales de la serie HC de Azure para aplicaciones HPC. Vamos a utilizar el término "pNUMA" para referirnos a un dominio de NUMA físico y "vNUMA" para referirnos a un dominio virtualizado de NUMA. De igual modo, utilizaremos el término "pCore" para los núcleos de CPU físicos y "vCore" para los núcleos de CPU virtualizados.

Físicamente, un servidor de la [serie HC](../../hc-series.md) tiene dos CPU Xeon Platinum 8168 con 24 núcleos cada una, lo que hace un total de 48 núcleos físicos. Cada CPU es un único dominio de pNUMA y tiene acceso unificado a seis canales de DRAM. Las CPU Intel Xeon Platinum se caracterizan por tener una caché L2 cuatro veces más grande que las generaciones anteriores (256 KB/núcleo -> 1 MB/núcleo) y por tener una caché L3 más reducida que las anteriores CPU de Intel (2,5 MB/núcleo -> 1375 MB/núcleo).

La topología anterior también es aplicable a la configuración de los hipervisores de la serie HC. Para dejar margen suficiente para que el hipervisor de Azure pueda trabajar sin interferir con la máquina virtual, reservamos los pCores 0-1 y 24-25 (es decir, los dos primeros pCores de cada socket). A continuación, asignamos los dominios de pNUMA y todos los núcleos restantes a la máquina virtual. Por lo tanto, la máquina virtual verá:

`(2 vNUMA domains) * (22 cores/vNUMA) = 44` núcleos por máquina virtual

La máquina virtual no sabe que tiene asignados los pCores 0-1 y 24-25. Por tanto, expone cada vNUMA como si tuviera 22 núcleos de forma nativa.

Las CPU Intel Xeon Platinum, Gold y Silver también cuenta con una red de malla 2D en el mismo chip para realizar las comunicaciones internas y externas con el socket de la CPU. Se recomienda encarecidamente anclar los procesos para disfrutar de una coherencia y un rendimiento óptimos. En anclaje de procesos funcionará en las máquinas virtuales de la serie HC porque el silicio subyacente se expone tal cual está a la máquina virtual invitada.

En el siguiente diagrama se muestra la segregación de los núcleos reservados para el hipervisor de Azure y la máquina virtual de la serie HC.

![Segregación de los núcleos reservados para el hipervisor de Azure y la máquina virtual de la serie HC](./media/hc-series-overview/segregation-cores.png)

## <a name="hardware-specifications"></a>Especificaciones del hardware

| Especificaciones del hardware          | Máquina virtual de la serie HC                     |
|----------------------------------|----------------------------------|
| Núcleos                            | 44 (HT deshabilitado)                 |
| CPU                              | Intel Xeon Platinum 8168         |
| Frecuencia de CPU (no AVX)          | 3,7 GHz (un solo núcleo), 2,7-3,4 GHz (todos los núcleos) |
| Memoria                           | 8 GB/núcleo (total: 352)            |
| Disco local                       | SSD de 700 GB                       |
| Infiniband                       | EDR de 100 Gb Mellanox ConnectX-5   |
| Red                          | Ethernet de 50 Gb (40 Gb útiles) SmartNIC de segunda generación de Azure    |

## <a name="software-specifications"></a>Especificaciones de software

| Especificaciones de software     |Máquina virtual de la serie HC           |
|-----------------------------|-----------------------|
| Tamaño de trabajo de MPI máximo            | 13200 núcleos (300 máquinas virtuales en un solo conjunto de escalado de máquinas virtuales con singlePlacementGroup=true)  |
| Compatibilidad con MPI                 | HPC-X, Intel MPI, OpenMPI, MVAPICH2, MPICH, Platform MPI  |
| Otros marcos       | Unified Communication X, libfabric, PGAS |
| Soporte técnico para Azure Storage       | Discos estándar y premium (cuatro discos como máximo) |
| Soporte técnico de sistemas operativos para SRIOV RDMA   | CentOS/RHEL 7.6+, SLES 12 SP4+, WinServer 2016+  |
| Compatibilidad con Orchestrator        | CycleCloud, Batch  |

## <a name="next-steps"></a>Pasos siguientes

- Para más información, consulte la [arquitectura de SP de Intel Xeon](https://bit.ly/2RCYkiE).
- En los [blogs de la comunidad de Azure Compute Tech](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute), encontrará los anuncios más recientes y algunos ejemplos y resultados de HPC.
- Si desea una visión general de la arquitectura de la ejecución de cargas de trabajo de HPC, consulte [Informática de alto rendimiento (HPC) en Azure](/azure/architecture/topics/high-performance-computing/).
