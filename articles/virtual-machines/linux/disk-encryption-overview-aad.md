---
title: Requisitos previos de Azure Disk Encryption con una aplicación de Azure AD (versión anterior)
description: En este artículo se proporcionan los requisitos previos para usar Microsoft Azure Disk Encryption para máquinas virtuales IaaS.
author: msmbaldwin
ms.service: virtual-machines-linux
ms.subservice: security
ms.topic: conceptual
ms.author: mbaldwin
ms.date: 03/15/2019
ms.custom: seodec18
ms.openlocfilehash: c85d362f7295e8edef1b4070a779c6aa99c3991f
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87372644"
---
# <a name="azure-disk-encryption-with-azure-active-directory-ad-previous-release"></a>Azure Disk Encryption con Azure Active Directory (AD) (versión anterior)

La nueva versión de Azure Disk Encryption elimina el requisito de tener que proporcionar un parámetro de aplicación de Azure Active Directory (Azure AD) para habilitar el cifrado de disco de la máquina virtual. Con la nueva versión, ya no es necesario proporcionar credenciales de Azure AD durante el paso de habilitar el cifrado. Todas las máquinas virtuales nuevas se deben cifrar sin los parámetros de aplicación de Azure AD con la nueva versión. Para ver las instrucciones sobre cómo habilitar el cifrado de disco de máquina virtual con la nueva versión, consulte [Azure Disk Encryption para máquinas virtuales Linux](disk-encryption-overview.md). Las máquinas virtuales que ya se han cifrado con parámetros de la aplicación de Azure AD siguen siendo compatibles y deben continuar manteniéndose con la sintaxis de AAD.

Este artículo sirve de complemento al artículo [Azure Disk Encryption para máquinas virtuales Linux](disk-encryption-overview.md) con requisitos adicionales y requisitos previos para Azure Disk Encryption con Azure AD (versión anterior).

La información de estas secciones sigue siendo la misma:

- [Máquinas virtuales y sistemas operativos compatibles](disk-encryption-overview.md#supported-vms-and-operating-systems)
- [Requisitos adicionales de la máquina virtual](disk-encryption-overview.md#additional-vm-requirements)


## <a name="networking-and-group-policy"></a>Redes y directiva de grupo

Para habilitar la característica Azure Disk Encryption con la sintaxis de parámetro de ADD anterior, las máquinas virtuales de infraestructura como servicio (IaaS) deben cumplir los siguientes requisitos de configuración de puntos de conexión de red: 
  - Para que un token se conecte al almacén de claves, la máquina virtual de IaaS debe poder conectarse a un punto de conexión de Azure AD, \[login.microsoftonline.com\].
  - Para escribir las claves de cifrado en el almacén de claves, la máquina virtual IaaS debe poder conectarse al punto de conexión del almacén de claves.
  - La máquina virtual IaaS debe poder conectarse al punto de conexión de Azure Storage que hospeda el repositorio de extensiones de Azure y la cuenta de Azure Storage que hospeda los archivos VHD.
  -  Si su directiva de seguridad limita el acceso desde máquinas virtuales de Azure a Internet, puede resolver el URI anterior y configurar una regla concreta para permitir la conectividad de salida para las direcciones IP. Para más información, consulte [Azure Key Vault detrás de un firewall](../../key-vault/general/access-behind-firewall.md).
  - En Windows, si se deshabilitó explícitamente TLS 1.0 y la versión de .NET no se ha actualizado a la 4.6 o posterior, el siguiente cambio en el registro habilitará Azure Disk Encryption para seleccionar la versión más reciente de TLS:

  ```config-registry
  [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319]
  "SystemDefaultTlsVersions"=dword:00000001
  "SchUseStrongCrypto"=dword:00000001
    
  [HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319]
  "SystemDefaultTlsVersions"=dword:00000001
  "SchUseStrongCrypto"=dword:00000001` 
  ```

### <a name="group-policy"></a>Directiva de grupo
 - La solución Azure Disk Encryption usa el protector de claves externas de BitLocker para máquinas virtuales IaaS con Windows. Para las máquinas virtuales unidas en un dominio, no cree ninguna directiva de grupo que exija protectores de TPM. Para obtener información acerca de la directiva de grupo para la opción **Permitir BitLocker sin un TPM compatible**, consulte la [Referencia de la directiva de grupo de BitLocker](/windows/security/information-protection/bitlocker/bitlocker-group-policy-settings#bkmk-unlockpol1).

- La directiva de BitLocker en máquinas virtuales unidas a un dominio con directivas de grupo personalizadas debe incluir la siguiente configuración: [Configuración de almacenamiento de usuario de información de recuperación de BitLocker -> Permitir clave de recuperación de 256 bits](/windows/security/information-protection/bitlocker/bitlocker-group-policy-settings). Azure Disk Encryption presenta un error cuando la configuración de la directiva de grupo personalizada para BitLocker es incompatible. En máquinas que no tienen la configuración de directiva correcta, aplique la nueva directiva, fuerce a esta a actualizarse (gpupdate.exe /force) y luego reinicie. 

## <a name="encryption-key-storage-requirements"></a>Requisitos de almacenamiento de la clave de cifrado 

Azure Disk Encryption requiere Azure Key Vault para controlar y administrar las claves y los secretos de cifrado de discos. El almacén de claves y las máquinas virtuales deben residir en la misma región y suscripción de Azure.

Para más información, consulte [Creación y configuración de un almacén de claves para Azure Disk Encryption con Azure AD (versión anterior)](disk-encryption-key-vault-aad.md).
 
## <a name="next-steps"></a>Pasos siguientes

- [Creación y configuración de un almacén de claves para Azure Disk Encryption con Azure AD (versión anterior)](disk-encryption-key-vault-aad.md)
- [Habilitación de Azure Disk Encryption con Azure AD en máquinas virtuales Linux (versión anterior)](disk-encryption-linux-aad.md)
- [Script de la CLI de requisitos previos de Azure Disk Encryption](https://github.com/ejarvi/ade-cli-getting-started)
- [Script de PowerShell de requisitos previos de Azure Disk Encryption](https://github.com/Azure/azure-powershell/tree/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts)
