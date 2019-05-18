---
title: Cifrado de Azure Storage para datos en reposo | Microsoft Docs
description: Almacenamiento de Azure protege los datos cifrándolos automáticamente guardarlos antes en la nube. Todos los datos en Azure Storage se cifra y descifra de forma transparente mediante cifrado AES de 256 bits y es compatible con FIPS 140-2.
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 05/15/2019
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: 1e95adbd1a564fb34d3f0506ac1cc25bc5a63c62
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65790052"
---
# <a name="azure-storage-encryption-for-data-at-rest"></a>Cifrado de Azure Storage para datos en reposo

Azure Storage cifra automáticamente los datos al almacenarlos en la nube. El cifrado protege los datos y para ayudar a satisfacer los compromisos de cumplimiento y seguridad organizativos. Datos de Azure Storage se cifra y descifra de forma transparente de 256 bits [cifrado AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard), uno de los bloques más sólidos disponibles, y es compatible con FIPS 140-2. Cifrado de almacenamiento de Azure es similar a cifrado de BitLocker en Windows.

Cifrado de almacenamiento de Azure está habilitado para todas las cuentas de almacenamiento nuevas y existentes y no se puede deshabilitar. Dado que los datos están protegidos de forma predeterminada, no es necesario modificar el código o las aplicaciones para aprovechar las ventajas del cifrado de almacenamiento de Azure. 

Las cuentas de almacenamiento se cifran independientemente de su nivel de rendimiento (estándar o premium) o el modelo de implementación (Azure Resource Manager o clásico). Todas las opciones de redundancia de almacenamiento de Azure admiten el cifrado, y se cifran todas las copias de una cuenta de almacenamiento. Se cifran todos los recursos de Azure Storage, incluidos blobs, discos, archivos, colas y tablas. También se cifran todos los metadatos del objeto.

El cifrado no afectar al rendimiento de Azure Storage. No hay ningún costo adicional para el cifrado de almacenamiento de Azure.

Para obtener más información acerca de los módulos criptográficos subyacente cifrado de almacenamiento de Azure, consulte [Cryptography API: última generación](https://docs.microsoft.com/windows/desktop/seccng/cng-portal).

## <a name="key-management"></a>Administración de claves

Puede confiar en las claves administradas por Microsoft para el cifrado de la cuenta de almacenamiento, o puede administrar el cifrado con sus propias claves, junto con Azure Key Vault.

### <a name="microsoft-managed-keys"></a>Claves administradas por Microsoft

De forma predeterminada, la cuenta de almacenamiento utiliza claves de cifrado administradas por Microsoft. Puede ver la configuración de cifrado para la cuenta de almacenamiento en el **cifrado** sección de la [portal Azure](https://portal.azure.com), tal y como se muestra en la siguiente imagen.

![Ver cuenta cifrada con claves administradas por Microsoft](media/storage-service-encryption/encryption-microsoft-managed-keys.png)

### <a name="customer-managed-keys"></a>Claves administradas por el cliente

Puede administrar el cifrado de almacenamiento de Azure con claves administradas por el cliente. Claves administradas por el cliente proporcionan más flexibilidad para crear, girar, deshabilitar y revocar el acceso a los controles. También puede auditar las claves de cifrado utilizadas para proteger los datos. 

Usar Azure Key Vault para administrar las claves y auditar su uso. Puede crear sus propias claves y almacenarlas en un almacén de claves, o puede usar las API de Azure Key Vault para generar las claves. La cuenta de almacenamiento y el almacén de claves deben estar en la misma región, pero pueden estar en distintas suscripciones. Para obtener más información sobre Azure Key Vault, consulte [¿qué es Azure Key Vault?](../../key-vault/key-vault-overview.md).

Para revocar el acceso a las claves administradas por el cliente, consulte [PowerShell de Azure Key Vault](https://docs.microsoft.com/powershell/module/azurerm.keyvault/) y [CLI de Azure Key Vault](https://docs.microsoft.com/cli/azure/keyvault). Revocar acceso eficazmente bloquea el acceso a todos los datos en la cuenta de almacenamiento, como la clave de cifrado no es accesible mediante el almacenamiento de Azure.

Para obtener información sobre cómo usar claves administradas por el cliente con el almacenamiento de Azure, consulte uno de estos artículos:

- [Configurar las claves administradas por el cliente para el cifrado de almacenamiento de Azure desde el portal de Azure](storage-encryption-keys-portal.md)
- [Configurar las claves administradas por el cliente para el cifrado de almacenamiento de Azure desde PowerShell](storage-encryption-keys-powershell.md)
- [Usar claves administradas por el cliente con el cifrado de almacenamiento de Azure desde la CLI de Azure](storage-encryption-keys-cli.md)

> [!IMPORTANT]
> Claves administradas por el cliente se basan en las identidades administradas para los recursos de Azure, una característica de Azure Active Directory (Azure AD). No se actualizan cuando se transfiere una suscripción de un directorio de Azure AD a otras, administradas identidades y claves administradas por el cliente ya no funcionarán. Para obtener más información, consulte **transferir una suscripción entre directorios de Azure AD** en [preguntas más frecuentes y problemas conocidos con managed identidades para los recursos de Azure](../../active-directory/managed-identities-azure-resources/known-issues.md#transferring-a-subscription-between-azure-ad-directories).  

> [!NOTE]  
> No se admiten claves administradas por el cliente para [discos administrados de Azure](../../virtual-machines/windows/managed-disks-overview.md).

## <a name="azure-storage-encryption-versus-disk-encryption"></a>Cifrado de almacenamiento de Azure frente a cifrado de disco

Con el cifrado de almacenamiento de Azure, se cifran todas las cuentas de almacenamiento de Azure y los recursos que contienen, incluidos los blobs de página que guardan los discos de máquina virtual de Azure. Además, se pueden cifrar los discos de máquina virtual de Azure con [Azure Disk Encryption](../../security/azure-security-disk-encryption-overview.md). Azure Disk Encryption usa estándar del sector [BitLocker](https://docs.microsoft.com/windows/security/information-protection/bitlocker/bitlocker-overview) en Windows y [DM-Crypt](https://en.wikipedia.org/wiki/Dm-crypt) en Linux para ofrecer soluciones de cifrado basado en el sistema operativo que se integran con Azure Key Vault.

## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es Azure Key Vault?](../../key-vault/key-vault-overview.md)
