---
title: Configuración de la conmutación por error de máquinas virtuales de Hyper-V a Azure en Azure Site Recovery
description: Aprenda a conmutar por error máquinas virtuales de Hyper-V a Azure con Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 12/16/2019
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: b46a2ea12a697afde8223cc3595365c1286512c7
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "86132463"
---
# <a name="fail-over-hyper-v-vms-to-azure"></a>Conmutación por error de máquinas virtuales de Hyper-V a Azure

En este tutorial se describe cómo conmutar por error máquinas virtuales de Hyper-V a Azure con [Azure Site Recovery](site-recovery-overview.md). Después de que ha conmutado por error, conmutará por recuperación a su sitio local cuando esté disponible. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Comprobar las propiedades de máquinas virtuales de Hyper-V para confirmar que se cumplen los requisitos de Azure.
> * Conmutar por error máquinas virtuales específicas a Azure.


Este es el quinto tutorial de su serie. En él se da por hecho que ya ha realizado las tareas de los tutoriales anteriores.    

1. [Preparación de Azure](tutorial-prepare-azure.md)
2. [Preparación de un entorno de Hyper-V local](./hyper-v-prepare-on-premises-tutorial.md)
3. Configuración de la recuperación ante desastres de [máquinas virtuales de Hyper-V](./hyper-v-azure-tutorial.md) o de [máquinas virtuales de Hyper-V administradas en nubes VMM de System Center](./hyper-v-vmm-azure-tutorial.md)
4. [Exploración de la recuperación ante desastres](tutorial-dr-drill-azure.md)

[Más información](failover-failback-overview.md#types-of-failover) sobre los diferentes tipos de conmutación por error. Si quiere conmutar por error varias máquinas virtuales de un plan de recuperación, revise [este artículo](site-recovery-failover.md).

## <a name="prepare-for-failover"></a>Preparación para la conmutación por error 
Asegúrese de que no hay instantáneas en la máquina virtual y de que la máquina virtual local se ha desactivado durante la conmutación por recuperación. Esto ayuda a garantizar la coherencia de datos durante la replicación. No encienda la máquina virtual local durante la conmutación por recuperación. 

La conmutación por error y la conmutación por recuperación constan de tres etapas:

1. **Conmutación por error a Azure**: conmutar por error máquinas virtuales de Hyper-V desde el sitio local a Azure.
2. **Conmutación por recuperación a un sitio local**: conmutar por error máquinas virtuales de Azure al sitio local, cuando esté disponible. Inicia la sincronización de datos de Azure en el entorno local y, al finalizar, muestra las máquinas virtuales en el entorno local.  
3. **Replicación inversa en máquinas virtuales locales**: después de la conmutación por recuperación a un entorno local, realice replicación inversa en las máquinas virtuales locales para que comiencen a replicarse en Azure.

## <a name="verify-vm-properties"></a>Comprobar las propiedades de la máquina virtual

Antes de la conmutación por error, compruebe las propiedades de la máquina virtual y asegúrese de que cumple los [requisitos de Azure](hyper-v-azure-support-matrix.md#replicated-vms).

En **Elementos protegidos**, haga clic en **Elementos replicados** > VM.

1. En el panel **Elemento replicado**, puede ver un resumen de la información de la máquina virtual, el estado de mantenimiento y los puntos de recuperación disponibles más recientes. Haga clic en **Propiedades** para ver más detalles.

1. En **Proceso y red**, puede modificar el nombre de Azure, el grupo de recursos, el tamaño de destino, el [conjunto de disponibilidad](../virtual-machines/windows/tutorial-availability-sets.md) y la configuración de discos administrados.

1. Puede ver y modificar la configuración de red, incluida la red o subred en la que se va a ubicar la máquina virtual de Azure después de la conmutación por error y la dirección IP que se le va a asignar.

1. En **Discos** puede ver información sobre los discos de datos y el sistema operativo de la máquina virtual.

## <a name="fail-over-to-azure"></a>Conmutación por error a Azure

1. En **Configuración** > **Elementos replicados**, haga clic en VM > **Conmutación por error**.
2. En **Conmutación por error**, seleccione el punto de recuperación **más reciente**. 
3. Seleccione **Apague la máquina antes de comenzar con la conmutación por error**. Site Recovery intentará apagar las máquinas virtuales de origen antes de desencadenar la conmutación por error. La conmutación por error continúa aunque se produzca un error de cierre. Puede seguir el progreso de la conmutación por error en la página **Trabajos**.
4. Una vez verificada la conmutación por error, haga clic en **Confirmar**. Así se eliminan los puntos de recuperación disponibles.

> [!WARNING]
> **No cancele una conmutación por error en curso**: si cancela una conmutación por error en curso, esta se detiene, pero la máquina virtual no se volverá a replicar.

## <a name="connect-to-failed-over-vm"></a>Conexión a la máquina virtual conmutada por error

1. Si quiere conectarse a las máquinas virtuales de Azure después de la conmutación por error mediante el Protocolo de escritorio remoto (RDP) y Secure Shell (SSH), [compruebe que se han cumplido los requisitos](failover-failback-overview.md#connect-to-azure-after-failover).
2. Después de la conmutación por error, vaya a la máquina virtual y [conéctese](../virtual-machines/windows/connect-logon.md) a ella para realizar la validación.
3. Use **Cambiar punto de recuperación** si desea usar otro punto de recuperación después de la conmutación por error. Después de confirmar la conmutación por error en el paso siguiente, esta opción dejará de estar disponible.
4. Tras la validación, seleccione **Confirmar** para finalizar el punto de recuperación de la máquina virtual después de la conmutación por error.
5. Tras la confirmación, los demás puntos de recuperación disponibles se eliminan. Este paso finaliza la conmutación por error.

>[!TIP]
> Si tiene algún problema de conectividad después de la conmutación por error, siga la [guía para la solución de problemas](site-recovery-failover-to-azure-troubleshoot.md).


## <a name="next-steps"></a>Pasos siguientes

Tras la conmutación por error, vuelva a proteger las máquinas virtuales de Azure de modo que se repliquen de Azure al entorno local. Después de que las máquinas virtuales estén protegidas y replicando en el sitio local, realice una conmutación por recuperación desde Azure cuando esté listo.
