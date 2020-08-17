---
title: 'Detección inteligente: anomalías de error en Application Insights | Microsoft Docs'
description: Alerta de cambios no habituales en la frecuencia de solicitudes con errores a la aplicación web y proporciona análisis del diagnóstico. No necesita ninguna configuración.
ms.topic: conceptual
ms.date: 12/18/2018
ms.reviewer: yalavi
ms.openlocfilehash: a093d5d6bdb96aa6f0a8a92fea48835971aebe16
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87420216"
---
# <a name="smart-detection---failure-anomalies"></a>Detección inteligente: anomalías de error
[Application Insights](./app-insights-overview.md) le avisa automáticamente casi en tiempo real si la aplicación web sufre un aumento anómalo en la frecuencia de solicitudes erróneas. Asimismo, detecta un aumento inusual de la tasa de solicitudes HTTP o llamadas de dependencia notificadas como errores. En el caso de las solicitudes, las solicitudes con error suelen tener códigos de respuesta de 400 o superiores. Para ayudarle a evaluar las prioridades y a diagnosticar el problema, en los detalles de la alerta se proporciona un análisis de las características de los errores, así como datos de la aplicación relacionados. También hay vínculos en el portal de Application Insights para obtener un diagnóstico más amplio. La característica no necesita ninguna instalación o configuración, ya que usa algoritmos de aprendizaje automático para predecir la tasa normal de errores.

Esta característica funciona para cualquier aplicación web, hospedada en la nube o en sus propios servidores, que genere datos de dependencia o de solicitud de la aplicación. Por ejemplo, si tiene un rol de trabajo que llama a [TrackRequest()](./api-custom-events-metrics.md#trackrequest) o [TrackDependency()](./api-custom-events-metrics.md#trackdependency).

Después de configurar [Application Insights para su proyecto](./app-insights-overview.md), si la aplicación genera una cantidad mínima determinada de datos, la detección inteligente de anomalías de error tarda 24 horas en aprender el comportamiento normal de la aplicación antes de activarse y poder enviar alertas.

Esta es una alerta de ejemplo:

:::image type="content" source="./media/proactive-failure-diagnostics/013.png" alt-text="Alerta de ejemplo de detección inteligente que muestra el análisis del clúster en torno al error." lightbox="./media/proactive-failure-diagnostics/013.png":::

Los detalles de la alerta le indicarán lo siguiente:

* La tasa de errores en comparación con el comportamiento normal de la aplicación.
* Cuántos usuarios se ven afectados (para darle una idea de la gravedad).
* Un patrón característico relacionado con los errores. En este ejemplo encontrará un código de respuesta, un nombre de solicitud (operación) y una versión de aplicación concretos. Esto le indica inmediatamente por dónde empezar a buscar en el código. Otras posibilidades pueden ser un sistema operativo cliente o explorador específicos.
* Las excepciones, seguimientos de registros y errores de dependencias (bases de datos u otros componentes externos) que parecen estar asociados con los errores caracterizados.
* Vínculo directos a búsquedas significativas en los datos de Application Insights.

## <a name="benefits-of-smart-detection"></a>Ventajas de la detección inteligente
Las [alertas de métricas](../platform/alerts-log.md) normales le comunican que puede haber un problema. Sin embargo, la detección inteligente inicia el trabajo de diagnóstico y realiza gran parte del análisis que, de otra forma, tendría que hacer usted mismo. Los resultados se le presentan claramente organizados, lo que le ayuda a llegar rápidamente a la raíz del problema.

## <a name="how-it-works"></a>Funcionamiento
La detección inteligente supervisa los datos recibidos de su aplicación y, en particular, las tasas de errores. Esta regla cuenta el número de solicitudes para el que la propiedad `Successful request` es falsa y el número de llamadas de dependencia para el que la propiedad `Successful call` es falsa. De manera predeterminada, para las solicitudes, `Successful request == (resultCode < 400)` (a no ser que haya escrito código personalizado para [filtrar](./api-filtering-sampling.md#filtering) o generar sus propias llamadas de [TrackRequest](./api-custom-events-metrics.md#trackrequest)). 

El rendimiento de una aplicación tiene un patrón típico de comportamiento. Algunas solicitudes o llamadas de dependencia son más propensas a errores que otras; y la tasa de error general puede aumentar a medida que aumenta la carga. La detección inteligente usa aprendizaje automático para encontrar estas anomalías.

A medida que Application Insights recibe datos de su aplicación web, la detección inteligente compara el comportamiento actual con los patrones vistos a lo largo de los últimos días. Si se observa un incremento anómalo de la tasa de errores en comparación con el rendimiento previo, se desencadena un análisis.

Cuando se desencadena un análisis, el servicio realiza un análisis del clúster en la solicitud errónea, para tratar de identificar un patrón de valores que caracterice los errores. 

En el ejemplo anterior, el análisis ha detectado que la mayoría de los errores giran en torno a un código de resultado, nombre de solicitud, host de URL de servidor e instancia de rol específicos. 

Si se instrumenta el servicio con estas llamadas de datos, el analizador busca una excepción y un error de dependencia que estén asociados a solicitudes del clúster que ha identificado, junto con un ejemplo de cualquier registro de seguimiento asociado a estas solicitudes.

El análisis resultante se le envía como una alerta, a no ser que configurara lo contrario.

Al igual que con las [alertas que establece de forma manual](../platform/alerts-log.md), puede inspeccionar el estado de la alerta desencadenada y se puede resolver si el problema se ha corregido. Configure las reglas de alertas en la página Alertas de su recurso de Application Insights. Pero, a diferencia de otras alertas, no es necesario instalar ni configurar la detección inteligente. Si lo desea, puede deshabilitarla o cambiar sus direcciones de correo electrónico de destino.

### <a name="alert-logic-details"></a>Detalles de la lógica de alerta

Las alertas se desencadenan por nuestro algoritmo de aprendizaje automático propietario, por lo que no se pueden compartir los detalles exactos de la implementación. Dicho esto, entendemos que a veces se necesita saber más acerca de cómo funciona la lógica subyacente. Los principales factores que se evalúan para determinar si se debe desencadenar una alerta son: 

* Análisis del porcentaje de errores de las solicitudes o dependencias en una ventana de tiempo adaptable de 20 minutos.
* Una comparación del porcentaje de errores de los últimos 20 minutos con la tasa de los últimos 40 minutos y los últimos siete días, y la búsqueda de desviaciones significativas que superen X veces esa desviación estándar.
* Al utilizar un límite adaptativo para el porcentaje mínimo de errores, que varía en función del volumen de solicitudes o dependencias de la aplicación.
* Hay lógica que puede resolver automáticamente la condición supervisada de la alerta desencadenada si el problema deja de detectarse durante 8-24 horas.

## <a name="configure-alerts"></a>Configurar alertas

Puede deshabilitar la regla de alertas de la detección inteligente desde el portal o a través de Azure Resource Manager ([consulte el ejemplo de plantilla](./proactive-arm-config.md)).

Esta regla de alertas está creada con un [grupo de acciones](../platform/action-groups.md) asociado denominado "Detección inteligente de Application Insights" que contiene acciones de correo electrónico y webhook y se puede extender para desencadenar acciones adicionales cuando se activa la alerta.

> [!NOTE]
> Las notificaciones por correo electrónico que se envían desde esta regla de alertas ahora se envían de manera predeterminada a los usuarios asociados con los roles Lector de supervisión y Colaborador de supervisión de la suscripción. Encontrará más información sobre esto [aquí](./proactive-email-notification.md).
> Las notificaciones enviadas desde esta regla de alertas siguen el [esquema de alertas comunes](../platform/alerts-common-schema.md).
>

Abra la página Alertas. Las reglas de alertas de anomalías de error se incluyen junto con las alertas que se han establecido manualmente, y puede ver si están actualmente en el estado de alerta.

:::image type="content" source="./media/proactive-failure-diagnostics/021.png" alt-text="En la página del recurso de Application Insights, haga clic en el icono Alertas y luego en Administrar reglas de alerta." lightbox="./media/proactive-failure-diagnostics/021.png":::

Haga clic en la alerta para configurarla.

:::image type="content" source="./media/proactive-failure-diagnostics/032.png" alt-text="Pantalla de configuración de reglas." lightbox="./media/proactive-failure-diagnostics/032.png":::

Tenga en cuenta que puede deshabilitar o eliminar una regla de alertas de anomalías de error, pero no puede crear otra en el mismo recurso de Application Insights.

## <a name="example-of-failure-anomalies-alert-webhook-payload"></a>Ejemplo de la carga de webhook de una alerta de anomalías de error

```json
{
    "properties": {
        "essentials": {
            "severity": "Sev3",
            "signalType": "Log",
            "alertState": "New",
            "monitorCondition": "Resolved",
            "monitorService": "Smart Detector",
            "targetResource": "/subscriptions/4f9b81be-fa32-4f96-aeb3-fc5c3f678df9/resourcegroups/test-group/providers/microsoft.insights/components/test-rule",
            "targetResourceName": "test-rule",
            "targetResourceGroup": "test-group",
            "targetResourceType": "microsoft.insights/components",
            "sourceCreatedId": "1a0a5b6436a9b2a13377f5c89a3477855276f8208982e0f167697a2b45fcbb3e",
            "alertRule": "/subscriptions/4f9b81be-fa32-4f96-aeb3-fc5c3f678df9/resourcegroups/test-group/providers/microsoft.alertsmanagement/smartdetectoralertrules/failure anomalies - test-rule",
            "startDateTime": "2019-10-30T17:52:32.5802978Z",
            "lastModifiedDateTime": "2019-10-30T18:25:23.1072443Z",
            "monitorConditionResolvedDateTime": "2019-10-30T18:25:26.4440603Z",
            "lastModifiedUserName": "System",
            "actionStatus": {
                "isSuppressed": false
            },
            "description": "Failure Anomalies notifies you of an unusual rise in the rate of failed HTTP requests or dependency calls."
        },
        "context": {
            "DetectionSummary": "An abnormal rise in failed request rate",
            "FormattedOccurenceTime": "2019-10-30T17:50:00Z",
            "DetectedFailureRate": "50.0% (200/400 requests)",
            "NormalFailureRate": "0.0% (over the last 30 minutes)",
            "FailureRateChart": [["2019-10-30T05:20:00Z",
            0],
            ["2019-10-30T05:40:00Z",
            100],
            ["2019-10-30T06:00:00Z",
            0],
            ["2019-10-30T06:20:00Z",
            0],
            ["2019-10-30T06:40:00Z",
            100],
            ["2019-10-30T07:00:00Z",
            0],
            ["2019-10-30T07:20:00Z",
            0],
            ["2019-10-30T07:40:00Z",
            100],
            ["2019-10-30T08:00:00Z",
            0],
            ["2019-10-30T08:20:00Z",
            0],
            ["2019-10-30T08:40:00Z",
            100],
            ["2019-10-30T17:00:00Z",
            0],
            ["2019-10-30T17:20:00Z",
            0],
            ["2019-10-30T09:00:00Z",
            0],
            ["2019-10-30T09:20:00Z",
            0],
            ["2019-10-30T09:40:00Z",
            100],
            ["2019-10-30T10:00:00Z",
            0],
            ["2019-10-30T10:20:00Z",
            0],
            ["2019-10-30T10:40:00Z",
            100],
            ["2019-10-30T11:00:00Z",
            0],
            ["2019-10-30T11:20:00Z",
            0],
            ["2019-10-30T11:40:00Z",
            100],
            ["2019-10-30T12:00:00Z",
            0],
            ["2019-10-30T12:20:00Z",
            0],
            ["2019-10-30T12:40:00Z",
            100],
            ["2019-10-30T13:00:00Z",
            0],
            ["2019-10-30T13:20:00Z",
            0],
            ["2019-10-30T13:40:00Z",
            100],
            ["2019-10-30T14:00:00Z",
            0],
            ["2019-10-30T14:20:00Z",
            0],
            ["2019-10-30T14:40:00Z",
            100],
            ["2019-10-30T15:00:00Z",
            0],
            ["2019-10-30T15:20:00Z",
            0],
            ["2019-10-30T15:40:00Z",
            100],
            ["2019-10-30T16:00:00Z",
            0],
            ["2019-10-30T16:20:00Z",
            0],
            ["2019-10-30T16:40:00Z",
            100],
            ["2019-10-30T17:30:00Z",
            50]],
            "ArmSystemEventsRequest": "/subscriptions/4f9b81be-fa32-4f96-aeb3-fc5c3f678df9/resourceGroups/test-group/providers/microsoft.insights/components/test-rule/query?query=%0d%0a++++++++++++++++systemEvents%0d%0a++++++++++++++++%7c+where+timestamp+%3e%3d+datetime(%272019-10-30T17%3a20%3a00.0000000Z%27)+%0d%0a++++++++++++++++%7c+where+itemType+%3d%3d+%27systemEvent%27+and+name+%3d%3d+%27ProactiveDetectionInsight%27+%0d%0a++++++++++++++++%7c+where+dimensions.InsightType+in+(%275%27%2c+%277%27)+%0d%0a++++++++++++++++%7c+where+dimensions.InsightDocumentId+%3d%3d+%27718fb0c3-425b-4185-be33-4311dfb4deeb%27+%0d%0a++++++++++++++++%7c+project+dimensions.InsightOneClassTable%2c+%0d%0a++++++++++++++++++++++++++dimensions.InsightExceptionCorrelationTable%2c+%0d%0a++++++++++++++++++++++++++dimensions.InsightDependencyCorrelationTable%2c+%0d%0a++++++++++++++++++++++++++dimensions.InsightRequestCorrelationTable%2c+%0d%0a++++++++++++++++++++++++++dimensions.InsightTraceCorrelationTable%0d%0a++++++++++++&api-version=2018-04-20",
            "LinksTable": [{
                "Link": "<a href=\"https://portal.azure.com/#blade/AppInsightsExtension/ProactiveDetectionFeedBlade/ComponentId/{\"SubscriptionId\":\"4f9b81be-fa32-4f96-aeb3-fc5c3f678df9\",\"ResourceGroup\":\"test-group\",\"Name\":\"test-rule\"}/SelectedItemGroup/718fb0c3-425b-4185-be33-4311dfb4deeb/SelectedItemTime/2019-10-30T17:50:00Z/InsightType/5\" target=\"_blank\">View full details in Application Insights</a>"
            }],
            "SmartDetectorId": "FailureAnomaliesDetector",
            "SmartDetectorName": "Failure Anomalies",
            "AnalysisTimestamp": "2019-10-30T17:52:32.5802978Z"
        },
        "egressConfig": {
            "displayConfig": [{
                "rootJsonNode": null,
                "sectionName": null,
                "displayControls": [{
                    "property": "DetectionSummary",
                    "displayName": "What was detected?",
                    "type": "Text",
                    "isOptional": false,
                    "isPropertySerialized": false
                },
                {
                    "property": "FormattedOccurenceTime",
                    "displayName": "When did this occur?",
                    "type": "Text",
                    "isOptional": false,
                    "isPropertySerialized": false
                },
                {
                    "property": "DetectedFailureRate",
                    "displayName": "Detected failure rate",
                    "type": "Text",
                    "isOptional": false,
                    "isPropertySerialized": false
                },
                {
                    "property": "NormalFailureRate",
                    "displayName": "Normal failure rate",
                    "type": "Text",
                    "isOptional": false,
                    "isPropertySerialized": false
                },
                {
                    "chartType": "Line",
                    "xAxisType": "Date",
                    "yAxisType": "Percentage",
                    "xAxisName": "",
                    "yAxisName": "",
                    "property": "FailureRateChart",
                    "displayName": "Failure rate over last 12 hours",
                    "type": "Chart",
                    "isOptional": false,
                    "isPropertySerialized": false
                },
                {
                    "defaultLoad": true,
                    "displayConfig": [{
                        "rootJsonNode": null,
                        "sectionName": null,
                        "displayControls": [{
                            "showHeader": false,
                            "columns": [{
                                "property": "Name",
                                "displayName": "Name"
                            },
                            {
                                "property": "Value",
                                "displayName": "Value"
                            }],
                            "property": "tables[0].rows[0][0]",
                            "displayName": "All of the failed requests had these characteristics:",
                            "type": "Table",
                            "isOptional": false,
                            "isPropertySerialized": true
                        }]
                    }],
                    "property": "ArmSystemEventsRequest",
                    "displayName": "",
                    "type": "ARMRequest",
                    "isOptional": false,
                    "isPropertySerialized": false
                },
                {
                    "showHeader": false,
                    "columns": [{
                        "property": "Link",
                        "displayName": "Link"
                    }],
                    "property": "LinksTable",
                    "displayName": "Links",
                    "type": "Table",
                    "isOptional": false,
                    "isPropertySerialized": false
                }]
            }]
        }
    },
    "id": "/subscriptions/4f9b81be-fa32-4f96-aeb3-fc5c3f678df9/resourcegroups/test-group/providers/microsoft.insights/components/test-rule/providers/Microsoft.AlertsManagement/alerts/7daf8739-ca8a-4562-b69a-ff28db4ba0a5",
    "type": "Microsoft.AlertsManagement/alerts",
    "name": "Failure Anomalies - test-rule"
}
```

## <a name="triage-and-diagnose-an-alert"></a>Evaluación de prioridades y diagnóstico de una alerta

Una alerta indica que se detectó un aumento anómalo de la tasa de solicitudes con errores. Es probable que haya algún problema con la aplicación o con su entorno.

Si necesita investigar más, haga clic en los vínculos "Ver todos los detalles en Application Insights" de esta página que le llevarán directamente a una [página de búsqueda](./diagnostic-search.md) filtrada por las solicitudes, la excepción, la dependencia o el seguimiento correspondientes. 

También puede abrir [Azure Portal](https://portal.azure.com), navegar hasta el recurso de Application Insights de su aplicación y abrir la página Errores.

Al hacer clic en "Diagnosticar errores", obtendrá más detalles y podrá resolver el problema.

:::image type="content" source="./media/proactive-failure-diagnostics/051.png" alt-text="Búsqueda de diagnóstico." lightbox="./media/proactive-failure-diagnostics/051.png#lightbox":::

Puede determinar la urgencia del problema a partir del porcentaje de solicitudes y del número de usuarios afectados. En el ejemplo anterior, la tasa de error del 78,5 % se compara con una tasa normal del 2,2 %, lo que indica que hay algo que no va bien. Por otro lado, solo 46 usuarios se vieron afectados. Si se tratara de su aplicación, podría evaluar la gravedad del problema.

En muchos casos, podrá diagnosticar el problema rápidamente a partir del nombre de la solicitud, las excepciones, los errores de dependencias y otros datos de seguimiento proporcionados.

En este ejemplo, se produjo una excepción de SQL Database debido a que se alcanzó el límite de solicitudes.

:::image type="content" source="./media/proactive-failure-diagnostics/052.png" alt-text="Detalles de las solicitudes con errores." lightbox="./media/proactive-failure-diagnostics/052.png":::

## <a name="review-recent-alerts"></a>Revisar las alertas recientes

Haga clic en **Alertas** en la página de recursos de Application Insights para ver las alertas desencadenadas más recientes:

:::image type="content" source="./media/proactive-failure-diagnostics/070.png" alt-text="Resumen de alertas." lightbox="./media/proactive-failure-diagnostics/070.png":::

## <a name="whats-the-difference-"></a>¿Cuál es la diferencia ...
La detección inteligente de anomalías de errores complementa otras características similares pero distintas de Application Insights.

* Las [alertas de métricas](../platform/alerts-log.md) las configura el usuario y pueden supervisar una amplia variedad de métricas, como el uso de la CPU, las tasas de solicitudes y los tiempos de carga de las páginas, entre otras. Puede usarlas para avisarle, por ejemplo, de si necesita agregar más recursos. Por el contrario, la detección inteligente de anomalías de error abarca un pequeño conjunto de métricas críticas (actualmente solo la tasa de solicitudes con errores), diseñadas para notificarle casi en tiempo real el momento en el que la tasa de solicitudes con errores de la aplicación web aumenta en comparación con el comportamiento normal de esta. A diferencia de las alertas de métricas, la detección inteligente establece y actualiza automáticamente los umbrales en respuesta a los cambios en el comportamiento. La detección inteligente también inicia el trabajo de diagnóstico, lo que le ahorra tiempo en la resolución de problemas.

* La [detección inteligente de anomalías de rendimiento](proactive-performance-diagnostics.md) también usa la inteligencia automática para detectar patrones inusuales en las métricas y no requiere ninguna configuración por parte del usuario. Pero a diferencia de la detección inteligente de anomalías de errores, el objetivo de la detección inteligente de anomalías de rendimiento es encontrar segmentos del colector de uso que pudieran haberse servidor incorrectamente, por ejemplo, por páginas específicas de un tipo específico de explorador. El análisis se realiza diariamente y, si se encuentra algún resultado, probablemente sea mucho menos urgente que una alerta. Por el contrario, el análisis de anomalías de error se realiza continuamente sobre los datos de aplicación entrantes, y se le notificará en unos minutos si las tasas de errores del servidor son mayores de lo esperado.

## <a name="if-you-receive-a-smart-detection-alert"></a>Si recibe una alerta de detección inteligente
*¿Por qué he recibido esta alerta?*

* Hemos detectado un aumento anómalo en la tasa de solicitudes con errores en comparación con la línea de base normal del período anterior. Después de analizar los errores y los datos de aplicación asociados, creemos que hay un problema que debe examinar.

*¿La notificación significa que tengo definitivamente un problema?*

* Intentamos alertarle sobre las interrupciones o la degradación de la aplicación, aunque solo usted puede entender totalmente la semántica y el impacto en la aplicación o los usuarios.

*¿Está viendo los datos de mi aplicación?*

* No. El servicio es completamente automático. Solo obtendrá las notificaciones. Sus datos son [privados](./data-retention-privacy.md).

*¿Es necesario suscribirse a esta alerta?*

* No. Cada aplicación que envía datos de solicitud tiene la regla de alerta de detección inteligente.

*¿Puedo cancelar la suscripción u hacer que mis colegas reciban las notificaciones?*

* Sí, en Reglas de alerta, haga clic en la regla de detección inteligente para configurarla. Puede deshabilitar la alerta o cambiar a los destinatarios de la misma.

*Perdí el mensaje de correo electrónico. ¿Dónde puedo encontrar las notificaciones en el portal?*

* En los registros de actividad. En Azure, abra el recurso de Application Insights correspondiente a su aplicación y luego seleccione los registros de actividad.

*Algunas de las alertas se refieren a problemas conocidos y no deseo recibirlas.*

* Puede usar la característica de supresión de las [reglas de acción de alerta](../platform/alerts-action-rules.md).

## <a name="next-steps"></a>Pasos siguientes
Estas herramientas de diagnóstico lo ayudarán a inspeccionar los datos de su aplicación:

* [Explorador de métricas](../platform/metrics-charts.md)
* [Explorador de búsqueda](./diagnostic-search.md)
* [Analytics: Lenguaje de consulta eficaz](../log-query/get-started-portal.md)

Las detecciones inteligentes son automáticas. Pero ¿quizás le gustaría configurar algunas alertas más?

* [Alertas de métricas configuradas manualmente](../platform/alerts-log.md)
* [Pruebas web de disponibilidad](./monitor-web-app-availability.md)

