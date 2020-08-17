---
title: Escalado de cuotas y límites en su laboratorio en Azure DevTest Labs | Microsoft Docs
description: En este artículo se describe cómo se puede escalar el laboratorio en Azure DevTest Labs. Vea las cuotas y los límites de uso y solicite un aumento.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 2166eaab073e2abc24ca24494ae13eb876db1fcf
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87533961"
---
# <a name="scale-quotas-and-limits-in-devtest-labs"></a>Escalado de cuotas y límites en DevTest Labs
Mientras trabaja en DevTest Labs, puede que observe que hay ciertos límites predeterminados para algunos recursos de Azure, lo que puede afectar al servicio DevTest Labs. Estos límites se conocen como **cuotas**.

> [!NOTE]
> El servicio DevTest Labs no impone cuotas. Las cuotas que podría encontrar son restricciones predeterminadas de la suscripción de Azure en general.

Puede usar cada recurso de Azure hasta que alcance su cuota. Cada suscripción tiene cuotas independientes y se realiza el seguimiento del uso para cada suscripción.

Por ejemplo, cada suscripción tiene una cuota predeterminada de 20 núcleos. Por lo tanto, si va a crear en el laboratorio máquinas virtuales con cuatro núcleos, solo puede crear cinco.

En [Límites de suscripción y de servicios de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md), se enumeran algunas de las cuotas más frecuentes para recursos de Azure. Entre los recursos que más se usan en un laboratorio y para los que puede encontrar cuotas, están los núcleos de VM, las direcciones IP públicas, la interfaz de red, los discos administrados, la asignación de roles de Azure y los circuitos ExpressRoute.

## <a name="view-your-usage-and-quotas"></a>Visualización del uso y las cuotas
Estos pasos muestran cómo ver las cuotas actuales en su suscripción para recursos específicos de Azure y comprobar qué porcentaje de cada cuota ha usado.

1. Inicie sesión en [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040).
1. Seleccione **Más servicios** y **Facturación** en la lista.
1. En la hoja Facturación, seleccione una suscripción.
4. Seleccione **Uso y cuotas**.

   ![Botón Uso y cuotas](./media/devtest-lab-scale-lab/devtestlab-usage-and-quotas-new.png)

   Aparece la hoja Uso y cuotas, donde se muestran los distintos recursos disponibles en esa suscripción y el porcentaje de la cuota que está usando para cada uno de ellos.

   ![Cuotas y uso](./media/devtest-lab-scale-lab/devtestlab-view-quotas-new.png)

## <a name="requesting-more-resources-in-your-subscription"></a>Solicitud de más recursos en la suscripción
Si alcanza un límite de cuota, se puede aumentar el límite predeterminado de un recurso en una suscripción hasta un máximo, como se describe en [Límites de suscripción y de servicios de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md).

Estos pasos le muestran cómo solicitar un aumento de la cuota a través de [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040).

1. Seleccione **Más servicios**, **Facturación**, y **Uso y cuotas**.
1. En la hoja Uso y cuotas, seleccione el botón **Solicitar aumento**.

   ![Botón Solicitar aumento](./media/devtest-lab-scale-lab/devtestlab-request-increase-new.png)

1. Para completar y enviar la solicitud, rellene la información necesaria en las tres pestañas del formulario **Nueva solicitud de soporte técnico**.

   ![Formulario Solicitar aumento](./media/devtest-lab-scale-lab/devtestlab-support-form-new.png)

[Understanding Azure Limits and Increases](https://azure.microsoft.com/blog/azure-limits-quotas-increase-requests/) (Información sobre los límites y los aumentos de Azure) proporciona más información sobre cómo ponerse en contacto con el soporte técnico de Azure para solicitar un aumento de la cuota.



[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

### <a name="next-steps"></a>Pasos siguientes
* Explore la [galería de plantillas de inicio rápido de Azure Resource Manager de DevTest Labs](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/QuickStartTemplates).
