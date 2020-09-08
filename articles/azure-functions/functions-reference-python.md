---
title: Referencia para desarrolladores de Python para Azure Functions
description: Aprenda a desarrollar funciones con Python
ms.topic: article
ms.date: 12/13/2019
ms.custom: devx-track-python
ms.openlocfilehash: f9b81a7263dc9a1bdae9fd881519ac734da2c6bc
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88642204"
---
# <a name="azure-functions-python-developer-guide"></a>Guía de Azure Functions para desarrolladores de Python

Este artículo es una introducción al desarrollo de Azure Functions mediante Python. En lo que va a leer a continuación se supone que ya ha leído la [guía para desarrolladores de Azure Functions](functions-reference.md).

Para ver los proyectos de ejemplo de funciones independientes de Python, examine los [ejemplos de funciones de Python](/samples/browse/?products=azure-functions&languages=python).

## <a name="programming-model"></a>Modelo de programación

Azure Functions espera que una función sea un método sin estado de un script de Python que procese entradas y genere salidas. De forma predeterminada, el runtime espera que el modelo se implemente como un método global denominado `main()` en el archivo `__init__.py`. También puede [especificar un punto de entrada alternativo](#alternate-entry-point).

Los datos de los desencadenadores y enlaces se enlazan a la función a través de los atributos del método con la propiedad `name` definida en el archivo *function.json*. Por ejemplo, en el archivo _function.json_ siguiente se describe una función simple desencadenada por una solicitud HTTP denominada `req`:

:::code language="json" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-Python/function.json":::

Según esta definición, el archivo `__init__.py` que contiene el código de la función puede ser similar al siguiente ejemplo:

```python
def main(req):
    user = req.params.get('user')
    return f'Hello, {user}!'
```

También puede declarar de forma explícita los tipos de parámetros y el tipo de valor devuelto de la función mediante las anotaciones de tipos de Python. Esto le ayuda a usar las características de IntelliSense y Autocompletar proporcionadas por muchos editores de código de Python.

```python
import azure.functions


def main(req: azure.functions.HttpRequest) -> str:
    user = req.params.get('user')
    return f'Hello, {user}!'
```

Utilice las anotaciones de Python incluidas en el paquete [azure.functions.*](/python/api/azure-functions/azure.functions?view=azure-python) para enlazar las entradas y las salidas a los métodos.

## <a name="alternate-entry-point"></a>Punto de entrada alternativo

Puede cambiar el comportamiento predeterminado de una función si especifica opcionalmente las propiedades `scriptFile` y `entryPoint` en el archivo *function.json*. Por ejemplo, el archivo _function.json_ siguiente indica al runtime que use el método `customentry()` del archivo  _main.py_, como punto de entrada para la instancia de Azure Functions.

```json
{
  "scriptFile": "main.py",
  "entryPoint": "customentry",
  "bindings": [
      ...
  ]
}
```

## <a name="folder-structure"></a>Estructura de carpetas

La estructura de carpetas recomendada para un proyecto de Python en Azure Functions tiene la siguiente apariencia:

```
 __app__
 | - my_first_function
 | | - __init__.py
 | | - function.json
 | | - example.py
 | - my_second_function
 | | - __init__.py
 | | - function.json
 | - shared_code
 | | - my_first_helper_function.py
 | | - my_second_helper_function.py
 | - host.json
 | - requirements.txt
 | - Dockerfile
 tests
```
La carpeta de proyecto principal (\_\_app\_\_) puede contener los siguientes archivos:

* *local.settings.json*: se usa para almacenar la configuración y las cadenas de conexión de la aplicación cuando se ejecuta localmente. Este archivo no se publica en Azure. Para más información, consulte [local.settings.file](functions-run-local.md#local-settings-file).
* *requirements.txt*: contiene la lista de paquetes que se instalan al publicar en Azure.
* *host.json*: contiene las opciones de configuración global que afectan a todas las funciones de una aplicación de funciones. Este archivo se publica en Azure. No todas las opciones se admiten cuando se ejecuta localmente. Para más información, consulte [host.json](functions-host-json.md).
* *.funcignore*: (opcional) declara los archivos que no deben publicarse en Azure.
* *Dockerfile*: (Opcional) se usa al publicar el proyecto en un [contenedor personalizado](functions-create-function-linux-custom-image.md).

Cada función tiene su propio archivo de código y archivo de configuración de enlace (function.json).

Al implementar el proyecto en una aplicación de funciones de Azure, debe incluirse en el paquete todo el contenido de la carpeta principal del proyecto ( *\_\_app\_\_* ), pero no la propia carpeta. Se recomienda mantener las pruebas en una carpeta independiente de la carpeta del proyecto; en este ejemplo `tests`. Esto evita que se implemente código de prueba con la aplicación. Para más información, consulte [Pruebas unitarias](#unit-testing).

## <a name="import-behavior"></a>Comportamiento de la importación

Puede importar módulos en el código de su función mediante referencias absolutas o relativas explícitas. En el caso de la estructura de carpetas que se muestra arriba, las siguientes operaciones de importación funcionan desde el archivo de función *\_\_app\_\_\my\_first\_function\\_\_init\_\_.py*:

```python
from . import example #(explicit relative)
```

```python
from ..shared_code import my_first_helper_function #(explicit relative)
```

```python
from __app__ import shared_code #(absolute)
```

```python
import __app__.shared_code #(absolute)
```

Las siguientes operaciones de importación *no funcionan* desde ese mismo archivo:

```python
import example
```

```python
from example import some_helper_code
```

```python
import shared_code
```

El código compartido debe mantenerse en una carpeta independiente en *\_\_app\_\_* . Para hacer referencia a los módulos en la carpeta *shared\_code*, puede usar la sintaxis siguiente:

```python
from __app__.shared_code import my_first_helper_function
```

## <a name="triggers-and-inputs"></a>Desencadenadores y entradas

Las entradas se dividen en dos categorías dentro de Azure Functions: una entrada del desencadenador y otra adicional. Aunque son diferentes en el archivo `function.json`, se usan igual en el código de Python.  Las cadenas de conexión o los secretos de los orígenes de entrada y el desencadenador se asignan a valores en el archivo `local.settings.json` al ejecutarse localmente y a la configuración de la aplicación al ejecutarse en Azure.

Por ejemplo, el siguiente código muestra la diferencia entre las dos:

```json
// function.json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "req",
      "direction": "in",
      "type": "httpTrigger",
      "authLevel": "anonymous",
      "route": "items/{id}"
    },
    {
      "name": "obj",
      "direction": "in",
      "type": "blob",
      "path": "samples/{id}",
      "connection": "AzureWebJobsStorage"
    }
  ]
}
```

```json
// local.settings.json
{
  "IsEncrypted": false,
  "Values": {
    "FUNCTIONS_WORKER_RUNTIME": "python",
    "AzureWebJobsStorage": "<azure-storage-connection-string>"
  }
}
```

```python
# __init__.py
import azure.functions as func
import logging


def main(req: func.HttpRequest,
         obj: func.InputStream):

    logging.info(f'Python HTTP triggered function processed: {obj.read()}')
```

Cuando se invoca la función, la solicitud HTTP se pasa a la función como `req`. Se recuperará una entrada de Azure Blob Storage según el _identificador_ de la dirección URL de la ruta y estará disponible como `obj` en el cuerpo de la función.  Aquí, la cuenta de almacenamiento especificada es la cadena de conexión encontrada en la configuración de la aplicación AzureWebJobsStorage, que es la misma cuenta de almacenamiento que usa la aplicación de funciones.


## <a name="outputs"></a>Salidas

Las salidas se pueden expresar como valores devueltos y como parámetros de salida. Si hay una única salida, se recomienda usar el valor devuelto. Para varias salidas, deberá utilizar parámetros de salida.

Para usar el valor devuelto de una función como valor de un enlace de salida, la propiedad `name` del enlace debe establecerse como `$return` en `function.json`.

Si desea generar varias salidas, utilice el método `set()` que la interfaz [`azure.functions.Out`](/python/api/azure-functions/azure.functions.out?view=azure-python) ofrece para asignar un valor al enlace. Por ejemplo, la siguiente función puede insertar un mensaje en una cola y también devolver una respuesta HTTP.

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "req",
      "direction": "in",
      "type": "httpTrigger",
      "authLevel": "anonymous"
    },
    {
      "name": "msg",
      "direction": "out",
      "type": "queue",
      "queueName": "outqueue",
      "connection": "AzureWebJobsStorage"
    },
    {
      "name": "$return",
      "direction": "out",
      "type": "http"
    }
  ]
}
```

```python
import azure.functions as func


def main(req: func.HttpRequest,
         msg: func.Out[func.QueueMessage]) -> str:

    message = req.params.get('body')
    msg.set(message)
    return message
```

## <a name="logging"></a>Registro

El acceso al registrador del entorno de ejecución de Azure Functions está disponible a través de un controlador [`logging`](https://docs.python.org/3/library/logging.html#module-logging) raíz en la aplicación de función. Este registrador está asociado a Application Insights y permite marcar las advertencias y los errores detectados durante la ejecución de la función.

En el ejemplo siguiente se registra un mensaje de información cuando la función se invoca con un desencadenador HTTP.

```python
import logging


def main(req):
    logging.info('Python HTTP trigger function processed a request.')
```

Hay métodos de registro adicionales disponibles que permiten escribir en la consola en otros niveles de seguimiento:

| Método                 | Descripción                                |
| ---------------------- | ------------------------------------------ |
| **`critical(_message_)`**   | Escribe un mensaje con el nivel CRÍTICO en el registrador de raíz.  |
| **`error(_message_)`**   | Escribe un mensaje con el nivel ERROR en el registrador de raíz.    |
| **`warning(_message_)`**    | Escribe un mensaje con el nivel ADVERTENCIA en el registrador de raíz.  |
| **`info(_message_)`**    | Escribe un mensaje con el nivel INFO en el registrador de raíz.  |
| **`debug(_message_)`** | Escribe un mensaje con el nivel DEBUG en el registrador de raíz.  |

Para más información sobre el registro, consulte [Supervisión de Azure Functions](functions-monitoring.md).

## <a name="http-trigger-and-bindings"></a>Desencadenadores y enlaces HTTP

El desencadenador HTTP se define en el archivo function.json. El valor de `name` del enlace debe coincidir con el parámetro con nombre de la función.
En los ejemplos anteriores, se usa un nombre de enlace `req`. Este parámetro es un objeto [HttpRequest] y se devuelve un objeto [HttpResponse].

Desde el objeto [HttpRequest], puede obtener encabezados de solicitud, parámetros de consulta, parámetros de ruta y el cuerpo del mensaje.

El ejemplo siguiente es de la [plantilla de desencadenador HTTP para Python](https://github.com/Azure/azure-functions-templates/tree/dev/Functions.Templates/Templates/HttpTrigger-Python).

```python
def main(req: func.HttpRequest) -> func.HttpResponse:
    headers = {"my-http-header": "some-value"}

    name = req.params.get('name')
    if not name:
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            name = req_body.get('name')

    if name:
        return func.HttpResponse(f"Hello {name}!", headers=headers)
    else:
        return func.HttpResponse(
             "Please pass a name on the query string or in the request body",
             headers=headers, status_code=400
        )
```

En esta función, el valor del parámetro de consulta `name` se obtiene del parámetro `params` del objeto [HttpRequest]. El cuerpo del mensaje con codificación JSON se lee mediante el método `get_json`.

Del mismo modo, puede establecer `status_code` y `headers` para el mensaje de respuesta en el objeto [HttpResponse] devuelto.

## <a name="scaling-and-concurrency"></a>Escalado y simultaneidad

De forma predeterminada, Azure Functions supervisa automáticamente la carga en la aplicación y crea instancias de host adicionales para Python según sea necesario. Functions usa umbrales integrados (no configurables por el usuario) en diferentes tipos de desencadenadores para decidir cuándo se deben agregar instancias, como la antigüedad de los mensajes y el tamaño de la cola para QueueTrigger. Para más información, vea [Cómo funcionan los planes de consumo y Premium](functions-scale.md#how-the-consumption-and-premium-plans-work).

Este comportamiento de escalado es suficiente para muchas aplicaciones. Sin embargo, es posible que las aplicaciones con alguna de las siguientes características no se escalen de forma tan eficaz:

- La aplicación necesita administrar muchas invocaciones simultáneas.
- La aplicación procesa un gran número de eventos de E/S.
- La aplicación es dependiente de las operaciones de E/S.

En tales casos, puede mejorar aún más el rendimiento si emplea patrones asincrónicos y usa varios procesos de trabajo de lenguaje.

### <a name="async"></a>Async

Dado que Python es un entorno de ejecución de un solo subproceso, una instancia de host para Python solo puede procesar una invocación de función cada vez. En el caso de las aplicaciones que procesan un gran número de eventos de E/S o que son dependientes de las operaciones de E/S, puede mejorar el rendimiento mediante la ejecución de funciones de forma asincrónica.

Para ejecutar una función de forma asincrónica, use la instrucción `async def`, que ejecuta la función directamente con [asyncio](https://docs.python.org/3/library/asyncio.html):

```python
async def main():
    await some_nonblocking_socket_io_op()
```

Una función sin la palabra clave `async` se ejecuta de forma automática en un grupo de subprocesos de asyncio:

```python
# Runs in an asyncio thread-pool

def main():
    some_blocking_socket_io()
```

### <a name="use-multiple-language-worker-processes"></a>Uso de procesos de trabajo de varios lenguajes

De forma predeterminada, cada instancia de host de Functions tiene un único proceso de trabajo de lenguaje. Puede aumentar el número de procesos de trabajo por host (hasta 10) mediante la configuración de la aplicación [FUNCTIONS_WORKER_PROCESS_COUNT](functions-app-settings.md#functions_worker_process_count). Al hacerlo, Azure Functions intenta distribuir uniformemente las invocaciones de función simultáneas en estos trabajos.

FUNCTIONS_WORKER_PROCESS_COUNT se aplica a cada host que Functions crea al escalar horizontalmente la aplicación para satisfacer la demanda.

## <a name="context"></a>Context

Para obtener el contexto de invocación de una función durante la ejecución, incluya el argumento [`context`](/python/api/azure-functions/azure.functions.context?view=azure-python) en su firma.

Por ejemplo:

```python
import azure.functions


def main(req: azure.functions.HttpRequest,
         context: azure.functions.Context) -> str:
    return f'{context.invocation_id}'
```

La clase [**Context**](/python/api/azure-functions/azure.functions.context?view=azure-python) tiene los atributos de cadena siguientes:

`function_directory` El directorio en que se ejecuta la función.

`function_name` El nombre de la función.

`invocation_id` El identificador de la invocación de la función actual.

## <a name="global-variables"></a>Variables globales

No se garantiza la conservación del estado de la aplicación para las ejecuciones futuras. Sin embargo, Azure Functions Runtime suele reutilizar el mismo proceso para varias ejecuciones de la misma aplicación. Para almacenar en caché los resultados de un cálculo costoso, debe declararse como variable global.

```python
CACHED_DATA = None


def main(req):
    global CACHED_DATA
    if CACHED_DATA is None:
        CACHED_DATA = load_json()

    # ... use CACHED_DATA in code
```

## <a name="environment-variables"></a>Variables de entorno

En Functions, la [configuración de la aplicación](functions-app-settings.md), como las cadenas de conexión del servicio, se exponen como variables de entorno durante la ejecución. Puede acceder a esta configuración mediante la declaración de `import os` y el uso de `setting = os.environ["setting-name"]`.

En el siguiente ejemplo se obtiene la [configuración de la aplicación](functions-how-to-use-azure-function-app-settings.md#settings), con la clave denominada `myAppSetting`:

```python
import logging
import os
import azure.functions as func

def main(req: func.HttpRequest) -> func.HttpResponse:

    # Get the setting named 'myAppSetting'
    my_app_setting_value = os.environ["myAppSetting"]
    logging.info(f'My app setting value:{my_app_setting_value}')
```

Para el desarrollo local, la configuración de la aplicación se [mantiene en el archivo local.settings.json](functions-run-local.md#local-settings-file).

## <a name="python-version"></a>Versión de Python

Azure Functions admite las siguientes versiones de Python:

| Versión de Functions | Versiones de Python<sup>*</sup> |
| ----- | ----- |
| 3.x | 3.8<br/>3.7<br/>3.6 |
| 2.x | 3.7<br/>3.6 |

<sup>*</sup>Distribuciones oficiales de CPython

Para solicitar una versión específica de Python al crear la aplicación de funciones en Azure, use la opción `--runtime-version` del comando [`az functionapp create`](/cli/azure/functionapp#az-functionapp-create). La versión de tiempo de ejecución de Functions se establece mediante la opción `--functions-version`. La versión de Python se establece al crear la aplicación de funciones y no se puede cambiar.

Cuando se ejecuta localmente, el entorno de ejecución usa la versión de Python disponible.

## <a name="package-management"></a>Administración de paquetes

Al desarrollar localmente con Azure Functions Core Tools o Visual Studio Code, agregue los nombres y las versiones de los paquetes necesarios al archivo `requirements.txt` e instálelos mediante `pip`.

Por ejemplo, se puede usar el comando de archivo y pip siguiente para instalar el paquete `requests` desde PyPI.

```txt
requests==2.19.1
```

```bash
pip install -r requirements.txt
```

## <a name="publishing-to-azure"></a>Publicación en Azure

Cuando esté preparado para la publicación, asegúrese de que todas las dependencias disponibles públicamente están incluidas en el archivo requirements.txt, que se encuentra en la raíz del directorio del proyecto.

Los archivos de proyecto y las carpetas que se excluyen de la publicación, incluida la carpeta del entorno virtual, se enumeran en el archivo. funcignore.

Se admiten tres acciones de compilación para publicar el proyecto de Python en Azure: compilación remota, compilación local y compilaciones mediante dependencias personalizadas.

También puede usar Azure Pipelines para compilar las dependencias y publicarlas mediante la entrega continua (CD). Para más información, vea [Entrega continua con Azure DevOps](functions-how-to-azure-devops.md).

### <a name="remote-build"></a>Compilación remota

Cuando se usa la compilación remota, las dependencias restauradas en el servidor y las dependencias nativas coinciden con el entorno de producción. Esto da como resultado un paquete de implementación más pequeño para cargar. Use la compilación remota para desarrollar aplicaciones de Python en Windows. Si el proyecto tiene dependencias personalizadas, puede [usar la compilación remota con la dirección URL de índice adicional](#remote-build-with-extra-index-url).

las dependencias se obtienen de forma remota en función del contenido del archivo requirements.txt. La [compilación remota](functions-deployment-technologies.md#remote-build) es el método de compilación recomendado. De forma predeterminada, Azure Functions Core Tools solicita una compilación remota cuando se usa el siguiente comando [func azure functionapp publish](functions-run-local.md#publish) para publicar el proyecto de Python en Azure.

```bash
func azure functionapp publish <APP_NAME>
```

Reemplace `<APP_NAME>` por el nombre de la aplicación de función de Azure.

La [extensión de Azure Functions para Visual Studio Code](functions-create-first-function-vs-code.md#publish-the-project-to-azure) también solicita de forma predeterminada una compilación remota.

### <a name="local-build"></a>Compilación local

las dependencias se obtienen de forma local en función del contenido del archivo requirements.txt. Puede impedir que se lleve a cabo una compilación remota usando el siguiente comando [func azure functionapp publish](functions-run-local.md#publish) para publicar con una compilación local.

```command
func azure functionapp publish <APP_NAME> --build local
```

Reemplace `<APP_NAME>` por el nombre de la aplicación de función de Azure.

Con la opción `--build local`, las dependencias del proyecto se leen del archivo requirements.txt y los paquetes dependientes se descargan e instalan localmente. Los archivos de proyecto y las dependencias se implementan desde el equipo local en Azure. Esto hace que se cargue un paquete de implementación más grande en Azure. Si, por alguna razón, Core Tools no puede obtener las dependencias del archivo requirements.txt, debe usar la opción de dependencias personalizadas para la publicación.

No se recomienda usar compilaciones locales al desarrollar localmente en Windows.

### <a name="custom-dependencies"></a>Dependencias personalizadas

Cuando el proyecto tiene dependencias que no se encuentran en el [índice de paquetes de Python](https://pypi.org/), hay dos maneras de compilar el proyecto. El método de compilación depende de cómo se compile el proyecto.

#### <a name="remote-build-with-extra-index-url"></a>Compilación remota con dirección URL de índice adicional

Cuando los paquetes estén disponibles desde un índice de paquetes personalizado accesible, use una compilación remota. Antes de publicar, asegúrese de [crear una configuración de aplicación](functions-how-to-use-azure-function-app-settings.md#settings) denominada `PIP_EXTRA_INDEX_URL`. El valor de esta configuración es la dirección URL del índice de paquetes personalizado. El uso de esta configuración indica a la compilación remota que ejecute `pip install` mediante la opción `--extra-index-url`. Para más información, vea la [documentación de instalación de pip de Python](https://pip.pypa.io/en/stable/reference/pip_install/#requirements-file-format).

También puede utilizar las credenciales de autenticación básica con las direcciones URL del índice de paquetes adicional. Para más información, vea [Credenciales de autenticación básica](https://pip.pypa.io/en/stable/user_guide/#basic-authentication-credentials) en la documentación de Python.

#### <a name="install-local-packages"></a>Instalación de paquetes locales

Si el proyecto usa paquetes que no están disponibles públicamente para nuestras herramientas, puede ponerlos a disposición de la aplicación colocándolos en el directorio \_\_app\_\_/.python_packages. Antes de la publicación, ejecute el siguiente comando para instalar las dependencias localmente:

```command
pip install  --target="<PROJECT_DIR>/.python_packages/lib/site-packages"  -r requirements.txt
```

Al usar dependencias personalizadas, debe usar la opción de publicación `--no-build`, puesto que ya ha instalado las dependencias en la carpeta del proyecto.

```command
func azure functionapp publish <APP_NAME> --no-build
```

Reemplace `<APP_NAME>` por el nombre de la aplicación de función de Azure.

## <a name="unit-testing"></a>Pruebas unitarias

Las funciones escritas en Python se pueden probar como otro código de Python mediante marcos de pruebas. Para la mayoría de los enlaces, es posible crear un objeto de entrada ficticio creando una instancia de una clase adecuada a partir del paquete `azure.functions`. Dado que el paquete [`azure.functions`](https://pypi.org/project/azure-functions/) no está disponible inmediatamente, asegúrese de instalarlo a través del archivo `requirements.txt`, tal como se describe en la sección [Administración de paquetes](#package-management) anterior.

Por ejemplo, a continuación se muestra una prueba ficticia de una función desencadenada por HTTP:

```json
{
  "scriptFile": "__init__.py",
  "entryPoint": "my_function",
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
  ]
}
```

```python
# __app__/HttpTrigger/__init__.py
import azure.functions as func
import logging

def my_function(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    name = req.params.get('name')
    if not name:
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            name = req_body.get('name')

    if name:
        return func.HttpResponse(f"Hello {name}")
    else:
        return func.HttpResponse(
             "Please pass a name on the query string or in the request body",
             status_code=400
        )
```

```python
# tests/test_httptrigger.py
import unittest

import azure.functions as func
from __app__.HttpTrigger import my_function

class TestFunction(unittest.TestCase):
    def test_my_function(self):
        # Construct a mock HTTP request.
        req = func.HttpRequest(
            method='GET',
            body=None,
            url='/api/HttpTrigger',
            params={'name': 'Test'})

        # Call the function.
        resp = my_function(req)

        # Check the output.
        self.assertEqual(
            resp.get_body(),
            b'Hello Test',
        )
```

Este es otro ejemplo, con una función desencadenada por la cola:

```json
{
  "scriptFile": "__init__.py",
  "entryPoint": "my_function",
  "bindings": [
    {
      "name": "msg",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "python-queue-items",
      "connection": "AzureWebJobsStorage"
    }
  ]
}
```

```python
# __app__/QueueTrigger/__init__.py
import azure.functions as func

def my_function(msg: func.QueueMessage) -> str:
    return f'msg body: {msg.get_body().decode()}'
```

```python
# tests/test_queuetrigger.py
import unittest

import azure.functions as func
from __app__.QueueTrigger import my_function

class TestFunction(unittest.TestCase):
    def test_my_function(self):
        # Construct a mock Queue message.
        req = func.QueueMessage(
            body=b'test')

        # Call the function.
        resp = my_function(req)

        # Check the output.
        self.assertEqual(
            resp,
            'msg body: test',
        )
```
## <a name="temporary-files"></a>Archivos temporales

El método `tempfile.gettempdir()` devuelve una carpeta temporal, que en Linux es `/tmp`. La aplicación puede usar este directorio para almacenar los archivos temporales generados y usados por las funciones durante la ejecución.

> [!IMPORTANT]
> No se garantiza que los archivos escritos en el directorio temporal persistan entre invocaciones. Durante el escalado horizontal, los archivos temporales no se comparten entre instancias.

En el ejemplo siguiente se crea un archivo temporal con nombre en el directorio temporal (`/tmp`):

```python
import logging
import azure.functions as func
import tempfile
from os import listdir

#---
   tempFilePath = tempfile.gettempdir()
   fp = tempfile.NamedTemporaryFile()
   fp.write(b'Hello world!')
   filesDirListInTemp = listdir(tempFilePath)
```

Se recomienda mantener las pruebas en una carpeta independiente de la carpeta del proyecto. Esto evita que se implemente código de prueba con la aplicación.

## <a name="preinstalled-libraries"></a>Bibliotecas preinstaladas

Hay algunas bibliotecas que se incluyen en Functions Runtime para Python.

### <a name="python-standard-library"></a>Biblioteca estándar de Python

La biblioteca estándar de Python contiene una lista de módulos de Python integrados que se incluyen con cada distribución de Python. La mayoría de estas bibliotecas lo ayudan a acceder a la funcionalidad del sistema, como la E/S de archivos. En los sistemas Windows, estas bibliotecas se instalan con Python. En los sistemas basados en Unix, las proporcionan las colecciones de paquetes.

Para ver todos los detalles de la lista de estas bibliotecas, visite los vínculos siguientes:

* [Biblioteca estándar de Python 3.6](https://docs.python.org/3.6/library/)
* [Biblioteca estándar de Python 3.7](https://docs.python.org/3.7/library/)
* [Biblioteca estándar de Python 3.8](https://docs.python.org/3.8/library/)

### <a name="azure-functions-python-worker-dependencies"></a>Dependencias de trabajo de Python en Azure Functions

El trabajo de Python en Functions requiere un conjunto específico de bibliotecas. También puede usar estas bibliotecas en sus funciones, pero no forman parte del estándar de Python. Si las funciones se basan en cualquiera de estas bibliotecas, puede que no estén disponibles para el código cuando se ejecutan fuera de Azure Functions. Puede encontrar una lista detallada de las dependencias en la sección **install\_requires** del archivo [setup.py](https://github.com/Azure/azure-functions-python-worker/blob/dev/setup.py#L282).

> [!NOTE]
> Si el archivo requirements.txt de la aplicación de funciones contiene una entrada `azure-functions-worker`, quítela. El trabajo de funciones se administra automáticamente mediante la plataforma de Azure Functions y se actualiza periódicamente con nuevas características y correcciones de errores. La instalación manual de una versión anterior del trabajo en el archivo requirements.txt puede producir problemas inesperados.

### <a name="azure-functions-python-library"></a>Biblioteca de Python para Azure Functions

Cada actualización de trabajado de Python incluye una nueva versión de la [biblioteca de Python para Azure Functions (azure.functions)](https://github.com/Azure/azure-functions-python-library). Este enfoque facilita la actualización continua de las aplicaciones de funciones de Python, ya que cada actualización es compatible con versiones anteriores. Puede encontrar una lista de las versiones de esta biblioteca en [azure-functions en PyPi](https://pypi.org/project/azure-functions/#history).

La versión de la biblioteca en tiempo de ejecución la determina Azure y no se puede reemplazar por requirements.txt. La entrada `azure-functions` en requirements.txt es solo para linting y reconocimiento de clientes.

Use el código siguiente para realizar un seguimiento de la versión real de la biblioteca de Python para Functions en tiempo de ejecución:

```python
getattr(azure.functions, '__version__', '< 1.2.1')
```

### <a name="runtime-system-libraries"></a>Bibliotecas de sistema en tiempo de ejecución

Para obtener una lista de las bibliotecas de sistema preinstaladas en las imágenes de Docker de trabajo de Python, siga estos vínculos:

|  Sistema en tiempo de ejecución de Functions  | Versión de Debian | Versiones de Python |
|------------|------------|------------|
| Versión 2.x | Stretch  | [Python 3.6](https://github.com/Azure/azure-functions-docker/blob/master/host/2.0/stretch/amd64/python/python36/python36.Dockerfile)<br/>[Python 3.7](https://github.com/Azure/azure-functions-docker/blob/master/host/2.0/stretch/amd64/python/python37/python37.Dockerfile) |
| Versión 3.x | Buster | [Python 3.6](https://github.com/Azure/azure-functions-docker/blob/master/host/3.0/buster/amd64/python/python36/python36.Dockerfile)<br/>[Python 3.7](https://github.com/Azure/azure-functions-docker/blob/master/host/3.0/buster/amd64/python/python37/python37.Dockerfile)<br />[Python 3.8](https://github.com/Azure/azure-functions-docker/blob/master/host/3.0/buster/amd64/python/python38/python38.Dockerfile) |

## <a name="cross-origin-resource-sharing"></a>Uso compartido de recursos entre orígenes

[!INCLUDE [functions-cors](../../includes/functions-cors.md)]

CORS es totalmente compatible con las aplicaciones de funciones de Python.

## <a name="known-issues-and-faq"></a>Problemas conocidos y preguntas más frecuentes

A continuación, se muestra una lista de las guías de solución de problemas comunes:

* [ModuleNotFoundError e ImportError](recover-python-functions.md#troubleshoot-modulenotfounderror)
* [No se puede importar "cygrpc"](recover-python-functions.md#troubleshoot-cannot-import-cygrpc)

Todos los problemas conocidos y las solicitudes de características se siguen mediante la lista de[problemas de GitHub](https://github.com/Azure/azure-functions-python-worker/issues). Si le surge algún problema y no lo encuentra en GitHub, abra un nuevo problema e incluya una descripción detallada del mismo.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte los siguientes recursos:

* [Documentación de la API del paquete de Azure Functions](/python/api/azure-functions/azure.functions?view=azure-python)
* [Procedimientos recomendados para Azure Functions](functions-best-practices.md)
* [Enlaces y desencadenadores de Azure Functions](functions-triggers-bindings.md)
* [Enlaces de Blob Storage](functions-bindings-storage-blob.md)
* [Enlaces de HTTP y de Webhook](functions-bindings-http-webhook.md)
* [Enlaces de Queue Storage](functions-bindings-storage-queue.md)
* [Desencadenador de temporizador](functions-bindings-timer.md)


[HttpRequest]: /python/api/azure-functions/azure.functions.httprequest?view=azure-python
[HttpResponse]: /python/api/azure-functions/azure.functions.httpresponse?view=azure-python
