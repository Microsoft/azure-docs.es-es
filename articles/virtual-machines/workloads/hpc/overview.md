---
title: 'Informática de alto rendimiento en máquinas virtuales de las series H y N habilitadas con InfiniBand: Azure Virtual Machines'
description: Obtenga información sobre las características y funcionalidades de las máquinas virtuales de las series H y N habilitadas con InfiniBand optimizadas para HPC.
author: vermagit
ms.author: amverma
ms.service: virtual-machines
ms-subservice: hpc
ms.workload: infrastructure-services
ms.topic: overview
ms.date: 07/29/2020
ms.openlocfilehash: f9a672777fbc8a75cb6af10ba25232f8a274c894
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543731"
---
# <a name="high-performance-computing-on-infiniband-enabled-h-series-and-n-series-vms"></a>Informática de alto rendimiento en máquinas virtuales de las series H y N habilitadas con InfiniBand

Las máquinas virtuales de las series H y N habilitadas con InfiniBand de Azure están diseñadas para ofrecer rendimiento de primer nivel, escalabilidad de la interfaz de paso de mensajes (MPI) y rentabilidad para diversas cargas de trabajo de HPC e inteligencia artificial del mundo real. Estas máquinas virtuales optimizadas de informática de alto rendimiento (HPC) se usan para resolver algunos de los problemas de la ciencia e ingeniería que hacen un uso más intensivo de la informática, como la dinámica de fluidos, el modelado terrestre o las simulaciones meteorológicas.

En estos artículos se describen los primeros pasos con las máquinas virtuales de las series H y N habilitadas con InfiniBand en Azure, así como la configuración óptima de las cargas de trabajo de HPC e inteligencia artificial en dichas máquinas para ofrecer escalabilidad.

## <a name="features-and-capabilities"></a>Características y funcionalidades

Las máquinas virtuales de las series H y N habilitadas con InfiniBand están diseñadas para proporcionar el mejor rendimiento de HPC, escalabilidad de MPI y rentabilidad para cargas de trabajo de HPC. Consulte las máquinas virtuales de la [serie H](../../sizes-hpc.md) y de la [serie N](../../sizes-gpu.md) para más información sobre las características y funcionalidades de las máquinas virtuales.

### <a name="rdma-and-infiniband"></a>RDMA e InfiniBand

Las máquinas virtuales de [compatibles con RDMA](../../sizes-hpc.md#rdma-capable-instances) tanto de la [serie H](../../sizes-hpc.md) como de la [serie N](../../sizes-gpu.md) se comunican a través de la red InfiniBand de baja latencia y ancho de banda alto. La funcionalidad de RDMA sobre dicha interconexión es crítica para potenciar la escalabilidad y el rendimiento de las cargas de trabajo de inteligencia artificial y HPC de nodos distribuidos. Las máquinas virtuales de las series H y S habilitadas con InfiniBand se conectan en una estructura de árbol grueso sin bloqueos y con poco diámetro para lograr un rendimiento de RDMA coherente y optimizado.
Consulte [Habilitación de InfiniBand](enable-infiniband.md) para más información sobre la configuración de InfiniBand en las máquinas virtuales habilitadas con InfiniBand.

### <a name="message-passing-interface"></a>Interfaz de paso de mensajes

Tanto la serie H como la serie N, habilitadas con SR-IOV, admiten casi todas las bibliotecas y versiones de MPI. Estas son algunas de las bibliotecas de MPI más comunes que se admiten: Intel MPI, OpenMPI, MPICH, MVAPICH2, Platform MPI y todos los verbos de acceso directo a memoria remota (RDMA).
Para más información sobre la instalación de varias bibliotecas de MPI compatibles, así como sobre su configuración óptima, consulte el artículo [Configuración de la interfaz de paso de mensajes para HPC](setup-mpi.md).

## <a name="get-started"></a>Primeros pasos

El primer paso consiste en seleccionar el tipo de máquina virtual de la [serie H](../../sizes-hpc.md) y de la [serie N](../../sizes-gpu.md) óptima para la carga de trabajo en función de las especificaciones de la máquina virtual y de la [funcionalidad de RDMA](../../sizes-hpc.md#rdma-capable-instances).
El segundo consiste en configurar la máquina virtual mediante la habilitación de InfiniBand. Hay varios métodos para hacerlo, entre los que se incluye el uso de imágenes de máquina virtual optimizadas con controladores preparados; para más información, consulte [Optimización para Linux](configure.md) y [Habilitación de InfiniBand](enable-infiniband.md).
El tercer paso, para cargas de trabajo de nodos distribuidos, es crítico y consiste en elegir y configurar la interfaz de paso de mensajes. Para más información, consulte [Configuración de la interfaz de paso de mensajes para HPC](setup-mpi.md).
El cuarto, con el fin de mejorar el rendimiento y la escalabilidad, consiste en configurar las cargas de trabajo de manera óptima siguiendo las instrucciones específicas de la familia de máquinas virtuales, como por ejemplo [Introducción a las máquinas virtuales de la serie HB](hb-series-overview.md) e [Introducción a las máquinas virtuales de la serie HC](hc-series-overview.md).

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [configurar y optimizar](configure.md) las máquinas virtuales tanto de la [serie H](../../sizes-hpc.md) como de la [serie N](../../sizes-gpu.md) habilitadas con InfiniBand.
- En los artículos [Introducción a las máquinas virtuales de la serie HB](hb-series-overview.md) e [Introducción a las máquinas virtuales de la serie HC](hc-series-overview.md), aprenderá a configurar de forma óptima las cargas de trabajo para mejorar el rendimiento y la escalabilidad.
- En los [blogs de la comunidad de Azure Compute Tech](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute), encontrará los anuncios más recientes y algunos ejemplos y resultados de HPC.
- Si desea una visión general de la arquitectura de la ejecución de cargas de trabajo de HPC, consulte [Informática de alto rendimiento (HPC) en Azure](/azure/architecture/topics/high-performance-computing/).
