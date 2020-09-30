---
title: Inicio rápido de la API de REST Metrics Monitor
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: metrics-advisor
ms.topic: include
ms.date: 09/10/2020
ms.author: aahi
ms.openlocfilehash: 82eec57b30e177f75a3ac689dc096dfea54c6717
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945363"
---
## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* Una vez que tenga la suscripción de Azure, <a href="https://go.microsoft.com/fwlink/?linkid=2142156"  title="cree un recurso de Metrics Advisor "  target="_blank">create a Metrics Advisor resource <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para implementar la instancia de Metrics Advisor.  
* La versión actual de [cURL](https://curl.haxx.se/). En este artículo se usan varios modificadores de línea de comandos, que se indican en la [documentación de cURL](https://curl.haxx.se/docs/manpage.html).
    * En los siguientes ejemplos de BASH se usa el carácter de continuación de línea `\`. Si la consola o el terminal usan un carácter de continuación de línea diferente, use este carácter.

> [!TIP]
> * Puede encontrar un ejemplo de Python que usa la API de REST en [GitHub](https://github.com/Azure-Samples/cognitive-services-rest-api-samples/).
> * Puede que se tarde entre 10 y 30 minutos en implementar el recurso de Metrics Advisor. Haga clic en **Ir al recurso** una vez que se implemente correctamente.
> * Después de la implementación, puede empezar a usar la instancia de Metrics Advisor con el portal web y la API de REST. Puede encontrar ambas direcciones URL en el recurso que ha creado.
> * Necesitará la clave y el punto de conexión del recurso que cree para empezar a usar el servicio, que se puede encontrar en **Claves y punto de conexión**, en el recurso. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.

## <a name="add-a-data-feed-from-a-sample-or-data-source"></a>Incorporación de una fuente de distribución de datos desde un origen de datos o un ejemplo

Para empezar a supervisar los datos de la serie temporal, tiene que agregar una fuente de distribución de datos. Para ello, debe proporcionar un esquema de datos según el tipo de origen de datos y los parámetros. Guarde el cuerpo de la solicitud JSON siguiente en un archivo denominado *body.json* y ejecute el comando de cURL.

```json
{
        "dataSourceType": "SqlServer",
        "dataFeedName": "test_data_feed_00000001",
        "dataFeedDescription": "",
        "dataSourceParameter": {
            "connectionString": "Server=ad-sample.database.windows.net,1433;Initial Catalog=ad-sample;Persist Security Info=False;User ID=adreadonly;Password=Readonly@2020;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;",
            "query": "select * from adsample3 where Timestamp = @StartTime"
        },
        "granularityName": "Daily",
        "granularityAmount": 0,
        "metrics": [
            {
                "metricName": "revenue",
                "metricDisplayName": "revenue",
                "metricDescription": ""
            },
            {
                "metricName": "cost",
                "metricDisplayName": "cost",
                "metricDescription": ""
            }
        ],
        "dimension": [
            {
                "dimensionName": "city",
                "dimensionDisplayName": "city"
            },
            {
                "dimensionName": "category",
                "dimensionDisplayName": "category"
            }
        ],
        "timestampColumn": "timestamp",
        "dataStartFrom": "2020-06-01T00:00:00.000Z",
        "startOffsetInSeconds": 0,
        "maxConcurrency": -1,
        "minRetryIntervalInSeconds": -1,
        "stopRetryAfterInSeconds": -1,
        "needRollup": "NoRollup",
        "fillMissingPointType": "SmartFilling",
        "fillMissingPointValue": 0,
        "viewMode": "Private",
        "admins": [
            "xxx@microsoft.com"
        ],
        "viewers": [
        ],
        "actionLinkTemplate": ""
}
```

El comando de cURL se ejecuta desde un shell de BASH. Edite este comando con sus propios valores de nombre de recurso, clave de recurso y JSON.

```bash
curl -i https://REPLACE-WITH-YOUR-ENDPOINT/anomalydetector-ee/v1.0/datafeeds \
-X POST \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
-H "x-api-key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
-H "Content-Type:application/json" \
-d @body.json
```

#### <a name="example-response"></a>Respuesta de ejemplo

```
HTTP/1.1 201 Created
Content-Length: 0
Location: https://gualala-beta-0617.cognitiveservices.azure.com/anomalydetector-ee/v1.0/datafeeds/b5921405-8001-42b2-8746-004ddeeb780d
x-envoy-upstream-service-time: 564
apim-request-id: 4e4fe70b-d663-4fb7-a804-b9dc14ba02a3
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
x-content-type-options: nosniff
Date: Thu, 03 Sep 2020 18:29:27 GMT
```
En la respuesta anterior, el encabezado **Location** (Ubicación) es la dirección URL de la fuente de distribución de datos que ha creado y contiene el valor de **dataFeedID** (Identificador de la fuente de distribución de datos). 

Con la dirección URL anterior, puede consultar información detallada de la fuente de distribución de datos que creó en el paso anterior. (Usaremos **metricID** en la información de la fuente de distribución de datos en los pasos siguientes)

```bash
curl https://REPLACE-WITH-YOUR-ENDPOINT/anomalydetector-ee/v1.0/datafeeds/REPLACE-WITH-YOUR-DATA-FEED-ID \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
-H "x-api-key: REPLACE-WITH-YOUR-RESOURCE-KEY"
```

#### <a name="example-response"></a>Respuesta de ejemplo

```
{
   "dataFeedId":"90919c03-be13-4efa-86e5-aa9dc72764ce",
   "dataFeedName":"test_data_feed_00000007",
   "metrics":[
      {
         "metricId":"8d03e541-a56d-4c28-8d9c-09ce91c6d95f",
         "metricName":"cost",
         "metricDisplayName":"cost",
         "metricDescription":""
      },
      {
         "metricId":"82bbc63d-3739-4d57-b190-accb69721b6a",
         "metricName":"revenue",
         "metricDisplayName":"revenue",
         "metricDescription":""
      }
   ],
   "dimension":[
      {
         "dimensionName":"category",
         "dimensionDisplayName":"category"
      },
      {
         "dimensionName":"city",
         "dimensionDisplayName":"city"
      }
   ],
   "dataStartFrom":"2020-06-01T00:00:00Z",
   "dataSourceType":"SqlServer",
   "timestampColumn":"timestamp",
   "startOffsetInSeconds":0,
   "maxQueryPerMinute":30.0,
   "granularityName":"Daily",
   "granularityAmount":null,
   "allUpIdentification":null,
   "needRollup":"NoRollup",
   "fillMissingPointType":"SmartFilling",
   "fillMissingPointValue":0.0,
   "rollUpMethod":"None",
   "rollUpColumns":[
      
   ],
   "dataFeedDescription":"",
   "stopRetryAfterInSeconds":-1,
   "minRetryIntervalInSeconds":-1,
   "maxConcurrency":-1,
   "viewMode":"Private",
   "admins":[
      "zhli2@microsoft.com"
   ],
   "viewers":[
      
   ],
   "creator":"bowgong@microsoft.com",
   "status":"Active",
   "createdTime":"2020-09-08T08:39:28Z",
   "isAdmin":true,
   "actionLinkTemplate":"",
   "dataSourceParameter":{
      "connectionString":"Server=ad-sample.database.windows.net,1433;Initial Catalog=ad-sample;Persist Security Info=False;User ID=adreadonly;Password=Readonly@2020;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;",
      "query":"select * from adsample3 where Timestamp = @StartTime"
   }
}
```



## <a name="check-ingestion-status"></a>Comprobación del estado de la ingesta

Después de agregar la fuente de distribución de datos, si desea comprobar el progreso de un trabajo de ingesta, puede comprobar su estado. Guarde el cuerpo de la solicitud JSON siguiente en un archivo denominado *body.json* y ejecute el comando de cURL.

```json
{
  "startTime": "2020-01-01T00:00:00.0000000+00:00",
  "endTime": "2020-01-04T00:00:00.0000000+00:00"
}
```

El comando de cURL se ejecuta desde un shell de BASH. Edite este comando con su propio nombre de recurso, clave de recurso y valores y tamaño de JSON.


```bash
curl https://REPLACE-WITH-YOUR-ENDPOINT/anomalydetector-ee/v1.0/datafeeds/REPLACE-WITH-YOUR-DATA-FEED-ID/ingestionStatus/query \
-X POST \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
-H "x-api-key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
-H "Content-Type:application/json" \
-d @body.json
```


#### <a name="example-response"></a>Respuesta de ejemplo

```json
{
  "@nextLink": "https://demo.example.com/datafeeds/01234567-8901-2345-6789-012345678901/ingestionStatus/query?$skip=3&$top=1",
  "value": [
    {
      "timestamp": "2020-09-03T00:00:00.0000000+00:00",
      "status": "Running",
      "message": ""
    },
    {
      "timestamp": "2020-09-02T00:00:00.0000000+00:00",
      "status": "Succeeded",
      "message": ""
    },
    {
      "timestamp": "2020-09-01T00:00:00.0000000+00:00",
      "status": "Failed",
      "message": "No valid record pulled from source for current timestamp 2020-01-01T00:00:00Z"
    }
  ]
}
```

##  <a name="configure-anomaly-detection-configuration"></a>Configuración de la detección de anomalías

Aunque a cada métrica se le aplica automáticamente una configuración predeterminada, se pueden ajustar los modos de detección utilizados en los datos. Guarde el cuerpo de la solicitud JSON siguiente en un archivo denominado *body.json* y ejecute el comando de cURL.

```json
{
        "name": "test_detection_config0000000001",
        "description": "string",
        "metricId": "8d03e541-a56d-4c28-8d9c-09ce91c6d95f",
        "wholeMetricConfiguration": {
            "smartDetectionCondition": {
                "sensitivity": 100,
                "anomalyDetectorDirection": "Both",
                "suppressCondition": {
                    "minNumber": 1,
                    "minRatio": 1
                }
            }
        },
        "dimensionGroupOverrideConfigurations": [
        ],
        "seriesOverrideConfigurations": [
        ]
}
```

El comando de cURL se ejecuta desde un shell de BASH. Edite este comando con su propio nombre de recurso, clave de recurso y valores y tamaño de JSON.

```bash
curl -i https://REPLACE-WITH-YOUR-ENDPOINT/anomalydetector-ee/v1.0/enrichment/anomalyDetection/configurations \
-X POST \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
-H "x-api-key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
-H "Content-Type:application/json" \
-d @body.json
```

#### <a name="example-response"></a>Respuesta de ejemplo

```
HTTP/1.1 201 Created
Content-Length: 0
Location: https://gualala-beta-0617.cognitiveservices.azure.com/anomalydetector-ee/v1.0/enrichment/anomalyDetection/configurations/6a977d61-f0f5-488a-a162-2feb4643ae09
x-request-id: 17752fcc-9085-46d5-ad37-c4e9e9ba6a5a
x-envoy-upstream-service-time: 253
apim-request-id: 17752fcc-9085-46d5-ad37-c4e9e9ba6a5a
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
x-content-type-options: nosniff
Date: Tue, 08 Sep 2020 09:50:38 GMT
```

En el encabezado **Location** (Ubicación) anterior, se encuentra la dirección URL del nuevo recurso creado (configuración de detección). 

Con la dirección URL anterior en el encabezado **Location** (Ubicación), puede consultar la configuración de detección que ha creado (usaremos **anomalyDetectionConfigurationId** (Identificador de configuración de detección de anomalías) en el contenido de la respuesta en los pasos siguientes).

```
curl https://REPLACE-WITH-YOUR-ENDPOINT/anomalydetector-ee/v1.0/enrichment/anomalyDetection/configurations/REPLACE-WITH-YOUR-DETECTION-CONFIGURATION-ID \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
-H "x-api-key: REPLACE-WITH-YOUR-RESOURCE-KEY"
```

#### <a name="example-response"></a>Respuesta de ejemplo

```json
{
   "anomalyDetectionConfigurationId":"6a977d61-f0f5-488a-a162-2feb4643ae09",
   "name":"test_detection_config0000000001",
   "description":"string",
   "metricId":"8d03e541-a56d-4c28-8d9c-09ce91c6d95f",
   "wholeMetricConfiguration":{
      "smartDetectionCondition":{
         "sensitivity":100.0,
         "anomalyDetectorDirection":"Both",
         "suppressCondition":{
            "minNumber":1,
            "minRatio":1.0
         }
      }
   },
   "dimensionGroupOverrideConfigurations":[
      
   ],
   "seriesOverrideConfigurations":[
      
   ]
}
```



### <a name="configure-alert-configuration"></a>Configuración de las alertas

Antes de configurar la alerta, debe crear un enlace que se utilizará para notificarla. Hay dos maneras de recibir notificaciones si se desencadena una alerta: webhook y correo electrónico. Mientras crea el tipo de enlace, puede especificar cualquiera de ellas en la configuración del enlace.

Guarde el cuerpo de la solicitud JSON siguiente en un archivo denominado *body.json* y ejecute el comando de cURL.

```json
{
        "hookType": "Webhook",
        "hookName": "test_web_hook000001",
        "description": "",
        "externalLink": "",
        "hookParameter": {
            "endpoint": "https://www.xxx.com/aaa",
            "username": "",
            "password": ""
        }
}
```

El comando de cURL se ejecuta desde un shell de BASH. Edite este comando con su propio nombre de recurso, clave de recurso y valores y tamaño de JSON.

```bash
curl -i https://REPLACE-WITH-YOUR-ENDPOINT/anomalydetector-ee/v1.0/hooks \
-X POST \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
-H "x-api-key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
-H "Content-Type:application/json" \
-d @body.json
```

#### <a name="example-response"></a>Respuesta de ejemplo

```
HTTP/1.1 201 Created
Content-Length: 0
Location: https://gualala-beta-0617.cognitiveservices.azure.com/anomalydetector-ee/v1.0/hooks/34d677bd-0875-4760-8bf6-24d48abde7c3
x-request-id: 7b6cc1a6-02cb-405b-bee3-174fdae0a7d2
x-envoy-upstream-service-time: 1640
apim-request-id: 7b6cc1a6-02cb-405b-bee3-174fdae0a7d2
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
x-content-type-options: nosniff
Date: Tue, 08 Sep 2020 10:37:59 GMT
```

Con la dirección URL anterior en el encabezado **Ubicación**, puede consultar el webhook que ha creado.

```
curl https://REPLACE-WITH-YOUR-ENDPOINT/anomalydetector-ee/v1.0/hooks/REPLACE-WITH-YOUR-HOOK-ID \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
-H "x-api-key: REPLACE-WITH-YOUR-RESOURCE-KEY"
```

#### <a name="example-response"></a>Respuesta de ejemplo

```json
{
   "hookId":"34d677bd-0875-4760-8bf6-24d48abde7c3",
   "hookName":"test_web_hook000001",
   "hookType":"Webhook",
   "externalLink":"",
   "description":"",
   "admins":[
      "bowgong@microsoft.com"
   ],
   "hookParameter":{
      "endpoint":"https://www.xxx.com/aaa",
      "username":"",
      "password":"",
      "headers":{
         

      },
      "certificateKey":"",
      "certificatePassword":""

   }
}
```

Mediante la configuración de alertas, puede especificar la condición de detección que se puede usar para desencadenar una alerta. Guarde el cuerpo de la solicitud JSON siguiente en un archivo denominado *body.json* y ejecute el comando de cURL.

```json
{
        "name": "test_alert_config00000001",
        "description": "",
        "crossMetricsOperator": "AND",
        "hookIds": [
           "34d677bd-0875-4760-8bf6-24d48abde7c3" 
        ],
        "metricAlertingConfigurations": [
            {
                "anomalyDetectionConfigurationId": "6a977d61-f0f5-488a-a162-2feb4643ae09",
                "anomalyScopeType": "All",
                "severityFilter": {
                    "minAlertSeverity": "Low",
                    "maxAlertSeverity": "High"
                },
                "snoozeFilter": {
                    "autoSnooze": 0,
                    "snoozeScope": "Metric",
                    "onlyForSuccessive": true
                },
            }
        ]
}
```

El comando de cURL se ejecuta desde un shell de BASH. Edite este comando con su propio nombre de recurso, clave de recurso y valores y tamaño de JSON.

```bash
curl -i https://REPLACE-WITH-YOUR-ENDPOINT/anomalydetector-ee/v1.0/alert/anomaly/configurations \
-X POST \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
-H "x-api-key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
-H "Content-Type:application/json" \
-d @body.json
```

#### <a name="example-response"></a>Respuesta de ejemplo

```
HTTP/1.1 201 Created
Content-Length: 0
Location: https://gualala-beta-0617.cognitiveservices.azure.com/anomalydetector-ee/v1.0/alert/anomaly/configurations/40004c91-6996-47c0-b8c8-fd20a8f4f0ab
x-request-id: 17752fcc-9085-46d5-ad37-c4e9e9ba6a5a
x-envoy-upstream-service-time: 253
apim-request-id: 17752fcc-9085-46d5-ad37-c4e9e9ba6a5a
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
x-content-type-options: nosniff
Date: Tue, 08 Sep 2020 09:50:38 GMT
```

En el encabezado **Location** (Ubicación) anterior, se encuentra la dirección URL del nuevo recurso creado (configuración de detección). 

Con la dirección URL anterior en el encabezado **Location**, puede consultar la configuración de alertas que ha creado. (Usaremos **anomalyAlertingConfigurationId** en la configuración de alertas, en los pasos siguientes)

```
curl https://REPLACE-WITH-YOUR-ENDPOINT/anomalydetector-ee/v1.0/alert/anomaly/configurations/REPLACE-WITH-YOUR-ANOMALY-ALERTING-CONFIGURATION-ID \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
-H "x-api-key: REPLACE-WITH-YOUR-RESOURCE-KEY"
```

#### <a name="example-response"></a>Respuesta de ejemplo

```json
{
   "anomalyAlertingConfigurationId":"40004c91-6996-47c0-b8c8-fd20a8f4f0ab",
   "name":"test_alert_config00000001",
   "description":"",
   "hookIds":[
      "34d677bd-0875-4760-8bf6-24d48abde7c3"
   ],
   "metricAlertingConfigurations":[
      {
         "anomalyDetectionConfigurationId":"6a977d61-f0f5-488a-a162-2feb4643ae09",
         "anomalyScopeType":"All",
         "negationOperation":false,
         "severityFilter":{
            "minAlertSeverity":"Low",
            "maxAlertSeverity":"High"
         },
         "snoozeFilter":{
            "autoSnooze":0,
            "snoozeScope":"Metric",
            "onlyForSuccessive":true
         }
      }
   ]
}
```

### <a name="query-anomaly-detection-results"></a>Resultados de la detección de anomalías de las consultas

Hay diferentes maneras de obtener los resultados de la detección. Por ejemplo, puede consultar el resultado de la detección de forma periódica con la configuración de la detección que ha creado o bien podría recibir una notificación mediante la alerta y, a continuación, usar dicha alerta para consultar las anomalías correspondientes.

En el ejemplo siguiente, se muestra cómo consultar la alerta y cómo usarla para consultar las anomalías relacionadas con ella.

#### <a name="query-alert"></a>Alerta de consulta

Puede usar la configuración de alertas creada en el paso anterior para consultar la alerta.

```
curl https://REPLACE-WITH-YOUR-ENDPOINT/anomalydetector-ee/v1.0/anomalydetector-ee/v1.0/alert/anomaly/configurations/REPLACE-WITH-YOUR-ANOMALY-ALERTING-CONFIGURATION-ID/alerts/query \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
-H "x-api-key: REPLACE-WITH-YOUR-RESOURCE-KEY"
```

#### <a name="example-response"></a>Respuesta de ejemplo

```json
{
   "value":[
      {
         "alertId":"17465dcc000",
         "timestamp":"2020-09-07T00:00:00Z",
         "createdTime":"2020-09-08T19:12:46.532Z",
         "modifiedTime":"2020-09-08T19:12:46.588Z"
      }
   ],
   "@nextLink":null
}
```

En la respuesta anterior, se produjo una alerta. Con este **alertID**, podemos consultar todas las anomalías relacionadas que produjeron esta alerta.

(Otra manera de obtener una alerta es configurar el webhook y la alerta de recepción pasiva una vez que se encuentre).

#### <a name="query-anomalies-using-alertid"></a>Consulta de anomalías mediante alertID

```
curl https://REPLACE-WITH-YOUR-ENDPOINT/anomalydetector-ee/v1.0/alert/anomaly/configurations/REPLACE-WITH-YOUR-ANOMALY-ALERTING-CONFIGURATION-ID/alerts/REPLACE-WITH-YOUR-ALERTID/anomalies \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
-H "x-api-key: REPLACE-WITH-YOUR-RESOURCE-KEY"
```

#### <a name="example-response"></a>Respuesta de ejemplo

```json
{
   "value":[
      {
         "metricId":"8d03e541-a56d-4c28-8d9c-09ce91c6d95f",
         "anomalyDetectionConfigurationId":"6a977d61-f0f5-488a-a162-2feb4643ae09",
         "timestamp":"2020-09-07T00:00:00Z",
         "createdTime":"2020-09-08T19:12:46.566Z",
         "modifiedTime":"2020-09-08T19:12:46.566Z",
         "dimension":{
            "city":"Amphibian",
            "category":"Caucasian Fir"
         },
         "property":{
            "anomalySeverity":"High",
            "anomalyStatus":"Active"
         }
      },
      {
         "metricId":"8d03e541-a56d-4c28-8d9c-09ce91c6d95f",
         "anomalyDetectionConfigurationId":"6a977d61-f0f5-488a-a162-2feb4643ae09",
         "timestamp":"2020-09-07T00:00:00Z",
         "createdTime":"2020-09-08T19:12:46.566Z",
         "modifiedTime":"2020-09-08T19:12:46.566Z",
         "dimension":{
            "city":"Animals by number of neurons",
            "category":"Crack willow"
         },
         "property":{
            "anomalySeverity":"High",
            "anomalyStatus":"Active"
         }
      },
      {
         "metricId":"8d03e541-a56d-4c28-8d9c-09ce91c6d95f",
         "anomalyDetectionConfigurationId":"6a977d61-f0f5-488a-a162-2feb4643ae09",
         "timestamp":"2020-09-07T00:00:00Z",
         "createdTime":"2020-09-08T19:12:46.566Z",
         "modifiedTime":"2020-09-08T19:12:46.566Z",
         "dimension":{
            "city":"Amphibian",
            "category":"Crack willow"
         },
         "property":{
            "anomalySeverity":"High",
            "anomalyStatus":"Active"
         }
      },
      {
         "metricId":"8d03e541-a56d-4c28-8d9c-09ce91c6d95f",
         "anomalyDetectionConfigurationId":"6a977d61-f0f5-488a-a162-2feb4643ae09",
         "timestamp":"2020-09-07T00:00:00Z",
         "createdTime":"2020-09-08T19:12:46.566Z",
         "modifiedTime":"2020-09-08T19:12:46.566Z",
         "dimension":{
            "city":"Animals by number of neurons",
            "category":"Common Lime"
         },
         "property":{
            "anomalySeverity":"High",
            "anomalyStatus":"Active"
         }
      },
      {
         "metricId":"8d03e541-a56d-4c28-8d9c-09ce91c6d95f",
         "anomalyDetectionConfigurationId":"6a977d61-f0f5-488a-a162-2feb4643ae09",
         "timestamp":"2020-09-07T00:00:00Z",
         "createdTime":"2020-09-08T19:12:46.566Z",
         "modifiedTime":"2020-09-08T19:12:46.566Z",
         "dimension":{
            "city":"Chickadee",
            "category":"Common Lime"
         },
         "property":{
            "anomalySeverity":"High",
            "anomalyStatus":"Active"
         }
      },
      {
         "metricId":"8d03e541-a56d-4c28-8d9c-09ce91c6d95f",
         "anomalyDetectionConfigurationId":"6a977d61-f0f5-488a-a162-2feb4643ae09",
         "timestamp":"2020-09-07T00:00:00Z",
         "createdTime":"2020-09-08T19:12:46.566Z",
         "modifiedTime":"2020-09-08T19:12:46.566Z",
         "dimension":{
            "city":"Amphibian",
            "category":"Common Lime"
         },
         "property":{
            "anomalySeverity":"High",
            "anomalyStatus":"Active"
         }
      },
      {
         "metricId":"8d03e541-a56d-4c28-8d9c-09ce91c6d95f",
         "anomalyDetectionConfigurationId":"6a977d61-f0f5-488a-a162-2feb4643ae09",
         "timestamp":"2020-09-07T00:00:00Z",
         "createdTime":"2020-09-08T19:12:46.566Z",
         "modifiedTime":"2020-09-08T19:12:46.566Z",
         "dimension":{
            "city":"Chickadee",
            "category":"Crack willow"
         },
         "property":{
            "anomalySeverity":"High",
            "anomalyStatus":"Active"
         }
      }
   ],
   "@nextLink":null
}
```
