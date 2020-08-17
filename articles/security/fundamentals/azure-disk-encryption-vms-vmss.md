---
title: Azure Disk Encryption para máquinas virtuales y conjuntos de escalado de máquinas virtuales
description: Más información acerca de Azure Disk Encryption para máquinas virtuales (VM) y conjuntos de escalado de VM. Azure Disk Encryption funciona para VM Linux y Windows.
author: msmbaldwin
ms.service: security
ms.topic: article
ms.author: mbaldwin
ms.date: 10/15/2019
ms.custom: seodec18
ms.openlocfilehash: 21194bf2fe76a7eb0ee034d4a502c20ee3032dd9
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543680"
---
# <a name="azure-disk-encryption-for-virtual-machines-and-virtual-machine-scale-sets"></a>Azure Disk Encryption para máquinas virtuales y conjuntos de escalado de máquinas virtuales

Azure Disk Encryption se puede aplicar a máquinas virtuales Linux y Windows, así como a conjuntos de escalado de máquinas virtuales. 

## <a name="linux-virtual-machines"></a>Máquinas virtuales Linux

En los siguientes artículos se proporcionan instrucciones para el cifrado de máquinas virtuales Linux.

### <a name="current-version-of-azure-disk-encryption"></a>Versión actual de Azure Disk Encryption

- [Introducción sobre el uso de Azure Disk Encryption para máquinas virtuales Linux](../../virtual-machines/linux/disk-encryption-overview.md)
- [Escenarios de Azure Disk Encryption en VM Linux](../../virtual-machines/linux/disk-encryption-linux.md)
- [Creación y cifrado de una máquina virtual Linux con la CLI de Azure](../../virtual-machines/linux/disk-encryption-cli-quickstart.md)
- [Creación y cifrado de una máquina virtual Linux con Azure PowerShell](../../virtual-machines/linux/disk-encryption-powershell-quickstart.md)
- [Creación y cifrado de una máquina virtual Linux en Azure Portal](../../virtual-machines/linux/disk-encryption-portal-quickstart.md)
- [Esquema de la extensión Azure Disk Encryption para Linux](../../virtual-machines/extensions/azure-disk-enc-linux.md)
- [Creación y configuración de un almacén de claves para Azure Disk Encryption](../../virtual-machines/linux/disk-encryption-key-vault.md)
- [Scripts de ejemplo de Azure Disk Encryption](../../virtual-machines/linux/disk-encryption-sample-scripts.md)
- [Solución de problemas de Azure Disk Encryption](../../virtual-machines/linux/disk-encryption-troubleshooting.md)
- [Preguntas frecuentes sobre Azure Disk Encryption](../../virtual-machines/linux/disk-encryption-faq.md)

### <a name="azure-disk-encryption-with-azure-ad-previous-version"></a>Azure Disk Encryption con Azure AD (versión anterior)

- [Introducción sobre el uso de Azure Disk Encryption con Azure AD para máquinas virtuales Linux](../../virtual-machines/linux/disk-encryption-overview-aad.md)
- [Escenarios de Azure Disk Encryption con Azure AD en máquinas virtuales Linux](../../virtual-machines/linux/disk-encryption-linux.md)
- [Creación y configuración de un almacén de claves para Azure Disk Encryption con Azure AD (versión anterior)](../../virtual-machines/linux/disk-encryption-key-vault-aad.md)

## <a name="windows-virtual-machines"></a>Máquinas virtuales Windows

En los siguientes artículos se proporcionan instrucciones para el cifrado de máquinas virtuales Windows.

### <a name="current-version-of-azure-disk-encryption"></a>Versión actual de Azure Disk Encryption

- [Introducción a Azure Disk Encryption para máquinas virtuales Windows](../../virtual-machines/windows/disk-encryption-overview.md)
- [Escenarios de Azure Disk Encryption en máquinas virtuales Windows](../../virtual-machines/windows/disk-encryption-windows.md)
- [Creación y cifrado de una máquina virtual Windows con la CLI de Azure](../../virtual-machines/windows/disk-encryption-cli-quickstart.md)
- [Creación y cifrado de una máquina virtual Windows con Azure PowerShell](../../virtual-machines/windows/disk-encryption-powershell-quickstart.md)
- [Creación y cifrado de una máquina virtual Windows con Azure Portal](../../virtual-machines/windows/disk-encryption-portal-quickstart.md)
- [Esquema de la extensión Azure Disk Encryption para Windows](../../virtual-machines/extensions/azure-disk-enc-windows.md)
- [Creación y configuración de un almacén de claves para Azure Disk Encryption](../../virtual-machines/windows/disk-encryption-key-vault.md)
- [Scripts de ejemplo de Azure Disk Encryption](../../virtual-machines/windows/disk-encryption-sample-scripts.md)
- [Solución de problemas de Azure Disk Encryption](../../virtual-machines/windows/disk-encryption-troubleshooting.md)
- [Preguntas frecuentes sobre Azure Disk Encryption](../../virtual-machines/windows/disk-encryption-faq.md)

### <a name="azure-disk-encryption-with-azure-ad-previous-version"></a>Azure Disk Encryption con Azure AD (versión anterior)

- [Introducción sobre el uso de Azure Disk Encryption con Azure AD para máquinas virtuales Windows](../../virtual-machines/windows/disk-encryption-overview-aad.md)
- [Escenarios de Azure Disk Encryption con Azure AD en máquinas virtuales Windows](../../virtual-machines/windows/disk-encryption-windows.md)
- [Creación y configuración de un almacén de claves para Azure Disk Encryption con Azure AD (versión anterior)](../../virtual-machines/windows/disk-encryption-key-vault-aad.md)

## <a name="virtual-machine-scale-sets"></a>Conjuntos de escalado de máquinas virtuales

En los siguientes artículos se proporcionan instrucciones para el cifrado de conjuntos de escalado de máquinas virtuales.

- [Introducción al uso de Azure Disk Encryption para conjuntos de escalado de máquinas virtuales](../../virtual-machine-scale-sets/disk-encryption-overview.md) 
- [Cifrado de conjuntos de escalado de máquinas virtuales mediante la CLI de Azure](../../virtual-machine-scale-sets/disk-encryption-cli.md) 
- [Cifrado de conjuntos de escalado de máquinas virtuales mediante Azure PowerShell](../../virtual-machine-scale-sets/disk-encryption-powershell.md)
- [Cifrado de conjuntos de escalado de máquinas virtuales con Resource Manager](../../virtual-machine-scale-sets/disk-encryption-azure-resource-manager.md)
- [Creación y configuración de un almacén de claves para Azure Disk Encryption](../../virtual-machine-scale-sets/disk-encryption-key-vault.md)
- [Uso de Azure Disk Encryption con la secuenciación de extensiones de un conjunto de escalado de máquinas virtuales](../../virtual-machine-scale-sets/disk-encryption-extension-sequencing.md)

## <a name="next-steps"></a>Pasos siguientes

- [Información general del cifrado de Azure](encryption-overview.md)
- [Cifrado de datos en reposo](encryption-atrest.md)
- [Procedimientos recomendados de seguridad de datos y cifrado](data-encryption-best-practices.md)
