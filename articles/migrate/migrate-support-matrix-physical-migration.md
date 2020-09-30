---
title: Compatibilidad para la migración de servidores físicos en Azure Migrate
description: Obtenga más información sobre la compatibilidad para la migración de servidores físicos en Azure Migrate.
ms.topic: conceptual
ms.custom: fasttrack-edit
ms.date: 06/14/2020
ms.openlocfilehash: 30c96b11fb15ca1783b6ffc574d9d258dff9da06
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89051141"
---
# <a name="support-matrix-for-physical-server-migration"></a>Matriz de compatibilidad para la migración de servidores físicos

En este artículo se resumen los valores de compatibilidad y las limitaciones para migrar servidores físicos con [Azure Migrate: Server Migration](migrate-services-overview.md#azure-migrate-server-migration-tool). Si busca información sobre cómo evaluar servidores físicos para migrar a Azure, revise la [matriz de compatibilidad de evaluación](migrate-support-matrix-physical.md).

## <a name="migrating-machines-as-physical"></a>Migración de máquinas como físicas

Puede migrar máquinas locales como servidores físicos mediante la replicación basada en agente. Con esta herramienta, puede migrar una amplia gama de máquinas a Azure:

- Servidores físicos locales.
- Máquinas virtuales virtualizadas por plataformas como Xen, KVM.
- Máquinas virtuales de Hyper-V o máquinas virtuales de VMware si por alguna razón no desea usar los flujos estándar de [Hyper-V](tutorial-migrate-hyper-v.md) o [VMware](server-migrate-overview.md).
- Máquinas virtuales que se ejecutan en nubes privadas.
- Máquinas virtuales que se ejecutan en nubes públicas, como Amazon Web Services (AWS) o Google Cloud Platform (GCP).


## <a name="migration-limitations"></a>Limitaciones de la migración

Puede seleccionar hasta diez máquinas virtuales a la vez para la replicación. Si quiere migrar más máquinas, replique en grupos de 10.


## <a name="physical-server-requirements"></a>Requisitos del servidor físico

En la tabla se resume la compatibilidad de los servidores físicos que desea migrar mediante la migración basada en agente.

**Soporte técnico** | **Detalles**
--- | ---
**Carga de trabajo de la máquina** | Azure Migrate admite la migración de cualquier carga de trabajo (por ejemplo, Active Directory, SQL Server, etc.) que se ejecute en una máquina compatible.
**Sistemas operativos** | Para conocer la información más reciente, revise la [compatibilidad del sistema operativo](../site-recovery/vmware-physical-azure-support-matrix.md#replicated-machines) de Site Recovery. Azure Migrate proporciona compatibilidad idéntica con el sistema operativo.
**Sistema de archivos Linux/almacenamiento de invitados** | Para conocer la información más reciente, revise la [compatibilidad del sistema de archivos Linux](../site-recovery/vmware-physical-azure-support-matrix.md#linux-file-systemsguest-storage) de Site Recovery. Azure Migrate proporciona compatibilidad idéntica con el sistema de archivos Linux.
**Red y almacenamiento** | Para conocer la información más reciente, revise los requisitos previos de [red](../site-recovery/vmware-physical-azure-support-matrix.md#network) y [almacenamiento](../site-recovery/vmware-physical-azure-support-matrix.md#storage) de Site Recovery. Azure Migrate proporciona requisitos idénticos de red y almacenamiento.
**Requisitos de Azure** | Para conocer la información más reciente, revise los requisitos de [red](../site-recovery/vmware-physical-azure-support-matrix.md#azure-vm-network-after-failover), [almacenamiento](../site-recovery/vmware-physical-azure-support-matrix.md#azure-storage) y [proceso](../site-recovery/vmware-physical-azure-support-matrix.md#azure-compute) de Azure para Site Recovery. Azure Migrate tiene requisitos idénticos para la migración de servidores físicos.
**Servicio de movilidad** | El agente del servicio Mobility se debe instalar en cada máquina que quiera migrar.
**Arranque UEFI** | La máquina migrada en Azure se convertirá automáticamente en una máquina virtual de Azure de arranque del BIOS. Solo se admite un servidor que ejecute Windows Server 2012 o una versión posterior.<br/><br/> El disco del sistema operativo debe tener hasta cuatro particiones y los volúmenes deben formatearse con NTFS.
**UEFI: arranque seguro**         | No se admiten para la migración.
**Disco de destino** | Las máquinas solo se pueden migrar a discos administrados (HDD Estándar, SSD Premium) en Azure.
**Tamaño del disco** | Disco de sistema operativo de 2 TB y 8 TB para los discos de datos.
**Límites del disco** |  Hasta 63 discos por máquina.
**Discos/volúmenes cifrados** |  Las máquinas con volúmenes o discos cifrados no se admiten para la migración.
**Clúster de discos compartido** | No compatible.
**Discos independientes** | Compatible.
**Discos de acceso directo** | Compatible.
**NFS** | Los volúmenes NFS montados como volúmenes en las máquinas no se replicarán.
**Destinos iSCSI** | Las máquinas con destinos iSCSI no se admiten para la migración sin agente.
**E/S de varias rutas** | No compatible.
**Storage vMotion** | Compatible
**NIC en equipo** | No compatible.
**IPv6** | No compatible.



## <a name="replication-appliance-requirements"></a>Requisitos del dispositivo de replicación

Si configura el dispositivo de replicación manualmente en un servidor físico, asegúrese de que cumple con los requisitos resumidos en la tabla. Al configurar el dispositivo de replicación de Azure Migrate como una máquina virtual de VMware con la plantilla OVA proporcionada en el concentrador de Azure Migrate, dicho dispositivo se configura con Windows Server 2016 y cumple los requisitos de compatibilidad. 

- Obtenga más información sobre los [requisitos del dispositivo de replicación](migrate-replication-appliance.md#appliance-requirements).
- MySQL debe estar instalado en el dispositivo. Obtenga más información sobre las [opciones de instalación](migrate-replication-appliance.md#mysql-installation).
- Obtenga más información sobre las [direcciones URL](migrate-replication-appliance.md#url-access) a las que tiene que acceder el dispositivo de replicación.

## <a name="azure-vm-requirements"></a>Requisitos de VM de Azure

Todas las máquinas virtuales locales que se replican en Azure deben cumplir los requisitos de máquina virtual de Azure que se resumen en esta tabla. Cuando Site Recovery ejecuta una comprobación de requisitos previos para la replicación, se producirá un error si no se cumplen algunos de los requisitos.

**Componente** | **Requisitos** | **Detalles**
--- | --- | ---
Sistema operativo invitado | Comprueba los sistemas operativos compatibles.<br/> Puede migrar cualquier carga de trabajo que se ejecute en un sistema operativo compatible. | Se produce un error en la comprobación si no es compatible.
Arquitectura del sistema operativo invitado | 64 bits | Se produce un error en la comprobación si no es compatible.
Tamaño del disco del sistema operativo | Hasta 2048 GB | Se produce un error en la comprobación si no es compatible.
Número de discos del sistema operativo | 1 | Se produce un error en la comprobación si no es compatible.
Número de discos de datos | 64 o menos | Se produce un error en la comprobación si no es compatible.
Tamaño del disco de datos | Hasta 4095 GB | Se produce un error en la comprobación si no es compatible.
Adaptadores de red | Se admiten varios adaptadores. |
VHD compartido | No compatible. | Se produce un error en la comprobación si no es compatible.
Disco FC | No compatible. | Se produce un error en la comprobación si no es compatible.
BitLocker | No compatible. | Debe deshabilitar BitLocker antes de habilitar la replicación de una máquina.
Nombre de la máquina virtual | Entre 1 y 63 caracteres.<br/> Restringido a letras, números y guiones.<br/><br/> El nombre de la máquina debe empezar y terminar con una letra o un número. |  Actualice el valor de las propiedades de la máquina en Site Recovery.
Conexión después de la migración: Windows | Para conectarse a máquinas virtuales de Azure que se ejecutan en Windows después de la migración, siga estos pasos:<br/> -Antes de la migración, habilita RDP en la máquina virtual local. Asegúrese de que se hayan agregado las reglas de TCP y UDP para el perfil **Público**, y que RDP se permite en **Firewall de Windows** > **Aplicaciones permitidas** para todos los perfiles.<br/> Para el acceso a VPN de sitio a sitio, habilite RDP y permítalo en **Firewall de Windows** -> **Aplicaciones y características permitidas** para redes de **dominio y privadas**. Además, compruebe que la directiva SAN del sistema operativo está establecida en **OnlineAll**. [Más información](prepare-for-migration.md). |
Conexión después de la migración: Linux | Para conectarse a máquinas virtuales de Azure después de la migración mediante SSH, siga estos pasos:<br/> Antes de la migración, en la máquina local, compruebe que el servicio Secure Shell está establecido en Iniciar y que las reglas de firewall permiten una conexión SSH.<br/> Tras la conmutación por error, en la máquina virtual de Azure, permita conexiones entrantes al puerto SSH para las reglas del grupo de seguridad de red de la máquina virtual conmutada por error y para la subred de Azure a la que esta se conecta. Además, agregue una dirección IP pública para la máquina virtual. |  


## <a name="next-steps"></a>Pasos siguientes

[Migración](tutorial-migrate-physical-virtual-machines.md) de servidores físicos.
