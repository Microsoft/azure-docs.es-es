---
title: 'Análisis sobre la base de conocimiento: QnA Maker'
titleSuffix: Azure Cognitive Services
description: QnA Maker almacena todos los registros de chat y otra telemetría, si ha habilitado Application Insights durante la creación del servicio QnA Maker. Ejecute las consultas de ejemplo para obtener los registros de chat de Application Insights.
services: cognitive-services
manager: nitinme
displayName: chat history, history, chat logs, logs
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 11/05/2019
ms.openlocfilehash: 00b7b88aa4ce0cab2a2379756e40054f27fc633b
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87131657"
---
# <a name="get-analytics-on-your-knowledge-base"></a>Obtener análisis en la base de conocimiento

QnA Maker almacena todos los registros de chat y otra telemetría, si ha habilitado Application Insights durante la [creación del servicio QnA Maker](./set-up-qnamaker-service-azure.md). Ejecute las consultas de ejemplo para obtener los registros de chat de Application Insights.

1. Vaya al recurso de Application Insights.

    ![Selección del recurso de Application Insights](../media/qnamaker-how-to-analytics-kb/resources-created.png)

2. Selecciona **Log (Analytics)** (Registro [análisis]). Se abre una nueva ventana en la que puede consultar la telemetría de QnA Maker.

3. Pegue la consulta siguiente y ejecútela.

    ```kusto
    requests
    | where url endswith "generateAnswer"
    | project timestamp, id, url, resultCode, duration, performanceBucket
    | parse kind = regex url with *"(?i)knowledgebases/"KbId"/generateAnswer"
    | join kind= inner (
    traces | extend id = operation_ParentId
    ) on id
    | extend question = tostring(customDimensions['Question'])
    | extend answer = tostring(customDimensions['Answer'])
    | extend score = tostring(customDimensions['Score'])
    | project timestamp, resultCode, duration, id, question, answer, score, performanceBucket,KbId
    ```

    Seleccione **Ejecutar** para ejecutar la consulta.

    [![Ejecute la consulta para determinar las preguntas, respuestas y puntuaciones de los usuarios](../media/qnamaker-how-to-analytics-kb/run-query.png)](../media/qnamaker-how-to-analytics-kb/run-query.png#lightbox)

## <a name="run-queries-for-other-analytics-on-your-qna-maker-knowledge-base"></a>Ejecución de consultas de otros análisis en la base de conocimiento de QnA Maker

### <a name="total-90-day-traffic"></a>Tráfico total de 90 días

```kusto
//Total Traffic
requests
| where url endswith "generateAnswer" and name startswith "POST"
| parse kind = regex url with *"(?i)knowledgebases/"KbId"/generateAnswer"
| summarize ChatCount=count() by bin(timestamp, 1d), KbId
```

### <a name="total-question-traffic-in-a-given-time-period"></a>Tráfico total de preguntas en un período de tiempo determinado

```kusto
//Total Question Traffic in a given time period
let startDate = todatetime('2019-01-01');
let endDate = todatetime('2020-12-31');
requests
| where timestamp <= endDate and timestamp >=startDate
| where url endswith "generateAnswer" and name startswith "POST"
| parse kind = regex url with *"(?i)knowledgebases/"KbId"/generateAnswer"
| summarize ChatCount=count() by KbId
```

### <a name="user-traffic"></a>Tráfico de usuario

```kusto
//User Traffic
requests
| where url endswith "generateAnswer"
| project timestamp, id, url, resultCode, duration
| parse kind = regex url with *"(?i)knowledgebases/"KbId"/generateAnswer"
| join kind= inner (
traces | extend id = operation_ParentId
) on id
| extend UserId = tostring(customDimensions['UserId'])
| summarize ChatCount=count() by bin(timestamp, 1d), UserId, KbId
```

### <a name="latency-distribution-of-questions"></a>Distribución de latencia de preguntas

```kusto
//Latency distribution of questions
requests
| where url endswith "generateAnswer" and name startswith "POST"
| parse kind = regex url with *"(?i)knowledgebases/"KbId"/generateAnswer"
| project timestamp, id, name, resultCode, performanceBucket, KbId
| summarize count() by performanceBucket, KbId
```

### <a name="unanswered-questions"></a>Preguntas sin responder

```kusto
// Unanswered questions
requests
| where url endswith "generateAnswer"
| project timestamp, id, url
| parse kind = regex url with *"(?i)knowledgebases/"KbId"/generateAnswer"
| join kind= inner (
traces | extend id = operation_ParentId
) on id
| extend question = tostring(customDimensions['Question'])
| extend answer = tostring(customDimensions['Answer'])
| extend score = tostring(customDimensions['Score'])
| where  score  == "0" and message == "QnAMaker GenerateAnswer"
| project timestamp, KbId, question, answer, score
| order  by timestamp  desc
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Selección de la capacidad](./improve-knowledge-base.md)
