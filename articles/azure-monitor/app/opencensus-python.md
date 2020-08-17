---
title: Supervisión de aplicaciones de Python con Azure Monitor (versión preliminar) | Microsoft Docs
description: Se proporcionan instrucciones para conectar Python de OpenCensus con Azure Monitor.
ms.topic: conceptual
author: lzchen
ms.author: lechen
ms.date: 10/11/2019
ms.reviewer: mbullwin
ms.custom: devx-track-python
ms.openlocfilehash: ac7569a48e4bab25d4db17f2fc6dd92b31afcab5
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87850056"
---
# <a name="set-up-azure-monitor-for-your-python-application"></a>Configuración de Azure Monitor para la aplicación de Python

Azure Monitor admite seguimiento distribuido, recopilación de métricas y registro de aplicaciones de Python gracias a la integración con [OpenCensus](https://opencensus.io). En este artículo, encontrará instrucciones para configurar OpenCensus para Python y enviar los datos de supervisión a Azure Monitor.

## <a name="prerequisites"></a>Requisitos previos

- Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
- Instalación de Python. En este artículo se usa [Python 3.7.0](https://www.python.org/downloads/release/python-370/), aunque es probable que las versiones anteriores funcionen con cambios menores. El SDK solo admite Python v2.7 y v3.4-v3.7.
- Cree un [recurso](./create-new-resource.md) de Application Insights. Se le asignará su propia clave de instrumentación (iKey) para el recurso.

## <a name="instrument-with-opencensus-python-sdk-for-azure-monitor"></a>Instrumentación con el SDK de Python para OpenCensus en Azure Monitor

Instale los exportadores de Azure Monitor de OpenCensus:

```console
python -m pip install opencensus-ext-azure
```

Para una lista completa de los paquetes y las integraciones, consulte [Paquetes de OpenCensus](./nuget.md#common-packages-for-python-using-opencensus).

> [!NOTE]
> El comando `python -m pip install opencensus-ext-azure` supone que tiene una variable de entorno `PATH` establecida para la instalación de Python. Si no ha configurado esta variable, deberá proporcionar la ruta de acceso completa del directorio a la ubicación del archivo ejecutable de Python. El resultado es un comando como el siguiente: `C:\Users\Administrator\AppData\Local\Programs\Python\Python37-32\python.exe -m pip install opencensus-ext-azure`.

El SDK emplea tres exportadores de Azure Monitor para enviar distintos tipos de telemetría a Azure Monitor. Son seguimiento, métricas y registros. Para obtener más información sobre estos tipos de telemetría, consulte [la información general sobre la plataforma de datos](../platform/data-platform.md). Utilice las instrucciones siguientes para enviar estos tipos de telemetría a través de los tres exportadores.

## <a name="telemetry-type-mappings"></a>Asignaciones de tipos de telemetría

Estos son los exportadores que proporciona OpenCensus asignados a los tipos de telemetría que se ven en Azure Monitor.

| Pilar de observabilidad | Tipo de telemetría en Azure Monitor.    | Explicación                                         |
|-------------------------|------------------------------------|-----------------------------------------------------|
| Registros                    | Seguimientos, excepciones, customEvents   | Telemetría de registro, telemetría de excepciones, telemetría de eventos |
| Métricas                 | customMetrics, performanceCounters | Contadores de rendimiento de métricas personalizadas                |
| Seguimiento                 | Dependencias de solicitudes             | Solicitudes entrantes, solicitudes salientes                |

### <a name="logs"></a>Registros

1. Primero, se van a generar algunos datos de registro locales.

    ```python
    import logging

    logger = logging.getLogger(__name__)

    def valuePrompt():
        line = input("Enter a value: ")
        logger.warning(line)

    def main():
        while True:
            valuePrompt()

    if __name__ == "__main__":
        main()
    ```

1. El código pedirá continuamente que se escriba un valor. Se emite una entrada de registro para cada valor especificado.

    ```output
    Enter a value: 24
    24
    Enter a value: 55
    55
    Enter a value: 123
    123
    Enter a value: 90
    90
    ```

1. Aunque introducir valores es útil con fines de demostración, la finalidad básica es emitir los datos del registro a Azure Monitor. Pase la cadena de conexión directamente al exportador. O bien, puede especificarlo en una variable de entorno `APPLICATIONINSIGHTS_CONNECTION_STRING`. Modifique el código del paso anterior en función del ejemplo de código siguiente:

    ```python
    import logging
    from opencensus.ext.azure.log_exporter import AzureLogHandler

    logger = logging.getLogger(__name__)

    # TODO: replace the all-zero GUID with your instrumentation key.
    logger.addHandler(AzureLogHandler(
        connection_string='InstrumentationKey=00000000-0000-0000-0000-000000000000')
    )

    def valuePrompt():
        line = input("Enter a value: ")
        logger.warning(line)

    def main():
        while True:
            valuePrompt()

    if __name__ == "__main__":
        main()
    ```

1. El exportador envía los datos de registro a Azure Monitor. Puede encontrar los datos en `traces`. 

    > [!NOTE]
    > En este contexto, `traces` no es lo mismo que `tracing`. Aquí, `traces` hace referencia al tipo de telemetría que verá en Azure Monitor al usar `AzureLogHandler`. Pero `tracing` hace referencia a un concepto de OpenCensus y se relaciona con el [seguimiento distribuido](./distributed-tracing.md).

    > [!NOTE]
    > El registrador raíz se configura con el nivel de advertencia. Esto significa que los registros que se envían con menos de una gravedad se omiten y, a su vez, no se enviarán a Azure Monitor. Para obtener más información, vea la [documentación](https://docs.python.org/3/library/logging.html#logging.Logger.setLevel).

1. También puede agregar propiedades personalizadas a los mensajes de registro del argumento de palabra clave *extra* mediante el campo custom_dimensions. Estas propiedades aparecen como pares de valor clave en el `customDimensions` de Azure Monitor.
    > [!NOTE]
    > Para que esta característica funcione, debe pasar un diccionario al campo custom_dimensions. Si se pasan argumentos de cualquier otro tipo, el registrador los omitirá.

    ```python
    import logging

    from opencensus.ext.azure.log_exporter import AzureLogHandler

    logger = logging.getLogger(__name__)
    # TODO: replace the all-zero GUID with your instrumentation key.
    logger.addHandler(AzureLogHandler(
        connection_string='InstrumentationKey=00000000-0000-0000-0000-000000000000')
    )

    properties = {'custom_dimensions': {'key_1': 'value_1', 'key_2': 'value_2'}}

    # Use properties in logging statements
    logger.warning('action', extra=properties)
    ```

#### <a name="configure-logging-for-django-applications"></a>Configuración del registro para aplicaciones de Django.

Puede configurar el registro de forma explícita en su código de aplicación, como en el caso de las aplicaciones de Django, o puede especificarlo en la configuración de registro de Django. Este código puede entrar en cualquier archivo que use para la configuración de los valores de Django. Para ver cómo configurar los valores de Django, consulte [valores de Django](https://docs.djangoproject.com/en/3.0/topics/settings/). Para más información sobre la configuración del registro para Log4j, consulte [registro de Django](https://docs.djangoproject.com/en/3.0/topics/logging/).

```json
LOGGING = {
    "handlers": {
        "azure": {
            "level": "DEBUG",
        "class": "opencensus.ext.azure.log_exporter.AzureLogHandler",
            "instrumentation_key": "<your-ikey-here>",
         },
        "console": {
            "level": "DEBUG",
            "class": "logging.StreamHandler",
            "stream": sys.stdout,
         },
      },
    "loggers": {
        "logger_name": {"handlers": ["azure", "console"]},
    },
}
```

Asegúrese de usar el registrador con el mismo nombre que el especificado en su configuración.

```python
import logging

logger = logging.getLogger("logger_name")
logger.warning("this will be tracked")
```

#### <a name="send-exceptions"></a>Enviar excepciones

OpenCensus Python no realiza de forma automática el seguimiento y el envío de la telemetría de `exception`. Se envían mediante `AzureLogHandler` por medio de excepciones a través de la biblioteca de registro de Python. Puede agregar propiedades personalizadas de la misma manera que con el registro normal.

```python
import logging

from opencensus.ext.azure.log_exporter import AzureLogHandler

logger = logging.getLogger(__name__)
# TODO: replace the all-zero GUID with your instrumentation key.
logger.addHandler(AzureLogHandler(
    connection_string='InstrumentationKey=00000000-0000-0000-0000-000000000000')
)

properties = {'custom_dimensions': {'key_1': 'value_1', 'key_2': 'value_2'}}

# Use properties in exception logs
try:
    result = 1 / 0  # generate a ZeroDivisionError
except Exception:
    logger.exception('Captured an exception.', extra=properties)
```
Es decisión del usuario cómo quiere registrar las excepciones no controladas, ya que debe registrar las excepciones de manera explícita. OpenCensus no impone restricciones en el modo en que un usuario quiera hacerlo, siempre y cuando registre explícitamente una telemetría de excepciones.

#### <a name="send-events"></a>Envío de eventos

Puede enviar la telemetría de `customEvent` exactamente del mismo modo que envía la telemetría de `trace`, excepto mediante el uso de `AzureEventHandler` en su lugar.

```python
import logging

from opencensus.ext.azure.log_exporter import AzureEventHandler

logger = logging.getLogger(__name__)
logger.addHandler(AzureEventHandler(connection_string='InstrumentationKey=<your-instrumentation_key-here>'))
logger.setLevel(logging.INFO)
logger.info('Hello, World!')
```

#### <a name="sampling"></a>muestreo

Para información sobre el muestreo en OpenCensus, eche un vistazo al [muestreo en OpenCensus](sampling.md#configuring-fixed-rate-sampling-for-opencensus-python-applications).

#### <a name="log-correlation"></a>Correlación de registros

Para más información sobre cómo enriquecer los registros con los datos de contexto de seguimiento, vea la [integración de registros](./correlation.md#log-correlation) de Python para OpenCensus.

#### <a name="modify-telemetry"></a>Modificación de la telemetría

Para obtener detalles sobre cómo modificar la telemetría sometida a seguimiento antes de enviarla a Azure Monitor, vea los [procesadores de telemetría](./api-filtering-sampling.md#opencensus-python-telemetry-processors) de Python para OpenCensus.


### <a name="metrics"></a>Métricas

1. Primero, se van a generar algunos datos de métrica locales. Se creará una métrica sencilla para realizar el seguimiento del número de veces que el usuario selecciona la tecla **Entrar**.

    ```python
    from datetime import datetime
    from opencensus.stats import aggregation as aggregation_module
    from opencensus.stats import measure as measure_module
    from opencensus.stats import stats as stats_module
    from opencensus.stats import view as view_module
    from opencensus.tags import tag_map as tag_map_module

    stats = stats_module.stats
    view_manager = stats.view_manager
    stats_recorder = stats.stats_recorder

    prompt_measure = measure_module.MeasureInt("prompts",
                                               "number of prompts",
                                               "prompts")
    prompt_view = view_module.View("prompt view",
                                   "number of prompts",
                                   [],
                                   prompt_measure,
                                   aggregation_module.CountAggregation())
    view_manager.register_view(prompt_view)
    mmap = stats_recorder.new_measurement_map()
    tmap = tag_map_module.TagMap()

    def prompt():
        input("Press enter.")
        mmap.measure_int_put(prompt_measure, 1)
        mmap.record(tmap)
        metrics = list(mmap.measure_to_view_map.get_metrics(datetime.utcnow()))
        print(metrics[0].time_series[0].points[0])

    def main():
        while True:
            prompt()

    if __name__ == "__main__":
        main()
    ```
1. Al ejecutar el código varias veces, se le pedirá que seleccione **Entrar**. Se crea una métrica para realizar el seguimiento del número de veces que se presiona la tecla **Entrar**. Con cada entrada, el valor se incrementará y la información de la métrica se mostrará en la consola. La información incluye el valor actual y la marca de tiempo actual en el momento en que se actualizó la métrica.

    ```output
    Press enter.
    Point(value=ValueLong(5), timestamp=2019-10-09 20:58:04.930426)
    Press enter.
    Point(value=ValueLong(6), timestamp=2019-10-09 20:58:06.570167)
    Press enter.
    Point(value=ValueLong(7), timestamp=2019-10-09 20:58:07.138614)
    ```

1. Aunque introducir valores es útil con fines de demostración, la finalidad básica es emitir los datos de la métrica a Azure Monitor. Pase la cadena de conexión directamente al exportador. O bien, puede especificarlo en una variable de entorno `APPLICATIONINSIGHTS_CONNECTION_STRING`. Modifique el código del paso anterior en función del ejemplo de código siguiente:

    ```python
    from datetime import datetime
    from opencensus.ext.azure import metrics_exporter
    from opencensus.stats import aggregation as aggregation_module
    from opencensus.stats import measure as measure_module
    from opencensus.stats import stats as stats_module
    from opencensus.stats import view as view_module
    from opencensus.tags import tag_map as tag_map_module

    stats = stats_module.stats
    view_manager = stats.view_manager
    stats_recorder = stats.stats_recorder

    prompt_measure = measure_module.MeasureInt("prompts",
                                               "number of prompts",
                                               "prompts")
    prompt_view = view_module.View("prompt view",
                                   "number of prompts",
                                   [],
                                   prompt_measure,
                                   aggregation_module.CountAggregation())
    view_manager.register_view(prompt_view)
    mmap = stats_recorder.new_measurement_map()
    tmap = tag_map_module.TagMap()

    # TODO: replace the all-zero GUID with your instrumentation key.
    exporter = metrics_exporter.new_metrics_exporter(
        connection_string='InstrumentationKey=00000000-0000-0000-0000-000000000000')

    view_manager.register_exporter(exporter)

    def prompt():
        input("Press enter.")
        mmap.measure_int_put(prompt_measure, 1)
        mmap.record(tmap)
        metrics = list(mmap.measure_to_view_map.get_metrics(datetime.utcnow()))
        print(metrics[0].time_series[0].points[0])

    def main():
        while True:
            prompt()

    if __name__ == "__main__":
        main()
    ```

1. El exportador envía los datos de métricas a Azure Monitor según un intervalo fijo. El valor predeterminado es cada 15 segundos. Se está realizando el seguimiento de una sola métrica. Por lo tanto, los datos de esta métrica se envían en cada intervalo con el valor y la marca de tiempo que contengan. Puede encontrar los datos en `customMetrics`.

#### <a name="performance-counters"></a>Contadores de rendimiento

De forma predeterminada, el exportador de métricas envía un conjunto de contadores de rendimiento a Azure Monitor. Puede deshabilitar este comportamiento si establece la marca `enable_standard_metrics` en `False` en el constructor del exportador de métricas.

```python
...
exporter = metrics_exporter.new_metrics_exporter(
  enable_standard_metrics=False,
  connection_string='InstrumentationKey=<your-instrumentation-key-here>')
...
```

Estos contadores de rendimiento se envían actualmente:

- Memoria disponible (bytes)
- Tiempo de procesador de CPU (porcentaje)
- Tasa de solicitudes entrantes (por segundo)
- Tiempo de ejecución promedio de solicitudes entrantes (milisegundos)
- Uso de CPU de proceso (porcentaje)
- Bytes privados del proceso (bytes)

Debería poder ver estas métricas en `performanceCounters`. Para más información, consulte [Contadores de rendimiento](./performance-counters.md).

#### <a name="modify-telemetry"></a>Modificación de la telemetría

Para obtener detalles sobre cómo modificar la telemetría sometida a seguimiento antes de enviarla a Azure Monitor, vea los [procesadores de telemetría](./api-filtering-sampling.md#opencensus-python-telemetry-processors) de Python para OpenCensus.

### <a name="tracing"></a>Seguimiento

> [!NOTE]
> En OpenCensus, `tracing` hace referencia al [seguimiento distribuido](./distributed-tracing.md). `AzureExporter` envía datos de telemetría de `requests` y `dependency` a Azure Monitor.

1. En primer lugar, vamos a generar algunos datos de seguimiento localmente. En Python IDLE, o en el editor que prefiera, escriba el código siguiente:

    ```python
    from opencensus.trace.samplers import ProbabilitySampler
    from opencensus.trace.tracer import Tracer

    tracer = Tracer(sampler=ProbabilitySampler(1.0))

    def valuePrompt():
        with tracer.span(name="test") as span:
            line = input("Enter a value: ")
            print(line)

    def main():
        while True:
            valuePrompt()

    if __name__ == "__main__":
        main()
    ```

1. Al ejecutar el código varias veces, se le pedirá que introduzca un valor. Con cada entrada, el valor se imprime en el shell. El módulo de Python para OpenCensus genera una parte correspondiente de `SpanData`. El proyecto OpenCensus define un [seguimiento como un árbol de intervalos](https://opencensus.io/core-concepts/tracing/).
    
    ```output
    Enter a value: 4
    4
    [SpanData(name='test', context=SpanContext(trace_id=8aa41bc469f1a705aed1bdb20c342603, span_id=None, trace_options=TraceOptions(enabled=True), tracestate=None), span_id='15ac5123ac1f6847', parent_span_id=None, attributes=BoundedDict({}, maxlen=32), start_time='2019-06-27T18:21:22.805429Z', end_time='2019-06-27T18:21:44.933405Z', child_span_count=0, stack_trace=None, annotations=BoundedList([], maxlen=32), message_events=BoundedList([], maxlen=128), links=BoundedList([], maxlen=32), status=None, same_process_as_parent_span=None, span_kind=0)]
    Enter a value: 25
    25
    [SpanData(name='test', context=SpanContext(trace_id=8aa41bc469f1a705aed1bdb20c342603, span_id=None, trace_options=TraceOptions(enabled=True), tracestate=None), span_id='2e512f846ba342de', parent_span_id=None, attributes=BoundedDict({}, maxlen=32), start_time='2019-06-27T18:21:44.933405Z', end_time='2019-06-27T18:21:46.156787Z', child_span_count=0, stack_trace=None, annotations=BoundedList([], maxlen=32), message_events=BoundedList([], maxlen=128), links=BoundedList([], maxlen=32), status=None, same_process_as_parent_span=None, span_kind=0)]
    Enter a value: 100
    100
    [SpanData(name='test', context=SpanContext(trace_id=8aa41bc469f1a705aed1bdb20c342603, span_id=None, trace_options=TraceOptions(enabled=True), tracestate=None), span_id='f3f9f9ee6db4740a', parent_span_id=None, attributes=BoundedDict({}, maxlen=32), start_time='2019-06-27T18:21:46.157732Z', end_time='2019-06-27T18:21:47.269583Z', child_span_count=0, stack_trace=None, annotations=BoundedList([], maxlen=32), message_events=BoundedList([], maxlen=128), links=BoundedList([], maxlen=32), status=None, same_process_as_parent_span=None, span_kind=0)]
    ```

1. Aunque introducir valores es útil con fines de demostración, la finalidad básica es emitir `SpanData` a Azure Monitor. Pase la cadena de conexión directamente al exportador. O bien, puede especificarlo en una variable de entorno `APPLICATIONINSIGHTS_CONNECTION_STRING`. Modifique el código del paso anterior en función del ejemplo de código siguiente:

    ```python
    from opencensus.ext.azure.trace_exporter import AzureExporter
    from opencensus.trace.samplers import ProbabilitySampler
    from opencensus.trace.tracer import Tracer

    # TODO: replace the all-zero GUID with your instrumentation key.
    tracer = Tracer(
        exporter=AzureExporter(
            connection_string='InstrumentationKey=00000000-0000-0000-0000-000000000000'),
        sampler=ProbabilitySampler(1.0),
    )

    def valuePrompt():
        with tracer.span(name="test") as span:
            line = input("Enter a value: ")
            print(line)

    def main():
        while True:
            valuePrompt()

    if __name__ == "__main__":
        main()
    ```

1. Ahora, al ejecutar el script de Python, todavía se le seguirá pidiendo que especifique los valores, pero en el shell solo se imprime el valor. El valor de `SpanData` creado se envía a Azure Monitor. Puede encontrar los datos del intervalo emitidos en `dependencies`. Para obtener más información sobre las solicitudes de salida, vea las [dependencias](./opencensus-python-dependency.md) de Python para OpenCensus.
Para obtener más información sobre las solicitudes de entrada, vea las [solicitudes](./opencensus-python-request.md) de Python para OpenCensus.

#### <a name="sampling"></a>muestreo

Para información sobre el muestreo en OpenCensus, eche un vistazo al [muestreo en OpenCensus](sampling.md#configuring-fixed-rate-sampling-for-opencensus-python-applications).

#### <a name="trace-correlation"></a>Correlación de seguimiento

Para obtener más información sobre la correlación de telemetría de los datos de seguimiento, eche un vistazo a la [correlación de telemetría](./correlation.md#telemetry-correlation-in-opencensus-python) de Python para OpenCensus.

#### <a name="modify-telemetry"></a>Modificación de la telemetría

Para obtener más información sobre cómo modificar la telemetría sometida a seguimiento antes de enviarla a Azure Monitor, vea los [procesadores de telemetría](./api-filtering-sampling.md#opencensus-python-telemetry-processors) de Python para OpenCensus.

## <a name="configure-azure-monitor-exporters"></a>Configuración de los exportadores de Azure Monitor

Como se muestra, hay tres exportadores de Azure Monitor diferentes que admiten OpenCensus. Cada uno envía distintos tipos de telemetría a Azure Monitor. Para ver qué tipos de telemetría envía cada exportador, consulte la lista siguiente.

Cada exportador acepta los mismos argumentos para la configuración, que se pasan a través de los constructores. Aquí puede ver los detalles de cada uno de ellos:

- `connection_string`: La cadena de conexión usada para conectarse a su recurso de Azure Monitor. Tiene prioridad sobre `instrumentation_key`.
- `enable_standard_metrics`: Usado para `AzureMetricsExporter`. Indica al exportador que envíe las métricas de los [contadores de rendimiento](../platform/app-insights-metrics.md#performance-counters) de forma automática a Azure Monitor. Su valor predeterminado es `True`.
- `export_interval`: Se utiliza para especificar la frecuencia en segundos de exportación.
- `instrumentation_key`: La clave de instrumentación utilizada para conectarse al recurso de Azure Monitor.
- `logging_sampling_rate`: Usado para `AzureLogHandler`. Proporciona una frecuencia de muestreo [0, 1.0] para exportar registros. Su valor predeterminado es 1.0.
- `max_batch_size`: Especifica el tamaño máximo de la telemetría que se exporta a la vez.
- `proxies`: Especifica una secuencia de servidores proxy que se va a usar para enviar datos a Azure Monitor. Para obtener más información, consulte las [proxies](https://requests.readthedocs.io/en/master/user/advanced/#proxies).
- `storage_path`: Una ruta de acceso a la ubicación de la carpeta de almacenamiento local (telemetría sin enviar). A partir del v1.0.3 de `opencensus-ext-azure`, la ruta de acceso predeterminada es el directorio temporal del sistema operativo + `opencensus-python` + `your-ikey`. Antes de v1.0.3, la ruta de acceso predeterminada es $USER + `.opencensus` + `.azure` + `python-file-name`.

## <a name="view-your-data-with-queries"></a>Visualización de los datos con consultas

Puede ver los datos de telemetría que se enviaron desde la aplicación mediante la pestaña **Registros (Analytics)** .

![Captura de pantalla del panel de información general con la pestaña "Registros (Analytics)" seleccionada en el cuadro rojo.](./media/opencensus-python/0010-logs-query.png)

En la lista de **Activo**:

- En el caso de la telemetría enviada con el exportador de seguimiento de Azure Monitor, las solicitudes entrantes se muestran en `requests`. Las solicitudes salientes o en proceso se muestran en `dependencies`.
- En el caso de la telemetría enviada con el exportador de métricas de Azure Monitor, las métricas enviadas se muestran en `customMetrics`.
- En el caso de la telemetría enviada con el exportador de registros de Azure Monitor, los registros se muestran en `traces`. Las excepciones aparecen en `exceptions`.

Para obtener información más detallada sobre cómo usar las consultas y los registros, consulte [Registros en Azure Monitor](../platform/data-platform-logs.md).

## <a name="learn-more-about-opencensus-for-python"></a>Más información sobre OpenCensus para Python

* [Python de OpenCensus en GitHub](https://github.com/census-instrumentation/opencensus-python)
* [Personalización](https://github.com/census-instrumentation/opencensus-python/blob/master/README.rst#customization)
* [Exportadores de Azure Monitor en GitHub](https://github.com/census-instrumentation/opencensus-python/tree/master/contrib/opencensus-ext-azure)
* [Integraciones de OpenCensus](https://github.com/census-instrumentation/opencensus-python#extensions)
* [Aplicaciones de ejemplo de Azure Monitor](https://github.com/Azure-Samples/azure-monitor-opencensus-python)

## <a name="next-steps"></a>Pasos siguientes

* [Seguimiento de las solicitudes entrantes](./opencensus-python-dependency.md)
* [Seguimiento de las solicitudes salientes](./opencensus-python-request.md)
* [Mapa de aplicación](./app-map.md)
* [Supervisión del rendimiento de un extremo a otro](../learn/tutorial-performance.md)

### <a name="alerts"></a>Alertas

* [Pruebas de disponibilidad](./monitor-web-app-availability.md): cree pruebas para asegurarse de que el sitio sea visible en la Web.
* [Diagnósticos inteligentes](./proactive-diagnostics.md): estas pruebas se realizan automáticamente, por lo que no es preciso hacer nada para configurarlas. Le indican si la aplicación tiene una tasa de solicitudes con error inusual.
* [Alertas de métricas](../platform/alerts-log.md): Establezca alertas que le adviertan si una métrica supera un umbral. Puede establecerlas en las métricas personalizadas que codifique en la aplicación.

