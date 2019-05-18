---
title: Matriz de compatibilidad para la recuperación ante desastres de máquinas virtuales de Azure entre regiones de Azure con Azure Site Recovery | Microsoft Docs
description: Se resumen los requisitos previos y compatibilidad para la recuperación de desastres de máquinas virtuales de Azure desde una región a otra con Azure Site Recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: article
ms.date: 05/14/2019
ms.author: raynew
ms.openlocfilehash: de2e57901becad68f3fad16967faf3ae4833177a
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65797868"
---
# <a name="support-matrix-for-replicating-azure-vms-from-one-region-to-another"></a>Matriz de compatibilidad para replicar máquinas virtuales de Azure desde una región a otra

Este artículo resumen los requisitos previos y soporte técnico cuando se establece de recuperación ante desastres de máquinas virtuales de Azure desde una región de Azure a otro, mediante el [Azure Site Recovery](site-recovery-overview.md) service.


## <a name="deployment-method-support"></a>Compatibilidad de método de implementación

**Implementación** |  **Soporte técnico**
--- | ---
**Azure Portal** |  Se admite.
**PowerShell** |  Se admite. [Más información](azure-to-azure-powershell.md)
**API DE REST** |  Se admite.
**CLI** | No se admite actualmente.


## <a name="resource-support"></a>Compatibilidad de recursos

**Acción de recursos** | **Detalles**
--- | --- | ---
**Migrar los almacenes entre grupos de recursos** | No compatible
**Mover recursos de proceso, almacenamiento y red entre grupos de recursos** | No compatible.<br/><br/> Si mueve una máquina virtual o los componentes asociados, como el almacenamiento o la red, después de la replicación de esta, tendrá que deshabilitar la replicación y volver a habilitarla para la máquina virtual.
**Replicar máquinas virtuales de Azure de una suscripción a otra para la recuperación ante desastres** | Se admite en el mismo inquilino de Azure Active Directory.
**Migración de máquinas virtuales entre regiones dentro de los clústeres geográficos admitidos (dentro y entre suscripciones)** | Se admite en el mismo inquilino de Azure Active Directory.
**Migrar máquinas virtuales dentro de la misma región** | No compatible.

## <a name="region-support"></a>Regiones admitidas

Puede replicar y recuperar máquinas virtuales entre dos regiones cualesquiera dentro del mismo clúster geográfico. Los clústeres geográficos se definen teniendo presente la latencia y la soberanía de datos.


**Clúster geográfico** | **Regiones de Azure**
-- | --
América | Centro de Canadá y este de Canadá, centro-sur de EE. UU., centro-oeste de EE. UU., este de EE. UU., este de EE. UU. 2, oeste de EE. UU., oeste de EE. UU. 2 centro de EE. UU., centro-norte de EE. UU.
Europa | Oeste de Reino Unido, sur de Reino Unido, Europa del Norte, Europa occidental, sur de Francia, sur de Francia, Sudáfrica occidental, centro y Norte de Sudáfrica
Asia | India del Sur, centro de la India, Sudeste Asiático, Asia Oriental, Japón Oriental, Japón Occidental, Corea Central, Corea del Sur
Australia   | Este de Australia, Sudeste de Australia, Centro de Australia, Centro de Australia 2
Azure Government    | Virginia Gob. EE. UU., Iowa Gob. EE. UU., US Gov de Arizona, US Gov de Texas, US DOD este, US DOD central
Alemania | Centro de Alemania y Noreste de Alemania
China | Este de China, Norte de China, Norte de China 2 y Este de China 2

>[!NOTE]
>
> - Para **sur de Brasil**, puede replicar y conmutar por error a estas regiones: EE.UU. Central sur, centro occidental de EE.UU, este de Estados Unidos, East US 2, oeste de Estados Unidos, oeste de EE.UU. 2 y North Central US.
> - Sur de Brasil sólo puede utilizarse como una región de origen desde el que las máquinas virtuales pueden replicar con Site Recovery. No puede actuar como una región de destino. Esto es debido a problemas de latencia debido a distancias geográficas.
> - Puede trabajar dentro de las regiones para las que tenga acceso adecuado.
> - Si no aparece la región en la que desea crear un almacén, asegúrese de que su suscripción tiene acceso para crear recursos en esa región.
> - Si no ve una región dentro de un clúster geográfico al habilitar la replicación, asegúrese de que su suscripción tiene permisos para crear máquinas virtuales en dicha región.



## <a name="cache-storage"></a>Almacenamiento en caché

Esta tabla resume la compatibilidad con la cuenta de almacenamiento de la memoria caché que usa Site Recovery durante la replicación.

**Configuración** | **Soporte técnico** | **Detalles**
--- | --- | ---
Cuentas de almacenamiento de uso general V2 (capas de acceso frecuente y esporádico) | No compatible. | Existe la limitación para el almacenamiento en caché ya que los costos de las transacciones para V2 son significativamente más altos que los de las cuentas de almacenamiento V1.
Firewalls de Azure Storage para redes virtuales  | Compatible | Si usa una cuenta de almacenamiento en caché o de destino con firewall habilitado, asegúrese de seleccionar ["Permitir servicios de Microsoft de confianza"](https://docs.microsoft.com/azure/storage/common/storage-network-security#exceptions).


## <a name="replicated-machine-operating-systems"></a>Sistemas operativos de máquinas replicadas

Site Recovery admite la replicación de máquinas virtuales de Azure que ejecutan los sistemas operativos enumerados en esta sección.

### <a name="windows"></a> Windows

**Sistema operativo** | **Detalles**
--- | ---
Windows Server 2019 |
Windows Server 2016  | Server Core y Server con Experiencia de escritorio
Windows Server 2012 R2 |
Windows Server 2012 |
Windows Server 2008 R2 | Con ejecución de SP1 o posterior
Windows 10 (x64) |
Windows 8.1 (x64) |
Windows 8 (x64) |
Windows 7 (x64) | Ejecute SP1 o posterior (no se admite la RTM de Windows 7)

#### <a name="linux"></a>Linux

**Sistema operativo** | **Detalles**
--- | ---
Red Hat Enterprise Linux | 6.7, 6.8, 6.9, 6.10, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6  
CentOS | 6.5, 6.6, 6.7, 6.8, 6.9, 6.10, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6
Servidor Ubuntu 14.04 LTS | [Versiones de kernel admitidas](#supported-ubuntu-kernel-versions-for-azure-virtual-machines)
Servidor Ubuntu 16.04 LTS | [Versión de kernel admitida](#supported-ubuntu-kernel-versions-for-azure-virtual-machines)<br/><br/> Servidores Ubuntu que utilizan autenticación basada en contraseña y el inicio de sesión y el paquete de cloud-init para configurar las máquinas virtuales, en la nube podrían tener el inicio de sesión basado en contraseña deshabilitado en la conmutación por error (según la configuración de dicha inicialización). Inicio de sesión basado en contraseña puede habilitarse de nuevo en la máquina virtual al restablecer la contraseña de la compatibilidad con > solución de problemas > menú Configuración (de la VM conmutada por error en el portal de Azure.
Debian 7 | [Versiones de kernel admitidas](#supported-debian-kernel-versions-for-azure-virtual-machines)
Debian 8 | [Versiones de kernel admitidas](#supported-debian-kernel-versions-for-azure-virtual-machines)
SUSE Linux Enterprise Server 12 | SP1, SP2, SP3, SP4. [(Versiones de kernel admitidas)](#supported-suse-linux-enterprise-server-12-kernel-versions-for-azure-virtual-machines)
SUSE Linux Enterprise Server 11 | SP3<br/><br/> No se admite la actualización de máquinas de replicación de SP3 a SP4. Si se ha actualizado una máquina replicada, deberá deshabilitar la replicación y volverla a habilitar después de la actualización.
SUSE Linux Enterprise Server 11 | SP4
Oracle Linux | 6.4, 6.5, 6.6, 6.7, 6.8, 6.9, 6.10, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5 <br/><br/> Ejecutan el kernel compatible de Red Hat o Unbreakable Enterprise Kernel Release 3 (UEK3).


#### <a name="supported-ubuntu-kernel-versions-for-azure-virtual-machines"></a>Versiones de kernel de Ubuntu admitidas para máquinas virtuales de Azure

**Versión** | **Versión de Mobility service** | **Versión de kernel** |
--- | --- | --- |
14.04 LTS | 9.24 | 3.13.0-24-Generic a 3.13.0-167-generic,<br/>3.16.0-25-generic a 3.16.0-77-generic,<br/>3.19.0-18-generic a 3.19.0-80-generic,<br/>4.2.0-18-generic a 4.2.0-42-generic,<br/>4.4.0-21-a 4.4.0-143-generic,<br/>4.15.0-1023-Azure a 4.15.0-1040-azure |
14.04 LTS | 9.23 | 3.13.0-24-Generic a 3.13.0-165-generic,<br/>3.16.0-25-generic a 3.16.0-77-generic,<br/>3.19.0-18-generic a 3.19.0-80-generic,<br/>4.2.0-18-generic a 4.2.0-42-generic,<br/>4.4.0-21-a 4.4.0-142-generic,<br/>4.15.0-1023-Azure a 4.15.0-1037-azure |
14.04 LTS | 9.22 | 3.13.0-24-generic a 3.13.0-164-generic,<br/>3.16.0-25-generic a 3.16.0-77-generic,<br/>3.19.0-18-generic a 3.19.0-80-generic,<br/>4.2.0-18-generic a 4.2.0-42-generic,<br/>4.4.0-21-generic a 4.4.0-140-generic,<br/>4.15.0-1023-azure a 4.15.0-1036-azure |
14.04 LTS | 9.21 | 3.13.0-24-generic a 3.13.0-163-generic,<br/>3.16.0-25-generic a 3.16.0-77-generic,<br/>3.19.0-18-generic a 3.19.0-80-generic,<br/>4.2.0-18-generic a 4.2.0-42-generic,<br/>4.4.0-21-generic a 4.4.0-140-generic,<br/>4.15.0-1023-azure a 4.15.0-1035-azure |
|||
16.04 LTS | 9.24 | 4.4.0-21-a 4.4.0-143-generic,<br/>4.8.0-34-generic a 4.8.0-58-generic,<br/>4.10.0-14-generic a 4.10.0-42-generic,<br/>4.11.0-13-generic to 4.11.0-14-generic,<br/>4.13.0-16-generic a 4.13.0-45-generic,<br/>4.15.0-13-Generic a 4.15.0-46-generic<br/>4.11.0-1009-Azure a 4.11.0-1018-azure,<br/>4.13.0-1005-azure a 4.13.0-1018-azure <br/>4.15.0-1012-Azure a 4.15.0-1040-azure|
16.04 LTS | 9.23 | 4.4.0-21-a 4.4.0-142-generic,<br/>4.8.0-34-generic a 4.8.0-58-generic,<br/>4.10.0-14-generic a 4.10.0-42-generic,<br/>4.11.0-13-generic to 4.11.0-14-generic,<br/>4.13.0-16-generic a 4.13.0-45-generic,<br/>4.15.0-13-Generic a 4.15.0-45-generic<br/>4.11.0-1009-azure to 4.11.0-1016-azure,<br/>4.13.0-1005-azure a 4.13.0-1018-azure <br/>4.15.0-1012-Azure a 4.15.0-1037-azure|
16.04 LTS | 9.22 | 4.4.0-21-generic a 4.4.0-140-generic,<br/>4.8.0-34-generic a 4.8.0-58-generic,<br/>4.10.0-14-generic a 4.10.0-42-generic,<br/>4.11.0-13-generic to 4.11.0-14-generic,<br/>4.13.0-16-generic a 4.13.0-45-generic,<br/>4.15.0-13-generic a 4.15.0-43-generic<br/>4.11.0-1009-azure to 4.11.0-1016-azure,<br/>4.13.0-1005-azure a 4.13.0-1018-azure <br/>4.15.0-1012-azure a 4.15.0-1036-azure|
16.04 LTS | 9.21 | 4.4.0-21-generic a 4.4.0-140-generic,<br/>4.8.0-34-generic a 4.8.0-58-generic,<br/>4.10.0-14-generic a 4.10.0-42-generic,<br/>4.11.0-13-generic to 4.11.0-14-generic,<br/>4.13.0-16-generic a 4.13.0-45-generic,<br/>4.15.0-13-generic a 4.15.0-42-generic<br/>4.11.0-1009-azure to 4.11.0-1016-azure,<br/>4.13.0-1005-azure a 4.13.0-1018-azure <br/>4.15.0-1012-azure a 4.15.0-1035-azure|

#### <a name="supported-debian-kernel-versions-for-azure-virtual-machines"></a>Versiones de kernel de Debian admitidas para máquinas virtuales de Azure

**Versión** | **Versión de Mobility service** | **Versión de kernel** |
--- | --- | --- |
Debian 7 | 9.21,9.22,9.23,9.24 | 3.2.0-4-amd64 a 3.2.0-6-amd64, 3.16.0-0.bpo.4-amd64 |
|||
Debian 8 | 9.21, 9.22, 9.23, 9.24 | 3.16.0-4-amd64 a 3.16.0-7-amd64, 4.9.0-0.bpo.4-amd64 a 4.9.0-0.bpo.8-amd64 |

#### <a name="supported-suse-linux-enterprise-server-12-kernel-versions-for-azure-virtual-machines"></a>Versiones de kernel admitidas de SUSE Linux Enterprise Server 12 para máquinas virtuales de Azure

**Versión** | **Versión de Mobility service** | **Versión de kernel** |
--- | --- | --- |
SUSE Linux Enterprise Server 12 (SP1,SP2,SP3,SP4) | 9.24 | SP1 3.12.49-11-default a 3.12.74-60.64.40-default</br></br> SP1(LTSS) 3.12.74-60.64.45-default a 3.12.74-60.64.107-default</br></br> SP2 4.4.21-69-default a 4.4.120-92.70-default</br></br>SP2(LTSS) 4.4.121-92.73-default a 4.4.121-92.104-default</br></br>SP3 4.4.73-5-default a 4.4.176-94.88-default</br></br>SP4 4.12.14-94.41-default a 4.12.14-95.13-default |
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3, SP4) | 9.23 | SP1 3.12.49-11-default a 3.12.74-60.64.40-default</br></br> SP1(LTSS) 3.12.74-60.64.45-default a 3.12.74-60.64.107-default</br></br> SP2 4.4.21-69-default a 4.4.120-92.70-default</br></br>SP2(LTSS) 4.4.121-92.73-default a 4.4.121-92.101-default</br></br>SP3 4.4.73-5-default a 4.4.162-94.69-default</br></br>SP4 4.12.14-94.41-default a 4.12.14-95.6-default |
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3) | 9.22 | SP1 3.12.49-11-default a 3.12.74-60.64.40-default</br></br> SP1(LTSS) 3.12.74-60.64.45-default a 3.12.74-60.64.107-default</br></br> SP2 4.4.21-69-default a 4.4.120-92.70-default</br></br>SP2(LTSS) 4.4.121-92.73-default a 4.4.121-92.98-default</br></br>SP3 4.4.73-5-default a 4.4.162-94.72-default |
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3) | 9.21 | SP1 3.12.49-11-default a 3.12.74-60.64.40-default</br></br> SP1(LTSS) 3.12.74-60.64.45-default a 3.12.74-60.64.107-default</br></br> SP2 4.4.21-69-default a 4.4.120-92.70-default</br></br>SP2(LTSS) 4.4.121-92.73-default a 4.4.121-92.98-default</br></br>SP3 4.4.73-5-default a 4.4.162-94.72-default |

## <a name="replicated-machines---linux-file-systemguest-storage"></a>Máquinas replicadas: Sistema de archivos o almacenamiento de invitado de Linux

* Sistemas de archivos: ext3, ext4, ReiserFS (solo Suse Linux Enterprise Server), XFS, BTRFS
* Administrador de volúmenes: LVM2
* Software de múltiples rutas: Mapeador de dispositivo


## <a name="replicated-machines---compute-settings"></a>Máquinas replicadas: Configuración de proceso

**Configuración** | **Soporte técnico** | **Detalles**
--- | --- | ---
Tamaño | Cualquier tamaño de máquina virtual de Azure con al menos 2 núcleos de CPU y 1 GB de RAM | Compruebe los [tamaños de máquina virtual de Azure](../virtual-machines/windows/sizes.md).
Conjuntos de disponibilidad | Compatible | Si habilita la replicación para una máquina virtual de Azure con las opciones predeterminadas, un conjunto de disponibilidad se crea automáticamente, según la configuración de la región de origen. Puede modificar estos valores.
Zonas de disponibilidad | Compatible |
Ventaja de uso híbrido (HUB) | Compatible | Si la máquina virtual de origen tiene habilitada una licencia HUB, en la conmutación por error de prueba o la máquina virtual de conmutación por error también se usa la licencia HUB.
VM Scale Sets | No compatible |
Imágenes de la galería de Azure (publicadas por Microsoft) | Compatible | Se admiten si la máquina virtual se ejecuta en un sistema operativo compatible.
Imágenes de la Galería de Azure (publicadas por terceros) | Compatible | Se admiten si la máquina virtual se ejecuta en un sistema operativo compatible.
Imágenes personalizadas (publicadas de terceros) | Compatible | Se admiten si la máquina virtual se ejecuta en un sistema operativo compatible.
Máquinas virtuales migradas con Site Recovery | Compatible | Si se migró una máquina virtual de VMware o una física a Azure mediante Site Recovery, deberá desinstalar la versión antigua de Mobility Service que se ejecuta en la máquina y reiniciarla antes de replicarla en otra región de Azure.
Directivas de RBAC | No compatible | Directivas de control (RBAC) en máquinas virtuales no se replican en la máquina virtual en la región de destino de conmutación por error de acceso basado en roles.
Extensiones | No compatible | Las extensiones no se replican en la máquina virtual en la región de destino de conmutación por error. Debe instalarse manualmente después de la conmutación por error.

## <a name="replicated-machines---disk-actions"></a>Máquinas replicadas: acciones de disco

**Acción** | **Detalles**
-- | ---
Cambiar el tamaño de disco en una máquina virtual replicada | Compatible
Agregar un disco a una máquina virtual replicada | No compatible.<br/><br/> Tiene que deshabilitar la replicación para la máquina virtual, agregar el disco y volver a habilitar la replicación.

## <a name="replicated-machines---storage"></a>Máquinas replicadas: almacenamiento

Esta tabla resume la compatibilidad con el disco del sistema operativo, el disco de datos y el disco temporal de Azure VM.

- Es importante respetar los límites y destinos de los discos de máquinas virtuales [Linux](../virtual-machines/linux/disk-scalability-targets.md) y [Windows](../virtual-machines/windows/disk-scalability-targets.md) para evitar cualquier problema de rendimiento.
- Si implementa con la configuración predeterminada, Site Recovery creará automáticamente las cuentas de almacenamiento y los discos necesarios en función de la configuración de origen.
- Si desea personalizar, asegúrese de que sigue las directrices.

**Componente** | **Soporte técnico** | **Detalles**
--- | --- | ---
Tamaño máximo del disco de sistema operativo | 2048 GB | [Más información](../virtual-machines/windows/managed-disks-overview.md) sobre discos de máquina virtual.
Disco temporal | No compatible | El disco temporal se excluye de la replicación siempre.<br/><br/> No almacene los datos persistentes en el disco temporal. [Más información](../virtual-machines/windows/managed-disks-overview.md).
Tamaño máximo del disco de datos | 4095 GB |
Número máximo de discos de datos | Hasta 64, según la compatibilidad con un tamaño específico de máquina virtual de Azure | [Más información](../virtual-machines/windows/sizes.md) sobre tamaños de máquina virtual.
Tasa de cambio de disco de datos | Máximo de 10 MBps por disco para el almacenamiento Premium. Máximo de 2 MBps por disco para el almacenamiento Estándar. | Si la tasa media de cambio de los datos en el disco sobrepasa el máximo continuamente, la replicación no mantendrá el ritmo.<br/><br/>  Sin embargo, si se supera el máximo esporádicamente, la replicación podrá mantenerlo, aunque podría ver puntos de recuperación ligeramente retrasados.
Disco de datos: cuenta de almacenamiento Estándar | Compatible |
Disco de datos: cuenta de almacenamiento Premium | Compatible | Si una máquina virtual tiene discos repartidos entre cuentas de almacenamiento Estándar y Premium, puede seleccionar una cuenta de almacenamiento de destino diferente para cada disco a fin de garantizar que tenga la misma configuración de almacenamiento en la región de destino.
Disco administrado: estándar | Se admite en regiones de Azure en las que se pueda usar Azure Site Recovery. |
Disk administrado: premium | Se admite en regiones de Azure en las que se pueda usar Azure Site Recovery. |
SSD estándar | Compatible |
Redundancia | Se admiten LRS y GRS.<br/><br/> No se admite ZRS.
Almacenamiento esporádico y almacenamiento frecuente | No compatible | Los discos de máquina virtual no admiten el almacenamiento esporádico ni el frecuente
Espacios de almacenamiento | Compatible |
Cifrado en reposo (SSE) | Compatible | SSE es la configuración predeterminada en las cuentas de almacenamiento.   
Azure Disk Encryption (ADE) para sistemas operativos Windows | Se admiten las máquinas virtuales habilitadas para [cifrado con la aplicación de Azure AD](https://aka.ms/ade-aad-app) |
Azure Disk Encryption (ADE) para sistemas operativos Linux | No compatible |
En caliente | Compatible | Habilitar la replicación para un disco de datos que se agrega a una máquina virtual de Azure replicada se admite para máquinas virtuales que usan discos administrados.
Quitar disco de acceso frecuente | No compatible | Si quita el disco de datos en la máquina virtual, deberá deshabilitar la replicación y habilite la replicación de nuevo para la máquina virtual.
Excluir el disco | soporte técnico. Debe usar [Powershell](azure-to-azure-exclude-disks.md) para configurar. |  De forma predeterminada, se excluyen discos temporales.
Espacios de almacenamiento directo  | Compatible con puntos de recuperación coherentes con los bloqueos. Los puntos de recuperación coherentes de la aplicación no son compatibles. |
Servidor de archivos de escalado horizontal  | Compatible con puntos de recuperación coherentes con los bloqueos. Los puntos de recuperación coherentes de la aplicación no son compatibles. |
LRS | Compatible |
GRS | Compatible |
RA-GRS | Compatible |
ZRS | No compatible |
Almacenamiento en frío y en caliente | No compatible | Los discos de máquina virtual no admiten el almacenamiento temporal y permanente.
Firewalls de Azure Storage para redes virtuales  | Compatible | Si restringir el acceso de red virtual a las cuentas de almacenamiento, habilite [permitir que los servicios de Microsoft de confianza](https://docs.microsoft.com/azure/storage/common/storage-network-security#exceptions).
Cuentas de almacenamiento de uso general V2 (capas de acceso frecuente y esporádico) | Sin  | Los costos de transacción aumentan considerablemente en comparación con las cuentas de almacenamiento de uso general V1

>[!IMPORTANT]
> Para evitar problemas de rendimiento, asegúrese de que sigue objetivos de escalabilidad y rendimiento del disco de máquina virtual para [Linux](../virtual-machines/linux/disk-scalability-targets.md) o [Windows](../virtual-machines/windows/disk-scalability-targets.md) máquinas virtuales. Si usa la configuración predeterminada, Site Recovery crea las cuentas de almacenamiento, según la configuración de origen y los discos necesarios. Si personaliza y selecciona su propia configuración, siga los objetivos de escalabilidad y rendimiento de disco para el origen de las máquinas virtuales.

## <a name="limits-and-data-change-rates"></a>Las frecuencias de cambio de datos y límites

En la tabla siguiente se resume los límites de Site Recovery.

- Estos límites se basan en nuestras pruebas, pero obviamente no cubren todas las combinaciones de E/S posibles de la aplicación.
- Los resultados reales pueden variar en función de la combinación de E/S de la aplicación.
- Hay dos límites a tener en cuenta por disco de datos de actividad y datos de la máquina virtual por renovación.
- Por ejemplo, si usamos un disco Premium P20 como se describe en la tabla siguiente, Site Recovery puede controlar los 5 MB de la actividad por disco, con como máximo de cinco estos discos por máquina virtual, debido al límite de la actividad total de 25 MB/s por máquina virtual.

**Destino de almacenamiento** | **E/S de disco promedio de origen** |**Actividad de datos media de disco de origen** | **Actividad de datos de disco de origen total por día**
---|---|---|---
Standard Storage | 8 KB | 2 MB/s | 168 GB por disco
Disco Premium P10 o P15 | 8 KB  | 2 MB/s | 168 GB por disco
Disco Premium P10 o P15 | 16 KB | 4 MB/s |  336 GB por disco
Disco Premium P10 o P15 | 32 KB, o más | 8 MB/s | 672 GB por disco
Disco Premium P20, P30, P40 o P50 | 8 KB    | 5 MB/s | 421 GB por disco
Disco Premium P20, P30, P40 o P50 | 16 KB, o más |20 MB/s | 1684 GB por disco

## <a name="replicated-machines---networking"></a>Máquinas replicadas: redes
**Configuración** | **Soporte técnico** | **Detalles**
--- | --- | ---
NIC | Número máximo admitido para un tamaño específico de máquina virtual de Azure | Las NIC se crean cuando se crea la máquina virtual durante la conmutación por error.<br/><br/> El número de tarjetas NIC en la máquina virtual de conmutación por error viene determinado por el número de tarjetas NIC que haya en la máquina virtual de origen en el momento de habilitar la replicación. Si agrega o quita una tarjeta NIC después de habilitar la replicación, esto no influirá en el número de NIC de la máquina virtual replicada después de la conmutación por error. También observe que el orden de NIC después de la conmutación por error no se garantiza que sea el mismo que el orden original.
Equilibrador de carga de Internet | Compatible | Asocie el equilibrador de carga configurado previamente con un script de Azure Automation de un plan de recuperación.
Equilibrador de carga interno | Compatible | Asocie el equilibrador de carga configurado previamente con un script de Azure Automation de un plan de recuperación.
Dirección IP pública | Compatible | Asocie una dirección IP pública existente a la NIC. O bien, cree una dirección IP pública y asóciela con la NIC mediante un script de Azure Automation de un plan de recuperación.
Grupo de seguridad de red en una tarjeta NIC | Compatible | Asocie el grupo de seguridad de red a la NIC con un script de Azure Automation de un plan de recuperación.
Grupo de seguridad de red en la subred | Compatible | Asocie el grupo de seguridad de red a la subred con un script de Azure Automation de un plan de recuperación.
Dirección IP (estática) reservada | Compatible | Si la NIC de la máquina virtual de origen tiene una dirección IP estática y la subred de destino tiene la misma dirección IP disponible, esta se asigna a la máquina virtual de conmutación por error.<br/><br/> Si la subred de destino no tiene la misma dirección IP disponible, una de las direcciones IP disponibles de la subred se reserva para esta máquina virtual.<br/><br/> También puede especificar una dirección IP fija y una subred en **Elementos replicados** > **Configuración** > **Proceso y red** > **Interfaces de red**.
Dirección IP dinámica | Compatible | Si la NIC de origen tiene una dirección IP dinámica, la NIC de la máquina virtual de conmutación por error será también dinámica de forma predeterminada.<br/><br/> Puede modificar esta a una dirección IP fija si es necesario.
Varias direcciones IP | No compatible | Cuando se conmuta a una máquina virtual con una NIC con varias direcciones IP, se mantiene solo la dirección IP principal de la NIC en la región de origen. Para asignar varias direcciones IP, puede agregar las máquinas virtuales a un [plan de recuperación](recovery-plan-overview.md) y adjuntará un script para asignar direcciones IP adicionales al plan o se puede realizar el cambio manualmente o con una secuencia de comandos después de la conmutación por error. 
Traffic Manager     | Compatible | Puede configurar previamente Traffic Manager de forma que el tráfico se dirija regularmente al punto de conexión de la región de origen y al punto de conexión de la región de destino en caso de conmutación por error.
Azure DNS | Compatible |
DNS personalizado  | Compatible |
Proxy no autenticado | Compatible | [Más]. (site-recovery-azure-to-azure-networking-guidance.md)   
Proxy autenticado | No compatible | Si la máquina virtual usa un proxy autenticado para la conectividad saliente, no se puede replicar mediante Azure Site Recovery.    
Conexión de sitio a sitio VPN en el entorno local<br/><br/>(con o sin ExpressRoute)| Compatible | Garantiza que los Udr y NSG estén configurados de tal manera que no se enruta el tráfico de Site Recovery en el entorno local. [Más información](site-recovery-azure-to-azure-networking-guidance.md)    
Conexión de red virtual a red virtual | Compatible | [Más información](site-recovery-azure-to-azure-networking-guidance.md)  
Puntos de conexión de servicio de red virtual | Compatible | Si está restringiendo el acceso de red virtual a las cuentas de almacenamiento, asegúrese de que los servicios de Microsoft de confianza pueden acceder a la cuenta de almacenamiento.
Redes aceleradas | Compatible | Se deben habilitar las redes aceleradas en una máquina virtual de origen. [Más información](azure-vm-disaster-recovery-with-accelerated-networking.md).



## <a name="next-steps"></a>Pasos siguientes
- Lectura [instrucciones sobre redes](site-recovery-azure-to-azure-networking-guidance.md) para la replicación de máquinas virtuales de Azure.
- Implemente la recuperación ante desastres mediante [la replicación de máquinas virtuales de Azure](site-recovery-azure-to-azure.md).
