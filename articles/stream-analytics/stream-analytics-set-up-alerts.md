---
title: Configuración de alertas de supervisión en Azure Stream Analytics
description: En este artículo se describe cómo usar Azure Portal para configurar la supervisión y las alertas para los trabajos de Azure Stream Analytics.
author: jseb225
ms.author: sidram
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 06/21/2019
ms.openlocfilehash: de8b69cbe3117a3ec248cee4808b676b39c56658
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87324801"
---
# <a name="set-up-alerts-for-azure-stream-analytics-jobs"></a>Configuración de alertas en Azure Stream Analytics

Es importante supervisar el trabajo de Azure Stream Analytics para asegurarse de que se está ejecutando continuamente sin problemas. En este artículo se describe cómo configurar alertas para escenarios comunes que se deben supervisar. 

Puede definir reglas en las métricas desde los datos de los registros de operaciones hasta el portal, así como [mediante programación](https://code.msdn.microsoft.com/windowsazure/Receive-Email-Notifications-199e2c9a).

## <a name="set-up-alerts-in-the-azure-portal"></a>Configuración de alertas en Azure Portal
### <a name="get-alerted-when-a-job-stops-unexpectedly"></a>Obtención de una alerta cuando un trabajo se detiene de manera inesperada

El ejemplo siguiente muestra cómo configurar alertas para cuando el trabajo entra en un estado de error. Esta alerta se recomienda para todos los trabajos.

1. En Azure Portal, abra el trabajo de Stream Analytics para el que desea crear una alerta.

2. En la página **Trabajo**, vaya a la sección **Supervisión**.  

3. Seleccione **Métricas** y luego **Nueva regla de alertas**.

   ![Configuración de alertas de Stream Analytics en Azure Portal](./media/stream-analytics-set-up-alerts/stream-analytics-set-up-alerts.png)  

4. El nombre del trabajo de Stream Analytics debe aparecer automáticamente en **RECURSO**. Haga clic en **Agregar condición** y seleccione **All Administrative operations** (Todas las operaciones administrativas) en **Configurar lógica de señal**.

   ![Selección del nombre de la señal para la alerta de Stream Analytics](./media/stream-analytics-set-up-alerts/stream-analytics-condition-signal.png)  

5. En **Configurar lógica de señal**, cambie **Nivel de evento** a **Todo** y cambie **Estado** a **Con error**. Deje la opción **Evento iniciado por** en blanco y seleccione **Listo**.

   ![Configuración de señal lógica para alerta de Stream Analytics](./media/stream-analytics-set-up-alerts/stream-analytics-configure-signal-logic.png) 

6. Seleccione un grupo de acciones existente o cree uno nuevo. En este ejemplo, un nuevo grupo de acciones llamado **TIDashboardGroupActions** se ha creado con la acción **Emails** que envía un correo electrónico a los usuarios con el rol de **propietario** de Azure Resource Manager.

   ![Configuración de una alerta para un trabajo Azure Stream Analytics](./media/stream-analytics-set-up-alerts/stream-analytics-add-group-email-action.png)

7. En **RECURSO**, **CONDICIÓN** y **GRUPOS DE ACCIONES** debería haber una entrada. Tenga en cuenta que para que se activen las alertas, es preciso que se den las condiciones definidas. Por ejemplo, puede medir el valor promedio de una métrica en los últimos 15 minutos, cada 5 minutos.

   ![Creación de una regla de alerta de Stream Analytics](./media/stream-analytics-set-up-alerts/stream-analytics-create-alert-rule-2.png)

   Agregue un **nombre de regla de alerta**, una **descripción**y su **grupo de recursos** en **DETALLES DE LA ALERTA** y haga clic en **Crear regla de alertas** para crear la regla para el trabajo de Stream Analytics.

   ![Creación de una regla de alerta de Stream Analytics](./media/stream-analytics-set-up-alerts/stream-analytics-create-alert-rule.png)
   
## <a name="scenarios-to-monitor"></a>Escenarios que supervisar

Se recomiendan las siguientes alertas para supervisar el rendimiento del trabajo de Stream Analytics. Estas métricas deben evaluarse cada minuto durante los últimos 5 minutos.

|Métrica|Condición|Agregación de tiempo|Umbral|Acciones correctivas|
|-|-|-|-|-|
|Porcentaje de uso de SU|Mayor que|Máxima|80|Existen diversos factores que incrementan el porcentaje de uso de unidades de streaming. Puede escalar con paralelización de consultas o aumentar el número de unidades de streaming. Para más información, consulte [Aprovechamiento de la paralelización de consultas en Azure Stream Analytics](stream-analytics-parallelization.md).|
|Errores de tiempo de ejecución|Mayor que|Total|0|Examine los registros de recursos o actividad y realice los cambios correspondientes en las entradas, la consulta o las salidas.|
|Retraso de la marca de agua|Mayor que|Máxima|Cuando el valor medio de esta métrica en los últimos 15 minutos es mayor que la tolerancia de llegada tardía (en segundos). Si no ha modificado la tolerancia de llegada tardía, el valor predeterminado se establece en 5 segundos.|Pruebe a aumentar el número de SU o a paralelizar su consulta. Para más información sobre las SU, consulte [Descripción y ajuste de las unidades de streaming](stream-analytics-streaming-unit-consumption.md#how-many-sus-are-required-for-a-job). Para más información sobre la paralelización de su consulta, consulte [Aprovechamiento de la paralelización de consultas en Azure Stream Analytics](stream-analytics-parallelization.md).|
|Errores de deserialización de entrada|Mayor que|Total|0|Examine los registros de recursos o actividad y realice los cambios correspondientes en la entrada. Para más información sobre los registros de recursos, consulte [Solución de problemas de Azure Stream Analytics mediante registros de recursos](stream-analytics-job-diagnostic-logs.md).|

## <a name="next-steps"></a>Pasos siguientes

* [Escalado de trabajos de Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Referencia del lenguaje de consulta de Azure Stream Analytics](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference)

