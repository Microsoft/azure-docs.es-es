---
title: Guía de solución de problemas de Azure Disk Encryption
description: En este artículo se ofrecen sugerencias para solucionar problemas de Microsoft Azure Disk Encryption para máquinas virtuales Windows.
author: msmbaldwin
ms.service: virtual-machines-windows
ms.subservice: security
ms.topic: troubleshooting
ms.author: mbaldwin
ms.date: 08/06/2019
ms.custom: seodec18
ms.openlocfilehash: b3b83899ad21cf125105881a7ffb526f5c607c6d
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87322217"
---
# <a name="azure-disk-encryption-troubleshooting-guide"></a>Guía de solución de problemas de Azure Disk Encryption

Esta guía está destinada a profesionales de tecnologías de la información (TI), analistas de seguridad de la información y administradores de la nube cuyas organizaciones utilizan Azure Disk Encryption. En este artículo se ofrece ayuda para solucionar problemas relacionados con el cifrado de disco.

Antes de realizar cualquiera de los pasos siguientes, asegúrese de que las máquinas virtuales que intenta cifrar tienen [tamaños y sistemas operativos admitidos](disk-encryption-overview.md#supported-vms-and-operating-systems) y que ha cumplido todos los requisitos previos:

- [Requisitos de red](disk-encryption-overview.md#networking-requirements)
- [Requisitos de la directiva de grupo](disk-encryption-overview.md#group-policy-requirements)
- [Requisitos de almacenamiento de la clave de cifrado](disk-encryption-overview.md#encryption-key-storage-requirements)

## <a name="troubleshooting-azure-disk-encryption-behind-a-firewall"></a>Solución de problemas de Azure Disk Encryption detrás de un firewall

Cuando la conectividad está limitada por un firewall, la configuración del grupo de seguridad de red (NSG) o el requisito de proxy, podría perderse la capacidad de la extensión para realizar las tareas necesarias. Esto puede dar lugar a mensajes de estado similares a "El estado de extensión no está disponible en la máquina virtual". En los escenarios esperados, el cifrado no puede finalizar. En las secciones siguientes se enumeran algunos problemas comunes del firewall que podría investigar.

### <a name="network-security-groups"></a>Grupos de seguridad de red
Parte de la configuración del grupo de seguridad de red que se aplica debe permitir todavía que el punto de conexión cumpla con los [requisitos previos](disk-encryption-overview.md#networking-requirements) documentados de la configuración de red para el cifrado del disco.

### <a name="azure-key-vault-behind-a-firewall"></a>Azure Key Vault detrás de un firewall

Cuando se está habilitando el cifrado con [credenciales de Azure AD](disk-encryption-windows-aad.md#), la máquina virtual de destino debe permitir la conectividad a los puntos de conexión de Azure Active Directory y Key Vault. Los puntos de conexión de autenticación actuales de Azure Active Directory se mantienen en las secciones 56 y 59 de la documentación sobre los [intervalos de direcciones IP y direcciones URL de Office 365](/office365/enterprise/urls-and-ip-address-ranges). En la documentación sobre cómo [acceder a Azure Key Vault detrás de un firewall](../../key-vault/general/access-behind-firewall.md) se proporcionan instrucciones de Key Vault.

### <a name="azure-instance-metadata-service"></a>Servicio de metadatos de instancia de Azure 
La máquina virtual debe poder acceder al punto de conexión de [Azure Instance Metadata Service](../windows/instance-metadata-service.md) que utiliza una dirección IP no enrutable conocida (`169.254.169.254`) a la que solo se puede acceder desde la máquina virtual.  No se admiten las configuraciones de proxy que modifican el tráfico HTTP local hacia esta dirección (por ejemplo, agregando un encabezado X-Forwarded-For).

## <a name="troubleshooting-windows-server-2016-server-core"></a>Solución de problemas de Windows Server 2016 Server Core

En Windows Server 2016 Server Core, el componente bdehdcfg no está disponible de forma predeterminada. y Azure Disk Encryption necesita dicho componente. Se utiliza para dividir el volumen del sistema del volumen del sistema operativo, lo que se realiza una única vez en toda la duración de la máquina virtual. Estos archivos binarios no son necesarios durante las operaciones de cifrado posteriores.

Para solucionar este problema, copie los cuatro archivos siguientes de una máquina virtual del centro de datos de Windows Server 2016 en la misma ubicación de Server Core:

   ```
   \windows\system32\bdehdcfg.exe
   \windows\system32\bdehdcfglib.dll
   \windows\system32\en-US\bdehdcfglib.dll.mui
   \windows\system32\en-US\bdehdcfg.exe.mui
   ```

1. Escriba el comando siguiente:

   ```
   bdehdcfg.exe -target default
   ```

1. Este comando crea una partición de sistema de 550 MB. Reinicie el sistema.

1. Use DiskPart para comprobar los volúmenes y, a continuación, siga.  

Por ejemplo:

```
DISKPART> list vol

  Volume ###  Ltr  Label        Fs     Type        Size     Status     Info
  ----------  ---  -----------  -----  ----------  -------  ---------  --------
  Volume 0     C                NTFS   Partition    126 GB  Healthy    Boot
  Volume 1                      NTFS   Partition    550 MB  Healthy    System
  Volume 2     D   Temporary S  NTFS   Partition     13 GB  Healthy    Pagefile
```

## <a name="troubleshooting-encryption-status"></a>Solución de problemas de estado del cifrado 

Es posible que el portal muestre que un disco está cifrado incluso después de haberse descifrado en la máquina virtual.  Esto puede ocurrir cuando se usan comandos de nivel inferior para descifrar directamente el disco de la máquina virtual, en lugar de usar los comandos de administración de Azure Disk Encryption de nivel superior.  Los comandos del nivel superior no solo descifran el disco de la máquina virtual, sino fuera de la máquina virtual también actualizan configuración importante de cifrado de nivel de plataforma y configuración de extensión asociada con la máquina virtual.  Si estos no se mantienen en la alineación, la plataforma no podrá informar del estado de cifrado ni aprovisionar la máquina virtual correctamente.

Para deshabilitar Azure Disk Encryption con PowerShell, use [Disable-AzVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption) seguido de [Remove-AzVMDiskEncryptionExtension](/powershell/module/az.compute/remove-azvmdiskencryptionextension). Si se ejecuta Remove-AzVMDiskEncryptionExtension antes de deshabilitar el cifrado, se producirá un error.

Para deshabilitar Azure Disk Encryption con la CLI, use [az vm encryption disable](/cli/azure/vm/encryption). 

## 



## <a name="next-steps"></a>Pasos siguientes

En este documento, aprendió más acerca de algunos problemas comunes de Azure Disk Encryption y cómo solucionarlos. Para más información acerca de este servicio y su funcionalidad, consulte los artículos siguientes:

- [Aplicación de cifrado de discos en Azure Security Center](../../security-center/security-center-virtual-machine-protection.md)
- [Cifrado de datos en reposo de Azure](../../security/fundamentals/encryption-atrest.md)
