---
title: Preguntas comunes sobre la recuperación ante desastres de VMware con Azure Site Recovery
description: Obtenga respuestas a las preguntas comunes sobre cómo configurar la recuperación ante desastres de máquinas virtuales de VMware locales en Azure mediante Azure Site Recovery.
ms.date: 11/14/2019
ms.topic: conceptual
ms.openlocfilehash: 603dc77e6f2a53abb1d65688ced77e58297b8ab5
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87086156"
---
# <a name="common-questions-about-vmware-to-azure-replication"></a>Preguntas frecuentes sobre la replicación de VMware en Azure

En este artículo se da respuesta a las preguntas comunes que pueden surgir al implementar la recuperación ante desastres de máquinas virtuales de VMware locales en Azure.

## <a name="general"></a>General

### <a name="what-do-i-need-for-vmware-vm-disaster-recovery"></a>¿Qué es necesario para la recuperación ante desastres de máquinas virtuales de VMware?

[Obtenga información sobre los componentes que intervienen](vmware-azure-architecture.md) en la recuperación ante desastres de máquinas virtuales de VMware.

### <a name="can-i-use-site-recovery-to-migrate-vmware-vms-to-azure"></a>¿Puedo usar Site Recovery para migrar máquinas virtuales de VMware a Azure?

Sí. Además de para configurar la recuperación ante desastres completa de máquinas virtuales de VMware, Site Recovery también se puede usar para migrar máquinas virtuales locales de VMware a Azure. En este escenario, replicamos máquinas virtuales de VMware locales en Azure Storage. Luego, conmuta por error desde el entorno local a Azure. Después de la conmutación por error, las aplicaciones y cargas de trabajo están disponibles y se ejecutan en máquinas virtuales de Azure. El proceso es similar a configurar una recuperación ante desastres completa, salvo por el hecho de que en una migración no se puede conmutar por recuperación desde Azure.

### <a name="does-my-azure-account-need-permissions-to-create-vms"></a>¿La cuenta de Azure necesita permisos para crear máquinas virtuales?

Si es administrador de una suscripción, ya tiene los permisos de replicación que necesita. Si no es un administrador, necesitará permisos para realizar las siguientes acciones:

- Crear una máquina virtual de Azure en el grupo de recursos y en la red virtual especificados al configurar Site Recovery.
- Escribir en la cuenta de almacenamiento seleccionada o en un disco administrado según la configuración.

[Más información](site-recovery-role-based-linked-access-control.md#permissions-required-to-enable-replication-for-new-virtual-machines) sobre los permisos necesarios.

### <a name="what-applications-can-i-replicate"></a>¿Qué aplicaciones se pueden replicar?

Puede replicar cualquier aplicación o carga de trabajo en ejecución en una máquina virtual de VMware que cumpla con los [requisitos de replicación](vmware-physical-azure-support-matrix.md#replicated-machines).

- Site Recovery admite la replicación compatible con aplicaciones, para que estas se puedan conmutar por error y por recuperación a un estado inteligente.
- Site Recovery se integra con aplicaciones de Microsoft, como SharePoint, Exchange, Dynamics, SQL Server y Active Directory. También trabaja en estrecha colaboración con proveedores líderes como Oracle, SAP, IBM y Red Hat.

[Obtenga más información](site-recovery-workload.md) acerca de la protección de la carga de trabajo.

### <a name="can-i-use-a-guest-os-server-license-on-azure"></a>¿Es posible usar la licencia de servidor de SO invitado en Azure?

Sí, los clientes de Microsoft Software Assurance pueden usar la [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/) para ahorrar en los costes de licencia de máquinas Windows Server que se migran a Azure o usar Azure para la recuperación ante desastres.

## <a name="security"></a>Seguridad

### <a name="what-access-to-vmware-servers-does-site-recovery-need"></a>¿Qué acceso a los servidores de VMware necesita Site Recovery?

Site Recovery necesita acceso a los servidores de VMware para:

- Configurar una máquina virtual de VMware donde se ejecute el servidor de configuración de Site Recovery.
- Detectar automáticamente máquinas virtuales para su replicación.

### <a name="what-access-to-vmware-vms-does-site-recovery-need"></a>¿Qué acceso a las máquinas virtuales de VMware necesita Site Recovery?

- Para poder realizar la replicación, una máquina virtual de VMware debe tener Site Recovery Mobility Service instalado y en ejecución. Puede implementar manualmente la herramienta o especificar que Site Recovery debe realizar una instalación de inserción del servicio cuando habilite la replicación de una máquina virtual.
- Durante la replicación, las máquinas virtuales se comunican con Site Recovery como se indica a continuación:
    - Las máquinas virtuales se comunican con el servidor de configuración en el puerto HTTPS 443 para la administración de la replicación.
    - Las máquinas virtuales envían los datos de replicación al servidor de procesos en el puerto HTTPS 9443 (esta configuración no se puede modificar).
    - Si habilita la coherencia entre varias máquinas virtuales, estas se comunican entre sí a través del puerto 20004.

### <a name="is-replication-data-sent-to-site-recovery"></a>¿Los datos de replicación se envían a Site Recovery?

No, Site Recovery no intercepta los datos replicados ni tiene información sobre lo que se ejecuta en las máquinas virtuales. Los datos de replicación se intercambian entre los hipervisores de VMware y Azure Storage. Site Recovery no tiene capacidad para interceptar los datos. Únicamente se envían los metadatos necesarios para coordinar la replicación y la conmutación por error al servicio Site Recovery.

Site Recovery está certificado para ISO 27001: 2013 y 27018, HIPAA y DPA. Aparte de esto, hay evaluaciones de SOC2 y FedRAMP JAB en curso.

## <a name="pricing"></a>Precios

### <a name="how-do-i-calculate-approximate-charges-for-vmware-disaster-recovery"></a>¿Cómo se pueden calcular los costes aproximados de VMware y la recuperación ante desastres?

Use la [calculadora de precios](https://aka.ms/asr_pricing_calculator) para estimar los costes al usar Site Recovery.

Para obtener una estimación detallada de los costes, ejecute la herramienta Deployment Planner para [VMware](https://aka.ms/siterecovery_deployment_planner) y use el [informe de estimación de costes](https://aka.ms/asr_DP_costreport).

### <a name="is-there-any-difference-in-cost-between-replicating-to-storage-or-directly-to-managed-disks"></a>¿Existe alguna diferencia de coste entre replicar en el almacenamiento o directamente en discos administrados?

Los discos administrados tienen un cargo ligeramente distinto del de las cuentas de almacenamiento. [Más información](https://azure.microsoft.com/pricing/details/managed-disks/) sobre los precios de los discos administrados.

### <a name="is-there-any-difference-in-cost-when-replicating-to-general-purpose-v2-storage-account"></a>¿Existe alguna diferencia de coste cuando se replica en la cuenta de almacenamiento de uso general v2?

Normalmente, verá un aumento en el coste de las transacciones en que se incurre al usar cuentas de almacenamiento de GPv2, ya que las transacciones en Azure Site Recovery son muy frecuentes. [Más información](../storage/common/storage-account-upgrade.md#pricing-and-billing) para calcular el cambio.

## <a name="mobility-service"></a>Mobility Service

### <a name="where-can-i-find-the-mobility-service-installers"></a>¿Dónde puedo encontrar los instaladores de Mobility Service?

Los instaladores están en la carpeta %ProgramData%\ASR\home\svsystems\pushinstallsvc\repository del servidor de configuración.

## <a name="how-do-i-install-the-mobility-service"></a>¿Cómo instalo Mobility Service?

En cada máquina virtual que quiera replicar, instale el servicio empleando uno de estos métodos:

- [Instalación de inserción](vmware-physical-mobility-service-overview.md#push-installation)
- [Instalación manual](vmware-physical-mobility-service-overview.md#install-the-mobility-service-using-ui) desde la interfaz de usuario o PowerShell
- Implementación mediante una herramienta de implementación, como [Configuration Manager](vmware-azure-mobility-install-configuration-mgr.md)

## <a name="managed-disks"></a>Discos administrados

### <a name="where-does-site-recovery-replicate-data-to"></a>¿Dónde replica datos Site Recovery?

Site Recovery replica servidores físicos y máquinas virtuales locales de VMware en discos administrados en Azure.

- El servidor de procesos de Site Recovery escribe los registros de replicación en una cuenta de almacenamiento en caché de la región de destino.
- Estos registros se utilizan para crear puntos de recuperación en los discos administrados de Azure que tengan el prefijo **asrseeddisk**.
- Cuando se produzca una conmutación por error, el punto de recuperación que seleccione se usa para crear un disco administrado de destino. Este disco administrado está conectado a la máquina virtual en Azure.
- Las máquinas virtuales que antes se replicaban en una cuenta de almacenamiento (antes de marzo de 2019) no se ven afectadas por esto.

### <a name="can-i-replicate-new-machines-to-storage-accounts"></a>¿Puedo replicar máquinas nuevas en cuentas de almacenamiento?

No. Desde marzo de 2019, en Azure Portal, solo se puede realizar la replicación en discos administrados de Azure.

La replicación de máquinas virtuales nuevas en una cuenta de almacenamiento será posible solo mediante PowerShell o la API de REST (versión 2018-01-10 o 2016-08-10).

### <a name="what-are-the-benefits-of-replicating-to-managed-disks"></a>¿Cuáles son las ventajas de la replicación en discos administrados?

[Más información sobre cómo](https://azure.microsoft.com/blog/simplify-disaster-recovery-with-managed-disks-for-vmware-and-physical-servers/) Site Recovery simplifica la recuperación ante desastres con discos administrados.

### <a name="can-i-change-the-managed-disk-type-after-a-machine-is-protected"></a>¿Puedo cambiar el tipo de disco administrado después de proteger una máquina?

Sí, le resultará muy fácil [cambiar el tipo de disco administrado](../virtual-machines/windows/convert-disk-storage.md) de las replicaciones en curso. Antes de cambiar el tipo, asegúrese de que no se ha generado ninguna dirección URL de firma de acceso compartido en el disco administrado:

1. Vaya al recurso **Disco administrado** en Azure Portal y compruebe si tiene un banner de dirección URL de firma de acceso compartido en la hoja **Información general**.
1. Si ve el banner, selecciónelo para cancelar la exportación en curso.
1. Cambie el tipo del disco en los minutos siguientes. Si cambia el tipo del disco administrado, espere a que Azure Site Recovery genere puntos de recuperación nuevos.
1. Use esos puntos de recuperación nuevos en cualquier conmutación por error de prueba o conmutación por error futura.

### <a name="can-i-switch-replication-from-managed-disks-to-unmanaged-disks"></a>¿Puedo cambiar la replicación de disco administrado a disco no administrado?

No. No se puede cambiar de administrado a no administrado.

## <a name="replication"></a>Replicación

### <a name="what-are-the-replicated-vm-requirements"></a>¿Cuáles son los requisitos de las máquinas virtuales replicadas?

[Más información](vmware-physical-azure-support-matrix.md#replicated-machines) sobre los requisitos de compatibilidad de servidores físicos y máquinas virtuales de VMware.

### <a name="how-often-can-i-replicate-to-azure"></a>¿Con qué frecuencia se puede replicar en Azure?

La replicación es continua cuando se replican máquinas virtuales de VMware en Azure.

### <a name="can-i-extend-replication"></a>¿Se puede extender la replicación?

No se admite la replicación extendida o encadenada. Solicite esta característica en el [foro de comentarios](https://feedback.azure.com/forums/256299-site-recovery/suggestions/6097959).

### <a name="can-i-do-an-offline-initial-replication"></a>¿Se puede realizar una replicación inicial sin conexión?

La replicación sin conexión no es posible. Solicite esta característica en el [foro de comentarios](https://feedback.azure.com/forums/256299-site-recovery/suggestions/6227386-support-for-offline-replication-data-transfer-from).

### <a name="what-is-asrseeddisk"></a>¿Qué es asrseeddisk?

En cada disco de origen, los datos se replican en un disco administrado en Azure. Este disco tiene un prefijo, **asrseeddisk**, y en él se almacena la copia del disco de origen y todas las instantáneas de punto de recuperación.

### <a name="can-i-exclude-disks-from-replication"></a>¿Se pueden excluir discos de la replicación?

Sí, se puede.

### <a name="can-i-replicate-vms-that-have-dynamic-disks"></a>¿Se pueden replicar máquinas virtuales que tienen discos dinámicos?

Los discos dinámicos se pueden replicar. El disco del sistema operativo debe ser un disco básico.

### <a name="if-i-use-replication-groups-for-multi-vm-consistency-can-i-add-a-new-vm-to-an-existing-replication-group"></a>Si utilizo grupos de replicación para mantener la coherencia con varias máquinas virtuales, ¿puedo agregar una nueva máquina virtual a un grupo de replicación existente?

Sí, puede agregar nuevas máquinas virtuales a un grupo de replicación existente al habilitar la replicación para ellas. Pero:

- No se puede agregar una máquina virtual a un grupo de replicación existente una vez iniciada la replicación.
- No se puede crear un grupo de replicación para máquinas virtuales existentes.

### <a name="can-i-modify-vms-that-are-replicating-by-adding-or-resizing-disks"></a>¿Puedo modificar máquinas virtuales que se replican mediante la adición o el cambio de tamaño de discos?

Para realizar una replicación de VMware en Azure, el tamaño del disco de las VM de origen se puede modificar. Si desea agregar discos nuevos, debe agregar el disco y volver a habilitar la protección de la máquina virtual.

### <a name="can-i-migrate-on-premises-machines-to-a-new-vcenter-server-without-impacting-ongoing-replication"></a>¿Puedo migrar máquinas locales a una nueva instancia de vCenter Server sin que ello repercuta en la replicación en curso?

No. Un cambio en la instancia de VMware vCenter o la migración afectará a la replicación en curso. Configure Site Recovery con la nueva instancia de vCenter Server y habilite la replicación de máquinas de nuevo.

### <a name="can-i-replicate-to-a-cache-or-target-storage-account-that-has-a-virtual-network-with-azure-firewalls-configured-on-it"></a>¿Puedo replicar en una cuenta de almacenamiento en caché o de destino que tenga una red virtual (con instancias de Azure Firewall) configurada?

No, Site Recovery no admite la replicación ni en Azure Storage ni en redes virtuales.

### <a name="what-is-the-frequency-of-generation-of-crash-consistent-recovery-points"></a>¿Cuál es la frecuencia de generación de puntos de recuperación coherentes frente a bloqueos?

Site Recovery genera puntos de recuperación coherentes frente a bloqueos cada 5 minutos.

## <a name="component-upgrade"></a>Actualización de componentes

### <a name="my-version-of-the-mobility-services-agent-or-configuration-server-is-old-and-my-upgrade-failed-what-do-i-do"></a>Mi versión del agente de Mobility Service o del servidor de configuración es antigua y no he podido actualizarla. ¿Qué puedo hacer?

Site Recovery sigue el modelo de soporte técnico de N-4. [Más información](https://aka.ms/asr_support_statement) sobre cómo actualizar desde versiones muy antiguas.

### <a name="where-can-i-find-the-release-notes-and-update-rollups-for-azure-site-recovery"></a>¿Dónde puedo encontrar las notas de la versión o los paquetes acumulativos de actualización de Azure Site Recovery?

[Más información sobre nuevas actualizaciones](site-recovery-whats-new.md) y [la acumulación](service-updates-how-to.md).

### <a name="where-can-i-find-upgrade-information-for-disaster-recovery-to-azure"></a>¿Dónde puedo encontrar información de actualización de la recuperación ante desastres en Azure?

[Más información sobre cómo actualizar](https://aka.ms/asr_vmware_upgrades).

## <a name="do-i-need-to-reboot-source-machines-for-each-upgrade"></a>¿Es necesario reiniciar las máquinas de origen en cada actualización?

No es obligatorio reiniciar en cada actualización, si bien es recomendable. [Más información](https://aka.ms/asr_vmware_upgrades).

## <a name="configuration-server"></a>Servidor de configuración

### <a name="what-does-the-configuration-server-do"></a>¿Qué hace el servidor de configuración?

El servidor de configuración ejecuta los componentes locales de Site Recovery, incluidos:

- El servidor de configuración en sí. El servidor se encarga de coordinar las comunicaciones entre los componentes locales y Azure, además de administrar la replicación de datos.
- El servidor de procesos que actúa como puerta de enlace de replicación. Este servidor:
    1. Recibe datos de replicación.
    2. Optimiza los datos con el almacenamiento en caché, la compresión y cifrado.
    3. Envía los datos a Azure Storage.
  El servidor de procesos también lleva a cabo una instalación de inserción de Mobility Service en las máquinas virtuales, además de realizar la detección automática de las máquinas virtuales de VMware locales.
- El servidor de destino maestro que controla los datos de replicación durante la conmutación por recuperación desde Azure.

[Más información](vmware-azure-architecture.md) acerca de los componentes y procesos del servidor de configuración.

### <a name="where-do-i-set-up-the-configuration-server"></a>¿Dónde configuro el servidor de configuración?

Necesita una sola máquina virtual de VMware local de alta disponibilidad para el servidor de configuración. Para la recuperación ante desastres de servidores físicos, instale el servidor de configuración en una máquina física.

### <a name="what-do-i-need-for-the-configuration-server"></a>¿Qué es necesario para el servidor de configuración?

Revise los [requisitos previos](vmware-azure-deploy-configuration-server.md#prerequisites).

### <a name="can-i-manually-set-up-the-configuration-server-instead-of-using-a-template"></a>¿Puedo configurar manualmente el servidor de configuración en lugar de usar una plantilla?

Se recomienda [crear la máquina virtual del servidor de configuración](vmware-azure-deploy-configuration-server.md) con la versión más reciente de la plantilla de Open Virtualization Format (OVF). Si no puede usar la plantilla (por ejemplo, si no tiene acceso al servidor de VMware), [descargue](physical-azure-set-up-source.md) el archivo de configuración de Portal y configure el servidor de configuración.

### <a name="can-a-configuration-server-replicate-to-more-than-one-region"></a>¿Un servidor de configuración se puede replicar en más de una región?

No. Para replicar en más de una región, se necesita un servidor de configuración en cada región.

### <a name="can-i-host-a-configuration-server-in-azure"></a>¿Puedo hospedar un servidor de configuración en Azure?

Si bien es posible, la máquina virtual de Azure que ejecuta el servidor de configuración necesitaría comunicarse con las máquinas virtuales y la infraestructura de VMware locales. Esta comunicación agrega latencia y afecta a la replicación en curso.

### <a name="how-do-i-update-the-configuration-server"></a>¿Cómo actualizo el servidor de configuración?

[Más información](vmware-azure-manage-configuration-server.md#upgrade-the-configuration-server) sobre cómo actualizar el servidor de configuración.

- Encontrará información sobre las actualizaciones más recientes en la [página de actualizaciones de Azure](https://azure.microsoft.com/updates/?product=site-recovery).
- Puede descargar la versión más reciente desde Portal, o bien puede descargar la versión más reciente del servidor de configuración directamente desde el [Centro de descarga de Microsoft](https://aka.ms/asrconfigurationserver).
- Si su versión es anterior a cuatro versiones de la versión actual, vea nuestra [declaración de compatibilidad](https://aka.ms/asr_support_statement) para obtener orientación sobre la actualización.

### <a name="should-i-back-up-the-configuration-server"></a>¿Debo hacer copias de seguridad del servidor de configuración?

Se recomienda programar copias de seguridad periódicas del servidor de configuración.

- Para que la conmutación por recuperación sea correcta, la máquina virtual que se conmutó debe existir en la base de datos del servidor de configuración.
- El servidor de configuración debe estar en ejecución y en un estado conectado.
- [Más información](vmware-azure-manage-configuration-server.md) sobre las tareas comunes de administración del servidor de configuración.

### <a name="when-im-setting-up-the-configuration-server-can-i-download-and-install-mysql-manually"></a>Cuando estoy configurando el servidor de configuración, ¿puedo descargar e instalar MySQL manualmente?

Sí. Descargue MySQL y colóquelo en la carpeta C:\Temp\ASRSetup. Después, instálelo manualmente. Cuando configure la máquina virtual del servidor de configuración y acepte los términos, MySQL aparecerá como **Ya instalado**en **Descargar e instalar**.

### <a name="can-i-avoid-downloading-mysql-but-let-site-recovery-install-it"></a>¿Se puedo evitar descargar MySQL pero dejar que Site Recovery lo instale?

Sí. Descargue el instalador de MySQL y colóquelo en la carpeta C:\Temp\ASRSetup. Al configurar la máquina virtual del servidor de configuración, acepte los términos y seleccione **Descargar e instalar**. Portal usará el instalador que haya agregado para instalar MySQL.

### <a name="can-i-use-the-configuration-server-vm-for-anything-else"></a>¿Se puede usar la VM del servidor de configuración para cualquier otra cosa?

No. Use la máquina virtual solo con el servidor de configuración.

### <a name="can-i-clone-a-configuration-server-and-use-it-for-orchestration"></a>¿Puedo clonar un servidor de configuración y usarlo para orquestación?

No. Configure un nuevo servidor de configuración para evitar problemas de registro.

### <a name="can-i-change-the-vault-in-which-the-configuration-server-is-registered"></a>¿Se puede cambiar el almacén en el que el servidor de configuración está registrado?

No. Una vez que un almacén esté asociado con el servidor de configuración, no se puede cambiar. [Más información](vmware-azure-manage-configuration-server.md#register-a-configuration-server-with-a-different-vault) sobre cómo registrar un servidor de configuración con un almacén diferente.

### <a name="can-i-use-the-same-configuration-server-for-disaster-recovery-of-both-vmware-vms-and-physical-servers"></a>¿Se puede usar el mismo servidor de configuración para la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos?

Sí, pero tenga en cuenta que solo se puede realizar la conmutación por recuperación de una máquina física a una máquina virtual de VMware.

### <a name="where-can-i-download-the-passphrase-for-the-configuration-server"></a>¿Dónde puedo descargar la frase de contraseña para el servidor de configuración?

[Más información](vmware-azure-manage-configuration-server.md#generate-configuration-server-passphrase) sobre cómo descargar la frase de contraseña.

### <a name="where-can-i-download-vault-registration-keys"></a>¿Dónde puedo descargar las claves de registro del almacén?

En el almacén de Recovery Services, seleccione **Servidores de configuración** en **Infraestructura de Site Recovery** > **Administrar**. Después, en **Servidores**, seleccione **Descargar clave de registro** para descargar el archivo de credenciales del almacén.

### <a name="can-a-single-configuration-server-be-used-to-protect-multiple-vcenter-instances"></a>¿Se puede usar un solo servidor de configuración para proteger varias instancias de vCenter?

Sí, un solo servidor de configuración puede proteger las máquinas virtuales en varias instancias de vCenter.  No hay un máximo de instancias de vCenter que se puedan agregar al servidor de configuración; sin embargo, sí se aplican los límites al número de máquinas virtuales que puede proteger un solo servidor de configuración.

### <a name="can-a-single-configuration-server-protect-multiple-clusters-within-vcenter"></a>¿Puede un solo servidor de configuración proteger varios clústeres en vCenter?

Sí, Azure Site Recovery puede proteger las máquinas virtuales de diferentes clústeres.

## <a name="process-server"></a>Servidor de proceso

### <a name="why-am-i-unable-to-select-the-process-server-when-i-enable-replication"></a>¿Por qué no puedo seleccionar el servidor de procesos al habilitar la replicación?

Ahora, las actualizaciones en las versiones 9.24 en adelante indican el [estado del servidor de procesos al habilitar la replicación](vmware-azure-enable-replication.md#enable-replication). Esta característica ayuda a evitar la limitación del servidor de procesos y a minimizar el uso de servidores de procesos en mal estado.

### <a name="how-do-i-update-the-process-server-to-version-924-or-later-for-accurate-health-information"></a>¿Cómo puedo actualizar el servidor de procesos a la versión 9.24 o posterior para obtener información de estado precisa?

A partir de la [versión 9.24](service-updates-how-to.md#links-to-currently-supported-update-rollups), se han agregado más alertas para indicar el estado del servidor de procesos. [Actualice los componentes de Site Recovery a la versión 9.24 o posterior](service-updates-how-to.md#links-to-currently-supported-update-rollups) para que se generen todas las alertas.

### <a name="how-can-i-ensure-high-availability-of-the-process-server"></a>¿Cómo se puede garantizar la alta disponibilidad del servidor de procesos?

Al configurar más de un servidor de procesos, el diseño proporciona flexibilidad para trasladar las máquinas protegidas de un servidor de procesos incorrecto al servidor de procesos de trabajo. El movimiento de una máquina de un servidor de procesos a otro debe iniciarse explícita o manualmente a través de los pasos definidos aquí: [mover máquinas virtuales entre servidores de procesos](vmware-azure-manage-process-server.md#move-vms-to-balance-the-process-server-load).

## <a name="failover-and-failback"></a>Conmutación por error y conmutación por recuperación

### <a name="can-i-use-the-on-premises-process-server-for-failback"></a>¿Puedo usar el servidor de procesos local para la conmutación por recuperación?

Se recomienda encarecidamente crear un servidor de procesos en Azure con fines de conmutación por recuperación para así evitar las latencias de las transferencias de datos. Además, si separó la red de máquinas virtuales de origen de la red accesible desde Azure en el servidor de configuración, es fundamental usar el servidor de procesos creado en Azure para la conmutación por recuperación.

### <a name="can-i-keep-the-ip-address-on-failover"></a>¿Se puede conservar la dirección IP en la conmutación por error?

Sí, se puede conservar la dirección IP en la conmutación por error. Asegúrese de especificar la dirección IP de destino en la configuración **Proceso y red** de la máquina virtual antes de la conmutación por error. Asimismo, apague las máquinas en el momento de la conmutación por error para evitar conflictos de direcciones IP durante el proceso.

### <a name="can-i-change-the-target-vm-size-or-vm-type-before-failover"></a>¿Puedo cambiar el tamaño de la máquina virtual de destino o el tipo de máquina virtual antes de la conmutación por error?

Sí, puede cambiar el tipo o el tamaño de la máquina virtual en cualquier momento antes de proceder a la conmutación por error. En Portal, abra la configuración **Proceso y red** de la máquina virtual replicada.

### <a name="how-far-back-can-i-recover"></a>¿Hasta cuánto tiempo atrás puedo recuperar?

En el caso de VMware en Azure, el punto de recuperación más antiguo que puede usar es de 72 horas.

### <a name="how-do-i-access-azure-vms-after-failover"></a>¿Cómo accedo a las máquinas virtuales de Azure después de la conmutación por error?

Después de la conmutación por error, puede tener acceso a las máquinas virtuales de Azure a través de una conexión segura a Internet o a través de una VPN de sitio a sitio o mediante Azure ExpressRoute. Para conectarse, debe preparar varias cosas. [Más información](site-recovery-test-failover-to-azure.md#prepare-to-connect-to-azure-vms-after-failover).

### <a name="is-failed-over-data-resilient"></a>¿Son resistentes los datos conmutados por error?

Azure está diseñado para la resistencia. Site Recovery está diseñado para la conmutación por error en un centro de datos de Azure secundario, según lo requiere el Acuerdo de nivel de servicio (SLA) de Azure. Cuando se produce una conmutación por error, nos aseguramos de que los metadatos y los almacenes permanecen en la misma región geográfica que eligió para el almacén.

### <a name="is-failover-automatic"></a>¿La conmutación por error es automática?

La [conmutación por error](site-recovery-failover.md) no es automática. Puede iniciar las conmutaciones por error realizando una selección única en Portal, o bien usando [PowerShell](/powershell/module/az.recoveryservices) para desencadenar una conmutación por error.

### <a name="can-i-fail-back-to-a-different-location"></a>¿Es posible conmutar por recuperación en otra ubicación?

Sí. Si conmutó por error en Azure, puede conmutar por recuperación en otra ubicación si la ubicación original no está disponible. [Más información](concepts-types-of-failback.md#alternate-location-recovery-alr).

### <a name="why-do-i-need-a-vpn-or-expressroute-with-private-peering-to-fail-back"></a>¿Por qué se necesita una VPN o ExpressRoute con emparejamiento privado para conmutar por recuperación?

Cuando se conmuta por recuperación desde Azure, los datos provenientes de Azure se copian de nuevo en la máquina virtual local y se requiere acceso privado.


## <a name="automation-and-scripting"></a>Automatización y scripts

### <a name="can-i-set-up-replication-with-scripting"></a>¿Puedo configurar la replicación con scripts?

Sí. Puede automatizar los flujos de trabajo de Site Recovery mediante la API de REST, PowerShell o el SDK de Azure. [Más información](vmware-azure-disaster-recovery-powershell.md).

## <a name="performance-and-capacity"></a>Rendimiento y capacidad

### <a name="can-i-throttle-replication-bandwidth"></a>¿Puedo limitar el ancho de banda de replicación?

Sí. [Más información](site-recovery-plan-capacity-vmware.md).

## <a name="next-steps"></a>Pasos siguientes

- [Revise](vmware-physical-azure-support-matrix.md) los requisitos de compatibilidad.
- [Configure](vmware-azure-tutorial.md) la replicación de VMware en Azure.
