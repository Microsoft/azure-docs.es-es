---
title: Profundización técnica en la migración del modelo clásico a Azure Resource Manager
description: Profundización técnica en la migración compatible con la plataforma del modelo de implementación clásico a Azure Resource Manager
author: tanmaygore
manager: vashan
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.topic: conceptual
ms.date: 02/06/2020
ms.author: tagore
ms.openlocfilehash: cffecb843eead26e9c3b0d01fa19d175ca15d161
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87268091"
---
# <a name="technical-deep-dive-on-platform-supported-migration-from-classic-to-azure-resource-manager"></a>Profundización técnica en la migración compatible con la plataforma de la implementación clásica a la de Azure Resource Manager

> [!IMPORTANT]
> En la actualidad, aproximadamente el 90 % de las máquinas virtuales de IaaS usan [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/). A partir del 28 de febrero de 2020, las máquinas virtuales clásicas han quedado en desuso y se van a retirar por completo el 1 de marzo de 2023. [Obtenga más información]( https://aka.ms/classicvmretirement) sobre esta caída en desuso y [cómo se va a ver afectado](../classic-vm-deprecation.md#how-does-this-affect-me).

Profundicemos en la migración desde el modelo de implementación clásica de Azure hasta el modelo de implementación de Azure Resource Manager. Nos centramos en los recursos en un nivel de recursos y función que le ayudarán a comprender cómo la Plataforma de Azure migra los recursos entre los dos modelos de implementación. Para obtener más información, lea el artículo de anuncio de servicio: [Migración compatible con la plataforma de recursos de IaaS del modelo clásico al de Azure Resource Manager](migration-classic-resource-manager-overview.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

[!INCLUDE [virtual-machines-common-migration-deep-dive](../../../includes/virtual-machines-common-classic-resource-manager-migration-deep-dive.md)]

## <a name="next-steps"></a>Pasos siguientes

* [Información general sobre la migración compatible con la plataforma de recursos de IaaS desde el modelo de implementación clásica a Azure Resource Manager](migration-classic-resource-manager-overview.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Planning for migration of IaaS resources from classic to Azure Resource Manager](migration-classic-resource-manager-plan.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) (Planificación de la migración de recursos de IaaS del modelo clásico a Azure Resource Manager)
* [Migración de recursos de IaaS de la implementación clásica a Resource Manager con Azure PowerShell](../windows/migration-classic-resource-manager-ps.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Migración de recursos de IaaS de la implementación clásica a Azure Resource Manager con la CLI de Azure](migration-classic-resource-manager-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Herramientas de la comunidad para ayudar con la migración de recursos de IaaS del modelo de implementación clásica a Azure Resource Manager](../windows/migration-classic-resource-manager-community-tools.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Revisión de los errores más comunes en la migración](migration-classic-resource-manager-errors.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Revisión de las preguntas más frecuentes acerca de cómo migrar recursos de IaaS del modelo de implementación clásica a Azure Resource Manager](migration-classic-resource-manager-faq.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
