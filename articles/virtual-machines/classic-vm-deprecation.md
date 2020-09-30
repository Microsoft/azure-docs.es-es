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
ms.openlocfilehash: de590168ecf4febe3c18442a6dc6c1c2eba4dc51
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89297610"
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

- Haga una lista de todas las máquinas virtuales afectadas. 
    - El tipo de máquina virtual "Máquinas virtuales (clásico)" en la [hoja Máquinas virtuales de Azure Portal](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.ClassicCompute%2FVirtualMachines) son todas las máquinas virtuales afectadas dentro de la suscripción. 
    - También puede realizar consultas en Azure Resource Graph mediante el [portal](https://portal.azure.com/#blade/HubsExtension/ArgQueryBlade/query/resources%0A%7C%20where%20type%20%3D%3D%20%22microsoft.classiccompute%2Fvirtualmachines%22) o [PowerShell](https://docs.microsoft.com/azure/governance/resource-graph/concepts/work-with-data) para ver la lista de todas las máquinas virtuales marcadas (clásicas), y su información relacionada, de las suscripciones seleccionadas. 
    - Hemos enviado correos electrónicos a los propietarios de suscripciones con la lista de todas las suscripciones que contienen estas máquinas virtuales (clásicas). Úselas para crear esta lista. 

- [Más información](./windows/migration-classic-resource-manager-overview.md) sobre cómo migrar las máquinas virtuales [Linux](./linux/migration-classic-resource-manager-plan.md) y [Windows](./windows/migration-classic-resource-manager-plan.md) clásicas a Azure Resource Manager.

- Para más información, consulte las [preguntas más frecuentes sobre la migración del método clásico al de Azure Resource Manager](./migration-classic-resource-manager-faq.md).

- Se recomienda iniciar el planeamiento con la [herramienta de migración de compatibilidad de la plataforma](https://docs.microsoft.com/azure/virtual-machines/windows/migration-classic-resource-manager-overview) para migrar las máquinas virtuales existentes en tres sencillos pasos: validar, preparar y confirmar. 
    - La herramienta está diseñada para migrar las máquinas virtuales sin apenas tiempo de inactividad o ninguno en absoluto. 
    - El primer paso, validación, no afecta de ningún modo a la implementación existente y proporciona una lista de todos los escenarios de migración no admitidos. 
    - Examine la [lista de soluciones alternativas](https://docs.microsoft.com/azure/virtual-machines/windows/migration-classic-resource-manager-overview#unsupported-features-and-configurations) para corregir su implementación y prepararla para la migración. 
    - Lo normal es que después de corregir todos los errores de validación no se encuentre con ningún problema durante los pasos de preparación y confirmación. Después de que la confirmación se realiza correctamente, la implementación se migra en vivo a Azure Resource Manager donde se puede administrar con las nuevas API que expone este marco de trabajo. 
    - Si esta herramienta no es adecuada para su migración, puede explorar [otras ofertas de proceso](https://docs.microsoft.com/azure/architecture/guide/technology-choices/compute-decision-tree). Dado que hay muchas ofertas de proceso de Azure, todas muy diferentes entre sí, no podemos proporcionar un método de migración admitido por la plataforma para estas ofertas.  
- Para consultar preguntas técnicas y problemas, y agregar suscripciones a la lista de permitidos, [póngase en contacto con el soporte técnico](https://ms.portal.azure.com/#create/Microsoft.Support/Parameters/{"pesId":"6f16735c-b0ae-b275-ad3a-03479cfa1396","supportTopicId":"8a82f77d-c3ab-7b08-d915-776b4ff64ff4"}).

- Complete la migración lo antes posible para evitar el impacto en el negocio y aproveche el rendimiento mejorado, la seguridad y las características nuevas que proporciona Azure Resource Manager. 

## <a name="what-resources-are-provided-to-me-for-this-migration"></a>¿Qué recursos se proporcionan para esta migración?

- [Microsoft Q&A](https://docs.microsoft.com/answers/topics/azure-virtual-machines-migration.html): soporte técnico de Microsoft y de la comunidad para la migración

- [Soporte técnico para la migración de Azure](https://ms.portal.azure.com/#create/Microsoft.Support/Parameters/{"pesId":"6f16735c-b0ae-b275-ad3a-03479cfa1396","supportTopicId":"1135e3d0-20e2-aec5-4ef0-55fd3dae2d58"}): equipo de soporte técnico dedicado para brindar asistencia técnica durante la migración

- [Microsoft Fast Track](https://www.microsoft.com/fasttrack): el equipo de Microsoft Fast Track puede proporcionar asistencia técnica durante la migración a los clientes elegibles. 

- Si su empresa u organización se ha asociado con Microsoft o trabaja con un representante de Microsoft como arquitecto de soluciones en la nube (CSA), administradores técnicos de cuentas (TAM), trabaje en conjunto con ellos para obtener recursos adicionales para la migración. 

