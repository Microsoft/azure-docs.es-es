---
title: 'Uso de las identidades administradas en una VM de Azure con SDK de Azure: Azure AD'
description: Ejemplos de código para usar SDK de Azure con una máquina virtual de Azure con identidades administradas de recursos de Azure.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/01/2017
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: ecfb2fa5f45a23d387741d4865aa9707df960e86
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018423"
---
# <a name="how-to-use-managed-identities-for-azure-resources-on-an-azure-vm-with-azure-sdks"></a>Cómo usar las identidades administradas de recursos de Azure en una VM de Azure con SDK de Azure 

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]  
En este artículo se proporciona una lista de ejemplos de SDK que demuestran el uso de sus SDK de Azure respectivos con identidades administradas de recursos de Azure.

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

> [!IMPORTANT]
> - En todo el script o código de ejemplo de este artículo se da por supuesto que el cliente se ejecuta en una VM con identidades administradas de recursos de Azure habilitadas. Use la característica "Conectar" de la máquina virtual en Azure Portal para conectarse a la máquina virtual de forma remota. Para más información sobre la habilitación de identidades administradas de recursos de Azure en una VM, consulte [Configure managed identities for Azure resources on a VM using the Azure portal](qs-configure-portal-windows-vm.md) (Configuración de identidades administradas de recursos de Azure en una VM mediante Azure Portal) o uno de los artículos derivados (mediante PowerShell, la CLI, una plantilla o un SDK de Azure). 

## <a name="sdk-code-samples"></a>Ejemplos de código de SDK

| SDK             | Código de ejemplo |
| --------------- | ----------- |
| .NET            | [Deploy an Azure Resource Manager template from a Windows VM using managed identities for Azure resources](https://github.com/Azure-Samples/windowsvm-msi-arm-dotnet) (Implementación de una plantilla de Azure Resource Manager desde una máquina virtual Windows mediante identidades administradas de recursos de Azure) |
| .NET Core       | [Call Azure services from a Linux VM using managed identities for Azure resources](https://github.com/Azure-Samples/linuxvm-msi-keyvault-arm-dotnet/) (Llamada a servicios de Azure desde una máquina virtual Linux mediante identidades administradas de recursos de Azure) |
| Node.js         | [Administración de recursos mediante identidades administradas de recursos de Azure](https://azure.microsoft.com/resources/samples/resources-node-manage-resources-with-msi/) |
| Python          | [Uso de identidades administradas de recursos de Azure para la autenticación simple desde una máquina virtual](https://azure.microsoft.com/resources/samples/resource-manager-python-manage-resources-with-msi/) |
| Ruby            | [Administración de recursos de una máquina virtual con identidades administradas de recursos de Azure habilitadas](https://github.com/Azure-Samples/resources-ruby-manage-resources-with-msi/) |

## <a name="next-steps"></a>Pasos siguientes

- Consulte los [SDK de Azure](https://azure.microsoft.com/downloads/) para ver la lista completa de recursos de SDK de Azure, como las descargas de bibliotecas, la documentación y mucho más.
- Para habilitar las identidades administradas de recursos de Azure en una máquina virtual de Azure, consulte [Configure managed identities for Azure resources on a VM using the Azure portal](qs-configure-portal-windows-vm.md) (Configuración de identidades administradas de recursos de Azure en una VM mediante Azure Portal).








