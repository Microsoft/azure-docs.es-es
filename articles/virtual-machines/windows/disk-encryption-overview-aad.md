---
title: Azure Disk Encryption con Azure AD (versión anterior)
description: En este artículo se proporcionan los requisitos previos para usar Microsoft Azure Disk Encryption para máquinas virtuales IaaS.
author: msmbaldwin
ms.service: virtual-machines-windows
ms.subservice: security
ms.topic: conceptual
ms.author: mbaldwin
ms.date: 03/15/2019
ms.custom: seodec18
ms.openlocfilehash: dc8349ed86a11b6588dbe4de3bf3d274de289374
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87284513"
---
# <a name="azure-disk-encryption-with-azure-ad-previous-release"></a>Azure Disk Encryption con Azure AD (versión anterior)

**La nueva versión de Azure Disk Encryption elimina la necesidad de proporcionar un parámetro de aplicación de Azure AD para habilitar el cifrado de disco de máquina virtual. Con la nueva versión, ya no es necesario proporcionar credenciales de Azure AD durante el paso de habilitar el cifrado. Todas las nuevas máquinas virtuales deben estar cifradas sin los parámetros de aplicación de Azure AD con la nueva versión. Para conocer las instrucciones necesarias para habilitar el cifrado de disco de máquina virtual mediante la nueva versión, consulte [Azure Disk Encryption para máquinas virtuales Windows](disk-encryption-overview.md). Las máquinas virtuales que ya se han cifrado con parámetros de aplicación de Azure AD se siguen admitiendo y se deben seguir manteniendo con la sintaxis de AAD.**

Este artículo complementa el artículo [Azure Disk Encryption para máquinas virtuales Windows](disk-encryption-overview.md), ya que incluye con requisitos adicionales y requisitos previos para Azure Disk Encryption con Azure AD (versión anterior). La sección [Máquinas virtuales y sistemas operativos compatibles](disk-encryption-overview.md#supported-vms-and-operating-systems) no cambia.

## <a name="networking-and-group-policy"></a>Redes y directiva de grupo

**Para habilitar la característica Azure Disk Encryption con la sintaxis de parámetro de ADD anterior, las máquinas virtuales IaaS deben cumplir los siguientes requisitos de configuración de puntos de conexión de red:** 
  - Para que un token se conecte al almacén de claves, la máquina virtual IaaS debe poder conectarse a un punto de conexión de Azure Active Directory, \[login.microsoftonline.com\].
  - Para escribir las claves de cifrado en el almacén de claves, la máquina virtual IaaS debe poder conectarse al punto de conexión del almacén de claves.
  - La máquina virtual IaaS debe poder conectarse al punto de conexión de Azure Storage que hospeda el repositorio de extensiones de Azure y la cuenta de Azure Storage que hospeda los archivos VHD.
  -  Si su directiva de seguridad limita el acceso desde máquinas virtuales de Azure a Internet, puede resolver el URI anterior y configurar una regla concreta para permitir la conectividad de salida para las direcciones IP. Para más información, consulte [Azure Key Vault detrás de un firewall](../../key-vault/general/access-behind-firewall.md).
  - La máquina virtual que se va a cifrar debe estar configurada para usar TLS 1.2 como el protocolo predeterminado. Si TLS 1.0 se deshabilitó explícitamente y la versión de .NET no se ha actualizado a la 4.6 o posterior, el siguiente cambio en el registro habilitará ADE para seleccionar la versión más reciente de TLS:

```console
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319]
"SystemDefaultTlsVersions"=dword:00000001
"SchUseStrongCrypto"=dword:00000001

[HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319]
"SystemDefaultTlsVersions"=dword:00000001
"SchUseStrongCrypto"=dword:00000001` 
```

**Directiva de grupo:**
 - La solución Azure Disk Encryption usa el protector de claves externas de BitLocker para máquinas virtuales IaaS con Windows. Para las máquinas virtuales unidas en un dominio, no cree ninguna directiva de grupo que exija protectores de TPM. Para obtener información acerca de la directiva de grupo para "Permitir BitLocker sin un TPM compatible", consulte la [Referencia de la directiva de grupo de BitLocker](/windows/security/information-protection/bitlocker/bitlocker-group-policy-settings#bkmk-unlockpol1).

-  La directiva de BitLocker en máquinas virtuales de unión a un dominio con directivas de grupo personalizadas debe incluir la siguiente configuración: [Configuración de almacenamiento de usuario de información de recuperación de BitLocker -> Permitir clave de recuperación de 256 bits](/windows/security/information-protection/bitlocker/bitlocker-group-policy-settings). Azure Disk Encryption presentará un error cuando la configuración de la directiva de grupo personalizada para BitLocker sea incompatible. En máquinas que no tengan la configuración de directiva correcta, puede que sea necesario aplicar la nueva directiva, forzar la nueva directiva a actualizarse (gpupdate.exe /force) y luego reiniciar.  

## <a name="encryption-key-storage-requirements"></a>Requisitos de almacenamiento de la clave de cifrado  

Azure Disk Encryption requiere Azure Key Vault para controlar y administrar las claves y los secretos de cifrado de discos. El almacén de claves y las máquinas virtuales deben residir en la misma región y suscripción de Azure.

Para más información, consulte [Creación y configuración de un almacén de claves para Azure Disk Encryption con Azure AD (versión anterior)](disk-encryption-key-vault-aad.md).
 
## <a name="next-steps"></a>Pasos siguientes

- [Creación y configuración de un almacén de claves para Azure Disk Encryption con Azure AD (versión anterior)](disk-encryption-key-vault-aad.md)
- [Habilitación de Azure Disk Encryption con Azure AD en máquinas virtuales Windows (versión anterior)](disk-encryption-windows-aad.md)
- [Script de la CLI de requisitos previos de Azure Disk Encryption](https://github.com/ejarvi/ade-cli-getting-started)
- [Script de PowerShell de requisitos previos de Azure Disk Encryption](https://github.com/Azure/azure-powershell/tree/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts)
