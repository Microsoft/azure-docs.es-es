---
title: Seguimiento de dependencia en Azure Application Insights con OpenCensus Python | Microsoft Docs
description: Supervise las llamadas de dependencia de las aplicaciones de Python a través de OpenCensus Python.
ms.topic: conceptual
author: lzchen
ms.author: lechen
ms.date: 10/15/2019
ms.custom: devx-track-python
ms.openlocfilehash: abfd5e104bd4854781a0d3c9d08544506279518a
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87850090"
---
# <a name="track-dependencies-with-opencensus-python"></a>Seguimiento de dependencias con OpenCensus Python

Una dependencia es un componente externo al que la aplicación llama. Los datos de dependencia se recopilan con OpenCensus Python y sus diversas integraciones. Los datos se envían después a Application Insights en Azure Monitor como telemetría `dependencies`.

En primer lugar, instrumente la aplicación con el [SDK de OpenCensus para Python](./opencensus-python.md) más reciente.

## <a name="in-process-dependencies"></a>Dependencias en proceso

El SDK de OpenCensus para Python para Azure Monitor le permite enviar datos de telemetría de dependencias "en proceso" (información y lógica que se produce dentro de la aplicación). Las dependencias en proceso tendrán el campo `type` como `INPROC` en Analytics.

```python
from opencensus.ext.azure.trace_exporter import AzureExporter
from opencensus.trace.samplers import ProbabilitySampler
from opencensus.trace.tracer import Tracer

tracer = Tracer(exporter=AzureExporter(connection_string="InstrumentationKey=<your-ikey-here>"), sampler=ProbabilitySampler(1.0))

with tracer.span(name='foo'): # <-- A dependency telemetry item will be sent for this span "foo"
    print('Hello, World!')
```

## <a name="dependencies-with-requests-integration"></a>Dependencias con la integración de "solicitudes"

Realice un seguimiento de las solicitudes salientes con la integración de `requests` de OpenCensus.

Descargue e instale `opencensus-ext-requests` desde [PyPI](https://pypi.org/project/opencensus-ext-requests/) y agréguelo a las integraciones de seguimiento. Se realizará un seguimiento de las solicitudes enviadas que usan la biblioteca de [solicitudes](https://pypi.org/project/requests/) de Python.

```python
import requests
from opencensus.ext.azure.trace_exporter import AzureExporter
from opencensus.trace import config_integration
from opencensus.trace.samplers import ProbabilitySampler
from opencensus.trace.tracer import Tracer

config_integration.trace_integrations(['requests'])  # <-- this line enables the requests integration

tracer = Tracer(exporter=AzureExporter(connection_string="InstrumentationKey=<your-ikey-here>"), sampler=ProbabilitySampler(1.0))

with tracer.span(name='parent'):
    response = requests.get(url='https://www.wikipedia.org/wiki/Rabbit') # <-- this request will be tracked
```

## <a name="dependencies-with-httplib-integration"></a>Dependencias con la integración de "httplib"

Realice un seguimiento de las solicitudes salientes con la integración de `httplib` de OpenCensus.

Descargue e instale `opencensus-ext-httplib` desde [PyPI](https://pypi.org/project/opencensus-ext-httplib/) y agréguelo a las integraciones de seguimiento. Se realizará un seguimiento de las solicitudes enviadas mediante [http.client](https://docs.python.org/3.7/library/http.client.html) para Python3 o [httplib](https://docs.python.org/2/library/httplib.html) para Python2.

```python
import http.client as httplib
from opencensus.ext.azure.trace_exporter import AzureExporter
from opencensus.trace import config_integration
from opencensus.trace.samplers import ProbabilitySampler
from opencensus.trace.tracer import Tracer

config_integration.trace_integrations(['httplib'])
conn = httplib.HTTPConnection("www.python.org")

tracer = Tracer(
    exporter=AzureExporter(),
    sampler=ProbabilitySampler(1.0)
)

conn.request("GET", "http://www.python.org", "", {})
response = conn.getresponse()
conn.close()
```

## <a name="dependencies-with-django-integration"></a>Dependencias con la integración de "django"

Realice un seguimiento de las solicitudes salientes de Django con la integración de `django` de OpenCensus.

> [!NOTE]
> Las únicas solicitudes de Django de salida de las que se realiza un seguimiento son las llamadas a una base de datos. Para las solicitudes realizadas a la aplicación de Django, consulte [solicitudes entrantes](./opencensus-python-request.md#tracking-django-applications).

Descargue e instale `opencensus-ext-django` desde [PyPI](https://pypi.org/project/opencensus-ext-django/) y agregue la siguiente línea a la sección `MIDDLEWARE` del archivo `settings.py` de Django.

```python
MIDDLEWARE = [
    ...
    'opencensus.ext.django.middleware.OpencensusMiddleware',
]
```

Se puede proporcionar una configuración adicional; consulte las [personalizaciones](https://github.com/census-instrumentation/opencensus-python#customization) para obtener una referencia completa.

```python
OPENCENSUS = {
    'TRACE': {
        'SAMPLER': 'opencensus.trace.samplers.ProbabilitySampler(rate=1)',
        'EXPORTER': '''opencensus.ext.azure.trace_exporter.AzureExporter(
            connection_string="InstrumentationKey=<your-ikey-here>"
        )''',
    }
}
```

## <a name="dependencies-with-mysql-integration"></a>Dependencias con la integración de "mysql"

Realice un seguimiento de las solicitudes salientes de MySQL con la integración de `mysql` de OpenCensus. Esta integración es compatible con la biblioteca de [MySQL-Connector](https://pypi.org/project/mysql-connector-python/).

Descargue e instale `opencensus-ext-mysql` desde [PyPI](https://pypi.org/project/opencensus-ext-mysql/) y agregue las líneas siguientes al código.

```python
from opencensus.trace import config_integration

config_integration.trace_integrations(['mysql'])
```

## <a name="dependencies-with-pymysql-integration"></a>Dependencias con la integración de "pymysql"

Realice un seguimiento de las dependencias de PyMySQL con la integración de `pymysql` de OpenCensus.

Descargue e instale `opencensus-ext-pymysql` desde [PyPI](https://pypi.org/project/opencensus-ext-pymysql/) y agregue las líneas siguientes al código.

```python
from opencensus.trace import config_integration

config_integration.trace_integrations(['pymysql'])
```

## <a name="dependencies-with-postgresql-integration"></a>Dependencias con la integración de "postgreSQL"

Realice un seguimiento de las dependencias de PostgreSQL con la integración de `postgresql` de OpenCensus. Esta integración es compatible con la biblioteca [psycopg2](https://pypi.org/project/psycopg2/).

Descargue e instale `opencensus-ext-postgresql` desde [PyPI](https://pypi.org/project/opencensus-ext-postgresql/) y agregue las líneas siguientes al código.

```python
from opencensus.trace import config_integration

config_integration.trace_integrations(['postgresql'])
```

## <a name="dependencies-with-pymongo-integration"></a>Dependencias con la integración de "pymongo"

Realice un seguimiento de las dependencias de MongoDB con la integración de `pymongo` de OpenCensus. Esta integración es compatible con la biblioteca [pymongo](https://pypi.org/project/pymongo/).

Descargue e instale `opencensus-ext-pymongo` desde [PyPI](https://pypi.org/project/opencensus-ext-pymongo/) y agregue las líneas siguientes al código.

```python
from opencensus.trace import config_integration

config_integration.trace_integrations(['pymongo'])
```

### <a name="dependencies-with-sqlalchemy-integration"></a>Dependencias con la integración de "sqlalchemy"

Realice un seguimiento de las dependencias mediante SQLAlchemy con la integración de `sqlalchemy` de OpenCensus. Esta integración realiza un seguimiento del uso del paquete [sqlalchemy](https://pypi.org/project/SQLAlchemy/), independientemente de la base de datos subyacente.

```python
from opencensus.trace import config_integration

config_integration.trace_integrations(['sqlalchemy'])
```

## <a name="next-steps"></a>Pasos siguientes

* [Mapa de aplicación](./app-map.md)
* [Disponibilidad](./monitor-web-app-availability.md)
* [Búsqueda](./diagnostic-search.md)
* [Consulta de Log (Analytics)](../log-query/log-query-overview.md)
* [Diagnósticos de transacción](./transaction-diagnostics.md)

