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
ms.openlocfilehash: 6ddc9299f72e8a0b25e8b1f85968933bd6868af8
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87288539"
---
# <a name="migrate-your-iaas-resources-to-azure-resource-manager-by-march-1-2023"></a>Migre los recursos de IaaS a Azure Resource Manager antes del 1 de marzo de 2023. 

En 2014, se inició IaaS en Azure Resource Manager y se han mejorado las funcionalidades desde entonces. Dado que [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/) tiene ahora funcionalidades completas de IaaS y otras mejoras, hemos puesto en desuso las máquinas virtuales de IaaS mediante Azure Service Manager el 28 de febrero de 2020 y esta funcionalidad se retirará por completo el 1 de marzo de 2023. 

En la actualidad, aproximadamente el 90 % de las máquinas virtuales de IaaS usan Azure Resource Manager. Si va a utilizar recursos de IaaS a través de Azure Service Manager (ASM), empiece a planear la migración ahora y complétela antes del 1 de marzo de 2023 para aprovechar [Azure Resource Manager](../azure-resource-manager/management/index.yml).

Las máquinas virtuales clásicas seguirán la [directiva de ciclo de vida actual](https://support.microsoft.com/help/30881/modern-lifecycle-policy) para su retirada.

## <a name="how-does-this-affect-me"></a>¿Cómo me afecta esto? 

1) A partir del 28 de febrero de 2020, los clientes que no han usado las máquinas virtuales de IaaS a través de Azure Service Manager (ASM) en febrero de 2020 no podrán crear máquinas virtuales clásicas. 
2) El 1 de marzo de 2023 los clientes ya no podrán iniciar máquinas virtuales de IaaS mediante Azure Service Manager y aquellas que estén en ejecución o asignadas se detendrán y desasignarán. 
2) El 1 de marzo de 2023, se informará a las suscripciones que no se hayan migrado a Azure Resource Manager acerca de nuestras escalas de tiempo para eliminar las máquinas virtuales clásicas restantes.  

Los siguientes servicios y funcionalidades de Azure **NO** se verán afectados por esta retirada: 
- Cloud Services 
- Las cuentas de almacenamiento **no** se usan en máquinas virtuales clásicas. 
- Las redes virtuales **no** se usan en máquinas virtuales clásicas. 
- Otros recursos clásicos

## <a name="what-actions-should-i-take"></a>¿Qué medidas debo tomar? 

- Empiece ya a planear la migración a Azure Resource Manager. 

- [Más información](./windows/migration-classic-resource-manager-overview.md) sobre cómo migrar las máquinas virtuales [Linux](./linux/migration-classic-resource-manager-plan.md) y [Windows](./windows/migration-classic-resource-manager-plan.md) clásicas a Azure Resource Manager.

- Para más información, consulte las [preguntas más frecuentes sobre la migración del método clásico al de Azure Resource Manager](./windows/migration-classic-resource-manager-faq.md).

- Para consultar preguntas técnicas y problemas, y agregar suscripciones a la lista de permitidos, [póngase en contacto con el soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

- Para otras preguntas que no formen parte de las preguntas más frecuentes y comentarios, envíe un comentario a continuación.
