---
title: Recuperación ante desastres de VMware con Azure Site Recovery
description: En este artículo se proporciona información general acerca de la recuperación ante desastres de máquinas virtuales de VMware en Azure mediante el servicio Azure Site Recovery.
author: rayne-wiselman
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/12/2019
ms.author: raynew
ms.openlocfilehash: dfbdff01064b483085233ece47d1d3b635b68743
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87021467"
---
# <a name="about-disaster-recovery-of-vmware-vms-to-azure"></a>Acerca de la recuperación ante desastres de máquinas virtuales de VMware en Azure

En este artículo se proporciona información general acerca de la recuperación ante desastres de máquinas virtuales de VMware locales en Azure mediante el servicio [Azure Site Recovery](site-recovery-overview.md).

## <a name="what-is-bcdr"></a>¿Qué es BCDR?

Una estrategia de continuidad empresarial y recuperación ante desastres (BCDR) le ayuda a mantener su negocio en funcionamiento. Durante el tiempo de inactividad planeado e interrupciones inesperadas, BCDR mantiene los datos protegidos y disponibles y garantiza que las aplicaciones sigan ejecutándose. Además de las características BCDR de la plataforma, como el emparejamiento regional y el almacenamiento de alta disponibilidad, Azure proporciona Recovery Services como una parte integral de la solución de BCDR. Recovery Services incluye: 

- [Azure Backup](../backup/backup-overview.md) realiza una copia de seguridad de datos locales y de la máquina virtual de Azure. Puede realizar copias de seguridad de archivos y carpetas, cargas de trabajo específicas o toda una máquina virtual. 
- [Azure Site Recovery](site-recovery-overview.md) proporciona resistencia y recuperación ante desastres para aplicaciones y cargas de trabajo que se ejecutan en máquinas locales o máquinas virtuales de IaaS de Azure. Site Recovery organiza la replicación y controla la conmutación por error en Azure cuando se producen interrupciones. También controla la recuperación desde Azure a su sitio primario. 

> [!NOTE]
> Site Recovery no mueve ni almacena los datos de los clientes fuera de la región de destino, en la que se ha configurado la recuperación ante desastres para las máquinas de origen. Los clientes pueden seleccionar un almacén de Recovery Services de otra región si así lo deciden. El almacén de Recovery Services contiene metadatos, pero ningún dato real de los clientes.

## <a name="how-does-site-recovery-do-disaster-recovery"></a>¿Cómo realiza la recuperación ante desastres Site Recovery?

1. Después de preparar Azure y su sitio local, configure y habilite la replicación para las máquinas locales.
2. Site Recovery organiza la replicación inicial de la máquina según la configuración de la directiva.
3. Tras la recuperación inicial, Site Recovery replica los cambios diferenciales en Azure. 
4. Cuando todo esté replicando según lo previsto, ejecute un simulacro de recuperación ante desastres.
    - El simulacro le ayuda a garantizar que la conmutación por error funcione según lo esperado cuando surja una necesidad real.
    - El simulacro realiza una conmutación por error de prueba sin afectar a su entorno de producción.
5. Si se produce una interrupción, ejecute una conmutación por error completa en Azure. Puede conmutar por error una sola máquina o puede crear un plan de recuperación que conmute por error varias máquinas al mismo tiempo.
6. En una conmutación por error, se crean máquinas virtuales de Azure a partir de los datos de la máquina virtual de discos administrados o cuentas de almacenamiento. Los usuarios pueden seguir teniendo acceso a aplicaciones y cargas de trabajo desde la máquina virtual de Azure.
7. Cuando el sitio local está disponible de nuevo, puede realizar una conmutación por recuperación desde Azure.
8. Después de conmutar por recuperación y estar trabajando desde el sitio primario una vez más, vuelva a iniciar la replicación de máquinas virtuales locales en Azure.


## <a name="how-do-i-know-if-my-environment-is-suitable-for-disaster-recovery-to-azure"></a>¿Cómo se puede saber si mi entorno es adecuado para la recuperación ante desastres en Azure?

Site Recovery puede replicar cualquier carga de trabajo que se ejecute en una máquina virtual de VMware o servidor físico compatible. A continuación se indican los aspectos que debe comprobar en el entorno:

- Si va a replicar máquinas virtuales de VMware, ¿ejecuta las versiones correctas de los servidores de virtualización de VMware? [Haga clic aquí](vmware-physical-azure-support-matrix.md#on-premises-virtualization-servers).
- ¿Las máquinas que desea replicar ejecutan un sistema operativo compatible? [Haga clic aquí](vmware-physical-azure-support-matrix.md#replicated-machines).
- En el caso de la recuperación ante desastres de Linux, ¿las máquinas ejecutan un almacenamiento de invitados o sistema de archivos compatibles? [Haga clic aquí](vmware-physical-azure-support-matrix.md#linux-file-systemsguest-storage)
- ¿Las máquinas virtuales que desea replicar cumplen con los requisitos de Azure? [Haga clic aquí](vmware-physical-azure-support-matrix.md#azure-vm-requirements).
- ¿La configuración de red es compatible? [Haga clic aquí](vmware-physical-azure-support-matrix.md#network).
- ¿La configuración de almacenamiento es compatible? [Haga clic aquí](vmware-physical-azure-support-matrix.md#storage).


## <a name="what-do-i-need-to-set-up-in-azure-before-i-start"></a>¿Qué es necesario configurar en Azure antes de empezar?

En Azure, deberá preparar lo siguiente:

1. Compruebe que la cuenta de Azure tiene permisos para crear máquinas virtuales en Azure.
2. Cree una red de Azure a la que se conectarán las máquinas virtuales de Azure cuando se creen desde cuentas de almacenamiento o discos administrados después de la conmutación por error.
3. Configure un almacén de Azure Recovery Services para Site Recovery. El almacén se encuentra en Azure Portal y se usa para implementar, configurar, organizar, supervisar y solucionar problemas de implementación de Site Recovery.

*¿Necesita más ayuda?*

Obtenga información sobre cómo configurar Azure mediante la [comprobación de su cuenta](tutorial-prepare-azure.md#verify-account-permissions), la creación de una [red](tutorial-prepare-azure.md#set-up-an-azure-network) y la configuración de un [almacén](tutorial-prepare-azure.md#create-a-recovery-services-vault).



## <a name="what-do-i-need-to-set-up-on-premises-before-i-start"></a>¿Qué es necesario configurar en el entorno local antes de empezar?

Esto es lo que se necesita en el entorno local:

1. Es necesario configurar un par de cuentas:

    - Si va a replicar máquinas virtuales de VMware, se necesita una cuenta para que Site Recovery obtenga acceso a los hosts de vSphere ESXi o vCenter Server para detectar automáticamente las máquinas virtuales.
    - Se necesita una cuenta para instalar al agente de Mobility Service de Site Recovery en cada máquina física o máquina virtual que desee replicar.

2. Debe comprobar la compatibilidad de la infraestructura de VMware si no lo ha hecho anteriormente.
3. Asegúrese de que puede conectarse a máquinas virtuales de Azure después de una conmutación por error. Debe configurar RDP en máquinas Windows locales o SSH en máquinas Linux.

*¿Necesita más ayuda?*
- Prepare cuentas para la [detección automática](vmware-azure-tutorial-prepare-on-premises.md#prepare-an-account-for-automatic-discovery) y para la [instalación de Mobility Service](vmware-azure-tutorial-prepare-on-premises.md#prepare-an-account-for-mobility-service-installation).
- [Compruebe](vmware-azure-tutorial-prepare-on-premises.md#check-vmware-requirements) que la configuración de VMware es compatible.
- [Prepárela](vmware-azure-tutorial-prepare-on-premises.md#prepare-to-connect-to-azure-vms-after-failover) para poder conectarse a Azure después de una conmutación por error.
- Si desea obtener más ayuda detallada acerca de cómo configurar el direccionamiento IP para máquinas virtuales de Azure tras la conmutación por error, [lea este artículo](concepts-on-premises-to-azure-networking.md).

## <a name="how-do-i-set-up-disaster-recovery"></a>¿Cómo se configura la recuperación ante desastres?

Tras prepara la infraestructura local y de Azure, puede configurar la recuperación ante desastres.

1. Para comprender los componentes que deberá implementar, revise la [arquitectura de VMware a Azure](vmware-azure-architecture.md)y la [arquitectura física a Azure](physical-azure-architecture.md). Hay una serie de componentes, por lo que es importante comprender cómo encajan.
2. **Entorno de origen**: como primer paso de la implementación, debe configurar el entorno de origen de replicación. Debe especificar aquello que desea replicar y la ubicación donde se va a realizar la replicación.
3. **Servidor de configuración**: deberá configurar un servidor de configuración en el entorno de origen local:
    - El servidor de configuración es una máquina local. En el caso de la recuperación ante desastres de VMware, se recomienda implementarla como una máquina virtual de VMware que pueda implementarse desde una plantilla de OVF descargable.
    - El servidor de configuración coordina la comunicación entre Azure y el entorno local.
    - Otro par de componentes se ejecutan en la máquina del servidor de configuración.
        - El servidor de procesos recibe, optimiza y envía datos de replicación a la cuenta de almacenamiento en caché de Azure. También controla la instalación automática de Mobility Service en las máquinas que desea replicar, además de realizar la detección automática de máquinas virtuales en servidores de VMware.
        - El servidor de destino maestro controla los datos de replicación durante la conmutación por recuperación desde Azure.
    - La configuración incluye el registro del servidor de configuración en el almacén, la descarga de MySQL Server y VMware PowerCLI y la especificación de las cuentas creadas para la detección automática y la instalación de Mobility Service.
4. **Entorno de destino**: debe configurar el entorno de destino de Azure mediante la especificación de la suscripción de Azure y la configuración de red.
5. **Directiva de replicación**: debe especificar cómo realizar la replicación. La configuración incluye la frecuencia con que se crean y almacenan los puntos de recuperación, así como si deben crearse instantáneas coherentes con la aplicación.
6. **Habilite la replicación**. Debe habilitar la replicación de máquinas locales. Si ha creado una cuenta para instalar Mobility Service, se instalará cuando se habilite la replicación de una máquina. 

*¿Necesita más ayuda?*

- Para ver un tutorial rápido de estos pasos, puede probar nuestro [tutorial de VMware](vmware-azure-tutorial.md) y [tutorial de servidor físico](physical-azure-disaster-recovery.md).
- [Obtenga más información](vmware-azure-set-up-source.md) acerca de cómo configurar el entorno de origen.
- [Obtenga información sobre](vmware-azure-deploy-configuration-server.md) los requisitos del servidor de configuración y acerca de cómo configurar el servidor de configuración con una plantilla OVF para replicación de VMware. Si por algún motivo no puede usar una plantilla o está replicando servidores físicos, [siga estas instrucciones](physical-azure-set-up-source.md#set-up-the-source-environment).
- [Obtenga más información](vmware-azure-set-up-target.md) sobre la configuración de destino.
- [Obtenga más información](vmware-azure-set-up-replication.md) acerca de cómo configurar una directiva de replicación.
- [Obtenga más información](vmware-azure-enable-replication.md) acerca de cómo habilitar la replicación y [excluir](vmware-azure-exclude-disk.md) los discos de esta.


## <a name="something-went-wrong-how-do-i-troubleshoot"></a>Ha habido algún error, ¿cómo lo soluciono?

- Como primer paso, intente [supervisar la implementación](site-recovery-monitor-and-troubleshoot.md) para comprobar el estado de los elementos replicados, los trabajos y los problemas de infraestructura e identifique los errores.
- Si no se puede completar la replicación inicial o la replicación en curso no funciona según lo previsto, [revise este artículo](vmware-azure-troubleshoot-replication.md) para errores comunes y sugerencias para solucionar problemas.
- Si tiene problemas con la instalación automática de Mobility Service en máquinas que desea replicar, revise los errores comunes en [este artículo](vmware-azure-troubleshoot-push-install.md).
- Si la conmutación por error no funciona según lo previsto, compruebe los errores comunes en [este artículo](site-recovery-failover-to-azure-troubleshoot.md).
- Si la conmutación por recuperación no funciona, compruebe si el problema aparece en [este artículo](vmware-azure-troubleshoot-failback-reprotect.md).



## <a name="next-steps"></a>Pasos siguientes

Con la replicación ahora en su lugar, debería [ejecutar un simulacro de recuperación ante desastres](tutorial-dr-drill-azure.md) para asegurarse de que la conmutación por error funciona según lo previsto. 
