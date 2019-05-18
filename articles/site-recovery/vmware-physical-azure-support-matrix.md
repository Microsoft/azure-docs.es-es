---
title: Matriz de compatibilidad para la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos en Azure con Azure Site Recovery | Microsoft Docs
description: Aquí se resumen los sistemas operativos y los componentes admitidos para la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos en Azure mediante Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
services: site-recovery
ms.topic: conceptual
ms.date: 05/10/2019
ms.author: raynew
ms.openlocfilehash: 2d1999077f6315658dbfd69473ddf5561bd76e0b
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540588"
---
# <a name="support-matrix-for-disaster-recovery--of-vmware-vms-and-physical-servers-to-azure"></a>Matriz de compatibilidad para la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos en Azure.

En este artículo se resumen los ajustes y los componentes admitidos para la recuperación ante desastres de máquinas virtuales de VMware en Azure mediante [Azure Site Recovery](site-recovery-overview.md).

Para empezar a usar Azure Site Recovery con el escenario de implementación más sencillo, visite nuestros [tutoriales](tutorial-prepare-azure.md). Puede aprender más sobre la arquitectura de Azure Site Recovery [aquí](vmware-azure-architecture.md).

## <a name="deployment-scenario"></a>Escenario de implementación

**Escenario** | **Detalles**
--- | ---
Recuperación ante desastres de máquinas virtuales de VMware | Replicación de máquinas virtuales de VMware locales en Azure. Puede implementar este escenario en Azure Portal o mediante [PowerShell](vmware-azure-disaster-recovery-powershell.md).
Recuperación ante desastres de servidores físicos | Replicación de servidores Windows/Linux físicos locales en Azure. Puede implementar este escenario en Azure Portal.

## <a name="on-premises-virtualization-servers"></a>Servidores de virtualización locales

**Servidor** | **Requisitos** | **Detalles**
--- | --- | ---
VMware | vCenter Server 6.7, 6.5, 6.0 o 5.5, o vSphere 6.7, 6.5, 6.0 o 5.5 | Se recomienda usar vCenter Server.<br/><br/> Se recomienda que los hosts de vSphere y los servidores vCenter se encuentren en la misma red que el servidor de procesos. De forma predeterminada, los componentes del servidor de procesos se ejecutan en el servidor de configuración, por lo esta será la red en la que se configurará el servidor de configuración, a menos que elija un servidor de procesos especializado.
Físico | N/D

## <a name="site-recovery-configuration-server"></a>Servidor de configuración de Site Recovery

El servidor de configuración es una máquina local que ejecuta componentes de Site Recovery locales, incluido el servidor de configuración, el servidor de procesos y el servidor de destino maestro. Para la replicación de VMware se configura el servidor de configuración con todos los requisitos, mediante una plantilla OVF para crear una máquina virtual de VMware. Para la replicación de un servidor físico, la máquina del servidor de configuración se configura manualmente.

**Componente** | **Requisitos**
--- |---
Núcleos de CPU | 8
RAM | 16 GB
Número de discos | 3 discos<br/><br/> Los discos incluyen el disco del sistema operativo, el disco de memoria caché del servidor de procesos y la unidad de retención para la conmutación por recuperación.
Espacio libre en el disco | 600 GB de espacio necesarios para la memoria caché del servidor de procesos.
Espacio libre en el disco | 600 GB de espacio necesarios para la unidad de retención.
Sistema operativo  | Windows Server 2012 R2 o Windows Server 2016 |
Configuración regional del sistema operativo | Español (es-es)
PowerCLI | [PowerCLI 6.0](https://my.vmware.com/web/vmware/details?productId=491&downloadGroup=PCLI600R1 "PowerCLI 6.0") no es necesario para el servidor de configuración con las versiones de [9.14](https://support.microsoft.com/help/4091311/update-rollup-23-for-azure-site-recovery).
Roles de Windows Server | No habilite: <br/> - Active Directory Domain Services <br/>- Internet Information Services <br/> - Hyper-V |
Directivas de grupo| No habilite: <br/> - Impedir el acceso al símbolo del sistema. <br/> - Impedir el acceso a herramientas de edición del Registro. <br/> - Confiar en la lógica de datos adjuntos de archivos. <br/> - Activar la ejecución de scripts. <br/> [Más información](https://technet.microsoft.com/library/gg176671(v=ws.10).aspx)|
IIS | Asegúrese de lo siguiente:<br/><br/> - No hay ningún sitio web preexistente predeterminado <br/> - Habilitar la [autenticación anónima](https://technet.microsoft.com/library/cc731244(v=ws.10).aspx) <br/> - Habilitar la configuración de [FastCGI](https://technet.microsoft.com/library/cc753077(v=ws.10).aspx)  <br/> - No hay ningún sitio web o aplicación preexistente que escuche en el puerto 443<br/>
Tipo de NIC | VMXNET3 (cuando se implementa como una máquina virtual VMware)
Tipo de dirección IP | estática
Puertos | 443 se usa para la orquestación del canal de control<br/>9443 se usa para el transporte de datos

## <a name="replicated-machines"></a>Máquinas replicadas

Site Recovery admite la replicación de cualquier carga de trabajo que se ejecute en una máquina compatible.

**Componente** | **Detalles**
--- | ---
Configuración del equipo | Las máquinas que se replican en Azure deben cumplir con los [requisitos de Azure](#azure-vm-requirements).
Carga de trabajo del equipo | Site Recovery admite la replicación de cualquier carga de trabajo (por ejemplo, Active Directory, SQL Server, etc.) que se ejecute en una máquina compatible. [Más información](https://aka.ms/asr_workload).
Sistema operativo Windows | Windows Server 2019 (desde [9.22 versiones](service-updates-how-to.md#links-to-currently-supported-update-rollups)), 64-bit Windows Server 2016 (Server Core, Server con experiencia de escritorio), Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 con al menos SP1. </br> Desde, [9.24 versiones](https://support.microsoft.com/en-in/help/4503156), 64 bits de Windows 10, Windows 8.1 de 64 bits, 64-bit Windows 8, 64-bit Windows 7 (no se admite la RTM de Windows 7)</br>  [Windows Server 2008 con al menos SP2: 32 y 64 bits](migrate-tutorial-windows-server-2008.md) (solo en la migración). </br></br> No se admite Windows 2016 Nano Server.
Arquitectura del sistema operativo Linux | Sistema de 64 bits solo se admite. no se admite el sistema de 32 bits
Sistema operativo Linux | Red Hat Enterprise Linux: Desde 5.2 hasta 5.11<b>\*\*</b>, desde 6.1 hasta 6.10<b>\*\*</b> y desde 7.0 hasta 7.6 <br/><br/>CentOS: Desde 5.2 hasta 5.11<b>\*\*</b>, desde 6.1 hasta 6.10<b>\*\*</b> y desde 7.0 hasta 7.6 <br/><br/>Servidor Ubuntu 14.04 LTS [(versiones de kernel admitidas)](#ubuntu-kernel-versions)<br/><br/>Servidor Ubuntu 16.04 LTS [(versiones de kernel admitidas)](#ubuntu-kernel-versions)<br/><br/>Debian 7/Debian 8 [(versiones de kernel admitidas)](#debian-kernel-versions)<br/><br/>SUSE Linux Enterprise Server 12 SP1, SP2, SP3, SP4 [(versiones de kernel admitidas)](#suse-linux-enterprise-server-12-supported-kernel-versions)<br/><br/>SUSE Linux Enterprise Server 11 SP3<b>\*\*</b>, SUSE Linux Enterprise Server 11 SP4 * </br></br>Oracle Linux 6.4, 6.5, 6.6, 6.7, 6.8, 6.9, 6.10, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6 ejecuta el kernel compatible Red Hat o Unbreakable Enterprise Kernel Release 3, en 4 y 5 (UEK5 UEK3, que es UEK4) <br/><br/></br>- No se admite la actualización de máquinas replicadas de SUSE Linux Enterprise Server 11 SP3 a SP4. Para actualizar, deshabilite la replicación y habilítela de nuevo después de la actualización.</br></br> - [Obtenga más información](https://support.microsoft.com/help/2941892/support-for-linux-and-open-source-technology-in-azure) sobre la compatibilidad con Linux y la tecnología de código abierto en Azure. Site Recovery organiza la conmutación por error para ejecutar servidores Linux en Azure. Sin embargo, los proveedores de Linux podrían limitar la compatibilidad a solo las versiones de distribución que no hayan alcanzado el final del ciclo de vida.<br/><br/> - En las distribuciones de Linux, solo se admiten kernels de stock que forman parte del lanzamiento o la actualización de la versión secundaria de la distribución.<br/><br/> - La actualización de máquinas protegidas entre distribuciones de versiones principales de Linux no se admite. Para actualizar, deshabilite la replicación, actualice el sistema operativo y, a continuación, habilite la replicación de nuevo.<br/><br/> - Los servidores con Red Hat Enterprise Linux 5.2-5.11 o CentOS 5.2-5.11 deben tener los [componentes de Linux Integration Services (LIS)](https://www.microsoft.com/download/details.aspx?id=55106) instalados para que las máquinas puedan iniciar Azure.


### <a name="ubuntu-kernel-versions"></a>Versiones de kernel de Ubuntu


**Versión compatible** | **Versión de Azure Site Recovery Mobility Service** | **Versión de kernel** |
--- | --- | --- |
14.04 LTS | [9,24] [9,24 UR] | 3.13.0-24-Generic a 3.13.0-167-generic,<br/>3.16.0-25-generic a 3.16.0-77-generic,<br/>3.19.0-18-generic a 3.19.0-80-generic,<br/>4.2.0-18-generic a 4.2.0-42-generic,<br/>4.4.0-21-a 4.4.0-143-generic,<br/>4.15.0-1023-Azure a 4.15.0-1040-azure |
14.04 LTS | [9.23][9.23 UR] | 3.13.0-24-Generic a 3.13.0-165-generic,<br/>3.16.0-25-generic a 3.16.0-77-generic,<br/>3.19.0-18-generic a 3.19.0-80-generic,<br/>4.2.0-18-generic a 4.2.0-42-generic,<br/>4.4.0-21-a 4.4.0-142-generic,<br/>4.15.0-1023-Azure a 4.15.0-1037-azure |
14.04 LTS | [9.22][9.22 UR] | 3.13.0-24-generic a 3.13.0-164-generic,<br/>3.16.0-25-generic a 3.16.0-77-generic,<br/>3.19.0-18-generic a 3.19.0-80-generic,<br/>4.2.0-18-generic a 4.2.0-42-generic,<br/>4.4.0-21-generic a 4.4.0-140-generic,<br/>4.15.0-1023-azure a 4.15.0-1036-azure |
14.04 LTS | [9.21][9.21 UR] | 3.13.0-24-generic a 3.13.0-163-generic,<br/>3.16.0-25-generic a 3.16.0-77-generic,<br/>3.19.0-18-generic a 3.19.0-80-generic,<br/>4.2.0-18-generic a 4.2.0-42-generic,<br/>4.4.0-21-generic a 4.4.0-140-generic,<br/>4.15.0-1023-azure a 4.15.0-1035-azure |
|||
16.04 LTS | [9.23] [9,24 UR] | 4.4.0-21-a 4.4.0-143-generic,<br/>4.8.0-34-generic a 4.8.0-58-generic,<br/>4.10.0-14-generic a 4.10.0-42-generic,<br/>4.11.0-13-generic to 4.11.0-14-generic,<br/>4.13.0-16-generic a 4.13.0-45-generic,<br/>4.15.0-13-Generic a 4.15.0-46-generic<br/>4.11.0-1009-Azure a 4.11.0-1018-azure,<br/>4.13.0-1005-azure a 4.13.0-1018-azure <br/>4.15.0-1012-Azure a 4.15.0-1040-azure|
16.04 LTS | [9.23][9.23 UR] | 4.4.0-21-a 4.4.0-142-generic,<br/>4.8.0-34-generic a 4.8.0-58-generic,<br/>4.10.0-14-generic a 4.10.0-42-generic,<br/>4.11.0-13-generic to 4.11.0-14-generic,<br/>4.13.0-16-generic a 4.13.0-45-generic,<br/>4.15.0-13-Generic a 4.15.0-45-generic<br/>4.11.0-1009-azure to 4.11.0-1016-azure,<br/>4.13.0-1005-azure a 4.13.0-1018-azure <br/>4.15.0-1012-Azure a 4.15.0-1037-azure|
16.04 LTS | [9.22][9.22 UR] | 4.4.0-21-generic a 4.4.0-140-generic,<br/>4.8.0-34-generic a 4.8.0-58-generic,<br/>4.10.0-14-generic a 4.10.0-42-generic,<br/>4.11.0-13-generic to 4.11.0-14-generic,<br/>4.13.0-16-generic a 4.13.0-45-generic,<br/>4.15.0-13-generic a 4.15.0-43-generic<br/>4.11.0-1009-azure to 4.11.0-1016-azure,<br/>4.13.0-1005-azure a 4.13.0-1018-azure <br/>4.15.0-1012-azure a 4.15.0-1036-azure|
16.04 LTS | [9.21][9.21 UR] | 4.4.0-21-generic a 4.4.0-140-generic,<br/>4.8.0-34-generic a 4.8.0-58-generic,<br/>4.10.0-14-generic a 4.10.0-42-generic,<br/>4.11.0-13-generic to 4.11.0-14-generic,<br/>4.13.0-16-generic a 4.13.0-45-generic,<br/>4.15.0-13-generic a 4.15.0-42-generic<br/>4.11.0-1009-azure to 4.11.0-1016-azure,<br/>4.13.0-1005-azure a 4.13.0-1018-azure <br/>4.15.0-1012-azure a 4.15.0-1035-azure|
16.04 LTS | [9.20][9.20 UR] | 4.4.0-21-generic a 4.4.0-138-generic,<br/>4.8.0-34-generic a 4.8.0-58-generic,<br/>4.10.0-14-generic a 4.10.0-42-generic,<br/>4.11.0-13-generic to 4.11.0-14-generic,<br/>4.13.0-16-generic a 4.13.0-45-generic,<br/>4.15.0-13-generic a 4.15.0-38-generic<br/>4.11.0-1009-azure to 4.11.0-1016-azure,<br/>4.13.0-1005-azure a 4.13.0-1018-azure <br/>4.15.0-1012-azure a 4.15.0-1025-azure|

### <a name="debian-kernel-versions"></a>Versiones de kernel de Debian


**Versión compatible** | **Versión de Azure Site Recovery Mobility Service** | **Versión de kernel** |
--- | --- | --- |
Debian 7 | [9.21][9.21 UR], [9.22][9.22 UR],[9.23][9.23 UR], [9.24][9.24 UR]| 3.2.0-4-amd64 a 3.2.0-6-amd64, 3.16.0-0.bpo.4-amd64 |
|||
Debian 8 | [9.21][9.21 UR],[9.22][9.22 UR],[9.23][9.23 UR], [9.24][9.24 UR] | 3.16.0-4-amd64 a 3.16.0-7-amd64, 4.9.0-0.bpo.4-amd64 a 4.9.0-0.bpo.8-amd64 |


### <a name="suse-linux-enterprise-server-12-supported-kernel-versions"></a>Versiones de kernel admitidas de SUSE Linux Enterprise Server 12

**Versión** | **Versión de Mobility service** | **Versión de kernel** |
--- | --- | --- |
SUSE Linux Enterprise Server 12 (SP1,SP2,SP3,SP4) | [9,24] [9,24 UR] | SP1 3.12.49-11-default a 3.12.74-60.64.40-default</br></br> SP1(LTSS) 3.12.74-60.64.45-default a 3.12.74-60.64.107-default</br></br> SP2 4.4.21-69-default a 4.4.120-92.70-default</br></br>SP2(LTSS) 4.4.121-92.73-default a 4.4.121-92.101-default</br></br>SP3 4.4.73-5-default a 4.4.175-94.79-default</br></br>SP4 4.12.14-94.41-default a 4.12.14-95.6-default |
SUSE Linux Enterprise Server 12 (SP1,SP2,SP3,SP4) | [9.23][9.23 UR] | SP1 3.12.49-11-default a 3.12.74-60.64.40-default</br></br> SP1(LTSS) 3.12.74-60.64.45-default a 3.12.74-60.64.107-default</br></br> SP2 4.4.21-69-default a 4.4.120-92.70-default</br></br>SP2(LTSS) 4.4.121-92.73-default a 4.4.121-92.101-default</br></br>SP3 4.4.73-5-default a 4.4.162-94.69-default</br></br>SP4 4.12.14-94.41-default a 4.12.14-95.6-default |
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3) | [9.22][9.22 UR] | SP1 3.12.49-11-default a 3.12.74-60.64.40-default</br></br> SP1(LTSS) 3.12.74-60.64.45-default a 3.12.74-60.64.107-default</br></br> SP2 4.4.21-69-default a 4.4.120-92.70-default</br></br>SP2(LTSS) 4.4.121-92.73-default a 4.4.121-92.98-default</br></br>SP3 4.4.73-5-default a 4.4.162-94.72-default |
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3) | [9.21][9.21 UR] | SP1 3.12.49-11-default a 3.12.74-60.64.40-default</br></br> SP1(LTSS) 3.12.74-60.64.45-default a 3.12.74-60.64.107-default</br></br> SP2 4.4.21-69-default a 4.4.120-92.70-default</br></br>SP2(LTSS) 4.4.121-92.73-default a 4.4.121-92.98-default</br></br>SP3 4.4.73-5-default a 4.4.156-94.72-default |


## <a name="linux-file-systemsguest-storage"></a>Sistemas de archivos Linux/almacenamiento de invitados

**Componente** | **Compatible**
--- | ---
Sistemas de archivos | ext3, ext4, XFS
Administrador de volúmenes | Antes de la [versión 9.20](https://support.microsoft.com/en-in/help/4478871/update-rollup-31-for-azure-site-recovery), <br/> 1. Se admite el LVM. <br/> 2. no se admite /boot en el volumen LVM. <br/> 3. No se admiten varios discos del sistema operativo.<br/><br/>Desde [versión 9,20](https://support.microsoft.com/en-in/help/4478871/update-rollup-31-for-azure-site-recovery) y versiones posteriores, se admite/Boot en LVM. No se admiten varios discos del sistema operativo.
Dispositivos de almacenamiento paravirtualizados | No se admiten dispositivos que se hayan exportado con controladores paravirtualizados.
Dispositivos de E/S de bloque multicola | No compatible.
Servidores físicos con controlador de almacenamiento HP CCISS | No compatible.
Convención de nomenclatura de punto de montaje/dispositivo | El nombre de dispositivo o el nombre de punto de montaje debe ser único. Asegúrese de que los nombres de dos dispositivos o puntos de montaje no solo se diferencien en las mayúsculas y minúsculas. </br> Ejemplo: no puede asignarle a dos dispositivos de la misma máquina virtual el nombre *device1* y *Device1*.
Directorios | Antes de la [versión 9.20](https://support.microsoft.com/en-in/help/4478871/update-rollup-31-for-azure-site-recovery), <br/> 1. Los siguientes directorios (si están configurados como sistemas de archivos o particiones independientes) deben estar en el mismo disco de sistema operativo del servidor de origen: /(root), /boot, /usr, /usr/local, /var, /etc.</br>2. /boot debe estar en una partición de disco y no en un volumen LVM.<br/><br/> A partir de la [versión 9.20](https://support.microsoft.com/en-in/help/4478871/update-rollup-31-for-azure-site-recovery), las restricciones mencionadas no son aplicables. No se admite /boot en el volumen LVM en más de un disco.
Directorio raíz | No se admiten varios discos de arranque en una máquina virtual. <br/><br/> Una máquina sin disco de arranque no se puede proteger.
Espacio en disco necesario| 2 GB en la partición /root <br/><br/> 250 MB en la carpeta de instalación
XFSv5 | Las características de XFSv5 en sistemas de archivos XFS, como la suma de comprobación de metadatos, se admiten a partir de la versión 9.10 de Mobility Service. Puede usar la utilidad xfs_info para comprobar el superbloque XFS de la partición. Si `ftype` está establecido en 1, a continuación, las características de XFSv5 están en uso.
BTRFS |Desde la versión 9.22, se admite BTRFS, excepto para los escenarios siguientes</br>Si se cambia el volumen en subcarpetas del sistema de archivos BTRFS después de habilitar la protección, no se admite BTRFS. </br>Si el sistema de archivos BTRFS esté repartido entre varios discos, BTRFS no se admite.</br>Si el sistema de archivos BTRFS admite RAID, no se admite BTRFS.

## <a name="vmdisk-management"></a>Administración de máquinas virtuales o discos

**Acción** | **Detalles**
--- | ---
Cambiar el tamaño de disco en una máquina virtual replicada |  Se admite.
Agregar disco a una máquina virtual replicada | Deshabilite la replicación para la máquina virtual, agregue el disco y vuelva a habilitar la replicación. Agregar un disco en una máquina virtual de replicación no se admite en la actualidad.

## <a name="network"></a>Red

**Componente** | **Compatible**
--- | ---
Formación de equipos de adaptador de red de la red de host | Se admite en máquinas virtuales de VMware. <br/><br/>No se admite en la replicación de máquinas físicas.
VLAN de la red de host | Sí.
IPv4 de la red de host | Sí.
IPv6 de la red de host | No.
Formación de equipos de adaptador de red de la red de invitado o de servidor | No.
IPv4 de la red de invitado o de servidor | Sí.
IPv6 de la red de invitado o de servidor | No.
IP estática de la red de invitado o de servidor (Windows) | Sí.
IP estática de la red de invitado o de servidor (Linux) | Sí. <br/><br/>Las máquinas virtuales se configuran para usar DHCP en la conmutación por recuperación.
Varios adaptadores de red de la red de invitado o de servidor | Sí.


## <a name="azure-vm-network-after-failover"></a>Red de máquinas virtuales de Azure (después de la conmutación por error)

**Componente** | **Compatible**
--- | ---
Azure ExpressRoute | Sí
ILB | Sí
ELB | Sí
Administrador de tráfico de Azure | Sí
Varias NIC | Sí
Dirección IP reservada | Sí
IPv4 | Sí
Conservar la dirección IP de origen | Sí
Punto de conexión de servicio de red virtual de Azure<br/> | Sí
Redes aceleradas | No

## <a name="storage"></a>Almacenamiento
**Componente** | **Compatible**
--- | ---
Disco dinámico | El disco del sistema operativo debe ser un disco básico. <br/><br/>Los discos de datos pueden ser discos dinámicos.
Configuración del disco de Docker | No
NFS de host | Sí para VMware<br/><br/> No para servidores físicos
SAN de host (ISCSI/FC) | Sí
vSAN de host | Sí para VMware<br/><br/> N/D para servidores físicos
Varias rutas de host (MPIO) | Sí, probado con DSM de Microsoft, EMC PowerPath 5.7 SP4, EMC PowerPath DSM para CLARiiON.
Hospedaje de volúmenes virtuales (VVols) | Sí para VMware<br/><br/> N/D para servidores físicos
VMDK de invitado/servidor | Sí
Disco de clúster compartido de invitado/servidor | No
Disco cifrado de invitado/servidor | No
NFS de invitado/servidor | No
Invitado/servidor iSCSI | No
SMB 3.0 de invitado/servidor | No
RDM de invitado/servidor | Sí<br/><br/> N/D para servidores físicos
Disco de invitado/servidor > 1 TB | Sí<br/><br/>Hasta 4095 GB<br/><br/> El disco debe ser de un tamaño superior a 1024 MB.
Disco de invitado/servidor con tamaño de sector físico de 4K y lógico de 4K | Sí
Disco de invitado/servidor con tamaño de sector físico de 512 bytes y lógico de 4K | Sí
Volumen de invitado/servidor con disco seccionado > 4 TB <br/><br/>Administración de volúmenes lógicos (LVM)| Sí
Invitado/servidor: espacios de almacenamiento | No
Invitado/servidor: adición/eliminación de disco en caliente | No
Invitado/servidor: disco de exclusión | Sí
Varias rutas (MPIO) de invitado/servidor | No
Arranque EFI/UEFI de invitado/servidor | Se admite al migrar máquinas virtuales de VMware o servidores físicos que ejecutan Windows Server 2012 o posterior para Azure.<br/><br/> Solo puede replicar máquinas virtuales para la migración. No se admite la conmutación por recuperación en el entorno local.<br/><br/> El servidor no debe tener más de cuatro particiones en el disco del sistema operativo.<br/><br/> Requiere la versión de Mobility Service 9.13 o una posterior.<br/><br/> Solo se admite NTFS.

## <a name="replication-channels"></a>Canales de replicación

|**Tipo de replicación**   |**Compatible**  |
|---------|---------|
|Transferencias de datos descargados (ODX)    |       No  |
|Propagación sin conexión        |   No      |
| Azure Data Box | No


## <a name="azure-storage"></a>Almacenamiento de Azure

**Componente** | **Compatible**
--- | ---
Almacenamiento con redundancia local | Sí
Almacenamiento con redundancia geográfica | Sí
Almacenamiento con redundancia geográfica con acceso de lectura | Sí
Almacenamiento de acceso esporádico | No
Almacenamiento de acceso frecuente| No
Blobs en bloques | No
Cifrado en reposo (Cifrado del servicio Storage)| Sí
Premium Storage | Sí
Servicio Import/Export | No
Los firewalls de Azure Storage para redes virtuales se configuran en la cuenta de almacenamiento o la cuenta de almacenamiento en caché de destino (se usa para almacenar datos de replicación) | Sí
Cuentas de almacenamiento de uso general v2 (niveles de acceso frecuente y esporádico) | No

## <a name="azure-compute"></a>Azure Compute

**Característica** | **Compatible**
--- | ---
Conjuntos de disponibilidad | Sí
Zonas de disponibilidad | No
CONCENTRADOR | Sí
Discos administrados | Sí

## <a name="azure-vm-requirements"></a>Requisitos de VM de Azure

Las máquinas virtuales locales que se replican en Azure deben cumplir con los requisitos de VM de Azure que se resumen en esta tabla. Cuando Site Recovery ejecuta una comprobación de requisitos previos, se producirá un error si no se cumplen algunos de los requisitos.

**Componente** | **Requisitos** | **Detalles**
--- | --- | ---
Sistema operativo invitado | Compruebe los [sistemas operativos compatibles](#replicated-machines) con máquinas replicadas. | Se produce un error en la comprobación si no es compatible.
Arquitectura del sistema operativo invitado | 64 bits | Se produce un error en la comprobación si no es compatible.
Tamaño del disco del sistema operativo | Hasta 2048 GB | Se produce un error en la comprobación si no es compatible.
Número de discos del sistema operativo | 1 | Se produce un error en la comprobación si no es compatible.
Número de discos de datos | 64 o menos | Se produce un error en la comprobación si no es compatible.
Tamaño del disco de datos | Hasta 4095 GB | Se produce un error en la comprobación si no es compatible.
Adaptador de red | Se admiten varios adaptadores. |
VHD compartido | No compatible. | Se produce un error en la comprobación si no es compatible.
Disco FC | No compatible. | Se produce un error en la comprobación si no es compatible.
BitLocker | No compatible. | Debe deshabilitar BitLocker antes de habilitar la replicación de una máquina. |
Nombre de VM | Entre 1 y 63 caracteres.<br/><br/> Restringido a letras, números y guiones.<br/><br/> El nombre de la máquina debe empezar y terminar con una letra o un número. |  Actualice el valor de las propiedades de la máquina en Site Recovery.

## <a name="azure-site-recovery-churn-limits"></a>Límites de la actividad de Azure Site Recovery

En la tabla siguiente se proporcionan los límites de Azure Site Recovery. Estos límites se basan en nuestras pruebas, pero no pueden cubrir todas las combinaciones de E/S posibles de la aplicación. Los resultados reales pueden variar en función de la combinación de E/S de la aplicación. Para obtener mejores resultados, se recomienda encarecidamente [ejecutar la herramienta deployment planner](site-recovery-deployment-planner.md) y aplicación de forma exhaustiva las pruebas mediante la emisión de una conmutación por error de prueba para obtener una imagen real del rendimiento de la aplicación.

**Destino de almacenamiento de la replicación** | **Tamaño medio de E/S de disco de origen** |**Actividad de datos media de disco de origen** | **Actividad de datos de disco de origen total por día**
---|---|---|---
Standard Storage | 8 KB | 2 MB/s | 168 GB por disco
Disco Premium P10 o P15 | 8 KB  | 2 MB/s | 168 GB por disco
Disco Premium P10 o P15 | 16 KB | 4 MB/s |  336 GB por disco
Disco Premium P10 o P15 | 32 KB, o más | 8 MB/s | 672 GB por disco
Disco Premium P20, P30, P40 o P50 | 8 KB    | 5 MB/s | 421 GB por disco
Disco Premium P20, P30, P40 o P50 | 16 KB, o más |20 MB/s | 1684 GB por disco

**Actividad de datos de origen** | **Límite máximo**
---|---
Actividad de datos media por máquina virtual| 25 MB/s
Actividad de datos máxima entre todos los discos de una máquina virtual | 54 MB/s
Actividad de datos máxima por día admitida por un servidor de procesos | 2 TB

Estos son los números promedio si la superposición de E/S es del 30 %. Site Recovery es capaz de controlar un mayor rendimiento en función de la relación de superposición, tamaños de escritura mayores y el comportamiento real de E/S de la carga de trabajo. Los números anteriores asumen un trabajo pendiente típico de aproximadamente cinco minutos. Es decir, una vez que se cargan los datos, se procesan y se crea un punto de recuperación en menos de cinco minutos.

## <a name="vault-tasks"></a>Tareas de almacén

**Acción** | **Compatible**
--- | ---
Mover el almacén entre grupos de recursos<br/><br/> Entre las suscripciones | No
Mover el almacenamiento, la red y las máquinas virtuales de Azure entre grupos de recursos<br/><br/> Entre las suscripciones | No


## <a name="download-latest-azure-site-recovery-components"></a>Descargar los componentes más recientes de Azure Site Recovery

**Nombre** | **Descripción** | **Instrucciones de descarga de la versión más reciente**
--- | --- | ---
Servidor de configuración | Coordina las comunicaciones entre servidores de VMware locales y Azure  <br/><br/>  Se instala en servidores de VMware locales | Para obtener más información, visite nuestra orientación sobre [instalación nueva](vmware-azure-deploy-configuration-server.md) y [actualización de un componente existente a la versión más reciente](vmware-azure-manage-configuration-server.md#upgrade-the-configuration-server).
Servidor de procesos|Se instala de forma predeterminada en el servidor de configuración. Recibe los datos de la replicación; los optimiza mediante almacenamiento en caché, compresión y cifrado, y los envía a Azure Storage. A medida que crece la implementación, puede agregar más servidores de procesos independientes para controlar mayores volúmenes de tráfico de replicación.| Para obtener más información, visite nuestra orientación sobre [instalación nueva](vmware-azure-set-up-process-server-scale.md) y [actualización de un componente existente a la versión más reciente](vmware-azure-manage-process-server.md#upgrade-a-process-server).
Servicio de Movilidad | Coordina la replicación entre servidores de VMware locales o servidores físicos y el sitio secundario o Azure<br/><br/> Se instala en cada máquina virtual de VMware o servidor físico que se va a replicar | Para obtener más información, visite nuestra orientación sobre [instalación nueva](vmware-azure-install-mobility-service.md) y [actualización de un componente existente a la versión más reciente](vmware-physical-manage-mobility-service.md#update-mobility-service-from-azure-portal).

Para obtener más información acerca de las características más recientes, visite [notas de la versión más reciente](https://aka.ms/ASR_latest_release_notes).


## <a name="next-steps"></a>Pasos siguientes
[Aprenda](tutorial-prepare-azure.md) a preparar Azure para la recuperación ante desastres de las máquinas virtuales de VMware.

[9.23 UR]: https://support.microsoft.com/help/4489582/update-rollup-33-for-azure-site-recovery
[9.22 UR]: https://support.microsoft.com/help/4489582/update-rollup-33-for-azure-site-recovery
[9.21 UR]: https://support.microsoft.com/help/4485985/update-rollup-32-for-azure-site-recovery
[9.20 UR]: https://support.microsoft.com/help/4478871/update-rollup-31-for-azure-site-recovery
[9.19 UR]: https://support.microsoft.com/help/4468181/azure-site-recovery-update-rollup-30
[9.18 UR]: https://support.microsoft.com/help/4466466/update-rollup-29-for-azure-site-recovery
