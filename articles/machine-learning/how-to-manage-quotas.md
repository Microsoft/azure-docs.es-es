---
title: Administración de recursos y cuotas
titleSuffix: Azure Machine Learning
description: Obtenga información sobre las cuotas de los recursos de Azure Machine Learning y cómo solicitar más cuota.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: jmartens
author: nishankgu
ms.author: nigup
ms.date: 05/08/2020
ms.topic: conceptual
ms.custom: troubleshooting,contperfq4
ms.openlocfilehash: a9ae3d2789758d03405fb5be82181c799d1ea692
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88141132"
---
# <a name="manage--increase-quotas-for-resources-with-azure-machine-learning"></a>Administración y aumento de las cuotas de los recursos con Azure Machine Learning
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo, obtendrá información sobre los límites preconfigurados en los recursos de Azure para la suscripción a [Azure Machine Learning](overview-what-is-azure-ml.md) y las cuotas que puede administrar. Estos límites se establecen para evitar salirse del presupuesto debido a fraudes y para respetar las restricciones de capacidad de Azure. 

Al igual que en otros servicios de Azure, existen límites en determinados recursos asociados con Azure Machine Learning. Dichos límites oscilan entre un extremo en el número de [áreas de trabajo](concept-workspace.md) y los límites en el proceso subyacente real que se usa para entrenar modelos o para la puntuación o inferencia de modelos. 

Tenga en cuenta estos límites al diseñar y escalar sus recursos de Azure Machine Learning para cargas de trabajo de producción. Por ejemplo, si su clúster no alcanza el número objetivo de nodos, es posible que se haya alcanzado el límite de núcleos del Proceso de Azure Machine Learning para la suscripción. Si desea aumentar el límite o la cuota por encima del límite predeterminado, abra una solicitud de soporte técnico al cliente en línea sin cargo alguno. Los límites no se pueden subir por encima del valor de Límite máximo que se muestra en las tablas siguientes debido a las restricciones de capacidad de Azure. Si la columna Límite máximo no existe, el recurso no tiene límites ajustables.


Además de administrar cuotas, también puede aprender a [planear y administrar los costos de Azure Machine Learning](concept-plan-manage-cost.md).

## <a name="special-considerations"></a>Consideraciones especiales

+ Una cuota es un límite de crédito, no una garantía de capacidad. Si tiene necesidades de capacidad a gran escala, póngase en contacto con el soporte técnico de Azure. También puede [aumentar las cuotas](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-quota-errors).

+ La cuota se comparte en todos los servicios de las suscripciones, incluso en Azure Machine Learning. La única excepción es el Proceso de Azure Machine Learning, el que tiene una cuota independiente de la cuota de procesos principales. Al evaluar las necesidades de capacidad, asegúrese de calcular el uso de la cuota entre todos los servicios.

+ Los límites predeterminados varían según el tipo de categoría de la oferta, por ejemplo, evaluación gratuita, pago por uso y series de máquina virtual, como Dv2, F, G, etc.

## <a name="default-resource-quotas"></a>Cuotas de recursos predeterminadas

Este es un desglose de los límites de cuota de los distintos tipos de recursos de su suscripción de Azure.

> [!IMPORTANT]
> Los límites están sujetos a cambios. El más reciente siempre se puede encontrar en el [documento](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits/) de la cuota de nivel de servicio para todo Azure.

### <a name="virtual-machines"></a>Máquinas virtuales
En cada suscripción de Azure, hay un límite en el número de máquinas virtuales que se pueden tener entre todos los servicios o de forma independiente. Los núcleos de las máquinas virtuales tienen un límite regional total y un límite de serie por tamaño (Dv2, F, etc.), y ambos límites se aplican por separado. Por ejemplo, considere una suscripción con un límite total de núcleos de máquinas virtuales de Este de EE. UU. de 30, un límite de núcleos de serie A de 30 y un límite de núcleos de serie D de 30. Esta suscripción podría volver a implementar 30 máquinas virtuales A1, 30 máquinas virtuales D1 o una combinación de las dos, para que no se superen los 30 núcleos (por ejemplo, 10 máquinas virtuales A1 y 20 máquinas virtuales D1).

[!INCLUDE [azure-subscription-limits-azure-resource-manager](../../includes/azure-subscription-limits-azure-resource-manager.md)]

### <a name="azure-machine-learning-compute"></a>Proceso de Azure Machine Learning
Para [Proceso de Azure Machine Learning](concept-compute-target.md#azure-machine-learning-compute-managed), hay un límite de cuota predeterminado tanto sobre el número de núcleos como el número de recursos de proceso único permitido por región en una suscripción. Esta cuota es independiente de la cuota de núcleos de máquina virtual anterior y los límites de núcleo no se comparten entre los dos tipos de recursos, ya que AmlCompute es un servicio administrado que implementa los recursos en un modelo hospedado representado.

Recursos disponibles:
+ Los núcleos dedicados por región tienen un límite predeterminado de 24 a 300 según el tipo de oferta de la suscripción con valores predeterminados más altos para los tipos de oferta EA y CSP.  El número de núcleos dedicados por suscripción se puede aumentar y es diferente para cada familia de máquinas virtuales. Ciertas familias de máquinas virtuales especializadas, como las series NCv2, NCv3 o ND, comienzan con un valor predeterminado de cero núcleos. Póngase en contacto con el Soporte técnico de Azure mediante la generación de una solicitud de cuota para analizar las opciones de aumento.

+ Los núcleos de baja prioridad por región tienen un límite predeterminado de 100 a 3000 según el tipo de oferta de la suscripción con valores predeterminados más altos para los tipos de oferta EA y CSP. El número de núcleos de baja prioridad por suscripción se puede aumentar y es un valor único en todas las familias de máquinas virtuales. Póngase en contacto con el soporte técnico de Azure para analizar las opciones de aumento.

+ Los clústeres por región tienen un límite predeterminado de 200. Se comparten entre un clúster de entrenamiento y una instancia de proceso (que se considera un clúster de un solo nodo con fines de cuota). Póngase en contacto con el soporte técnico de Azure si desea solicitar un número que supere este límite.

+ Hay otros límites estrictos que no se pueden superar una vez alcanzados.

| **Recurso** | **Límite máximo** |
| --- | --- |
| Número máximo de áreas de trabajo por grupo de recursos | 800 |
| Número máximo de nodos en un solo recurso de Proceso de Azure Machine Learning (AmlCompute) | 100 nodos |
| Número máximo de procesos MPI de GPU por nodo | 1-4 |
| Número máximo de trabajos de GPU por nodo | 1-4 |
| Duración máxima del trabajo | 21 días<sup>1</sup> |
| Duración máxima de un trabajo en un nodo de prioridad baja | 7 días<sup>2</sup> |
| Número máximo de servidores de parámetro por nodo | 1 |

<sup>1</sup> La duración máxima se refiere al tiempo en que empieza una ejecución y el momento en que finaliza. Las ejecuciones completadas se mantienen de forma indefinida; los datos de las ejecuciones no completadas dentro de la duración máxima no son accesibles.
<sup>2</sup> Los trabajos en un nodo de prioridad baja pueden anularse en el momento que exista una restricción de capacidad. Le recomendamos que implemente los puntos de comprobación en el trabajo.

### <a name="azure-machine-learning-pipelines"></a>Canalizaciones de Azure Machine Learning
En el caso de las [canalizaciones de Azure Machine Learning](concept-ml-pipelines.md), existe un límite de cuota sobre el número de pasos en una canalización y sobre el número de ejecuciones basadas en programación de canalizaciones publicadas por región en una suscripción.
- El número máximo de pasos permitidos en una canalización es de 30 000.
- El número máximo de la suma de ejecuciones basadas en programación y de extracción de blobs para las programaciones desencadenadas por el blog de canalizaciones publicadas por suscripción al mes es 100.000

### <a name="container-instances"></a>Instancias de contenedor

También hay un límite en el número de instancias de contenedor que se pueden poner en marcha en un período de tiempo determinado (el ámbito es una hora) o en toda la suscripción.
Para conocer los límites, consulte [Límites de las instancias de contenedor](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#container-instances-limits).

### <a name="storage"></a>Storage
Hay un límite en el número de cuentas de almacenamiento por región, así como en una suscripción determinada. El límite predeterminado es de 250 e incluye las cuentas Estándar y Premium Storage. Si necesita más de 250 cuentas de almacenamiento en una región dada, realice una solicitud a través del [servicio de soporte técnico de Azure](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest/). El equipo de Azure Storage revisará su caso empresarial y podría aprobar hasta 250 cuentas de almacenamiento para una región dada.


## <a name="workspace-level-quota"></a>Cuota de nivel de área de trabajo

Para administrar mejor las asignaciones de recursos para el destino de Proceso de Azure Machine Learning (Amlcompute) entre varias [áreas de trabajo](concept-workspace.md), hemos introducido una característica que le permite distribuir cuotas de nivel de suscripción (por familia de máquinas virtuales) y configurarlas en el nivel de área de trabajo. El comportamiento predeterminado es que todas las áreas de trabajo tienen la misma cuota que la cuota de nivel de suscripción de cualquier familia de máquinas virtuales. Sin embargo, a medida que aumenta el número de áreas de trabajo y las cargas de trabajo de distintas prioridades comienzan a compartir los mismos recursos, los usuarios quieren una manera de compartir mejor la capacidad y evitar problemas de contención de recursos. Azure Machine Learning proporciona una solución con su oferta de proceso administrada permitiendo a los usuarios establecer una cuota máxima para una familia de máquinas virtuales determinada en cada área de trabajo. Esto es análogo a la distribución de la capacidad entre áreas de trabajo y los usuarios pueden optar por sobreasignar para impulsar la máxima utilización. 

Para establecer cuotas en el nivel de área de trabajo, vaya a cualquier área de trabajo de la suscripción y haga clic en **Usos y cuotas** en el panel izquierdo. Después, seleccione la pestaña **Configurar cuotas** para ver las cuotas, expandir cualquier familia de máquinas virtuales y establecer un límite de cuota en cualquier área de trabajo que aparezca en esa familia de máquinas virtuales. Recuerde que no puede establecer un valor negativo o un valor mayor que la cuota de nivel de suscripción. Además, como observaría, de forma predeterminada, a todas las áreas de trabajo se les asigna toda la cuota de suscripción para permitir el uso completo de la cuota asignada.

[![Cuota de nivel de área de trabajo de Azure Machine Learning](./media/how-to-manage-quotas/azure-machine-learning-workspace-quota.png)](./media/how-to-manage-quotas/azure-machine-learning-workspace-quota.png)


> [!NOTE]
> Esta es una característica exclusiva de la edición Enterprise. Si dispone de un área de trabajo de la [edición Basic y de la edición Enterprise](overview-what-is-azure-ml.md#sku) en su suscripción, puede usarla solo para establecer cuotas en las áreas de trabajo Enterprise. Las áreas de trabajo Basic seguirán teniendo la cuota de nivel de suscripción, que es el comportamiento predeterminado.
>
> Necesita permisos de nivel de suscripción para establecer la cuota en el nivel de área de trabajo. Esto se aplica para que los propietarios de áreas de trabajo individuales no editen ni aumenten sus cuotas y comiencen a utilizar los recursos reservados para otra área de trabajo. Por lo tanto, un administrador de suscripciones es más adecuado para asignar y distribuir estas cuotas entre áreas de trabajo.



## <a name="view-your-usage-and-quotas"></a>Visualización del uso y las cuotas

La cuota de Proceso de Azure Machine Learning de la suscripción se administra de forma independiente de otras cuotas de recursos de Azure. Para ver esta cuota, debe profundizar en los servicios de Machine Learning Service.  

1. En el panel izquierdo, seleccione **Machine Learning Service** y, a continuación, seleccione cualquier área de trabajo en la lista que aparecerá.

2. En la tabla siguiente, en la **sección Soporte técnico y solución de problemas**, seleccione **Uso y cuotas** para ver el uso y los límites de cuota actuales.

3. Seleccione una suscripción para ver los límites de cuota. No olvide filtrar por la región en que esté interesado.

4. Ahora puede alternar entre una vista de nivel de suscripción y una vista de nivel de área de trabajo:
    + **Vista de suscripción:** esta vista le permite ver el uso de la cuota de núcleos por familia de máquinas virtuales, ampliarla por área de trabajo y ampliarla aún más por los nombres de clúster reales. Esta vista es óptima para entrar rápidamente en los detalles del uso de los núcleos de una familia de máquinas virtuales en particular para ver la división por áreas de trabajo y por clústeres subyacentes de cada una de esas áreas de trabajo. La convención general de esta vista es (uso/cuota), donde el uso es el número actual de núcleos escalados verticalmente y la cuota es el número máximo lógico de núcleos al que se puede escalar el recurso. Para cada **área de trabajo**, la cuota sería la cuota de nivel de área de trabajo (como se explicó anteriormente), que indica el número máximo de núcleos a los que se puede escalar para una determinada familia de máquinas virtuales. De manera similar para un **clúster**, la cuota es en realidad los núcleos correspondientes al número máximo de nodos a los que el clúster puede escalar definido por la propiedad max_nodes.
    
    + **Vista de área de trabajo:** esta vista le permite ver el uso de la cuota de núcleos por área de trabajo, ampliarla por familia de máquinas virtuales y ampliarla aún más por los nombres de clúster reales. Esta vista es óptima para entrar rápidamente en los detalles del uso de los núcleos de un área de trabajo en particular para ver la división por familias de máquinas virtuales y por clústeres subyacentes de cada una de esas familias.

Mediante Azure Portal es fácil ver la cuota de otros diversos recursos de Azure, como las máquinas virtuales, el almacenamiento o la red.

1. En el panel izquierdo, seleccione **Todos los servicios** y, después, seleccione **Suscripciones** en la categoría General.

2. En la lista de suscripciones, seleccione la suscripción cuya cuota busca.

3. Seleccione **Uso y cuotas** para ver los límites y el uso actuales de la cuota. Use los filtros para seleccionar el proveedor y las ubicaciones. 

## <a name="request-quota-increases"></a>Solicitud de aumento de cuota

Si desea aumentar el límite o la cuota por encima del límite predeterminado, [abra una solicitud de soporte técnico al cliente en línea](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest/) sin cargo alguno.

Los límites no se pueden subir por encima del valor del límite máximo que se muestra en las tablas siguientes. Si no existe dicho límite, significa que el recurso no tiene límites ajustables. [Consulte las instrucciones paso a paso sobre cómo aumentar la cuota](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-quota-errors).

Al solicitar un aumento de cuota, es preciso seleccionar el servicio para el que se realiza la solicitud, que puede ser un servicio como cuota de Azure Machine Learning Service, instancias de contenedor o cuota de almacenamiento. Además de Proceso de Azure Machine Learning, puede hacer clic en el botón **Solicitar cuota** al visualizar la cuota según los pasos anteriores.

> [!NOTE]
> Las [suscripciones de evaluación gratuita](https://azure.microsoft.com/offers/ms-azr-0044p) no son aptas para aumentar el límite ni la cuota. Si tiene una [suscripción de evaluación gratuita](https://azure.microsoft.com/offers/ms-azr-0044p), puede actualizar a una suscripción de [Pago por uso](https://azure.microsoft.com/offers/ms-azr-0003p/). Para obtener más información, consulte [Actualización de evaluación gratuita de Azure a pago por uso](../billing/billing-upgrade-azure-subscription.md) y [Free Trial subscription FAQ](https://azure.microsoft.com/free/free-account-faq) (Preguntas más frecuentes de la suscripción de evaluación gratuita).

## <a name="next-steps"></a>Pasos siguientes

+ [Planeamiento y administración de los costos de Azure Machine Learning](concept-plan-manage-cost.md)
