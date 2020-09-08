---
title: Administración de presupuestos de Cloudyn en Azure
description: Este artículo le ayuda a crear presupuestos e iniciar su administración en Cloudyn rápidamente.
author: bandersmsft
ms.author: banders
ms.date: 03/12/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: cloudyn
ms.reviewer: vitavor
ms.custom: seodec18
ROBOTS: NOINDEX
ms.openlocfilehash: b7a26e7d94bbfea77bfc8121923377a086aef037
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88690874"
---
# <a name="manage-azure-budgets-with-cloudyn"></a>Administración de presupuestos de Azure con Cloudyn

La configuración de presupuestos y de alertas basadas en presupuestos ayuda a mejorar la gobernanza de la nube y la responsabilidad. Este artículo le ayuda a crear presupuestos e iniciar su administración en Cloudyn rápidamente.

Cuando tenga una cuenta Enterprise o de MSP, puede utilizar la estructura de entidades de costo jerárquicas para asignar cuotas de presupuesto mensuales a diferentes unidades de negocio, departamentos o cualquier otra entidad de costo. Cuando tiene una cuenta Premium, puede utilizar la funcionalidad de administración de presupuestos, que se aplica luego a todos sus gastos de la nube. Todos los presupuestos se asignan manualmente.

En función de los presupuestos asignados, puede establecer alertas de umbral basadas en el porcentaje del presupuesto que se consume y definir la gravedad de cada umbral.

Los informes de presupuesto muestran el presupuesto asignado. Los usuarios pueden ver si sus gastos superan, no llegan o son iguales a su consumo a lo largo del tiempo. Cuando selecciona **Show/Hide Fields** (Mostrar/Ocultar campos) en la parte superior de un informe de presupuesto, puede ver el costo, el presupuesto, el costo acumulado o el presupuesto total.

Azure Cost Management ofrece una funcionalidad similar a Cloudyn. Azure Cost Management es una solución de administración de costos de Azure nativa. Le ayuda a analizar costos, crear y administrar presupuestos, exportar datos y revisar y actuar sobre recomendaciones de optimización para ahorrar dinero. Para más información acerca de los presupuestos en Cost Management, consulte [Creación y administración de presupuestos](../costs/tutorial-acm-create-budgets.md).

[!INCLUDE [cloudyn-note](../../../includes/cloudyn-note.md)]

## <a name="create-budgets"></a>Creación de presupuestos

Cuando crea un presupuesto, lo establece para su año fiscal y se aplica a una entidad específica.

Para crear un presupuesto y asignarlo a una entidad:

1. Desplácese a **Costos** &gt; **Cost Management** &gt; **Presupuesto**.
2. En la página Budget Management (Administración del presupuesto), en **Entidades**, seleccione la entidad en la que desea crear el presupuesto.
3. En el año del presupuesto, seleccione el año en el que desea crear el presupuesto.
4. Para cada mes, establezca un valor de presupuesto. Cuando haya terminado, haga clic en **Guardar**.
En este ejemplo, el presupuesto mensual para junio de 2018 está establecido en 135.000 $. El presupuesto total para el año es de 1.615.000,00 $.
![Página para la creación de un presupuesto cuando se establece un presupuesto para cada mes](./media/manage-budgets/set-budget.png)


Para importar un archivo para el presupuesto anual:

1. En **Acciones**, seleccione **Exportar** para exportar una plantilla CSV vacía y utilizarla como base para el presupuesto.
2. Complete el archivo CSV con las entradas del presupuesto y guárdelo localmente.
3. En **Acciones**, seleccione **Importar**.
4. Seleccione el archivo guardado y haga clic en **Aceptar**.

Para exportar el presupuesto completado como archivo CSV, en **Acciones**, seleccione **Exportar** para descargar el archivo.

## <a name="view-budget-in-reports"></a>Visualización del presupuesto en informes

Una vez completado, el presupuesto se muestra en la mayoría de los informes de costo en **Costos**&gt;**Análisis de costos** y en el informe de costo frente a presupuesto a lo largo del tiempo. También puede programar informes basados en umbrales de presupuesto mediante **Acciones**.

Este es un ejemplo del informe de análisis de costos. Muestra el presupuesto total y el costo por carga de trabajo y tipos de uso desde el inicio del año.

![Ejemplo de informe de análisis de costos con presupuesto](./media/manage-budgets/cost-analysis-budget-example.png)

En este ejemplo, se supone que la fecha actual es el 22 de junio. El costo para junio de 2018 es de 71.611,28 $ en comparación con el presupuesto mensual de 135.000 $. El costo es mucho más bajo que el presupuesto mensual porque todavía quedan ocho días de gastos antes del final del mes.

Otra forma de ver el informe es mirar el costo acumulado frente al presupuesto. Para ver los costos acumulados, en **Show/Hide Fields** (Mostrar/Ocultar campos), seleccione **Accumulated Cost** (Costo acumulado) y **Total Budget** (Presupuesto total). Este es un ejemplo que muestra el costo acumulado desde el inicio del año.

![Se muestra un ejemplo de costo acumulado y presupuesto total en el informe de comparación entre el costo y el presupuesto a lo largo del tiempo](./media/manage-budgets/accumulated-budget.png)

En algún momento del futuro, es posible que su costo acumulado supere el presupuesto. Puede verlo con mayor claridad si cambia la vista de gráfico al tipo de _líneas_.

![El presupuesto se muestra en un gráfico de líneas en el informe Costo por meses](./media/manage-budgets/budget-line.png)

## <a name="create-budget-alerts-for-a-filter"></a>Creación de alertas de presupuesto para un filtro

En el ejemplo anterior, puede ver que el costo acumulado ha alcanzado el presupuesto. Puede crear alertas de presupuesto automáticas para que le notifiquen cuando los gastos alcancen o superen el presupuesto. Básicamente, la alerta es un informe programado con un umbral. Las métricas de umbral de alertas de presupuesto incluyen:

- Costo restante frente a presupuesto: para especificar un umbral de valor de moneda
- Porcentaje de costo frente a presupuesto: para especificar un umbral de valor de porcentaje

Veamos un ejemplo.

En el informe de costo frente a presupuesto a lo largo del tiempo, haga clic en **Acciones** y, a continuación, seleccione **Schedule report** (Programar informe). En la pestaña de umbral, seleccione una métrica de umbral. Por ejemplo, **Cost percentage vs budget** (Porcentaje de costo frente a presupuesto). Seleccione un tipo de alerta y escriba un valor de porcentaje del presupuesto. Si desea que se le notifique solo una vez, seleccione **Number of consecutive alerts** (Número de alertas consecutivas) y luego el tipo _1_. Haga clic en **Save**(Guardar).

![Creación de una alerta de presupuesto en el cuadro Guardar o programar este informe](./media/manage-budgets/budget-alert.png)

## <a name="next-steps"></a>Pasos siguientes

- Si aún no ha completado el primer tutorial de Cloudyn, puede consultarlo en [Revisión del uso y los costos](tutorial-review-usage.md).
- Más información sobre los [informes disponibles en Cloudyn](use-reports.md).
