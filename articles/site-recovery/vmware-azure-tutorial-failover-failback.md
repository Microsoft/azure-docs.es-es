---
title: Conmutación por error de máquinas virtuales de VMware a Azure con Site Recovery
description: Aprenda a conmutar por error máquinas virtuales de VMware a Azure en Azure Site Recovery.
ms.service: site-recovery
ms.topic: tutorial
ms.date: 12/16/2019
ms.custom: MVC
ms.openlocfilehash: bf47f08ac555cf60f59ba2b1a84750b6a9e2e0a1
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "86132005"
---
# <a name="fail-over--vmware-vms"></a>Conmutación por error de máquinas virtuales de VMware

En este artículo se describe cómo conmutar por error una máquina virtual (VM) local de VMware a Azure con [Azure Site Recovery](site-recovery-overview.md).

Este es el quinto tutorial de una serie que muestra cómo configurar la recuperación ante desastres en Azure para máquinas locales.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Comprobar que las propiedades de la máquina virtual de VMware cumplen los requisitos de Azure
> * Conmutar por error máquinas virtuales específicas a Azure.

> [!NOTE]
> Los tutoriales muestran la ruta de implementación más sencilla para un escenario. Usan opciones predeterminadas siempre que es posible y no muestran todos los valores y rutas de acceso posibles. Si quiere conocer en detalle la conmutación por error, consulte [Conmutación por error de servidores físicos y máquinas virtuales de VMware](site-recovery-failover.md).

[Más información](failover-failback-overview.md#types-of-failover) sobre los diferentes tipos de conmutación por error. Si quiere conmutar por error varias máquinas virtuales de un plan de recuperación, revise [este artículo](site-recovery-failover.md).

## <a name="before-you-start"></a>Antes de comenzar

Complete los tutoriales anteriores:

1. Asegúrese de que ha [configurado Azure](tutorial-prepare-azure.md) para la recuperación ante desastres local tanto de máquinas virtuales de VMware como de máquinas virtuales de Hyper-V y de equipos físicos a Azure.
2. Prepare el entorno de [VMware](vmware-azure-tutorial-prepare-on-premises.md) local para la recuperación ante desastres. 
3. Configure la recuperación ante desastres para [máquinas virtuales de VMware](vmware-azure-tutorial.md).
4. Realice una [exploración en profundidad de la recuperación ante desastres](tutorial-dr-drill-azure.md) para asegurarse de que todo funciona según lo previsto.

## <a name="verify-vm-properties"></a>Comprobar las propiedades de la máquina virtual

Antes de ejecutar una conmutación por error, compruebe las propiedades de las máquinas virtuales para asegurarse de que cumplen los [requisitos de Azure](vmware-physical-azure-support-matrix.md#replicated-machines).

Compruebe que son estas las propiedades:

1. En **Elementos protegidos**, seleccione **Elementos replicados** y, luego, seleccione la máquina virtual que quiere comprobar.

2. En el panel **Elemento replicado**, puede ver un resumen de la información de la máquina virtual, el estado de mantenimiento y los puntos de recuperación disponibles más recientes. Seleccione **Propiedades** para ver más detalles.

3. En **Proceso y red**, estas propiedades se pueden modificar según sea necesario:
    * Nombre de Azure
    * Resource group
    * Tamaño de destino
    * [El conjunto de disponibilidad](../virtual-machines/windows/tutorial-availability-sets.md)
    * Configuración de discos administrados

4. Puede ver y modificar la configuración de red, por ejemplo:

    * La red y la subred en la que se ubicará la máquina virtual de Azure después de la conmutación por error.
    * La dirección IP que se le asignará.

5. En **Discos** puede ver información sobre los discos de datos y el sistema operativo de la máquina virtual.

## <a name="run-a-failover-to-azure"></a>Ejecutar una conmutación por error en Azure.

1. En **Configuración** > **Elementos replicados**, seleccione la máquina virtual que quiere conmutar por error y, luego, seleccione **Conmutar por error**.
2. En **Conmutación por error**, seleccione un **Punto de recuperación** en el que realizar la conmutación por error. Puede seleccionar una de las siguientes opciones:
   * **Más reciente**: esta opción procesa primero todos los datos enviados a Site Recovery. Proporciona el objetivo de punto de recuperación (RPO) mínimo, ya que la máquina virtual de Azure creada después de la conmutación por error tiene todos los datos que se replicaron en Site Recovery al desencadenarse la conmutación por error.
   * **Procesado más recientemente**: con esta opción se realiza una conmutación por error de la máquina virtual al último punto de recuperación procesado por Site Recovery. Esta opción proporciona un objetivo de tiempo de recuperación (RTO) bajo, ya que no se invierte tiempo en el procesamiento de datos sin procesar.
   * **Más reciente coherente con la aplicación**: con esta opción se realiza una conmutación por error de la máquina virtual al punto de recuperación más reciente coherente con la aplicación que procesó Site Recovery.
   * **Personalizado**: esta opción permite especificar un punto de recuperación.

3. Seleccione **Apague la máquina antes de comenzar con la conmutación por error.** para intentar apagar las máquinas virtuales de origen antes de desencadenar la conmutación por error. La conmutación por error continúa aunque se produzca un error de cierre. Puede seguir el progreso de la conmutación por error en la página **Trabajos**.

En algunos escenarios, la conmutación por error requiere un procesamiento adicional que tarda aproximadamente de ocho a diez minutos en completarse. Puede observar que la conmutación por error de prueba tarda más tiempo en realizarse:

* Máquinas virtuales de VMware que ejecutan una versión de Mobility Service anterior a la 9.8
* Servidores físicos
* Máquinas virtuales de VMware Linux
* Máquinas virtuales de Hyper-V protegidas como servidores físicos
* Máquinas virtuales de VMware que no tienen habilitado el servicio DHCP
* Máquinas virtuales de VMware que no tienen los siguientes controladores de arranque: storvsc, vmbus, storflt, intelide o atapi.

> [!WARNING]
> No cancele una conmutación por error en curso. Antes de iniciar la conmutación por error, se detiene la replicación de la máquina virtual. Si se cancela una conmutación por error en curso, la conmutación por error se detiene, pero no se replica la máquina virtual de nuevo.

## <a name="connect-to-failed-over-vm"></a>Conexión a la máquina virtual conmutada por error

1. Si quiere conectarse a las máquinas virtuales de Azure después de la conmutación por error mediante el Protocolo de escritorio remoto (RDP) y Secure Shell (SSH), [compruebe que se han cumplido los requisitos](failover-failback-overview.md#connect-to-azure-after-failover).
2. Después de la conmutación por error, vaya a la máquina virtual y [conéctese](../virtual-machines/windows/connect-logon.md) a ella para realizar la validación.
3. Use **Cambiar punto de recuperación** si desea usar otro punto de recuperación después de la conmutación por error. Después de confirmar la conmutación por error en el paso siguiente, esta opción dejará de estar disponible.
4. Tras la validación, seleccione **Confirmar** para finalizar el punto de recuperación de la máquina virtual después de la conmutación por error.
5. Tras la confirmación, los demás puntos de recuperación disponibles se eliminan. Este paso finaliza la conmutación por error.

>[!TIP]
> Si tiene algún problema de conectividad después de la conmutación por error, siga la [guía para la solución de problemas](site-recovery-failover-to-azure-troubleshoot.md).

## <a name="next-steps"></a>Pasos siguientes

Después de la conmutación por error, vuelva a proteger las máquinas virtuales de Azure en el entorno local. Después de que las máquinas virtuales estén protegidas y replicando en el sitio local, realice una conmutación por recuperación desde Azure cuando esté listo.

> [!div class="nextstepaction"]
> [Volver a proteger máquinas virtuales de Azure](vmware-azure-reprotect.md)
> [Conmutación por recuperación desde Azure](vmware-azure-failback.md)
