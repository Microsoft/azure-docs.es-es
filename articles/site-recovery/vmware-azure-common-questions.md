---
title: 'Preguntas comunes: recuperación ante desastres de VMware a Azure con Azure Site Recovery | Microsoft Docs'
description: En este artículo se resume comunes preguntas fir recuperación de desastres de máquinas virtuales de VMware locales en Azure con Azure Site Recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
services: site-recovery
ms.date: 05/10/2019
ms.topic: conceptual
ms.author: raynew
ms.openlocfilehash: 767f087910136ce558d71602a1c4c632090168cd
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540625"
---
# <a name="common-questions---vmware-to-azure-replication"></a>Preguntas frecuentes: replicación de VMware en Azure

Este artículo proporciona respuestas a preguntas habituales al implementar la recuperación ante desastres de máquinas virtuales de VMware locales en Azure. 

## <a name="general"></a>General 
### <a name="what-do-i-need-for-vmware-vm-disaster-recovery"></a>¿Qué es necesario para la recuperación ante desastres de VM de VMware?

[Obtenga información sobre](vmware-azure-architecture.md) los componentes implicados en la recuperación ante desastres de máquinas virtuales de VMware. 

### <a name="can-i-use-site-recovery-to-migrate-vmware-vms-to-azure"></a>¿Puedo usar Site Recovery para migrar máquinas virtuales de VMware a Azure?

Sí, además de utilizar Site Recovery para configurar la recuperación ante desastres completa para las máquinas virtuales de VMware, también puede usar Site Recovery para migrar máquinas virtuales de VMware locales a Azure. En este escenario, replica máquinas virtuales de VMware locales en Azure Storage. Luego, conmuta por error desde el entorno local a Azure. Después de la conmutación por error, las aplicaciones y cargas de trabajo están disponibles y se ejecutan en máquinas virtuales de Azure. El proceso es similar a la configuración de recuperación ante desastres completa, salvo que en una migración no se puede conmutar por recuperación desde Azure.


### <a name="does-my-azure-account-need-permissions-to-create-vms"></a>¿La cuenta de Azure necesita permisos para crear máquinas virtuales?
Si es administrador de una suscripción, ya tiene los permisos de replicación que necesita. Si no es así, necesita permisos para crear una máquina virtual de Azure en el grupo de recursos y especifique al configurar Site Recovery y los permisos para escribir en la cuenta de almacenamiento seleccionada o administradas basadas en disco en la configuración de red virtual. [Más información](site-recovery-role-based-linked-access-control.md#permissions-required-to-enable-replication-for-new-virtual-machines).

### <a name="what-applications-can-i-replicate"></a>¿Qué aplicaciones se pueden replicar?
Puede replicar cualquier aplicación o carga de trabajo en ejecución en una máquina virtual de VMware que cumpla con los [requisitos de replicación](vmware-physical-azure-support-matrix.md##replicated-machines).
- Site Recovery proporciona compatibilidad para la replicación compatible con aplicaciones, para que estas se puedan conmutar por error y por recuperación a un estado inteligente.
- Site Recovery se integra con aplicaciones de Microsoft como SharePoint, Exchange, Dynamics, SQL Server y Active Directory; además, colabora estrechamente con los principales proveedores, como Oracle, SAP, IBM y Red Hat.
- [Obtenga más información](site-recovery-workload.md) acerca de la protección de la carga de trabajo.

### <a name="can-i-use-a-guest-os-server-license-on-azure"></a>¿Puedo usar una licencia de servidor de sistema operativo invitado en Azure?
Sí, los clientes de Microsoft Software Assurance pueden usar la [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/) para ahorrar en los costos de licencia de **máquinas Windows Server** que se migran a Azure o usar Azure para la recuperación ante desastres.

## <a name="security"></a>Seguridad

### <a name="what-access-does-site-recovery-need-to-vmware-servers"></a>¿Qué acceso a los servidores de VMware necesita Site Recovery?
Site Recovery necesita acceso a los servidores de VMware para:

- Configurar una VM de VMware que ejecuta el servidor de configuración de Site Recovery
- Detectar automáticamente máquinas virtuales para su replicación. 


### <a name="what-access-does-site-recovery-need-to-vmware-vms"></a>¿Qué acceso a las máquinas virtuales de VMware necesita Site Recovery?

- Para poder realizar la replicación, una máquina virtual de VMware debe tener Site Recovery Mobility Service instalado y en ejecución. Puede implementar manualmente la herramienta o especificar que Site Recovery debe realizar una instalación de inserción del servicio cuando habilite la replicación de una máquina virtual. 
- Durante la replicación, las máquinas virtuales se comunican con Site Recovery como se indica a continuación:
    - Las máquinas virtuales se comunican con el servidor de configuración en el puerto HTTPS 443 para la administración de la replicación.
    - Las máquinas virtuales envían los datos de replicación al servidor de procesos en el puerto HTTPS 9443 (se puede modificar).
    - Si habilita la coherencia entre varias máquinas virtuales, estas se comunican entre sí a través del puerto 20004.


### <a name="is-replication-data-sent-to-site-recovery"></a>¿Los datos de replicación se envían a Site Recovery?
No, Site Recovery no intercepta los datos replicados ni tiene información sobre lo que se ejecuta en las máquinas virtuales. Los datos de replicación se intercambian entre los hipervisores de VMware y Azure Storage. Site Recovery no tiene capacidad para interceptar los datos. Únicamente se envían los metadatos necesarios para coordinar la replicación y la conmutación por error al servicio Site Recovery.  

Site Recovery está certificado según la norma ISO 27001:2013, 27018, además de HIPAA y DPA, y está completando sus evaluaciones de SOC2 y FedRAMP JAB.


## <a name="pricing"></a>Precios
### <a name="how-can-i-calculate-approximate-charges-for-vmware-disaster-recovery"></a>¿Cómo puedo calcular cargos aproximados de VMware y recuperación ante desastres?

Puede usar el [Calculadora de precios](https://aka.ms/asr_pricing_calculator) para estimar los costos mientras usa Site Recovery.

Para la estimación detallada en los costos, ejecute la herramienta deployment planner para [VMware](https://aka.ms/siterecovery_deployment_planner)y usar el [informe de estimación de costos](https://aka.ms/asr_DP_costreport).

### <a name="is-there-any-difference-in-cost-between-replicating-to-storage-or-directly-to-managed-disks"></a>¿Hay alguna diferencia en el costo entre replicar al almacenamiento o directamente a discos administrados?

Discos administrados se cobran ligeramente diferentes de las cuentas de almacenamiento. [Obtenga más información](https://azure.microsoft.com/pricing/details/managed-disks/) sobre los precios de disco administrado.

## <a name="mobility-service"></a>Mobility Service

### <a name="where-can-i-find-the-mobility-service-installers"></a>¿Dónde puedo encontrar los instaladores de Mobility Service?
Los instaladores se encuentran en el **%ProgramData%\ASR\home\svsystems\pushinstallsvc\repository** carpeta en el servidor de configuración.

## <a name="how-do-i-install-the-mobility-service"></a>¿Cómo instalo Mobility Service?
Instalar en cada máquina virtual que desea replicar, mediante una serie de métodos:
- [Instalación de inserción](vmware-physical-mobility-service-overview.md#push-installation)
- [Instalación manual](vmware-physical-mobility-service-overview.md#install-mobility-agent-through-ui) desde la interfaz de usuario o Powershell.
- Implementación mediante una herramienta de implementación como [System Center Configuration Manager](vmware-azure-mobility-install-configuration-mgr.md).

## <a name="managed-disks"></a>Discos administrados

### <a name="where-does-site-recovery-replicate-data-to"></a>¿En Site Recovery replicar datos a?

Site Recovery replica servidores físicos y máquinas virtuales de VMware locales a managed disks en Azure.
- El servidor de procesos de Site Recovery escribe los registros de replicación en una cuenta de almacenamiento de caché en la región de destino.
- Estos registros se utilizan para crear puntos de recuperación en Azure discos que tienen el prefijo de asrseeddisk administrados.
- Cuando se produce conmutación por error, el punto de recuperación que seleccione se utiliza para crear un nuevo disco administrado de destino. Este disco administrado está conectado a la máquina virtual en Azure.
- Las máquinas virtuales que anteriormente se han replicado en una cuenta de almacenamiento (antes de marzo de 2019) no se ven afectadas.


### <a name="can-i-replicate-new-machines-to-storage-accounts"></a>¿Puedo replicar nuevas máquinas en cuentas de almacenamiento?

No, a partir de marzo de 2019, en el portal, puede replicar únicamente a Azure managed disks. 

Replicación de máquinas virtuales nuevas a una cuenta de almacenamiento solo está disponible mediante PowerShell o la API de REST (versión 2018-01-10 ó 10-08-2016).

### <a name="what-are-the-benefits-in-replicating-to-managed-disks"></a>¿Cuáles son las ventajas en la replicación de discos administrados?

[Obtenga información sobre cómo](https://azure.microsoft.com/blog/simplify-disaster-recovery-with-managed-disks-for-vmware-and-physical-servers/) Site Recovery simplifica la recuperación ante desastres con discos administrados.


### <a name="can-i-change-the-managed-disk-type-after-machine-is-protected"></a>¿Puedo cambiar el tipo de disco administrado después de proteger el equipo?

Sí, le resultará muy fácil [cambiar el tipo de disco administrado](https://docs.microsoft.com/azure/virtual-machines/windows/convert-disk-storage) para replicaciones en curso. Antes de cambiar el tipo, asegúrese de que ninguna dirección URL de SAS se genera en el disco administrado. Vaya al recurso de disco administrado en el portal de Azure y compruebe si tiene un banner de la dirección URL de SAS en la hoja de información general. Si está presente, haga clic para cancelar la exportación en curso. Una vez hecho, cambie el tipo del disco en los próximos minutos. Sin embargo, si cambia el tipo de disco administrado, espere de puntos de recuperación nuevo para generarse mediante Azure Site Recovery. Use los nuevos puntos de recuperación para cualquier prueba de conmutación por error o conmutación por error en el futuro.

### <a name="can-i-switch-replication-from-managed-disks-to-unmanaged-disks"></a>¿Puedo cambiar la replicación de discos administrados a discos no administrados?

No, cambia de administrado a no administrado no se admite.

## <a name="replication"></a>Replicación


### <a name="what-are-the-replicated-vm-requirements"></a>¿Cuáles son los requisitos de las máquinas virtuales replicadas?

[Obtenga más información](vmware-physical-azure-support-matrix.md##replicated-machines) sobre los requisitos de servidor de máquina virtual de VMware y físicas y soporte técnico.

### <a name="how-often-can-i-replicate-to-azure"></a>¿Con qué frecuencia se puede replicar en Azure?
La replicación es continua cuando se replican máquinas virtuales de VMware en Azure.


### <a name="can-i-extend-replication"></a>¿Se puede extender la replicación?
No se admite la replicación extendida o encadenada. Solicite esta característica en el [foro de comentarios](https://feedback.azure.com/forums/256299-site-recovery/suggestions/6097959).

### <a name="can-i-do-an-offline-initial-replication"></a>¿Se puede realizar una replicación inicial sin conexión?
No es una opción admitida. Solicite esta característica en el [foro de comentarios](https://feedback.azure.com/forums/256299-site-recovery/suggestions/6227386-support-for-offline-replication-data-transfer-from).


### <a name="what-is-asrseeddisk"></a>¿Qué es asrseeddisk?
Para cada disco de origen, los datos se replican en un disco administrado en Azure. Este disco tiene el prefijo asrseeddisk. Almacena la copia del disco de origen y todas las instantáneas de punto de recuperación.

### <a name="can-i-exclude-disks-from-replication"></a>¿Se puede excluir discos de la replicación?
Sí, puede excluir discos.

### <a name="can-i-replicate-vms-with-dynamic-disks"></a>¿Se pueden replicar máquinas virtuales con discos dinámicos?
Los discos dinámicos se pueden replicar. El disco del sistema operativo debe ser un disco básico.

### <a name="if-i-use-replication-groups-for-multi-vm-consistency-can-i-add-a-new-vm-to-an-existing-replication-group"></a>Si utilizo grupos de replicación para mantener la coherencia con varias máquinas virtuales, ¿puedo agregar una nueva máquina virtual a un grupo de replicación existente?
Sí, puede agregar nuevas máquinas virtuales a un grupo de replicación existente al habilitar la replicación para ellas.
- No se puede agregar una máquina virtual a un grupo de replicación existente una vez iniciada la replicación.
- No se puede crear un grupo de replicación para máquinas virtuales existentes.

### <a name="can-i-modify-vms-that-are-replicating-by-adding-or-resizing-disks"></a>¿Puedo modificar máquinas virtuales que se replican mediante la adición o el cambio de tamaño de discos?

Para la replicación de VMware en Azure, puede modificar el tamaño del disco. Si desea agregar discos nuevos, debe agregar el disco y volver a habilitar la protección de la máquina virtual.

### <a name="can-i-migrate-on-premises-machines-to-a-new-vcenter-server-without-impacting-ongoing-replication"></a>¿Puedo migrar máquinas locales a un nuevo servidor de vCenter sin afectar a la replicación en curso?
No, el cambio de instancia de Vcenter o migración afectará a la replicación en curso. Deberá configurar Site Recovery con el nuevo vCenter Server y habilitar la replicación de máquinas de nuevo.

### <a name="can-i-replicate-to-a-cachetarget-storage-account-which-has-a-vnet-with-azure-storage-firewalls-configured-on-it"></a>¿Puedo replicar en una cuenta de almacenamiento de destino o la memoria caché que tiene una red virtual (con firewalls de Azure storage) configurada en él?
No, Site Recovery no admite la replicación en el almacenamiento en la red virtual.





## <a name="component-upgrade"></a>Actualización del componente

### <a name="my-version-of-the-mobility-services-agent-or-configuration-server-is-old-and-my-upgrade-failed-what-do-i-do"></a>Mi versión de que el servidor de agente o la configuración de servicios de movilidad es antigua y Mi error de actualización. ¿Qué puedo hacer?  

Site Recovery sigue el modelo de soporte técnico de n-4. [Obtenga más información](https://aka.ms/asr_support_statement) sobre cómo actualizar de versiones muy anteriores.

### <a name="where-can-i-find-the-release-notesupdate-rollups-of-azure-site-recovery"></a>¿Dónde puedo encontrar las notas de la versión o los paquetes acumulativos de actualización de Azure Site Recovery?

[Obtenga información sobre](site-recovery-whats-new.md) acerca de nuevas actualizaciones y [obtener información de consolidado](service-updates-how-to.md).

### <a name="where-can-i-find-upgrade-information-for-disaster-recovery-to-azure"></a>¿Dónde puedo encontrar información de actualización para la recuperación ante desastres en Azure?

[Obtenga información sobre](https://aka.ms/asr_vmware_upgrades) actualizar.

## <a name="do-i-need-to-reboot-source-machines-for-each-upgrade"></a>¿Es necesario reiniciar las máquinas de origen para cada actualización?

Aunque se recomienda, no es obligatorio para cada actualización. [Más información](https://aka.ms/asr_vmware_upgrades).


## <a name="configuration-server"></a>Servidor de configuración

### <a name="what-does-the-configuration-server-do"></a>¿Qué hace el servidor de configuración?

El servidor de configuración ejecuta los componentes locales de Site Recovery, incluidos:
- El servidor de configuración propia que coordina las comunicaciones entre el entorno local y Azure y administra la replicación de datos.
- El servidor de procesos que actúa como puerta de enlace de replicación. Recibe datos de replicación, los optimiza con almacenamiento en caché, compresión y cifrado, y los envía a Azure Storage. El servidor de procesos también una instalación de inserción de Mobility Service en máquinas virtuales, realiza y la detección automática de máquinas virtuales de VMware locales.
- El servidor de destino maestro que controla los datos de replicación durante la conmutación por recuperación desde Azure.

[Más información](vmware-azure-architecture.md) acerca de los componentes y procesos del servidor de configuración.

### <a name="where-do-i-set-up-the-configuration-server"></a>¿Dónde configuro el servidor de configuración?
Necesita una sola máquina virtual de VMware local de alta disponibilidad para el servidor de configuración. Recuperación ante desastres de servidores físicos, puede instalar al servidor de configuración de una máquina física.

### <a name="what-do-i-need-for-the-configuration-server"></a>¿Qué es necesario para el servidor de configuración?

Revise los [requisitos previos](vmware-azure-deploy-configuration-server.md#prerequisites).

### <a name="can-i-manually-set-up-the-configuration-server-instead-of-using-a-template"></a>¿Puedo configurar manualmente el servidor de configuración en lugar de usar una plantilla?
Se recomienda [crear la máquina virtual del servidor de configuración](vmware-azure-deploy-configuration-server.md) con la versión más reciente de la plantilla OVF. Si por algún motivo no puede, por ejemplo no tiene acceso al servidor de VMware, puede [descargar](physical-azure-set-up-source.md) el archivo de instalación del portal y configurar el servidor de configuración.

### <a name="can-a-configuration-server-replicate-to-more-than-one-region"></a>¿Un servidor de configuración se puede replicar en más de una región?
No. Para ello, necesita un servidor de configuración en cada región.

### <a name="can-i-host-a-configuration-server-in-azure"></a>¿Puedo hospedar un servidor de configuración en Azure?
Si bien es posible, la máquina virtual de Azure que ejecuta el servidor de configuración necesitaría comunicarse con las máquinas virtuales y la infraestructura de VMware locales. Esto agrega latencia y afecta a la replicación en curso.

### <a name="how-do-i-update-the-configuration-server"></a>¿Cómo actualizo el servidor de configuración?

[Obtenga información sobre](vmware-azure-manage-configuration-server.md#upgrade-the-configuration-server) cómo actualizar el servidor de configuración.
- Puede encontrar la información de actualización más reciente en el [página actualizaciones de Azure](https://azure.microsoft.com/updates/?product=site-recovery).
- Puede descargar la versión más reciente desde el portal. Como alternativa, puede descargar directamente la versión más reciente del servidor de configuración de la [Microsoft Download Center](https://aka.ms/asrconfigurationserver).
- Si su versión es anterior a la versión actual de más de cuatro versiones, consulte nuestra [admite instrucción](https://aka.ms/asr_support_statement) para obtener orientación.

### <a name="should-i-back-up-the-configuration-server"></a>¿Debo hacer copias de seguridad del servidor de configuración?
Se recomienda programar copias de seguridad periódicas del servidor de configuración.
- Para la conmutación por recuperación correcta, la máquina virtual que se ha conmutado debe existir en la base de datos del servidor de configuración.
- El servidor de configuración debe ser en ejecución y en un estado conectado.
- [Obtenga más información](vmware-azure-manage-configuration-server.md) sobre tareas comunes de administración de servidor de configuración.

### <a name="when-im-setting-up-the-configuration-server-can-i-download-and-install-mysql-manually"></a>Cuando estoy configurando el servidor de configuración, ¿puedo descargar e instalar MySQL manualmente?

Sí. Descargue MySQL y colóquelo en la carpeta **C:\Temp\ASRSetup**. Después, instálelo manualmente. Cuando configure la máquina virtual del servidor de configuración y acepte los términos, MySQL aparecerá como **Ya instalado**en **Descargar e instalar**.

### <a name="can-i-avoid-downloading-mysql-but-let-site-recovery-install-it"></a>¿Se puedo evitar descargar MySQL pero dejar que Site Recovery lo instale?

Sí. Descargue el instalador de MySQL y colóquelo en la carpeta **C:\Temp\ASRSetup**.  Al configurar la máquina virtual del servidor de configuración, acepte los términos y haga clic en **descargue e instale**. El portal usa el programa de instalación que se hayan agregado para instalar MySQL.
 
### <a name="can-i-use-the-configuration-server-vm-for-anything-else"></a>¿Se puede usar la VM del servidor de configuración para cualquier otra cosa?
No, solo se debe usar la máquina virtual para el servidor de configuración. 

### <a name="can-i-clone-a-configuration-server-and-use-it-for-orchestration"></a>¿Puedo clonar un servidor de configuración y usarlo para orquestación?
No, debe configurar un servidor de configuración actualizados para evitar problemas de registro.

### <a name="can-i-change-the-vault-in-which-the-configuration-server-is-registered"></a>¿Puedo cambiar el almacén en el que está registrado el servidor de configuración?
No. Después de un almacén está asociado con el servidor de configuración, no se puede cambiar. Revisión [en este artículo](vmware-azure-manage-configuration-server.md#register-a-configuration-server-with-a-different-vault) para obtener información acerca de cómo volver a registrar.

### <a name="can-i-use-the-same-configuration-server-for-disaster-recovery-of-both-vmware-vms-and-physical-servers"></a>¿Se puede usar el mismo servidor de configuración para la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos?
Sí, pero tenga en cuenta que solo se puede con errores de esa máquina física a una VM de VMware.

### <a name="where-can-i-download-the-passphrase-for-the-configuration-server"></a>¿Dónde puedo descargar la frase de contraseña para el servidor de configuración?
[Obtenga información sobre cómo](vmware-azure-manage-configuration-server.md#generate-configuration-server-passphrase) descargar la frase de contraseña.

### <a name="where-can-i-download-vault-registration-keys"></a>¿Dónde puedo descargar las claves de registro del almacén?

En el almacén de Recovery Services, haga clic en **servidores de configuración** en **infraestructura de Site Recovery** > **administrar**. A continuación, en **servidores**, seleccione **Descargar clave de registro** para descargar el archivo de credenciales de almacén.

## <a name="process-server"></a>Servidor de procesos

### <a name="why-am-i-unable-to-select-process-server-during-enable-replication"></a>¿Por qué no se puede seleccionar el servidor de procesos durante la que habilitar la replicación?

Desde la versión 9.24, se han realizado mejoras para mostrar [estado del servidor de procesos durante la habilitación de la replicación](vmware-azure-enable-replication.md#enable-replication). Esto es para evitar la limitación del servidor de proceso y minimizar el uso de servidores de procesos en mal estado.

### <a name="what-is-meant-by-upgrading-process-server-to-924-or-higher-versions-for-accurate-health"></a>¿Qué se entiende por la actualización de servidor de procesos a versiones 9.24 o superiores para el estado precisa?

Desde [9.24 versiones](service-updates-how-to.md#links-to-currently-supported-update-rollups), se presentan las alertas adicionales para mejorar las alertas de estado del servidor de procesos. Actualizar componentes de Site Recovery 9.24 versiones o versiones posteriores para que todas las alertas que se genere.

## <a name="failover-and-failback"></a>Conmutación por error y conmutación por recuperación
### <a name="can-i-use-the-process-server-at-on-premises-for-failback"></a>¿Puedo usar el servidor de procesos en un entorno local para la conmutación por recuperación?
Se recomienda encarecidamente crear un servidor de procesos en Azure con fines de conmutación por recuperación, para evitar las latencias de transferencia de datos. Además, en caso de que separan la red de máquinas virtuales de origen con la red accesible desde Azure en el servidor de configuración, es esencial para usar el servidor de procesos creado en Azure para la conmutación por recuperación.

### <a name="can-i-retain-the-ip-address-on-failover"></a>¿Se puede conservar la dirección IP en la conmutación por error?
Sí, se puede conservar la dirección IP en la conmutación por error. Asegúrese de especificar dirección IP de destino en la configuración de "Proceso y red" para la máquina virtual antes de la conmutación por error. Además, apagar las máquinas en el momento de la conmutación por error para evitar conflictos de direcciones IP durante la conmutación por recuperación.

### <a name="can-i-change-the-target-vm-size-or-vm-type-before-failover"></a>¿Puedo cambiar el tamaño de máquina virtual de destino o el tipo de máquina virtual antes de la conmutación por error?
Sí, puede cambiar el tipo o tamaño de la máquina virtual en cualquier momento antes de la conmutación por error en la configuración de proceso y red de la máquina virtual replicada en el portal.

### <a name="how-far-back-can-i-recover"></a>¿Hasta cuánto tiempo atrás puedo recuperar?
Para VMware en Azure, el punto de recuperación más antiguo que puede usar es de 72 horas.

### <a name="how-do-i-access-azure-vms-after-failover"></a>¿Cómo accedo a las máquinas virtuales de Azure después de la conmutación por error?
Después de la conmutación por error, puede tener acceso a las máquinas virtuales de Azure a través de una conexión segura a Internet o a través de una VPN de sitio a sitio o mediante Azure ExpressRoute. Debe preparar algunas cosas para la conexión. [Más información](site-recovery-test-failover-to-azure.md#prepare-to-connect-to-azure-vms-after-failover)

### <a name="is-failed-over-data-resilient"></a>¿Son resistentes los datos conmutados por error?

Azure está diseñado para la resistencia. Site Recovery está diseñado para la conmutación por error en un centro de datos de Azure secundario según el Acuerdo de Nivel de Servicio de Azure. Cuando se produce una conmutación por error, nos aseguramos de que los metadatos y los almacenes permanecen en la misma región geográfica que eligió para el almacén.

### <a name="is-failover-automatic"></a>¿La conmutación por error es automática?
La [conmutación por error](site-recovery-failover.md) no es automática. Puede iniciar las conmutaciones por error con solo un clic en el portal, o puede usar [PowerShell](/powershell/module/az.recoveryservices) para desencadenar una conmutación por error.

### <a name="can-i-fail-back-to-a-different-location"></a>¿Es posible conmutar por recuperación en otra ubicación?
Sí, si conmutó por error en Azure, puede conmutar por recuperación en otra ubicación si la ubicación original no está disponible. [Más información](concepts-types-of-failback.md#alternate-location-recovery-alr).

### <a name="why-do-i-need-a-vpn-or-expressroute-to-fail-back"></a>¿Por qué se necesita una VPN o ExpressRoute para conmutar por recuperación?
Cuando conmuta por recuperación desde Azure, los datos provenientes de Azure se copia nuevamente en la máquina virtual local y se requiere acceso privado.

### <a name="can-i-resize-the-azure-vm-after-failover"></a>¿Puedo cambiar el tamaño de la máquina virtual de Azure después de la conmutación por error?
No, no se puede cambiar el tamaño ni el tipo de la máquina virtual de destino después de la conmutación por error.


## <a name="automation-and-scripting"></a>Automatización y scripts

### <a name="can-i-set-up-replication-with-scripting"></a>¿Puedo configurar la replicación con scripts?
Sí. Puede automatizar los flujos de trabajo de Site Recovery mediante la API de Rest, PowerShell o el SDK de Azure.[Más información](vmware-azure-disaster-recovery-powershell.md).

## <a name="performance-and-capacity"></a>Rendimiento y capacidad
### <a name="can-i-throttle-replication-bandwidth"></a>¿Puedo limitar el ancho de banda de replicación?
Sí. [Más información](site-recovery-plan-capacity-vmware.md).


## <a name="next-steps"></a>Pasos siguientes
* [Revise](vmware-physical-azure-support-matrix.md) los requisitos de compatibilidad.
* [Configure](vmware-azure-tutorial.md) la replicación de VMware en Azure.
