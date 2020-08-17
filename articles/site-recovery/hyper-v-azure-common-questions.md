---
title: Preguntas comunes sobre la recuperación ante desastres en Hyper-V con Azure Site Recovery
description: En este artículo se resumen preguntas comunes sobre la configuración de la recuperación ante desastres de máquinas virtuales de Hyper-V locales a Azure con el sitio de Azure Site Recovery.
ms.date: 11/12/2019
ms.topic: conceptual
ms.openlocfilehash: c168ba9ff14e57f238069e8ca5b0c34a8fb58015
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87799895"
---
# <a name="common-questions---hyper-v-to-azure-disaster-recovery"></a>Preguntas comunes: recuperación ante desastres de Hyper-V a Azure

En este artículo se responden las preguntas frecuentes que vemos al replicar máquinas virtuales de Hyper-V locales en Azure. 

## <a name="general"></a>General

### <a name="how-is-site-recovery-priced"></a>¿Cómo se fija el precio de Site Recovery?
Consulte los detalles de los [precios de Azure Site Recovery](https://azure.microsoft.com/pricing/details/site-recovery/).

### <a name="how-do-i-pay-for-azure-vms"></a>¿Cómo se pagan las máquinas virtuales de Azure?
Durante la replicación, los datos se replican en Azure Storage y no se paga ningún cambio en las máquinas virtuales. Cuando se ejecuta una conmutación por error en Azure, Site Recovery crea automáticamente máquinas virtuales IaaS de Azure. Después de eso, se le facturan los recursos de proceso que consume en Azure.

### <a name="is-there-any-difference-in-cost-when-replicating-to-general-purpose-v2-storage-account"></a>¿Existe alguna diferencia de coste cuando se replica en la cuenta de almacenamiento de uso general v2?

Normalmente, verá un aumento en el coste de las transacciones en que se incurre al usar cuentas de almacenamiento de GPv2, ya que las transacciones en Azure Site Recovery son muy frecuentes. [Más información](../storage/common/storage-account-upgrade.md#pricing-and-billing) para calcular el cambio.

## <a name="azure"></a>Azure

### <a name="what-do-i-need-in-hyper-v-to-orchestrate-replication-with-site-recovery"></a>¿Qué necesito en Hyper-V para orquestar la replicación con Site Recovery?

Los requisitos para el servidor host de Hyper-V dependen del escenario de implementación. Revise los requisitos previos de Hyper-V en:

* [Replicación de máquinas virtuales de Hyper-V (sin VMM) a Azure](./hyper-v-azure-tutorial.md)
* [Replicación de máquinas virtuales de Hyper-V (con VMM) a Azure](./hyper-v-vmm-disaster-recovery.md)
* [Replicación de máquinas virtuales de Hyper-V a un centro de datos secundario](./hyper-v-vmm-disaster-recovery.md)
* Si está replicando en un centro de datos secundario, consulte [Sistemas operativos invitados compatibles para máquinas virtuales Hyper-V](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/mt126277(v=ws.11)).
* Si está replicando a Azure, Site Recovery es compatible con todos los sistemas operativos invitados [admitidos por Azure](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc794868(v=ws.10)).

### <a name="can-i-protect-vms-when-hyper-v-is-running-on-a-client-operating-system"></a>¿Se pueden proteger las máquinas virtuales cuando se ejecuta Hyper-V en un sistema operativo cliente?
No, las máquinas virtuales deben estar ubicadas en un servidor host de Hyper-V en el que se esté ejecutando una máquina de servidor de Windows compatible. Si necesita proteger un equipo cliente, podría replicarlo como una máquina física [en Azure](./vmware-azure-tutorial.md) o en un [centro de datos secundario](./vmware-physical-secondary-disaster-recovery.md).

### <a name="do-hyper-v-hosts-need-to-be-in-vmm-clouds"></a>¿Los hosts de Hyper-V deben estar en nubes VMM?
Si desea replicar en un centro de datos secundario, las máquinas virtuales de Hyper-V deben estar en servidores host de Hyper-V ubicados en una nube VMM. Si desea replicar en Azure, puede replicar las máquinas virtuales con o sin nubes VMM. [Obtenga más información](./hyper-v-azure-tutorial.md) acerca de la replicación de Hyper-V en Azure.


### <a name="can-i-replicate-hyper-v-generation-2-virtual-machines-to-azure"></a>¿Se pueden replicar máquinas virtuales de generación 2 de Hyper-V a Azure?
Sí. Site Recovery convierte de la generación 2 a la generación 1 durante una conmutación por error. En la conmutación por recuperación, la máquina se convierte de nuevo a la generación 2. [Más información](https://azure.microsoft.com/blog/2015/04/28/disaster-recovery-to-azure-enhanced-and-were-listening/).


### <a name="can-i-deploy-site-recovery-with-vmm-if-i-only-have-one-vmm-server"></a>¿Se puede implementar Site Recovery con VMM si solo se tiene un servidor de VMM?

Sí. Puede replicar en Azure las máquinas virtuales que están en servidores de Hyper-V en la nube de VMM, o bien puede replicar entre nubes de VMM en el mismo servidor. Para la replicación de local a local es recomendable que tenga un servidor de VMM tanto en el sitio principal como en el secundario. 

### <a name="what-do-i-need-in-azure"></a>¿Qué necesito tener en Azure?
Necesita una suscripción de Azure, un almacén de Recovery Services, una cuenta de almacenamiento y una red virtual. El almacén, la cuenta de almacenamiento y la red deben estar en la misma región.

### <a name="what-azure-storage-account-do-i-need"></a>¿Qué cuenta de Azure Storage necesito?
Puede usar una cuenta de almacenamiento LRS o GRS. Se recomienda GRS para que los datos sean resistentes si se produce una interrupción regional o si no se puede recuperar la región principal. No se admite Premium Storage.

### <a name="does-my-azure-account-need-permissions-to-create-vms"></a>¿La cuenta de Azure necesita permisos para crear máquinas virtuales?
Si es administrador de una suscripción, ya tiene los permisos de replicación que necesita. Si no es así, necesita permisos para crear una máquina virtual de Azure en el grupo de recursos y en la red virtual que especifica al configurar Site Recovery, además de permisos para escribir en la cuenta de almacenamiento seleccionada. [Más información](site-recovery-role-based-linked-access-control.md#permissions-required-to-enable-replication-for-new-virtual-machines).

### <a name="is-replication-data-sent-to-site-recovery"></a>¿Los datos de replicación se envían a Site Recovery?
No, Site Recovery no intercepta los datos replicados ni tiene información sobre lo que se ejecuta en las máquinas virtuales. Los datos de replicación se intercambian entre los hosts de Hyper-V y Azure Storage. Site Recovery no tiene capacidad para interceptar los datos. Únicamente se envían los metadatos necesarios para coordinar la replicación y la conmutación por error al servicio Site Recovery.  

Site Recovery está certificado según la norma ISO 27001:2013, 27018, además de HIPAA y DPA, y está completando sus evaluaciones de SOC2 y FedRAMP JAB.

### <a name="can-we-keep-on-premises-metadata-within-a-geographic-region"></a>¿Podemos conservar los metadatos locales dentro de una región geográfica?
Sí. Cuando crea un almacén en una región, nos aseguramos de que todos los datos que Site Recovery usa permanezcan dentro del límite geográfico de esa región.

### <a name="does-site-recovery-encrypt-replication"></a>¿Site Recovery cifra la replicación?
Sí, se admite tanto el cifrado en tránsito como el [cifrado en Azure](../storage/common/storage-service-encryption.md).


## <a name="deployment"></a>Implementación

### <a name="what-can-i-do-with-hyper-v-to-azure-replication"></a>¿Qué puedo hacer con la replicación de Hyper-V en Azure?

- **Recuperación ante desastres**: puede configurar la recuperación ante desastres completa. En este escenario, replica máquinas virtuales de Hyper-V locales en Azure Storage:
    - Puede replicar máquinas virtuales en Azure. Si la infraestructura local no está disponible, conmute por error en Azure.
    - Cuando conmuta por error, las máquinas virtuales de Azure se crean con los datos replicados. Puede tener acceso a aplicaciones y cargas de trabajo en las máquinas virtuales de Azure.
    - Cuando el centro de datos local vuelva a estar disponible, puede realizar una conmutación por recuperación desde Azure en el sitio local.
- **Migración**: puede usar Site Recovery para migrar máquinas virtuales de Hyper-V locales a Azure Storage. Luego, conmuta por error desde el entorno local a Azure. Después de la conmutación por error, las aplicaciones y cargas de trabajo están disponibles y se ejecutan en máquinas virtuales de Azure.


### <a name="what-do-i-need-on-premises"></a>¿Qué necesito tener en el entorno local?

Necesita una o varias máquinas virtuales que se ejecuten en uno o varios hosts de Hyper-V independientes o en clúster. También puede replicar las máquinas virtuales que se ejecutan en hosts administrados por System Center Virtual Machine Manager (VMM).
- Si no ejecuta VMM, durante la implementación de Site Recovery, recopila los hosts y clústeres de Hyper-V en sitios de Hyper-V. Instale los agentes de Site Recovery, es decir, el Proveedor de Azure Site Recovery y el agente de Recovery Services, en cada host de Hyper-V.
- Si los hosts de Hyper-V se encuentran en una nube de VMM, organice la replicación en VMM. Instale el proveedor de Site Recovery en el servidor de VMM y el agente de Recovery Services en cada host de Hyper-V. Realice asignaciones entre las redes de máquina virtual o las redes lógicas de VMM y las redes virtuales de Azure.
- Obtenga [más información](hyper-v-azure-architecture.md) sobre la arquitectura de Hyper-V en Azure.

### <a name="can-i-replicate-vms-located-on-a-hyper-v-cluster"></a>¿Puedo replicar máquinas virtuales ubicadas en un clúster de Hyper-V?

Sí, Site Recovery admite los hosts de Hyper-V en clústeres. Observe lo siguiente:

- Todos los nodos del clúster deben estar registrados en el mismo almacén.
- Si no usa VMM, todos los hosts de Hyper-V del clúster se deben agregar al mismo sitio de Hyper-V.
- Instale el proveedor de Azure Site Recovery y el agente de Recovery Services en cada host de Hyper-V del clúster y agregue cada host a un sitio de Hyper-V.
- No es necesario realizar pasos específicos en el clúster.
- Si ejecuta la herramienta Deployment Planner para Hyper-V, la herramienta recopila los datos de perfil del nodo que está en ejecución y en el que también se ejecuta la máquina virtual. La herramienta no puede recopilar los datos de un nodo que está desactivado, pero realizará el seguimiento de ese nodo. Una vez que el nodo está en funcionamiento, la herramienta empieza a recopilar de él los datos de perfil de la máquina virtual (si la máquina virtual forma parte de la lista de máquinas virtuales de perfil y está en ejecución en el nodo).
- Si una máquina virtual en un host de Hyper-V de un almacén de Site Recovery se migra a un host de Hyper-V distinto en el mismo clúster, o bien a un host independiente, la replicación de la máquina virtual no resulta afectada. El host de Hyper-V debe cumplir los [requisitos previos](hyper-v-azure-support-matrix.md#on-premises-servers) y estar configurado en un almacén de Site Recovery. 


### <a name="can-i-protect-vms-when-hyper-v-is-running-on-a-client-operating-system"></a>¿Se pueden proteger las máquinas virtuales cuando se ejecuta Hyper-V en un sistema operativo cliente?
No, las máquinas virtuales deben estar ubicadas en un servidor host de Hyper-V en el que se esté ejecutando una máquina de servidor de Windows compatible. Si necesita proteger un equipo cliente, podría [replicarlo como una máquina física](physical-azure-disaster-recovery.md) en Azure.

### <a name="can-i-replicate-hyper-v-generation-2-virtual-machines-to-azure"></a>¿Se pueden replicar máquinas virtuales de generación 2 de Hyper-V a Azure?
Sí. Site Recovery convierte de la generación 2 a la generación 1 durante una conmutación por error. En la conmutación por recuperación, la máquina se convierte de nuevo a la generación 2.

### <a name="can-i-automate-site-recovery-scenarios-with-an-sdk"></a>¿Se pueden automatizar escenarios de Site Recovery con un SDK?
Sí. Puede automatizar los flujos de trabajo de Site Recovery mediante la API de Rest, PowerShell o el SDK de Azure. Escenarios admitidos actualmente para la replicación de Hyper-V en Azure mediante PowerShell:

- [Replicación de Hyper-V sin VMM mediante PowerShell](hyper-v-azure-powershell-resource-manager.md)
- [Replicación de Hyper-V con VMM mediante PowerShell](hyper-v-vmm-powershell-resource-manager.md)

## <a name="replication"></a>Replicación

### <a name="where-do-on-premises-vms-replicate-to"></a>¿A dónde se replican las máquinas virtuales locales?
Los datos se replican a Azure Storage. Cuando se ejecuta una conmutación por error, Site Recovery crea automáticamente máquinas virtuales de Azure a partir de la cuenta de almacenamiento.

### <a name="what-apps-can-i-replicate"></a>¿Qué aplicaciones se pueden replicar?
Puede replicar cualquier aplicación o carga de trabajo en ejecución en una máquina virtual de Hyper-V que cumpla con los [requisitos de replicación](hyper-v-azure-support-matrix.md#replicated-vms). Site Recovery proporciona compatibilidad para la replicación compatible con aplicaciones, para que estas se puedan conmutar por error y por recuperación a un estado inteligente. Site Recovery se integra con aplicaciones de Microsoft como SharePoint, Exchange, Dynamics, SQL Server y Active Directory; además, colabora estrechamente con los principales proveedores, como Oracle, SAP, IBM y Red Hat. [Obtenga más información](site-recovery-workload.md) acerca de la protección de la carga de trabajo.

### <a name="whats-the-replication-process"></a>¿Cuál es el proceso de replicación?

1. Cuando se desencadena la replicación inicial, se toma una instantánea de la máquina virtual de Hyper-V.
2. Los discos duros virtuales en la máquina virtual se replican uno a uno hasta que se copian todos en Azure. Esto puede tardar unos minutos, en función del tamaño de la máquina virtual y del ancho de banda de la red. Aprenda a aumentar el ancho de banda de red.
3. Si se producen cambios en el disco con la replicación inicial en curso, el seguimiento de replicaciones de Réplica de Hyper-V realiza un seguimiento de esos cambios en los registros de replicación de Hyper-V (.hrl). Estos archivos de registro se encuentran en la misma carpeta que los discos. Cada disco tiene un archivo .hrl asociado que se envía al almacenamiento secundario. Los archivos de instantáneas y de registro consumen recursos de disco mientras la replicación inicial está en curso.
4. Cuando finalice la replicación inicial, se elimina la instantánea de la máquina virtual.
5. Todos los cambios diferenciales en el registro del disco se sincronizan y se combinan en el disco primario.
6. Una vez finalizada la replicación inicial, se ejecuta el trabajo Finalizar la protección en la máquina virtual. Configura la red y otras opciones posteriores a la replicación, para que la máquina virtual esté protegida.
7. En esta fase puede comprobar la configuración de la máquina virtual para asegurarse de que está lista para la conmutación por error. Puede ejecutar una exploración de la recuperación ante desastres (conmutación por error de prueba) para la máquina virtual, a fin de comprobar que la conmutación se realiza según lo previsto.
8. Después de la replicación inicial, comienza la diferencial, según la directiva de replicación.
9. Los cambios son archivos .hrl registrados. Cada disco configurado para la replicación tiene un archivo .hrl asociado.
10. El registro se envía a la cuenta de almacenamiento del cliente. Cuando un registro se encuentra en tránsito hacia Azure, los cambios en el disco principal se siguen en otro archivo de registro de la misma carpeta.
11. Durante la replicación inicial y diferencial, puede supervisar la máquina virtual en Azure Portal.

Obtenga [más información](hyper-v-azure-architecture.md#replication-process) sobre el proceso de replicación.

### <a name="can-i-replicate-to-azure-with-a-site-to-site-vpn"></a>¿Puedo replicar en Azure con una VPN de sitio a sitio?

Site Recovery replica los datos desde el entorno local a una instancia de Azure Storage a través de un punto de conexión público o mediante el uso del emparejamiento de Microsoft de ExpressRoute. No se admite la replicación a través de una red VPN de sitio a sitio.

### <a name="can-i-replicate-to-azure-with-expressroute"></a>¿Puedo replicar en Azure con ExpressRoute?

Sí, puede usar ExpressRoute para replicar máquinas virtuales en Azure. Site Recovery replica datos en una cuenta de Azure Storage a través de un punto de conexión público y es necesario configurar el [emparejamiento de Microsoft](../expressroute/expressroute-circuit-peerings.md#microsoftpeering) para la replicación de Site Recovery. Una vez que las máquinas virtuales conmutan por error en una red virtual de Azure, puede acceder a ellas a través del [emparejamiento privado](../expressroute/expressroute-circuit-peerings.md#privatepeering).


### <a name="why-cant-i-replicate-over-vpn"></a>¿Por qué no puedo replicar a través de VPN?

Cuando se replica en Azure, el tráfico de replicación alcanza los puntos de conexión públicos de una cuenta de Azure Storage. Por lo tanto, solo puede replicar a través de una conexión a Internet pública con ExpressRoute (emparejamiento de Microsoft), ya que la conexión mediante VPN no funciona. 

### <a name="what-are-the-replicated-vm-requirements"></a>¿Cuáles son los requisitos de las máquinas virtuales replicadas?

Para realizar la replicación, una máquina virtual de Hyper-V debe ejecutar un sistema operativo compatible. Además, la máquina virtual debe cumplir con los requisitos de las máquinas virtuales de Azure. [Más información](hyper-v-azure-support-matrix.md#replicated-vms) en la matriz de compatibilidad.

### <a name="why-is-an-additional-standard-storage-account-required-if-i-replicate-my-virtual-machine-disks-to-premium-storage"></a>¿Por qué se requiere una cuenta de almacenamiento estándar adicional si replico los discos de máquina virtual en Premium Storage?

Al replicar los servidores físicos o máquinas virtuales locales en Premium Storage, todos los datos que residen en los discos de la máquina protegida se replican en la cuenta de Premium Storage. Se requiere una cuenta de almacenamiento estándar adicional para almacenar los registros de replicación. Una vez completada la fase inicial de replicación de los datos del disco, se realiza un seguimiento continuo de todos los cambios realizados en los datos del disco local y se almacenan como registros de replicación en esta cuenta de almacenamiento estándar adicional.

### <a name="how-often-can-i-replicate-to-azure"></a>¿Con qué frecuencia se puede replicar en Azure?

Las máquinas virtuales de Hyper-V se pueden replicar cada treinta segundos (excepto en el caso de Premium Storage) o cinco minutos.

### <a name="can-i-extend-replication"></a>¿Se puede extender la replicación?
No se admite la replicación extendida o encadenada. Solicite esta característica en el [foro de comentarios](https://feedback.azure.com/forums/256299-site-recovery/suggestions/6097959).

### <a name="can-i-do-an-offline-initial-replication"></a>¿Se puede realizar una replicación inicial sin conexión?
No es una opción admitida. Solicite esta característica en el [foro de comentarios](https://feedback.azure.com/forums/256299-site-recovery/suggestions/6227386-support-for-offline-replication-data-transfer-from).

### <a name="can-i-exclude-disks"></a>¿Se pueden excluir discos?
Sí, puede excluir discos de la replicación. 

### <a name="can-i-replicate-vms-with-dynamic-disks"></a>¿Se pueden replicar máquinas virtuales con discos dinámicos?
Los discos dinámicos se pueden replicar. El disco del sistema operativo debe ser un disco básico.



## <a name="security"></a>Seguridad

### <a name="what-access-does-site-recovery-need-to-hyper-v-hosts"></a>¿Qué acceso a los host de Site Recovery necesita Site Recovery?

Site Recovery necesita acceso a los hosts de Hyper-V para replicar las máquinas virtuales que seleccione. Site Recovery instala lo siguiente en los hosts de Hyper-V:

- Si no ejecuta VMM, el proveedor de Site Recovery y el agente de Recovery Services se instalan en cada host.
- Si ejecuta VMM, el agente de Recovery Services se instala en cada host. El proveedor se ejecuta en el servidor de VMM.


### <a name="what-does-site-recovery-install-on-hyper-v-vms"></a>¿Qué instala Site Recovery en las máquinas virtuales de Hyper-V?

Site Recovery no instala nada explícitamente en las máquinas virtuales de Hyper-V habilitadas para la replicación.




## <a name="failover-and-failback"></a>Conmutación por error y conmutación por recuperación


### <a name="how-do-i-fail-over-to-azure"></a>¿Cómo puedo realizar la conmutación por error en Azure?

Puede ejecutar una conmutación por error, planeada o no, desde máquinas virtuales de Hyper-V locales en Azure.

- Si ejecuta una conmutación por error planeada, las máquinas virtuales de origen se apagan para garantizar que no se pierdan datos.
- Puede ejecutar una conmutación por error no planeada, si el sitio primario no está accesible.
- Puede conmutar por error una única máquina o crear planes de recuperación para organizar la conmutación por error de varias máquinas.
- La conmutación por error consta de dos partes:
    - Una vez que se completa la primera fase de la conmutación por error, debería poder ver las máquinas virtuales de la réplica creadas en Azure. Puede asignar una dirección IP pública a la máquina virtual si es necesario.
    - Después, confirme la conmutación por error, para iniciar el acceso a la carga de trabajo desde la máquina virtual de Azure de la réplica.
   

### <a name="how-do-i-access-azure-vms-after-failover"></a>¿Cómo accedo a las máquinas virtuales de Azure después de la conmutación por error?
Después de la conmutación por error, puede tener acceso a las máquinas virtuales de Azure a través de una conexión segura a Internet o a través de una VPN de sitio a sitio o mediante Azure ExpressRoute. Debe preparar algunas cosas para la conexión. [Más información](failover-failback-overview.md#connect-to-azure-after-failover).

### <a name="is-failed-over-data-resilient"></a>¿Son resistentes los datos conmutados por error?
Azure está diseñado para la resistencia. Site Recovery está diseñado para la conmutación por error en un centro de datos de Azure secundario según el Acuerdo de Nivel de Servicio de Azure. Cuando se produce una conmutación por error, nos aseguramos de que los metadatos y los almacenes permanecen en la misma región geográfica que eligió para el almacén.

### <a name="is-failover-automatic"></a>¿La conmutación por error es automática?
La [conmutación por error](site-recovery-failover.md) no es automática. Puede iniciar las conmutaciones por error con solo un clic en el portal o bien puede usar [PowerShell](/powershell/module/az.recoveryservices) para desencadenar una conmutación por error.

### <a name="how-do-i-fail-back"></a>¿Cómo puedo realizar una conmutación por recuperación?

Una vez que la infraestructura local está activa y ejecutándose, puede conmutar por recuperación. La conmutación por recuperación se produce en tres fases:

1. Puede iniciar una conmutación por error planeada desde Azure en el sitio local con un par de opciones diferentes:

    - Minimizar el tiempo de inactividad: si usa esta opción, Site Recovery sincroniza los datos antes de la conmutación por error. Comprueba si hay bloques de datos que han cambiado y los descarga en el sitio local, mientras la máquina virtual de Azure se sigue ejecutando, lo que minimiza el tiempo de inactividad. Cuando especifica manualmente que la conmutación por error se debe completar, la máquina virtual de Azure se apaga, los cambios de deferencias finales se copian y se inicia la conmutación por error.
    - Descarga completa: con esta opción, los datos se sincronizan durante la conmutación por error. Esta opción descarga todo el disco. Es más rápido porque no se calculan las sumas de comprobación, pero hay más tiempo de inactividad. Utilice esta opción si ha ejecutado la réplica de máquinas virtuales de Azure durante algún tiempo, o si se ha eliminado la máquina virtual local.

2. Puede seleccionar conmutar por recuperación en la misma máquina virtual o en otra alternativa. Puede especificar que Site Recovery deba crear la máquina virtual si aún no existe.
3. Una vez finalizada la sincronización inicial, seleccione esta opción para completar la conmutación por error. Una vez que se completa, puede iniciar sesión en la máquina virtual local para comprobar que todo funciona según lo previsto. En Azure Portal, puede ver que las máquinas virtuales de Azure se han detenido.
4. Confirme la conmutación por error para terminar y comience a acceder de nuevo a la carga de trabajo desde la máquina virtual local.
5. Una vez que las cargas de trabajo han conmutado por recuperación, habilite la replicación inversa, para que las máquinas virtuales locales se repliquen en Azure de nuevo.

### <a name="can-i-fail-back-to-a-different-location"></a>¿Es posible conmutar por recuperación en otra ubicación?
Sí, si conmutó por error en Azure, puede conmutar por recuperación en otra ubicación si la ubicación original no está disponible. [Más información](hyper-v-azure-failback.md#fail-back-to-an-alternate-location).
