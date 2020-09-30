---
title: Matriz de compatibilidad con la migración de máquinas virtuales de Azure a otra región con Azure Resource Mover
description: Revise la compatibilidad con la migración de máquinas virtuales de Azure entre regiones con Azure Resource Mover.
author: rayne-wiselman
manager: evansma
ms.service: resource-move
ms.topic: how-to
ms.date: 09/07/2020
ms.author: raynew
ms.openlocfilehash: fa71cd502f730844e4f4398d41d06ada56fc2413
ms.sourcegitcommit: 80b9c8ef63cc75b226db5513ad81368b8ab28a28
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2020
ms.locfileid: "90602292"
---
# <a name="support-for-moving-azure-vms-between-azure-regions"></a>Compatibilidad con la migración de máquinas virtuales de Azure entre regiones de Azure

En este artículo se resumen la compatibilidad y los requisitos previos a la hora de migrar máquinas virtuales y recursos de red relacionados entre regiones de Azure mediante Resource Mover.

> [!IMPORTANT]
> Azure Resource Mover se encuentra actualmente en versión preliminar.


## <a name="windows-vm-support"></a>Compatibilidad con máquinas virtuales Windows

Resource Mover admite máquinas virtuales de Azure con estos sistemas operativos Windows.

**Sistema operativo** | **Detalles**
--- | ---
Windows Server 2019 | Se admite para Server Core y Server con Experiencia de escritorio.
Windows Server 2016  | Se admite para Server Core y Server con Experiencia de escritorio
Windows Server 2012 R2 | Compatible.
Windows Server 2012 | Compatible.
Windows Server 2008 R2 con SP1/SP2 | Compatible.<br/><br/> En máquinas con Windows Server 2008 R2 con SP1/SP2 debe instalar una [actualización de la pila de servicio (SSU)](https://support.microsoft.com/help/4490628) de Windows y la [actualización de SHA-2](https://support.microsoft.com/help/4474419).  SHA-1 no se admite desde septiembre de 2019 y, si la firma de código SHA-2 no está habilitada, la extensión del agente no se instalará ni actualizará según lo previsto. Más información sobre los [requisitos y la actualización de SHA-2](https://aka.ms/SHA-2KB).
Windows 10 (x64) | Compatible.
Windows 8.1 (x64) | Compatible.
Windows 8 (x64) | Compatible.
Windows 7 (x64) con SP1 en adelante | Instale una [actualización de la pila de servicio (SSU)](https://support.microsoft.com/help/4490628) de Windows y la [actualización de SHA-2](https://support.microsoft.com/help/4474419) en máquinas con Windows 7 con SP1.  SHA-1 no se admite desde septiembre de 2019 y, si la firma de código SHA-2 no está habilitada, el paso de preparación no se realiza correctamente. Más información sobre los [requisitos y la actualización de SHA-2](https://aka.ms/SHA-2KB).


## <a name="linux-vm-support"></a>Compatibilidad con máquinas virtuales Linux

Resource Mover admite máquinas virtuales de Azure con estos sistemas operativos Linux.

**Sistema operativo** | **Detalles**
--- | ---
Red Hat Enterprise Linux | 6.7, 6.8, 6.9, 6.10, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, [7.7](https://support.microsoft.com/help/4528026/update-rollup-41-for-azure-site-recovery), [8.0](https://support.microsoft.com/help/4531426/update-rollup-42-for-azure-site-recovery), 8.1
CentOS | 6.5, 6.6, 6.7, 6.8, 6.9, 6.10, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, 7.7, 8.0, 8.1
Servidor Ubuntu 14.04 LTS | [Versiones de kernel admitidas](#supported-ubuntu-kernel-versions)
Servidor Ubuntu 16.04 LTS | [Versión de kernel admitida](#supported-ubuntu-kernel-versions)<br/><br/> Los servidores Ubuntu con inicio de sesión y autenticación mediante contraseña y el paquete cloud-init para configurar máquinas virtuales en la nube pueden tener el inicio de sesión mediante contraseña deshabilitado tras una conmutación por error (en función de la configuración de cloud-init). Para volver a habilitar el inicio de sesión mediante contraseña en la máquina virtual, restablezca la contraseña desde el menú Soporte > Solución de problemas > Configuración (de la máquina virtual conmutada por error en Azure Portal).
Servidor Ubuntu 18.04 LTS | [Versión de kernel admitida](#supported-ubuntu-kernel-versions).
Debian 7 | [Versiones de kernel admitidas](#supported-debian-kernel-versions).
Debian 8 | [Versiones de kernel admitidas](#supported-debian-kernel-versions).
SUSE Linux Enterprise Server 12 | SP1, SP2, SP3, SP4. [Versiones de kernel admitidas](#supported-suse-linux-enterprise-server-12-kernel-versions)
SUSE Linux Enterprise Server 15 | 15 y 15 SP1. [(Versiones de kernel admitidas)](#supported-suse-linux-enterprise-server-15-kernel-versions)
SUSE Linux Enterprise Server 11 | SP3
SUSE Linux Enterprise Server 11 | SP4
Oracle Linux | 6.4, 6.5, 6.6, 6.7, 6.8, 6.9, 6.10, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, [7.7](https://support.microsoft.com/help/4531426/update-rollup-42-for-azure-site-recovery) <br/><br/> Ejecución del kernel compatible de Red Hat o Unbreakable Enterprise Kernel Release 3, 4 y 5 (UEK3, UEK4, UEK5)


### <a name="supported-ubuntu-kernel-versions"></a>Versiones de kernel de Ubuntu admitidas

**Versión** | **Versión de kernel** 
--- | --- 
14.04 LTS |  3.13.0-24-generic a 3.13.0-170-generic,<br/>3.16.0-25-generic a 3.16.0-77-generic,<br/>3.19.0-18-generic a 3.19.0-80-generic,<br/>4.2.0-18-generic a 4.2.0-42-generic,<br/>4.4.0-21-generic a 4.4.0-148-generic,<br/>4.15.0-1023-azure a 4.15.0-1045-azure 
16.04 LTS |  4.4.0-21-generic a 4.4.0-171-generic,<br/>4.8.0-34-generic a 4.8.0-58-generic,<br/>4.10.0-14-generic a 4.10.0-42-generic,<br/>4.11.0-13-generic to 4.11.0-14-generic,<br/>4.13.0-16-generic a 4.13.0-45-generic,<br/>4.15.0-13-generic a 4.15.0-74-generic<br/>4.11.0-1009-azure to 4.11.0-1016-azure,<br/>4.13.0-1005-azure a 4.13.0-1018-azure <br/>4.15.0-1012-azure a 4.15.0-1066-azure
18.04 LTS | 4.15.0-20-generic a 4.15.0-74-generic </br> De 4.18.0-13-generic a 4.18.0-25-generic </br> 5.0.0-15-generic a 5.0.0-37-generic </br> 5.3.0-19-generic a 5.3.0-24-generic </br> De 4.15.0-1009-azure a 4.15.0-1037-azure </br> De 4.18.0-1006-azure a 4.18.0-1025-azure </br> 5.0.0-1012-azure a 5.0.0-1028-azure </br> 5.3.0-1007-azure a 5.3.0-1009-azure


### <a name="supported-debian-kernel-versions"></a>Versiones de kernel de Debian admitidas 

**Versión** |  **Versión de kernel** 
--- |  --- 
Debian 7 |  3.2.0-4-amd64 a 3.2.0-6-amd64, 3.16.0-0.bpo.4-amd64 
Debian 8 |  3.16.0-4-amd64 a 3.16.0-10-amd64, 4.9.0-0.bpo.4-amd64 a 4.9.0-0.bpo.11-amd64 
Debian 8 |  3.16.0-4-amd64 a 3.16.0-10-amd64, 4.9.0-0.bpo.4-amd64 a 4.9.0-0.bpo.9-amd64

### <a name="supported-suse-linux-enterprise-server-12-kernel-versions"></a>Versiones de kernel de SUSE Linux Enterprise Server 12 admitidas 

**Versión** | **Versión de kernel** 
--- |  --- 
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3, SP4) |  Se admiten todos los [kernels de SUSE 12 SP1, SP2, SP3 y SP4](https://www.suse.com/support/kb/doc/?id=000019587).</br></br> 4.4.138-4.7-azure a 4.4.180-4.31-azure,</br>4.12.14-6.3-azure a 4.12.14-6.34-azure  


### <a name="supported-suse-linux-enterprise-server-15-kernel-versions"></a>Versiones de kernel de SUSE Linux Enterprise Server 15 admitidas

**Versión** | **Versión de kernel** |
--- |  --- |
SUSE Linux Enterprise Server 15 y 15 SP1 |  Se admiten todos los kernels stock de SUSE 15 y 15.</br></br> 4.12.14-5.5-azure a 4.12.14-8.22-azure |

## <a name="supported-linux-file-systemguest-storage"></a>Sistema de archivos y almacenamiento de invitados de Linux admitidos

* Sistemas de archivos: ext3, ext4, XFS y BTRFS
* Administrador de volúmenes: LVM2
* Software de múltiples rutas: Mapeador de dispositivo


## <a name="supported-vm-compute-settings"></a>Configuración de proceso de máquina virtual admitida

**Configuración** | **Soporte técnico** | **Detalles**
--- | --- | ---
Size | Cualquier tamaño de máquina virtual de Azure con al menos dos núcleos de CPU y 1 GB de RAM | Compruebe los [tamaños de máquina virtual de Azure](../virtual-machines/sizes-general.md).
Conjuntos de disponibilidad | No se admite actualmente. | Si agrega una máquina virtual de Azure con un conjunto de disponibilidad a la colección movida con las opciones predeterminadas, se produce un error en el proceso de preparación. Puede optar por migrar la máquina virtual a una zona de disponibilidad o moverla como una máquina virtual de instancia única. Puede modificar esta configuración en la página Editar propiedades de destino.
Zonas de disponibilidad | Compatible | Compatible, según la compatibilidad de la región de destino.
Imágenes de la galería de Azure (publicadas por Microsoft) | Compatible | Se admiten si la máquina virtual se ejecuta en un sistema operativo compatible.
Imágenes de la galería de Azure (publicadas por terceros)  | Compatible | Se admiten si la máquina virtual se ejecuta en un sistema operativo compatible.
Imágenes personalizadas (publicadas por terceros)| Compatible | Se admiten si la máquina virtual se ejecuta en un sistema operativo compatible.
Máquinas virtuales con Site Recovery | No compatible | Mueva los recursos de las máquinas virtuales entre regiones mediante Site Recovery en el back-end. Si ya está usando Site Recovery, deshabilite la replicación y luego inicie el proceso de preparación.
Directivas RBAC | No compatible | Las directivas de control de acceso basado en rol (RBAC) de las máquinas virtuales no se copian en la máquina virtual de la región de destino.
Extensiones | No compatible | Las extensiones no se copian en la máquina virtual de la región de destino. Instálelas manualmente una vez completada la migración.


## <a name="supported-vm-storage-settings"></a>Configuración de almacenamiento de máquina virtual admitida

Esta tabla resume la compatibilidad con el disco del sistema operativo, el disco de datos y el disco temporal de Azure VM. Es importante respetar los límites y destinos de los discos de máquinas virtuales [Linux](../virtual-machines/linux/disk-scalability-targets.md) y [Windows](../virtual-machines/windows/disk-scalability-targets.md) para evitar cualquier problema de rendimiento.

> [!NOTE]
> El tamaño de la máquina virtual de destino debe ser igual o mayor que el de la máquina virtual de origen. Los parámetros usados para la validación son: Número de discos de datos, Número de NIC, CPU disponibles, Memoria en GB. Si no es así, se emite un error.


**Componente** | **Soporte técnico** | **Detalles**
--- | --- | ---
Tamaño máximo del disco de sistema operativo | 2048 GB | [Más información](../virtual-machines/windows/managed-disks-overview.md) sobre discos de máquina virtual.
Disco temporal | No compatible | El disco temporal siempre se excluye del proceso de preparación.<br/><br/> No almacene los datos persistentes en el disco temporal. [Más información](../virtual-machines/windows/managed-disks-overview.md#temporary-disk).
Tamaño máximo del disco de datos | 8192 GB para discos administrados
Tamaño mínimo del disco de datos |  2 GB en discos administrados |
Número máximo de discos de datos | Hasta 64, según la compatibilidad con un tamaño específico de máquina virtual de Azure | [Más información](../virtual-machines/windows/sizes.md) sobre tamaños de máquina virtual.
Tasa de cambio de disco de datos | Máximo de 10 MBps por disco para el almacenamiento Premium. Máximo de 2 MBps por disco para el almacenamiento Estándar. | Si la tasa media de cambio de datos en el disco sobrepasa el máximo continuamente, la preparación no se recupera.<br/><br/>  Pero si se supera el máximo esporádicamente, la preparación sí se recupera, aunque se podrían observar puntos de recuperación ligeramente retrasados.
Disco de datos (cuenta de almacenamiento Estándar) | No compatible. | Cambie el tipo de almacenamiento a disco administrado e intente migrar la máquina virtual.
Disco de datos (cuenta de almacenamiento Premium) | No compatible | Cambie el tipo de almacenamiento a disco administrado e intente migrar la máquina virtual.
Disco administrado (Estándar) | Compatible  |
Disco administrado (Premium) | Compatible |
SSD estándar | Compatible |
Generación 2 (Arranque UEFI) | Compatible
Cuenta de almacenamiento de diagnóstico de arranque | No compatible | Vuelva a habilitarla después de migrar la máquina virtual a la región de destino.

### <a name="limits-and-data-change-rates"></a>Límites y tasas de cambio de datos

En la tabla siguiente se resumen los límites que se han basado en nuestras pruebas. No cubren todas las combinaciones posibles de E/S de una aplicación. Los resultados reales varían en función de la combinación de E/S de la aplicación. Hay dos límites que se deben tener en cuenta: renovación de datos por disco y por máquina virtual.

**Destino de almacenamiento** | **Promedio de E/S de disco de origen** |**Actividad de datos media de disco de origen** | **Actividad de datos de disco de origen total por día**
---|---|---|---
Standard Storage | 8 KB    | 2 MB/s | 168 GB por disco
Disco Premium P10 o P15 | 8 KB    | 2 MB/s | 168 GB por disco
Disco Premium P10 o P15 | 16 KB | 4 MB/s |    336 GB por disco
Disco Premium P10 o P15 | 32 KB, o más | 8 MB/s | 672 GB por disco
Disco Premium P20, P30, P40 o P50 | 8 KB    | 5 MB/s | 421 GB por disco
Disco Premium P20, P30, P40 o P50 | 16 KB, o más |20 MB/s | 1684 GB por disco

## <a name="supported-vm-networking-settings"></a>Configuración de redes de máquina virtual admitida

**Configuración** | **Soporte técnico** | **Detalles**
--- | --- | ---
NIC | Compatible | Especifique un recurso existente en la región de destino o cree uno nuevo durante el proceso de preparación. 
Equilibrador de carga interno | Compatible | Especifique un recurso existente en la región de destino o cree uno nuevo durante el proceso de preparación.  
Equilibrador de carga público | No se admite actualmente. | Especifique un recurso existente en la región de destino o cree uno nuevo durante el proceso de preparación.  
Dirección IP pública | Compatible | Especifique un recurso existente en la región de destino o cree uno nuevo durante el proceso de preparación.  
Grupo de seguridad de red | Compatible | Especifique un recurso existente en la región de destino o cree uno nuevo durante el proceso de preparación.  
Dirección IP (estática) reservada | Compatible | Actualmente no se puede configurar. El valor predeterminado es el valor de origen. <br/><br/> Si la NIC de la máquina virtual de origen tiene una dirección IP estática y la subred de destino tiene la misma dirección IP disponible, se asigna a la máquina virtual de destino.<br/><br/> Si la subred de destino no tiene disponible la misma dirección IP, se produce un error al iniciar la migración de la máquina virtual.
Dirección IP dinámica | Compatible | Actualmente no se puede configurar. El valor predeterminado es el valor de origen.<br/><br/> Si la NIC de origen tiene una dirección IP dinámica, la NIC de la máquina virtual de destino también es dinámica de forma predeterminada.
Configuraciones IP | Compatible | Actualmente no se puede configurar. El valor predeterminado es el valor de origen.

## <a name="outbound-access-requirements"></a>Requisitos de acceso de salida

Las máquinas virtuales de Azure que quiere migrar necesitan acceso de salida.


### <a name="url-access"></a>acceso URL

 Si usa un proxy de firewall basado en direcciones URL para controlar la conectividad de salida, debe permitir el acceso a ellas:

**Nombre** | **Nube pública de Azure** | **Detalles** 
--- | --- | --- 
Storage | `*.blob.core.windows.net`  | Permite que los datos se puedan escribir desde la máquina virtual a la cuenta de almacenamiento de caché en la región de origen. 
Azure Active Directory | `login.microsoftonline.com`  | Proporciona autorización y autenticación de las direcciones URL del servicio Site Recovery. 
Replicación | `*.hypervrecoverymanager.windowsazure.com` | Permite que la máquina virtual se comunique con el servicio Site Recovery. 
Azure Service Bus | `*.servicebus.windows.net` | Permite que la máquina virtual escriba los datos de diagnóstico y supervisión de Site Recovery. 

## <a name="nsg-rules"></a>Reglas de grupo de seguridad de red
Si usa reglas de grupo de seguridad de red (NSG) para controlar la conectividad de salida, cree estas reglas de [etiqueta de servicio](../virtual-network/service-tags-overview.md). Cada regla debe permitir el acceso de salida en HTTPS (443).
- Cree una regla de etiqueta de Storage para la región de origen.
- Cree una regla de etiqueta de *AzureSiteRecovery* para permitir el acceso al servicio Site Recovery en cualquier región. Esta etiqueta tiene dependencias en estas otras etiquetas, por lo que debe crear reglas para ellas:
    - *AzureActiveDirectory*
    - **EventHub*
    - *AzureKeyVault*
    - *GuestAndHybridManagement*
- Se recomienda probar las reglas en un entorno que no sea de producción. [Vea algunos ejemplos](../site-recovery/azure-to-azure-about-networking.md#outbound-connectivity-using-service-tags). 


## <a name="next-steps"></a>Pasos siguientes

Intente [trasladar una máquina virtual de Azure](tutorial-move-region-virtual-machines.md) a otra región con Resource Mover.
