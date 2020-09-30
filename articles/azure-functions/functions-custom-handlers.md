---
title: Controladores personalizados de Azure Functions (versión preliminar)
description: Aprenda a utilizar Azure Functions con cualquier versión de lenguaje o de tiempo de ejecución.
author: anthonychu
ms.author: antchu
ms.date: 8/18/2020
ms.topic: article
ms.openlocfilehash: f3106553def982eb90ccc90822206e75a11ce354
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89294601"
---
# <a name="azure-functions-custom-handlers-preview"></a>Controladores personalizados de Azure Functions (versión preliminar)

Cada aplicación de Functions se ejecuta mediante un controlador específico de lenguaje. Aunque Azure Functions admite muchos [controladores de lenguajes](./supported-languages.md) de forma predeterminada, hay casos en los que es posible que desee usar otros lenguajes o runtimes.

Los controladores personalizados son servidores web ligeros que reciben eventos del host de Functions. Cualquier lenguaje que admita primitivas de HTTP puede implementar un controlador personalizado.

Los controladores personalizados son más adecuados para las situaciones en las que desea:

- Implemente una aplicación de funciones en un lenguaje que no se admite actualmente, como Go y Rust.
- Implemente una aplicación de funciones en un runtime que no se admite actualmente, como Deno.

Con los controladores personalizados, puede usar [desencadenadores y enlaces de entrada y salida](./functions-triggers-bindings.md) mediante [conjuntos de extensiones](./functions-bindings-register.md).

## <a name="overview"></a>Información general

En el diagrama siguiente se muestra la relación entre el host de Functions y un servidor web implementado como controlador personalizado.

![Introducción a los controladores personalizados de Azure Functions](./media/functions-custom-handlers/azure-functions-custom-handlers-overview.png)

1. Cada evento desencadena una solicitud enviada al host de Functions. Un evento es cualquier desencadenador compatible con Azure Functions.
1. A continuación, el host de Functions emite una [carga de la solicitud](#request-payload) al servidor web. La carga contiene datos de enlace de entrada y del desencadenador, así como otros metadatos para la función.
1. El servidor web ejecuta la función individual y devuelve una [carga de respuesta](#response-payload) al host de Functions.
1. El host de Functions pasa datos de la respuesta a los enlaces de salida de la función para su procesamiento.

Una aplicación Azure Functions implementada como controlador personalizado debe configurar los archivos *host.json*, *local.settings.json* y *function.json* siguiendo algunas convenciones.

## <a name="application-structure"></a>Estructura de la aplicación

Para implementar un controlador personalizado, necesita los siguientes aspectos para la aplicación:

- Un archivo *host.json* en la raíz de la aplicación
- Un archivo *local.settings.json* en la raíz de la aplicación
- Un archivo *function.json* para cada función (dentro de una carpeta que coincida con el nombre de función)
- Un comando, script o archivo ejecutable, que ejecuta un servidor web.

En el diagrama siguiente se muestra cómo estos archivos buscan en el sistema de archivos una función denominada "MyQueueFunction" y un ejecutable del controlador personalizado denominado *handler.exe*.

```bash
| /MyQueueFunction
|   function.json
|
| host.json
| local.settings.json
| handler.exe
```

### <a name="configuration"></a>Configuración

La aplicación se configura mediante los archivos *host.json* y *local.settings.json*.

#### <a name="hostjson"></a>host.json

*host.json* indica al host de Functions dónde se deben enviar las solicitudes al apuntar a un servidor web capaz de procesar eventos HTTP.

Para definir un controlador personalizado, configure el archivo *host.json* con detalles sobre cómo ejecutar el servidor web siguiendo la sección `customHandler`.

```json
{
  "version": "2.0",
  "customHandler": {
    "description": {
      "defaultExecutablePath": "handler.exe"
    }
  }
}
```

La sección `customHandler` apunta a un destino tal y como se define en `defaultExecutablePath`. El destino de ejecución puede ser un comando, ejecutable o archivo en el que esté implementado el servidor web.

Use la matriz `arguments` para pasar cualquier argumento al ejecutable. Los argumentos admiten la expansión de las variables de entorno (configuración de la aplicación) mediante la notación `%%`.

También puede cambiar el directorio de trabajo que usa el ejecutable con `workingDirectory`.

```json
{
  "version": "2.0",
  "customHandler": {
    "description": {
      "defaultExecutablePath": "app/handler.exe",
      "arguments": [
        "--database-connection-string",
        "%DATABASE_CONNECTION_STRING%"
      ],
      "workingDirectory": "app"
    }
  }
}
```

##### <a name="bindings-support"></a>Compatibilidad con enlaces

Los desencadenadores estándar junto con los enlaces de entrada y salida están disponibles al hacer referencia a los [conjuntos de extensión](./functions-bindings-register.md) en el archivo *host.json*.

#### <a name="localsettingsjson"></a>local.settings.json

*local.settings.json* define la configuración de la aplicación que se usa al ejecutar la aplicación de funciones localmente. Como puede contener secretos, *local.settings.json* debe excluirse del control de código fuente. En Azure, use la configuración de la aplicación en su lugar.

En el caso de los controladores personalizados, establezca `FUNCTIONS_WORKER_RUNTIME` en `Custom` en *local.settings.json*.

```json
{
  "IsEncrypted": false,
  "Values": {
    "FUNCTIONS_WORKER_RUNTIME": "Custom"
  }
}
```

> [!NOTE]
> Es posible que no se reconozca `Custom` como runtime válido en los planes de App Service o Linux Premium. Si ese es su destino de implementación, establezca `FUNCTIONS_WORKER_RUNTIME` en una cadena vacía.

### <a name="function-metadata"></a>Metadatos de función

Cuando se usa con un controlador personalizado, el contenido de *function.json* no es diferente a cómo se define una función en cualquier otro contexto. El único requisito es que los archivos  *function.json* deben estar en una carpeta con nombre que coincida con el nombre de la función.

El siguiente archivo *function.json* configura una función que tiene un desencadenador de cola y un enlace de salida de cola. Dado que se encuentra en una carpeta denominada *MyQueueFunction*, define una función llamada *MyQueueFunction*.

**MyQueueFunction/function.json**

```json
{
  "bindings": [
    {
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "messages-incoming",
      "connection": "AzureWebJobsStorage"
    },
    {
      "name": "$return",
      "type": "queue",
      "direction": "out",
      "queueName": "messages-outgoing",
      "connection": "AzureWebJobsStorage"
    }
  ]
}
```

### <a name="request-payload"></a>Carga de solicitud

Cuando se recibe un mensaje en cola, el host de Functions envía una solicitud HTTP POST al controlador personalizado con una carga en el cuerpo.

El código siguiente representa una carga de solicitud de ejemplo. La carga incluye una estructura JSON con dos miembros: `Data` y `Metadata`.

El miembro `Data` incluye claves que coinciden con los nombres de los desencadenadores y de entrada, tal y como se definen en la matriz de enlaces del archivo *function.json*.

El miembro `Metadata` incluye [metadatos generados a partir del origen del evento](./functions-bindings-expressions-patterns.md#trigger-metadata).

```json
{
  "Data": {
    "myQueueItem": "{ message: \"Message sent\" }"
  },
  "Metadata": {
    "DequeueCount": 1,
    "ExpirationTime": "2019-10-16T17:58:31+00:00",
    "Id": "800ae4b3-bdd2-4c08-badd-f08e5a34b865",
    "InsertionTime": "2019-10-09T17:58:31+00:00",
    "NextVisibleTime": "2019-10-09T18:08:32+00:00",
    "PopReceipt": "AgAAAAMAAAAAAAAAAgtnj8x+1QE=",
    "sys": {
      "MethodName": "QueueTrigger",
      "UtcNow": "2019-10-09T17:58:32.2205399Z",
      "RandGuid": "24ad4c06-24ad-4e5b-8294-3da9714877e9"
    }
  }
}
```

### <a name="response-payload"></a>Carga de respuesta

Por convención, las respuestas de función tienen el formato de pares clave-valor. Las claves admitidas son:

| <nobr>Clave de carga</nobr>   | Tipo de datos | Observaciones                                                      |
| ------------- | --------- | ------------------------------------------------------------ |
| `Outputs`     | object    | Contiene los valores de respuesta definidos por la matriz `bindings` del archivo *function.json*.<br /><br />Por ejemplo, si se configura una función con un enlace de salida de cola llamado "myQueueOutput", `Outputs` contiene una clave denominada `myQueueOutput`, la cual establece el controlador personalizado en los mensajes que se envían a la cola. |
| `Logs`        | array     | Los mensajes aparecen en los registros de invocación de Functions.<br /><br />Cuando se ejecuta en Azure, los mensajes aparecen en Application Insights. |
| `ReturnValue` | string    | Se usa para proporcionar una respuesta cuando se configura una salida como `$return` en el archivo *function.json*. |

Este es un ejemplo de una carga de respuesta.

```json
{
  "Outputs": {
    "res": {
      "body": "Message enqueued"
    },
    "myQueueOutput": [
      "queue message 1",
      "queue message 2"
    ]
  },
  "Logs": [
    "Log message 1",
    "Log message 2"
  ],
  "ReturnValue": "{\"hello\":\"world\"}"
}
```

## <a name="examples"></a>Ejemplos

Los controladores personalizados se pueden implementar en cualquier lenguaje que admita la recepción de eventos HTTP. En los siguientes ejemplos se muestra cómo implementar un controlador personalizado mediante el lenguaje de programación Go.

### <a name="function-with-bindings"></a>Función con enlaces

El escenario implementado en este ejemplo incluye una función denominada `order` que acepta `POST` con una carga que representa un pedido de producto. A medida que se publica un pedido en la función, se crea un mensaje de Queue Storage y se devuelve una respuesta HTTP.

<a id="bindings-implementation" name="bindings-implementation"></a>

#### <a name="implementation"></a>Implementación

En una carpeta denominada *order*, el archivo *function.json* configura la función desencadenada por HTTP.

**order/function.json**

```json
{
  "bindings": [
    {
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": ["post"]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    },
    {
      "type": "queue",
      "name": "message",
      "direction": "out",
      "queueName": "orders",
      "connection": "AzureWebJobsStorage"
    }
  ]
}
```

Esta función se define como una [función desencadenada por HTTP](./functions-bindings-http-webhook-trigger.md) que devuelve una [respuesta HTTP](./functions-bindings-http-webhook-output.md) y genera un mensaje de [Queue Storage](./functions-bindings-storage-queue-output.md).

En la raíz de la aplicación, el archivo *host.json* se configura para ejecutar un archivo ejecutable nombrado `handler.exe` (`handler` en Linux o macOS).

```json
{
  "version": "2.0",
  "customHandler": {
    "description": {
      "defaultExecutablePath": "handler.exe"
    }
  },
  "extensionBundle": {
    "id": "Microsoft.Azure.Functions.ExtensionBundle",
    "version": "[1.*, 2.0.0)"
  }
}
```

Esta es la solicitud HTTP enviada al runtime de Functions.

```http
POST http://127.0.0.1:7071/api/order HTTP/1.1
Content-Type: application/json

{
  "id": 1005,
  "quantity": 2,
  "color": "black"
}
```

A continuación, el runtime de Functions enviará la siguiente solicitud HTTP al controlador personalizado:

```http
POST http://127.0.0.1:<FUNCTIONS_CUSTOMHANDLER_PORT>/order HTTP/1.1
Content-Type: application/json

{
  "Data": {
    "req": {
      "Url": "http://localhost:7071/api/order",
      "Method": "POST",
      "Query": "{}",
      "Headers": {
        "Content-Type": [
          "application/json"
        ]
      },
      "Params": {},
      "Body": "{\"id\":1005,\"quantity\":2,\"color\":\"black\"}"
    }
  },
  "Metadata": {
  }
}
```

> [!NOTE]
> Algunas partes de la carga se quitaron por motivos de brevedad.

*handler.exe* es el programa de controlador personalizado Go compilado que ejecuta un servidor web y responde a solicitudes de invocación de la función del host de Functions.

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "os"
)

type InvokeRequest struct {
    Data     map[string]json.RawMessage
    Metadata map[string]interface{}
}

type InvokeResponse struct {
    Outputs     map[string]interface{}
    Logs        []string
    ReturnValue interface{}
}

func orderHandler(w http.ResponseWriter, r *http.Request) {
    var invokeRequest InvokeRequest

    d := json.NewDecoder(r.Body)
    d.Decode(&invokeRequest)

    var reqData map[string]interface{}
    json.Unmarshal(invokeRequest.Data["req"], &reqData)

    outputs := make(map[string]interface{})
    outputs["message"] = reqData["Body"]

    resData := make(map[string]interface{})
    resData["body"] = "Order enqueued"
    outputs["res"] = resData
    invokeResponse := InvokeResponse{outputs, nil, nil}

    responseJson, _ := json.Marshal(invokeResponse)

    w.Header().Set("Content-Type", "application/json")
    w.Write(responseJson)
}

func main() {
    customHandlerPort, exists := os.LookupEnv("FUNCTIONS_CUSTOMHANDLER_PORT")
    if !exists {
        customHandlerPort = "8080"
    }
    mux := http.NewServeMux()
    mux.HandleFunc("/order", orderHandler)
    fmt.Println("Go server Listening on: ", customHandlerPort)
    log.Fatal(http.ListenAndServe(":"+customHandlerPort, mux))
}
```

En este ejemplo, el controlador personalizado ejecuta un servidor web para administrar eventos HTTP, y se configura para que escuche solicitudes mediante `FUNCTIONS_CUSTOMHANDLER_PORT`.

Aunque el host de Functions recibió una solicitud HTTP original en `/api/order`, invoca el controlador personalizado mediante el nombre de la función (su nombre de carpeta). En este ejemplo, la función se define en la ruta de acceso de `/order`. El host envía al controlador personalizado una solicitud HTTP en la ruta de acceso de `/order`.

A medida que se envían solicitudes `POST` a esta función, los datos del desencadenador y los metadatos de la función están disponibles a través del cuerpo de la solicitud HTTP. Se puede tener acceso al cuerpo de la solicitud HTTP original en el `Data.req.Body` de la carga.

El formato de la respuesta de la función se aplica a pares clave-valor, donde el miembro `Outputs` contiene un valor JSON en el que las claves coinciden con las salidas definidas en el archivo *function.json*.

Esta es una carga de ejemplo que este controlador devuelve al host de Functions.

```json
{
  "Outputs": {
    "message": "{\"id\":1005,\"quantity\":2,\"color\":\"black\"}",
    "res": {
      "body": "Order enqueued"
    }
  },
  "Logs": null,
  "ReturnValue": null
}
```

Al establecer la salida `message` equivalente a los datos del pedido procedentes de la solicitud, la función genera dichos datos del pedido en la cola configurada. El host de Functions también devuelve la respuesta HTTP configurada en `res` al llamador.

### <a name="http-only-function"></a>Función de solo HTTP

En el caso de las funciones desencadenadas por HTTP sin enlaces ni salidas adicionales, es posible que desee que el controlador trabaje directamente con la solicitud y respuesta HTTP en lugar de las cargas de [solicitud](#request-payload) y [respuesta](#response-payload) del controlador personalizadas. Este comportamiento se puede configurar en *host.json* mediante la configuración `enableForwardingHttpRequest`.

> [!IMPORTANT]
> El propósito principal de la característica de controladores personalizados es habilitar los lenguajes y runtimes que actualmente no tengan soporte de primera clase en Azure Functions. Aunque puede ser posible ejecutar aplicaciones web mediante controladores personalizados, Azure Functions no es un proxy inverso estándar. Algunas características como el streaming de respuesta, HTTP/2 y WebSockets no están disponibles. Pueden restringirse algunos componentes de la solicitud HTTP, por ejemplo, algunos encabezados y rutas. La aplicación también puede experimentar un [arranque en frío](functions-scale.md#cold-start) excesivo.
>
> Para abordar estas circunstancias, considere la posibilidad de ejecutar las aplicaciones web en [Azure App Service](../app-service/overview.md).

En el ejemplo siguiente se muestra cómo configurar una función desencadenada por HTTP sin enlaces ni salidas adicionales. El escenario implementado en este ejemplo incluye una función denominada `hello` que acepta `GET` o `POST`.

<a id="hello-implementation" name="hello-implementation"></a>

#### <a name="implementation"></a>Implementación

En una carpeta denominada *hello*, el archivo *function.json* configura la función desencadenada por HTTP.

**hello/function.json**

```json
{
  "bindings": [
    {
      "type": "httpTrigger",
      "authLevel": "anonymous",
      "direction": "in",
      "name": "req",
      "methods": ["get", "post"]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    }
  ]
}
```

La función está configurada para aceptar las solicitudes `GET` y `POST`, y el valor del resultado se proporciona mediante un argumento denominado `res`.

En la raíz de la aplicación, el archivo *host.json* está configurado para ejecutar `handler.exe` y `enableForwardingHttpRequest` se establece en `true`.

```json
{
  "version": "2.0",
  "customHandler": {
    "description": {
      "defaultExecutablePath": "handler.exe"
    },
    "enableForwardingHttpRequest": true
  }
}
```

Cuando `enableForwardingHttpRequest` es `true`, el comportamiento de las funciones de solo HTTP difiere del comportamiento de los controladores personalizados predeterminados de las siguientes formas:

* La solicitud HTTP no contiene la carga de la [solicitud](#request-payload) de los controladores personalizados. En su lugar, el host de Functions invoca el controlador con una copia de la solicitud HTTP original.
* El host de Functions invoca el controlador con la misma ruta de acceso que la solicitud original, incluidos los parámetros de cadena de consulta.
* El host de Functions devuelve una copia de la respuesta HTTP del controlador como respuesta a la solicitud original.

A continuación se muestra una solicitud POST al host de Functions. A continuación, el host de Functions envía una copia de la solicitud al controlador personalizado en la misma ruta de acceso.

```http
POST http://127.0.0.1:7071/api/hello HTTP/1.1
Content-Type: application/json

{
  "message": "Hello World!"
}
```

El archivo *handler.go* implementa un servidor web y una función HTTP.

```go
package main

import (
    "fmt"
    "io/ioutil"
    "log"
    "net/http"
    "os"
)

func helloHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    if r.Method == "GET" {
        w.Write([]byte("hello world"))
    } else {
        body, _ := ioutil.ReadAll(r.Body)
        w.Write(body)
    }
}

func main() {
    customHandlerPort, exists := os.LookupEnv("FUNCTIONS_CUSTOMHANDLER_PORT")
    if !exists {
        customHandlerPort = "8080"
    }
    mux := http.NewServeMux()
    mux.HandleFunc("/api/hello", helloHandler)
    fmt.Println("Go server Listening on: ", customHandlerPort)
    log.Fatal(http.ListenAndServe(":"+customHandlerPort, mux))
}
```

En este ejemplo, el controlador personalizado crea un servidor web para administrar eventos HTTP, y se configura para que escuche solicitudes mediante `FUNCTIONS_CUSTOMHANDLER_PORT`.

Las solicitudes `GET` se administran al devolver una cadena y las solicitudes `POST` tienen acceso al cuerpo de la solicitud.

La ruta para la función Order aquí es `/api/hello`, igual que la solicitud original.

>[!NOTE]
>`FUNCTIONS_CUSTOMHANDLER_PORT` no es el puerto de acceso público que se usa para llamar a la función. El host de Functions usa este puerto para llamar al controlador personalizado.

## <a name="deploying"></a>Implementando

Se puede implementar un controlador personalizado en todas las opciones de hospedaje de Azure Functions. Si el controlador requiere dependencias del sistema operativo o de la plataforma (por ejemplo, un tiempo de ejecución de lenguaje), es posible que tenga que usar un [contenedor personalizado](./functions-create-function-linux-custom-image.md).

Al crear una aplicación de funciones en Azure para los controladores personalizados, se recomienda seleccionar .NET Core como la pila. En el futuro, se agregará una pila "personalizada" para los controladores personalizados.

En Azure Functions Core Tools, ejecute el siguiente comando para implementar una aplicación de controlador personalizada.

```bash
func azure functionapp publish $functionAppName
```

> [!NOTE]
> Asegúrese de que todos los archivos necesarios para ejecutar el controlador personalizado están en la carpeta y se incluyen en la implementación. Si el controlador personalizado es un archivo ejecutable binario o tiene dependencias específicas de la plataforma, asegúrese de que estos archivos coinciden con la plataforma de implementación de destino.

## <a name="restrictions"></a>Restricciones

- El servidor web del controlador personalizado se debe iniciar en un plazo de 60 segundos.

## <a name="samples"></a>Ejemplos

Consulte el [repositorio de GitHub de ejemplos del controlador personalizado](https://github.com/Azure-Samples/functions-custom-handlers) para ver ejemplos sobre cómo implementar funciones en una variedad de lenguajes distintos.

## <a name="troubleshooting-and-support"></a>Solución de problemas y soporte técnico

### <a name="trace-logging"></a>Registro de seguimiento

Si el proceso del controlador personalizado no se inicia o tiene problemas para comunicarse con el host de Functions, puede aumentar el nivel de registro de la aplicación de funciones a `Trace` para ver más mensajes de diagnóstico del host.

Para cambiar el nivel de registro predeterminado de la aplicación de funciones, establezca la configuración `logLevel` en la sección `logging` de *host.json*.

```json
{
  "version": "2.0",
  "customHandler": {
    "description": {
      "defaultExecutablePath": "handler.exe"
    }
  },
  "logging": {
    "logLevel": {
      "default": "Trace"
    }
  }
}
```

El host de Functions genera mensajes de registro adicionales, incluida información relacionada con el proceso del controlador personalizado. Use los registros para investigar problemas al iniciar el proceso del controlador personalizado o invocar funciones en este.

Localmente, los registros se imprimen en la consola.

En Azure, [consulte los seguimientos de Application Insights](functions-monitoring.md#query-telemetry-data) para ver los mensajes de registro. Si la aplicación genera un gran volumen de registros, solo se enviará un subconjunto de mensajes de registro a Application Insights. [Deshabilite el muestreo](functions-monitoring.md#configure-sampling) para asegurarse de que se registran todos los mensajes.

### <a name="test-custom-handler-in-isolation"></a>Prueba del controlador personalizado en aislamiento

Las aplicaciones del controlador personalizado son un proceso del servidor web, por lo que puede resultar útil para iniciarlo por su cuenta y probar invocaciones de función enviando [solicitudes HTTP](#request-payload) ficticias mediante una herramienta como [cURL](https://curl.haxx.se/) o [Postman](https://www.postman.com/).

También puede usar esta estrategia en las canalizaciones de CI/CD para ejecutar pruebas automatizadas en el controlador personalizado.

### <a name="execution-environment"></a>Entorno de ejecución

Los controladores personalizados se ejecutan en el mismo entorno que una aplicación de Azure Functions típica. Pruebe el controlador para asegurarse de que el entorno contiene todas las dependencias que necesita para ejecutarse. En el caso de las aplicaciones que requieren dependencias adicionales, puede que necesite ejecutarlas mediante una [imagen de contenedor personalizada](functions-create-function-linux-custom-image.md) hospedada en el [plan Premium](functions-premium-plan.md) de Azure Functions.

### <a name="get-support"></a>Obtención de soporte técnico

Si necesita ayuda en una aplicación de funciones con controladores personalizados, puede enviar una solicitud a través de canales de soporte técnico normales. Sin embargo, debido a la amplia variedad de lenguajes posibles que se usan para compilar aplicaciones de controladores personalizados, el soporte técnico no es ilimitado.

El soporte técnico estará disponible si el host de Functions tiene problemas para iniciarse o comunicarse con el proceso del controlador personalizado. En el caso de experimentar problemas específicos del funcionamiento interno del proceso del controlador personalizado, como, por ejemplo, problemas con el lenguaje o el marco elegidos, nuestro equipo de soporte técnico no puede proporcionar ayuda en este contexto.
