---
title: Solución de problemas de alertas de registro en Azure Monitor | Microsoft Docs
description: Problemas comunes, errores y resoluciones para las reglas de alertas de registro en Azure.
author: yanivlavi
ms.author: yalavi
ms.topic: conceptual
ms.subservice: alerts
ms.date: 10/29/2018
ms.openlocfilehash: d61e052b10b7255cac37531f889324075d596f3c
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87828462"
---
# <a name="troubleshoot-log-alerts-in-azure-monitor"></a>Solución de problemas de alertas de registro en Azure Monitor  

En este artículo se muestra cómo solucionar problemas habituales con las alertas de registro en Azure Monitor. Además, se proporcionan soluciones a los problemas comunes sobre la funcionalidad y la configuración de las alertas de registro.

El término *alertas de registro* describe las reglas que se desencadenan en función de una consulta de registros en un [área de trabajo de Azure Log Analytics](../log-query/get-started-portal.md) o en [Azure Application Insights](../log-query/log-query-overview.md). Obtenga más información acerca de la funcionalidad, la terminología y los tipos en [Alertas de registro en Azure Monitor](./alerts-unified-log.md).

> [!NOTE]
> En este artículo no se tienen en cuenta los casos en que Azure Portal muestra una regla de alertas desencadenada y no se realiza una notificación a través de un grupo de acciones asociado. Para estos casos, consulte los detalles de [Crear y administrar grupos de acciones en Azure Portal](./action-groups.md).

## <a name="log-alert-didnt-fire"></a>No se activó la alerta de registro

Estas son algunas causas habituales por las que el estado de una [regla de alerta de registro en Azure Monitor](./alerts-log.md) configurada no se muestra [como *desencadenada* cuando se espera](./alerts-managing-alert-states.md).

### <a name="data-ingestion-time-for-logs"></a>Tiempo de ingesta de datos para registros

La alerta de registro ejecuta periódicamente la consulta basada en [Log Analytics](../log-query/get-started-portal.md) o [Application Insights](../log-query/log-query-overview.md). Dado que Azure Monitor procesa muchos terabytes de datos de miles de clientes desde diferentes orígenes en todo el mundo, el servicio es susceptible de sufrir retrasos variables. Para obtener más información, consulte [Tiempo de la ingesta de datos de registro en Azure Monitor](./data-ingestion-time.md).

Para mitigar los retrasos, el sistema espera y vuelve a intentar la consulta de alerta varias veces si detecta que aún no se han ingerido los datos necesarios. El sistema tiene un tiempo de espera establecido que aumenta exponencialmente. La alerta de registro solo se desencadena una vez que los datos están disponibles, por lo que el retraso puede deberse a una ingesta de datos de registro lenta.

### <a name="incorrect-time-period-configured"></a>Período de tiempo configurado incorrectamente

Como se describe en el artículo sobre la [terminología para alertas de registro](./alerts-unified-log.md#log-search-alert-rule---definition-and-types), el período de tiempo que se indica en la configuración especifica el intervalo de tiempo para la consulta. La consulta devuelve solo los registros que se crearon dentro de este intervalo.

El período restringe los datos capturados para una consulta de registros, con el fin de evitar abusos, y evita todos los comandos de tiempo (como **ago**) utilizados en la consulta de registros. Por ejemplo, si el período se establece en 60 minutos, y la consulta se ejecuta a la 1:15 p. m., los registros creados entre las 12:15 p. m. y la 1:15 p. m. son los únicos que se usan para la consulta de registros. Si la consulta de registros usa un comando de tiempo como **ago (1d)** , seguirá utilizando solo los datos entre las 12:15 p. m. y la 1:15 p. m., ya que el período está establecido en ese intervalo.

Compruebe que el período de tiempo en la configuración coincida con la consulta. En el ejemplo anterior, si la consulta de registros utiliza **ago (1d)** con el marcador verde, el período de tiempo debe establecerse en 24 horas o 1440 minutos (como se indica en rojo). Esta configuración garantiza que la consulta se ejecute según lo previsto.

![Período de tiempo](media/alert-log-troubleshoot/LogAlertTimePeriod.png)

### <a name="suppress-alerts-option-is-set"></a>Se ha establecido la opción Suprimir alertas

Como se describe en el paso 8 del artículo sobre cómo [crear una regla de alerta de registro en Azure Portal](./alerts-log.md#create-a-log-alert-rule-with-the-azure-portal), las alertas de registro proporcionan una opción **Suprimir alertas** destinada a suprimir las acciones de desencadenamiento y notificación durante un período de tiempo configurado. Como resultado, puede parecerle que una alerta no se desencadena. De hecho, se ha desencadenado, pero se ha suprimido.  

![Suprimir alertas](media/alert-log-troubleshoot/LogAlertSuppress.png)

### <a name="metric-measurement-alert-rule-is-incorrect"></a>Regla de alertas de unidades métricas

*Las alertas de registro de unidades métricas* constituyen un subtipo de alertas de registro que tienen funcionalidades especiales y una sintaxis de consulta de alerta restringida. Una regla para una alerta de registro de unidades métricas requiere que el resultado de la consulta sea una serie temporal métrica. Es decir, el resultado es una tabla con períodos de tiempo distintos, de igual tamaño, junto con los valores agregados correspondientes.

Puede optar por tener variables adicionales en la tabla junto con **AggregatedValue**. Estas variables se pueden usar para ordenar la tabla.

Por ejemplo, suponga que se ha configurado una regla para una alerta de registro de unidades métricas como:

- Consulta de `search *| summarize AggregatedValue = count() by $table, bin(timestamp, 1h)`  
- Período de tiempo de 6 horas
- Umbral de 50
- Lógica de alerta de tres infracciones consecutivas
- **Aggregate Upon** establecido en **$table**

Dado que el comando incluye **summarize ... by** (resumir … por) y se han proporcionado dos variables (**timestamp** y **$table**), el sistema elegirá **$table** para **Aggregate Upon**. El sistema ordena la tabla de resultados por el campo **$table**, como se muestra en la captura de pantalla siguiente. A continuación, examina las distintas instancias de **AggregatedValue** para cada tipo de tabla (como **availabilityResults**) para ver si se han producido tres o más infracciones consecutivas.

![Ejecución de consultas de unidades métricas con varios valores](media/alert-log-troubleshoot/LogMMQuery.png)

Dado que **Aggregate Upon** está definido en **$table**, los datos se ordenan en una columna **$table** (se indica en rojo). A continuación, se agrupan y buscan los tipos de campo **Aggregate Upon**.

Por ejemplo, para **$table**, los valores de **availabilityResults** se considerarán un trazado o una entidad (se indica en naranja). En el trazado o la entidad de este valor, el servicio de alerta busca tres infracciones consecutivas (se indican en verde). Las infracciones desencadenan una alerta para el valor de la tabla **availabilityResults**.

De forma similar, si se producen tres infracciones consecutivas para cualquier otro valor de **$table**, se desencadena otra notificación de alerta para el mismo fin. El servicio de alertas ordena automáticamente los valores en un trazado o una entidad (se indica en naranja) por tiempo.

Ahora supongamos que la regla de la alerta de registro de unidades métricas se modificó y que la consulta era `search *| summarize AggregatedValue = count() by bin(timestamp, 1h)`. El resto de la configuración sigue siendo la misma que antes, incluida la lógica de alerta de las tres infracciones consecutivas. En este caso, la opción de **Aggregate Upon** será **timestamp** de forma predeterminada. Solo se proporciona un valor en la consulta de **summarize... by** (es decir, **timestamp**). Al igual que en el ejemplo anterior, el resultado al final de la ejecución será el que se muestra en la siguiente ilustración.

   ![Ejecución de consultas de unidades métricas con un valor singular](media/alert-log-troubleshoot/LogMMtimestamp.png)

Dado que **Aggregate Upon** está definido en **timestamp**, los datos se ordenan en una columna **timestamp** (se indica en rojo). A continuación, se realiza la agrupación por **timestamp**. Por ejemplo, los valores de `2018-10-17T06:00:00Z` se considerarán un trazado o una entidad (se indica en naranja). En el trazado o la entidad de este valor, el servicio de alerta no encontrará ninguna infracción consecutiva (ya que cada valor de **timestamp** tiene una sola entrada). Por tanto, la alerta nunca se desencadenará. En este caso, el usuario tiene que:

- Agregar una variable ficticia o una variable existente (como **$table**) para ordenar correctamente con el campo **Aggregate Upon** configurado.
- Volver a configurar una regla de alerta para usar, en su lugar, la lógica de alerta basada en **infracción total**.

## <a name="log-alert-fired-unnecessarily"></a>Alerta de registro activada innecesariamente

Una [regla de alertas de registro en Azure Monitor](./alerts-log.md) configurada podría desencadenarse inesperadamente al verla en [Alertas de Azure](./alerts-managing-alert-states.md). En las secciones siguientes se describen algunos motivos comunes.

### <a name="alert-triggered-by-partial-data"></a>Alerta desencadenada por datos parciales

Log Analytics y Application Insights están sujetos al procesamiento y a retrasos de ingesta. Al ejecutar una consulta de alerta de registro, es posible que encuentre que no hay datos disponibles o que solo algunos datos estén disponibles. Para obtener más información, consulte [Tiempo de la ingesta de datos de registro en Azure Monitor](./data-ingestion-time.md).

En función de cómo configure la regla de alertas, puede producirse una activación incorrecta si no hay ningún dato en los registros, o los datos que hay son parciales, en el momento de la ejecución de la alerta. En tales casos, se recomienda cambiar la configuración o la consulta de alerta.

Por ejemplo, si la regla de alertas de registro se configura para desencadenarse cuando el número de resultados de una consulta de Analytics sea menor que 5, la alerta se desencadenará cuando no haya datos (cero registros) o los resultados sean parciales (un registro). Sin embargo, después del retraso de la ingesta de datos, la misma consulta con todos los datos podría proporcionar un resultado de 10 registros.

### <a name="alert-query-output-is-misunderstood"></a>Salida de la consulta de alerta no comprendida

La lógica de las alertas de registros se proporcionan en una consulta de Analytics. La consulta de Analytics puede usar varias funciones matemáticas y macrodatos. El servicio de alertas ejecuta la consulta a los intervalos especificados con datos de un período especificado. Este servicio realiza pequeños cambios en la consulta en función del tipo de alerta. Este cambio puede verse en la sección **Consulta que se va a ejecutar** en la pantalla **Configurar lógica de señal**:

![Consulta que se va a ejecutar](media/alert-log-troubleshoot/LogAlertPreview.png)

El contenido del cuadro **Consulta que se va a ejecutar** es lo que ejecuta el servicio de alertas de registro. Si quiere saber cuál será el resultado de la consulta de alerta antes de crear la alerta,puede ejecutar la consulta indicada y el intervalo de tiempo a través del [portal de Analytics](../log-query/log-query-overview.md) o [Analytics API](/rest/api/loganalytics/).

## <a name="log-alert-was-disabled"></a>La alerta de registro se deshabilitó

En las siguientes secciones se enumeran algunos de los motivos por los que Azure Monitor podría deshabilitar la [regla de alertas de registro](./alerts-log.md).

### <a name="resource-where-the-alert-was-created-no-longer-exists"></a>El recurso donde se creó la alerta ya no existe

Las reglas de alertas de registro creadas en Azure Monitor están destinadas a un recurso específico, como un área de trabajo de Azure Log Analytics, una aplicación de Azure Application Insights y un recurso de Azure. A continuación, el servicio de alertas de registro ejecutará una consulta de Analytics proporcionada en la regla para el destino especificado. No obstante, tras la creación de la regla, los usuarios tienden a eliminarla de Azure, o a moverla en Azure (el destino de la regla de alertas de registro). Dado que el destino de la regla de alertas ya no es válido, se produce un error en la ejecución de la regla.

En estos casos, Azure Monitor se deshabilita la alerta de registro y se asegura que no se le facture innecesariamente si la regla no se puede ejecutar continuamente durante un período cuantificable (por ejemplo, una semana). Puede averiguar la hora exacta en que Azure Monitor deshabilitó la alerta de registro a través del [registro de actividad de Azure](../../azure-resource-manager/management/view-activity-logs.md). En el registro de actividad de Azure, se agrega un evento cuando Azure Monitor deshabilita la regla de alertas de registro.

El siguiente evento de ejemplo del registro de actividad de Azure es para una regla de alertas que se ha deshabilitado debido a un error continuo.

```json
{
    "caller": "Microsoft.Insights/ScheduledQueryRules",
    "channels": "Operation",
    "claims": {
        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/spn": "Microsoft.Insights/ScheduledQueryRules"
    },
    "correlationId": "abcdefg-4d12-1234-4256-21233554aff",
    "description": "Alert: test-bad-alerts is disabled by the System due to : Alert has been failing consistently with the same exception for the past week",
    "eventDataId": "f123e07-bf45-1234-4565-123a123455b",
    "eventName": {
        "value": "",
        "localizedValue": ""
    },
    "category": {
        "value": "Administrative",
        "localizedValue": "Administrative"
    },
    "eventTimestamp": "2019-03-22T04:18:22.8569543Z",
    "id": "/SUBSCRIPTIONS/<subscriptionId>/RESOURCEGROUPS/<ResourceGroup>/PROVIDERS/MICROSOFT.INSIGHTS/SCHEDULEDQUERYRULES/TEST-BAD-ALERTS",
    "level": "Informational",
    "operationId": "",
    "operationName": {
        "value": "Microsoft.Insights/ScheduledQueryRules/disable/action",
        "localizedValue": "Microsoft.Insights/ScheduledQueryRules/disable/action"
    },
    "resourceGroupName": "<Resource Group>",
    "resourceProviderName": {
        "value": "MICROSOFT.INSIGHTS",
        "localizedValue": "Microsoft Insights"
    },
    "resourceType": {
        "value": "MICROSOFT.INSIGHTS/scheduledqueryrules",
        "localizedValue": "MICROSOFT.INSIGHTS/scheduledqueryrules"
    },
    "resourceId": "/SUBSCRIPTIONS/<subscriptionId>/RESOURCEGROUPS/<ResourceGroup>/PROVIDERS/MICROSOFT.INSIGHTS/SCHEDULEDQUERYRULES/TEST-BAD-ALERTS",
    "status": {
        "value": "Succeeded",
        "localizedValue": "Succeeded"
    },
    "subStatus": {
        "value": "",
        "localizedValue": ""
    },
    "submissionTimestamp": "2019-03-22T04:18:22.8569543Z",
    "subscriptionId": "<SubscriptionId>",
    "properties": {
        "resourceId": "/SUBSCRIPTIONS/<subscriptionId>/RESOURCEGROUPS/<ResourceGroup>/PROVIDERS/MICROSOFT.INSIGHTS/SCHEDULEDQUERYRULES/TEST-BAD-ALERTS",
        "subscriptionId": "<SubscriptionId>",
        "resourceGroup": "<ResourceGroup>",
        "eventDataId": "12e12345-12dd-1234-8e3e-12345b7a1234",
        "eventTimeStamp": "03/22/2019 04:18:22",
        "issueStartTime": "03/22/2019 04:18:22",
        "operationName": "Microsoft.Insights/ScheduledQueryRules/disable/action",
        "status": "Succeeded",
        "reason": "Alert has been failing consistently with the same exception for the past week"
    },
    "relatedEvents": []
}
```

### <a name="query-used-in-a-log-alert-is-not-valid"></a>La consulta que se usa en una alerta de registro no es válida

Cada regla de alertas de registro creada en Azure Monitor como parte de su configuración debe especificar una consulta de Analytics que el servicio de alerta ejecutará periódicamente. Es posible que la consulta de Analytics tuviera la sintaxis correcta en el momento de crear o actualizar la regla. Pero a veces, durante un período de tiempo, la consulta proporcionada en la regla de alertas de registro puede desarrollar problemas de sintaxis y provocar un error en la ejecución de la regla. Algunas razones comunes por las que una consulta de Analytics proporcionada en una regla de alertas de registro puede desarrollar errores son:

- La consulta está escrita para [ejecutarse en varios recursos](../log-query/cross-workspace-query.md). Uno o varios de los recursos especificados ya no existen.
- La [alerta de registro del tipo de unidades métricas](./alerts-unified-log.md#metric-measurement-alert-rules) configurada tiene una consulta de alerta que no cumple con las normas de sintaxis.
- No ha habido ningún flujo de datos para la plataforma de Analytics. El [ejecución de la consulta produce un error](https://dev.loganalytics.io/documentation/Using-the-API/Errors) porque no hay ningún dato para la consulta proporcionada.
- Los cambios en el [lenguaje de consulta](/azure/kusto/query/) incluyen un formato revisado para comandos y funciones. Por lo tanto, la consulta proporcionada anteriormente en una regla de alertas ya no es válida.

[Azure Advisor](../../advisor/advisor-overview.md) le advierte acerca de este comportamiento. Se agregó una recomendación para la regla de alertas de registro específica en Azure Advisor, bajo la categoría de alta disponibilidad con un impacto medio y una descripción similar a "Reparar la regla de alertas de registro para garantizar la supervisión".

> [!NOTE]
> Si una consulta de alerta de la regla de alertas de registro no se rectifica después de que Azure Advisor proporcione una recomendación durante siete días, Azure Monitor deshabilitará la alerta de registro y se asegurará de que no le facturen innecesariamente cuando la regla no pueda ejecutarse continuamente durante un período cuantificable (siete días). Para averiguar la hora exacta en que Azure Monitor deshabilitó la regla de alertas de registro, puede buscar un evento en el [registro de actividad de Azure](../../azure-resource-manager/management/view-activity-logs.md).

## <a name="alert-rule-quota-was-reached"></a>Se alcanzó la cuota de la regla de alerta

El número de reglas de alertas de búsqueda de registros por suscripción y recurso está sujeto a los límites de cuota descritos [en este artículo](../service-limits.md).

### <a name="recommended-steps"></a>Pasos recomendados
    
Si ha alcanzado el límite de cuota, los siguientes pasos pueden ayudar a resolver el problema.

1. Intente eliminar o deshabilitar las reglas de alertas de búsqueda de registros que ya no se usan.
2. Si necesita aumentar el límite de cuota, abra una solicitud de soporte técnico y proporcione la siguiente información:

    - Los identificadores de suscripción para los que tiene que aumentar los límites de cuota.
    - Motivo de aumento de cuota
    - Tipo de recurso para el que se va a aumentar la cuota: **Log Analytics**, **Application Insights**, etc.
    - Límite de cuota solicitado


### <a name="to-check-the-current-usage-of-new-log-alert-rules"></a>Para comprobar el uso actual de las nuevas reglas de alertas de registro
    
#### <a name="from-the-azure-portal"></a>Desde Azure Portal

1. Abra la pantalla *Alertas* y haga clic en *Administrar reglas de alertas.*
2. Filtre la suscripción correspondiente mediante el control desplegable *Suscripción*.
3. Asegúrese de NO filtrar por un grupo de recursos, tipo de recurso o recurso específico.
4. En el control desplegable *Tipo de señal*, seleccione "Búsqueda de registros".
5. Compruebe que el control desplegable *Estado* está establecido en "Habilitado".
6. El número total de reglas de alertas de búsqueda de registros se mostrará encima de la lista de reglas.

#### <a name="from-api"></a>Desde la API

- PowerShell: [Get-AzScheduledQueryRule](/powershell/module/az.monitor/get-azscheduledqueryrule?view=azps-3.7.0)
- API REST: [Lista por suscripción](/rest/api/monitor/scheduledqueryrules/listbysubscription)

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre las [alertas de registro en Azure](./alerts-unified-log.md).
- Más información sobre [Application Insights](../log-query/log-query-overview.md).
- Obtenga más información sobre las [consultas de registro](../log-query/log-query-overview.md).
