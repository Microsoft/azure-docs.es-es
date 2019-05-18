---
title: Configurar la recuperación ante desastres en Azure para un gran número de máquinas virtuales de VMware o servidores físicos con Azure Site Recovery | Microsoft Docs
description: Obtenga información sobre cómo configurar la recuperación ante desastres en Azure para un gran número de máquinas virtuales de VMware locales o servidores físicos con Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 05/14/2019
ms.author: raynew
ms.openlocfilehash: 593a82221ec94e43d50b7382ba89053d5f68864a
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65805927"
---
# <a name="set-up-disaster-recovery-at-scale-for-vmware-vmsphysical-servers"></a>Configurar la recuperación ante desastres a escala para los servidores físicos o máquinas virtuales de VMware

En este artículo se describe cómo configurar la recuperación ante desastres en Azure para un gran número (> 1000) de máquinas virtuales de VMware locales o servidores físicos en el entorno de producción, mediante el [Azure Site Recovery](site-recovery-overview.md) service.


## <a name="define-your-bcdr-strategy"></a>Definir la estrategia de BCDR

Como parte de la continuidad del negocio y la estrategia de recuperación (BCDR), definir objetivos de punto de recuperación (RPO) y objetivos de tiempo de recuperación (RTO) para sus aplicaciones de negocio y las cargas de trabajo. RTO mide la duración de tiempo y el servicio de nivel en el que una aplicación empresarial o un proceso debe ser restaurado y esté disponible, con el fin de evitar problemas de continuidad.
- Site Recovery proporciona replicación continua para máquinas virtuales de VMware y servidores físicos y un [SLA](https://azure.microsoft.com/support/legal/sla/site-recovery/v1_2/) para RTO.
- Como plan de recuperación ante desastres a gran escala de máquinas virtuales de VMware y determinar cuál de los recursos de Azure que necesita, puede especificar un valor RTO que se usará para los cálculos de capacidad.


## <a name="best-practices"></a>Procedimientos recomendados

Algunas prácticas recomendadas generales para la recuperación ante desastres a gran escala. Estos procedimientos recomendados se tratan con más detalle en las secciones del documento.

- **Identificar requisitos de destino**: Calcular out necesidades de capacidad y recursos en Azure antes de configurar la recuperación ante desastres.
- **Plan para los componentes de Site Recovery**: Averiguar qué componentes de Site Recovery (servidor de configuración, servidores de procesos) que necesita para satisfacer su capacidad estimada.
- **Configurar uno o más servidores de procesos de escalado horizontal**: No use el servidor de procesos que se ejecuta de forma predeterminada en el servidor de configuración. 
- **Ejecute las actualizaciones más recientes**: El equipo de Site Recovery publica las nuevas versiones de componentes de Site Recovery de forma periódica, y debe asegurarse de que está ejecutando las versiones más recientes. Para ayudarle a que, realizar un seguimiento de [novedades](site-recovery-whats-new.md) para las actualizaciones, y [habilitar e instalar actualizaciones](service-updates-how-to.md) conforme liberan.
- **Supervisar de forma proactiva**: A medida que se la recuperación ante desastres ponerse en marcha, debe supervisar de forma proactiva el estado y el estado de las máquinas replicadas y los recursos de infraestructura.
- **Maniobras de recuperación ante desastres**: Debe ejecutar maniobras de recuperación ante desastres de forma periódica. Estas no afecta al entorno de producción, pero ayuda a garantizar que conmutan por error a Azure funcionará según lo esperado cuando sea necesario.



## <a name="gather-capacity-planning-information"></a>Recopilar información de planeación de capacidad

Recopilar información sobre su entorno local, para ayudar a evaluar y estimación de que necesidades de su capacidad de destino (Azure).
- Para VMware, ejecución de Deployment Planner para máquinas virtuales de VMware para hacer esto.
- Los servidores físicos, recopile la información manualmente.

### <a name="run-the-deployment-planner-for-vmware-vms"></a>Ejecución de Deployment Planner para máquinas virtuales de VMware

Deployment Planner le ayuda a recopilar información sobre su entorno de VMware locales.

- Ejecución de Deployment Planner durante un período que representa la actividad típica para las máquinas virtuales. Esto generará más precisa las estimaciones y recomendaciones.
- Se recomienda ejecutar Deployment Planner en el equipo del servidor de configuración, ya que la herramienta de planeación calcula el rendimiento del servidor en el que se está ejecutando. [Obtenga más información](site-recovery-vmware-deployment-planner-run.md#get-throughput) sobre cómo medir el rendimiento.
- Si aún no tiene configurado un servidor de configuración:
    - [Obtenga información general sobre](vmware-physical-azure-config-process-server-overview.md) componentes de Site Recovery.
    - [Configurar un servidor de configuración](vmware-azure-deploy-configuration-server.md), con el fin de ejecutar Deployment Planner en él.

A continuación, ejecute la herramienta de planeación de la manera siguiente:

1. [Obtenga información sobre](site-recovery-deployment-planner.md) Deployment Planner. Puede descargar la versión más reciente desde el portal, o [descargarlo directamente](https://aka.ms/asr-deployment-planner).
2. Revise el [requisitos previos](site-recovery-deployment-planner.md#prerequisites) y [actualizaciones más recientes](site-recovery-deployment-planner-history.md) de Deployment Planner y [descargue y extraiga](site-recovery-deployment-planner.md#download-and-extract-the-deployment-planner-tool) la herramienta.
3. [Ejecución de Deployment Planner](site-recovery-vmware-deployment-planner-run.md) en el servidor de configuración.
4. [Generar un informe](site-recovery-vmware-deployment-planner-run.md#generate-report) para resumir las estimaciones y recomendaciones.
5. Analizar la [informe recomendaciones](site-recovery-vmware-deployment-planner-analyze-report.md) y [estimaciones de costos](site-recovery-vmware-deployment-planner-cost-estimation.md).

>[!NOTE]
> De forma predeterminada, la herramienta está configurada para el perfil y genera informes de hasta 1000 máquinas virtuales. Puede cambiar este límite si aumenta el valor de la clave MaxVMsSupported en el archivo ASRDeploymentPlanner.exe.config.

## <a name="plan-target-azure-requirements-and-capacity"></a>Planear la capacidad y requisitos de destino (Azure)

Con las estimaciones recopiladas y recomendaciones, puede planear capacidad y recursos de destino. Si se ha ejecutado la herramienta de planeación de implementación para máquinas virtuales de VMware, puede usar un número de la [informe recomendaciones](site-recovery-vmware-deployment-planner-analyze-report.md#recommendations) para ayudarle.

- **Las máquinas virtuales compatibles**: Use este número para identificar el número de máquinas virtuales que están listos para la recuperación ante desastres en Azure. Recomendaciones sobre el ancho de banda de red y los núcleos de Azure se basan en este número.
- **Ancho de banda de red necesario**: Tenga en cuenta el ancho de banda que necesario para la replicación de máquinas virtuales compatibles. 
    - Al ejecutar la herramienta de planeación especifique el RPO deseado en minutos. Las recomendaciones muestran el ancho de banda necesario para satisfacer esa RPO 100% y el 90% del tiempo. 
    - Las recomendaciones de ancho de banda de red tienen en cuenta el ancho de banda necesario para el número total de servidores de configuración y servidores de procesos que se recomienda en la herramienta de planeación.
- **Núcleos de Azure necesarios**: Tenga en cuenta el número de núcleos que necesita en la región de Azure de destino en función del número de máquinas virtuales compatibles. Si no tiene suficientes núcleos, en la conmutación por error de Site Recovery no podrá crear las máquinas virtuales de Azure necesarios.
- **Tamaño de lote VM recomendado**: El tamaño de lote recomendado se basa en la capacidad de finalizar la replicación inicial para el lote en 72 horas de forma predeterminada, al tiempo que satisface un RPO de 100%. Se puede modificar el valor de hora.

Puede usar estas recomendaciones para planear los recursos de Azure, el ancho de banda de red y el procesamiento por lotes de máquina virtual.

## <a name="plan-azure-subscriptions-and-quotas"></a>Planear las cuotas y suscripciones de Azure

Queremos asegurarse de que las cuotas disponibles en la suscripción de destino son suficientes para controlar la conmutación por error.

**Task** | **Detalles** | **Acción**
--- | --- | ---
**Comprobación de núcleos** | Si los núcleos de la cuota disponible no es igual o superar el número total de destino en el momento de la conmutación por error, se producirá un error en las conmutaciones por error. | Las máquinas virtuales de VMware, compruebe que dispone de suficientes núcleos en la suscripción de destino para cumplir con la recomendación de núcleo de Deployment Planner.<br/><br/> Los servidores físicos, compruebe que núcleos de Azure cumplen las estimaciones manuales.<br/><br/> Para comprobar las cuotas en Azure portal > **suscripción**, haga clic en **uso y cuotas**.<br/><br/> [Obtenga más información](https://docs.microsoft.com/azure/azure-supportability/resource-manager-core-quotas-request) acerca de cómo aumentar las cuotas.
**Compruebe los límites de la conmutación por error** | El número de las conmutaciones por error no debe superar los límites de la conmutación por error de Site Recovery. |  Si las conmutaciones por error superan los límites, puede agregar suscripciones y conmutar por error a varias suscripciones o aumentar la cuota de una suscripción. 


### <a name="failover-limits"></a>Límites de la conmutación por error

Los límites de indican el número de conmutaciones por error que son compatibles con Site Recovery dentro de una hora, suponiendo que los tres discos por máquina.

¿Lo que cumplen Media? Para iniciar una máquina virtual de Azure, Azure requiere algunos controladores estén en estado de inicio del arranque y servicios, como DHCP debe establecerse para iniciarse automáticamente.
- Las máquinas que cumplen ya tendrán estas configuraciones en su lugar.
- Para las máquinas que ejecutan Windows, proactivamente puede comprobar el cumplimiento normativo y hacen compatibles si es necesario. [Más información](site-recovery-failover-to-azure-troubleshoot.md#failover-failed-with-error-id-170010).
- Máquinas Linux solo se convertirán en compatibles en el momento de la conmutación por error.

**¿Equipo cumple con Azure?** | **Límites de máquina virtual de Azure (conmutación por error de disco administrado)**
--- | --- 
Sí | 2000
No | 1000

- Los límites se suponen es mínimos que son otros trabajos en curso en la región de destino para la suscripción.
- Algunas regiones de Azure son más pequeñas y podrían tener límites ligeramente inferiores.

## <a name="plan-infrastructure-and-vm-connectivity"></a>Planear la infraestructura y conectividad de máquina virtual

Después de la conmutación por error a Azure necesita sus cargas de trabajo para que funcione como lo hacían en el entorno local y para permitir que los usuarios tener acceso a las cargas de trabajo que se ejecutan en las máquinas virtuales de Azure.

- [Obtenga más información](site-recovery-active-directory.md#test-failover-considerations) aproximadamente la conmutación por error su infraestructura local de Active Directory o DNS en Azure.
- [Obtenga más información](site-recovery-test-failover-to-azure.md#prepare-to-connect-to-azure-vms-after-failover) acerca de la preparación para conectarse a máquinas virtuales de Azure después de la conmutación por error.



## <a name="plan-for-source-capacity-and-requirements"></a>Planear la capacidad de origen y los requisitos

Es importante disponer de suficientes servidores de configuración y los servidores de procesos de escalado horizontal para satisfacer los requisitos de capacidad. Cuando comience la implementación a gran escala, comenzar con un servidor de configuración y un servidor de procesos de escalado horizontal. A medida que se alcanzan los límites prescritos, agregar servidores adicionales.

>[!NOTE]
> Para las máquinas virtuales de VMware, Deployment Planner realiza algunas recomendaciones acerca de los servidores de configuración y de procesos que necesita. Se recomienda que use las tablas incluidas en los procedimientos siguientes, en lugar de seguir la recomendación de Deployment Planner. 


## <a name="set-up-a-configuration-server"></a>Configurar un servidor de configuración
 
Capacidad del servidor de configuración se ve afectado por el número de máquinas se replican y no por los datos de índice de actividad. Para averiguar si necesita servidores de configuración adicionales, use estos definen los límites de máquina virtual.

**CPU** | **Memoria** | **Disco de caché** | **Replica el límite del equipo**
 --- | --- | --- | ---
8 vCPU<br> 2 sockets * 4 núcleos @ 2,5 Ghz | 16 GB | 600 TB | Hasta 550 máquinas<br> Se supone que cada máquina tiene tres discos de 100 GB.

- Estos límites se basan en un servidor de configuración que se definen mediante una plantilla de OVF.
- Los límites se suponen que no está usando el servidor de procesos que se ejecuta de forma predeterminada en el servidor de configuración.

Si necesita agregar un nuevo servidor de configuración, siga estas instrucciones:

- [Configurar un servidor de configuración](vmware-azure-deploy-configuration-server.md) para la recuperación ante desastres de VM de VMware, mediante una plantilla de OVF.
- [Configurar un servidor de configuración](physical-azure-set-up-source.md) manualmente para los servidores físicos, o para las implementaciones de VMware que no se pueden usar una plantilla de OVF.

Configurar un servidor de configuración, tenga en cuenta que:

- Al configurar un servidor de configuración, es importante tener en cuenta la suscripción y el almacén en el que reside, puesto que estos no deben cambiarse tras la instalación. Si necesita cambiar el almacén, deberá desasociar el servidor de configuración del almacén y vuelva a registrarlo. Esto detiene la replicación de máquinas virtuales en el almacén.
- Si desea configurar un servidor de configuración con varios adaptadores de red, debe hacerlo durante el conjunto de. No se puede hacer esto después del registro del servidor de configuración en el almacén.

## <a name="set-up-a-process-server"></a>Configurar un servidor de procesos

Capacidad del servidor de proceso se ve afectado por las tasas de renovación de datos y no el número de máquinas habilitadas para la replicación.

- Para implementaciones a gran escala siempre debe tener al menos un servidor de procesos de escalado horizontal.
- Para averiguar si necesita servidores adicionales, utilice la siguiente tabla.
- Le recomendamos que agregue un servidor con la especificación de más alto. 


**CPU** | **Memoria** | **Disco de caché** | **Índice de actividad**
 --- | --- | --- | --- 
12 vCPU<br> 2 sockets * 6 núcleos @ 2,5 Ghz | 24 GB | 1 GB | Día hasta 2 TB

Configure el servidor de procesos de la manera siguiente:

1. Revise los [requisitos previos](vmware-azure-set-up-process-server-scale.md#prerequisites).
2. Instalar el servidor en el [portal](vmware-azure-set-up-process-server-scale.md#install-from-the-ui), o desde el [línea de comandos](vmware-azure-set-up-process-server-scale.md#install-from-the-command-line).
3. Configurar las máquinas replicadas para usar el nuevo servidor. Si dispone de las máquinas que replican:
    - También puede [mover](vmware-azure-manage-process-server.md#switch-an-entire-workload-to-another-process-server) una carga de trabajo del servidor de todo el proceso para el nuevo servidor de procesos.
    - Como alternativa, puede [mover](vmware-azure-manage-process-server.md#move-vms-to-balance-the-process-server-load) máquinas virtuales específicas para el nuevo servidor de procesos.



## <a name="enable-large-scale-replication"></a>Habilitar la replicación a gran escala

Después de la capacidad de planear e implementar los componentes necesarios y la infraestructura, habilite la replicación para un gran número de máquinas virtuales.

1. Ordenar las máquinas en lotes. Habilitar la replicación de máquinas virtuales dentro de un lote y, a continuación, pasar a la siguiente lote.

    - Las máquinas virtuales de VMware, puede usar el [recomienda el tamaño de lote de la máquina virtual](site-recovery-vmware-deployment-planner-analyze-report.md#recommended-vm-batch-size-for-initial-replication) en el informe de Deployment Planner.
    - Para las máquinas físicas, se recomienda que identificar lotes basándose en los equipos que tienen un tamaño y cantidad de datos similares y en el rendimiento de red disponible. El objetivo consiste en máquinas de batch que puedan finalizar su replicación inicial en torno a la misma cantidad de tiempo.
    
2. Si la actividad de disco para una máquina es alta o supera los límites en thePlanner de implementación, puede mover los archivos no críticos que no es necesario para la replicación (por ejemplo, los volcados de registro o los archivos temporales) la máquina. Las máquinas virtuales de VMware, puede mover estos archivos en un disco aparte y, a continuación, [excluir ese disco](vmware-azure-exclude-disk.md) de la replicación.
3. Antes de habilitar la replicación, compruebe que los equipos cumplen [los requisitos de replicación](vmware-physical-azure-support-matrix.md#replicated-machines).
4. Configurar una directiva de replicación para [máquinas virtuales de VMware](vmware-azure-set-up-replication.md#create-a-policy) o [servidores físicos](physical-azure-disaster-recovery.md#create-a-replication-policy).
5. Habilitar la replicación de [máquinas virtuales de VMware](vmware-azure-enable-replication.md) o [servidores físicos](physical-azure-disaster-recovery.md#enable-replication). Esto inicia la replicación inicial para las máquinas seleccionadas.

## <a name="monitor-your-deployment"></a>Supervisión de la implementación

Después de iniciar la replicación para el primer lote de máquinas virtuales, comenzar a supervisar la implementación como se indica a continuación:  

1. Asigna un administrador de recuperación ante desastres para supervisar el estado de mantenimiento de máquinas replicadas.
2. [Supervisar eventos](site-recovery-monitor-and-troubleshoot.md) para los elementos replicados y la infraestructura.
3. [Supervisar el estado](vmware-physical-azure-monitor-process-server.md) de los servidores de procesos de escalado horizontal.
4. Suscríbase para obtener [notificaciones por correo electrónico](https://docs.microsoft.com/azure/site-recovery/site-recovery-monitor-and-troubleshoot.md#subscribe-to-email-notifications) para los eventos, para facilitar la supervisión.
5. Llevar a cabo regular [maniobras de recuperación ante desastres](site-recovery-test-failover-to-azure.md)para asegurarse de que todo funciona según lo previsto.


## <a name="plan-for-large-scale-failovers"></a>Plan para las conmutaciones por error a gran escala

En el caso de desastre, es posible que deberá conmutar por error un gran número de equipos y cargas de trabajo en Azure. Prepararse para este tipo de evento, como se indica a continuación.

Puede preparar de antemano para conmutación por error como sigue:

- [Preparar la infraestructura y las máquinas virtuales](#plan-infrastructure-and-vm-connectivity) para que las cargas de trabajo estará disponibles tras la conmutación por error, y para que los usuarios puedan acceder a las máquinas virtuales de Azure.
- Tenga en cuenta la [los límites de la conmutación por error](#failover-limits) anteriormente en este documento. Asegúrese de que las conmutaciones por error se encuentran dentro de estos límites.
- Ejecute regular [maniobras de recuperación ante desastres](site-recovery-test-failover-to-azure.md). Maniobras de ayudan para:
    - Detectar deficiencias en la implementación antes de la conmutación por error.
    - Calcular el RTO to-end para sus aplicaciones.
    - Calcular el RPO de extremo a otro para las cargas de trabajo.
    - Identificar los conflictos de intervalo de direcciones IP.
    - Cuando ejecute aumenta, se recomienda que no usar las redes de producción para aumenta, evite usar los mismos nombres de subred en las redes de producción y prueba y limpiar las conmutaciones por error de prueba después de cada exploración en profundidad.

Para ejecutar una conmutación por error a gran escala, se recomienda lo siguiente:

1. Crear planes de recuperación para conmutación por error de carga de trabajo.
    - Cada plan de recuperación puede desencadenar una conmutación por error de hasta 50 máquinas.
    - [Obtenga más información](recovery-plan-overview.md) sobre los planes de recuperación.
2. Agregar scripts de runbook de Azure Automation en planes de recuperación para automatizar las tareas manuales necesarias en Azure. Las tareas típicas incluyen la configuración de equilibradores de carga, actualizar DNS etcetera. [Más información](site-recovery-runbook-automation.md)
2. Antes de la conmutación por error, prepare las máquinas de Windows para que cumplen con el entorno de Azure. [Los límites de la conmutación por error](#plan-azure-subscriptions-and-quotas) son mayores para las máquinas que cumplen. [Obtenga más información](site-recovery-failover-to-azure-troubleshoot.md#failover-failed-with-error-id-170010) sobre los runbooks.
4.  Desencadenar la conmutación por error con el [inicio AzRecoveryServicesAsrPlannedFailoverJob](https://docs.microsoft.com/powershell/module/az.recoveryservices/start-azrecoveryservicesasrplannedfailoverjob?view=azps-2.0.0&viewFallbackFrom=azps-1.1.0) cmdlet de PowerShell, junto con un plan de recuperación.



## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Monitor de recuperación de sitios](site-recovery-monitor-and-troubleshoot.md)
