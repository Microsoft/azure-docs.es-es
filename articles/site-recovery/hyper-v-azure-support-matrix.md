---
title: Compatibilidad con la recuperación ante desastres de máquinas virtuales de Hyper-V en Azure con Azure Site Recovery
description: Se resumen los requisitos y componentes compatibles para la recuperación ante desastres de máquinas virtuales de Hyper-V en Azure con Azure Site Recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 7/14/2020
ms.author: raynew
ms.openlocfilehash: 53967ab0bec9488691ff60cdabb8fedbb6b9730e
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87386714"
---
# <a name="support-matrix-for-disaster-recovery-of-on-premises-hyper-v-vms-to-azure"></a>Matriz de compatibilidad para la recuperación ante desastres de máquinas virtuales locales de Hyper-V en Azure

En este artículo se resumen los ajustes y los componentes compatibles para la recuperación ante desastres de máquinas virtuales de Hyper-V locales en Azure mediante [Azure Site Recovery](site-recovery-overview.md).

>[!NOTE]
> Site Recovery no mueve ni almacena los datos de los clientes fuera de la región de destino, en la que se ha configurado la recuperación ante desastres para las máquinas de origen. Los clientes pueden seleccionar un almacén de Recovery Services de otra región si así lo deciden. El almacén de Recovery Services contiene metadatos, pero ningún dato real de los clientes.

## <a name="supported-scenarios"></a>Escenarios admitidos

**Escenario** | **Detalles**
--- | ---
Hyper-V con Virtual Machine Manager <br> <br>| Ahora puede realizar la recuperación ante desastres en Azure de las máquinas virtuales que se ejecutan en los hosts de Hyper-V administrados en el tejido de System Center Virtual Machine Manager.<br/><br/> Puede implementar este escenario en Azure Portal o mediante PowerShell.<br/><br/> Cuando Virtual Machine Manager administra los hosts de Hyper-V, también puede realizar la recuperación ante desastres en un sitio local secundario. Lea [este tutorial](hyper-v-vmm-disaster-recovery.md) para más información sobre este escenario.
Hyper-V sin Virtual Machine Manager | Ahora puede realizar la recuperación ante desastres en Azure de las máquinas virtuales que se ejecutan en los hosts de Hyper-V no administrados por Virtual Machine Manager.<br/><br/> Puede implementar este escenario en Azure Portal o mediante PowerShell.

## <a name="on-premises-servers"></a>Servidores locales

**Server** | **Requisitos** | **Detalles**
--- | --- | ---
Hyper-V en ejecución sin Virtual Machine Manager |  Windows Server 2019, Windows Server 2016, Windows Server 2012 R2 con las últimas actualizaciones <br/><br/> **Nota:** También se admite la instalación de Server Core de estos sistemas operativos. | Si ya ha configurado Windows Server 2012 R2 o SCVMM 2012 R2 con Azure Site Recovery, y va a actualizar el sistema operativo, siga las instrucciones de la [documentación.](upgrade-2012R2-to-2016.md)
Hyper-V en ejecución con Virtual Machine Manager | Virtual Machine Manager 2019, Virtual Machine Manager 2016, Virtual Machine Manager 2012 R2 <br/><br/> **Nota:** También se admite la instalación de Server Core de estos sistemas operativos.  | Si se usa Virtual Machine Manager, los hosts de Windows Server 2019 deben administrarse en Virtual Machine Manager 2019. De manera similar, los hosts de Windows Server 2016 se deben administrar en Virtual Machine Manager 2016.

> [!NOTE]
> Asegúrese de que .NET Framework 4.6.2 o superior esté presente en el servidor local.

## <a name="replicated-vms"></a>Máquinas virtuales replicadas


En la tabla siguiente se resume la compatibilidad de las máquinas virtuales. Site Recovery admite todas las cargas de trabajo que se ejecutan en un sistema operativo compatible.

 **Componente** | **Detalles**
--- | ---
Configuración de VM | Las máquinas virtuales que se replican en Azure deben cumplir con los [requisitos de Azure](#azure-vm-requirements).
Sistema operativo invitado | Cualquier sistema operativo invitado [compatible con Azure](../cloud-services/cloud-services-guestos-update-matrix.md#family-5-releases).<br/><br/> No se admite Windows Server 2016 Nano Server.


## <a name="vmdisk-management"></a>Administración de máquinas virtuales o discos

**Acción** | **Detalles**
--- | ---
Cambiar el tamaño de disco en una máquina virtual de Hyper-V replicada | No compatible. Deshabilite la replicación, realice el cambio y, luego, vuelva a habilitar la replicación para la VM.
Agregar disco en máquina virtual de Hyper-V replicada | No compatible. Deshabilite la replicación, realice el cambio y, luego, vuelva a habilitar la replicación para la VM.

## <a name="hyper-v-network-configuration"></a>Configuración de la red de Hyper-V

**Componente** | **Hyper-V con Virtual Machine Manager** | **Hyper-V sin Virtual Machine Manager**
--- | --- | ---
Red de host: Formación de equipos NIC | Sí | Sí
Red de host: VLAN | Sí | Sí
Red de host: IPv4 | Sí | Sí
Red de host: IPv6 | No | Sin
Red de máquina virtual invitada: Formación de equipos NIC | No | No
Red de máquina virtual invitada: IPv4 | Sí | Sí
Red de máquina virtual invitada: IPv6 | Sin | Sí
Red de máquina virtual invitada: Dirección IP estática (Windows) | Sí | Sí
Red de máquina virtual invitada: Dirección IP estática (Linux) | No | No
Red de máquina virtual invitada: Varias NIC | Sí | Sí
Proxy HTTPS | No | Sin
Acceso de vínculo privado al servicio Site Recovery | Sí. [Más información](hybrid-how-to-enable-replication-private-endpoints.md). | Sí. [Más información](hybrid-how-to-enable-replication-private-endpoints.md).




## <a name="azure-vm-network-configuration-after-failover"></a>Configuración de la red de máquina virtual de Azure (después de la conmutación por error)

**Componente** | **Hyper-V con Virtual Machine Manager** | **Hyper-V sin Virtual Machine Manager**
--- | --- | ---
Azure ExpressRoute | Sí | Sí
ILB | Sí | Sí
ELB | Sí | Sí
Administrador de tráfico de Azure | Sí | Sí
Varias NIC | Sí | Sí
IP reservada | Sí | Sí
IPv4 | Sí | Sí
Conservar la dirección IP de origen | Sí | Sí
Punto de conexión de servicio de red virtual de Azure<br/> (sin firewalls de Azure Storage) | Sí | Sí
Redes aceleradas | Sin | Sin


## <a name="hyper-v-host-storage"></a>Almacenamiento de host de Hyper-V

**Storage** | **Hyper-V con Virtual Machine Manager** | **Hyper-V sin Virtual Machine Manager**
--- | --- | --- 
NFS | N/D | N/D
SMB 3.0 | Sí | Sí
SAN (ISCSI) | Sí | Sí
Varias rutas (MPIO). Probado con:<br></br> Microsoft DSM, EMC PowerPath 5.7 SP4, EMC PowerPath DSM para CLARiiON | Sí | Sí

## <a name="hyper-v-vm-guest-storage"></a>Almacenamiento de invitado de máquina virtual de Hyper-V

**Storage** | **Hyper-V con Virtual Machine Manager** | **Hyper-V sin Virtual Machine Manager**
--- | --- | ---
VMDK | N/D | N/D
VHD/VHDX | Sí | Sí
VM de generación 2 | Sí | Sí
EFI/UEFI<br></br>La máquina virtual migrada en Azure se convertirá automáticamente en una VM de arranque del BIOS. La máquina virtual debe estar ejecutando Windows Server 2012 o una versión posterior. El disco del sistema operativo debe tener un máximo de cinco particiones y el tamaño del disco del sistema operativo debe ser inferior a 300 GB.| Sí | Sí
Disco en clúster compartido | No | No
Disco cifrado | Sin | Sin
NFS | N/D | N/D
SMB 3.0 | Sin | No
RDM | N/D | N/D
Disco > 1 TB | Sí, hasta 4095 GB | Sí, hasta 4095 GB
Disco: sector físico y lógico de 4 K | No compatible: Gen 1, Gen 2 | No compatible: Gen 1, Gen 2
Disco: Sector lógico de 4 K y sector físico de 512 bytes | Sí |  Sí
Administración de volúmenes lógicos (LVM). Se admite LVM solo en discos de datos. Azure proporciona un único disco de sistema operativo. | Sí | Sí
Volumen con disco seccionado > 1 TB | Sí | Sí
Espacios de almacenamiento | Sin | Sin
Agregar/quitar disco en caliente | No | Sin
Excluir el disco | Sí | Sí
Varias rutas (MPIO) | Sí | Sí

## <a name="azure-storage"></a>Azure Storage

**Componente** | **Hyper-V con Virtual Machine Manager** | **Hyper-V sin Virtual Machine Manager**
--- | --- | ---
Almacenamiento con redundancia local | Sí | Sí
Almacenamiento con redundancia geográfica | Sí | Sí
Almacenamiento con redundancia geográfica con acceso de lectura | Sí | Sí
Almacenamiento de acceso esporádico | Sin | Sin
Almacenamiento de acceso frecuente| Sin | No
Blobs en bloques | No | Sin
Cifrado en reposo (SSE)| Sí | Sí
Cifrado en reposo (CMK) <br></br> (Solo para conmutación por error en discos administrados)| Sí (a través del módulo PowerShell Az 3.3.0 en adelante) | Sí (a través del módulo PowerShell Az 3.3.0 en adelante)
Cifrado de datos en reposo doble <br></br> (Solo para conmutación por error en discos administrados) <br></br> Más información sobre las regiones admitidas para [Windows](../virtual-machines/windows/disk-encryption.md) y [Linux](../virtual-machines/linux/disk-encryption.md) | Sí (a través del módulo PowerShell Az 3.3.0 en adelante) | Sí (a través del módulo PowerShell Az 3.3.0 en adelante)
Premium Storage | Sí | Sí
Standard Storage | Sí | Sí
Servicio Import/Export | Sin | No
Cuentas de Azure Storage con firewall habilitado | Sí. Para almacenamiento de destino y en caché. | Sí. Para almacenamiento de destino y en caché.
Modificar cuenta de almacenamiento | No. No se puede modificar la cuenta de Azure Storage de destino después de habilitar la replicación. Para modificarla, deshabilite la recuperación ante desastres y vuelva a habilitarla. | No
Opción de transferencia segura | Sí


## <a name="azure-compute-features"></a>Características de proceso de Azure

**Característica** | **Hyper-V con Virtual Machine Manager** | **Hyper-V sin Virtual Machine Manager**
--- | --- | ---
Conjuntos de disponibilidad | Sí | Sí
CONCENTRADOR | Sí | Sí  
Discos administrados | Sí, para la conmutación por error.<br/><br/> No se admite la conmutación por recuperación de los discos administrados. | Sí, para la conmutación por error.<br/><br/> No se admite la conmutación por recuperación de los discos administrados.

## <a name="azure-vm-requirements"></a>Requisitos de VM de Azure

Las máquinas virtuales locales que se replican en Azure deben cumplir con los requisitos de VM de Azure que se resumen en esta tabla.

**Componente** | **Requisitos** | **Detalles**
--- | --- | ---
Sistema operativo invitado | Site Recovery es compatible con todos los sistemas operativos [admitidos por Azure](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc794868(v=ws.10)).  | Se producirá un error en la comprobación de los requisitos previos si no es compatible.
Arquitectura del sistema operativo invitado | 32 bits (Windows Server 2008) y 64 bits | Se producirá un error en la comprobación de los requisitos previos si no es compatible.
Tamaño del disco del sistema operativo | Hasta 2048 GB para máquinas virtuales de generación 1.<br/><br/> Hasta 300 GB para máquinas virtuales de generación 2.  | Se producirá un error en la comprobación de los requisitos previos si no es compatible.
Número de discos del sistema operativo | 1 | Se producirá un error en la comprobación de los requisitos previos si no es compatible.
Número de discos de datos | 16 o menos  | Se producirá un error en la comprobación de los requisitos previos si no es compatible.
Tamaño de VHD del disco de datos | Hasta 4095 GB | Se producirá un error en la comprobación de los requisitos previos si no es compatible.
Adaptadores de red | Se admiten varios adaptadores |
VHD compartido | No compatible | Se producirá un error en la comprobación de los requisitos previos si no es compatible.
Disco FC | No compatible | Se producirá un error en la comprobación de los requisitos previos si no es compatible.
Formato de disco duro | VHD <br/><br/> VHDX | Site Recovery convierte automáticamente VHDX en VHD cuando se conmuta por error en Azure. Cuando se realiza la conmutación por recuperación en el entorno local, las máquinas virtuales siguen usando el formato VHDX.
BitLocker | No compatible | Debe deshabilitar BitLocker antes de habilitar la replicación para una máquina virtual.
Nombre de la máquina virtual | Entre 1 y 63 caracteres. Restringido a letras, números y guiones. El nombre de la VM debe empezar y terminar con una letra o un número. | Actualice el valor de las propiedades de la máquina virtual en Site Recovery.
Tipo de máquina virtual | Generación 1<br/><br/> Generación 2: Windows | Las VM de generación 2 con un tipo de disco de SO básico, que incluye uno o dos volúmenes de datos con el formato VHDX y menos de 300 GB de espacio en disco, son compatibles.<br></br>No se admiten las VM Linux de generación 2. [Más información](https://azure.microsoft.com/blog/2015/04/28/disaster-recovery-to-azure-enhanced-and-were-listening/).|

## <a name="recovery-services-vault-actions"></a>Acciones del almacén de Recovery Services

**Acción** |  **Hyper-V con VMM** | **Hyper-V sin VMM**
--- | --- | ---
Mover el almacén entre grupos de recursos<br/><br/> Entre las suscripciones | No | Sin
Mover el almacenamiento, la red y las máquinas virtuales de Azure entre grupos de recursos<br/><br/> Entre las suscripciones | No | No

> [!NOTE]
> Al replicar máquinas virtuales de Hyper-V desde un entorno local en Azure, solo se puede replicar a un inquilino de AD de un entorno concreto (sitio Hyper-V o Hyper-V, lo que corresponda).


## <a name="provider-and-agent"></a>Proveedor y agente

Para garantizar que la implementación sea compatible con la configuración que se muestra en este artículo, asegúrese de ejecutar las versiones más recientes de proveedor y agente.

**Nombre** | **Descripción** | **Detalles**
--- | --- | --- 
Proveedor de Azure Site Recovery | Coordina las comunicaciones entre los servidores locales y Azure <br/><br/> Hyper-V con Virtual Machine Manager: Se instala en servidores de Virtual Machine Manager<br/><br/> Hyper-V sin Virtual Machine Manager: Se instala en hosts de Hyper-V| Versión más reciente: 5.1.2700.1 (disponible en Azure Portal)<br/><br/> [Características y correcciones más recientes](https://support.microsoft.com/help/4091311/update-rollup-23-for-azure-site-recovery)
Agente de Microsoft Azure Recovery Services | Coordina la replicación entre máquinas virtuales de Hyper-V y Azure<br/><br/> Se instala en servidores de Hyper-V locales (con o sin Virtual Machine Manager) | Agente más reciente disponible en el portal






## <a name="next-steps"></a>Pasos siguientes
Obtenga información sobre cómo [preparar Azure](tutorial-prepare-azure.md) para la recuperación ante desastres de las máquinas virtuales de Hyper-V locales.
