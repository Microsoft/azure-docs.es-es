---
title: 'Problemas conocidos con las máquinas virtuales de las series HB y HC: Azure Virtual Machines | Microsoft Docs'
description: Obtenga información sobre problemas conocidos con los tamaños de VM de la serie HB en Azure.
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
ms.openlocfilehash: 6316bcc91bb381facb4f77b2d8dbd8b22f9ed387
ms.sourcegitcommit: d18a59b2efff67934650f6ad3a2e1fe9f8269f21
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88660102"
---
# <a name="known-issues-with-h-series-and-n-series-vms"></a>Problemas conocidos con las máquinas virtuales de las series H y N

Este artículo proporciona los problemas y soluciones más comunes al usar máquinas virtuales de las [series H](../../sizes-hpc.md) y [N](../../sizes-gpu.md).

## <a name="dram-on-hb-series"></a>DRAM de la serie HB

Las máquinas virtuales de serie HB solo pueden exponer 228 GB de RAM para las máquinas virtuales invitadas en este momento. Esto es debido a una limitación conocida del hipervisor de Azure para evitar que las páginas se asignen a la DRAM local de CCX AMD (dominios NUMA) reservada para la máquina virtual invitada.

## <a name="accelerated-networking"></a>Redes aceleradas

Azure Accelerated Networking no está habilitado en este momento, pero lo estará a medida que progresemos en el período de la versión preliminar. Se notificará a los clientes cuando se admita esta característica.

## <a name="qp0-access-restriction"></a>Restricción de acceso qp0

Para evitar el acceso de hardware de bajo nivel que pueda provocar vulnerabilidades de seguridad, el par de cola 0 no es accesible para las VM invitadas. Esto solo debería afectar a las acciones normalmente asociadas con la administración de la NIC ConnectX-5 y a la ejecución de algunos diagnósticos de InfiniBand, como ibdiagnet, pero no a las aplicaciones de usuario final.

## <a name="gss-proxy"></a>Proxy GSS

Proxy GSS tiene un problema conocido en CentOS/RHEL 7.5 que se puede producir como una reducción en la capacidad de respuesta y en el rendimiento cuando se usa con NFS. Para mitigarlo, puede llevar a cabo:

```console
sed -i 's/GSS_USE_PROXY="yes"/GSS_USE_PROXY="no"/g' /etc/sysconfig/nfs
```

## <a name="cache-cleaning"></a>Una limpieza de la memoria caché

En los sistemas HPC, a menudo resulta útil limpiar la memoria cuando finaliza un trabajo antes de que al usuario siguiente se le asigne el mismo nodo. Después de ejecutar aplicaciones en Linux, es posible que la memoria disponible se reduzca mientras aumenta la memoria del búfer, a pesar de que no se está ejecutando ninguna aplicación.

![Captura de pantalla de símbolo del sistema](./media/known-issues/cache-cleaning-1.png)

Con `numactl -H` se mostrará con qué NUMAnodes se almacenan en memoria en el búfer (posiblemente, con todos). En Linux, los usuarios pueden limpiar las memorias caché de tres formas para devolver la memoria en búfer o en caché al estado "libre". Debe ser la raíz o tener permisos de sudo.

```console
echo 1 > /proc/sys/vm/drop_caches [frees page-cache]
echo 2 > /proc/sys/vm/drop_caches [frees slab objects e.g. dentries, inodes]
echo 3 > /proc/sys/vm/drop_caches [cleans page-cache and slab objects]
```

![Captura de pantalla de símbolo del sistema](./media/known-issues/cache-cleaning-2.png)

## <a name="kernel-warnings"></a>Advertencias de kernel

Es posible que vea los siguientes mensajes de advertencia de kernel al iniciar una máquina virtual de la serie HB en Linux.

```console
[  0.004000] WARNING: CPU: 4 PID: 0 at arch/x86/kernel/smpboot.c:376 topology_sane.isra.3+0x80/0x90
[  0.004000] sched: CPU #4's llc-sibling CPU #0 is not on the same node! [node: 1 != 0]. Ignoring dependency.
[  0.004000] Modules linked in:
[  0.004000] CPU: 4 PID: 0 Comm: swapper/4 Not tainted 3.10.0-957.el7.x86_64 #1
[  0.004000] Hardware name: Microsoft Corporation Virtual Machine/Virtual Machine, BIOS 090007 05/18/2018
[  0.004000] Call Trace:
[  0.004000] [<ffffffffb8361dc1>] dump_stack+0x19/0x1b
[  0.004000] [<ffffffffb7c97648>] __warn+0xd8/0x100
[  0.004000] [<ffffffffb7c976cf>] warn_slowpath_fmt+0x5f/0x80
[  0.004000] [<ffffffffb7c02b34>] ? calibrate_delay+0x3e4/0x8b0
[  0.004000] [<ffffffffb7c574c0>] topology_sane.isra.3+0x80/0x90
[  0.004000] [<ffffffffb7c57782>] set_cpu_sibling_map+0x172/0x5b0
[  0.004000] [<ffffffffb7c57ce1>] start_secondary+0x121/0x270
[  0.004000] [<ffffffffb7c000d5>] start_cpu+0x5/0x14
[  0.004000] ---[ end trace 73fc0e0825d4ca1f ]---
```

Puede pasarlos por alto. Esto es debido a una limitación conocida del hipervisor de Azure que se solucionará con el tiempo.


## <a name="infiniband-driver-installation-on-infiniband-enabled-n-series-vm-sizes"></a>Instalación del controlador InfiniBand en los tamaños de máquina virtual de la serie N habilitados para InfiniBand

NC24r_v3 y ND40r_v2 tienen habilitada la opción SR-IOV, mientras que NC24r y NC24r_v2, no. Encontrará algunos detalles sobre la bifurcación [aquí](../../sizes-hpc.md#rdma-capable-instances).
InfiniBand (IB) puede configurarse en los tamaños de máquina virtual habilitados para SR-IOV con los controladores OFED, mientras que los tamaños de máquina virtual que no son SR-IOV requieren de los controladores ND. Esta compatibilidad con IB está disponible de forma adecuada en las [VMI de CentOS-HPC](configure.md). Para Ubuntu, consulte [esta instrucción](https://techcommunity.microsoft.com/t5/azure-compute/configuring-infiniband-for-ubuntu-hpc-and-gpu-vms/ba-p/1221351) para instalar los controladores OFED y ND tal y como se describe en la [documentación](enable-infiniband.md#vm-images-with-infiniband-drivers).


## <a name="next-steps"></a>Pasos siguientes

- En los artículos [Introducción a las máquinas virtuales de la serie HB](hb-series-overview.md) e [Introducción a las máquinas virtuales de la serie HC](hc-series-overview.md), aprenderá a configurar de forma óptima las cargas de trabajo para mejorar el rendimiento y la escalabilidad.
- En los [blogs de la comunidad de Azure Compute Tech](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute), encontrará los anuncios más recientes y algunos ejemplos y resultados de HPC.
- Si desea una visión general de la arquitectura de la ejecución de cargas de trabajo de HPC, consulte [Informática de alto rendimiento (HPC) en Azure](/azure/architecture/topics/high-performance-computing/).
