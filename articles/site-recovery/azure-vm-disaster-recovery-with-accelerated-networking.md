---
title: Habilitación de redes aceleradas para la recuperación ante desastres de las máquinas virtuales de Azure con Azure Site Recovery
description: Descripción de la habilitación de las redes aceleradas con Azure Site Recovery para la recuperación ante desastres de máquinas virtuales de Azure
services: site-recovery
documentationcenter: ''
author: Harsha-CS
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 04/08/2019
ms.author: harshacs
ms.openlocfilehash: 1d2d3b3aacc00428c96cde0f8230421a98151ae2
ms.sourcegitcommit: 1fe5127fb5c3f43761f479078251242ae5688386
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/14/2020
ms.locfileid: "90068020"
---
# <a name="accelerated-networking-with-azure-virtual-machine-disaster-recovery"></a>Redes aceleradas con recuperación ante desastres de máquinas virtuales de Azure

Las redes aceleradas habilitan la virtualización de E/S de raíz única (SR-IOV) en una máquina virtual, lo que mejora considerablemente su rendimiento en la red. Este método de alto rendimiento omite el host de la ruta de acceso de datos, lo que reduce la latencia, la inestabilidad y la utilización de la CPU para usarse con las cargas de trabajo de red más exigentes en los tipos de máquina virtual admitidos. En la siguiente imagen, se muestra la comunicación entre dos máquinas virtuales (VM) con y sin Accelerated Networking:

![De comparación](./media/azure-vm-disaster-recovery-with-accelerated-networking/accelerated-networking-benefit.png)

Azure Site Recovery le permite usar las ventajas de las redes aceleradas para las máquinas virtuales que se conmutan por error a otra región de Azure. En este artículo se describe la habilitación de las redes aceleradas para las máquinas virtuales de Azure replicadas con Azure Site Recovery.

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar, asegúrese de que comprende:
-   La [arquitectura de replicación](azure-to-azure-architecture.md) de máquinas virtuales de Azure
-   La [configuración de replicación](azure-to-azure-tutorial-enable-replication.md) de máquinas virtuales de Azure
-   La [conmutación por error](azure-to-azure-tutorial-failover-failback.md) de máquinas virtuales de Azure

## <a name="accelerated-networking-with-windows-vms"></a>Redes aceleradas con máquinas virtuales Windows

Azure Site Recovery admite la habilitación de las redes aceleradas para las máquinas virtuales replicadas únicamente si la de origen las tiene habilitadas. Si la máquina virtual de origen no tiene las redes aceleradas habilitadas, puede aprender a habilitarlas en las máquinas virtuales Windows [aquí](../virtual-network/create-vm-accelerated-networking-powershell.md#enable-accelerated-networking-on-existing-vms).

### <a name="supported-operating-systems"></a>Sistemas operativos admitidos
Se admiten las siguientes distribuciones de fábrica desde la galería de Azure:
* **Windows Server 2016 Datacenter**
* **Windows Server 2012 R2 Datacenter**

### <a name="supported-vm-instances"></a>Instancias de máquina virtual admitidas
Accelerated Networking se admite con la mayoría de los tamaños de instancia de uso general y optimizados para procesos de dos o más vCPU.  Estas series admitidas son: D/DSv2 y F/Fs

En instancias que admiten hyperthreading, las redes aceleradas se admiten en instancias de máquina virtual con cuatro o más vCPU. Las series admitidas son: D/DSv3, E/ESv3, Fsv2 y Ms/Mms

Para más información sobre las instancias de máquinas virtuales, consulte [Tamaños de las máquinas virtuales con Windows](../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

## <a name="accelerated-networking-with-linux-vms"></a>Redes aceleradas con máquinas virtuales Linux

Azure Site Recovery admite la habilitación de las redes aceleradas para las máquinas virtuales replicadas únicamente si la de origen las tiene habilitadas. Si la máquina virtual de origen no tiene las redes aceleradas habilitadas, puede aprender a habilitarlas en las máquinas virtuales Linux [aquí](../virtual-network/create-vm-accelerated-networking-cli.md#enable-accelerated-networking-on-existing-vms).

### <a name="supported-operating-systems"></a>Sistemas operativos admitidos
Se admiten las siguientes distribuciones de fábrica desde la galería de Azure:
* **Ubuntu 16.04**
* **SLES 12 SP3**
* **RHEL 7.4**
* **CentOS 7.4**
* **CoreOS Linux**
* **Debian "Stretch" con kernel backports**
* **Oracle Linux 7.4**

### <a name="supported-vm-instances"></a>Instancias de máquina virtual admitidas
Accelerated Networking se admite con la mayoría de los tamaños de instancia de uso general y optimizados para procesos de dos o más vCPU.  Estas series admitidas son: D/DSv2 y F/Fs

En instancias que admiten hyperthreading, las redes aceleradas se admiten en instancias de máquina virtual con cuatro o más vCPU. Las series admitidas son: D/DSv3, E/ESv3, Fsv2 y Ms/Mms.

Para más información sobre las instancias de máquinas virtuales, consulte [Tamaños de las máquinas virtuales Linux](../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

## <a name="enabling-accelerated-networking-for-replicated-vms"></a>Habilitación de las redes aceleradas en máquinas virtuales replicadas

Cuando se [habilita la replicación](azure-to-azure-tutorial-enable-replication.md) para las máquinas virtuales de Azure, Site Recovery detecta automáticamente si las interfaces de red de máquina virtual tienen las redes aceleradas habilitadas. Si lo están, Site Recovery las configura automáticamente en las interfaces de red de la máquina virtual replicada.

Se puede comprobar el estado de las redes aceleradas en la sección **Interfaces de red** de la configuración **Compute and Network** (Proceso y red) de la máquina virtual replicada.

![Configuración de las redes aceleradas](./media/azure-vm-disaster-recovery-with-accelerated-networking/compute-network-accelerated-networking.png)

Si ha habilitado las redes aceleradas en la máquina virtual de origen después de habilitar la replicación, puede habilitarlas también en las interfaces de red de la máquina virtual replicada mediante el siguiente proceso:
1. Abra la configuración **Compute and Network** (Proceso y red) de la máquina virtual replicada
2. Haga clic en el nombre de la interfaz de red en la sección **Interfaces de red**
3. Seleccione **Habilitado** en la lista desplegable para las redes aceleradas en la columna **Destino**

![Habilitar Accelerated Networking](./media/azure-vm-disaster-recovery-with-accelerated-networking/network-interface-accelerated-networking-enabled.png)

También debe seguir el proceso anterior para las máquinas virtuales replicadas existentes que no tuvieran las redes aceleradas habilitadas automáticamente mediante Site Recovery.

## <a name="next-steps"></a>Pasos siguientes
- Más información sobre las [ventajas de las redes aceleradas](../virtual-network/create-vm-accelerated-networking-powershell.md#benefits).
- Más información sobre las limitaciones y las restricciones de las redes aceleradas en las [máquinas virtuales Windows](../virtual-network/create-vm-accelerated-networking-powershell.md#limitations-and-constraints) y las [máquinas virtuales Linux](../virtual-network/create-vm-accelerated-networking-cli.md#limitations-and-constraints).
- Más información sobre los [planes de recuperación](site-recovery-create-recovery-plans.md) para automatizar la conmutación por error de las aplicaciones.