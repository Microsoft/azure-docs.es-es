---
title: Administración y solicitud de cuotas de recursos
titleSuffix: Azure Machine Learning service
description: En esta guía se explican las distintas cuotas de los recursos de Azure Machine Learning y cómo ver y solicitar una cuota mayor.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: jmartens
author: nishankgu
ms.author: nigup
ms.date: 05/10/2019
ms.custom: seodec18
ms.openlocfilehash: f9734a5d8f34536558fbf0c861889f3c7d6719da
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65524000"
---
# <a name="manage-and-request-quotas-for-azure-resources"></a>Administración y solicitud de cuotas para recursos de Azure

Al igual que en otros servicios de Azure, existen límites en determinados recursos asociados con el servicio Azure Machine Learning. Dichos límites oscilan entre un tope en el número de áreas de trabajo que se pueden crear a límites en el proceso subyacente real que se usa para entrenar o inferior modelos. En este artículo se proporciona más información acerca de los límites previamente configurados de varios recursos de Azure de su suscripción. También encontrará vínculos útiles para solicitar mejoras de la cuota de todos los tipos de recurso. Estos límites se establecen para evitar salirse del presupuesto debido a fraudes y para respetar las restricciones de capacidad de Azure.

Tenga en cuenta estas cuotas al diseñar y escalar verticalmente sus recursos de Azure Machine Learning Service para cargas de trabajo de producción. Por ejemplo, si el clúster no alcance el número de nodos especificado, a continuación, es posible que haya alcanzado un límite de núcleos de Azure Machine Learning Compute para su suscripción. Si desea aumentar el límite o la cuota por encima del límite predeterminado, abra una solicitud de soporte técnico al cliente en línea sin cargo alguno. Los límites no se pueden subir por encima del valor de Límite máximo que se muestra en las tablas siguientes debido a las restricciones de capacidad de Azure. Si la columna Límite máximo no existe, el recurso no tiene límites ajustables.

## <a name="special-considerations"></a>Consideraciones especiales

+ Una cuota es un límite de crédito, no una garantía de capacidad. Si tiene necesidades de capacidad a gran escala, póngase en contacto con el soporte técnico de Azure.

+ La cuota se comparte en todos los servicios de las suscripciones, incluso en Azure Machine Learning Service. La única excepción es el Proceso de Azure Machine Learning, el que tiene una cuota independiente de la cuota de procesos principales. Al evaluar las necesidades de capacidad, asegúrese de calcular el uso de la cuota entre todos los servicios.

+ Los límites predeterminados varían según el tipo de categoría de la oferta, por ejemplo, evaluación gratuita, pago por uso y series, como Dv2, F, G, etc.

## <a name="default-resource-quotas"></a>Cuotas de recursos predeterminadas

Este es un desglose de los límites de cuota de los distintos tipos de recursos de su suscripción de Azure.

> [!Important]
> Los límites están sujetos a cambios. El más reciente siempre se puede encontrar en el [documento](https://docs.microsoft.com/azure/azure-subscription-service-limits/) de la cuota de nivel de servicio para todo Azure.

### <a name="virtual-machines"></a>Máquinas virtuales
Hay un límite en el número de máquinas virtuales que se pueden aprovisionar en una suscripción de Azure mediante los servicios, o bien de forma independiente. Dicho límite se produce a nivel de región tanto en el total de núcleos como por familia.

Es importante destacar que los núcleos de las máquinas virtuales tienen un límite regional total y un límite de serie por tamaño (Dv2, F, etc.), y ambos límites se aplican por separado. Por ejemplo, considere una suscripción con un límite total de núcleos de máquinas virtuales de Este de EE. UU. de 30, un límite de núcleos de serie A de 30 y un límite de núcleos de serie D de 30. Esta suscripción podría volver a implementar 30 máquinas virtuales A1, 30 máquinas virtuales D1 o una combinación de las dos, para que no se superen los 30 núcleos (por ejemplo, 10 máquinas virtuales A1 y 20 máquinas virtuales D1).

[!INCLUDE [azure-subscription-limits-azure-resource-manager](../../../includes/azure-subscription-limits-azure-resource-manager.md)]

Para obtener una lista más detallada y actualizada de los límites de cuota, consulte [aquí](https://docs.microsoft.com/azure/azure-subscription-service-limits) el artículo de la cuota en todo Azure.

### <a name="azure-machine-learning-compute"></a>Proceso de Azure Machine Learning
Para Proceso de Azure Machine Learning, hay un límite de cuota predeterminado tanto sobre el número de núcleos como el número de recursos de proceso único permitido por región en una suscripción. Esta cuota es independiente de la cuota de los núcleos de las máquinas virtuales anterior y actualmente los límites de los núcleos no se comparten entre los dos tipos de recursos.

Recursos disponibles:
+ Núcleos dedicados por región tienen un límite predeterminado de 24-300 según el tipo de oferta de suscripción.  El número de núcleos dedicados por suscripción se puede aumentar. Póngase en contacto con el soporte técnico de Azure para analizar las opciones de aumento.

+ Núcleos de baja prioridad por cada región tienen un límite predeterminado de 24-300 según el tipo de oferta de suscripción.  El número de núcleos de baja prioridad por suscripción se puede aumentar. Póngase en contacto con el soporte técnico de Azure para analizar las opciones de aumento.

+ El límite predeterminado de clústeres por región es de 100, mientras que el máximo es 200. Póngase en contacto con el soporte técnico de Azure si desea solicitar un número que supere este límite.

+ Hay **otros límites estrictos** que no se pueden superar una vez alcanzados.

| **Recurso** | **Límite máximo** |
| --- | --- |
| Número máximo de áreas de trabajo por grupo de recursos | 800 |
| Número máximo de nodos en un solo recurso de Proceso de Azure Machine Learning (AmlCompute) | 100 nodos |
| Número máximo de procesos MPI de GPU por nodo | 1-4 |
| Número máximo de trabajos de GPU por nodo | 1-4 |
| Duración máxima del trabajo | 90 días<sup>1</sup> |
| Duración máxima de un trabajo en un nodo de prioridad baja | 1 día<sup>2</sup> |
| Número máximo de servidores de parámetro por nodo | 1 |

<sup>1</sup> La duración máxima se refiere al tiempo en que empieza una ejecución y el momento en que finaliza. Las ejecuciones completadas se mantienen de forma indefinida; los datos de las ejecuciones no completadas dentro de la duración máxima no son accesibles.
<sup>2</sup> trabajos en un nodo de prioridad baja se podrían tenderá a cualquier momento hay una restricción de capacidad. Se recomienda implementar los puntos de comprobación en su trabajo.

### <a name="container-instances"></a>Instancias de contenedor

También hay un límite en el número de instancias de contenedor que se pueden poner en marcha en un período de tiempo determinado (el ámbito es una hora) o en toda la suscripción.

[!INCLUDE [container-instances-limits](../../../includes/container-instances-limits.md)]

Para obtener una lista más detallada y actualizada de los límites de cuota, consulte [aquí](https://docs.microsoft.com/azure/azure-subscription-service-limits#container-instances-limits) el artículo de la cuota en todo Azure.

### <a name="storage"></a>Almacenamiento
Hay un límite en el número de cuentas de almacenamiento por región, así como en una suscripción determinada. Dicho límite es de 200 e incluye las cuentas Estándar y Premium Storage. Si necesita más de 200 cuentas de almacenamiento en una región dada, realice una solicitud a través del [servicio de soporte técnico de Azure](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest/). El equipo de Azure Storage revisará su caso empresarial y podría aprobar hasta 250 cuentas de almacenamiento para una región dada.


## <a name="find-your-quotas"></a>Búsqueda de cuotas

Mediante Azure Portal es fácil ver la cuota de varios recursos, como las máquinas virtuales, el almacenamiento la red.

1. En el panel izquierdo, seleccione **Todos los servicios** y, después, seleccione **Suscripciones** en la categoría General.

1. En la lista de suscripciones, seleccione la suscripción cuya cuota busca.

   **Hay una salvedad**, específicamente para ver la cuota de Proceso de Azure Machine Learning. Como ya se ha indicado, dicha cuota es independiente de la cuota de proceso de su suscripción.

1. En el panel izquierdo, seleccione **servicio Machine Learning** y, a continuación, seleccione cualquier área de trabajo en la lista mostrada

1. En la tabla siguiente, en la **sección Soporte técnico y solución de problemas**, seleccione **Uso y cuotas** para ver el uso y los límites de cuota actuales.

1. Seleccione una suscripción para ver los límites de cuota. No olvide filtrar por la región en que esté interesado.


## <a name="request-quota-increases"></a>Solicitud de aumento de cuota

Si desea aumentar el límite o la cuota por encima del límite predeterminado, [abra una solicitud de soporte técnico al cliente en línea](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest/) sin cargo alguno.

Los límites no se pueden subir por encima del valor del límite máximo que se muestra en las tablas siguientes. Si no existe dicho límite, significa que el recurso no tiene límites ajustables. En [este](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-quota-errors) artículo se trata el proceso de aumento de cuota con más detalle.

Al solicitar un aumento de cuota, es preciso seleccionar el servicio para el que se realiza la solicitud, que puede ser un servicio como cuota de Azure Machine Learning Service, instancias de contenedor o cuota de almacenamiento. Además de Proceso de Azure Machine Learning, puede simplemente hacer clic en el botón **Solicitar cuota** al visualizar la cuota según los pasos anteriores.

> [!NOTE]
> Las [suscripciones de evaluación gratuita](https://azure.microsoft.com/offers/ms-azr-0044p) no son aptas para aumentar el límite ni la cuota. Si tiene una [suscripción de evaluación gratuita](https://azure.microsoft.com/offers/ms-azr-0044p), puede actualizar a una suscripción de [Pago por uso](https://azure.microsoft.com/offers/ms-azr-0003p/). Para obtener más información, consulte [Actualización de evaluación gratuita de Azure a pago por uso](../../billing/billing-upgrade-azure-subscription.md) y [Free Trial subscription FAQ](https://azure.microsoft.com/free/free-account-faq) (Preguntas más frecuentes de la suscripción de evaluación gratuita).
