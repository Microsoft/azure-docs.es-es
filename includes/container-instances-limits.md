---
author: dlepow
ms.service: container-instances
ms.topic: include
ms.date: 07/22/2020
ms.author: danlep
ms.openlocfilehash: 6878180ffedfaa53f25d2bdc6db72dcd7dd8b38b
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87384841"
---
| Resource | Límite |
| --- | :--- |
| Grupos de contenedores de SKU estándar por región por suscripción | 100<sup>1</sup> |
| Grupos de contenedores de SKU dedicados por región por suscripción | 0<sup>1</sup> |
| Número de contenedores por grupo de contenedor | 60 |
| Número de volúmenes por grupo de contenedor | 20 |
| Núcleos de SKU estándar (CPU) por región por suscripción | 10<sup>1,2</sup> | 
| Núcleos de SKU estándar (CPU) para GPU K80 por región por suscripción | 18<sup>1,2</sup> |
| Núcleos de SKU estándar (CPU) para GPU P100 o V100 por región por suscripción | 0<sup>1,2</sup> |
| Puertos por IP | 5 |
| Tamaño del registro de instancia de contenedor: instancia en ejecución | 4 MB |
| Tamaño del registro de instancia de contenedor: instancia detenida | 16 KB o 1000 líneas |
| Creaciones de contenedores cada hora |300<sup>1</sup> |
| Creaciones de contenedores cada 5 minutos | 100<sup>1</sup> |
| Eliminaciones de contenedores cada hora | 300<sup>1</sup> |
| Eliminaciones de contenedores cada 5 minutos | 100<sup>1</sup> |


<sup>1</sup>Para solicitar un aumento del límite, cree una [solicitud de soporte técnico de Azure][azure-support]. En las suscripciones gratuitas, como la [cuenta gratuita de Azure](https://azure.microsoft.com/offers/ms-azr-0044p/) y [Microsoft Azure for Students](https://azure.microsoft.com/offers/ms-azr-0170p/) no se pueden aumentar los límites ni las cuotas. Si tiene una suscripción gratuita, puede [realizar la actualización](../articles/cost-management-billing/manage/upgrade-azure-subscription.md) a una suscripción de Pago por uso.<br />
<sup>2</sup>Límite predeterminado de la suscripción de [Pago por uso](https://azure.microsoft.com/offers/ms-azr-0003p/). El límite puede variar en otros tipos de categoría.<br/>

<!-- LINKS - External -->
[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest
