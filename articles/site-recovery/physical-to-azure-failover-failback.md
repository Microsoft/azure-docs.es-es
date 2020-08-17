---
title: Configuración de la conmutación por error y la conmutación por recuperación de servidores físicos con Site Recovery
description: Obtenga información sobre cómo realizar una conmutación por error de servidores físicos en Azure y una conmutación por recuperación en el sitio local para la recuperación ante desastres con Azure Site Recovery.
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: article
ms.date: 12/17/2019
ms.author: raynew
ms.openlocfilehash: 2994f68e4159c7c4aa7d82bef7a5891deb5055a0
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87292820"
---
# <a name="fail-over-and-fail-back-physical-servers-replicated-to-azure"></a>Conmutación por error y conmutación por recuperación de servidores físicos replicados en Azure

En este tutorial se describe cómo conmutar por error servidores físicos locales que replican a Azure con [Azure Site Recovery](site-recovery-overview.md). Una vez realizada la conmutación por error, se realizará la conmutación por recuperación a su sitio local cuando esté disponible.

## <a name="before-you-start"></a>Antes de comenzar

- [Obtenga información](failover-failback-overview.md) sobre el proceso de conmutación por error en la recuperación ante desastres.
- Si desea conmutar por error varias máquinas, [aprenda](recovery-plan-overview.md) a juntar todas ellas en un plan de recuperación.
- Antes de realizar una conmutación por error completa, realice una [exploración de la recuperación ante desastres](site-recovery-test-failover-to-azure.md) para asegurarse de que todo funciona según lo previsto.
- Siga [estas instrucciones](site-recovery-failover.md#prepare-to-connect-after-failover) para preparar la conexión con las máquinas virtuales de Azure después de la conmutación por error.



## <a name="run-a-failover"></a>Ejecución de la conmutación por error

### <a name="verify-server-properties"></a>Comprobar lar propiedades del servidor

Compruebe las propiedades del servidor y asegúrese de que cumpla con los [requisitos de Azure](vmware-physical-azure-support-matrix.md#replicated-machines) de las máquinas virtuales de Azure.

1. En **Elementos protegidos**, haga clic en **Elementos replicados** y seleccione la máquina.
2. En el panel **Elemento replicado**, puede ver un resumen de la información del equipo, el estado de mantenimiento y los puntos de recuperación disponibles más recientes. Haga clic en **Propiedades** para ver más detalles.
3. En **Proceso y red**, puede modificar el nombre de Azure, el grupo de recursos, el tamaño de destino, el [conjunto de disponibilidad](../virtual-machines/windows/tutorial-availability-sets.md) y la configuración de discos administrados
4. Puede ver y modificar la configuración de red, incluida la red o subred en la que se va a ubicar la máquina virtual de Azure después de la conmutación por error y la dirección IP que se le va a asignar.
5. En **Discos** puede ver información sobre los discos de datos y el sistema operativo del equipo.

### <a name="fail-over-to-azure"></a>Conmutación por error a Azure

1. En **Configuración** > **Elementos replicados**, haga clic en la máquina > **Conmutación por error**.
2. En **Conmutación por error**, seleccione un **punto de recuperación** en el que realizar la conmutación por error. Puede seleccionar una de las siguientes opciones:
   - **Último**: esta opción procesa primero todos los datos enviados a Site Recovery. Ofrece el objetivo de punto de recuperación (RPO) mínimo, ya que la máquina virtual de Azure creada después de la conmutación por error tiene todos los datos replicados en Site Recovery al desencadenarse la conmutación por error.
   - **Procesado más recientemente**: con esta opción se realiza una conmutación por error de la máquina al último punto de recuperación que procesó Site Recovery. Esta opción proporciona un objetivo de tiempo de recuperación (RTO) bajo, ya que no se invierte tiempo en el procesamiento de datos sin procesar.
   - **Más reciente coherente con la aplicación**: esta opción realiza una conmutación por error de la máquina al punto de recuperación más reciente coherente con la aplicación que haya procesado Site Recovery.
   - **Personalizado**: especifique un punto de recuperación.

3. Seleccione **Apague la máquina antes de comenzar con la conmutación por error** si desea que Site Recovery intente apagar la máquina de origen antes de desencadenar la conmutación por error. La conmutación por error continúa aunque se produzca un error de cierre. Puede seguir el progreso de la conmutación por error en la página **Trabajos**.
4. Si está preparado para conectarse a la máquina virtual de Azure, hágalo para validarla después de la conmutación por error.
5. Después de la comprobación, **confirme** la conmutación por error. Esta acción elimina todos los puntos de recuperación disponibles.

> [!WARNING]
> No cancele una conmutación por error en curso. Antes de que comience la conmutación por error, se detendrá la replicación de la máquina. Si cancela la conmutación por error esta se detiene, pero no replica de nuevo la máquina.
> En cuanto a los servidores físicos, el proceso de conmutación por error adicional puede tardar de ocho a diez minutos en completarse.

## <a name="automate-actions-during-failover"></a>Automatización de acciones durante la conmutación por error

Es posible que desee automatizar acciones durante la conmutación por error. Para ello, puede utilizar scripts o runbooks de Azure Automation en planes de recuperación.

- [Aprenda](site-recovery-create-recovery-plans.md) a crear y personalizar planes de recuperación, lo que incluye la incorporación de scripts.
- [Aprenda](site-recovery-runbook-automation.md) a agregar runbooks de Azure Automation a los planes de recuperación.

## <a name="configure-settings-after-failover"></a>Configuración tras la conmutación por error

Después de la conmutación por error debe [configurar Azure](site-recovery-failover.md#prepare-in-azure-to-connect-after-failover) para conectarse a las máquinas virtuales de Azure replicadas. Además, debe configurar tanto las direcciones IP [públicas como las internas](site-recovery-failover.md#set-up-ip-addressing).

## <a name="prepare-for-reprotection-and-failback"></a>Preparación para la reprotección y la conmutación por recuperación

Después de realizar la conmutación por error a Azure, vuelva a proteger las máquinas virtuales de Azure, para lo que debe replicarlas en el sitio local. Luego, después de que se hayan replicado, puede conmutarlas por recuperación al entorno local, para lo que debe realizar una conmutación por error desde Azure al sitio local.

1. Los servidores físicos replicados en Azure mediante Site Recovery solo pueden realizar conmutaciones por recuperación como máquinas virtuales de VMware. Esto significa que necesita una infraestructura de VMware para realizar conmutaciones por recuperación. Siga los pasos descritos en [este artículo](vmware-azure-prepare-failback.md) para prepararse para la reprotección y la conmutación por recuperación. Esto incluye la configuración de un servidor de procesos en Azure y un servidor de destino maestro local, así como la configuración de una VPN de sitio a sitio, o del emparejamiento privado de ExpressRoute para la conmutación por recuperación.
2. Asegúrese de que el servidor de configuración local está en ejecución y conectado a Azure. Durante la conmutación por error a Azure, es posible que no se pueda acceder al sitio local y el servidor de configuración puede estar no disponible o apagado. Durante la conmutación por recuperación, la VM debe encontrarse en la base de datos del servidor de configuración. En caso contrario, la conmutación por recuperación no será correcta.
3. Elimine todas las instantáneas en el servidor de destino maestro local. La reprotección no funcionará si existe alguna instantánea.  Las instantáneas de la VM se combinan automáticamente durante los trabajos de reprotección.
4. Si va a reproteger máquinas virtuales agrupadas en un grupo de replicación para asegurar la coherencia de varias máquinas virtuales, asegúrese de que todas tienen el mismo sistema operativo (Windows o Linux) y de que el servidor de destino maestro que va a implementar tiene el mismo tipo de sistema operativo. Todas las máquinas virtuales de un grupo de replicación deben usar el mismo servidor de destino maestro.
5. Abra [los puertos necesarios](vmware-azure-prepare-failback.md#ports-for-reprotectionfailback) para realizar la conmutación por recuperación.
6. Asegúrese de que vCenter Server esté conectado antes de realizar la conmutación por recuperación. En caso contrario, se producirá un error al desconectar los discos y conectarlos a la máquina virtual.
7. Si una instancia de vCenter Server administra las máquinas virtuales a las que quiere conmutar por recuperación, asegúrese de tener los permisos necesarios. Si realiza la detección de usuarios de solo lectura de vCenter y protege las máquinas virtuales, la protección se ejecutará correctamente y la conmutación por error funcionará. Sin embargo, durante la reprotección, se produce un error de conmutación por error porque no se pueden detectar los almacenes de datos y no aparecen durante la reprotección. Para solucionar este problema, puede actualizar las credenciales de vCenter con la [cuenta o los permisos adecuados](vmware-azure-tutorial-prepare-on-premises.md#prepare-an-account-for-automatic-discovery) y volver a intentar el trabajo. 
8. Si ha usado una plantilla para crear las máquinas virtuales, asegúrese de que cada una tenga su propio UUID para los discos. Si el UUID de la VM local entra en conflicto con el UUID del servidor de destino maestro porque ambos se crearon a partir de la misma plantilla, se produce un error de reprotección. En este caso, implemente desde una plantilla diferente.
9. Si va a conmutar por recuperación en una instancia de vCenter Server alternativa, asegúrese de que se detectan tanto el nuevo vCenter Server como el servidor de destino maestro. Normalmente, si no se detectaron, significa que los almacenes de datos no son accesibles o visibles en **Reproteger**.
10. Compruebe los siguientes escenarios en los que no se puede realizar la conmutación por recuperación:
    - Si utiliza las ediciones gratuitas de ESXi 5.5 o vSphere Hypervisor 6, actualice a una versión diferente.
    - Si tiene un servidor físico de Windows Server 2008 R2 SP1.
    - Máquinas virtuales que han migrado.
    - Un a máquina virtual que se ha migrado a otro grupo de recursos.
    - Una máquina virtual de Azure de réplica que se ha eliminado.
    - Una máquina virtual de Azure de réplica que no está protegida (se está replicando en el sitio local).
10. [Revise los tipos de conmutación por recuperación](concepts-types-of-failback.md) que puede usar: recuperación de ubicación original y recuperación de ubicación alternativa.


## <a name="reprotect-azure-vms-to-an-alternate-location"></a>Reprotección de máquinas virtuales de Azure en otra ubicación

En este procedimiento se da por supuesto que la máquina virtual local no está disponible.

1. En el almacén > **Configuración** > **Elementos replicados**, haga clic con el botón derecho en la máquina virtual en que se ha realizado la conmutación por error > **Volver a proteger**.
2. En **Reproteger**, compruebe que la opción **De Azure a local** esté seleccionada.
3. Especifique el servidor de destino maestro local y el servidor de procesos.
4. En **Almacén de datos**, seleccione el almacén de datos de destino maestro en el que quiera recuperar los discos en el entorno local.
       - Use esta opción si la máquina virtual local se ha eliminado o no existe, y es preciso crear discos.
       - Esta configuración se omite si existen discos, pero es necesario especificar un valor.
5. Seleccione la unidad de retención de destino maestra. La directiva de conmutación por recuperación se selecciona automáticamente.
6. Haga clic en **Aceptar** para comenzar con la reprotección. Comienza un trabajo para replicar la máquina virtual de Azure en el sitio local. Puede realizar el seguimiento del progreso en la pestaña **Trabajos**.

> [!NOTE]
> Si desea recuperar la máquina virtual de Azure en una máquina virtual local existente, el almacén de datos de la máquina virtual local debe montarse con acceso de lectura/escritura en el host ESXi del servidor de destino maestro.


## <a name="fail-back-from-azure"></a>Conmutación por recuperación desde Azure

Ejecute la conmutación por error de la manera siguiente:

1. En la página **Elementos replicados**, haga clic con el botón derecho en la máquina y seleccione **Conmutación por error no planeada**.
2. En **Confirmar conmutación por error**, compruebe que la dirección de la conmutación por error sea desde Azure.
3. Seleccione el punto de recuperación que desea usar para la conmutación por error.
    - Se recomienda usar el punto de recuperación **Más reciente**. El punto coherente con la aplicación se encuentra detrás del último momento dado y provoca cierta pérdida de datos.
    - **Más reciente** es un punto de recuperación coherente con los bloqueos.
    - Cuando se ejecuta la conmutación por error, Site Recovery apaga las máquinas virtuales de Azure y arranca la máquina virtual local. Habrá cierto tiempo de inactividad, por lo que debe elegir la hora adecuada.
4. Haga clic con el botón derecho en la máquina y haga clic en **Confirmar**. Esto desencadenará un trabajo en el que se quitarán las máquinas virtuales de Azure.
5. Compruebe que las máquinas virtuales de Azure se hayan apagado según lo previsto.


## <a name="reprotect-on-premises-machines-to-azure"></a>Reprotección de máquinas locales en Azure

Los datos ahora deben volver a estar en el sitio local, pero no se está replicando en Azure. Puede volver a iniciar la replicación en Azure de la siguiente forma:

1. En el almacén > **Configuración** >**Elementos replicados**, seleccione las máquinas virtuales que han sido objeto de la conmutación por recuperación y haga clic en **Reproteger**.
2. Seleccione el servidor de procesos que se usa para enviar los datos replicados a Azure y haga clic en **Aceptar**.


## <a name="next-steps"></a>Pasos siguientes

Una vez finalizado el trabajo de reprotección, la máquina virtual local se replica en Azure. Si fuera necesario, podría [ejecutar otra conmutación por error](site-recovery-failover.md) en Azure.
