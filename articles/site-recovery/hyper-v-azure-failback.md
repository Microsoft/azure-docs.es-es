---
title: Conmutación por recuperación de máquinas virtuales de Hyper-V desde Azure con Azure Site Recovery
description: Cómo conmutar por recuperación máquinas virtuales de Hyper-V en un sitio local de Azure con Azure Site Recovery.
services: site-recovery
author: Rajeswari-Mamilla
manager: gaggupta
ms.service: site-recovery
ms.topic: article
ms.date: 09/12/2019
ms.author: ramamill
ms.openlocfilehash: a31a28728dd0521262bd0518cc49a385f4314302
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87416237"
---
# <a name="run-a-failback-for-hyper-v-vms"></a>Ejecución de una conmutación por recuperación para máquinas virtuales de Hyper-V

En este artículo se describe cómo conmutar por recuperación las máquinas virtuales de Azure que se crearon después de la conmutación por error de las máquinas virtuales de Hyper-V desde un sitio local a Azure con [Azure Site Recovery](site-recovery-overview.md).

- La conmutación por recuperación de las máquinas virtuales de Hyper-V desde Azure se realiza mediante la ejecución de una conmutación por error planeada de Azure al sitio local. Si la dirección de la conmutación por error es de Azure al sitio local, es una conmutación por recuperación.
- Puesto que Azure es un entorno de alta disponibilidad y las máquinas virtuales siempre están disponibles, la conmutación por recuperación desde Azure es una actividad planeada. Puede planear un tiempo de inactividad reducido para que las cargas de trabajo puedan empezar a ejecutarse de nuevo en el entorno local. 
- La conmutación por recuperación planeada desactiva las máquinas virtuales de Azure y descarga los cambios más recientes. No se prevé ninguna pérdida de datos.

## <a name="before-you-start"></a>Antes de comenzar

1. [Revise los tipos de conmutación por recuperación](failover-failback-overview.md#hyper-v-reprotectionfailback) que puede usar: recuperación de ubicación original y recuperación de ubicación alternativa.
2. Asegúrese de que las máquinas virtuales de Azure usan una cuenta de almacenamiento y no discos administrados. No se admite la conmutación por recuperación de máquinas virtuales de Hyper-V replicadas mediante discos administrados.
3. Compruebe que el host de Hyper-V local (o el servidor de System Center VMM si usa Site Recovery) está en ejecución y conectado a Azure. 
4. Asegúrese de que la conmutación por error y la confirmación se hayan completado para las máquinas virtuales. No es necesario configurar ningún componente de Site Recovery específico para la conmutación por recuperación de máquinas virtuales de Hyper-V de Azure.
5. El tiempo necesario para completar la sincronización de datos e iniciar la máquina virtual local dependerá de varios factores. Para acelerar la descarga de datos, puede configurar el agente de Microsoft Recovery Services para utilizar más subprocesos a fin de equiparar la descarga. [Más información](https://support.microsoft.com/help/3056159/how-to-manage-on-premises-to-azure-protection-network-bandwidth-usage).


## <a name="fail-back-to-the-original-location"></a>Conmutación por recuperación a la ubicación original

Para conmutar por recuperación las máquinas virtuales de Hyper-V en Azure a la máquina virtual local original, ejecute una conmutación por error planeada de Azure al sitio local como se indica a continuación:

1. En el almacén > **Elementos replicados**, seleccione la máquina virtual. Haga clic derecho en la máquina virtual > **Conmutación por error planeada**. Si realiza la conmutación por recuperación de un plan de recuperación, seleccione el nombre del plan y haga clic en **Conmutación por error** > **Conmutación por error planeada**.
2. En **Confirmar conmutación por error planeada**, elija las ubicaciones de origen y de destino. Tenga en cuenta la dirección de la conmutación por error. Si la conmutación por error desde la ubicación principal ha funcionado como se esperaba y todas las máquinas virtuales están en la ubicación secundaria, este dato solo es informativo.
3. En **Sincronización de datos**, seleccione una opción:
    - **Sincronizar datos antes de la conmutación por error (sincronizar solo cambios incrementales)** : esta opción reduce al mínimo el tiempo de inactividad de las máquinas virtuales, ya que realiza la sincronización sin apagarlas.
        - **Fase 1**: toma una instantánea de la máquina virtual de Azure y la copia en el host de Hyper-V local. El equipo continúa ejecutándose en Azure.
        - **Fase 2**: apaga la máquina virtual de Azure de modo que no se produzca ningún cambio nuevo. El último conjunto de cambios incrementales se transfiere al servidor local y se inicia la máquina virtual local.
    - **Sincronizar datos solo durante la conmutación por error (descarga completa)** : esta opción es más rápida porque se supone que la mayor parte del disco ha cambiado y no se quiere dedicar tiempo a calcular las sumas de comprobación. Esta opción no realiza cálculos de suma de comprobación.
        - Realiza una descarga del disco. 
        - Se recomienda utilizar esta opción si ha estado trabajando con Azure durante un tiempo (un mes o más) o si se elimina la máquina virtual local.

4. Para VMM solamente, si en la nube está habilitado el cifrado de datos, en **Clave de cifrado**, seleccione el certificado que se emitió al habilitarlo durante la instalación del proveedor en el servidor de VMM.
5. Inicie la conmutación por error. Puede seguir el progreso de la conmutación por error en la pestaña **Trabajos** .
6. Si seleccionó la opción para sincronizar los datos antes de la conmutación por error, cuando la sincronización de datos inicial esté completa y esté listo para apagar las máquinas virtuales en Azure, haga clic en **Trabajos** > nombre del trabajo > **Completar conmutación por error**. Esto hace lo siguiente:
    - Apaga la máquina de Azure.
    - Transfiere los cambios más recientes a la máquina virtual local.
    - Inicia la máquina virtual local.
7. Ahora puede iniciar sesión en la máquina virtual local para comprobar que está disponible según lo previsto.
8. La máquina virtual está en un estado pendiente de confirmación. Haga clic en **Confirmar** para confirmar la conmutación por error.
9. Para completar la conmutación por recuperación, haga clic en **Replicación inversa** para iniciar la replicación de la máquina virtual local en Azure nuevamente.



## <a name="fail-back-to-an-alternate-location"></a>Conmutación por recuperación a la ubicación alternativa 

Conmute por recuperación a la ubicación alternativa del siguiente modo:

1. Si configura nuevo hardware, instale una [versión compatible de Windows](hyper-v-azure-support-matrix.md#replicated-vms) y el rol de Hyper-V en la máquina.
2. Cree un conmutador de red virtual con el mismo nombre que tenía en el servidor original.
3. En **Elementos protegidos** > **Grupo de protección** > \<ProtectionGroupName> -> \<VirtualMachineName>, seleccione la máquina virtual en la que desea realizar la conmutación por recuperación y seleccione **Conmutación por error planeada**.
4. En **Confirmar conmutación por error planeada**, seleccione **Crear máquina virtual local si no existe**.
5. En **Nombre de host**, seleccione el nuevo servidor host de Hyper-V en el que desea incluir la máquina virtual.
6. En **Sincronización de datos**, se recomienda seleccionar la opción para sincronizar los datos antes de la conmutación por error. Así se reduce el tiempo de inactividad de las máquinas virtuales, ya que la sincronización se realiza sin apagarlas. Realiza las operaciones siguientes:
    - **Fase 1**: toma una instantánea de la máquina virtual de Azure y la copia en el host de Hyper-V local. El equipo continúa ejecutándose en Azure.
    - **Fase 2**: apaga la máquina virtual de Azure de modo que no se produzca ningún cambio nuevo. El último conjunto de cambios se transfiere al servidor local y se inicia la máquina virtual local.
    
7. Haga clic en la marca de verificación para iniciar la conmutación por error (conmutación por recuperación).
8. Después de que finalice la sincronización inicial y esté listo para apagar la máquina virtual de Azure, haga clic en **Trabajos** > \<planned failover job> > **Completar conmutación por error**. Esta opción apaga la máquina de Azure, transfiere los últimos cambios a la máquina virtual local y la inicia.
9. Puede iniciar sesión en la máquina virtual local para comprobar que todo funciona según lo previsto.
10. Haga clic en **Confirmar** para finalizar la conmutación por error. La confirmación elimina la máquina virtual de Azure y sus discos, y prepara la máquina virtual local para volver a protegerla.
10. Haga clic en **Replicación inversa** para iniciar la replicación de la máquina virtual local en Azure. Solo se replicarán los cambios incrementales realizados desde que se apagó la máquina virtual en Azure.

    > [!NOTE]
    > Si cancela el trabajo de conmutación por recuperación durante la sincronización de datos, la máquina virtual local tendrá un estado dañado. Esto se debe a que la sincronización de datos copia los datos más recientes de los discos de la máquina virtual de Azure a los discos de datos locales y puede que los datos de los discos no sean coherentes hasta que la sincronización se complete. Si la máquina virtual local inicia después de cancelar la sincronización de datos, es posible que no arranque. En este caso, vuelva a ejecutar la conmutación por error para completar la sincronización de datos.


## <a name="next-steps"></a>Pasos siguientes
Después de replicar la máquina virtual local en Azure, puede [ejecutar otra conmutación por error](site-recovery-failover.md) en Azure según sea necesario.
