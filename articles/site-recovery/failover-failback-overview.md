---
title: Acerca de la conmutación por error y la conmutación por recuperación en Azure Site Recovery
description: Obtenga información sobre la conmutación por error y la conmutación por recuperación en Azure Site Recovery.
ms.topic: conceptual
ms.date: 12/24/2019
ms.openlocfilehash: d9b54f3c452212e12419a5ffd67b116c8660308d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87089539"
---
# <a name="about-on-premises-disaster-recovery-failoverfailback"></a>Acerca de la conmutación por error o conmutación por recuperación de la recuperación ante desastres local

En este artículo se proporciona información general sobre la conmutación por error y la conmutación por recuperación durante la recuperación ante desastres de máquinas locales en Azure mediante [Azure Site Recovery](site-recovery-overview.md).

## <a name="recovery-stages"></a>Fases de recuperación

La conmutación por error y la conmutación por recuperación en Site Recovery constan de cuatro fases:

- **Fase 1: conmutación por error desde una ubicación local**: después de configurar la replicación en Azure para las máquinas locales, cuando el sitio local deje de funcionar, dichas máquinas se conmutarán por error en Azure. Tras la conmutación por error, se crean máquinas virtuales de Azure a partir de los datos replicados.
- **Fase 2: reproteger las máquinas virtuales de Azure**: en Azure, las máquinas virtuales de Azure se reprotegen para que empiecen a replicarse de nuevo en el sitio local. La máquina virtual local (si está disponible) está apagada durante la reprotección, para ayudar a garantizar la coherencia de los datos.
- **Fase 3: conmutación por error desde Azure**: cuando el sitio local funcione nuevamente con normalidad, ejecute otra conmutación por error. Esta vez, para conmutar por recuperación las máquinas virtuales de Azure en el sitio local. Puede realizar una conmutación por recuperación en la ubicación original de la conmutación por error o en una ubicación alternativa.
- **Fase 4: reprotección de máquinas locales en Azure**: después de la conmutación por recuperación, habilite de nuevo la replicación de las máquinas locales en Azure.

## <a name="failover"></a>Conmutación por error

Se realiza una conmutación por error como parte de la estrategia de continuidad empresarial y recuperación ante desastres (BCDR).

- Como primer paso de la estrategia de BCDR, puede replicar las máquinas locales en Azure de forma continua. Los usuarios tienen acceso a las cargas de trabajo y aplicaciones que se ejecutan en las máquinas de origen locales.
- Si surge la necesidad (por ejemplo, si hay una interrupción en el entorno local), las máquinas de replicación se conmutan por error en Azure. Las máquinas virtuales de Azure se crean mediante los datos replicados.
- Para la continuidad empresarial, los usuarios pueden seguir accediendo a las aplicaciones en las máquinas virtuales de Azure.

La conmutación por error es una actividad de dos fases:

- **Conmutación por error**: la conmutación por error que crea y pone en marcha una máquina virtual de Azure mediante el punto de recuperación seleccionado.
- **Confirmación**: después de la conmutación por error, compruebe la máquina virtual en Azure:
    - Después, puede confirmar la conmutación por error al punto de recuperación seleccionado, o seleccionar un punto diferente para la confirmación.
    - No se puede cambiar el punto de recuperación después de confirmar la conmutación por error.


## <a name="connect-to-azure-after-failover"></a>Conexión a Azure después de la conmutación por error

Para conectarse a las máquinas virtuales de Azure creadas después de la conmutación por error mediante RDP o SSH, debe cumplir una serie de requisitos.

**Conmutación por error** | **Ubicación** | **Acciones**
--- | --- | ---
**Máquina virtual de Azure (Windows)** | En la máquina local antes de la conmutación por error | **Acceso a través de Internet**: habilite RDP. Asegúrese de que se hayan agregado las reglas de TCP y UDP para **Público**, y que RDP se permite en todos los perfiles de **Firewall de Windows** > **Aplicaciones permitidas**.<br/><br/> **Acceso a través de VPN de sitio a sitio**: habilite RDP en la máquina. Compruebe que se permite RDP en **Firewall de Windows** -> **Aplicaciones y características permitidas** para redes de **dominio y privadas**.<br/><br/>  Asegúrese de que la directiva SAN del sistema operativo está establecida en **OnlineAll**. [Más información](https://support.microsoft.com/kb/3031135).<br/><br/> Asegúrese de que no haya actualizaciones de Windows pendientes en la máquina virtual cuando se desencadena una conmutación por error. Windows Update puede iniciarse cuando realice la conmutación por error y no podrá iniciar sesión en la máquina virtual hasta que se completen las actualizaciones.
**Máquina virtual de Azure que ejecuta Windows** | En la máquina virtual de Azure después de la conmutación por error |  [Agregue una dirección IP pública](https://aka.ms/addpublicip) para la máquina virtual.<br/><br/> Las reglas del grupo de seguridad de red de la máquina virtual conmutada por error (y la subred de Azure a la que esta se conecta) deben permitir las conexiones entrantes al puerto de RDP.<br/><br/> Seleccione **Diagnósticos de arranque** para comprobar una captura de pantalla de la máquina virtual. Si no puede conectarse, compruebe que se está ejecutando la máquina virtual y revise las [sugerencias de solución de problemas](https://social.technet.microsoft.com/wiki/contents/articles/31666.troubleshooting-remote-desktop-connection-after-failover-using-asr.aspx).
**Máquina virtual de Azure que ejecuta Linux** | En la máquina local antes de la conmutación por error | Asegúrese de que el servicio de Secure Shell en la máquina virtual está configurado para iniciarse automáticamente en el arranque del sistema.<br/><br/> Compruebe que las reglas de firewall permiten una conexión SSH.
**Máquina virtual de Azure que ejecuta Linux** | En la máquina virtual de Azure después de la conmutación por error | Las reglas del grupo de seguridad de red de la máquina virtual conmutada por error (y la subred de Azure a la que esta se conecta) necesitan permitir las conexiones entrantes al puerto SSH.<br/><br/> [Agregue una dirección IP pública](https://aka.ms/addpublicip) para la máquina virtual.<br/><br/> Seleccione **Diagnósticos de arranque** para obtener una captura de pantalla de la máquina virtual.<br/><br/>

## <a name="types-of-failover"></a>Tipos de conmutación por error

Site Recovery proporciona diferentes opciones de conmutación por error.

**Conmutación por error** | **Detalles** | **Recuperación** | **Workflow**
--- | --- | --- | ---
**Conmutación por error de prueba** | Se usa para ejecutar un simulacro que valida la estrategia de BCDR sin pérdida de datos ni tiempo de inactividad.| Crea una copia de la máquina virtual en Azure, sin afectar a la replicación en curso ni al entorno de producción. | 1. Ejecute una conmutación por error de prueba en una sola máquina virtual o en varias en un plan de recuperación.<br/><br/> 2. Seleccione un punto de recuperación para usarlo en la conmutación por error de prueba.<br/><br/> 3. Seleccione una red de Azure en la que la máquina virtual de Azure se encuentra cuando se crea después de la conmutación por error. La red solo se usa para la conmutación por error de prueba.<br/><br/> 4. Compruebe que el simulacro funciona según lo previsto. Site Recovery limpia automáticamente las máquinas virtuales creadas en Azure durante el simulacro.
**Conmutación por error planeada de Hyper-V**  | Normalmente se usa para el tiempo de inactividad planeado.<br/><br/> Las máquinas virtuales de origen se apagan. Los datos más recientes se sincronizan antes de iniciar la conmutación por error. | Cero pérdida de datos para el flujo de trabajo planeado. | 1. Planee un período de inactividad por mantenimiento y notifique a los usuarios.<br/><br/> 2. Desconecte las aplicaciones orientadas al usuario.<br/><br/> 3. Inicie una conmutación por error planeada con el punto de recuperación más reciente. La conmutación por error no se ejecuta si el equipo no se apaga, o si se producen errores.<br/><br/> 4. Después de la conmutación por error, compruebe que la máquina virtual de Azure de réplica está activa en Azure.<br/><br/> 5. Confirme la conmutación por error para finalizar. La acción de confirmación elimina todos los puntos de recuperación.
**Conmutación por error de Hyper-V** | Normalmente se ejecuta si se produce una interrupción no planeada o el sitio primario no está disponible.<br/><br/> También puede apagar la máquina virtual y sincronizar los cambios finales antes de iniciar la conmutación por error.  | Pérdida de datos mínima para las aplicaciones. | 1. Inicie el plan de BCDR. <br/><br/> 2. Inicie una conmutación por error. Especifique si Site Recovery debe apagar la máquina virtual y sincronizar/replicar los cambios más recientes antes de desencadenar la conmutación por error.<br/><br/> 3. Puede conmutar por error a varias opciones de puntos de recuperación, que están resumidas en la tabla siguiente.<br/><br/> Si no habilita la opción para apagar la máquina virtual, o si Site Recovery no puede apagarla, se usa el punto de recuperación más reciente.<br/>La conmutación por error se ejecuta incluso si la máquina no se puede apagar.<br/><br/> 4. Después de la conmutación por error, debe comprobar que la máquina virtual de Azure de réplica está activa en Azure.<br/> Si es necesario, puede seleccionar un punto de recuperación diferente dentro del período de retención de 24 horas.<br/><br/> 5. Confirme la conmutación por error para finalizar. La acción de confirmación elimina todos los puntos de recuperación disponibles.
**Conmutación por error de VMware** | Normalmente se ejecuta si se produce una interrupción no planeada o el sitio primario no está disponible.<br/><br/> De manera opcional, especifique si Site Recovery debe tratar de apagar la máquina virtual y sincronizar y replicar los cambios finales antes de iniciar la conmutación por error.  | Pérdida de datos mínima para las aplicaciones. | 1. Inicie el plan de BCDR. <br/><br/> 2. Inicie una conmutación por error desde Site Recovery. Especifique si Site Recovery debe tratar de apagar y sincronizar la máquina virtual antes de ejecutar la conmutación por error.<br/> La conmutación por error se ejecuta incluso si las máquinas no se pueden apagar.<br/><br/> 3. Después de la conmutación por error, compruebe que la máquina virtual de Azure de réplica está activa en Azure. <br/>Si es necesario, puede seleccionar un punto de recuperación diferente dentro del período de retención de 72 horas.<br/><br/> 5. Confirme la conmutación por error para finalizar. La acción de confirmación elimina todos los puntos de recuperación.<br/> En el caso de las máquinas virtuales Windows, Site Recovery deshabilita las herramientas de VMware durante la conmutación por error.

## <a name="failover-processing"></a>Procesamiento de la conmutación por error

En algunos escenarios, la conmutación por error requiere un procesamiento adicional que tarda aproximadamente de ocho a diez minutos en completarse. Puede observar que la conmutación por error de prueba tarda más tiempo en realizarse:

* Máquinas virtuales de VMware que ejecutan una versión de Mobility Service anterior a la 9.8
* Servidores físicos
* Máquinas virtuales de VMware Linux
* Máquinas virtuales de Hyper-V protegidas como servidores físicos
* Máquinas virtuales de VMware que no tienen habilitado el servicio DHCP
* Máquinas virtuales de VMware que no tienen los siguientes controladores de arranque: storvsc, vmbus, storflt, intelide o atapi.

## <a name="recovery-point-options"></a>Opciones de punto de recuperación

Durante la conmutación por error, puede seleccionar varias opciones de punto de recuperación.

**Opción** | **Detalles**
--- | ---
**Más reciente (RPO más bajo)** | Esta opción proporciona el objetivo de punto de recuperación (RPO) más bajo. Procesa primero todos los datos que se han enviado al servicio Site Recovery para crear un punto de recuperación para cada máquina virtual antes de conmutarla por error a dicho punto de recuperación. Este punto de recuperación tiene todos los datos replicados en Site Recovery cuando se desencadenó la conmutación por error.
**Procesado más recientemente**  | Con esta opción se realiza una conmutación por error de las máquinas virtuales al último punto de recuperación que procesó Site Recovery. Para ver el último punto de recuperación de una máquina virtual específica, seleccione **Puntos de recuperación más recientes** en la configuración de la máquina virtual. Esta opción proporciona un objetivo de tiempo de recuperación (RTO) bajo, ya que no se invierte tiempo en el procesamiento de datos sin procesar.
**Más reciente coherente con la aplicación** |  Esta opción conmuta por error las máquinas virtuales al punto de recuperación más reciente coherente con la aplicación que haya procesado Site Recovery si están habilitados los puntos de recuperación coherentes con la aplicación. Compruebe el punto de recuperación más reciente en la configuración de la máquina virtual.
**Último procesamiento de máquinas virtuales múltiples** | esta opción está disponible para los planes de recuperación con una o varias máquinas virtuales que tienen habilitada la coherencia entre varias máquinas virtuales. Máquinas virtuales con la conmutación por error habilitada para el último punto de recuperación común coherente con varias máquinas virtuales. Las demás máquinas virtuales del plan realizan la conmutación por error al último punto de recuperación procesado.
**Último coherente con aplicación de máquinas virtuales múltiples** |  esta opción está disponible para los planes de recuperación con una o varias máquinas virtuales que tienen habilitada la coherencia entre varias máquinas virtuales. Las máquinas virtuales que forman parte de un grupo de replicación conmutan por error al último punto de recuperación común coherente con la aplicación de varias máquinas virtuales. Otras máquinas virtuales conmutan por error a su punto de recuperación coherente con la aplicación más reciente.
**Personalizada** | Use esta opción para conmutar por error una máquina virtual específica a un punto de recuperación concreto en el tiempo. Esta opción no está disponible para los planes de recuperación.

> [!NOTE]
> Los puntos de recuperación no se pueden migrar a otro almacén de Recovery Services.

## <a name="reprotectionfailback"></a>Reprotección y conmutación por recuperación

Después de la conmutación por error en Azure, las máquinas virtuales de Azure replicadas quedan en un estado no protegido.

- Como primer paso para la conmutación por recuperación al sitio local, debe iniciar las máquinas virtuales de Azure que se replican en el entorno local. El proceso de reprotección depende del tipo de máquinas que se conmutaron por error.
- Una vez que las máquinas se replican de Azure a un entorno local, puede ejecutar una conmutación por error de Azure al sitio local.
- Una vez que las máquinas de nuevo estén en ejecución de forma local, puede habilitar la replicación para que se repliquen en Azure para la recuperación ante desastres.

La conmutación por recuperación funciona como se indica a continuación:

- Para conmutar por recuperación, una máquina virtual necesita al menos un punto de recuperación para realizar la conmutación por recuperación. En un plan de recuperación, todas las máquinas virtuales del plan necesitan al menos un punto de recuperación.
- Se recomienda usar el **último** punto de recuperación para conmutar por recuperación (se trata de un punto coherente frente a bloqueos).
    - Existe una opción de punto de recuperación coherente con la aplicación. En este caso, una sola máquina virtual se recupera al último punto de recuperación coherente con la aplicación disponible. Para el caso de un plan de recuperación con un grupo de replicación, todos los grupos de replicación se recuperarán a su punto de recuperación disponible común.
    - Los puntos de recuperación coherentes con la aplicación pueden ser anteriores, por lo que podrían perderse datos.
- Durante la conmutación por error de Azure al sitio local, Site Recovery apaga las máquinas virtuales de Azure. Al confirmar la conmutación por error, Site Recovery quita las máquinas virtuales de Azure conmutadas por error en Azure.


## <a name="vmwarephysical-reprotectionfailback"></a>VMware/reprotección física/conmutación por recuperación

Para reproteger y conmutar por recuperación las máquinas y servidores físicos de VMware de Azure a un entorno local, necesita una infraestructura de conmutación por recuperación y debe cumplir una serie de requisitos.

- **Servidor de procesos temporal de Azure**: para realizar una conmutación por recuperación desde Azure, debe configurar una máquina virtual de Azure para que actúe como servidor de procesos y controle la replicación desde Azure. Dicha máquina virtual se puede eliminar cuando finalice la conmutación por recuperación.
- **Conexión VPN**: para la conmutación por recuperación, necesita una conexión VPN o ExpressRoute desde la red de Azure al sitio local.
- **Servidor de destino maestro independiente**: de manera predeterminada, el servidor de destino maestro que se instaló con el servidor de configuración en la máquina virtual local de VMware controla la conmutación por recuperación. Si necesita realizar la conmutación por recuperación en grandes volúmenes de tráfico, debe configurar un servidor de destino maestro local diferente para este propósito.
- **Directiva de conmutación por recuperación**: para replicar de nuevo en el sitio local, necesita una directiva de conmutación por recuperación. Esta directiva se crea automáticamente cuando crea una directiva de replicación entre el entorno local y Azure.
    - Esta directiva se asocia automáticamente al servidor de configuración.
    - Esta directiva no se puede editar.
    - Valores de la directiva: Umbral de RPO: 15 minutos; retención de punto de recuperación: 24 horas; frecuencia de instantánea coherente con la aplicación: 60 minutos.

Obtenga más información sobre la reprotección y la conmutación por recuperación físicas o de VMware:
- [Revise](vmware-azure-reprotect.md#before-you-begin) los requisitos adicionales para la reprotección y la conmutación por recuperación.
- [Implemente](vmware-azure-prepare-failback.md#deploy-a-process-server-in-azure) un servidor de procesos en Azure.
- [Implemente](vmware-azure-prepare-failback.md#deploy-a-separate-master-target-server) un servidor de destino maestro independiente.

Al reproteger las máquinas virtuales de Azure en un entorno local, puede especificar si quiere conmutar por recuperación a la ubicación original o a una ubicación alternativa.

- **Recuperación de ubicación original**: esta opción conmuta por recuperación de Azure a la misma máquina local de origen (si existe). En este escenario solo los cambios se replican de nuevo al entorno local.
- **Recuperación de ubicación alternativa**: si no existe la máquina local, puede conmutar por recuperación de Azure a una ubicación alternativa. Al reproteger la máquina virtual de Azure en un entorno local, se crea la máquina local. La replicación de datos completa se produce de Azure a un entorno local. --[Revise](concepts-types-of-failback.md) los requisitos y las limitaciones de la conmutación por recuperación en la ubicación.



## <a name="hyper-v-reprotectionfailback"></a>Reprotección y conmutación por recuperación de Hyper-V

Para reproteger y conmutar por recuperación máquinas virtuales de Hyper-V de Azure a un entorno local, tome en cuenta lo siguiente:

- Solo puede conmutar por recuperación máquinas virtuales de Hyper-V que se replican con una cuenta de almacenamiento. No se admite la conmutación por recuperación de máquinas virtuales de Hyper-V que se replican mediante discos administrados.
- Los hosts de Hyper-V locales (o System Center VMM si se usan) deben conectarse a Azure.
- Ejecute una conmutación por recuperación planeada de Azure a un entorno local.
- No es necesario configurar ningún componente específico para la conmutación por recuperación de las máquinas virtuales de Hyper-V.
- Durante la conmutación por error planeada, puede seleccionar opciones para sincronizar los datos antes de realizar la conmutación por recuperación:
    - **Sincronizar datos antes de la conmutación por error**: esta opción reduce el tiempo de inactividad de las máquinas virtuales, ya que las sincroniza sin apagarlas.
        - Fase 1: toma una instantánea de la máquina virtual de Azure y la copia en el host de Hyper-V local. El equipo continúa ejecutándose en Azure.
        - Fase 2: apaga la máquina virtual de Azure de modo que no se produzca ningún cambio nuevo. El último conjunto de cambios incrementales se transfiere al servidor local y se inicia la máquina virtual local.
    - **Sincronizar datos solo durante la conmutación por error**: esta opción es más rápida, ya que se prevé que la mayoría del disco ha cambiado y, por lo tanto, no se realizan los cálculos de la suma de comprobación. Realiza una descarga del disco. Se recomienda utilizar esta opción si la máquina virtual ha estado trabajando con Azure durante un tiempo (un mes o más) o si se ha eliminado a la máquina virtual local.

[Obtenga más información](hyper-v-azure-failback.md) sobre los procesos de reprotección y conmutación por recuperación de Hyper-V.

Al reproteger las máquinas virtuales de Azure en un entorno local, puede especificar si quiere conmutar por recuperación a la ubicación original o a una ubicación alternativa.

- **Recuperación de ubicación original**: esta opción conmuta por recuperación de Azure a la misma máquina local de origen (si existe). En este escenario, se selecciona una de las opciones de sincronización descritas en el procedimiento anterior.
- **Recuperación de ubicación alternativa**: si no existe la máquina local, puede conmutar por recuperación de Azure a una ubicación alternativa. Al reproteger la máquina virtual de Azure en un entorno local, se crea la máquina local. En este caso, se recomienda seleccionar la opción para sincronizar los datos antes de la conmutación por error.
- [Revise](hyper-v-azure-failback.md) los requisitos y las limitaciones de la conmutación por recuperación en la ubicación.


Después de la conmutación por recuperación al sitio local, habilite la **Replicación inversa** para empezar a replicar la máquina virtual en Azure, de modo que se complete el ciclo.




## <a name="next-steps"></a>Pasos siguientes
- Conmutar por error [máquinas virtuales específicas de VMware](vmware-azure-tutorial-failover-failback.md).
- Conmutar por error [máquinas virtuales específicas de Hyper-V](hyper-v-azure-failover-failback-tutorial.md).
- [Crear](site-recovery-create-recovery-plans.md) un plan de recuperación.
- Conmutar por error [máquinas virtuales en un plan de recuperación](site-recovery-failover.md).
- [Prepararse para](vmware-azure-failback.md) la reprotección y la conmutación por recuperación de VMware.
- Conmutar por recuperación [máquinas virtuales de Hyper-V](hyper-v-azure-failback.md)

