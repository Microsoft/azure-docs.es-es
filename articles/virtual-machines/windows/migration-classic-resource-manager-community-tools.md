---
title: 'Herramientas de la comunidad: mover recursos clásicos a Azure Resource Manager'
description: En este artículo se catalogan las herramientas que la comunidad proporcionó para ayudar en la migración de los recursos de IaaS del modelo de implementación clásico al de Azure Resource Manager.
author: tanmaygore
manager: vashan
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: conceptual
ms.date: 02/06/2020
ms.author: tagore
ms.openlocfilehash: afc20a39a5b426f37d6a6752056e3dd35dd8438a
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88504966"
---
# <a name="community-tools-to-migrate-iaas-resources-from-classic-to-azure-resource-manager"></a>Herramientas de la comunidad para la migración de recursos de IaaS de la implementación clásica a Azure Resource Manager

> [!IMPORTANT]
> En la actualidad, aproximadamente el 90 % de las VM de IaaS usan [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/). A partir del 28 de febrero de 2020, las máquinas virtuales clásicas han quedado en desuso y se van a retirar por completo el 1 de marzo de 2023. [Obtenga más información]( https://aka.ms/classicvmretirement) sobre esta caída en desuso y [cómo se verá afectado](../classic-vm-deprecation.md#how-does-this-affect-me).

En este artículo se catalogan las herramientas que la comunidad proporcionó para ayudar en la migración de los recursos de IaaS del modelo de implementación clásico al de Azure Resource Manager.

> [!NOTE]
> Estas herramientas no se incluyen en el soporte técnico de Microsoft. Por lo tanto, son de código abierto en GitHub y estamos encantados de aceptar PR para correcciones o escenarios adicionales. Para informar sobre un problema, use la característica de problemas de GitHub.
> 
> La migración con estas herramientas provocará tiempo de inactividad en la máquina virtual clásica. Si lo que busca es información sobre la migración compatible con la plataforma, visite 
> 
>   * [Migración compatible con la plataforma de recursos de IaaS del modelo clásico al de Azure Resource Manager](migration-classic-resource-manager-overview.md)
>   * [Profundización técnica en la migración compatible con la plataforma de la implementación clásica a Azure Resource Manager](../migration-classic-resource-manager-deep-dive.md)
>   * [Migración de recursos de IaaS de la implementación clásica a Resource Manager con Azure PowerShell](migration-classic-resource-manager-ps.md)
> 
> 

## <a name="asmmetadataparser"></a>AsmMetadataParser
Se trata de una colección de herramientas auxiliares creadas como parte de las migraciones empresariales de Azure Service Management a Azure Resource Manager. Esta herramienta permite replicar la infraestructura en otra suscripción, que puede usarse para probar la migración y resolver los posibles problemas antes de ejecutar la migración en la suscripción de producción.

[Vínculo a la documentación sobre la herramienta](https://github.com/Azure/classic-iaas-resourcemanager-migration/tree/master/AsmToArmMigrationApiToolset)

## <a name="migaz"></a>migAz
migAz es una opción adicional para migrar un conjunto completo de recursos de IaaS clásicos a recursos de IaaS de Azure Resource Manager. La migración se puede producir dentro de la misma suscripción o entre suscripciones distintas y distintos tipos de suscripción (por ejemplo, suscripciones de CSP).

[Vínculo a la documentación sobre la herramienta](https://github.com/Azure/migAz)

## <a name="next-steps"></a>Pasos siguientes

* [Información general sobre la migración compatible con la plataforma de recursos de IaaS desde el modelo de implementación clásica a Azure Resource Manager](migration-classic-resource-manager-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Profundización técnica en la migración compatible con la plataforma de la implementación clásica a la de Azure Resource Manager](../migration-classic-resource-manager-deep-dive.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Planning for migration of IaaS resources from classic to Azure Resource Manager](migration-classic-resource-manager-plan.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) (Planificación de la migración de recursos de IaaS del modelo clásico a Azure Resource Manager)
* [Migración de recursos de IaaS de la implementación clásica a Resource Manager con Azure PowerShell](migration-classic-resource-manager-ps.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Migración de recursos de IaaS de la implementación clásica a Azure Resource Manager con la CLI de Azure](../linux/migration-classic-resource-manager-cli.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Revisión de los errores más comunes en la migración](migration-classic-resource-manager-errors.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Revisión de las preguntas más frecuentes acerca de cómo migrar recursos de IaaS del modelo de implementación clásica a Azure Resource Manager](../migration-classic-resource-manager-faq.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
