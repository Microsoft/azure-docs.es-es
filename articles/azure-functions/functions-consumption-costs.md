---
title: Estimación de los costos del plan de consumo en Azure Functions
description: Obtenga información sobre cómo estimar mejor los costos que se pueden producir al ejecutar la aplicación de funciones en un plan de consumo de Azure.
ms.date: 9/20/2019
ms.topic: conceptual
ms.openlocfilehash: 880d1c20c75ce297b556ac203e309e446227e97a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87083045"
---
# <a name="estimating-consumption-plan-costs"></a>Estimación de los costos según el plan de consumo

Actualmente hay tres tipos de planes de hospedaje para una aplicación que se ejecuta en Azure Functions, donde cada plan tiene su propio modelo de precios: 

| Plan | Descripción |
| ---- | ----------- |
| [**Consumo**](functions-scale.md#consumption-plan) | Solo se le cobrará por el tiempo que se ejecute la aplicación de funciones. Este plan incluye una [concesión gratuita][página de precios] por suscripción.|
| [**Premium**](functions-scale.md#premium-plan) | Proporciona las mismas características y mecanismo de escalado que el plan de consumo, pero con un rendimiento mejorado y acceso a redes virtuales. El costo se basa en el plan de tarifa elegido. Para más información, consulte [Plan Premium de Azure Functions](functions-premium-plan.md). |
| [**Dedicado (App Service)** ](functions-scale.md#app-service-plan) <br/>(nivel básico o superior) | Cuando necesite ejecutar en máquinas virtuales dedicadas o en aislamiento, utilice imágenes personalizadas o quiera usar el exceso de capacidad del plan de App Service. Usa la [facturación del plan de App Service regular](https://azure.microsoft.com/pricing/details/app-service/). El costo se basa en el plan de tarifa elegido.|

Elija el plan que mejor se adapte a sus requisitos de rendimiento y costo de sus funciones. Para más información, consulte [Escalado y hospedaje de Azure Functions](functions-scale.md).

En este artículo solo se trata el plan de consumo, ya que este plan da como resultado costos variables. Este artículo reemplaza al artículo [Preguntas más frecuentes sobre la facturación de costos del plan de consumo](https://github.com/Azure/Azure-Functions/wiki/Consumption-Plan-Cost-Billing-FAQ).

Durable Functions también puede ejecutarse en un plan de consumo. Para más información sobre las consideraciones de costo al usar Durable Functions, consulte [Facturación de Durable Functions](./durable/durable-functions-billing.md).

## <a name="consumption-plan-costs"></a>Costos según el plan de consumo

El *costo* de ejecución de una única ejecución de función se mide en *GB por segundo*. El costo de ejecución se calcula mediante la combinación del uso de memoria con el tiempo de ejecución. Una función que se ejecuta durante más tiempo cuesta más, al igual que una función que consume más memoria. 

Considere un caso en el que la cantidad de memoria que usa la función permanece constante. En este caso, el cálculo del costo es una multiplicación simple. Por ejemplo, suponga que la función ha consumido 0,5 GB durante tres segundos. Después, el costo de ejecución es `0.5GB * 3s = 1.5 GB-seconds`. 

Dado que el uso de la memoria cambia con el tiempo, el cálculo es esencialmente la parte integral del uso de la memoria a lo largo del tiempo.  El sistema realiza este cálculo mediante el muestreo del uso de memoria del proceso (junto con procesos secundarios) a intervalos regulares. Como se mencionó en la [página de precios], el uso de memoria se redondea al cubo de 128 MB más próximo. Cuando el proceso use 160 MB, se le cobrará por 256 MB. El cálculo tiene en cuenta la simultaneidad, que son varias ejecuciones de funciones simultáneas en el mismo proceso.

> [!NOTE]
> Aunque el uso de CPU no se considera directamente en el costo de la ejecución, puede tener un impacto en el costo cuando afecta al tiempo de ejecución de la función.

## <a name="other-related-costs"></a>Otros costos relacionados

Al calcular el costo general de la ejecución de las funciones en cualquier plan, recuerde que el entorno en tiempo de ejecución de Functions usa otros servicios de Azure, que se facturan por separado. Al calcular los precios de las aplicaciones de funciones, los desencadenadores y enlaces que tenga que se integren con otros servicios de Azure requieren que cree y pague por estos servicios adicionales. 

En el caso de las funciones que se ejecutan en un plan de consumo, el costo total es el costo de ejecución de las funciones, más el costo del ancho de banda y servicios adicionales. 

Al calcular los costos generales de la aplicación de funciones y los servicios relacionados, use la [calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/?service=functions). 

| Costo relacionado | Descripción |
| ------------ | ----------- |
| **Cuenta de almacenamiento** | Cada aplicación de funciones requiere que tenga una [cuenta de almacenamiento de Azure](../storage/common/storage-introduction.md#types-of-storage-accounts) de uso general asociada, que se [factura por separado](https://azure.microsoft.com/pricing/details/storage/). El tiempo de ejecución de Functions usa esta cuenta internamente, pero también se puede usar para los desencadenadores y enlaces de almacenamiento. Si no tiene una cuenta de almacenamiento, se crea una automáticamente cuando se crea la aplicación de funciones. Para más información, consulte [Requisitos de la cuenta de almacenamiento](storage-considerations.md#storage-account-requirements).|
| **Application Insights** | Functions se basa en [Application Insights](../azure-monitor/app/app-insights-overview.md) para proporcionar una experiencia de supervisión de alto rendimiento para las aplicaciones de funciones. Aunque no es necesario, debe [habilitar la integración de Application Insights](functions-monitoring.md#enable-application-insights-integration). Cada mes se incluye una concesión gratuita de datos de telemetría. Para más información, consulte la [página de precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/). |
| **Ancho de banda de la red** | No tiene que pagar por las transferencias de datos entre los servicios de Azure de la misma región. Sin embargo, puede incurrir en costos de transferencia de datos de salida a otra región o fuera de Azure. Para más información, consulte [Detalles de precios de ancho de banda](https://azure.microsoft.com/pricing/details/bandwidth/). |

## <a name="behaviors-affecting-execution-time"></a>Comportamientos que afectan al tiempo de ejecución

Los siguientes comportamientos de las funciones pueden afectar al tiempo de ejecución:

+ **Desencadenadores y enlaces**: El tiempo necesario para leer la entrada y escribir la salida en los [enlaces de funciones](functions-triggers-bindings.md) se cuenta como tiempo de ejecución. Por ejemplo, cuando la función usa un enlace de salida para escribir un mensaje en una cola de almacenamiento de Azure, el tiempo de ejecución incluye el tiempo necesario para escribir el mensaje en la cola, que se incluye en el cálculo del costo de la función. 

+ **Ejecución asincrónica**: El tiempo que la función espera los resultados de una solicitud asincrónica (`await` en C#) se cuenta como tiempo de ejecución. El cálculo de GB por segundo se basa en la hora de inicio y finalización de la función y el uso de memoria durante ese período. Lo que sucede durante ese tiempo en términos de actividad de la CPU no se factoriza en el cálculo. Es posible que pueda reducir los costos durante las operaciones asincrónicas mediante el uso de [Durable Functions](durable/durable-functions-overview.md). No se le facturará por el tiempo de espera en las funciones del orquestador.

## <a name="view-execution-data"></a>Visualización del plan de ejecución

En la [factura](../cost-management-billing/understand/download-azure-invoice.md), puede ver los datos relacionados con los costos del **total de ejecuciones: Funciones** y el **tiempo de ejecución: Funciones**, junto con los costos facturados reales. Sin embargo, estos datos de factura son un agregado mensual para un período de facturación anterior. 

Para comprender mejor el impacto en el costo de las funciones, puede usar Azure Monitor para ver las métricas relacionadas con los costos que se están generando actualmente en las aplicaciones de funciones. Puede usar el [explorador de métricas Azure Monitor](../azure-monitor/platform/metrics-getting-started.md) en [Azure Portal] o las API REST para obtener estos datos.

### <a name="monitor-metrics-explorer"></a>Explorador de métricas de Monitor

Use el [explorador de métricas de Azure Monitor](../azure-monitor/platform/metrics-getting-started.md) para ver los datos relacionados con los costos de las aplicaciones de funciones del plan de consumo en un formato gráfico. 

1. En la parte superior de [Azure Portal], en **Buscar recursos, servicios y documentos**, busque `monitor` y seleccione **Monitor** en **Servicios**.

1. A la izquierda, seleccione **Métricas** > **Seleccionar un recurso** y, después, use la configuración que se encuentra debajo de la imagen para elegir la aplicación de funciones.

    ![Selección del recurso de la aplicación de funciones](media/functions-consumption-costing/select-a-resource.png)

      
    |Configuración  |Valor sugerido  |Descripción  |
    |---------|---------|---------|
    | Subscription    |  Su suscripción  | La suscripción con la aplicación de funciones.  |
    | Resource group     | Su grupo de recursos  | El grupo de recursos que contiene la aplicación de funciones.   |
    | Tipo de recurso     |  Servicios de aplicaciones | Las aplicaciones de funciones se muestran como instancias de App Services en Monitor. |
    | Resource     |  Su aplicación de función  | La aplicación de funciones que se va a supervisar.        |

1. Seleccione **Aplicar** para elegir la aplicación de funciones como el recurso que se va a supervisar.

1. En **Métrica**, elija **Recuento de la ejecución de la función** y **Suma** en **Agregación**. Esto agrega al gráfico la suma de los recuentos de ejecución durante el período elegido.

    ![Definición de una métrica de aplicación de funciones para agregarla al gráfico](media/functions-consumption-costing/monitor-metrics-add-metric.png)

1. Seleccione **Agregar métrica** y repita los pasos 2 a 4 para agregar **Unidades de ejecución de la función** al gráfico. 

El gráfico resultante contiene los totales de ambas métricas de ejecución en el intervalo de tiempo elegido, que en este caso es de dos horas.

![Gráfico de recuentos de ejecución de funciones y unidades de ejecución](media/functions-consumption-costing/monitor-metrics-execution-sum.png)

Dado que el número de unidades de ejecución es mucho mayor que el recuento de ejecuciones, el gráfico simplemente muestra las unidades de ejecución.

En este gráfico se muestra un total de 1,11 mil millones de `Function Execution Units` consumidas en un período de dos horas, medido en MB por milisegundo. Para convertir a GB por segundo, divídalo por 1 024 000. En este ejemplo, la aplicación de funciones ha consumido `1110000000 / 1024000 = 1083.98` GB por segundo. Puede tomar este valor y multiplicar por el precio actual del tiempo de ejecución en la [página de precios de Functions][página de precios], que le proporciona el costo de estas dos horas, suponiendo que ya ha usado cualquier concesión gratuita de tiempo de ejecución. 

### <a name="azure-cli"></a>Azure CLI

La [CLI de Azure](/cli/azure/) tiene comandos para recuperar métricas. Puede usar la CLI desde un entorno de comandos local o directamente desde el portal mediante [Azure Cloud Shell](../cloud-shell/overview.md). Por ejemplo, el comando siguiente [az monitor metrics list](/cli/azure/monitor/metrics#az-monitor-metrics-list) devuelve datos por hora durante el mismo período de tiempo utilizado antes.

Asegúrese de reemplazar `<AZURE_SUBSCRIPTON_ID>` por el identificador de suscripción de Azure que ejecuta el comando.

```azurecli-interactive
az monitor metrics list --resource /subscriptions/<AZURE_SUBSCRIPTION_ID>/resourceGroups/metrics-testing-consumption/providers/Microsoft.Web/sites/metrics-testing-consumption --metric FunctionExecutionUnits,FunctionExecutionCount --aggregation Total --interval PT1H --start-time 2019-09-11T21:46:00Z --end-time 2019-09-11T23:18:00Z
```

Este comando devuelve una carga JSON similar a la del ejemplo siguiente:

```json
{
  "cost": 0.0,
  "interval": "1:00:00",
  "namespace": "Microsoft.Web/sites",
  "resourceregion": "centralus",
  "timespan": "2019-09-11T21:46:00Z/2019-09-11T23:18:00Z",
  "value": [
    {
      "id": "/subscriptions/XXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXX/resourceGroups/metrics-testing-consumption/providers/Microsoft.Web/sites/metrics-testing-consumption/providers/Microsoft.Insights/metrics/FunctionExecutionUnits",
      "name": {
        "localizedValue": "Function Execution Units",
        "value": "FunctionExecutionUnits"
      },
      "resourceGroup": "metrics-testing-consumption",
      "timeseries": [
        {
          "data": [
            {
              "average": null,
              "count": null,
              "maximum": null,
              "minimum": null,
              "timeStamp": "2019-09-11T21:46:00+00:00",
              "total": 793294592.0
            },
            {
              "average": null,
              "count": null,
              "maximum": null,
              "minimum": null,
              "timeStamp": "2019-09-11T22:46:00+00:00",
              "total": 316576256.0
            }
          ],
          "metadatavalues": []
        }
      ],
      "type": "Microsoft.Insights/metrics",
      "unit": "Count"
    },
    {
      "id": "/subscriptions/XXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXX/resourceGroups/metrics-testing-consumption/providers/Microsoft.Web/sites/metrics-testing-consumption/providers/Microsoft.Insights/metrics/FunctionExecutionCount",
      "name": {
        "localizedValue": "Function Execution Count",
        "value": "FunctionExecutionCount"
      },
      "resourceGroup": "metrics-testing-consumption",
      "timeseries": [
        {
          "data": [
            {
              "average": null,
              "count": null,
              "maximum": null,
              "minimum": null,
              "timeStamp": "2019-09-11T21:46:00+00:00",
              "total": 33538.0
            },
            {
              "average": null,
              "count": null,
              "maximum": null,
              "minimum": null,
              "timeStamp": "2019-09-11T22:46:00+00:00",
              "total": 13040.0
            }
          ],
          "metadatavalues": []
        }
      ],
      "type": "Microsoft.Insights/metrics",
      "unit": "Count"
    }
  ]
}
```
Esta respuesta determinada muestra que, de `2019-09-11T21:46` a `2019-09-11T23:18`, la aplicación ha consumido 1,11 mil millones de MB por milisegundo (1083,98 GB por segundo).

## <a name="determine-memory-usage"></a>Determinación del uso de memoria

Las unidades de ejecución de la función son una combinación de tiempo de ejecución y de uso de memoria, lo que hace que sea una métrica difícil de entender. Los datos de memoria no son una métrica disponible actualmente en Azure Monitor. Sin embargo, si desea optimizar el uso de memoria de la aplicación, puede utilizar los datos del contador de rendimiento recopilados por Application Insights.  

Si aún no lo ha hecho, [habilite Application Insights en la aplicación de funciones](functions-monitoring.md#enable-application-insights-integration). Con esta integración habilitada, puede [consultar estos datos de telemetría en el portal](functions-monitoring.md#query-telemetry-data).  

En **Supervisión**, seleccione **Registros (Analytics)** , copie la siguiente consulta de telemetría y péguela en la ventana de consulta y seleccione **Ejecutar**. Esta consulta devuelve el uso de memoria total en cada tiempo muestreado.

```
performanceCounters
| where name == "Private Bytes"
| project timestamp, name, value
```

Los resultados deben tener un aspecto similar al ejemplo siguiente:

| marca de tiempo \[UTC\]          | name          | value       |
|----------------------------|---------------|-------------|
| 9/12/2019, 1:05:14\.947 AM | Bytes privados | 209,932,288 |
| 9/12/2019, 1:06:14\.994 AM | Bytes privados | 212 189 184 |
| 9/12/2019, 1:06:30\.010 AM | Bytes privados | 231,714,816 |
| 9/12/2019, 1:07:15\.040 AM | Bytes privados | 210,591,744 |
| 9/12/2019, 1:12:16\.285 AM | Bytes privados | 216,285,184 |
| 9/12/2019, 1:12:31\.376 AM | Bytes privados | 235,806,720 |

## <a name="function-level-metrics"></a>Métricas de nivel de función

Azure Monitor realiza un seguimiento de las métricas en el nivel de recursos, que para en el caso de Functions es la aplicación de funciones. La integración de Application Insights emite métricas por función. Esta es una consulta de análisis de ejemplo para obtener la duración media de una función:

```
customMetrics
| where name contains "Duration"
| extend averageDuration = valueSum / valueCount
| summarize averageDurationMilliseconds=avg(averageDuration) by name
```

| name                       | averageDurationMilliseconds |
|----------------------------|-----------------------------|
| QueueTrigger AvgDurationMs | 16\.087                     |
| QueueTrigger MaxDurationMs | 90\.249                     |
| QueueTrigger MinDurationMs | 8\.522                      |

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre la supervisión de las aplicaciones de funciones](functions-monitoring.md)

[Página de precios]:https://azure.microsoft.com/pricing/details/functions/
[Azure Portal]: https://portal.azure.com
