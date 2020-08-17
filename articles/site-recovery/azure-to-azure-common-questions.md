---
title: Preguntas comunes sobre la recuperación ante desastres de máquinas virtuales de Azure con Azure Site Recovery
description: En este artículo se responde a preguntas comunes sobre la recuperación ante desastres de máquinas virtuales de Azure cuando se usa Azure Site Recovery.
author: sideeksh
manager: rochakm
ms.date: 04/29/2019
ms.topic: conceptual
ms.openlocfilehash: 7bc8427a51a9931ca82155232569767f12a8e266
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87534029"
---
# <a name="common-questions-azure-to-azure-disaster-recovery"></a>Preguntas frecuentes: Recuperación ante desastres de Azure a Azure

En este artículo se responde a preguntas comunes sobre la recuperación ante desastres de máquinas virtuales de Azure a otra región de Azure cuando se usa [Azure Site Recovery](site-recovery-overview.md).

## <a name="general"></a>General

### <a name="how-is-site-recovery-priced"></a>¿Cómo se fija el precio de Site Recovery?

Revise los [precios de Azure Site Recovery para máquinas virtuales](https://azure.microsoft.com/blog/know-exactly-how-much-it-will-cost-for-enabling-dr-to-your-azure-vm/).

### <a name="how-does-the-free-tier-for-azure-site-recovery-work"></a>¿Cómo funciona el nivel Gratis de Azure Site Recovery?

Todas las instancias protegidas con Azure Site Recovery son gratuitas los primeros 31 días de protección. Después de ese período, la protección de cada instancia sigue las tarifas que se indican en los [precios de Azure Site Recovery para Azure Virtual Machines](https://azure.microsoft.com/blog/know-exactly-how-much-it-will-cost-for-enabling-dr-to-your-azure-vm/).

### <a name="during-the-first-31-days-will-i-incur-any-other-azure-charges"></a>Durante los primeros 31 días, ¿puedo incurrir en otros cargos de Azure?

Sí. Aunque Azure Site Recovery sea gratuito los primeros 31 días de una instancia protegida, se pueden generar cargos por Azure Storage, transacciones de almacenamiento y transferencia de datos. Una máquina virtual recuperada también puede generar cargos por proceso de Azure. Puede obtener información completa en [Precios de Azure Site Recovery](https://azure.microsoft.com/pricing/details/site-recovery).

### <a name="what-are-the-best-practices-for-azure-virtual-machines-disaster-recovery"></a>¿Cuáles son los procedimientos recomendados para la recuperación ante desastres de Azure Virtual Machines?

1. [Descripción de la arquitectura de Azure a Azure](azure-to-azure-architecture.md)
1. [Revisión de las configuraciones admitidas y no admitidas](azure-to-azure-support-matrix.md)
1. [Configuración de la recuperación ante desastres de máquinas virtuales de Azure](azure-to-azure-how-to-enable-replication.md)
1. [Ejecución de una conmutación por error de prueba](azure-to-azure-tutorial-dr-drill.md)
1. [Conmutación por error y conmutación por recuperación a la región principal](azure-to-azure-tutorial-failover-failback.md)

### <a name="how-is-capacity-ensured-in-the-target-region"></a>¿Cómo se garantiza la capacidad en la región de destino?

El equipo de Site Recovery y el equipo de administración de la capacidad de Azure planean que haya una capacidad de infraestructura suficiente. Cuando se inicia una conmutación por error, los equipos también ayudan a garantizar que las instancias que están protegidas mediante Site Recovery se implementarán en la región de destino.

## <a name="replication"></a>Replicación

### <a name="can-i-replicate-vms-enabled-through-azure-disk-encryption"></a>¿Puedo replicar máquinas virtuales habilitadas a través de Azure Disk Encryption?

Sí. Site Recovery admite la recuperación ante desastres de máquinas virtuales con Azure Disk Encryption habilitado. Cuando está habilitada la replicación, Azure copia todos los secretos y claves de cifrado del disco necesarios de la región de origen a la de destino en el contexto del usuario. Si no tiene los permisos adecuados, el administrador de seguridad puede usar un script para copiarlos.

- Site Recovery admite Azure Disk Encryption para máquinas virtuales de Azure que ejecutan Windows.
- Site Recovery admite la versión 0.1 de Azure Disk Encryption, que tiene un esquema que requiere Azure Active Directory (Azure AD). Site Recovery también admite la versión 1.1, que no necesita Azure AD. [Más información sobre el esquema de extensión de Azure Disk Encryption](../virtual-machines/extensions/azure-disk-enc-windows.md#extension-schema).
  - En el caso de la versión 1.1 de Azure Disk Encryption, tiene que usar las máquinas virtuales Windows con discos administrados.
  - [Más información](azure-to-azure-how-to-enable-replication-ade-vms.md) sobre la habilitación de la replicación para máquinas virtuales cifradas.

### <a name="can-i-select-an-automation-account-from-a-different-resource-group"></a>¿Puedo seleccionar una cuenta de Automation de otro grupo de recursos?

Actualmente, esto no se admite desde el portal, pero puede elegir una cuenta de Automation de un grupo de recursos diferente mediante PowerShell.

### <a name="after-specifying-an-automation-account-that-is-in-a-different-resource-group-than-the-vault-am-i-permitted-to-delete-the-runbook-if-there-is-no-other-vault-to-specify"></a>Después de especificar una cuenta de Automation que se encuentra en otro grupo de recursos distinto al del almacén, ¿puedo eliminar el runbook si no hay ningún otro almacén que especificar?

El runbook personalizado creado es una herramienta y es seguro eliminarlo si ya no se necesita.

### <a name="can-i-replicate-vms-to-another-subscription"></a>¿Se pueden replicar máquinas virtuales en otra suscripción?

Sí, puede replicar máquinas virtuales de Azure en otra suscripción que esté en el mismo inquilino de Azure AD.

Configure la recuperación ante desastres [entre suscripciones](https://azure.microsoft.com/blog/cross-subscription-dr) mediante la selección de otra suscripción en el momento de la replicación.

### <a name="can-i-replicate-zone-pinned-azure-vms-to-another-region"></a>¿Se pueden replicar máquinas virtuales de Azure ancladas por zona en otra región?

Sí, puede [replicar máquinas virtuales ancladas por zona](https://azure.microsoft.com/blog/disaster-recovery-of-zone-pinned-azure-virtual-machines-to-another-region) en otra región.

### <a name="can-i-replicate-vms-in-a-region-that-has-zones-from-non-zone-to-zonal-configuration"></a>¿Puedo replicar las VM de una región que tiene zonas de una configuración que no es de zona a zona?

No, actualmente no se admite esta función. Como solución alternativa, puede replicar la VM con ASR en una configuración de zona en otra región y, a continuación, deshabilitar la replicación. A continuación, vuelva a habilitar la replicación de esa región en la región original y elija una configuración de zona para la conmutación por error.

### <a name="can-i-exclude-disks"></a>¿Se pueden excluir discos?

Sí, puede excluir discos en el momento de la protección mediante PowerShell. Para más información, consulte [cómo excluir discos de la replicación](azure-to-azure-exclude-disks.md).

### <a name="can-i-add-new-disks-to-replicated-vms-and-enable-replication-for-them"></a>¿Puedo agregar nuevos discos a máquinas virtuales replicadas y habilitar la replicación para ellos?

Sí, en máquinas virtuales de Azure con discos administrados se admite la incorporación de nuevos discos a las máquinas virtuales replicadas y la habilitación de la replicación para ellas. Cuando se agrega un nuevo disco a una máquina virtual de Azure que está habilitada para la replicación, el estado de replicación de la máquina virtual muestra una advertencia. Esa advertencia indica que uno o varios discos de la máquina virtual están disponibles para la protección. Puede habilitar la replicación para los discos agregados.

- Si habilita la protección para los discos agregados, la advertencia desaparecerá después de la replicación inicial.
- Si no permite la replicación para el disco, puede descartar la advertencia.
- Si conmuta por error una máquina virtual que tiene un disco agregado y la replicación habilitada, existen puntos de replicación. Los puntos de replicación mostrarán los discos que están disponibles para la recuperación.

Por ejemplo, supongamos que una máquina virtual tiene un único disco y agrega uno nuevo. Podría haber un punto de replicación que se haya creado antes de agregar el disco. Este punto de replicación mostrará que consta de "1 de 2 discos".

Site Recovery no admite la eliminación en caliente de un disco de una máquina virtual replicada. Si quita un disco de la máquina virtual, deberá deshabilitar la replicación y luego volver a habilitarla para la máquina virtual.

### <a name="how-often-can-i-replicate-to-azure"></a>¿Con qué frecuencia se puede replicar en Azure?

La replicación es continua cuando se replican máquinas virtuales de Azure en otra región de Azure. Para más información, consulte [Arquitectura de recuperación ante desastres de Azure a Azure](./azure-to-azure-architecture.md#replication-process).

### <a name="can-i-replicate-virtual-machines-within-a-region-i-need-this-functionality-to-migrate-vms"></a>¿Se pueden replicar máquinas virtuales de una misma región? Esta funcionalidad es necesaria para migrar máquinas virtuales.

No puede usar una solución de recuperación de discos de Azure a Azure para replicar máquinas virtuales en una región.

### <a name="can-i-replicate-vm-instances-to-any-azure-region"></a>¿Se pueden replicar instancias de máquina virtual en cualquier región de Azure?

Con Site Recovery, puede replicar y recuperar máquinas virtuales entre dos regiones cualesquiera dentro del mismo clúster geográfico. Los clústeres geográficos se definen teniendo presente la latencia y la soberanía de datos. Para obtener más información, consulte la [matriz de regiones admitidas](./azure-to-azure-support-matrix.md#region-support) de Site Recovery.

### <a name="does-site-recovery-require-internet-connectivity"></a>¿Site Recovery requiere conectividad a Internet?

No, Site Recovery no requiere conectividad a Internet. Sin embargo, requiere acceso a los intervalos IP y direcciones URL de Site Recovery, como se menciona en el artículo sobre las [redes en la recuperación ante desastres de máquinas virtuales de Azure](./azure-to-azure-about-networking.md#outbound-connectivity-for-urls).

### <a name="can-i-replicate-an-application-that-has-a-separate-resource-group-for-separate-tiers"></a>¿Se puede replicar una aplicación que tiene grupos de recursos independientes para niveles independientes?

Sí, se puede replicar la aplicación y mantener también la configuración de recuperación ante desastres en un grupo de recursos independiente.

Por ejemplo, si la aplicación tiene la aplicación, la base de datos y la web de cada nivel en un grupo de recursos independiente, debe seleccionar el [asistente de replicación](./azure-to-azure-how-to-enable-replication.md#enable-replication) tres veces para proteger todos los niveles. Site Recovery replicará estos tres niveles en tres grupos de recursos diferentes.

### <a name="can-i-move-storage-accounts-across-resource-groups"></a>¿Puedo mover cuentas de almacenamiento entre grupos de recursos?

No, se trata de un escenario no admitido. Pero si mueve accidentalmente cuentas de almacenamiento a otro grupo de recursos y elimina el grupo de recursos original, puede crear un nuevo grupo de recursos con el mismo nombre que el antiguo y luego mover la cuenta de almacenamiento a este grupo de recursos.

## <a name="replication-policy"></a>Directiva de replicación

### <a name="what-is-a-replication-policy"></a>¿Qué es una directiva de replicación?

Una directiva de replicación define la configuración del historial de retención de puntos de recuperación. La directiva también define la frecuencia de las instantáneas coherentes con la aplicación. De forma predeterminada, Azure Site Recovery crea una nueva directiva de replicación con la configuración predeterminada de:

- 24 horas para el historial de retención de puntos de recuperación.
- 4 horas para la frecuencia de las instantáneas coherentes con la aplicación.

[Más información sobre la configuración de replicación](./azure-to-azure-tutorial-enable-replication.md#configure-replication-settings).

### <a name="what-is-a-crash-consistent-recovery-point"></a>¿Qué es un punto de recuperación coherente con los bloqueos?

Un punto de recuperación coherente con los bloqueos tiene los datos en el disco, como si se tirara del cable de alimentación del servidor durante la instantánea. No incluye nada que estuviera en memoria cuando se tomó la instantánea.

Hoy en día, la mayoría de las aplicaciones se pueden recuperar bien a partir de instantáneas coherentes con los bloqueos. Un punto de recuperación coherente con los bloqueos suele ser suficiente para aplicaciones y sistemas operativos que no tienen bases de datos, como los servidores de archivos, los servidores DHCP y los servidores de impresión.

### <a name="what-is-the-frequency-of-crash-consistent-recovery-point-generation"></a>¿Cuál es la frecuencia de generación de puntos de recuperación coherentes frente al bloqueo?

Site Recovery crea un punto de recuperación coherente con los bloqueos cada 5 minutos.

### <a name="what-is-an-application-consistent-recovery-point"></a>¿Qué es un punto de recuperación coherente con la aplicación?

Los puntos de recuperación coherentes con la aplicación se crean a partir de instantáneas coherentes con la aplicación. Los puntos de recuperación coherentes con la aplicación capturan los mismos datos que las instantáneas coherentes con los bloqueos, al tiempo que también se capturan los datos en memoria y todas las transacciones en curso.

Debido a su contenido adicional, las instantáneas coherentes con la aplicación son las más complejas y las que más tardan. Se recomiendan puntos de recuperación coherentes con la aplicación para sistemas operativos de bases de datos y aplicaciones como SQL Server.

### <a name="what-is-the-impact-of-application-consistent-recovery-points-on-application-performance"></a>¿Cómo afectan los puntos de recuperación coherentes con la aplicación en el rendimiento de la aplicación?

Los puntos de recuperación coherentes con la aplicación capturan todos los datos en memoria y en curso. Como los puntos de recuperación capturan esas datos, requieren un marco como el Servicio de instantáneas de volumen de Windows para poner en modo inactivo la aplicación. Si el proceso de captura es frecuente, puede afectar al rendimiento cuando la carga de trabajo ya está ocupada. No se recomienda usar baja frecuencia con los puntos de recuperación coherentes con la aplicación en cargas de trabajo que no son de base de datos. Incluso con cargas de trabajo de base de datos, 1 hora es suficiente.

### <a name="what-is-the-minimum-frequency-of-application-consistent-recovery-point-generation"></a>¿Cuál es la frecuencia mínima de generación de puntos de recuperación coherentes frente a la aplicación?

Site Recovery puede crear un punto de recuperación coherente con la aplicación con una frecuencia mínima de 1 hora.

### <a name="how-are-recovery-points-generated-and-saved"></a>¿Cómo se generan y guardan los puntos de recuperación?

Para comprender cómo Site Recovery genera puntos de recuperación, veamos un ejemplo de una directiva de replicación. Esta directiva de replicación tiene un punto de recuperación con una ventana de retención de 24 horas y una instantánea con una frecuencia coherente con la aplicación de 1 hora.

Site Recovery crea un punto de recuperación coherente con los bloqueos cada 5 minutos. No se puede cambiar esta frecuencia. Durante la última hora, puede elegir entre 12 puntos coherentes con los bloqueos y 1 punto coherente con la aplicación. Conforme transcurre el tiempo, Site Recovery elimina todos los puntos de recuperación de hace más de una hora y solo guarda un punto de recuperación por hora.

La siguiente captura de pantalla ilustra el ejemplo. En la captura de pantalla:

- En la pasada hora, hay puntos de recuperación con una frecuencia de 5 minutos.
- Por encima de esta, Site Recovery conserva solo un punto de recuperación.

   ![Lista de puntos de recuperación generados](./media/azure-to-azure-troubleshoot-errors/recoverypoints.png)

### <a name="how-far-back-can-i-recover"></a>¿Hasta cuánto tiempo atrás puedo recuperar?

El punto de recuperación más antiguo que puede usar es de 72 horas.

### <a name="i-have-a-replication-policy-of-24-hours-what-will-happen-if-a-problem-prevents-site-recovery-from-generating-recovery-points-for-more-than-24-hours-will-my-previous-recovery-points-be-lost"></a>Tengo una directiva de replicación de 24 horas. ¿Qué ocurrirá si un problema impide que Site Recovery genere puntos de recuperación durante más de 24 horas? ¿Se perderán mis puntos de recuperación anteriores?

No, Site Recovery mantendrá todos los puntos de recuperación anteriores. En función de la ventana de retención de los puntos de recuperación, Site Recovery reemplaza el punto más antiguo solo si genera nuevos puntos. A consecuencia de este problema, Site Recovery no puede generar ningún punto de recuperación nuevo. Hasta que no haya nuevos puntos de recuperación, todos los puntos anteriores permanecerán después de alcanzar la ventana de retención.

### <a name="after-replication-is-enabled-on-a-vm-how-do-i-change-the-replication-policy"></a>Después de habilitar la replicación en una máquina virtual, ¿cómo puedo cambiar la directiva de replicación?

Vaya a **Almacén de Site Recovery** > **Site Recovery Infrastructure (Infraestructura de Site Recovery)**  > **Directivas de replicación**. Seleccione la directiva que quiera editar y guarde los cambios. Todos los cambios se aplicarán también en todas las replicaciones existentes.

### <a name="are-all-the-recovery-points-a-complete-copy-of-the-vm-or-a-differential"></a>¿Todos los puntos de recuperación son una copia completa de la máquina virtual o diferencial?

El primer punto de recuperación que se genera tiene la copia completa. Los puntos de recuperación sucesivos tienen los cambios diferenciales.

### <a name="does-increasing-the-retention-period-of-recovery-points-increase-the-storage-cost"></a>¿Al aumentar el período de retención de los puntos de recuperación se aumenta el costo de almacenamiento?

Sí. Si aumenta el período de retención de 24 a 72 horas, Site Recovery guardará los puntos de recuperación durante otras 48 horas. El tiempo adicional implicará cargos de almacenamiento. Por ejemplo, un único punto de recuperación podría tener cambios diferenciales de 10 GB con un costo por GB de 0,16 USD al mes. Los cargos adicionales serían de 1,60 USD × 48 al mes.

### <a name="can-i-enable-replication-with-app-consistency-in-linux-servers"></a>¿Se puede habilitar la replicación con coherencia de aplicaciones en los servidores Linux?

Sí. Azure Site Recovery para el sistema operativo Linux admite scripts personalizados de aplicaciones para la coherencia de aplicaciones. El agente de movilidad de Azure Site Recovery usará el script personalizado con opciones previas y posteriores durante la coherencia de aplicaciones. [Más información](./site-recovery-faq.md#can-i-enable-replication-with-app-consistency-in-linux-servers)

## <a name="multi-vm-consistency"></a>Coherencia con múltiples máquinas virtuales

### <a name="what-is-multi-vm-consistency"></a>¿Qué es la coherencia de múltiples máquinas virtuales?

La coherencia de múltiples máquinas virtuales garantiza que el punto de recuperación sea coherente en todas las máquinas virtuales replicadas.

Site Recovery proporciona una opción **Coherencia de múltiples VM**, que crea un grupo de replicación de todas las máquinas.

Al conmutar por error las máquinas virtuales, tendrán puntos de recuperación compartidos coherentes con los bloqueos y coherentes con la aplicación.

Revise el tutorial para [habilitar la coherencia de múltiples máquinas virtuales](./azure-to-azure-tutorial-enable-replication.md#enable-replication-for-a-vm).

### <a name="can-i-fail-over-a-single-virtual-machine-within-a-multi-vm-consistency-replication-group"></a>¿Se puede conmutar por error una sola máquina virtual de un grupo de replicación de coherencia de múltiples máquinas virtuales?

Al seleccionar la opción **Coherencia de múltiples VM**, está indicando que la aplicación depende de todas las máquinas virtuales de un grupo. No se permite la conmutación por error de una sola máquina virtual.

### <a name="how-many-virtual-machines-can-i-replicate-as-a-part-of-a-multi-vm-consistency-replication-group"></a>¿Cuántas máquinas virtuales se pueden replicar como parte de un grupo de replicación de coherencia de múltiples máquinas virtuales?

Puede replicar 16 máquinas virtuales juntas en un grupo de replicación.

### <a name="when-should-i-enable-multi-vm-consistency"></a>¿Cuándo se debe habilitar la coherencia de múltiples máquinas virtuales?

Dado que la coherencia de múltiples máquinas virtuales consume mucha CPU, su habilitación puede afectar al rendimiento. Use la coherencia de múltiples máquinas virtuales solo si las máquinas ejecutan la misma carga de trabajo y necesita coherencia entre varias máquinas virtuales. Por ejemplo, si tiene dos instancias de SQL Server y dos servidores web en una aplicación, debe tener coherencia de múltiples máquinas virtuales solo para las instancias de SQL Server.

### <a name="can-you-add-an-already-replicating-vm-to-a-replication-group"></a>¿Se puede agregar una máquina virtual que ya se esté replicando a un grupo de replicación?
Puede agregar una máquina virtual a un nuevo grupo de replicación mientras permite la replicación. También puede agregar una máquina virtual a un nuevo grupo de replicación existente mientras permite la replicación. Sin embargo, no puede agregar una máquina virtual que ya se esté replicando a un nuevo grupo de replicación o a un grupo de replicación existente.
 
## <a name="failover"></a>Conmutación por error


### <a name="how-is-capacity-ensured-in-the-target-region-for-azure-vms"></a>¿Cómo se garantiza la capacidad en la región de destino de las máquinas virtuales de Azure?

El equipo de Site Recovery y el equipo de administración de la capacidad de Azure planean que haya una capacidad de infraestructura suficiente. Cuando se inicia una conmutación por error, los equipos también ayudan a garantizar que las instancias que están protegidas mediante Site Recovery se implementarán en la región de destino.

### <a name="is-failover-automatic"></a>¿La conmutación por error es automática?

La conmutación por error no es automática. Puede iniciar conmutaciones por error con solo un clic en el portal o bien puede usar [PowerShell](azure-to-azure-powershell.md) para desencadenar una conmutación por error.

### <a name="can-i-keep-a-public-ip-address-after-a-failover"></a>¿Se puede conservar una dirección IP pública después de la conmutación por error?

No se puede conservar la dirección IP pública de la aplicación de producción tras la conmutación por error.

Al incorporar una carga de trabajo como parte del proceso de conmutación por error, debe asignarle un recurso de IP pública de Azure. El recurso de IP pública de Azure tiene que estar disponible en la región de destino. Puede asignar el recurso de IP pública manualmente o automatizarlo con un plan de recuperación. Aprenda a [configurar direcciones IP públicas tras la conmutación por error](concepts-public-ip-address-with-site-recovery.md#public-ip-address-assignment-using-recovery-plan).

### <a name="can-i-keep-a-private-ip-address-during-a-failover"></a>¿Se puede conservar una dirección IP privada durante una conmutación por error?

Sí, puede conservar una dirección IP privada. De forma predeterminada, cuando se habilita la recuperación ante desastres para máquinas virtuales de Azure, Site Recovery crea recursos de destino en función de la configuración de los recursos de origen. En el caso de máquinas virtuales de Azure configuradas con direcciones IP estáticas, Site Recovery trata de aprovisionar la misma dirección IP para la máquina virtual de destino, si no está en uso.
Aprenda sobre cómo [conservar las direcciones IP durante la conmutación por error](site-recovery-retain-ip-azure-vm-failover.md).

### <a name="after-a-failover-why-is-the-server-assigned-a-new-ip-address"></a>Después de una conmutación por error, ¿por qué se asigna al servidor una nueva dirección IP?

Site Recovery trata de proporcionar la dirección IP en el momento de la conmutación por error. Si otra máquina virtual se lleva esa dirección, Site Recovery establece la siguiente dirección IP disponible como destino.

Más información sobre [cómo configurar la asignación de red y el direccionamiento IP para redes virtuales](azure-to-azure-network-mapping.md#set-up-ip-addressing-for-target-vms).

### <a name="what-are-latest-lowest-rpo-recovery-points"></a>¿Qué son los puntos de recuperación **Más reciente (RPO más bajo)** ?

La opción **Más reciente (RPO más bajo)** procesa primero todos los datos que se han enviado a Site Recovery. Luego, crea un punto de recuperación para cada máquina virtual antes de realizar la conmutación por error a estas. Esta opción proporciona el objetivo de punto de recuperación (RPO) más bajo. La máquina virtual creada después de la conmutación por error tiene todos los datos replicados en Site Recovery desde el momento en que se desencadenó este proceso.

### <a name="do-latest-lowest-rpo-recovery-points-have-an-impact-on-failover-rto"></a>¿Los puntos de recuperación con la opción **Más reciente (RPO más bajo)** afectan al RTO de conmutación por error?

Sí. Site Recovery procesa todos los datos pendientes antes de la conmutación por error, por lo que esta opción tiene un objetivo de tiempo de recuperación (RTO) mayor en comparación con otras opciones.

### <a name="what-does-the-latest-processed-option-in-recovery-points-mean"></a>¿Qué significa la opción **Procesado más recientemente** en los puntos de recuperación?

La opción **Procesado más recientemente** realiza una conmutación por error de todas las máquinas virtuales del plan al último punto de recuperación procesado por Site Recovery. Para ver el último punto de recuperación de una máquina virtual específica, seleccione **Puntos de recuperación más recientes** en la configuración de la máquina virtual. Esta opción proporciona un objetivo de tiempo de recuperación bajo, ya que no se invierte tiempo en el procesamiento de datos sin procesar.

### <a name="what-happens-if-my-primary-region-experiences-an-unexpected-outage"></a>¿Qué ocurre si mi región primaria experimenta una interrupción inesperada?

Puede desencadenar una conmutación por error después de la interrupción. Site Recovery no necesita conectividad desde la región primaria para realizar la conmutación por error.

### <a name="what-is-an-rto-of-a-vm-failover"></a>¿Qué es el RTO de una conmutación por error de una máquina virtual?

Site Recovery tiene un [Acuerdo de Nivel de Servicio de RTO de dos horas](https://azure.microsoft.com/support/legal/sla/site-recovery/v1_2/). Sin embargo, la mayoría de las veces, Site Recovery conmuta por error las máquinas virtuales en cuestión de minutos. Para calcular el RTO, vaya a los trabajos de conmutación por error, que muestran el tiempo que tardó en aparecer la máquina virtual. En el caso del RTO del plan de recuperación, consulte la siguiente sección.

## <a name="recovery-plans"></a>Planes de recuperación

### <a name="what-is-a-recovery-plan"></a>¿Qué es un plan de recuperación?

Un plan de recuperación en Site Recovery coordina la recuperación de conmutación por error de máquinas virtuales. Ayuda a que la recuperación sea coherente y precisa, repetible y automatizada. Un plan de recuperación aborda las siguientes necesidades:

- Definición de un grupo de máquinas virtuales que conmutan por error de forma conjunta
- Definición de las dependencias entre las máquinas virtuales para que la aplicación aparezca con precisión
- Automatización de la recuperación junto con acciones manuales personalizadas para lograr tareas distintas de la conmutación por error de las máquinas virtuales

Más información [sobre la creación de planes de recuperación](site-recovery-create-recovery-plans.md).

### <a name="how-is-sequencing-achieved-in-a-recovery-plan"></a>¿Cómo se logra la secuenciación en un plan de recuperación?

En un plan de recuperación, puede crear varios grupos para lograr la secuenciación. Todos los grupos conmutan por error a la vez. Las máquinas virtuales que forman parte del mismo grupo conmutan por error a la vez, y a continuación lo hace otro grupo. Para obtener información sobre cómo modelar una aplicación mediante el uso de un plan de recuperación, consulte [Acerca de los planes de recuperación](recovery-plan-overview.md#model-apps).

### <a name="how-can-i-find-the-rto-of-a-recovery-plan"></a>¿Cómo se puede encontrar el RTO de un plan de recuperación?

Para comprobar el RTO de un plan de recuperación, haga una conmutación por error de prueba del plan de recuperación y vaya a **Trabajos de Site Recovery**.
En el ejemplo siguiente, consulte el trabajo **SAPTestRecoveryPlan**. El trabajo tardó 8 minutos y 59 segundos en conmutar por error todas las máquinas virtuales y en realizar acciones específicas.

![Lista de trabajos de Site Recovery](./media/azure-to-azure-troubleshoot-errors/recoveryplanrto.PNG)

### <a name="can-i-add-automation-runbooks-to-the-recovery-plan"></a>¿Se pueden agregar automáticamente runbooks al plan de recuperación?

Sí, puede integrar runbooks de Azure Automation en el plan de recuperación. Más información sobre [cómo agregar runbooks de Azure Automation](site-recovery-runbook-automation.md).

## <a name="reprotection-and-failback"></a>Reprotección y conmutación por recuperación

### <a name="i-failed-over-from-the-primary-region-to-a-disaster-recovery-region-are-vms-in-a-dr-region-protected-automatically"></a>Se ha realizado la conmutación por error desde la región primaria hasta una región de recuperación ante desastres. ¿Las máquinas virtuales de una región de recuperación ante desastres están protegidas automáticamente?

No. No, al [conmutar por error](./azure-to-azure-tutorial-failover-failback.md) las máquinas virtuales de Azure desde una región a otra, las máquinas virtuales se inician en la región de recuperación ante desastres con un estado desprotegido. Para conmutar por recuperación las máquinas virtuales en la región primaria, tiene que [volver a protegerlas](./azure-to-azure-how-to-reprotect.md) en la región secundaria.

### <a name="at-the-time-of-reprotection-does-site-recovery-replicate-complete-data-from-the-secondary-region-to-the-primary-region"></a>En el momento de la reprotección, ¿Site Recovery replica los datos completos de la región secundaria en la región primaria?

Depende de la situación. Si la máquina virtual de la región de origen existe, solo se sincronizan los cambios entre el disco de origen y el disco de destino. Site Recovery calcula las copias de seguridad diferenciales mediante la comparación de los discos y, a continuación, transfiere los datos. Este proceso suele tardar unas horas. Para más información sobre lo que ocurre durante la reprotección, consulte [Reprotección de instancias de máquina virtual de Azure conmutadas por error en la región primaria](./azure-to-azure-how-to-reprotect.md#what-happens-during-reprotection).

### <a name="how-much-time-does-it-take-to-fail-back"></a>¿Cuánto se tarda en conmutar por recuperación?

Después de la reprotección, la conmutación por recuperación tarda casi lo mismo que la conmutación por error de la región primaria a una región secundaria.

## <a name="capacity"></a><a name="capacity"></a>Capacidad

### <a name="how-is-capacity-ensured-in-the-target-region-for-azure-vms"></a>¿Cómo se garantiza la capacidad en la región de destino de las máquinas virtuales de Azure?

El equipo de Site Recovery y el equipo de administración de la capacidad de Azure planean que haya una capacidad de infraestructura suficiente. Cuando se inicia una conmutación por error, los equipos también ayudan a garantizar que las instancias que están protegidas mediante Site Recovery se implementarán en la región de destino.

### <a name="does-site-recovery-work-with-reserved-instances"></a>¿Funciona Site Recovery con instancias reservadas?

Sí, puede comprar [máquinas virtuales de Azure reservadas](https://azure.microsoft.com/pricing/reserved-vm-instances/) en la región de recuperación ante desastres, que se usarán en las operaciones de conmutación por error de Site Recovery. No se necesita ninguna configuración adicional.

## <a name="security"></a>Seguridad

### <a name="is-replication-data-sent-to-the-site-recovery-service"></a>¿Se envían mis datos de replicación al servicio de Site Recovery?

No, Site Recovery no intercepta los datos replicados ni tiene información sobre lo que se ejecuta en las máquinas virtuales. Únicamente se envían los metadatos necesarios para coordinar la replicación y la conmutación por error al servicio Site Recovery.

Site Recovery tiene la certificación ISO 27001:2013, 27018, HIPAA y DPA. El servicio está llevando a cabo evaluaciones de SOC2 y FedRAMP JAB.

### <a name="does-site-recovery-encrypt-replication"></a>¿Site Recovery cifra la replicación?

Sí, se admite tanto el cifrado en tránsito como el [cifrado en reposo de Azure](../storage/common/storage-service-encryption.md).

## <a name="next-steps"></a>Pasos siguientes

- [Revise los requisitos de recuperación ante desastres entre regiones de Azure](azure-to-azure-support-matrix.md).
- [Configure la replicación de Azure a Azure](azure-to-azure-tutorial-enable-replication.md).
- Si tiene alguna pregunta después de leer este artículo, publíquela en la [Página de preguntas y respuestas de Microsoft para Azure Recovery Services](/answers/topics/azure-site-recovery.html).
