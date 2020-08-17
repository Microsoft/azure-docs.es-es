---
title: Administración del uso y los costos de Azure Application Insights | Microsoft Docs
description: Administre los volúmenes de telemetría y supervise los costos en Application Insights.
ms.topic: conceptual
author: DaleKoetke
ms.author: dalek
ms.date: 5/7/2020
ms.reviewer: mbullwin
ms.openlocfilehash: 3914764035d65482bcf224f8d0eda9c6579e03a4
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87309688"
---
# <a name="manage-usage-and-costs-for-application-insights"></a>Administración del uso y los costos de Application Insights

> [!NOTE]
> En este artículo se describe cómo entender y controlar los costos de Application Insights.  En un artículo relacionado, [Supervisión del uso y costos estimados](../platform/usage-estimated-costs.md), se describe cómo ver el uso y los costos estimados mediante varias características de supervisión de Azure para los distintos modelos de precios.

Application Insights está diseñado para obtener todo lo que necesita para supervisar la disponibilidad, el rendimiento y el uso de las aplicaciones web, tanto si están hospedadas en Azure como en un entorno local. Application Insights admite lenguajes y plataformas populares, como .NET, Java y Node.js, y se integra con procesos y herramientas DevOps como Azure DevOps, Jira y PagerDuty. Es importante comprender lo que determina los costos de la supervisión de aplicaciones. En este artículo se revisan los costos de supervisión de las aplicaciones y cómo puede supervisarlas y controlarlas de forma activa.

Si tiene preguntas sobre cómo funcionan los precios para Application Insights, no dude en publicar una pregunta en nuestra [página de preguntas y respuestas de Microsoft](/answers/topics/azure-monitor.html).

## <a name="pricing-model"></a>Modelo de precios

Los precios de [Azure Application Insights][start] son de un modelo de **pago por uso**, que se basa en el volumen de datos ingeridos y, opcionalmente, para obtener una mayor retención de datos. Cada recurso de Application Insights se cobra como un servicio independiente y contribuye a la factura de la suscripción a Azure. El volumen de datos se mide como el tamaño del paquete de datos JSON sin comprimir que recibe Application Insights de su aplicación. No hay ningún cargo por volumen de datos para usar [Live Metrics Stream](./live-stream.md).

Las [pruebas web de varios pasos](./availability-multistep.md) conllevan un cargo adicional. Las pruebas web de varios pasos son pruebas web que realizan una secuencia de acciones. No hay ningún cargo aparte para las *pruebas de ping* de una sola página. La telemetría de las pruebas de ping y de las de varios pasos se carga igual que el resto de la telemetría de su aplicación.

La opción Application Insights para [Habilitar la creación de alertas sobre las dimensiones de las métricas personalizadas](./pre-aggregated-metrics-log-metrics.md#custom-metrics-dimensions-and-pre-aggregation) también puede generar costos adicionales, ya que puede dar lugar a la creación de métricas de agregación previas adicionales. [Obtenga más información](./pre-aggregated-metrics-log-metrics.md) sobre las métricas basadas en registro y las agregadas previamente en Application Insights y sobre los [precios](https://azure.microsoft.com/pricing/details/monitor/) para las métricas personalizadas de Azure Monitor.

### <a name="workspace-based-application-insights"></a>Application Insights basado en áreas de trabajo

En el caso de los recursos de Application Insights que envían sus datos a un área de trabajo de Log Analytics, denominada [recursos de Application Insights basados en áreas de trabajo](create-workspace-resource.md), la facturación de la ingesta y retención de datos se realiza en el área de trabajo donde se encuentran los datos de Application Insights. Esto permite a los clientes aprovechar todas las opciones del [modelo de precios](../platform/manage-cost-storage.md#pricing-model) de Log Analytics, que incluye Reservas de capacidad además de Pago por uso. Log Analytics también tiene más opciones para la retención de datos, incluida la [retención por tipo de datos](../platform/manage-cost-storage.md#retention-by-data-type). Los tipos de datos de Application Insights en el área de trabajo obtienen 90 días de retención sin cargos. El uso de las pruebas web y la habilitación de las alertas sobre las dimensiones de la métrica personalizada todavía se notifica a través de Application Insights. Obtenga información sobre cómo realizar un seguimiento de los costos de retención y de ingesta de datos en Log Analytics mediante el [Uso y costos estimados](../platform/manage-cost-storage.md#understand-your-usage-and-estimate-costs), [Administración de costos + facturación de Azure](../platform/manage-cost-storage.md#viewing-log-analytics-usage-on-your-azure-bill) y [Consultas de Log Analytics](#data-volume-for-workspace-based-application-insights-resources). 

## <a name="estimating-the-costs-to-manage-your-application"></a>Estimación de los costos de administración de la aplicación

Si aún no usa Application Insights, puede usar la [calculadora de precios de Azure Monitor](https://azure.microsoft.com/pricing/calculator/?service=monitor) para calcular el costo del uso de Application Insights. Comience por escribir "Azure Monitor" en el cuadro de búsqueda y haga clic en el icono de Azure Monitor resultante. Desplácese hacia abajo en la página hasta Azure Monitor y seleccione Application Insights en la lista desplegable Tipo.  Aquí puede especificar el número de GB de datos que espera recopilar al mes, por lo que la pregunta es cuántos datos recopilará Application Insights supervisando la aplicación.

Hay dos enfoques para abordar esta cuestión: usar la supervisión predeterminada y el muestreo adaptable, que está disponible en el SDK de ASP.NET, o calcular la probable ingesta de datos en función de lo que han visto otros clientes similares.

### <a name="data-collection-when-using-sampling"></a>Colección de datos al usar el muestreo

Con el [muestreo adaptable](sampling.md#adaptive-sampling) del SDK de ASP.NET, el volumen de datos se ajusta automáticamente para mantener una velocidad de tráfico máxima específica para la supervisión predeterminada de Application Insights. Si la aplicación genera una cantidad baja de telemetría, como al depurar o debido a un uso bajo, el procesador de muestreo no podrá descargar los elementos mientras el volumen se encuentre por debajo del nivel configurado de eventos por segundo. En el caso de una aplicación de gran volumen, con el umbral predeterminado de cinco eventos por segundo, el muestreo adaptable limitará el número de eventos diarios a 432 000. Con un tamaño de evento promedio típico de 1 KB, corresponde a 13,4 GB de telemetría por mes de 31 días por nodo que hospeda la aplicación (ya que el muestreo se realiza de forma local en cada nodo). 

En el caso de los SDK que no admiten el muestreo adaptable, puede emplear el [muestreo de ingesta](./sampling.md#ingestion-sampling), que toma muestras cuando Application Insights recibe los datos en función de un porcentaje de datos que se deben conservar, o el [muestreo de frecuencia fija para sitios web ASP.NET, ASP.NET Core y Java](sampling.md#fixed-rate-sampling) para reducir el tráfico enviado desde el servidor web y los exploradores web.

### <a name="learn-from-what-similar-customers-collect"></a>Más información sobre qué recopilan los clientes similares

En la calculadora de precios de Azure Monitor para Application Insights, si habilita la funcionalidad "Estimar el volumen de datos en función de la actividad de la aplicación", puede proporcionar entradas sobre la aplicación (solicitudes por mes y vistas de página al mes, en caso de que se recopile la telemetría del lado cliente) y, a continuación, la calculadora le indicará el valor medio y la cantidad de percentil 90 de los datos recopilados por aplicaciones similares. Estas aplicaciones abarcan el intervalo de configuración de Application Insights (por ejemplo, algunas tienen el [muestreo](./sampling.md) predeterminado, otras no tienen muestreo, etc.), por lo que todavía tiene el control para reducir el volumen de datos que ingiere muy por debajo del nivel medio con muestreo. Pero se trata de un punto de partida para comprender lo que ven otros clientes similares.

## <a name="understand-your-usage-and-estimate-costs"></a>Información útil del uso y los costos estimados

Application Insights permite entender fácilmente cuáles serán los costos en función de patrones de uso reciente. Para comenzar, en Azure Portal, en el recurso de Application Insights, vaya a la página **Uso y costos estimados**:

![Elija el precio](./media/pricing/pricing-001.png)

A. Revise el volumen de datos para el mes. Esto incluye todos los datos recibidos y retenidos (después de cualquier [muestreo](./sampling.md)) de las aplicaciones de servidor y cliente, y de las pruebas de disponibilidad.  
B. Se realiza un cargo adicional para las [pruebas web de varios pasos](./availability-multistep.md). (Esto no incluye pruebas de disponibilidad simples, que se incluyen en la carga de volumen de datos).  
C. Vea las tendencias de volumen de datos durante el mes anterior.  
D. Habilite el [muestreo](./sampling.md) de ingesta de datos.
E. Establezca el límite de volumen de datos diario.  

(Tenga en cuenta que todos los precios mostrados en las capturas de pantalla de este artículo son solo con fines de ejemplo. Para ver los precios vigentes actualmente en su moneda y región, consulte [Precios de Application Insights][pricing]).

Para investigar el uso de Application Insights con más detalle, abra la página **Métricas**, agregue la métrica denominada "Volumen de puntos de datos" y, después, seleccione la opción *Aplicar división* para dividir los datos por "Tipo de elemento de telemetría".

Los cargos de Application Insights se agregarán a la factura de Azure. Puede ver los detalles de su factura de Azure en la sección **Cost Management + facturación** de Azure Portal o en el [Portal de facturación de Azure](https://account.windowsazure.com/Subscriptions).  [Vea más abajo](#viewing-application-insights-usage-on-your-azure-bill) para obtener información detallada sobre el uso de esto para Application Insights. 

![En el menú de la izquierda, seleccione Facturación](./media/pricing/02-billing.png)

### <a name="using-data-volume-metrics"></a>Uso de métricas de volumen de datos
<a id="understanding-ingested-data-volume"></a>

Para obtener más información sobre los volúmenes de datos, seleccione **Métricas** para el recurso de Application Insights y agregue un nuevo gráfico. En la métrica del gráfico, en **Métricas basadas en registros**, seleccione **Volumen de puntos de datos**. Haga clic en **Aplicar separación** y seleccione el grupo mediante el **tipo `Telemetryitem`** .

![Uso de métricas para examinar el volumen de datos](./media/pricing/10-billing.png)

### <a name="queries-to-understand-data-volume-details"></a>Consultas para comprender los detalles del volumen de datos

Existen dos enfoques para investigar los volúmenes de datos de Application Insights. El primero utiliza la información agregada en la tabla `systemEvents` y el segundo usa la propiedad `_BilledSize`, que está disponible en cada evento ingerido. `systemEvents` no tendrá información sobre el tamaño de datos para [Application Insights basado en el área de trabajo](#data-volume-for-workspace-based-application-insights-resources).

#### <a name="using-aggregated-data-volume-information"></a>Uso de información del volumen de datos agregados

Por ejemplo, puede usar la tabla `systemEvents` para ver el volumen de datos ingerido en las últimas 24 horas con la consulta:

```kusto
systemEvents
| where timestamp >= ago(24h)
| where type == "Billing"
| extend BillingTelemetryType = tostring(dimensions["BillingTelemetryType"])
| extend BillingTelemetrySizeInBytes = todouble(measurements["BillingTelemetrySize"])
| summarize sum(BillingTelemetrySizeInBytes)
```

O bien, para ver un gráfico de volumen de datos (en bytes) por tipo de datos de los últimos 30 días, puede usar:

```kusto
systemEvents
| where timestamp >= startofday(ago(30d))
| where type == "Billing"
| extend BillingTelemetryType = tostring(dimensions["BillingTelemetryType"])
| extend BillingTelemetrySizeInBytes = todouble(measurements["BillingTelemetrySize"])
| summarize sum(BillingTelemetrySizeInBytes) by BillingTelemetryType, bin(timestamp, 1d) | render barchart  
```

Tenga en cuenta que esta consulta se puede usar en una [alerta de registro de Azure](../platform/alerts-unified-log.md) para configurar las alertas en los volúmenes de datos.  

Para más información sobre los cambios en los datos de telemetría, se puede obtener el número de eventos por tipo mediante la consulta:

```kusto
systemEvents
| where timestamp >= startofday(ago(30d))
| where type == "Billing"
| extend BillingTelemetryType = tostring(dimensions["BillingTelemetryType"])
| summarize count() by BillingTelemetryType, bin(timestamp, 1d)
| render barchart  
```

#### <a name="using-data-size-per-event-information"></a>Uso de la información de tamaño de datos por evento

Para más información sobre el origen de los volúmenes de datos, puede usar la propiedad `_BilledSize` que se encuentra en cada evento ingerido.

Por ejemplo, para ver qué operaciones generan el mayor volumen de datos en los últimos 30 días, se puede sumar `_BilledSize` de todos los eventos de dependencia:

```kusto
dependencies
| where timestamp >= startofday(ago(30d))
| summarize sum(_BilledSize) by operation_Name
| render barchart  
```

#### <a name="data-volume-for-workspace-based-application-insights-resources"></a>Volumen de datos para recursos de Application Insights basados en áreas de trabajo

Para ver las tendencias de volumen de datos de todos los [recursos de Application Insights basados en el área de trabajo](create-workspace-resource.md) en un área de trabajo durante la última semana, vaya al área de trabajo Log Analytics y ejecute la consulta siguiente:

```kusto
union (AppAvailabilityResults),
      (AppBrowserTimings),
      (AppDependencies),
      (AppExceptions),
      (AppEvents),
      (AppMetrics),
      (AppPageViews),
      (AppPerformanceCounters),
      (AppRequests),
      (AppSystemEvents),
      (AppTraces)
| where TimeGenerated >= startofday(ago(7d) and TimeGenerated < startofday(now())
| summarize sum(_BilledSize) by _ResourceId, bin(TimeGenerated, 1d)
| render areachart
```

Para consultar las tendencias de volumen de datos por tipo para un recurso de Application Insights basado en un área de trabajo específica, use lo siguiente en el área de trabajo Log Analytics:

```kusto
union (AppAvailabilityResults),
      (AppBrowserTimings),
      (AppDependencies),
      (AppExceptions),
      (AppEvents),
      (AppMetrics),
      (AppPageViews),
      (AppPerformanceCounters),
      (AppRequests),
      (AppSystemEvents),
      (AppTraces)
| where TimeGenerated >= startofday(ago(7d) and TimeGenerated < startofday(now())
| where _ResourceId contains "<myAppInsightsResourceName>"
| summarize sum(_BilledSize) by Type, bin(TimeGenerated, 1d)
| render areachart
```

## <a name="viewing-application-insights-usage-on-your-azure-bill"></a>Visualización del uso de Application Insights en la factura de Azure

Azure proporciona una gran cantidad de funcionalidades útiles en el centro [Azure Cost Management + Facturación](../../cost-management-billing/costs/quick-acm-cost-analysis.md?toc=/azure/billing/TOC.json). Por ejemplo, la funcionalidad de "Análisis de costos" le permite ver los gastos de los recursos de Azure. Al agregar un filtro por tipo de recurso (en microsoft.insights/components para Application Insights), podrá realizar un seguimiento de los gastos. Después, para "Agrupar por", seleccione "Categoría del medidor" o "Medidor".  Para los recursos de Application Insights de los planes de precios actuales, la mayoría del uso se mostrará como Log Analytics de la Categoría del medidor, ya que hay un back-end de registros único para todos los componentes de Azure Monitor. 

Puede obtener información sobre el uso mediante la [descarga del uso desde Azure Portal](../../cost-management-billing/manage/download-azure-invoice-daily-usage-date.md#download-usage-in-azure-portal).
En la hoja de cálculo descargada puede ver el uso por recurso de Azure al día. En esta hoja de cálculo de Excel, el uso de los recursos de Application Insights se puede encontrar filtrando, en primer lugar, la columna "Categoría de medición" para mostrar "Application Insights" y "Log Analytics" y, a continuación, agregando un filtro en la columna "Id. de instancia", que es "contiene microsoft.insights/components".  La mayor parte del uso de Application Insights se muestra en medidores con la Categoría de medición de Log Analytics, ya que hay un back-end de registros único para todos los componentes de Azure Monitor.  Con una Categoría de medición de Application Insights, solo se muestran recursos de Application Insights de los planes de tarifa heredados y las pruebas web de varios pasos.  El uso se muestra en la columna "Cantidad consumida" y la unidad de cada entrada se muestra en la columna "Unidad de medida".  Hay más detalles disponibles para ayudarle a [entender la factura de Microsoft Azure](../../cost-management-billing/understand/review-individual-bill.md).

## <a name="managing-your-data-volume"></a>Administración del volumen de datos

El volumen de datos que envíe se puede administrar con las técnicas siguientes:

* **Muestreo**: puede usar el muestreo para reducir la cantidad de telemetría enviada desde las aplicaciones de servidor y de cliente, con mínima distorsión de las métricas. El muestreo es la principal herramienta que puede usar para ajustar la cantidad de datos que envía. Obtenga más información sobre las [características de muestreo](./sampling.md).

* **Limitar las llamadas Ajax**: Puede [limitar el número de llamadas Ajax que se pueden notificar](./javascript.md#configuration) en cada vista de página o desactive los informes de Ajax.

* **Deshabilitar los módulos innecesarios**: [Edite ApplicationInsights.config](./configuration-with-applicationinsights-config.md) para desactivar los módulos de recopilación que no necesite. Por ejemplo, podría decidir que los contadores de rendimiento o datos de dependencia no son esenciales.

* **Agregar métricas previamente**: Si coloca llamadas a TrackMetric en su aplicación, puede reducir el tráfico mediante la sobrecarga que acepta el cálculo de la media y la desviación estándar de un lote de medidas. O bien, puede usar un [paquete de agregación previa](https://www.myget.org/gallery/applicationinsights-sdk-labs).
 
* **Límite diario**: al crear un recurso de Application Insights en Azure Portal, el límite diario se establece en 100 GB/día. Al crear un recurso de Application Insights en Visual Studio, la capacidad predeterminada es pequeña (solo 32,3 MB/día). El límite diario predeterminado se establece para facilitar las pruebas. El objetivo es que el usuario aumente el límite diario antes de implementar la aplicación en producción. 

    El límite máximo es de 1,000 GB/día, a menos que solicite un máximo superior para una aplicación con mucho tráfico.
    
    Los correos electrónicos de advertencia sobre el límite diario se envían a cuentas que son miembros de estos roles en el recurso de Application Insights: "ServiceAdmin", "AccountAdmin", "coadmin", "Owner".

    Tenga cuidado al establecer el límite diario. Su objetivo debe ser *no alcanzar nunca el límite diario*. Si lo alcanza, perderá datos durante el resto del día y no podrá supervisar su aplicación. Para cambiar el límite diario, use la opción **Límite de volumen diario**. Puede tener acceso a esta opción en el panel **Uso y costos estimados** (se describe con más detalle más adelante en el artículo).
    
    Hemos quitado la restricción en algunos tipos de suscripción con crédito que no se podía usar para Application Insights. Anteriormente, si la suscripción tenía un límite de gasto, el cuadro de diálogo de límite diario mostraba instrucciones sobre cómo quitarlo y habilitarlo para superar los 32,3 MB/día.
    
* **Limitación**: la limitación restringe la velocidad de datos a 32 000 eventos por segundo, promediados durante 1 minuto por clave de instrumentación. El volumen de datos que su aplicación envía se evalúa cada minuto. Si se supera la tasa por segundo promediada por minuto, el servidor rechazará algunas solicitudes. El SDK almacena en búfer los datos y, a continuación, intenta volver a enviarlos. Extiende un aumento durante varios minutos. Si su aplicación envía de forma consistente un volumen de datos que supera la tasa de limitación, es posible que algunos se eliminen (los SDK de ASP.NET, Java y JavaScript intentan volver a enviarlos de esta manera; otros SDK pueden simplemente eliminar los datos limitados). Si se produce la limitación, una notificación de advertencia le indica que esto ha sucedido.

## <a name="manage-your-maximum-daily-data-volume"></a>Administración del volumen de datos diario máximo

Puede usar el límite de volumen diario para restringir los datos recopilados. Sin embargo, si se alcanza el límite, se producirá una pérdida de todas las telemetrías enviadas desde su aplicación durante el resto del día. No *es aconsejable* hacer que su aplicación alcance el límite diario. No puede realizar un seguimiento del estado y rendimiento de su aplicación una vez que alcance el límite diario.

En lugar de usar el límite de volumen diario, use el [muestreo](./sampling.md) para ajustar el volumen de datos al nivel que desee. A continuación, use el límite diario solo como "último recurso" en caso de que su aplicación empiece a enviar de forma inesperada volúmenes de telemetría mucho más altos.

### <a name="identify-what-daily-data-limit-to-define"></a>Identificación del límite diario de datos para definir

Revise el uso de Application Insights y los costos estimados para comprender la tendencia de ingesta de datos y cuál es el límite de volumen diario que se va a definir. Se debe considerar con cuidado, ya que no podrá supervisar los recursos una vez que se alcance el límite.

### <a name="set-the-daily-cap"></a>Establecer el límite diario

Para cambiar el límite diario, en la sección de **Configuración** del recurso de Application Insights, en la página **Uso y costos estimados**, seleccione **Límite diario**.

![Ajuste del límite de volumen de telemetría diario](./media/pricing/pricing-003.png)

Para [cambiar el límite diario a través de Azure Resource Manager](./powershell.md), la propiedad a cambiar es el `dailyQuota`.  A través de Azure Resource Manager también puede establecer `dailyQuotaResetTime` y el límite diario `warningThreshold`.

### <a name="create-alerts-for-the-daily-cap"></a>Creación de alertas para el límite diario

El límite diario de Application Insights crea un evento en el registro de actividades de Azure cuando los volúmenes de datos ingeridos alcanzan el nivel de advertencia o el nivel de límite diario.  Puede [crear una alerta basada en estos eventos del registro de actividad](../platform/alerts-activity-log.md#create-with-the-azure-portal). Los nombres de señal de estos eventos son:

* Se alcanzó el umbral de advertencia de límite diario del componente Application Insights

* Se alcanzó el límite diario del componente Application Insights

## <a name="sampling"></a>muestreo
El [muestreo](./sampling.md) es un método que permite reducir la velocidad a la que se envían datos de telemetría a la aplicación, al mismo tiempo que se conserva la capacidad de buscar eventos relacionados durante las búsquedas de diagnósticos. También retiene recuentos de eventos correctos.

El muestreo es una manera efectiva de reducir los gastos y permanecer dentro de la cuota mensual. El algoritmo de muestreo conserva elementos relacionados con la telemetría, de modo que, por ejemplo, al usar Search se puede buscar la solicitud relacionada con una excepción determinada. El algoritmo también conserva recuentos correctos, para que pueda ver en el Explorador de métricas los valores correctos de tasas de solicitudes, tasas de excepciones y otros recuentos.

Existen varias formas de muestreo.

* [Muestreo adaptable](./sampling.md) es el valor predeterminado del SDK de ASP.NET. El muestreo adaptable se ajusta automáticamente al volumen de telemetría que envía la aplicación. Funciona automáticamente en el SDK de la aplicación web, de modo que se reduce el tráfico de telemetría en la red. 
* *muestreo de ingesta* es una opción alternativa que funciona en el momento en que la telemetría procedente de su aplicación entra en el servicio Application Insights. El muestreo de ingesta no afecta al volumen de telemetría que envía la aplicación, pero reduce el volumen que retiene el servicio. Puede utilizar el muestreo de ingesta para reducir la cuota que utiliza la telemetría de exploradores y otros SDK.

Para establecer el muestreo de ingesta, vaya al panel **Precios**:

![En el panel Cuota y precios, seleccione el icono Ejemplos y, a continuación, seleccione una fracción de muestreo.](./media/pricing/pricing-004.png)

> [!WARNING]
> El panel **Muestreo de datos** solo controla el valor de muestreo de ingesta. No refleja la velocidad de muestreo que se aplica mediante el SDK de Application Insights en la aplicación. Si ya se ha muestreado la telemetría entrante en el SDK, no se aplica el muestreo de ingesta.
>

Para conocer la frecuencia de muestreo real independientemente de dónde se ha aplicado, use una [consulta de Analytics](../log-query/log-query-overview.md). La consulta tiene este aspecto:

```kusto
requests | where timestamp > ago(1d)
| summarize 100/avg(itemCount) by bin(timestamp, 1h)
| render areachart
```

En cada registro retenido, `itemCount` indica el número de registros originales que representa. Es igual a 1 + el número de registros descartados anteriores.

## <a name="change-the-data-retention-period"></a>Cambio del período de retención de datos

La retención predeterminada de los recursos de Application Insights es de 90 días. Es posible seleccionar distintos períodos de retención para cada recurso de Application Insights. El conjunto completo de períodos de retención disponibles es de 30, 60, 90, 120, 180, 270, 365, 550 o 730 días. [Obtenga más información](https://azure.microsoft.com/pricing/details/monitor/) sobre los precios de la retención de datos de mayor duración. 

Para cambiar la retención, en el recurso de Application Insights, vaya a la página **Uso y costos estimados** y seleccione la opción **Retención de datos**:

![Ajuste del límite de volumen de telemetría diario](./media/pricing/pricing-005.png)

Cuando se reduce la retención, hay un período de gracia de varios días antes de que se quiten los datos más antiguos.

La retención también se puede [establecer mediante programación con PowerShell](powershell.md#set-the-data-retention) con el parámetro `retentionInDays`. Si configura la retención de datos en 30 días, puede desencadenar una purga inmediata de los datos más antiguos mediante el parámetro `immediatePurgeDataOn30Days`, que puede serle útil en los escenarios relacionados con el cumplimiento. Esta funcionalidad de purga solo se expone con Azure Resource Manager y debe utilizarse con extrema precaución. El tiempo de restablecimiento diario del extremo de volumen de datos se puede configurar mediante Azure Resource Manager para establecer el parámetro `dailyQuotaResetTime`.

## <a name="data-transfer-charges-using-application-insights"></a>Cargos por transferencia de datos al usar Application Insights

Al enviar datos a Application Insights se pueden aplicar ciertos cargos debido al ancho de banda de datos. Tal como se describe en la [página de precios de Azure Bandwidth](https://azure.microsoft.com/pricing/details/bandwidth/), la transferencia de datos entre los servicios de Azure ubicados en dos regiones se cobra como transferencia de datos salientes a precio normal. La transferencia de datos entrantes es gratuita. Sin embargo, este cargo es muy pequeño (un tanto por ciento mínimo) en comparación con los costos de la ingesta de datos de registro de Application Insights. En consecuencia, para controlar los costos de Log Analytics, debe centrarse en el volumen de datos ingeridos; para ello, le ofrecemos información [ aquí ](#managing-your-data-volume).

## <a name="limits-summary"></a>Resumen de límites

[!INCLUDE [application-insights-limits](../../../includes/application-insights-limits.md)]

## <a name="disable-daily-cap-e-mails"></a>Deshabilitación de los correos electrónicos de límite diario

Para deshabilitar los correos electrónicos de límite de volumen diario, en la sección de **Configuración** del recurso de Application Insights, en el panel **Uso y costos estimados**, seleccione **Límite diario**. Hay opciones para enviar correos electrónicos cuando se alcanza el límite, así como cuando se ha alcanzado un nivel de advertencia ajustable. Si quiere deshabilitar todos los correos electrónicos relacionados con el límite de volumen diario, desactive ambas casillas.

## <a name="legacy-enterprise-per-node-pricing-tier"></a>Plan de tarifa de Enterprise heredado (por nodo)

Para los usuarios pioneros de Azure Application Insights, todavía hay dos planes de tarifa posibles: Básico y Enterprise. El plan de tarifa Básico es el mismo que se describió anteriormente y es el predeterminado. Incluye todas las características del plan Enterprise sin ningún coste adicional. En el nivel Básico, la facturación se realiza principalmente en función del volumen de datos ingerido.

> [!NOTE]
> Estos planes de tarifa heredados han cambiado de nombre. El plan de tarifa Enterprise ahora se llama **Por nodo** y el plan de tarifa Básico ahora se llama **Por GB**. Estos nuevos nombres se usan a continuación y en Azure Portal.  

El nivel Por nodo (anteriormente Enterprise) tiene un cargo por nodo y cada nodo recibe una concesión de datos diaria. En el plan de tarifa Por nodo, se le cobrará por los datos ingeridos por encima de la concesión incluida. Si usa Operations Management Suite, debería elegir el nivel Por nodo.

Para ver los precios vigentes actualmente en su moneda y región, consulte [Precios de Application Insights](https://azure.microsoft.com/pricing/details/application-insights/).

> [!NOTE]
> En abril de 2018, [introdujimos](https://azure.microsoft.com/blog/introducing-a-new-way-to-purchase-azure-monitoring-services/) un nuevo modelo de precios para la supervisión de Azure. Este modelo adopta un modelo de "pago por uso" sencillo en toda la cartera de servicios de supervisión. Más información sobre el [nuevo modelo de precios](../platform/usage-estimated-costs.md), cómo [evaluar el impacto de pasar a este modelo](../platform/usage-estimated-costs.md#understanding-your-azure-monitor-costs) en función de los patrones de uso y [cómo participar en el nuevo modelo](../platform/usage-estimated-costs.md#azure-monitor-pricing-model).

### <a name="per-node-tier-and-operations-management-suite-subscription-entitlements"></a>Derechos del nivel Por nodo y la suscripción a Operations Management Suite

Como se [anunció anteriormente](/archive/blogs/msoms/azure-application-insights-enterprise-as-part-of-operations-management-suite-subscription), los clientes que compran Operations Management Suite E1 y E2 pueden obtener Application Insights Por nodo como componente adicional sin coste alguno. Específicamente, cada unidad de Operations Management Suite E1 y E2 incluye el derecho de usar un nodo del nivel Application Insights Por nodo. Cada nodo de Application Insights incluye hasta 200 MB de datos ingeridos por día (independientes de la ingesta de datos de Log Analytics), con la retención de datos de 90 días sin costo adicional alguno. El nivel se describe con más detalle más adelante en el artículo.

Dado que este nivel solo se aplica a clientes con una suscripción a Operations Management Suite, los clientes sin una suscripción no verán una opción para seleccionar este nivel.

> [!NOTE]
> Para garantizar que tiene este derecho, sus recursos de Application Insights deben estar en el plan de tarifa Por nodo. Este derecho se aplica solo como nodos. Los recursos de Application Insights del nivel Por GB no consiguen ningún beneficio. Este derecho no es visible en los costos estimados que se muestran en el panel **Uso y costos estimados**. Además, si traslada una suscripción al nuevo modelo de precios de supervisión de Azure en abril de 2018, el nivel Por GB será el único nivel disponible. El traslado de una suscripción al nuevo modelo de precios de supervisión de Azure no es aconsejable si tiene una suscripción a Operations Management Suite.

### <a name="how-the-per-node-tier-works"></a>Funcionamiento del nivel Por nodo

* En el nivel Por nodo, se paga por cada nodo que envíe datos de telemetría desde cualquier aplicación.
  * Un *nodo* es un servidor físico o virtual o una instancia del rol de plataforma como servicio que hospeda la aplicación.
  * Los equipos de desarrollo, los exploradores cliente y los dispositivos móviles no se cuentan como nodos.
  * Si la aplicación contiene varios componentes que envían datos telemetría, como un servicio web y un trabajo de back-end, estos componentes se cuentan por separado.
  * Los datos de la [secuencia de métricas en directo](./live-stream.md) no cuentan para calcular los precios. En una suscripción, se cobra por nodo, no por aplicación. Si tiene cinco nodos que envían telemetría desde 12 aplicaciones, se le cobrarán 5 nodos.
* Aunque los cargos se presupuestan mensualmente, solo se le cobrarán las horas en las que un nodo envíe telemetría de una aplicación. El cargo por hora es el cargo mensual presupuestado dividido entre 744 (el número de horas que tiene un mes de 31 días).
* Cada día, se asigna un volumen de datos de 200 MB por cada nodo detectado (con una granularidad por hora). Las asignaciones de datos que no se utilizan no se guardan de un día para otro.
  * Si elige el plan de tarifa Por nodo, cada suscripción recibirá una concesión de datos diaria en función del número de nodos que envíen datos de telemetría a los recursos de Application Insights de esa suscripción. Por tanto, si tiene cinco nodos que envían datos todo el día, tendrá una asignación global de 1 GB para todos los recursos de Application Insights de esa suscripción. No importa si algunos nodos envían más datos que otros, ya que los datos incluidos se comparten entre todos los nodos. Si, un día determinado, los recursos de Application Insights reciben más datos de los que se incluyen en la concesión de datos diaria para esta suscripción, se aplicarán los cargos de datos por encima del límite por cada GB. 
  * La asignación de datos diaria se calcula como el número de horas del día (de acuerdo con UTC) que cada nodo envía datos de telemetría dividido entre 24 y multiplicado por 200 MB. Es decir, si tiene cuatro nodos que envían datos de telemetría durante 15 de las 24 horas del día, los datos incluidos para ese día serán ((4 &#215; 15) / 24) &#215; 200 MB = 500 MB. A un precio de 2,30 USD por GB en el caso de uso por encima del límite de datos, el cargo sería de 1,15 USB si los nodos envían 1 GB de datos ese día.
  * La concesión diaria del nivel Por nodo no se comparte con las aplicaciones para las que haya elegido el nivel Por GB. La asignación que no se ha utilizado no se guarda de un día para otro.

### <a name="examples-of-how-to-determine-distinct-node-count"></a>Ejemplos acerca de cómo determinar el número de nodos

| Escenario                               | Número de nodos total diario |
|:---------------------------------------|:----------------:|
| 1 aplicación utiliza 3 instancias de Azure App Service y 1 servidor virtual | 4 |
| 3 aplicaciones se ejecutan en 2 VM; los recursos de Application Insights de estas aplicaciones están en la misma suscripción y en el nivel Por nodo | 2 | 
| 4 aplicaciones cuyos recursos de Applications Insights están en la misma suscripción; cada aplicación ejecuta 2 instancias durante 16 horas de poca actividad y 4 instancias durante 8 horas punta | 13,33 |
| Servicios en la nube con 1 rol de trabajo y 1 rol web; cada uno ejecuta 2 instancias. | 4 | 
| Un clúster de Azure Service Fabric con 5 nodos ejecuta 50 microservicios; cada microservicio ejecuta 3 instancias | 5|

* El número preciso de nodos depende del SDK de Application Insights que utilice la aplicación. 
  * En las versiones 2.2 y posteriores del SDK, tanto el [SDK básico](https://www.nuget.org/packages/Microsoft.ApplicationInsights/) como el [SDK web](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Web/) de Application Insights cuentan cada host de aplicación como un nodo. Algunos ejemplos son el nombre de equipo del servidor físico y los hosts de máquina virtual, o el nombre de instancia de los servicios en la nube.  La única excepción es una aplicación que solamente usa [.NET Core](https://dotnet.github.io/) y el SDK básico de Application Insights. En ese caso, solo se cuenta un nodo para todos los hosts porque el nombre de host no está disponible.
  * En las versiones anteriores del SDK, el [SDK web](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Web/) se comporta del mismo modo que las versiones más nuevas. Sin embargo, el [SDK básico](https://www.nuget.org/packages/Microsoft.ApplicationInsights/) solo cuenta un nodo, independientemente del número real de hosts de la aplicación.
  * Si la aplicación utiliza el SDK para establecer **roleInstance** en un valor personalizado, ese mismo valor se usará de manera predeterminada para calcular el número de nodos.
  * Si utiliza una nueva versión del SDK con una aplicación que se ejecuta desde equipos cliente o dispositivos móviles, es posible que el recuento de nodos devuelva un número grande (debido a la gran cantidad de equipos cliente o de dispositivos móviles).

## <a name="automation"></a>Automation

Puede escribir un script para establecer el plan de tarifa mediante Azure Resource Management. [Más información](powershell.md#price).

## <a name="next-steps"></a>Pasos siguientes

* [muestreo](./sampling.md)

[api]: app-insights-api-custom-events-metrics.md
[apiproperties]: app-insights-api-custom-events-metrics.md#properties
[start]: ./app-insights-overview.md
[pricing]: https://azure.microsoft.com/pricing/details/application-insights/
[pricing]: https://azure.microsoft.com/pricing/details/application-insights/

