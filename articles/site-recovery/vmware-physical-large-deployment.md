---
title: Escale la recuperación ante desastres de VMware/física con Azure Site Recovery
description: Obtenga información sobre cómo configurar la recuperación ante desastres en Azure para grandes cantidades de servidores físicos o máquinas virtuales de VMware locales con Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/14/2019
ms.author: raynew
ms.openlocfilehash: 101e42263e46c5a21f26b0fa9cdeed798525fee9
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89047094"
---
# <a name="set-up-disaster-recovery-at-scale-for-vmware-vmsphysical-servers"></a>Configuración de recuperación ante desastres a escala para servidores físicos o máquinas virtuales de VMware

En este artículo se describe cómo configurar la recuperación ante desastres en Azure para grandes cantidades (más de 1000) de servidores físicos o máquinas virtuales de VMware en el entorno de producción mediante el servicio [Azure Site Recovery](site-recovery-overview.md).


## <a name="define-your-bcdr-strategy"></a>Definición de la estrategia de BCDR

Como parte de la estrategia de continuidad empresarial y recuperación ante desastres (BCDR), puede definir los objetivos de punto de recuperación (RPO) y los de tiempo de recuperación (RTO) para las cargas de trabajo y aplicaciones empresariales. El RTO mide la duración del período y el nivel del servicio dentro de los cuales se debe restaurar un proceso o una aplicación empresarial y estar disponible para evitar los problemas de continuidad.
- Site Recovery brinda una replicación continua para los servidores físicos y las máquinas virtuales de VMware y un [SLA](https://azure.microsoft.com/support/legal/sla/site-recovery/v1_2/) para el RTO.
- Cuando planea una recuperación ante desastres a gran escala de máquinas virtuales de VMware y determina los recursos de Azure que necesita, puede especificar un valor de RTO que se usará para calcular la capacidad.


## <a name="best-practices"></a>Procedimientos recomendados

Algunos procedimientos recomendados generales para la recuperación ante desastres a gran escala. Estos procedimientos recomendados se analizan con más detalle en las secciones siguientes del documento.

- **Identificar los requisitos de destino**: calcule las necesidades de capacidad y recursos de Azure antes de configurar la recuperación ante desastres.
- **Planear los componentes de Site Recovery**: sepa cuáles son los componentes de Site Recovery (servidor de configuración, servidores de proceso) que necesita para satisfacer la capacidad calculada.
- **Configurar uno o más servidores de proceso de escalabilidad horizontal**: no use el servidor de proceso que se ejecuta de manera predeterminada en el servidor de configuración. 
- **Ejecutar las actualizaciones más recientes**: el equipo de Site Recovery lanza de manera periódica versiones nuevas de los componentes de Site Recovery y el usuario debe asegurarse de ejecutar las versiones más recientes. Para ayudarlo con eso, haga un seguimiento de las [novedades](site-recovery-whats-new.md) para ver las actualizaciones y [habilite e instale las actualizaciones](service-updates-how-to.md) a medida que aparecen.
- **Supervisar de manera proactiva**: cuando pone en marcha la recuperación ante desastres, debe supervisar de forma proactiva el estado de las máquinas replicadas y los recursos de la infraestructura.
- **Maniobras de recuperación ante desastres**: debe realizar maniobras de recuperación ante desastres de manera periódica. Estas maniobras no afectan el entorno de producción, pero sí permiten garantizar que la conmutación por error en Azure funcionará según lo esperado cuando sea necesario.



## <a name="gather-capacity-planning-information"></a>Recopilación de información sobre el planeamiento de capacidad

Recopile información sobre el entorno local para ayudar a evaluar y calcular las necesidades de capacidad de destino (Azure).
- En el caso de VMware, ejecute Deployment Planner para máquinas virtuales de VMware para hacer esto.
- En el caso de los servidores físicos, recopile manualmente la información.

### <a name="run-the-deployment-planner-for-vmware-vms"></a>Ejecución de Deployment Planner para máquinas virtuales de VMware

Deployment Planner lo ayuda a recopilar información sobre el entorno local de VMware.

- Ejecute Deployment Planner durante un período que representa la actividad típica de las máquinas virtuales. Esto generará recomendaciones y cálculos más precisos.
- Se recomienda ejecutar Deployment Planner en la máquina del servidor de configuración, debido a que Planner calcula el rendimiento desde el servidor en que se ejecuta. [Obtenga más información](site-recovery-vmware-deployment-planner-run.md#get-throughput) sobre cómo medir el rendimiento.
- Si todavía no tiene configurado un servidor de configuración:
    - [Obtenga información general](vmware-physical-azure-config-process-server-overview.md) de los componentes de Site Recovery.
    - [Configure un servidor de configuración](vmware-azure-deploy-configuration-server.md) donde ejecutar Deployment Planner.

Luego, ejecute Planner como se indica a continuación:

1. [Obtenga información](site-recovery-deployment-planner.md) sobre Deployment Planner. Puede descargar la versión más reciente desde el portal o [descargarla directamente](https://aka.ms/asr-deployment-planner).
2. Revise los [requisitos previos](site-recovery-deployment-planner.md#prerequisites) y las [actualizaciones más recientes](site-recovery-deployment-planner-history.md) para Deployment Planner y [descargue y extraiga](site-recovery-deployment-planner.md#download-and-extract-the-deployment-planner-tool) la herramienta.
3. [Ejecute Deployment Planner](site-recovery-vmware-deployment-planner-run.md) en el servidor de configuración.
4. [Genere un informe](site-recovery-vmware-deployment-planner-run.md#generate-report) para resumir los cálculos y las recomendaciones.
5. Analice las [recomendaciones de informe](site-recovery-vmware-deployment-planner-analyze-report.md) y los [cálculos de costos](site-recovery-vmware-deployment-planner-cost-estimation.md).

>[!NOTE]
> De manera predeterminada, la herramienta está configurada tanto para generar perfiles como para generar informes de hasta 1000 máquinas virtuales. Para cambiar este límite, aumente el valor de la clave MaxVMsSupported en el archivo ASRDeploymentPlanner.exe.config.

## <a name="plan-target-azure-requirements-and-capacity"></a>Planeamiento de la capacidad y los requisitos de destino (Azure)

Con las recomendaciones y los cálculos recopilados, puede planear la capacidad y los recursos de destino. Si ejecutó Deployment Planner para máquinas virtuales de VMware, puede usar un número de las [recomendaciones de informe](site-recovery-vmware-deployment-planner-analyze-report.md#recommendations) como ayuda.

- **Máquinas virtuales compatibles**: use este número para identificar la cantidad de máquinas virtuales que están preparadas para la recuperación ante desastres en Azure. Las recomendaciones sobre el ancho de banda de red y los núcleos de Azure se basan en este número.
- **Ancho de banda de red requerido**: observe el ancho de banda que necesita para la replicación diferencial de las máquinas virtuales compatibles. 
    - Cuando ejecute Planner, especifique el RPO deseado en minutos. Las recomendaciones muestran el ancho de banda necesario para cumplir con ese RPO el 100 % y el 90 % de las veces. 
    - Las recomendaciones de ancho de banda de red consideran el ancho de banda que se necesita para un número total de servidores de configuración y servidores de proceso que Planner recomienda.
- **Núcleos requeridos de Azure**: observe el número de núcleos que necesita en la región de Azure de destino en función del número de máquinas virtuales compatibles. Si no tiene los núcleos suficientes, Site Recovery no podrá crear las máquinas virtuales de Azure en la conmutación por error.
- **Tamaño de lote de máquinas virtuales recomendado**: El tamaño de lote recomendado se basa en la capacidad de finalizar la replicación inicial del lote dentro de 72 horas de manera predeterminada, mientras se cumple un RPO de 100 %. Es posible modificar el valor de la hora.

Puede usar estas recomendaciones para planear los recursos de Azure, el ancho de banda de red y el procesamiento por lotes de máquinas virtuales.

## <a name="plan-azure-subscriptions-and-quotas"></a>Planeamiento de cuotas y suscripciones de Azure

Queremos asegurarnos de que las cuotas disponibles en la suscripción de destino sean suficientes para administrar la conmutación por error.

**Task** | **Detalles** | **Acción**
--- | --- | ---
**Comprobación de núcleos** | Si los núcleos de la cuota disponible no son iguales ni superiores al recuento de destino total en el momento de la conmutación por error, las conmutación por error generarán un error. | En el caso de las máquinas virtuales de VMware, compruebe que tiene los núcleos suficientes en la suscripción de destino para cumplir la recomendación de núcleos de Deployment Planner.<br/><br/> En el caso de los servidores físicos, compruebe que los núcleos de Azure cumplen con los cálculos manuales.<br/><br/> Para comprobar las cuotas, en Azure Portal > **Suscripción**, haga clic en **Uso y cuotas**.<br/><br/> [Obtenga más información](../azure-portal/supportability/resource-manager-core-quotas-request.md) sobre cómo aumentar las cuotas.
**Comprobación de los límites de conmutación por error** | El número de conmutaciones por error no debe superar los límites de conmutación por error de Site Recovery. |  Si las conmutaciones por error superan los límites, puede agregar suscripciones y conmutar por error a varias suscripciones, o bien puede aumentar la cuota para una suscripción. 


### <a name="failover-limits"></a>Límites de conmutación por error

Los límites indican la cantidad de conmutaciones por error que Site Recovery admite durante una hora, suponiendo que hay tres discos por máquina.

¿Qué significa el cumplimiento? Para iniciar una máquina virtual de Azure, Azure requiere que algunos controladores estén en estado de inicio del arranque y que servicios como DHCP estén establecidos para iniciarse de manera automática.
- Las máquinas que cumplen los requisitos ya tienen implementada esta configuración.
- En el caso de las máquinas que ejecutan Windows, puede comprobar de manera proactiva el cumplimiento y, si es necesario, hacer que sean compatibles. [Más información](site-recovery-failover-to-azure-troubleshoot.md#failover-failed-with-error-id-170010).
- Las máquinas Linux solo son compatibles en el momento de la conmutación por error.

**¿La máquina es compatible con Azure?** | **Límites de máquinas virtuales de Azure (conmutación por error de un disco administrado)**
--- | --- 
Sí | 2000
No | 1000

- Los límites supone que, como mínimo, otros trabajos están en curso en la región de destino de la suscripción.
- Algunas regiones de Azure son más pequeñas y es posible que tengan límites ligeramente más bajos.

## <a name="plan-infrastructure-and-vm-connectivity"></a>Planeamiento de la infraestructura y la conectividad de VM

Después de la conmutación por error en Azure, necesita que las cargas de trabajo funcionen como lo harían en el entorno local y permitan que los usuarios accedan a las cargas de trabajo que se ejecutan en las máquinas virtuales de Azure.

- [Obtenga más información](site-recovery-active-directory.md#test-failover-considerations) sobre la conmutación por error de la infraestructura local DNS o de Active Directory en Azure.
- [Obtenga más información](site-recovery-test-failover-to-azure.md#prepare-to-connect-to-azure-vms-after-failover) sobre cómo preparar la conexión a las máquinas virtuales de Azure después de la conmutación por error.



## <a name="plan-for-source-capacity-and-requirements"></a>Planeamiento de los requisitos y la capacidad de origen

Es importante que tenga los servidores de configuración y los servidores de proceso de escalabilidad horizontal suficientes para cumplir con los requisitos de capacidad. Cuando empiece una implementación a gran escala, comience con un servidor de configuración único y un único servidor de proceso de escalabilidad horizontal. A medida que alcance los límites prescritos, agregue servidores adicionales.

>[!NOTE]
> En el caso de las máquinas virtuales de VMware, Deployment Planner brinda algunas recomendaciones sobre los servidores de configuración y de proceso que necesita. Se recomienda usar las tablas incluidas en los procedimientos siguientes, en lugar de seguir la recomendación de Deployment Planner. 


## <a name="set-up-a-configuration-server"></a>Configuración de un servidor de configuración
 
La capacidad del servidor de configuración se ve afectada por la cantidad de las máquinas que se replican y no por la tasa de renovación de los datos. Para saber si necesita más servidores de configuración, use estos límites de VM definidos.

**CPU** | **Memoria** | **Disco de caché** | **Límite de máquinas replicadas**
 --- | --- | --- | ---
8 vCPU<br> 2 sockets * 4 núcleos a 2,5 Ghz | 16 GB | 600 GB | Hasta 550 máquinas<br> Se supone que cada máquina tiene tres discos de 100 GB cada uno.

- Estos límites se basan en un servidor de configuración configurado mediante una plantilla de OVF.
- En los límites se supone que no está usando el servidor de proceso que se ejecuta de manera predeterminada en el servidor de configuración.

Si necesita agregar un servidor de configuración nuevo, siga estas instrucciones:

- [Configure un servidor de configuración](vmware-azure-deploy-configuration-server.md) para la recuperación ante desastres de máquinas virtuales de VMware mediante una plantilla de OVF.
- [Configure un servidor de configuración](physical-azure-set-up-source.md) de manera manual para los servidores físicos o para las implementaciones de VMware que no pueden usar una plantilla de OVF.

Cuando configure un servidor de configuración, tenga en cuenta que:

- Cuando configura un servidor de configuración, es importante tener en cuenta la suscripción y el almacén donde este reside, porque no se deben cambiar después de la configuración. Si necesita cambiar el almacén, debe desasociar el servidor de configuración del almacén y volver a registrarlo. Esto detiene la replicación de las máquinas virtuales del almacén.
- Si quiere configurar un servidor de configuración con varios adaptadores de red, debe hacerlo durante la configuración. No puede hacerlo después de registrar el servidor de configuración en el almacén.

## <a name="set-up-a-process-server"></a>Configuración de un servidor de proceso

La capacidad del servidor de proceso se ve afectada por las tasas de renovación de los datos y no por la cantidad de máquinas habilitadas para la replicación.

- En el caso de implementaciones grandes, siempre debe tener al menos un servidor de proceso de escalabilidad horizontal.
- Para saber si necesita más servidores, use la tabla siguiente.
- Se recomienda agregar un servidor con la especificación más alta. 


**CPU** | **Memoria** | **Disco de caché** | **Tasa de renovación**
 --- | --- | --- | --- 
12 vCPU<br> 2 sockets * 6 núcleos a 2,5 GHz | 24 GB | 1 GB | Hasta 2 TB al día

Configure el servidor de proceso como se indica a continuación:

1. Revise los [requisitos previos](vmware-azure-set-up-process-server-scale.md#prerequisites).
2. Instale el servidor en el [portal](vmware-azure-set-up-process-server-scale.md#install-from-the-ui) o desde la [línea de comandos](vmware-azure-set-up-process-server-scale.md#install-from-the-command-line).
3. Configure las máquinas replicadas para que usen el servidor nuevo. Si ya tiene máquinas que se replican:
    - Puede [migrar](vmware-azure-manage-process-server.md#switch-an-entire-workload-to-another-process-server) toda la carga de trabajo de un servidor de proceso al servidor de proceso nuevo.
    - También puede [migrar](vmware-azure-manage-process-server.md#move-vms-to-balance-the-process-server-load) máquinas virtuales específicas al servidor de proceso nuevo.



## <a name="enable-large-scale-replication"></a>Habilitación de la replicación a gran escala

Después de planear la capacidad e implementar la infraestructura y los componentes necesarios, habilite la replicación para grandes cantidades de máquinas virtuales.

1. Ordene las máquinas en lotes. Puede habilitar la replicación de las máquinas virtuales dentro de un lote y, luego, pasar al lote siguiente.

    - En el caso de las máquinas virtuales de VMware, puede usar el [tamaño recomendado de lote de VM](site-recovery-vmware-deployment-planner-analyze-report.md#recommended-vm-batch-size-for-initial-replication) en el informe de Deployment Planner.
    - En el caso de las máquinas físicas, se recomienda identificar los lotes en función de las máquinas que tienen una cantidad de datos y un tamaño similares y del rendimiento de la red disponible. El objetivo es incluir en lotes las máquinas que probablemente vayan a completar la replicación inicial en la misma cantidad de tiempo.
    
2. Si la actividad de disco de una máquina es alta o supera los límites de Deployment Planner, puede mover los archivos no críticos que no necesita replicar (como los archivos temporales o los volcados de registro) fuera de la máquina. En el caso de las máquinas virtuales de VMware, puede mover estos archivos a un disco aparte y, luego, [excluir dicho disco](vmware-azure-exclude-disk.md) de la replicación.
3. Antes de habilitar la replicación, compruebe que las máquinas cumplen con los [requisitos de replicación](vmware-physical-azure-support-matrix.md#replicated-machines).
4. Configure una directiva de replicación para las [máquinas virtuales de VMware](vmware-azure-set-up-replication.md#create-a-policy) o los [servidores físicos](physical-azure-disaster-recovery.md#create-a-replication-policy).
5. Habilite la replicación de las [máquinas virtuales de VMware](vmware-azure-enable-replication.md) o los [servidores físicos](physical-azure-disaster-recovery.md#enable-replication). Esto pone en marcha la replicación inicial de las máquinas seleccionadas.

## <a name="monitor-your-deployment"></a>Supervisión de la implementación

Después de iniciar la replicación del primer lote de máquinas virtuales, empiece a supervisar la implementación de la siguiente manera:  

1. Asigne un administrador de recuperación ante desastres para supervisar el estado de mantenimiento de las máquinas replicadas.
2. [Supervise los eventos](site-recovery-monitor-and-troubleshoot.md) de la infraestructura y los elementos replicados.
3. [Supervise el estado](vmware-physical-azure-monitor-process-server.md) de los servidores de proceso de escalabilidad horizontal.
4. Suscríbase para recibir [notificaciones por correo electrónico](./site-recovery-monitor-and-troubleshoot.md#subscribe-to-email-notifications) de eventos para supervisarlos de manera más sencilla.
5. Realice [maniobras de recuperación ante desastres](site-recovery-test-failover-to-azure.md) de manera periódica para garantizar que todo funciona según lo esperado.


## <a name="plan-for-large-scale-failovers"></a>Planeamiento de conmutaciones por error a gran escala

En caso de desastre, es posible que tenga que conmutar por error un gran número de máquinas o cargas de trabajo en Azure. Prepárese para este tipo de evento como se indica a continuación.

Puede prepararse por adelantado para una conmutación por error como sigue:

- [Prepare la infraestructura y las VM](#plan-infrastructure-and-vm-connectivity) para que las cargas de trabajo estén disponibles después de la conmutación por error, de manera que los usuarios puedan acceder a las máquinas virtuales de Azure.
- Tenga en cuenta los [límites de conmutación por error](#failover-limits) ya mencionados en este documento. Asegúrese de que las conmutaciones por error se encuentren dentro de estos límites.
- Ejecute [maniobras de recuperación ante desastres](site-recovery-test-failover-to-azure.md) periódicas. Las maniobras ayudan a:
    - Detectar deficiencias en la implementación antes de la conmutación por error.
    - Calcular el RTO integral de las aplicaciones.
    - Calcular el RPO integral de las cargas de trabajo.
    - Identificar conflictos de intervalos de dirección IP.
    - Cuando se ejecuten simulacros, se recomienda no usar redes de producción en estos escenarios y limpiar las conmutaciones por error de prueba después de cada uno.

Para ejecutar una conmutación por error a gran escala, se recomienda lo siguiente:

1. Cree planes de recuperación para la conmutación por error de cargas de trabajo.
    - Cada plan de recuperación puede desencadenar la conmutación por error de hasta 100 máquinas.
    - [Obtenga más información](recovery-plan-overview.md) sobre los planes de recuperación.
2. Agregue scripts de runbook de Azure Automation a los planes de recuperación para automatizar cualquier tarea manual en Azure. Las tareas típicas incluyen la configuración de los equilibradores de carga, la actualización de DNS, etc. [Más información](site-recovery-runbook-automation.md)
2. Antes de realizar la conmutación por error, prepare las máquinas Windows para que puedan cumplir con el entorno de Azure. Los [límites de conmutación por error](#plan-azure-subscriptions-and-quotas) son más altos para las máquinas compatibles. [Obtenga más información](site-recovery-failover-to-azure-troubleshoot.md#failover-failed-with-error-id-170010) sobre los runbooks.
4.  Desencadene la conmutación por error con el cmdlet [Start-AzRecoveryServicesAsrPlannedFailoverJob](/powershell/module/az.recoveryservices/start-azrecoveryservicesasrplannedfailoverjob?view=azps-2.0.0&viewFallbackFrom=azps-1.1.0) de PowerShell en conjunto con un plan de recuperación.



## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Supervisión de Site Recovery](site-recovery-monitor-and-troubleshoot.md)
