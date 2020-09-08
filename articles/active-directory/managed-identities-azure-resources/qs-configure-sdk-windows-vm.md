---
title: 'Uso de un SDK para configurar identidades administradas en una máquina virtual: Azure AD'
description: Instrucciones detalladas para configurar y usar identidades administradas de recursos de Azure en una VM de Azure mediante un SDK de Azure.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/28/2017
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6b4b55fa5a21ce56d27505a724bcf08f1a893c71
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89003463"
---
# <a name="configure-a-vm-with-managed-identities-for-azure-resources-using-an-azure-sdk"></a>Configuración de una máquina virtual con identidades administradas de recursos de Azure mediante un SDK de Azure

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Las identidades administradas de recursos de Azure proporcionan a los servicios de Azure una identidad administrada automáticamente en Azure Active Directory (AD). Puede usar esta identidad para autenticar a cualquier servicio que admita la autenticación de Azure AD, sin necesidad de tener credenciales en el código. 

En este artículo, aprenderá a habilitar y eliminar identidades administradas de recursos de Azure mediante un SDK de Azure.

## <a name="prerequisites"></a>Prerequisites

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

## <a name="azure-sdks-with-managed-identities-for-azure-resources-support"></a>Compatibilidad de SDK de Azure con identidades administradas de recursos de Azure 

Azure admite varias plataformas de programación a través de una serie de [SDK de Azure](https://azure.microsoft.com/downloads). Varias de ellas se han actualizado para ser compatibles con identidades administradas de recursos de Azure y proporcionan los ejemplos correspondientes para mostrar el uso. Esta lista se actualiza a medida que se agrega compatibilidad adicional:

| SDK | Muestra |
| --- | ------ | 
| .NET   | [Administración de recursos de una máquina virtual habilitada con identidades administradas de recursos de Azure habilitadas](https://azure.microsoft.com/resources/samples/aad-dotnet-manage-resources-from-vm-with-msi/) |
| Java   | [Administración del almacenamiento de una máquina virtual habilitada con identidades administradas de recursos de Azure](https://azure.microsoft.com/resources/samples/compute-java-manage-resources-from-vm-with-msi-in-aad-group/)|
| Node.js| [Creación de una máquina virtual con la identidad administrada asignada por el sistema habilitada](https://azure.microsoft.com/resources/samples/compute-node-msi-vm/) |
| Python | [Creación de una máquina virtual con la identidad administrada asignada por el sistema habilitada](https://azure.microsoft.com/resources/samples/compute-python-msi-vm/) |
| Ruby   | [Creación de una máquina virtual de Azure con la identidad asignada por el sistema habilitada](https://github.com/Azure-Samples/compute-ruby-msi-vm/) |

## <a name="next-steps"></a>Pasos siguientes

- Consulte los artículos relacionados en **Configuración de identidad para una máquina virtual de Azure** para obtener información sobre cómo puede usar también Azure Portal, PowerShell, la CLI y las plantillas de recursos.
