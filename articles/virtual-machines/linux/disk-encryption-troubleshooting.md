---
title: Solución de problemas de máquinas virtuales con Azure Disk Encryption para Linux
description: En este artículo se ofrecen sugerencias para solucionar problemas de Microsoft Azure Disk Encryption para máquinas virtuales Linux.
author: msmbaldwin
ms.service: virtual-machines-linux
ms.subservice: security
ms.topic: troubleshooting
ms.author: mbaldwin
ms.date: 08/06/2019
ms.custom: seodec18
ms.openlocfilehash: 5ca6431531f8cebf1205aa555c5545f4dc44abd3
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87372219"
---
# <a name="azure-disk-encryption-for-linux-vms-troubleshooting-guide"></a>Guía de solución de problemas de Azure Disk Encryption para máquinas virtuales Linux

Esta guía está destinada a profesionales de tecnologías de la información (TI), analistas de seguridad de la información y administradores de la nube cuyas organizaciones utilizan Azure Disk Encryption. En este artículo se ofrece ayuda para solucionar problemas relacionados con el cifrado de disco.

Antes de realizar cualquiera de los pasos siguientes, asegúrese de que las máquinas virtuales que intenta cifrar tienen [tamaños y sistemas operativos admitidos](disk-encryption-overview.md#supported-vms-and-operating-systems) y que ha cumplido todos los requisitos previos:

- [Requisitos adicionales de las máquinas virtuales](disk-encryption-overview.md#supported-vms-and-operating-systems)
- [Requisitos de red](disk-encryption-overview.md#networking-requirements)
- [Requisitos de almacenamiento de la clave de cifrado](disk-encryption-overview.md#encryption-key-storage-requirements)

 

## <a name="troubleshooting-linux-os-disk-encryption"></a>Solución de problemas del cifrado de discos con el SO Linux

El cifrado de discos con el sistema operativo Linux debe desmontar la unidad del sistema operativo antes de pasar por el proceso de cifrado de disco completo. Si no puede desmontar la unidad, es probable que aparezca el mensaje de error "no se pudo desmontar después de...".

Este error puede aparecer cuando se intenta realizar el cifrado del disco del sistema operativo en una máquina virtual con un entorno que se haya modificado desde su imagen de la galería admitida. Las desviaciones de la imagen admitida pueden interferir con la capacidad de la extensión para desmontar la unidad del sistema operativo. Algunos ejemplos de desviaciones pueden incluir los siguientes elementos:
- Imágenes personalizadas que ya no coinciden con un sistema de archivos o esquema de particiones compatibles.
- Las aplicaciones grandes, como SAP, MongoDB, Apache Cassandra y Docker, no se admiten cuando están instaladas y se ejecutan en el sistema operativo antes de que se realice el cifrado. Azure Disk Encryption no puede apagar con seguridad estos procesos tal como se requiere en la preparación de la unidad del sistema operativo para el cifrado de disco. Si hay procesos que siguen activos y que contienen identificadores de archivos abiertos en la unidad del sistema operativo, dicha unidad no se podrá desmontar y se producirá un error al cifrarla. 
- Se ejecutan scripts personalizados casi al mismo tiempo que se deshabilita el cifrado, o si se hace algún otro cambio en la máquina virtual durante el proceso de cifrado. Este conflicto puede ocurrir cuando una plantilla de Azure Resource Manager define varias extensiones para ejecutarse al mismo tiempo o cuando una extensión de script personalizada u otra acción se ejecutan simultáneamente que el cifrado del disco. La serialización y el aislamiento de estos pasos podría resolver el problema.
- Security Enhanced Linux (SELinux) no se ha deshabilitado antes de habilitar el cifrado, por lo que se ha producido un error en el paso de desmontaje. SELinux puede habilitarse de nuevo después de completarse el cifrado.
- El disco del sistema operativo utiliza un esquema de administrador de volúmenes lógicos (LVM). Aunque hay compatibilidad limitada con los discos de datos LVM, no se admite el disco del sistema operativo LVM.
- No se cumplen los requisitos mínimos de memoria (se recomiendan 7 GB para el cifrado del disco de sistema operativo).
- Las unidades de datos se han montado recursivamente en el directorio /mnt/ o entre sí (por ejemplo, /mnt/datos1, /mnt/datos2, /datos3 + /datos3/datos4).

## <a name="update-the-default-kernel-for-ubuntu-1404-lts"></a>Actualización del kernel predeterminado para Ubuntu 14.04 LTS

La imagen de Ubuntu 14.04 LTS se suministra con una versión de kernel predeterminada de 4.4. Esta versión de kernel tiene un problema conocido por el cual el cancelador de procesos por memoria insuficiente (OOM Killer) finaliza incorrectamente el comando dd durante el proceso de cifrado del sistema operativo. Este error se ha corregido en el kernel Linux más reciente ajustado para Azure. Para evitar este error, antes de habilitar el cifrado en la imagen, actualice al [kernel optimizado para Azure 4.15](https://packages.ubuntu.com/trusty/linux-azure) o posterior con los siguientes comandos:

```
sudo apt-get update
sudo apt-get install linux-azure
sudo reboot
```

Una vez la VM se haya reiniciado en el nuevo kernel, la nueva versión del kernel se puede confirmar con:

```
uname -a
```

## <a name="update-the-azure-virtual-machine-agent-and-extension-versions"></a>Actualización del agente de la máquina virtual de Azure y de las versiones de las extensiones

Las operaciones de Azure Disk Encryption pueden producir un error en las imágenes de máquina virtual que utilizan versiones no compatibles del agente de máquina virtual de Azure. Las imágenes de Linux con versiones de agente anteriores a 2.2.38 deben actualizarse antes de habilitar el cifrado. Para obtener más información, consulte [Actualización del agente Linux de Azure en una máquina virtual](../extensions/update-linux-agent.md) y [Soporte de versión mínima para los agentes de la máquina virtual en Azure](https://support.microsoft.com/en-us/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support).

También es necesaria la versión correcta de la extensión del agente invitado Microsoft.Azure.Security.AzureDiskEncryption o Microsoft.Azure.Security.AzureDiskEncryptionForLinux. La plataforma mantiene y actualiza automáticamente las versiones de extensión cuando se cumplen los requisitos previos del agente de máquina virtual de Azure y se usa una versión compatible del agente de máquina virtual.

La extensión Microsoft.OSTCExtensions.AzureDiskEncryptionForLinux está en desuso y ya no se admite.  

## <a name="unable-to-encrypt-linux-disks"></a>No se pueden cifrar los discos de Linux

En algunos casos, parece que el cifrado del disco de Linux se atasca en el mensaje que indica que se inició el cifrado de disco del sistema operativo, y SSH se deshabilita. El proceso de cifrado puede tardar entre 3 y 16 horas en completarse en una imagen de la galería de existencias. Si se agregan discos de datos de varios terabytes, el proceso puede tardar días.

La secuencia de cifrado del disco del sistema operativo Linux desmonta la unidad del sistema operativo temporalmente. A continuación, realiza el cifrado de bloque a bloque de todo el disco del sistema operativo, antes de que vuelva a montarse en su estado cifrado. El cifrado del disco de Linux no permite el uso simultáneo de la máquina virtual mientras el cifrado está en curso. Las características de rendimiento de la máquina virtual pueden suponer una diferencia significativa en el tiempo necesario para el cifrado completo. Estas características incluyen el tamaño del disco y si la cuenta de almacenamiento es almacenamiento premium (SSD) o estándar.

Mientras se cifra la unidad del sistema operativo, la máquina virtual entra en estado de mantenimiento y deshabilita SSH para evitar cualquier interrupción en el proceso en curso.  Para comprobar el estado de cifrado, use el comando [Get-AzVmDiskEncryptionStatus](/powershell/module/az.compute/get-azvmdiskencryptionstatus) de Azure PowerShell y compruebe el campo **ProgressMessage**. **ProgressMessage** notificará una serie de estados a medida que se cifran los discos de datos y del sistema operativo:

```azurepowershell
PS > Get-AzVMDiskEncryptionStatus -ResourceGroupName "MyResourceGroup" -VMName "myVM"

OsVolumeEncrypted          : EncryptionInProgress
DataVolumesEncrypted       : EncryptionInProgress
OsVolumeEncryptionSettings :
ProgressMessage            : Transitioning

PS > Get-AzVMDiskEncryptionStatus -ResourceGroupName "MyResourceGroup" -VMName "myVM"

OsVolumeEncrypted          : EncryptionInProgress
DataVolumesEncrypted       : EncryptionInProgress
OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
ProgressMessage            : Encryption succeeded for data volumes

PS > Get-AzVMDiskEncryptionStatus -ResourceGroupName "MyResourceGroup" -VMName "myVM"

OsVolumeEncrypted          : EncryptionInProgress
DataVolumesEncrypted       : EncryptionInProgress
OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
ProgressMessage            : Provisioning succeeded

PS > Get-AzVMDiskEncryptionStatus -ResourceGroupName "MyResourceGroup" -VMName "myVM"

OsVolumeEncrypted          : EncryptionInProgress
DataVolumesEncrypted       : EncryptionInProgress
OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
ProgressMessage            : OS disk encryption started
```

**ProgressMessage** permanecerá en **Cifrado del disco del sistema operativo iniciado** durante la mayor parte del proceso de cifrado.  Cuando el cifrado se complete y se realice correctamente, **ProgressMessage** devolverá:

```azurepowershell
PS > Get-AzVMDiskEncryptionStatus -ResourceGroupName "MyResourceGroup" -VMName "myVM"

OsVolumeEncrypted          : Encrypted
DataVolumesEncrypted       : NotMounted
OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
ProgressMessage            : Encryption succeeded for all volumes
```

Cuando este mensaje esté disponible, es de esperar que la unidad de sistema operativo cifrada y la máquina virtual estén listas para usarse de nuevo.

Si la información de arranque, el mensaje de progreso o un error informa de que se ha producido un error en el cifrado del sistema operativo en medio de este proceso, restaure la máquina virtual a la instantánea o copia de seguridad realizada inmediatamente antes del cifrado. Un ejemplo de mensaje es el error "no se pudo desmontar" que se describe en esta guía.

Antes de volver a intentar el cifrado, vuelva a evaluar las características de la máquina virtual y asegúrese de que se cumplen todos los requisitos previos.

## <a name="troubleshooting-azure-disk-encryption-behind-a-firewall"></a>Solución de problemas de Azure Disk Encryption detrás de un firewall

Vea [Cifrado de discos en una red aislada](disk-encryption-isolated-network.md).

## <a name="troubleshooting-encryption-status"></a>Solución de problemas de estado del cifrado 

Es posible que el portal muestre que un disco está cifrado incluso después de haberse descifrado en la máquina virtual.  Esto puede ocurrir cuando se usan comandos de nivel inferior para descifrar directamente el disco de la máquina virtual, en lugar de usar los comandos de administración de Azure Disk Encryption de nivel superior.  Los comandos del nivel superior no solo descifran el disco de la máquina virtual, sino fuera de la máquina virtual también actualizan configuración importante de cifrado de nivel de plataforma y configuración de extensión asociada con la máquina virtual.  Si estos no se mantienen en la alineación, la plataforma no podrá informar del estado de cifrado ni aprovisionar la máquina virtual correctamente.

Para deshabilitar Azure Disk Encryption con PowerShell, use [Disable-AzVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption) seguido de [Remove-AzVMDiskEncryptionExtension](/powershell/module/az.compute/remove-azvmdiskencryptionextension). Si se ejecuta Remove-AzVMDiskEncryptionExtension antes de deshabilitar el cifrado, se producirá un error.

Para deshabilitar Azure Disk Encryption con la CLI, use [az vm encryption disable](/cli/azure/vm/encryption).

## <a name="next-steps"></a>Pasos siguientes

En este documento, aprendió más acerca de algunos problemas comunes de Azure Disk Encryption y cómo solucionarlos. Para más información acerca de este servicio y su funcionalidad, consulte los artículos siguientes:

- [Aplicación de cifrado de discos en Azure Security Center](../../security-center/security-center-virtual-machine-protection.md)
- [Cifrado de datos en reposo de Azure](../../security/fundamentals/encryption-atrest.md)
