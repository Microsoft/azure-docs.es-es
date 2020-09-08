---
title: Vamos a retirar las máquinas virtuales clásicas de Azure el 1 de marzo de 2023
description: El artículo proporciona información general de alto nivel sobre la retirada de las máquinas virtuales clásicas.
author: tanmaygore
manager: vashan
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: conceptual
ms.date: 02/10/2020
ms.author: tagore
ms.openlocfilehash: 0b2b995a6fe4cedd14b2e4ceeddc5747ec2423cf
ms.sourcegitcommit: afa1411c3fb2084cccc4262860aab4f0b5c994ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/23/2020
ms.locfileid: "88754810"
---
# <a name="migrate-your-iaas-resources-to-azure-resource-manager-by-march-1-2023"></a>Migre los recursos de IaaS a Azure Resource Manager antes del 1 de marzo de 2023. 

En 2014, se inició IaaS en Azure Resource Manager y se han mejorado las funcionalidades desde entonces. Dado que [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/) tiene ahora funcionalidades completas de IaaS y otras mejoras, hemos puesto en desuso las máquinas virtuales de IaaS mediante [Azure Service Manager](https://docs.microsoft.com/azure/virtual-machines/windows/migration-classic-resource-manager-faq#what-is-azure-service-manager-and-what-does-it-mean-by-classic) el 28 de febrero de 2020 y esta funcionalidad se retirará por completo el 1 de marzo de 2023. 

En la actualidad, aproximadamente el 90 % de las máquinas virtuales de IaaS usan Azure Resource Manager. Si va a utilizar recursos de IaaS a través de Azure Service Manager (ASM), empiece a planear la migración ahora y complétela antes del 1 de marzo de 2023 para aprovechar [Azure Resource Manager](../azure-resource-manager/management/index.yml).

Las máquinas virtuales clásicas seguirán la [directiva de ciclo de vida actual](https://support.microsoft.com/help/30881/modern-lifecycle-policy) para su retirada.

## <a name="how-does-this-affect-me"></a>¿Cómo me afecta esto? 

- A partir del 28 de febrero de 2020, los clientes que no han usado las máquinas virtuales de IaaS a través de Azure Service Manager (ASM) en febrero de 2020 no podrán crear máquinas virtuales clásicas. 
- El 1 de marzo de 2023 los clientes ya no podrán iniciar máquinas virtuales de IaaS mediante Azure Service Manager y aquellas que estén en ejecución o asignadas se detendrán y desasignarán. 
- El 1 de marzo de 2023, se informará a las suscripciones que no se migran a Azure Resource Manager acerca de nuestras escalas de tiempo para eliminar las máquinas virtuales clásicas restantes.  

Los siguientes servicios y funcionalidades de Azure **NO** se verán afectados por esta retirada: 
- Cloud Services 
- Las cuentas de almacenamiento **no** se usan en máquinas virtuales clásicas. 
- Las redes virtuales **no** se usan en máquinas virtuales clásicas. 
- Otros recursos clásicos

## <a name="what-actions-should-i-take"></a>¿Qué medidas debo tomar? 

- Empiece ya a planear la migración a Azure Resource Manager. 

- Haga una lista de todas las máquinas virtuales afectadas. El tipo de máquina virtual "Máquinas virtuales (clásico)" en la [hoja Máquinas virtuales de Azure Portal](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.ClassicCompute%2FVirtualMachines) son todas las máquinas virtuales afectadas dentro de la suscripción. 

- [Más información](./windows/migration-classic-resource-manager-overview.md) sobre cómo migrar las máquinas virtuales [Linux](./linux/migration-classic-resource-manager-plan.md) y [Windows](./windows/migration-classic-resource-manager-plan.md) clásicas a Azure Resource Manager.

- Para más información, consulte las [preguntas más frecuentes sobre la migración del método clásico al de Azure Resource Manager](./migration-classic-resource-manager-faq.md).

- Para consultar preguntas técnicas y problemas, y agregar suscripciones a la lista de permitidos, [póngase en contacto con el soporte técnico](https://ms.portal.azure.com/#create/Microsoft.Support/Parameters/{"pesId":"6f16735c-b0ae-b275-ad3a-03479cfa1396","supportTopicId":"8a82f77d-c3ab-7b08-d915-776b4ff64ff4"}).

- Para otras preguntas que no formen parte de las preguntas más frecuentes y comentarios, envíe un comentario a continuación.

- Complete la migración lo antes posible para evitar el impacto en el negocio y aproveche el rendimiento mejorado, la seguridad y las características nuevas que proporciona Azure Resource Manager. 

## <a name="what-resources-are-provided-to-me-for-this-migration"></a>¿Qué recursos se proporcionan para esta migración?

- [Microsoft Q&A](https://docs.microsoft.com/answers/topics/azure-virtual-machines-migration.html): soporte técnico de Microsoft y de la comunidad para la migración

- [Soporte técnico para la migración de Azure](https://ms.portal.azure.com/#create/Microsoft.Support/Parameters/{"pesId":"6f16735c-b0ae-b275-ad3a-03479cfa1396","supportTopicId":"1135e3d0-20e2-aec5-4ef0-55fd3dae2d58"}): equipo de soporte técnico dedicado para brindar asistencia técnica durante la migración

- [Microsoft Fast Track](https://www.microsoft.com/fasttrack): el equipo de Microsoft Fast Track puede proporcionar asistencia técnica durante la migración a los clientes elegibles. 

- Si su empresa u organización se ha asociado con Microsoft o trabaja con un representante de Microsoft como arquitecto de soluciones en la nube (CSA), administradores técnicos de cuentas (TAM), trabaje en conjunto con ellos para obtener recursos adicionales para la migración. 

