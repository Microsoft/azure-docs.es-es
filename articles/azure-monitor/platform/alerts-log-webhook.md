---
title: Acciones de webhook para alertas de registro en alertas de Azure
description: En este artículo se describe cómo crear una regla de alertas de registro mediante el área de trabajo de Log Analytics o Application Insights, cómo la alerta envía los datos como un webhook de HTTP y los detalles de las diferentes personalizaciones posibles.
author: yanivlavi
ms.author: yalavi
services: monitoring
ms.topic: conceptual
ms.date: 06/25/2019
ms.subservice: alerts
ms.openlocfilehash: 3311819f021533a28a41daf2c2f08193218fae96
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87075275"
---
# <a name="webhook-actions-for-log-alert-rules"></a>Acciones de webhook para reglas de alertas de registro
Cuando se [crea una alerta de registro en Azure](alerts-log.md), se puede [configurar mediante grupos de acciones](action-groups.md), para así poder realizar una o varias acciones. En este artículo se describen las diferentes acciones de webhook que están disponibles y se muestra cómo configurar un webhook personalizado basado en JSON.

> [!NOTE]
> También se puede usar el [esquema de alertas comunes](https://aka.ms/commonAlertSchemaDocs) para las integraciones de webhook. El esquema de alerta común ofrece la ventaja de tener una carga útil de alerta única y extensible en todos los servicios de alerta de Azure Monitor. Tenga en cuenta que el esquema de alerta común no respeta la opción de JSON personalizada para las alertas de registro. Se aplaza a la carga del esquema de alerta común si se selecciona con independencia de la personalización que haya realizado en el nivel de la regla de alerta. [Obtenga más información sobre las definiciones de esquemas de alertas comunes.](https://aka.ms/commonAlertSchemaDefinitions)

## <a name="webhook-actions"></a>Acciones de webhook

Las acciones de webhook permiten invocar un proceso externo a través de una sola solicitud HTTP POST. El servicio al que se llama debe admitir webhooks y determinar cómo utilizará las cargas que reciba.

Las acciones de webhook requieren las propiedades de la siguiente tabla.

| Propiedad | Descripción |
|:--- |:--- |
| **Dirección URL de Webhook** |La dirección URL del webhook. |
| **Carga de JSON personalizada** |La carga personalizada que se envía con el webhook, cuando se elige esta opción durante la creación de la alerta. Para más información, consulte [Administración de alertas de registro](alerts-log.md).|

> [!NOTE]
> El botón **Ver webhook** y la opción **Incluir la carga personalizada de JSON para webhook** de la alerta del registro muestran una carga de webhook de ejemplo con la personalización proporcionada. No contiene datos reales, pero es representativo del esquema JSON que se usa para las alertas de registro. 

Los webhooks incluyen una dirección URL y una carga con formato JSON que corresponde a los datos enviados al servicio externo. De forma predeterminada, la carga incluye los valores en la tabla siguiente. Puede optar por reemplazar esta carga con una personalizada de su propiedad. En ese caso, puede utilizar las variables de la tabla para cada uno de los parámetros e incluir así sus valores en la carga personalizada.


| Parámetro | Variable | Descripción |
|:--- |:--- |:--- |
| *AlertRuleName* |#alertrulename |Nombre de la regla de alerta. |
| *Gravedad* |#severity |Gravedad establecida en la alerta de registros activada. |
| *AlertThresholdOperator* |ThresholdOperator |Operador de umbral para la regla de alertas, que usa mayor que o menor que. |
| *AlertThresholdValue* |#thresholdvalue |Valor de umbral para la regla de alerta. |
| *LinkToSearchResults* |#linktosearchresults |Vínculo al portal de Analytics que devuelve los registros de la consulta que creó la alerta. |
| *LinkToSearchResultsAPI* |#linktosearchresultsapi |Vínculo a la API de Analytics que devuelve los registros de la consulta que creó la alerta. |
| *LinkToFilteredSearchResultsUI* |#linktofilteredsearchresultsui |Vínculo al portal de Analytics que devuelve los registros de la consulta filtrada por las combinaciones de valores de dimensiones que crearon la alerta. |
| *LinkToFilteredSearchResultsAPI* |#linktofilteredsearchresultsapi |Vínculo a la API de Analytics que devuelve los registros de la consulta filtrada por las combinaciones de valores de dimensiones que crearon la alerta. |
| *ResultCount* |#searchresultcount |Número de registros en los resultados de la búsqueda. |
| *SearchIntervalEndtimeUtc* |#searchintervalendtimeutc |Hora de finalización de la consulta en hora UTC, con formato mm/dd/aaaa HH:mm:ss AM/PM. |
| *SearchIntervalInSeconds* |#searchinterval |Período de tiempo para la regla de alertas, con formato HH:mm:ss. |
| *SearchIntervalStartTimeUtc* |#searchintervalstarttimeutc |Hora de inicio de la consulta en hora UTC, con formato mm/dd/aaaa HH:mm:ss AM/PM. 
| *SearchQuery* |#searchquery |Consulta de búsqueda de registros utilizada por la regla de alerta. |
| *SearchResults* |"IncludeSearchResults": true|Los registros devueltos por la consulta en forma de una tabla JSON, limitados a los primeros 1000 registros. Se agrega "IncludeSearchResults": true en una definición personalizada de webhook de JSON como una propiedad de nivel superior. |
| *Dimensiones* |"IncludeDimensions": true|Combinaciones de valores de dimensiones que desencadenaron esa alerta como una sección JSON. Se agrega "IncludeDimensions": true en una definición personalizada de webhook de JSON como una propiedad de nivel superior. |
| *AlertType*| #alerttype | El tipo de regla de alertas de registro configurada, bien como [Unidades métricas](alerts-unified-log.md#metric-measurement-alert-rules) o como [Número de resultados](alerts-unified-log.md#number-of-results-alert-rules).|
| *WorkspaceID* |#workspaceid |Identificador del área de trabajo de Log Analytics. |
| *Identificador de la aplicación* |#applicationid |Identificador de la aplicación Application Insights. |
| *Subscription ID* |#subscriptionid |Identificador de la suscripción a Azure usada. 

> [!NOTE]
> Los vínculos proporcionados pasan parámetros como *SearchQuery*, *Search Interval StartTime* y *Search Interval End time* de la dirección URL a Azure Portal o a la API.

Por ejemplo, podría especificar la siguiente carga personalizada que incluye un único parámetro denominado *text*. El servicio al que llama este webhook espera este parámetro.

```json

    {
        "text":"#alertrulename fired with #searchresultcount over threshold of #thresholdvalue."
    }
```
Esta carga de ejemplo se resuelve en algo similar a lo siguiente al enviarse al webhook:

```json
    {
        "text":"My Alert Rule fired with 18 records over threshold of 10 ."
    }
```
Como todas las variables de un webhook personalizado deben especificarse dentro de un contenedor JSON, como "#searchinterval", el webhook resultante también tendrá datos de variable dentro de contenedores, como "00:05:00".

Para incluir resultados de búsqueda en una carga personalizada, asegúrese de que el parámetro **IncludeSearchResults** se establece como una propiedad de nivel superior en la carga de JSON. 

## <a name="sample-payloads"></a>Cargas de ejemplo
En esta sección se muestran cargas de ejemplo para los webhooks de alertas de registro. Estas cargas incluyen ejemplos en los que la carga es estándar y otros en los que es personalizada.

### <a name="standard-webhook-for-log-alerts"></a>Webhook estándar para alertas de registro 
En ambos ejemplos se incluye una carga ficticia con solo dos columnas y dos filas.

#### <a name="log-alert-for-log-analytics"></a>Alerta de registro para Log Analytics
La carga de ejemplo siguiente corresponde a una acción de webhook estándar *sin una opción JSON personalizada*, que se usa para las alertas basadas en Log Analytics:

```json
{
    "SubscriptionId": "12345a-1234b-123c-123d-12345678e",
    "AlertRuleName": "AcmeRule",
    "SearchQuery": "Perf | where ObjectName == \"Processor\" and CounterName == \"% Processor Time\" | summarize AggregatedValue = avg(CounterValue) by bin(TimeGenerated, 5m), Computer",
    "SearchIntervalStartTimeUtc": "2018-03-26T08:10:40Z",
    "SearchIntervalEndtimeUtc": "2018-03-26T09:10:40Z",
    "AlertThresholdOperator": "Greater Than",
    "AlertThresholdValue": 0,
    "ResultCount": 2,
    "SearchIntervalInSeconds": 3600,
    "LinkToSearchResults": "https://portal.azure.com/#Analyticsblade/search/index?_timeInterval.intervalEnd=2018-03-26T09%3a10%3a40.0000000Z&_timeInterval.intervalDuration=3600&q=Usage",
    "LinkToFilteredSearchResultsUI": "https://portal.azure.com/#Analyticsblade/search/index?_timeInterval.intervalEnd=2018-03-26T09%3a10%3a40.0000000Z&_timeInterval.intervalDuration=3600&q=Usage",
    "LinkToSearchResultsAPI": "https://api.loganalytics.io/v1/workspaces/workspaceID/query?query=Heartbeat&timespan=2020-05-07T18%3a11%3a51.0000000Z%2f2020-05-07T18%3a16%3a51.0000000Z",
    "LinkToFilteredSearchResultsAPI": "https://api.loganalytics.io/v1/workspaces/workspaceID/query?query=Heartbeat&timespan=2020-05-07T18%3a11%3a51.0000000Z%2f2020-05-07T18%3a16%3a51.0000000Z",
    "Description": "log alert rule",
    "Severity": "Warning",
    "AffectedConfigurationItems": [
        "INC-Gen2Alert"
    ],
    "Dimensions": [
        {
            "name": "Computer",
            "value": "INC-Gen2Alert"
        }
    ],
    "SearchResult": {
        "tables": [
            {
                "name": "PrimaryResult",
                "columns": [
                    {
                        "name": "$table",
                        "type": "string"
                    },
                    {
                        "name": "Computer",
                        "type": "string"
                    },
                    {
                        "name": "TimeGenerated",
                        "type": "datetime"
                    }
                ],
                "rows": [
                    [
                        "Fabrikam",
                        "33446677a",
                        "2018-02-02T15:03:12.18Z"
                    ],
                    [
                        "Contoso",
                        "33445566b",
                        "2018-02-02T15:16:53.932Z"
                    ]
                ]
            }
        ]
    },
    "WorkspaceId": "12345a-1234b-123c-123d-12345678e",
    "AlertType": "Metric measurement"
}
 ```

> [!NOTE]
> El valor del campo "Severity" puede cambiar si ha [cambiado la preferencia de API](alerts-log-api-switch.md) para las alertas de registro en Log Analytics.


#### <a name="log-alert-for-application-insights"></a>Alerta de registro para Application Insights
La carga de ejemplo siguiente corresponde a un webhook estándar *sin una opción JSON personalizada*, que se usa para alertas de registro basadas en Application Insights:
    
```json
{
    "schemaId": "Microsoft.Insights/LogAlert",
    "data": {
        "SubscriptionId": "12345a-1234b-123c-123d-12345678e",
        "AlertRuleName": "AcmeRule",
        "SearchQuery": "requests | where resultCode == \"500\" | summarize AggregatedValue = Count by bin(Timestamp, 5m), IP",
        "SearchIntervalStartTimeUtc": "2018-03-26T08:10:40Z",
        "SearchIntervalEndtimeUtc": "2018-03-26T09:10:40Z",
        "AlertThresholdOperator": "Greater Than",
        "AlertThresholdValue": 0,
        "ResultCount": 2,
        "SearchIntervalInSeconds": 3600,
        "LinkToSearchResults": "https://portal.azure.com/AnalyticsBlade/subscriptions/12345a-1234b-123c-123d-12345678e/?query=search+*+&timeInterval.intervalEnd=2018-03-26T09%3a10%3a40.0000000Z&_timeInterval.intervalDuration=3600&q=Usage",
        "LinkToFilteredSearchResultsUI": "https://portal.azure.com/AnalyticsBlade/subscriptions/12345a-1234b-123c-123d-12345678e/?query=search+*+&timeInterval.intervalEnd=2018-03-26T09%3a10%3a40.0000000Z&_timeInterval.intervalDuration=3600&q=Usage",
        "LinkToSearchResultsAPI": "https://api.applicationinsights.io/v1/apps/0MyAppId0/metrics/requests/count",
        "LinkToFilteredSearchResultsAPI": "https://api.applicationinsights.io/v1/apps/0MyAppId0/metrics/requests/count",
        "Description": null,
        "Severity": "3",
        "Dimensions": [
            {
                "name": "IP",
                "value": "1.1.1.1"
            }
        ],
        "SearchResult": {
            "tables": [
                {
                    "name": "PrimaryResult",
                    "columns": [
                        {
                            "name": "$table",
                            "type": "string"
                        },
                        {
                            "name": "Id",
                            "type": "string"
                        },
                        {
                            "name": "Timestamp",
                            "type": "datetime"
                        }
                    ],
                    "rows": [
                        [
                            "Fabrikam",
                            "33446677a",
                            "2018-02-02T15:03:12.18Z"
                        ],
                        [
                            "Contoso",
                            "33445566b",
                            "2018-02-02T15:16:53.932Z"
                        ]
                    ]
                }
            ]
        },
        "ApplicationId": "123123f0-01d3-12ab-123f-abc1ab01c0a1",
        "AlertType": "Metric measurement"
    }
}
```

#### <a name="log-alert-with-custom-json-payload"></a>Alerta de registro con carga de JSON personalizada
Por ejemplo, para crear una carga personalizada que solo incluya el nombre de la alerta y los resultados de la búsqueda, puede usar lo siguiente: 

```json
    {
       "alertname":"#alertrulename",
       "IncludeSearchResults":true
    }
```

A continuación se muestra una carga de ejemplo que corresponde a una acción de webhook personalizada para cualquier alerta de registro:
    
```json
    {
    "alertname":"AcmeRule","IncludeSearchResults":true,
    "SearchResults":
        {
        "tables":[
                    {"name":"PrimaryResult","columns":
                        [
                        {"name":"$table","type":"string"},
                        {"name":"Id","type":"string"},
                        {"name":"TimeGenerated","type":"datetime"}
                        ],
                    "rows":
                        [
                            ["Fabrikam","33446677a","2018-02-02T15:03:12.18Z"],
                            ["Contoso","33445566b","2018-02-02T15:16:53.932Z"]
                        ]
                    }
                ]
        }
    }
```


## <a name="next-steps"></a>Pasos siguientes
- Más información sobre las [alertas de registro en las alertas de Azure](alerts-unified-log.md).
- Aprenda a [administrar alertas de registro en Azure](alerts-log.md).
- Cree y administre [grupos de acciones en Azure](action-groups.md).
- Más información sobre [Application Insights](../log-query/log-query-overview.md).
- Obtenga más información sobre las [consultas de registro](../log-query/log-query-overview.md). 
