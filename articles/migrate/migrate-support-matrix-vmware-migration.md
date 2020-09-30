---
title: Compatibilidad con la migración de VMware en Azure Migrate
description: Aprenda sobre la compatibilidad con la migración de máquinas virtuales de VMware en Azure Migrate.
ms.topic: conceptual
ms.date: 06/08/2020
ms.openlocfilehash: 4c9ae6a5c3ed0d38b6abc952458422c7789fef8f
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89051124"
---
# <a name="support-matrix-for-vmware-migration"></a>Matriz de compatibilidad para la migración de VMware

En este artículo se resumen los valores de compatibilidad y las limitaciones para migrar VM de VMware con [Azure Migrate: Server Migration](migrate-services-overview.md#azure-migrate-server-migration-tool). Si busca información sobre cómo evaluar máquinas virtuales de VMware para migrar a Azure, revise la [matriz de compatibilidad de evaluación](migrate-support-matrix-vmware.md).


## <a name="migration-options"></a>Opciones de migración

Puede migrar máquinas virtuales de VMware de dos maneras:

- **Uso de la migración sin agente**: Las máquinas virtuales se migran sin necesidad de instalar nada en ellas. Se implementa el [dispositivo de Azure Migrate](migrate-appliance.md) para la migración sin agente.
- **Uso de la migración basada en agente**: Instale un agente en la máquina virtual para su replicación. En el caso de la migración basada en agente, implemente un [dispositivo de replicación](migrate-replication-appliance.md).

Revise [este artículo](server-migrate-overview.md) para averiguar qué método desea usar.

## <a name="migration-limitations"></a>Limitaciones de la migración

- Puede seleccionar hasta 10 máquinas virtuales a la vez para la replicación. Si quiere migrar más máquinas, replique en grupos de 10.
- En las migraciones sin agente de VMware se pueden ejecutar hasta 300 replicaciones a la vez.

## <a name="agentless-migration"></a>Migración sin agentes 

En esta sección se resumen los requisitos de la migración sin agente.

### <a name="vmware-requirements-agentless"></a>Requisitos de VMware (sin agente)

En la tabla se resumen los requisitos del hipervisor de VMware.

**VMware** | **Detalles**
--- | ---
**VMware vCenter Server** | Versión 5.5, 6.0, 6.5 o 6.7.
**Host ESXI de VMware vSphere** | Versión 5.5, 6.0, 6.5 o 6.7.
**Permisos de vCenter Server** | La migración sin agentes utiliza el [dispositivo de Azure Migrate](migrate-appliance.md). El dispositivo necesita estos permisos en vCenter Server:<br/><br/> - **Datastore.Browse**: Permite examinar los archivos de registro de máquina virtual para solucionar problemas de creación y eliminación de instantáneas.<br/><br/> - **Datastore.LowLevelFileOperations**: Permite operaciones de lectura, escritura, eliminación y cambio de nombre en el explorador del almacén de datos para solucionar problemas de creación y eliminación de instantáneas.<br/><br/> - **VirtualMachine.Configuration.DiskChangeTracking**: Permite habilitar o deshabilitar el seguimiento de cambios de los discos de máquina virtual para extraer los bloques de datos cambiados entre instantáneas.<br/><br/> - **VirtualMachine.Configuration.DiskLease**: Permite operaciones de concesión de discos para una máquina virtual, para leer el disco mediante el kit de desarrollo de discos virtuales de VMware vSphere (VDDK).<br/><br/> - **VirtualMachine.Provisioning.DiskAccess**: (específicamente para vSphere 6.0 y versiones posteriores) permite abrir un disco en una máquina virtual para obtener acceso de lectura aleatorio en el disco mediante VDDK.<br/><br/> - **VirtualMachine.Provisioning.ReadOnlyDiskAccess**: Permite abrir un disco en una máquina virtual para leerlo mediante VDDK.<br/><br/> - **VirtualMachine.Provisioning.DiskRandomAccess**: Permite abrir un disco en una máquina virtual para leerlo mediante VDDK.<br/><br/> - **VirtualMachine.Provisioning.VirtualMachineDownload**: Permite operaciones de lectura en los archivos asociados con una máquina virtual, para descargar los registros y solucionar problemas si se produce un error.<br/><br/> - **VirtualMachine.SnapshotManagement.\*** : Permite la creación y la administración de instantáneas de máquina virtual para la replicación.<br/><br/> - **Virtual Machine.Interaction.Power Off**: Permite apagar la máquina virtual durante la migración a Azure.



### <a name="vm-requirements-agentless"></a>Requisitos de VM (sin agente)

En la tabla se resumen los requisitos de migración sin agente para las VM de VMware.

**Soporte técnico** | **Detalles**
--- | ---
**Sistemas operativos compatibles** | Puede migrar los sistemas operativos [Windows](https://support.microsoft.com/help/2721672/microsoft-server-software-support-for-microsoft-azure-virtual-machines) y [Linux](../virtual-machines/linux/endorsed-distros.md) que admite Azure.
**VM Windows en Azure** | Es posible que tenga que [realizar algunos cambios](prepare-for-migration.md#verify-required-changes-before-migrating) en las VM antes de la migración. 
**VM Linux en Azure** | Es posible que algunas máquinas virtuales requieran cambios para poder ejecutarse en Azure.<br/><br/> En el caso de Linux, Azure Migrate realiza los cambios automáticamente para estos sistemas operativos:<br/> - Red Hat Enterprise Linux 6.5+, 7.0+<br/> - CentOS 6.5+, 7.0+</br> - SUSE Linux Enterprise Server 12 SP1+<br/> - Ubuntu 14.04LTS, 16.04LTS, 18.04LTS<br/> - Debian 7, 8. En el caso de otros sistemas operativos, realice [los cambios necesarios](prepare-for-migration.md#verify-required-changes-before-migrating) manualmente.
**Arranque de Linux** | Si/boot está en una partición dedicada, debe residir en el disco del sistema operativo y no distribuirse en varios discos.<br/> Si /boot forma parte de la partición raíz (/), la partición "/" debe estar en el disco del sistema operativo y no abarcar otros discos.
**Arranque UEFI** | Las máquinas virtuales con arranque UEFI no se admiten para la migración.
**Tamaño del disco** | Disco de sistema operativo de 2 TB y 8 TB para los discos de datos.
**Límites del disco** |  Hasta 60 discos por máquina virtual.
**Discos/volúmenes cifrados** | Las máquinas virtuales con volúmenes o discos cifrados no se admiten para la migración.
**Clúster de discos compartido** | No compatible.
**Discos independientes** | No compatible.
**Discos RDM/de acceso directo** | Si las máquinas virtuales tienen discos RDM o de acceso directo, estos discos no se replicarán en Azure.
**NFS** | Los volúmenes NFS montados como volúmenes en las máquinas virtuales no se replicarán.
**Destinos iSCSI** | Las máquinas virtuales con destinos iSCSI no se admiten para la migración sin agente.
**E/S de varias rutas** | No compatible.
**Storage vMotion** | No compatible. La replicación no funcionará si una máquina virtual usa Storage vMotion.
**NIC en equipo** | No compatible.
**IPv6** | No compatible.
**Disco de destino** | Las máquinas virtuales solo se pueden migrar a discos administrados (HDD Estándar, SSD Premium) en Azure.
**Replicación simultánea** | 300 máquinas virtuales por vCenter Server. Si tiene más, mígrelas en lotes de 300.


### <a name="appliance-requirements-agentless"></a>Requisitos de dispositivo (sin agente)

La migración sin agentes utiliza el [dispositivo de Azure Migrate](migrate-appliance.md). Puede implementar el dispositivo como una máquina virtual de VMware mediante una plantilla OVA, importada en vCenter Server, o bien mediante un [script de PowerShell](deploy-appliance-script.md).

- Obtenga más información sobre los [requisitos del dispositivo](migrate-appliance.md#appliance---vmware) para VMware.
- Obtenga información sobre las direcciones URL a las que el dispositivo necesita acceder en nubes [públicas](migrate-appliance.md#public-cloud-urls) y [gubernamentales](migrate-appliance.md#government-cloud-urls).
- En Azure Government, debe implementar el dispositivo [mediante el script](deploy-appliance-script-government.md).

### <a name="port-requirements-agentless"></a>Requisitos de puerto (sin agente)

**Dispositivo** | **Connection**
--- | ---
Dispositivo | Las conexiones salientes del puerto 443 para cargar los datos replicados en Azure y comunicarse con los servicios ce Azure Migrate que organizan la replicación y la migración.
Servidor vCenter | Conexiones salientes del puerto 443 que permiten al dispositivo organizar la replicación (crear instantáneas, copiar datos y liberar instantáneas)
Host vSphere/ESXI | Conexiones entrantes en el puerto TCP 902 para que el dispositivo replique datos de las instantáneas.

## <a name="agent-based-migration"></a>Migración basada en agente 


En esta sección se resumen los requisitos de la migración basada en agente.


### <a name="vmware-requirements-agent-based"></a>Requisitos de VMware (basado en agente)

En esta tabla se resume la compatibilidad con la evaluación y las limitaciones de los servidores de virtualización de VMware.

**Requisitos de VMware** | **Detalles**
--- | ---
**VMware vCenter Server** | Versión 5.5, 6.0, 6.5 o 6.7.
**Host ESXI de VMware vSphere** | Versión 5.5, 6.0, 6.5 o 6.7.
**Permisos de vCenter Server** | Una cuenta de solo lectura para vCenter Server.

### <a name="vm-requirements-agent-based"></a>Requisitos de VM (basada en agente)

En la tabla se resume la compatibilidad de las máquinas virtuales de VMware con las máquinas virtuales de VMware que desea migrar mediante la migración basada en agente.

**Soporte técnico** | **Detalles**
--- | ---
**Carga de trabajo de la máquina** | Azure Migrate admite la migración de cualquier carga de trabajo (por ejemplo, Active Directory, SQL Server, etc.) que se ejecute en una máquina compatible.
**Sistemas operativos** | Para conocer la información más reciente, revise la [compatibilidad del sistema operativo](../site-recovery/vmware-physical-azure-support-matrix.md#replicated-machines) de Site Recovery. Azure Migrate proporciona compatibilidad idéntica con el sistema operativo de máquina virtual.
**Sistema de archivos Linux/almacenamiento de invitados** | Para conocer la información más reciente, revise la [compatibilidad del sistema de archivos Linux](../site-recovery/vmware-physical-azure-support-matrix.md#linux-file-systemsguest-storage) de Site Recovery. Azure Migrate tiene compatibilidad idéntica con el sistema de archivos Linux.
**Red y almacenamiento** | Para conocer la información más reciente, revise los requisitos previos de [red](../site-recovery/vmware-physical-azure-support-matrix.md#network) y [almacenamiento](../site-recovery/vmware-physical-azure-support-matrix.md#storage) de Site Recovery. Azure Migrate proporciona requisitos idénticos de red y almacenamiento.
**Requisitos de Azure** | Para conocer la información más reciente, revise los requisitos de [red](../site-recovery/vmware-physical-azure-support-matrix.md#azure-vm-network-after-failover), [almacenamiento](../site-recovery/vmware-physical-azure-support-matrix.md#azure-storage) y [proceso](../site-recovery/vmware-physical-azure-support-matrix.md#azure-compute) de Azure para Site Recovery. Azure Migrate tiene requisitos idénticos para la migración de VMware.
**Servicio de movilidad** | El agente del servicio de movilidad se debe instalar en cada máquina virtual que quiera migrar.
**Arranque UEFI** | Compatible.
**UEFI: arranque seguro**         | No se admiten para la migración.
**Disco de destino** | Las máquinas virtuales solo se pueden migrar a discos administrados (HDD Estándar, SSD Premium) en Azure.
**Tamaño del disco** | Disco de sistema operativo de 2 TB y 8 TB para los discos de datos.
**Límites del disco** |  Hasta 63 discos por máquina virtual.
**Discos/volúmenes cifrados** | Las máquinas virtuales con volúmenes o discos cifrados no se admiten para la migración.
**Clúster de discos compartido** | No compatible.
**Discos independientes** | Compatible.
**Discos de acceso directo** | Compatible.
**NFS** | Los volúmenes NFS montados como volúmenes en las máquinas virtuales no se replicarán.
**Destinos iSCSI** | Compatible.
**E/S de varias rutas** | No compatible.
**Storage vMotion** | Compatible
**NIC en equipo** | No compatible.
**IPv6** | No compatible.




### <a name="appliance-requirements-agent-based"></a>Requisitos de dispositivo (basado en agente)

Al configurar el dispositivo de replicación con la plantilla OVA proporcionada en el concentrador de Azure Migrate, el dispositivo ejecuta Windows Server 2016 y cumple los requisitos de compatibilidad. Si configura el dispositivo de replicación manualmente en un servidor físico, asegúrese de que cumple los requisitos.

- Obtenga más información sobre los [requisitos del dispositivo de replicación](migrate-replication-appliance.md#appliance-requirements) para VMware.
- MySQL debe estar instalado en el dispositivo. Obtenga más información sobre las [opciones de instalación](migrate-replication-appliance.md#mysql-installation).
- Obtenga información sobre las direcciones URL a las que el dispositivo de replicación necesita acceder en nubes [públicas](migrate-replication-appliance.md#url-access) y [gubernamentales](migrate-replication-appliance.md#azure-government-url-access).
- Revise los [puertos](migrate-replication-appliance.md#port-access) a los que tiene que acceder el dispositivo de replicación.

### <a name="port-requirements-agent-based"></a>Requisitos de puerto (basado en agente)

**Dispositivo** | **Connection**
--- | ---
Máquinas virtuales | El servicio de movilidad que se ejecuta en las máquinas virtuales se comunica con el dispositivo de replicación local (servidor de configuración) en el puerto HTTPS 443 entrante para la administración de la replicación.<br/><br/> Las máquinas virtuales envían datos de replicación al servidor de procesos (que se ejecuta en la máquina del servidor de configuración) en el puerto HTTPS 9443 entrante. Este puerto se puede modificar.
Dispositivo de replicación | El dispositivo de replicación organiza la replicación con Azure a través del puerto HTTPS 443 saliente.
Servidor de proceso | El servidor de procesos recibe los datos de la replicación, los optimiza, los cifra y los envía a Azure Storage a través del puerto 443 de salida.<br/> De forma predeterminada, el servidor de procesos se ejecuta en el dispositivo de replicación.

## <a name="azure-vm-requirements"></a>Requisitos de VM de Azure

Todas las VM locales que se replican en Azure, con migración basada en agente o sin agente, deben cumplir los requisitos de VM de Azure que se resumen en esta tabla. 

**Componente** | **Requisitos** 
--- | --- | ---
Sistema operativo invitado | Comprueba los sistemas operativos de máquinas virtuales de VMware compatibles con la migración.<br/> Puede migrar cualquier carga de trabajo que se ejecute en un sistema operativo compatible. 
Arquitectura del sistema operativo invitado | 64 bits 
Tamaño del disco del sistema operativo | Hasta 2048 GB 
Número de discos del sistema operativo | 1 
Número de discos de datos | 64 o menos 
Tamaño del disco de datos | Hasta 8095 GB
Adaptadores de red | Se admiten varios adaptadores.
VHD compartido | No compatible. 
Disco FC | No compatible. 
BitLocker | No compatible.<br/><br/> Se debe deshabilitar BitLocker antes de migrar la máquina.
Nombre de la máquina virtual | Entre 1 y 63 caracteres.<br/><br/> Restringido a letras, números y guiones.<br/><br/> El nombre de la máquina debe empezar y terminar con una letra o un número. 
Conexión después de la migración: Windows | Para conectarse a máquinas virtuales de Azure que se ejecutan en Windows después de la migración, siga estos pasos:<br/><br/> -Antes de la migración, habilite RDP en la máquina virtual local.<br/><br/> Asegúrese de que se hayan agregado las reglas de TCP y UDP para el perfil **Público**, y que RDP se permite en **Firewall de Windows** > **Aplicaciones permitidas** para todos los perfiles.<br/><br/> Para el acceso a VPN de sitio a sitio, habilite RDP y permítalo en **Firewall de Windows** -> **Aplicaciones y características permitidas** para redes de **dominio y privadas**.<br/><br/> Además, compruebe que la directiva SAN del sistema operativo está establecida en **OnlineAll**. [Más información](prepare-for-migration.md).
Conexión después de la migración: Linux | Para conectarse a máquinas virtuales de Azure después de la migración mediante SSH, siga estos pasos:<br/><br/> Antes de la migración, en la máquina local, compruebe que el servicio Secure Shell está establecido en Iniciar y que las reglas de firewall permiten una conexión SSH.<br/><br/> Tras la conmutación por error, en la máquina virtual de Azure, permita conexiones entrantes al puerto SSH para las reglas del grupo de seguridad de red de la máquina virtual conmutada por error y para la subred de Azure a la que esta se conecta.<br/><br/> Además, agregue una dirección IP pública para la máquina virtual.  


## <a name="next-steps"></a>Pasos siguientes

[Selección](server-migrate-overview.md) de una opción de migración de VMware.
