---
title: 'Tamaños de las máquinas virtuales de Azure: HPC | Microsoft Docs'
description: Enumera los tamaños diferentes disponibles para las máquinas virtuales de informática de alto rendimiento en Azure. Se proporciona información sobre el número de unidades vCPU, discos de datos y NIC, así como sobre el rendimiento de almacenamiento y el ancho de banda de red para los tamaños de esta serie.
author: vermagit
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 08/01/2020
ms.author: amverma
ms.reviewer: jushiman
ms.openlocfilehash: d0344247ba4dfffee275782bc2db83142d34cff3
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87800252"
---
# <a name="high-performance-computing-vm-sizes"></a>Tamaños de VM para informática de alto rendimiento

La máquinas virtuales de la serie H de Azure están diseñada para ofrecer rendimiento de primer nivel, escalabilidad de MPI y rentabilidad para diversas cargas de trabajo de HPC del mundo real.

Las máquinas virtuales de la [serie HBv2](hbv2-series.md) cuentan con la característica Mellanox HDR InfiniBand a 200 GB/s, mientras que las máquinas virtuales de las series HB y HC cuentan con la característica MeliBox EDR InfiniBand a 100 GB/s. Cada uno de estos tipos de máquinas virtuales están conectados en un árbol FAT sin bloqueos para un rendimiento de RDMA optimizado y coherente. Las máquinas virtuales HBv2 admiten el enrutamiento adaptativo y el transporte conectado dinámico (DCT, además de los transportes estándar RC y UD). Estas características mejoran el rendimiento, la escalabilidad y la coherencia de las aplicaciones, y se recomienda totalmente su uso.

Las máquinas virtuales de la [serie HB](hb-series.md) están optimizadas para aplicaciones que funcionan con ancho de banda de la memoria, como la dinámica de fluidos, el análisis explícito de elementos finitos y los modelos de clima. Las máquinas virtuales HB cuentan con 60 núcleos de procesador AMD EPYC 7551, 4 GB de RAM por núcleo de CPU y no tienen hyperthreading. La plataforma AMD EPYC proporciona un ancho de banda de memoria de más de 260 GB/s.

Las máquinas virtuales de la [serie HC](hc-series.md) están optimizadas para aplicaciones basadas en cálculos intensivos, como el análisis implícito de elementos finitos, la dinámica molecular y la química computacional. Las máquinas virtuales HC cuentan con 44 núcleos de procesador Intel Xeon Platinum 8168, 8 GB de RAM por núcleo de CPU y no tienen hyperthreading. La plataforma Intel Xeon Platinum admite el rico ecosistema de herramientas de software de Intel, como la biblioteca Intel Math Kernel Library.

Las máquinas virtuales de la [serie H](h-series.md) están optimizadas para aplicaciones basadas en frecuencias altas de CPU o requisitos de gran cantidad de memoria por núcleo. Las máquinas virtuales de la serie H cuentan con 8 o 16 núcleos de procesador Intel Xeon E5 2667 v3, 7 o 14 GB de RAM por núcleo de CPU y no tienen hyperthreading. Las máquinas de la serie H cuentan con una interconexión Mellanox FDR InfiniBand de 56 Gb/s en una configuración de árbol de FAT sin bloqueo para proporcionar un rendimiento de RDMA consistente. Las máquinas virtuales de la serie H admiten Intel MPI 5.x y MS-MPI.

> [!NOTE]
> Está previsto que las VM A8-A11 se retiren en 3/2021. Para obtener más información, consulte la [guía de migración de HPC](https://azure.microsoft.com/resources/hpc-migration-guide/).

## <a name="rdma-capable-instances"></a>Instancias compatibles con RDMA

La mayoría de los tamaños de VM para HPC (HBv2, HB, HC, H16r, H16mr, A8 y A9) incluye una interfaz de red para la conectividad de acceso directo a memoria remota (RDMA). Los tamaños seleccionados de la [serie N](./nc-series.md) designados con la letra "r", (ND40rs_v2, ND24rs, NC24rs_v3, NC24rs_v2 y NC24r) también son compatibles con RDMA. Esta interfaz se agrega a la interfaz de red estándar de Azure disponible en los otros tamaños de máquina virtual.

Esta interfaz permite que las instancias compatibles con RDMA se comuniquen través de una red InfiniBand (IB), que funciona a velocidades HDR en la serie HBv2, a velocidades EDR en las series HB, HC y NDv2, a velocidades FDR en las series H16r y H16mr y en otras máquinas virtuales de la serie N compatibles con RDMA, y a velocidades QDR en máquinas virtuales A8 y A9. Estas funcionalidades RDMA pueden mejorar la escalabilidad y el rendimiento de determinadas aplicaciones de la Interfaz de paso de mensajes (MPI). Para más información acerca de la velocidad, consulte los detalles en las tablas de esta página.

> [!NOTE]
> En Azure HPC, hay dos clases de máquinas virtuales en función de si están habilitadas para SR-IOV de InfiniBand. Actualmente, las máquinas virtuales que tienen habilitada la opción de SR-IOV para InfiniBand son: HBv2, HB, HC, NCv3 y NDv2. El resto de las máquinas virtuales compatibles con InfiniBand no están habilitadas para SR-IOV actualmente.
> RDMA a través de IB se admite en todas las máquinas virtuales compatibles con RDMA.
> Solo se admite IP sobre IB en máquinas virtuales habilitadas para SR-IOV.

- **Sistema operativo**: Linux es muy compatible con las máquinas virtuales de HPC; habitualmente se usan distribuciones como CentOS, RHEL, Ubuntu y SUSE. En cuanto a la compatibilidad con Windows, en todas las máquinas virtuales de la serie HPC se admite Windows Server 2016 y versiones más recientes. También se admiten Windows Server 2012 R2 y Windows Server 2012 en las máquinas virtuales habilitadas que no son SR-IOV (H16r, H16mr, A8 t A9). Tenga en cuenta que [Windows Server 2012 R2 no se admite en HBv2 ni en otras máquinas virtuales con más de 64 núcleos (virtuales o físicos)](/windows-server/virtualization/hyper-v/supported-windows-guest-operating-systems-for-hyper-v-on-windows). Consulte [Imágenes de máquina virtual](./workloads/hpc/configure.md) para obtener una lista de imágenes de máquina virtual compatibles en Marketplace y cómo se pueden configurar de forma adecuada.

- **Controladores de InfiniBand y RDMA**: en las máquinas virtuales compatibles con InfiniBand, se necesitan los controladores adecuados para habilitar RDMA. En Linux, tanto en el caso de las máquinas virtuales compatibles con SR-IOV como las que no, las imágenes de máquinas virtuales de CentOS-HPC en Marketplace se configuran previamente con los controladores adecuados. Las imágenes de máquinas virtuales de Ubuntu se pueden configurar con los controladores adecuados mediante las [instrucciones que se indican aquí](https://techcommunity.microsoft.com/t5/azure-compute/configuring-infiniband-for-ubuntu-hpc-and-gpu-vms/ba-p/1221351). Consulte [Configuración y optimización de las máquinas virtuales para Linux OS](./workloads/hpc/configure.md) para obtener más información sobre las imágenes de sistema operativo Linux de máquinas virtuales listas para usar.

   En Linux, se puede usar la [extensión de máquina virtual InfiniBandDriverLinux](./extensions/hpc-compute-infiniband-linux.md) para instalar los controladores OFED de Mellanox y habilitar InfiniBand en las máquinas virtuales de serie H y serie N compatibles con SR-IOV. Obtenga más información sobre cómo habilitar InfiniBand en máquinas virtuales compatibles con RDMA en [Cargas de trabajo de HPC](./workloads/hpc/enable-infiniband.md).

   En Windows, la [extensión de máquina virtual InfiniBandDriverWindows](./extensions/hpc-compute-infiniband-windows.md) instala controladores Windows Network Direct (en máquinas virtuales sin SR-IOV) o controladores Mellanox OFED (en máquinas virtuales con SR-IOV) para la conectividad RDMA. En algunas implementaciones de las instancias A8 y A9, la extensión HpcVmDrivers se agrega automáticamente. Tenga en cuenta que la extensión de máquina virtual HpcVmDrivers está en desuso, por lo que no se actualizará.

   Para agregar la extensión de máquina virtual a una máquina virtual, puede usar los cmdlets de [Azure PowerShell](/powershell/azure/). Para obtener más información, consulte el artículo de [características y extensiones de las máquinas virtuales](./extensions/overview.md). También puede trabajar con las extensiones para las máquinas virtuales implementadas en el [modelo de implementación clásica](/previous-versions/azure/virtual-machines/windows/classic/agents-and-extensions-classic).

- **MPI**: los tamaños de máquina virtual habilitados para SR-IOV en Azure (HBv2, HB, HC, NCv3 y NDv2) permiten que se use prácticamente cualquier tipo de MPI con Mellanox OFED. En las máquinas virtuales que no están habilitadas para SR-IOV, las implementaciones de MPI compatibles usan la interfaz Microsoft Network Direct (ND) para comunicarse entre máquinas virtuales. Por lo tanto, solo se admiten las versiones de Microsoft MPI (MS-MPI) 2012 R2 o posterior e Intel MPI 5.x. Las versiones posteriores (2017 y 2018) de la biblioteca en tiempo de ejecución de MPI pueden ser o no compatibles con los controladores de Azure RDMA. Consulte [Configuración de MPI para HPC](./workloads/hpc/setup-mpi.md) para obtener más información sobre cómo configurar MPI en máquinas virtuales de HPC en Azure.

- **Espacio de direcciones de red RDMA** : la red RDMA en Azure reserva el espacio de direcciones 172.16.0.0/16. Para ejecutar aplicaciones MPI en instancias implementadas en una red virtual Azure, asegúrese de que el espacio de direcciones de la red virtual no se superpone a la red RDMA.

## <a name="cluster-configuration-options"></a>Opciones de configuración del clúster

Azure ofrece varias opciones para crear clústeres de máquinas virtuales de HPC de Windows que se pueden comunicar con la red RDMA, incluidos: 

- **Máquinas virtuales**: implemente las máquinas virtuales de HPC compatibles con RDMA en el mismo conjunto de escalado o de disponibilidad (cuando use el modelo de implementación de Azure Resource Manager). Si usa el modelo de implementación clásica, implemente las máquinas virtuales en el mismo servicio en la nube.

- **Conjuntos de escalado de máquinas virtuales**: en un conjunto de escalado de máquinas virtuales, asegúrese de limitar la implementación a un único grupo de selección de ubicación para la comunicación InfiniBand dentro del conjunto de escalado. Por ejemplo, en una plantilla de Resource Manager, establezca la propiedad `singlePlacementGroup` en `true`. Tenga en cuenta que el tamaño de conjunto de escalado máximo que se puede usar con la propiedad `singlePlacementGroup` en `true` está limitado a 100 máquinas virtuales de manera predeterminada. Si los requisitos de escalado de su trabajo de HPC son superiores a 100 máquinas virtuales en un único inquilino, puede solicitar un aumento. Para ello, [realice una solicitud de soporte técnico al cliente en línea](../azure-portal/supportability/how-to-create-azure-support-request.md) sin cargo alguno. El número máximo de máquinas virtuales en un único conjunto de escalado se puede aumentar hasta 300. Tenga en cuenta que al implementar máquinas virtuales con conjuntos de disponibilidad, el límite máximo es de 200 máquinas virtuales por conjunto de disponibilidad.

- **MPI entre las máquinas virtuales**: si la característica de RDMA (por ejemplo, para usar la comunicación de MPI) es necesaria entre las máquinas virtuales (VM), asegúrese de que las VM estén en el mismo conjunto de escalado de máquinas virtuales o conjunto de disponibilidad.

- **Azure CycleCloud**: cree un clúster de HPC en [Azure CycleCloud](/azure/cyclecloud/) para ejecutar trabajos MPI.

- **Azure Batch**: cree un grupo de [Azure Batch](../batch/index.yml) para ejecutar cargas de trabajo MPI. Para usar instancias de proceso intensivo para ejecutar aplicaciones MPI con Azure Batch, consulte [Uso de tareas de instancias múltiples para ejecutar aplicaciones de la Interfaz de paso de mensajes (MPI) en Azure Batch](../batch/batch-mpi.md).

- **Microsoft HPC Pack**: [HPC Pack](/powershell/high-performance-computing/overview) incluye un entorno de tiempo de ejecución para MS-MPI que usa la red RDMA de Azure cuando se implementa en máquinas virtuales Linux compatibles con RDMA. Para obtener ejemplos de implementación, consulte [Configuración de un clúster de RDMA de Linux con HPC Pack para ejecutar aplicaciones MPI](/powershell/high-performance-computing/hpcpack-linux-openfoam).

## <a name="deployment-considerations"></a>Consideraciones de la implementación

- **Suscripción de Azure**: para implementar más que algunas instancias de proceso intensivo, considere la posibilidad de usar una suscripción de pago por uso u otras opciones de compra. Si usa una [cuenta gratuita de Azure](https://azure.microsoft.com/free/), solo puede usar un número limitado de núcleos de proceso de Azure.

- **Disponibilidad y precios**: los tamaños de máquina virtual se ofrecen solo en el plan de tarifa estándar. Para ver la disponibilidad en las regiones de Azure, consulte [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/) .

- **Cuota de núcleos**: quizás tenga que aumentar la cuota de núcleos de su suscripción de Azure partiendo del valor predeterminado. La suscripción también podría limitar el número de núcleos que se pueden implementar en ciertas familias de tamaño de máquina virtual, como la serie H. Para solicitar un aumento de cuota, [abra una solicitud de soporte técnico al cliente en línea](../azure-portal/supportability/how-to-create-azure-support-request.md) sin cargo alguno. (Los límites predeterminados pueden variar según la categoría de suscripción).

  > [!NOTE]
  > Si tiene necesidades de capacidad a gran escala, póngase en contacto con el soporte técnico de Azure. Las cuotas de Azure son límites de crédito, no garantías de capacidad. Independientemente de la cuota, solamente se le cobrarán los núcleos que use.
  
- **Red virtual** : no se necesita una [red virtual](https://azure.microsoft.com/documentation/services/virtual-network/) de Azure para usar instancias de proceso intensivo. Sin embargo, para muchas implementaciones necesita al menos una red virtual de Azure basada en la nube o una conexión de sitio a sitio si necesita acceder a recursos locales. Si es necesario, cree una red virtual para implementar las instancias. No se admite la adición de máquinas virtuales de proceso intensivo a las redes virtuales de grupos de afinidad.

- **Cambio de tamaño**: debido a su hardware especializado, solo se puede cambiar el tamaño de las instancias de proceso intensivo dentro de la misma familia de tamaño (serie H o serie N). Por ejemplo, una máquina virtual de la serie H solo se puede cambiar de un tamaño de serie H a otro. Es posible que tenga que tener en cuenta las consideraciones adicionales sobre la compatibilidad del controlador de InfiniBand y los discos de NVMe para determinadas máquinas virtuales.


## <a name="other-sizes"></a>Otros tamaños

- [Uso general](sizes-general.md)
- [Proceso optimizado](sizes-compute.md)
- [Memoria optimizada](sizes-memory.md)
- [Almacenamiento optimizado](sizes-storage.md)
- [GPU optimizada](sizes-gpu.md)
- [Generaciones anteriores](sizes-previous-gen.md)

## <a name="next-steps"></a>Pasos siguientes

- Puede encontrar más información sobre la optimización de aplicaciones de HPC para Azure y algunos ejemplos en [Cargas de trabajo de HPC](./workloads/hpc/overview.md).
- En los [blogs de la comunidad de Azure Compute Tech](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute), encontrará los anuncios más recientes y algunos ejemplos y resultados de HPC.
- Si desea una visión general de la arquitectura de la ejecución de cargas de trabajo de HPC, consulte [Informática de alto rendimiento (HPC) en Azure](/azure/architecture/topics/high-performance-computing/).
