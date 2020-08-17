---
title: Seguimiento de solicitudes entrantes en Azure Application Insights con OpenCensus para Python | Microsoft Docs
description: Supervise las llamadas de solicitud de las aplicaciones Python a través de OpenCensus para Python.
ms.topic: conceptual
author: lzchen
ms.author: lechen
ms.date: 10/15/2019
ms.custom: devx-track-python
ms.openlocfilehash: c94bc949f13ee19a9d2150c9d3c1b6a2bdb959b2
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87850073"
---
# <a name="track-incoming-requests-with-opencensus-python"></a>Seguimiento de las solicitudes entrantes con OpenCensus para Python

Los datos de solicitudes entrantes se recopilan con OpenCensus para Python y sus diversas integraciones. Realice el seguimiento de los datos de solicitudes entrantes enviados a las aplicaciones web creadas sobre los marcos web más populares `django`, `flask` y `pyramid`. Los datos se envían después a Application Insights en Azure Monitor como telemetría `requests`.

En primer lugar, instrumente la aplicación con el [SDK de OpenCensus para Python](./opencensus-python.md) más reciente.

## <a name="tracking-django-applications"></a>Seguimiento de aplicaciones de Django

1. Descargue e instale `opencensus-ext-django` de [PyPI](https://pypi.org/project/opencensus-ext-django/) e instrumente la aplicación con el middleware de `django`. Se realizará el seguimiento de las solicitudes entrantes enviadas a la aplicación `django`.

2. Incluya `opencensus.ext.django.middleware.OpencensusMiddleware` en el archivo `settings.py` en `MIDDLEWARE`.

    ```python
    MIDDLEWARE = (
        ...
        'opencensus.ext.django.middleware.OpencensusMiddleware',
        ...
    )
    ```

3. Asegúrese de que AzureExporter está configurado correctamente en `settings.py` en `OPENCENSUS`. Para las solicitudes de direcciones URL de las que no quiera realizar el seguimiento, agréguelas a `BLACKLIST_PATHS`.

    ```python
    OPENCENSUS = {
        'TRACE': {
            'SAMPLER': 'opencensus.trace.samplers.ProbabilitySampler(rate=1)',
            'EXPORTER': '''opencensus.ext.azure.trace_exporter.AzureExporter(
                connection_string="InstrumentationKey=<your-ikey-here>"
            )''',
            'BLACKLIST_PATHS': ['https://example.com'],  <--- These sites will not be traced if a request is sent to it.
        }
    }
    ```

## <a name="tracking-flask-applications"></a>Seguimiento de aplicaciones de Flask

1. Descargue e instale `opencensus-ext-flask` de [PyPI](https://pypi.org/project/opencensus-ext-flask/) e instrumente la aplicación con el middleware de `flask`. Se realizará el seguimiento de las solicitudes entrantes enviadas a la aplicación `flask`.

    ```python
    
    from flask import Flask
    from opencensus.ext.azure.trace_exporter import AzureExporter
    from opencensus.ext.flask.flask_middleware import FlaskMiddleware
    from opencensus.trace.samplers import ProbabilitySampler
    
    app = Flask(__name__)
    middleware = FlaskMiddleware(
        app,
        exporter=AzureExporter(connection_string="InstrumentationKey=<your-ikey-here>"),
        sampler=ProbabilitySampler(rate=1.0),
    )
    
    @app.route('/')
    def hello():
        return 'Hello World!'
    
    if __name__ == '__main__':
        app.run(host='localhost', port=8080, threaded=True)
    
    ```

2. También puede configurar la aplicación de `flask` mediante `app.config`. Para las solicitudes de direcciones URL de las que no quiera realizar el seguimiento, agréguelas a `BLACKLIST_PATHS`.

    ```python
    app.config['OPENCENSUS'] = {
        'TRACE': {
            'SAMPLER': 'opencensus.trace.samplers.ProbabilitySampler(rate=1.0)',
            'EXPORTER': '''opencensus.ext.azure.trace_exporter.AzureExporter(
                connection_string="InstrumentationKey=<your-ikey-here>",
            )''',
            'BLACKLIST_PATHS': ['https://example.com'],  <--- These sites will not be traced if a request is sent to it.
        }
    }
    ```

## <a name="tracking-pyramid-applications"></a>Seguimiento de aplicaciones de Pyramid

1. Descargue e instale `opencensus-ext-django` de [PyPI](https://pypi.org/project/opencensus-ext-pyramid/) e instrumente la aplicación con la interpolación de `pyramid`. Se realizará el seguimiento de las solicitudes entrantes enviadas a la aplicación `pyramid`.

    ```python
    def main(global_config, **settings):
        config = Configurator(settings=settings)
    
        config.add_tween('opencensus.ext.pyramid'
                         '.pyramid_middleware.OpenCensusTweenFactory')
    ```

2. Puede configurar la interpolación de `pyramid` directamente en el código. Para las solicitudes de direcciones URL de las que no quiera realizar el seguimiento, agréguelas a `BLACKLIST_PATHS`.

    ```python
    settings = {
        'OPENCENSUS': {
            'TRACE': {
                'SAMPLER': 'opencensus.trace.samplers.ProbabilitySampler(rate=1.0)',
                'EXPORTER': '''opencensus.ext.azure.trace_exporter.AzureExporter(
                    connection_string="InstrumentationKey=<your-ikey-here>",
                )''',
                'BLACKLIST_PATHS': ['https://example.com'],  <--- These sites will not be traced if a request is sent to it.
            }
        }
    }
    config = Configurator(settings=settings)
    ```

## <a name="next-steps"></a>Pasos siguientes

* [Mapa de aplicación](./app-map.md)
* [Disponibilidad](./monitor-web-app-availability.md)
* [Búsqueda](./diagnostic-search.md)
* [Consulta de Log (Analytics)](../log-query/log-query-overview.md)
* [Diagnósticos de transacción](./transaction-diagnostics.md)

