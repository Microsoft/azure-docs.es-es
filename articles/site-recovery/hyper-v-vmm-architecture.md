---
title: Recuperación ante desastres de Architecture-Hyper-V en un sitio secundario con Azure Site Recovery
description: En este artículo se proporciona información general de la arquitectura usada para recuperación ante desastres local a un sitio System Center VMM secundario con Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/12/2019
ms.author: raynew
ms.openlocfilehash: 703a6afdc12c8a9863ff0f480ec7a577ec31ef77
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87496005"
---
# <a name="architecture---hyper-v-replication-to-a-secondary-site"></a>Arquitectura de replicación de Hyper-V en un sitio secundario

En este artículo se describen los componentes y procesos que tienen lugar al replicar máquinas virtuales de Hyper-V locales en nubes de System Center Virtual Machine Manager (VMM) a un sitio VMM secundario utilizando el servicio [Azure Site Recovery](site-recovery-overview.md) en Azure Portal.
a

## <a name="architectural-components"></a>Componentes de la arquitectura

En la siguiente tabla y gráfico se proporciona una visión general de los componentes que se usaron para la replicación de Hyper-V en un sitio secundario.

**Componente** | **Requisito** | **Detalles**
--- | --- | ---
**Azure** | Suscripción de Azure | Cree un almacén de Recovery Services en la suscripción de Azure, para orquestar y administrar la replicación entre ubicaciones de VMM.
**Servidor VMM** | Necesita una ubicación principal de VMM y otra secundaria. | Se recomienda un servidor VMM en el sitio principal y otro en el sitio secundario.
**Servidor de Hyper-V** |  Uno o varios servidores host de Hyper-V en las nubes VMM principal y secundaria. | Los datos se replican entre los servidores host de Hyper-V principales y secundarios a través de la LAN o VPN mediante autenticación Kerberos o de certificado.  
**Máquinas virtuales de Hyper-V** | En el servidor de host de Hyper-V. | El servidor host de origen debe tener, como mínimo, una máquina virtual que se desee replicar.

**Arquitectura entre dos sitios locales**

![Diagrama que muestra la protección de instalaciones locales a instalaciones locales.](./media/hyper-v-vmm-architecture/arch-onprem-onprem.png)

## <a name="replication-process"></a>Proceso de replicación

1. Cuando se desencadena la replicación inicial, se toma una [instantánea de la máquina virtual de Hyper-V](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd560637(v=ws.10)).
2. Los discos duros virtuales de la máquina virtual se replican uno a uno en la ubicación secundaria.
3. Si se producen cambios en el disco con la replicación inicial en curso, el seguimiento de replicaciones de Réplica de Hyper-V realiza un seguimiento de esos cambios en los registros de replicación de Hyper-V (.hrl). Estos archivos de registro se encuentran en la misma carpeta que los discos. Cada disco tiene un archivo .hrl asociado que se envía a la ubicación secundaria. Los archivos de instantáneas y de registro consumen recursos de disco mientras la replicación inicial está en curso.
4. Cuando finaliza la replicación inicial, se elimina la instantánea de la máquina virtual y la replicación diferencial comienza.
5. Los cambios diferenciales en el registro del disco se sincronizan y se combinan en el disco primario.


## <a name="failover-and-failback-process"></a>Proceso de conmutación por error y conmutación por recuperación

- Puede conmutar por error una única máquina o crear planes de recuperación para organizar la conmutación por error de varias máquinas.
- Puede ejecutar una conmutación por error, planeada o no, entre sitios locales. Si ejecuta una conmutación por error planeada, las máquinas virtuales de origen se apagan para garantizar que no se pierdan datos.
    - Si realizó una conmutación por error no planeada a un sitio secundario, tras ella las máquinas de la ubicación secundaria no están protegidas.
    - Si ejecutó una conmutación por error planeada, tras ella las máquinas de la ubicación secundaria están protegidas.
- Después que se ejecute la conmutación por error, debe confirmarla para que se inicie. Para ello, acceda a la carga de trabajo desde la máquina virtual de Replica.
- Cuando la ubicación principal vuelva a estar disponible, puede realizar la conmutación por recuperación.
    - Inicia la replicación inversa para que comience a replicar desde el sitio secundario en el principal. La replicación inversa pone las máquinas virtuales en un estado protegido pero el centro de datos secundario sigue siendo la ubicación activa.
    - Para que el sitio principal se convierta de nuevo en la ubicación activa, debe iniciar una conmutación por error planeada desde el secundario al principal, seguida de otra replicación inversa.



## <a name="next-steps"></a>Pasos siguientes


Siga [este tutorial](hyper-v-vmm-disaster-recovery.md) para habilitar la replicación de Hyper-V entre nubes de VMM.
