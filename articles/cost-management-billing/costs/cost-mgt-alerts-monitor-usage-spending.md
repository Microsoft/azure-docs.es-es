---
title: Supervisión del uso y el gasto con alertas de costos
description: En este artículo se describe cómo ayudan las alertas de costes a supervisar el uso y gasto en Azure Cost Management.
author: bandersmsft
ms.author: banders
ms.date: 09/03/2020
ms.topic: how-to
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: adwise
ms.openlocfilehash: 72e9fd0d5a178897cf84b2babe4c02f7ef920841
ms.sourcegitcommit: 03662d76a816e98cfc85462cbe9705f6890ed638
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90531345"
---
# <a name="use-cost-alerts-to-monitor-usage-and-spending"></a>Uso de alertas de costes para supervisar el uso y el gasto

Este artículo le ayuda a entender y usar las alertas de Cost Management para supervisar el uso y el gasto en Azure. Las alertas de costes se generan automáticamente en función de cuándo se usan recursos de Azure. Las alertas muestran todas las alertas de administración de costes y de facturación activas en un solo lugar. Cuando el consumo alcanza un umbral determinado, Cost Management genera las alertas. Hay tres tipos de alertas de costes: alertas de presupuesto, alertas de crédito y alertas de la cuota de gasto del departamento.

## <a name="budget-alerts"></a>Alertas de presupuesto

Las alertas de presupuesto le envían una notificación cuando el gasto, en función del uso o coste, alcanza o supera la cantidad definida en la [condición de alerta del presupuesto](tutorial-acm-create-budgets.md). Los presupuestos de Cost Management se crean mediante Azure Portal o la API de [Azure Consumption](https://docs.microsoft.com/rest/api/consumption).

En Azure Portal, los presupuestos se definen por el coste. Mediante la API de Azure Consumption, los presupuestos se definen por coste o por uso de consumo. Las alertas de presupuesto admiten presupuestos basado en costes o en uso. Las alertas de presupuesto se generan automáticamente cada vez que se cumplen las condiciones de alerta de presupuesto. Puede ver todas las alertas de costes en Azure Portal. Cada vez que se genera una alerta, esta se muestra en las alertas de costes. También se envía un correo electrónico de alerta a las personas de la lista de destinatarios de la alerta del presupuesto.

Puede usar Budgets API para enviar alertas por correo electrónico en otro idioma. Para más información, consulte [Configuraciones regionales admitidas para los correos electrónicos de alertas de presupuesto](manage-automation.md#supported-locales-for-budget-alert-emails).

## <a name="credit-alerts"></a>Alertas de crédito

Las alertas de crédito le avisan cuando se consumen los compromisos monetarios de crédito de Azure. Los compromisos monetarios son para organizaciones con contratos Enterprise. Las alertas de crédito se generan de forma automática al 90 % y al 100 % del saldo de crédito de Azure. Cada vez que se genera una alerta, se refleja en las alertas sobre los costos y en el correo electrónico que se envía a los propietarios de la cuenta.

## <a name="department-spending-quota-alerts"></a>Alertas de cuota de gasto de departamento

Las alertas de cuota de gasto de departamento notifican cuándo el gasto del departamento alcanza un umbral fijo de la cuota. Las cuotas de gasto se configuran en el portal de EA. Cada vez que se alcanza un umbral, se genera un correo electrónico para los propietarios del departamento y se muestra en las alertas sobre los costos. Por ejemplo, al 50 % o el 75 % de la cuota.

## <a name="supported-alert-features-by-offer-categories"></a>Características de alerta admitidas por categorías de oferta

La compatibilidad con los tipos de alerta depende del tipo de cuenta de Azure que tenga (oferta de Microsoft). En la tabla siguiente se muestran las características de alerta que varias ofertas de Microsoft admiten. Puede ver la lista completa de ofertas de Microsoft en [Descripción de los datos de Cost Management](understand-cost-mgt-data.md).

| Tipo de alerta | Contrato Enterprise | Contrato de cliente de Microsoft | Web directa/Pago por uso |
|---|---|---|---|
| Presupuesto | ✔ | ✔ | ✔ |
| Crédito | ✔ |✘ | ✘ |
| Cuota de gasto de departamento | ✔ | ✘ | ✘ |



## <a name="view-cost-alerts"></a>Visualización de alertas sobre los costos

Para ver las alertas sobre los costos, abra el ámbito que quiera en Azure Portal y seleccione **Presupuestos** en el menú. Use el intervalo **Ámbito** para cambiar a otro ámbito. Seleccione **Alertas sobre los costos** en el menú. Para más información sobre los ámbitos, consulte [Descripción y uso de ámbitos](understand-work-scopes.md).

![Imagen de ejemplo de alertas que se muestran en Cost Management](./media/cost-mgt-alerts-monitor-usage-spending/budget-alerts-fullscreen.png)

El número total de alertas activas y descartadas aparece en la página de alertas sobre los costos.

Todas las alertas muestran el tipo de alerta. Una alerta de presupuesto muestra la razón por la que se generó y el nombre del presupuesto al que se aplica. Cada alerta muestra la fecha en se generó, su estado y el ámbito (grupo de administración o suscripción) al que se aplica la alerta.

El posible estado incluye **activo** y **descartado**. El estado activo indica que la alerta sigue siendo relevante. El estado descartado indica que alguien ha marcado la alerta para establecerla como no relevante.

Seleccione una alerta en la lista para ver sus detalles. En los detalles de la alerta se muestra más información sobre la alerta. Las alertas de presupuesto incluyen un vínculo al presupuesto. Si hay una recomendación disponible para una alerta de presupuesto, también se muestra un vínculo a la recomendación. Las alertas de presupuesto, crédito y cuota de gasto del departamento tienen un vínculo para analizar en el análisis de costos donde puede explorar los costos para el ámbito de la alerta. El ejemplo siguiente muestra el gasto de un departamento con detalles de alerta.

![Imagen de ejemplo que muestra el gasto de un departamento con detalles de alerta](./media/cost-mgt-alerts-monitor-usage-spending/dept-spending-selected-with-credits.png)

Al ver los detalles de una alerta descartada, puede reactivarla si es necesario realizar una acción manual. En la imagen siguiente se muestra un ejemplo:

![Imagen de ejemplo que muestra las opciones de descartar y reactivar](./media/cost-mgt-alerts-monitor-usage-spending/Dismiss-reactivate-options.png)

## <a name="see-also"></a>Vea también

- Si aún no ha creado un presupuesto ni ha establecido condiciones de alerta para un presupuesto, complete el tutorial [Creación y administración de presupuestos](tutorial-acm-create-budgets.md).
