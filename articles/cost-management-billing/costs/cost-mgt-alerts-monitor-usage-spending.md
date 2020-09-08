---
title: Supervisión del uso y el gasto con alertas de costos
description: En este artículo se describe cómo ayudan las alertas de costes a supervisar el uso y gasto en Azure Cost Management.
author: bandersmsft
ms.author: banders
ms.date: 07/24/2020
ms.topic: how-to
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: adwise
ms.openlocfilehash: aeeb630cf15aedd62c085e2070e08fd223656094
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88683443"
---
# <a name="use-cost-alerts-to-monitor-usage-and-spending"></a>Uso de alertas de costes para supervisar el uso y el gasto

Este artículo le ayuda a entender y usar las alertas de Cost Management para supervisar el uso y el gasto en Azure. Las alertas de costes se generan automáticamente en función de cuándo se usan recursos de Azure. Las alertas muestran todas las alertas de administración de costes y de facturación activas en un solo lugar. Cuando el consumo alcanza un umbral determinado, Cost Management genera las alertas. Hay tres tipos de alertas de costes: alertas de presupuesto, alertas de crédito y alertas de la cuota de gasto del departamento.

## <a name="budget-alerts"></a>Alertas de presupuesto

Las alertas de presupuesto le envían una notificación cuando el gasto, en función del uso o coste, alcanza o supera la cantidad definida en la [condición de alerta del presupuesto](tutorial-acm-create-budgets.md). Los presupuestos de Cost Management se crean mediante Azure Portal o la API de [Azure Consumption](https://docs.microsoft.com/rest/api/consumption).

En Azure Portal, los presupuestos se definen por el coste. Mediante la API de Azure Consumption, los presupuestos se definen por coste o por uso de consumo. Las alertas de presupuesto admiten presupuestos basado en costes o en uso. Las alertas de presupuesto se generan automáticamente cada vez que se cumplen las condiciones de alerta de presupuesto. Puede ver todas las alertas de costes en Azure Portal. Cada vez que se genera una alerta, esta se muestra en las alertas de costes. También se envía un correo electrónico de alerta a las personas de la lista de destinatarios de la alerta del presupuesto.

## <a name="credit-alerts"></a>Alertas de crédito

Las alertas de crédito le envían una notificación cuando se consumen los compromisos monetarios de su crédito de Azure. Los compromisos monetarios son para organizaciones con contratos Enterprise. Las alertas de crédito se generan automáticamente al 90 % y el 100 % de su balance de crédito de Azure. Cada vez que se genera una alerta, esta se refleja en las alertas de costes y en el correo electrónico que se envía a los propietarios de la cuenta.

## <a name="department-spending-quota-alerts"></a>Alertas de cuota de gasto de departamento

Las alertas de cuota de gasto de departamento le informan del momento en que el gasto del departamento alcanza un umbral fijo de la cuota. Las cuotas de gastos se configuran en el portal del contrato Enterprise. Cada vez que se alcanza un umbral, se genera un correo electrónico para los propietarios del departamento y se muestra en las alertas de costes. Por ejemplo, al 50 % o el 75 % de la cuota.

## <a name="supported-alert-features-by-offer-categories"></a>Características de alertas compatibles por categorías de ofertas

La compatibilidad de tipos de alertas depende del tipo de cuenta de Azure que se tiene (oferta de Microsoft). En la tabla siguiente se muestran las características de alerta que varias ofertas de Microsoft admiten. Puede ver la lista completa de ofertas de Microsoft en [Descripción de los datos de Cost Management](understand-cost-mgt-data.md).

| Tipo de alerta | Contrato Enterprise | Contrato de cliente de Microsoft | Web directa/Pago por uso |
|---|---|---|---|
| Presupuesto | ✔ | ✔ | ✔ |
| Créditos | ✔ |✘ | ✘ |
| Cuota de gasto de departamento | ✔ | ✘ | ✘ |



## <a name="view-cost-alerts"></a>Visualización de alertas de costes

Para ver las alertas sobre los costos, abra el ámbito deseado en Azure Portal y seleccione **Presupuestos** en el menú. Use la píldora **Ámbito** para cambiar a un ámbito diferente. Seleccione **Alertas sobre los costos** en el menú. Para más información sobre los ámbitos, consulte [Descripción y uso de ámbitos](understand-work-scopes.md).

![Imagen de ejemplo de alertas que se muestran en Cost Management](./media/cost-mgt-alerts-monitor-usage-spending/budget-alerts-fullscreen.png)

El número total de alertas activas y descartadas aparece en la página de alertas sobre los costes.

Todas las alertas muestran el tipo de alerta. Una alerta de presupuesto muestra la razón por la que se generó y el nombre del presupuesto al que se aplica. Cada alerta muestra la fecha en se generó, su estado y el ámbito (grupo de administración o suscripción) al que se aplica la alerta.

El posible estado incluye **activo** y **descartado**. El estado activo indica que la alerta sigue siendo pertinente. El estado descartado indica que alguien ha marcado la alerta para indicar que ya no es pertinente.

Seleccione una alerta de la lista para ver los detalles. Los detalles de la alerta muestran más información sobre la alerta. Las alertas de presupuesto incluyen un vínculo al presupuesto. Si hay una recomendación disponible para una alerta de presupuesto, también se muestra un vínculo a la recomendación. Las alertas de cuota de gasto de presupuesto, crédito y departamento tienen un vínculo para analizar los costes donde puede explorar los costes del ámbito de la alerta. El ejemplo siguiente muestra el gasto de un departamento con detalles de alerta.

![Imagen de ejemplo que muestra el gasto de un departamento con detalles de alerta](./media/cost-mgt-alerts-monitor-usage-spending/dept-spending-selected-with-credits.png)

Al ver los detalles de una alerta descartada, puede reactivarla si es necesario realizar una acción manual. En la imagen siguiente se muestra un ejemplo:

![Imagen de ejemplo que muestra las opciones de descartar y reactivar](./media/cost-mgt-alerts-monitor-usage-spending/Dismiss-reactivate-options.png)

## <a name="see-also"></a>Consulte también

- Si aún no ha creado un presupuesto ni ha establecido condiciones de alerta para un presupuesto, complete el tutorial [Creación y administración de presupuestos](tutorial-acm-create-budgets.md).
