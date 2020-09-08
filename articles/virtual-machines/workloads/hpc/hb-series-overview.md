---
title: Introducción a las máquinas virtuales de la serie HB - Azure Virtual Machines| Microsoft Docs
description: Obtenga información sobre la compatibilidad en versión preliminar con el tamaño de las máquinas virtuales de la serie HB en Azure.
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
ms.openlocfilehash: 7c66af5184c4a943fd4b3185a87623112fe0d954
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88691248"
---
# <a name="hb-series-virtual-machines-overview"></a>Información general sobre las máquinas virtuales de la serie HB

Maximizar el rendimiento de la aplicación de proceso de alto rendimiento (HPC) en AMD EPYC requiere un enfoque bien meditado con respecto a la ubicación de los procesos y de la memoria. A continuación se describen la arquitectura de AMD EPYC y nuestra implementación de la misma en Azure para aplicaciones HPC. Vamos a utilizar el término "pNUMA" para referirnos a un dominio de NUMA físico y "vNUMA" para referirnos a un dominio virtualizado de NUMA.

Físicamente, un servidor de la [serie HB](../../hb-series.md) tiene dos CPU EPYC 7551 con 32 núcleos cada una, lo que hace un total de 64 núcleos físicos. Estos 64 núcleos se divide en 16 dominios pNUMA (8 por socket), cada uno de los cuales es de cuatro núcleos y se conoce como "CPU compleja" (o "CCX"). Cada CCX tiene su propia memoria caché L3, que es el modo en que un sistema operativo verá un límite pNUMA/vNUMA. Un par de recursos compartidos CCXs adyacentes acceden a dos canales de DRAM física (32 GB de DRAM en servidores de la serie HB).

Para dejar margen suficiente para que el hipervisor de Azure pueda trabajar sin interferir con la máquina virtual, reservamos el dominio 0 pNUMA físico (el primer CCX). A continuación, asignamos los dominios pNUMA 1 a 15 (las unidades CCX restantes) para la máquina virtual. La máquina virtual verá:

`(15 vNUMA domains) * (4 cores/vNUMA) = 60` núcleos por máquina virtual

La propia máquina virtual no sabe que pNUMA 0 no se le proporcionó. La máquina virtual entiende pNUMA 1 y 15 como vNUMA de 0 a 14, con 7 vNUMA en el vSocket 0 y 8 vNUMA en el vSocket 1. Aunque esto es asimétrico, el sistema operativo debería arrancar y funcionar con normalidad. Más adelante en esta guía, se indica a la mejor forma para ejecutar aplicaciones MPI en este diseño NUMA asimétrico.

El anclaje de procesos funcionará en las máquinas virtuales de la serie HB porque exponemos el silicio subyacente tal cual está a la máquina virtual invitada. Se recomienda encarecidamente anclar los procesos para disfrutar de una coherencia y un rendimiento óptimos.

En el siguiente diagrama se muestra la segregación de los núcleos reservados para el hipervisor de Azure y la máquina virtual de la serie HB.

![Segregación de los núcleos reservados para el hipervisor de Azure y la máquina virtual de la serie HB](./media/hb-series-overview/segregation-cores.png)

## <a name="hardware-specifications"></a>Especificaciones del hardware

| Especificaciones del hardware                | VM de la serie HB                     |
|----------------------------------|----------------------------------|
| Núcleos                            | 60 (SMT deshabilitado)                |
| CPU                              | AMD EPYC 7551                    |
| Frecuencia de CPU (no AVX)          | ~2,55 GHz (solo + todos los núcleos)   |
| Memoria                           | 4 GB/núcleo (240 GB en total)         |
| Disco local                       | SSD de 700 GB                       |
| Infiniband                       | EDR de 100 Gb Mellanox ConnectX-5 |
| Red                          | Ethernet de 50 Gb (40 Gb útiles) SmartNIC de segunda generación de Azure |

## <a name="software-specifications"></a>Especificaciones de software

| Especificaciones de software           |VM de la serie HB           |
|-----------------------------|-----------------------|
| Tamaño de trabajo de MPI máximo            | 18000 núcleos (300 máquinas virtuales en un solo conjunto de escalado de máquinas virtuales con singlePlacementGroup=true)  |
| Compatibilidad con MPI                 | HPC-X, Intel MPI, OpenMPI, MVAPICH2, MPICH, Platform MPI  |
| Otros marcos       | Unified Communication X, libfabric, PGAS |
| Soporte técnico para Azure Storage       | Discos estándar y premium (cuatro discos como máximo) |
| Soporte técnico de sistemas operativos para SRIOV RDMA   | CentOS/RHEL 7.6+, SLES 12 SP4+, WinServer 2016+  |
| Compatibilidad con Orchestrator        | CycleCloud, Batch  |

## <a name="next-steps"></a>Pasos siguientes

- Aprenda sobre la [arquitectura de EPYC AMD](https://bit.ly/2Epv3kC) y las [arquitecturas de varios chips](https://bit.ly/2GpQIMb). Para más información, consulte la [guía de optimización de HPC para procesadores de AMD EPYC](https://bit.ly/2T3AWZ9).
- En los [blogs de la comunidad de Azure Compute Tech](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute), encontrará los anuncios más recientes y algunos ejemplos y resultados de HPC.
- Si desea una visión general de la arquitectura de la ejecución de cargas de trabajo de HPC, consulte [Informática de alto rendimiento (HPC) en Azure](/azure/architecture/topics/high-performance-computing/).
