---
title: Referencia para desarrolladores de JavaScript para Azure Functions
description: Obtenga información sobre cómo desarrollar funciones con JavaScript.
ms.assetid: 45dedd78-3ff9-411f-bb4b-16d29a11384c
ms.topic: conceptual
ms.date: 07/17/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: ff3e5431481cba0d2d806d60ba5d7a291d1b2b69
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87810123"
---
# <a name="azure-functions-javascript-developer-guide"></a>Guía para el desarrollador de JavaScript para Azure Functions

Esta guía contiene información detallada para ayudarle a desarrollar correctamente Azure Functions con JavaScript.

Como desarrollador Express.js, Node.js o JavaScript, si no está familiarizado con Azure Functions, considere la posibilidad de leer primero uno de los siguientes artículos:

| Introducción | Conceptos| Aprendizaje guiado |
| -- | -- | -- | 
| <ul><li>[Función de Node.js mediante Visual Studio Code](./functions-create-first-function-vs-code.md?pivots=programming-language-javascript)</li><li>[Función de Node.js con el símbolo del sistema o terminal](./functions-create-first-azure-function-azure-cli.md?pivots=programming-language-javascript)</li></ul> | <ul><li>[Guía del desarrollador](functions-reference.md)</li><li>[Opciones de hospedaje](functions-scale.md)</li><li>[Funciones de TypeScript](#typescript)</li><li>[Consideraciones y rendimiento](functions-best-practices.md)</li></ul> | <ul><li>[Creación de aplicaciones sin servidor](/learn/paths/create-serverless-applications/)</li><li>[Refactorización de las API de Node.js y Express para las API sin servidor](/learn/modules/shift-nodejs-express-apis-serverless/)</li></ul> |

## <a name="javascript-function-basics"></a>Fundamentos de las funciones de JavaScript

Una función de JavaScript (Node.js) es una `function` exportada que se ejecuta al desencadenarse ([los desencadenadores se configuran en function.json](functions-triggers-bindings.md)). El primer argumento que se pasa a cada función es un objeto `context` que se usa para enviar y recibir datos de enlace, para el registro y para la comunicación con el entorno de ejecución.

## <a name="folder-structure"></a>Estructura de carpetas

La estructura de carpetas necesaria para un proyecto de JavaScript tiene el siguiente aspecto. Este valor predeterminado se puede cambiar. Para más información, consulte la sección [scriptFile](#using-scriptfile) a continuación.

```
FunctionsProject
 | - MyFirstFunction
 | | - index.js
 | | - function.json
 | - MySecondFunction
 | | - index.js
 | | - function.json
 | - SharedCode
 | | - myFirstHelperFunction.js
 | | - mySecondHelperFunction.js
 | - node_modules
 | - host.json
 | - package.json
 | - extensions.csproj
```

En la raíz del proyecto, hay un archivo [host.json](functions-host-json.md) compartido que se puede usar para configurar la aplicación de función. Cada función tiene una carpeta con su propio archivo de código (.js) y archivo de configuración de enlace (function.json). El nombre del directorio primario de `function.json` siempre es el nombre de la función.

Las extensiones de enlace necesarias en la [versión 2.x](functions-versions.md) del tiempo de ejecución de Functions se definen en el archivo `extensions.csproj`, con los archivos de biblioteca de la carpeta `bin`. Al desarrollar de forma local, debe [registrar las extensiones de enlace](./functions-bindings-register.md#extension-bundles). Al desarrollar funciones en Azure Portal, este registro se realiza automáticamente.

## <a name="exporting-a-function"></a>Exportación de una función

Deben exportarse las funciones de JavaScript mediante [`module.exports`](https://nodejs.org/api/modules.html#modules_module_exports) (o [`exports`](https://nodejs.org/api/modules.html#modules_exports)). La función exportada debe ser una función JavaScript que se ejecute al desencadenarse.

De forma predeterminada, el tiempo de ejecución de Functions busca la función en `index.js`, donde `index.js` comparte el mismo directorio primario que su `function.json` correspondiente. En el caso predeterminado, la función exportada debe ser la única exportación de su archivo, o bien la exportación denominada `run` o `index`. Para configurar la ubicación del archivo y el nombre de exportación de la función, obtenga información sobre la [configuración del punto de entrada de la función](functions-reference-node.md#configure-function-entry-point) a continuación.

Varios argumentos se pasan a la función exportada durante la ejecución. El primer argumento que toma siempre es un objeto `context`. Si la función es sincrónica (no devuelve una promesa), debe pasar el objeto `context`, ya que se requiere la llamada a `context.done` para que el uso sea correcto.

```javascript
// You should include context, other arguments are optional
module.exports = function(context, myTrigger, myInput, myOtherInput) {
    // function logic goes here :)
    context.done();
};
```

### <a name="exporting-an-async-function"></a>Exportación de una función asincrónica
Cuando se usa la declaración [`async function`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/async_function) o las [promesas](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise) de JavaScript sin formato en la versión 2.x del tiempo de ejecución de Functions, no tiene que llamar explícitamente a la devolución de llamada [`context.done`](#contextdone-method) para indicar que la función se ha completado. La función se completa cuando la función asincrónica o la promesa exportada se completa. Para las funciones destinadas a la versión 1.x del tiempo de ejecución, debe llamar a [`context.done`](#contextdone-method) cuando el código finalice la ejecución.

El ejemplo siguiente es una función simple que registra que se ha desencadenado y completa la ejecución inmediatamente.

```javascript
module.exports = async function (context) {
    context.log('JavaScript trigger function processed a request.');
};
```

Al exportar una función asincrónica, también puede configurar en enlace de salida para tomar el valor `return`. Esto se recomienda si solo tiene un enlace de salida.

Para asignar una salida mediante `return`, cambie la propiedad `name` a `$return` en `function.json`.

```json
{
  "type": "http",
  "direction": "out",
  "name": "$return"
}
```

En este caso, la función debe tener el aspecto del ejemplo siguiente:

```javascript
module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');
    // You can call and await an async method here
    return {
        body: "Hello, world!"
    };
}
```

## <a name="bindings"></a>Enlaces 
En JavaScript, los [enlaces](functions-triggers-bindings.md) se configuran y definen en el archivo function.json de una función. Las funciones interactúan con los enlaces de varias maneras.

### <a name="inputs"></a>Entradas
Las entradas se dividen en dos categorías Azure Functions: una es la entrada del desencadenador y la otra es una entrada adicional. Una función puede leer los enlaces del desencadenador y de entrada (enlaces de `direction === "in"`) de tres maneras:
 - **_[Recomendada]_ Como parámetros pasados a la función.** Se pasan a la función en el mismo orden en que se definen en *function.json*. La propiedad `name` definida en el archivo *function.json* no tiene que coincidir con el nombre del parámetro, aunque debería hacerlo.
 
   ```javascript
   module.exports = async function(context, myTrigger, myInput, myOtherInput) { ... };
   ```
   
 - **Como miembros del objeto [`context.bindings`](#contextbindings-property).** Cada miembro se denomina mediante la propiedad `name` definida en el archivo *function.json*.
 
   ```javascript
   module.exports = async function(context) { 
       context.log("This is myTrigger: " + context.bindings.myTrigger);
       context.log("This is myInput: " + context.bindings.myInput);
       context.log("This is myOtherInput: " + context.bindings.myOtherInput);
   };
   ```
   
 - **Como entradas mediante el objeto [`arguments`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments) de JavaScript.** Es básicamente lo mismo que pasar entradas como parámetros, pero permite controlar las entradas de manera dinámica.
 
   ```javascript
   module.exports = async function(context) { 
       context.log("This is myTrigger: " + arguments[1]);
       context.log("This is myInput: " + arguments[2]);
       context.log("This is myOtherInput: " + arguments[3]);
   };
   ```

### <a name="outputs"></a>Salidas
Las salidas (enlaces de `direction === "out"`) se pueden escribir mediante una función de varias maneras. En todos los casos, la propiedad `name` del enlace tal como se define en el archivo *function.json* corresponde al nombre del miembro de objeto escrito en la función. 

Puede asignar datos a los enlaces de salida de una de las maneras siguientes (no combine estos métodos):

- **_[Recomendado para varias salidas]_ Devolución de un objeto.** Si usa una función de devolución asincrónica o de promesa, puede devolver un objeto con datos de salida asignados. En el ejemplo siguiente, los enlaces de salida se denominan "httpResponse" y "queueOutput" en el archivo *function.json*.

  ```javascript
  module.exports = async function(context) {
      let retMsg = 'Hello, world!';
      return {
          httpResponse: {
              body: retMsg
          },
          queueOutput: retMsg
      };
  };
  ```

  Si utiliza una función sincrónica, puede devolver este objeto mediante [`context.done`](#contextdone-method) (vea el ejemplo).
- **_[Recomendado para una salida única]_ Devolución de un valor directamente y uso del nombre de enlace $return.** Este método solo funciona con las funciones de devolución asincrónicas o de promesa. Vea el ejemplo de [Exportación de una función asincrónica](#exporting-an-async-function). 
- **Asignación de valores a `context.bindings`** . Puede asignar valores directamente a context.bindings.

  ```javascript
  module.exports = async function(context) {
      let retMsg = 'Hello, world!';
      context.bindings.httpResponse = {
          body: retMsg
      };
      context.bindings.queueOutput = retMsg;
      return;
  };
  ```

### <a name="bindings-data-type"></a>Tipo de datos de enlaces

Para definir el tipo de datos para un enlace de entrada, use la propiedad `dataType` de la definición del enlace. Por ejemplo, para leer el contenido de una solicitud HTTP en formato binario, use el tipo `binary`:

```json
{
    "type": "httpTrigger",
    "name": "req",
    "direction": "in",
    "dataType": "binary"
}
```

Las opciones para `dataType` son: `binary`, `stream` y `string`.

## <a name="context-object"></a>objeto de contexto
El sistema en tiempo de ejecución usa un objeto `context` para transmitir datos desde la función y hacia esta, así como para posibilitar la comunicación con dicho sistema en tiempo de ejecución. El objeto de contexto se puede usar para leer y establecer los datos de los enlaces, escribir registros y usar la devolución de llamada `context.done` cuando la función exportada es sincrónica.

El objeto `context` siempre es el primer parámetro para una función. Debe incluirse porque tiene métodos importantes, tales como `context.done` y `context.log`. Puede asignar el nombre que desee al objeto (por ejemplo, `ctx` o `c`).

```javascript
// You must include a context, but other arguments are optional
module.exports = function(ctx) {
    // function logic goes here :)
    ctx.done();
};
```

### <a name="contextbindings-property"></a>Propiedad context.bindings

```js
context.bindings
```

Devuelve un objeto con nombre que se usa para leer o asignar datos de enlace. A los datos de enlace de entrada y desencadenador se puede acceder mediante las propiedades de lectura en `context.bindings`. Los datos de enlace de salida se pueden asignar mediante la adición de datos a `context.bindings`.

Por ejemplo, las siguientes definiciones de enlace del archivo function.json permiten acceder al contenido de una cola desde `context.bindings.myInput` y asignar salidas a una cola mediante `context.bindings.myOutput`.

```json
{
    "type":"queue",
    "direction":"in",
    "name":"myInput"
    ...
},
{
    "type":"queue",
    "direction":"out",
    "name":"myOutput"
    ...
}
```

```javascript
// myInput contains the input data, which may have properties such as "name"
var author = context.bindings.myInput.name;
// Similarly, you can set your output data
context.bindings.myOutput = { 
        some_text: 'hello world', 
        a_number: 1 };
```

Puede optar por definir datos de enlace de salida a través del método `context.done` en lugar del objeto `context.binding` (consulte a continuación).

### <a name="contextbindingdata-property"></a>Propiedad context.bindingData

```js
context.bindingData
```

Devuelve un objeto con nombre que contiene los metadatos de desencadenar y los datos de invocación de función (`invocationId`, `sys.methodName`, `sys.utcNow` y `sys.randGuid`). Para obtener un ejemplo de metadatos de desencadenador, consulte este [ejemplo de centros de eventos](functions-bindings-event-hubs-trigger.md).

### <a name="contextdone-method"></a>Método context.done

```js
context.done([err],[propertyBag])
```

Permite que el tiempo de ejecución sepa que el código se ha completado. Si su función usa la declaración [`async function`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/async_function), no es necesario que utilice `context.done()`. La devolución de llamada `context.done` se realiza implícitamente. Las funciones asincrónicas están disponibles en Node 8 o una versión posterior, que requiere la versión 2.x del tiempo de ejecución de Functions.

Si la función no es una función asincrónica, **debe llamar** a `context.done` para informar al entorno de ejecución de que la función está completa. La ejecución agota el tiempo de espera si no está presente.

El método `context.done` le permite devolver un error definido por el usuario al entorno en tiempo de ejecución y un objeto JSON que contiene los datos de enlace de salida. Las propiedades que se pasan a `context.done` sobrescriben lo que ha definido en el objeto `context.bindings`.

```javascript
// Even though we set myOutput to have:
//  -> text: 'hello world', number: 123
context.bindings.myOutput = { text: 'hello world', number: 123 };
// If we pass an object to the done function...
context.done(null, { myOutput: { text: 'hello there, world', noNumber: true }});
// the done method overwrites the myOutput binding to be: 
//  -> text: 'hello there, world', noNumber: true
```

### <a name="contextlog-method"></a>Método context.log  

```js
context.log(message)
```

Permite escribir en los registros de la función de streaming en el nivel de seguimiento predeterminado. Hay métodos de registro adicionales disponibles en `context.log` que permiten escribir registros de la función en otros niveles de seguimiento:


| Método                 | Descripción                                |
| ---------------------- | ------------------------------------------ |
| **error(_message_)**   | Escribe en el registro de nivel de error o inferior.   |
| **warn(_message_)**    | Escribe en el registro de nivel de advertencia o inferior. |
| **info(_message_)**    | Escribe en el registro de nivel de información o inferior.    |
| **verbose(_message_)** | Escribe en el registro de nivel detallado.           |

En el ejemplo siguiente se escribe un registro en el nivel de seguimiento de advertencia:

```javascript
context.log.warn("Something has happened."); 
```

Puede [configurar el umbral de nivel de seguimiento de los registros](#configure-the-trace-level-for-console-logging) en el archivo host.json. Para más información sobre cómo escribir registros, consulte cómo [escribir resultados de seguimiento](#writing-trace-output-to-the-console) a continuación.

Consulte cómo [supervisar Azure Functions](functions-monitoring.md) para obtener más información sobre cómo ver y consultar registros de la función.

## <a name="writing-trace-output-to-the-console"></a>Escribir las salidas de seguimiento en la consola 

En Functions, use los métodos `context.log` para escribir la salida de seguimiento en la consola. En Functions 2.x, las salidas de seguimiento mediante `console.log` se capturan en el nivel de Function App. Esto significa que las salidas de `console.log` no están vinculadas a la invocación de una función específica y, por lo tanto, no se muestran en los registros de una función determinada. No obstante, se propagan a Application Insights. En Functions 1.x, no puede usar `console.log` para escribir en la consola.

Cuando se llama a `context.log()`, el mensaje se escribe en la consola en el nivel de seguimiento predeterminado, que es el nivel de seguimiento de _información_. El siguiente código escribe en la consola en el nivel de seguimiento de información:

```javascript
context.log({hello: 'world'});  
```

Este código es equivalente al código anterior:

```javascript
context.log.info({hello: 'world'});  
```

Este código escribe en la consola en el nivel de seguimiento de error:

```javascript
context.log.error("An error has occurred.");  
```

Dado que _error_ es el nivel de seguimiento más alto, este seguimiento se escribe en la salida en todos los niveles de seguimiento, siempre y cuando el registro esté habilitado.

Todos los métodos `context.log` admiten el mismo formato de parámetro que el [método util.format](https://nodejs.org/api/util.html#util_util_format_format) de Node.js. Considere el siguiente código que escribe registros de función mediante el nivel de seguimiento predeterminado:

```javascript
context.log('Node.js HTTP trigger function processed a request. RequestUri=' + req.originalUrl);
context.log('Request Headers = ' + JSON.stringify(req.headers));
```

También puede escribir el mismo código con el formato siguiente:

```javascript
context.log('Node.js HTTP trigger function processed a request. RequestUri=%s', req.originalUrl);
context.log('Request Headers = ', JSON.stringify(req.headers));
```

### <a name="configure-the-trace-level-for-console-logging"></a>Configuración del nivel de seguimiento para el registro de la consola

Functions 1.x permite definir el nivel de seguimiento de umbral para escribir en la consola, que facilita el control de la forma en que se escriben los seguimientos en la consola desde la función. Para establecer el umbral para todos los seguimientos que se escriben en la consola, use la propiedad `tracing.consoleLevel` en el archivo host.json . Esta configuración se aplica a todas las funciones de Function App. En el ejemplo siguiente se establece el umbral de seguimiento para habilitar el registro detallado:

```json
{
    "tracing": {
        "consoleLevel": "verbose"
    }
}  
```

Los valores de **consoleLevel** corresponden a los nombres de los métodos `context.log`. Para deshabilitar todos los registros de seguimiento en la consola, establezca **consoleLevel** en _off_. Para más información, consulte la [referencia sobre host.json](functions-host-json-v1.md).

## <a name="http-triggers-and-bindings"></a>Desencadenadores y enlaces HTTP

Los desencadenadores HTTP y de webhook trigger y los enlaces de salida HTTP usan objetos de solicitud y respuesta para representar la mensajería HTTP.  

### <a name="request-object"></a>Objeto de solicitud

El objeto `context.req` (solicitud) tiene las siguientes propiedades:

| Propiedad      | Descripción                                                    |
| ------------- | -------------------------------------------------------------- |
| _body_        | Objeto que contiene el cuerpo de la solicitud.               |
| _headers_     | Objeto que contiene los encabezados de la solicitud.                   |
| _method_      | Método HTTP de la solicitud.                                |
| _originalUrl_ | Dirección URL de la solicitud.                                        |
| _params_      | Objeto que contiene los parámetros de enrutamiento de la solicitud. |
| _consulta_       | Objeto que contiene los parámetros de consulta.                  |
| _rawBody_     | Cuerpo del mensaje como una cadena.                           |


### <a name="response-object"></a>Objeto de respuesta

El objeto `context.res` (respuesta) tiene las siguientes propiedades:

| Propiedad  | Descripción                                               |
| --------- | --------------------------------------------------------- |
| _body_    | Objeto que contiene el cuerpo de la respuesta.         |
| _headers_ | Objeto que contiene los encabezados de la respuesta.             |
| _isRaw_   | Indica que se omite el formato en la respuesta.    |
| _status_  | Código de estado HTTP de la respuesta.                     |
| _cookies_ | Matriz de objetos de cookie HTTP que se establece en la respuesta. Un objeto de cookie HTTP tiene `name`, `value` y otras propiedades de cookie, como `maxAge` o `sameSite`. |

### <a name="accessing-the-request-and-response"></a>Acceso a solicitudes y respuestas 

Cuando se trabaja con desencadenadores HTTP, hay varias maneras de acceder a los objetos de solicitud y respuesta HTTP:

+ **Desde las propiedades `req` y `res` del objeto `context`.** De esta manera, puede usar el patrón convencional para acceder a los datos HTTP desde el objeto de contexto, en lugar de tener que utilizar el patrón `context.bindings.name` completo. En el ejemplo siguiente se muestra cómo acceder a los objetos `req` y `res` en `context`:

    ```javascript
    // You can access your HTTP request off the context ...
    if(context.req.body.emoji === ':pizza:') context.log('Yay!');
    // and also set your HTTP response
    context.res = { status: 202, body: 'You successfully ordered more coffee!' }; 
    ```

+ **Desde los enlaces de entrada y salida con nombre.** De esta manera, el desencadenador HTTP y los enlaces funcionan igual que cualquier otro enlace. En el ejemplo siguiente se establece el objeto de respuesta mediante un enlace `response` con nombre: 

    ```json
    {
        "type": "http",
        "direction": "out",
        "name": "response"
    }
    ```
    ```javascript
    context.bindings.response = { status: 201, body: "Insert succeeded." };
    ```
+ **_[Solo respuesta]_ Llamando a `context.res.send(body?: any)`.** Se crea una respuesta HTTP con la entrada `body` como cuerpo de la respuesta. Se llama a `context.done()` implícitamente.

+ **_[Solo respuesta]_ Llamando a `context.done()`.** Un tipo especial de enlace HTTP devuelve la respuesta que se pasa al método `context.done()`. El enlace de salida HTTP siguiente define un parámetro de salida `$return`:

    ```json
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
    ``` 
    ```javascript
     // Define a valid response object.
    res = { status: 201, body: "Insert succeeded." };
    context.done(null, res);   
    ```  

## <a name="scaling-and-concurrency"></a>Escalado y simultaneidad

De forma predeterminada, Azure Functions supervisa automáticamente la carga en la aplicación y crea instancias de host adicionales para Node.js según sea necesario. Functions usa umbrales integrados (no configurables por el usuario) en diferentes tipos de desencadenadores para decidir cuándo se deben agregar instancias, como la antigüedad de los mensajes y el tamaño de la cola para QueueTrigger. Para obtener más información, consulte [Cómo funcionan los planes de consumo y Premium](functions-scale.md#how-the-consumption-and-premium-plans-work).

Este comportamiento de escalado es suficiente para muchas aplicaciones de Node.js. En las aplicaciones dependientes de la CPU, puede mejorar aún más el rendimiento mediante el uso de varios procesos de trabajo de lenguaje.

De forma predeterminada, cada instancia de host de Functions tiene un único proceso de trabajo de lenguaje. Puede aumentar el número de procesos de trabajo por host (hasta 10) mediante la configuración de la aplicación [FUNCTIONS_WORKER_PROCESS_COUNT](functions-app-settings.md#functions_worker_process_count). Al hacerlo, Azure Functions intenta distribuir uniformemente las invocaciones de función simultáneas en estos trabajos. 

FUNCTIONS_WORKER_PROCESS_COUNT se aplica a cada host que Functions crea al escalar horizontalmente la aplicación para satisfacer la demanda. 

## <a name="node-version"></a>Versión de Node

En la tabla siguiente se muestran las versiones actuales de Node. js compatibles para cada versión principal del runtime de Functions, por sistema operativo:

| Versión de Functions | Versión de Node (Windows) | Versión de Node (Linux) |
|---|---| --- |
| 1.x | 6.11.2 (bloqueado por el entorno de tiempo de ejecución) | N/D |
| 2.x  | ~8<br/>~10 (recomendado)<br/>~12<sup>*</sup> | ~8 (recomendado)<br/>~10  |
| 3.x | ~10<br/>~12 (recomendado)  | ~10<br/>~12 (recomendado) |

<sup>*</sup>Node ~12 está permitido en la actualidad en la versión 2.x del runtime de Functions. De todas formas, para obtener el mejor rendimiento, se recomienda usar la versión 3.x del runtime de Functions con Node ~12. 

Puede ver la versión actual que el entorno de tiempo de ejecución usa consultando la configuración de aplicación anterior o mediante la impresión de `process.version` desde cualquier función. Seleccione el destino de la versión en Azure estableciendo la [configuración de aplicación](functions-how-to-use-azure-function-app-settings.md#settings) WEBSITE_NODE_DEFAULT_VERSION en una versión compatible con LTS, como `~10`.

## <a name="dependency-management"></a>Administración de dependencias
Para poder utilizar bibliotecas de la comunidad en el código de JavaScript, como se muestra en el ejemplo siguiente, debe asegurarse de que todas las dependencias estén instaladas en Function App en Azure.

```javascript
// Import the underscore.js library
var _ = require('underscore');
var version = process.version; // version === 'v6.5.0'

module.exports = function(context) {
    // Using our imported underscore.js library
    var matched_names = _
        .where(context.bindings.myInput.names, {first: 'Carla'});
```

> [!NOTE]
> Debe definir un archivo `package.json` en la raíz de Function App. La definición del archivo permite que todas las funciones de la aplicación compartan los mismos paquetes almacenados en caché, de tal manera que se ofrece el mejor rendimiento. Cuando hay conflictos con una versión, puede resolverlo mediante la adición de un archivo `package.json` en la carpeta de una función específica.  

Al implementar aplicaciones de función desde el control de código fuente, cualquier archivo `package.json` presente en el repositorio desencadenará un elemento `npm install` en su carpeta durante la implementación. Sin embargo, al implementarlas mediante el portal o la CLI, tendrá que instalar los paquetes manualmente.

Hay dos maneras de instalar paquetes en Function App: 

### <a name="deploying-with-dependencies"></a>Implementación con dependencias
1. Instale todos los paquetes necesarios localmente mediante la ejecución de `npm install`.

2. Implemente el código y asegúrese de que la carpeta `node_modules` se incluye en la implementación. 


### <a name="using-kudu"></a>Mediante Kudu
1. Ir a `https://<function_app_name>.scm.azurewebsites.net`.

2. Haga clic en **Consola de depuración** > **CMD**.

3. Vaya a `D:\home\site\wwwroot` y luego arrastre el archivo package.json a la carpeta **wwwroot** en la mitad superior de la página.  
    También puede cargar archivos en Function App de otras formas. Para obtener más información, vea [Actualización de los archivos de aplicación de función](functions-reference.md#fileupdate). 

4. Una vez cargado el archivo package.json, ejecute el comando`npm install` en la **consola de ejecución remota de Kudu**.  
    Esta acción descarga los paquetes indicados en el archivo package.json y se reinicia Function App.

## <a name="environment-variables"></a>Variables de entorno

En Functions, [la configuración de la aplicación](functions-app-settings.md), como las cadenas de conexión del servicio, se exponen como variables de entorno durante la ejecución. Puede acceder a esta configuración mediante `process.env`, como se muestra aquí en la segunda y tercera llamada a `context.log()` donde se registran las variables de entorno `AzureWebJobsStorage` y `WEBSITE_SITE_NAME`:

```javascript
module.exports = async function (context, myTimer) {
    var timeStamp = new Date().toISOString();

    context.log('Node.js timer trigger function ran!', timeStamp);
    context.log("AzureWebJobsStorage: " + process.env["AzureWebJobsStorage"]);
    context.log("WEBSITE_SITE_NAME: " + process.env["WEBSITE_SITE_NAME"]);
};
```

[!INCLUDE [Function app settings](../../includes/functions-app-settings.md)]

Cuando se ejecuta localmente, la configuración de la aplicación se lee desde el archivo del proyecto [local.settings.json](functions-run-local.md#local-settings-file).

## <a name="configure-function-entry-point"></a>Configuración del punto de entrada de la función

Las propiedades de `function.json``scriptFile` y `entryPoint` pueden usarse para configurar la ubicación y el nombre de la función exportada. Estas propiedades pueden ser importantes si se transpila el código JavaScript.

### <a name="using-scriptfile"></a>Usar `scriptFile`

De forma predeterminada, se ejecuta una función de JavaScript desde `index.js`, un archivo que comparte el mismo directorio primario que su archivo `function.json` correspondiente.

`scriptFile` se puede usar para obtener una estructura de carpetas que tenga el aspecto del ejemplo siguiente:

```
FunctionApp
 | - host.json
 | - myNodeFunction
 | | - function.json
 | - lib
 | | - sayHello.js
 | - node_modules
 | | - ... packages ...
 | - package.json
```

`function.json` para `myNodeFunction` debe incluir una propiedad `scriptFile` que señala al archivo con la función exportada que ejecutar.

```json
{
  "scriptFile": "../lib/sayHello.js",
  "bindings": [
    ...
  ]
}
```

### <a name="using-entrypoint"></a>Usar `entryPoint`

En `scriptFile` (o `index.js`), una función debe exportarse con `module.exports` para que se pueda encontrar y ejecutar. De forma predeterminada, la función que se ejecuta cuando se desencadena es la única exportación de ese archivo, la exportación denominada `run` o la exportación denominada `index`.

Se puede configurar mediante `entryPoint` en `function.json`, como en el ejemplo siguiente:

```json
{
  "entryPoint": "logFoo",
  "bindings": [
    ...
  ]
}
```

En Functions 2.x, que admite el parámetro `this` en funciones de usuario, el código de la función podría ser similar al del ejemplo siguiente:

```javascript
class MyObj {
    constructor() {
        this.foo = 1;
    };

    logFoo(context) { 
        context.log("Foo is " + this.foo); 
        context.done(); 
    }
}

const myObj = new MyObj();
module.exports = myObj;
```

En este ejemplo, es importante tener en cuenta que, aunque se exporta un objeto, no hay ninguna garantía de que se conserve el estado entre ejecuciones.

## <a name="local-debugging"></a>Depuración local

Cuando se inicia con el parámetro `--inspect`, un proceso de Node.js escucha un cliente de depuración en el puerto especificado. En Azure Functions 2.x, puede especificar argumentos para pasar al proceso de Node.js que ejecuta el código mediante la adición de la variable de entorno o la configuración de la aplicación `languageWorkers:node:arguments = <args>`. 

Para depurar de forma local, agregue `"languageWorkers:node:arguments": "--inspect=5858"` en `Values` en su archivo [local.settings.json](./functions-run-local.md#local-settings-file) y asocie un depurador al puerto 5858.

Al depurar con VS Code, el parámetro `--inspect` se agrega automáticamente mediante el valor `port` del archivo launch.json del proyecto.

En la versión 1.x, no funciona el valor de configuración `languageWorkers:node:arguments`. El puerto de depuración se puede seleccionar con el parámetro [`--nodeDebugPort`](./functions-run-local.md#start) en Azure Functions Core Tools.

## <a name="typescript"></a>TypeScript

Cuando el destino es la versión 2.x del sistema en tiempo de ejecución de Functions, tanto [Azure Functions para Visual Studio Code](functions-create-first-function-vs-code.md) como [Azure Functions Core Tools](functions-run-local.md) le permiten crear aplicaciones de función mediante una plantilla que admita proyectos de aplicación de función de TypeScript. La plantilla genera archivos de proyecto `package.json` y `tsconfig.json` que facilitan la transpilación, la ejecución y la publicación de funciones de JavaScript desde el código de TypeScript con estas herramientas.

El archivo `.funcignore` generado se usa para indicar qué archivos se excluyen cuando se publica un proyecto en Azure.  

Los archivos de TypeScript (TS) se transpilan en archivos de JavaScript (.js) en el directorio de salida `dist`. Las plantillas de TypeScript usan el parámetro [`scriptFile`](#using-scriptfile) en `function.json` para indicar la ubicación del archivo .js correspondiente en la carpeta `dist`. La ubicación de salida se establece mediante la plantilla con el parámetro `outDir` en el archivo `tsconfig.json`. Si cambia esta configuración o el nombre de la carpeta, el entorno de ejecución no podrá encontrar el código que se ejecutará.

La manera en que desarrolle e implemente de forma local a partir de un proyecto de TypeScript depende de la herramienta de desarrollo.

### <a name="visual-studio-code"></a>Visual Studio Code

La extensión [Azure Functions para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) le permite desarrollar las funciones con TypeScript. Core Tools es un requisito de la extensión de Azure Functions.

Para crear una aplicación de función de TypeScript en Visual Studio Code, elija `TypeScript` como lenguaje al crear una aplicación de función.

Al presionar **F5** para ejecutar la aplicación localmente, se realiza la transpilación antes de que se inicialice el host (func.exe). 

Cuando se implementa la aplicación de función en Azure mediante el botón **Deploy to function app...**  (Implementar en la aplicación de función), la extensión de Azure Functions genera primero una compilación lista para producción de los archivos JavaScript a partir de los archivos de origen de TypeScript.

### <a name="azure-functions-core-tools"></a>Azure Functions Core Tools

Hay varias cuestiones en las que un proyecto de TypeScript difiere de un proyecto de JavaScript cuando se usa Core Tools.

#### <a name="create-project"></a>Crear proyecto

Para crear un proyecto de aplicación de función de TypeScript con Core Tools, debe especificar la opción de lenguaje TypeScript al crear la aplicación de función. Para ello, siga uno de estos métodos:

- Ejecute el comando `func init`, seleccione `node` como la pila de lenguaje y, luego, seleccione `typescript`.

- Ejecute el comando `func init --worker-runtime typescript`.

#### <a name="run-local"></a>Ejecución local

Para ejecutar el código de la aplicación de función localmente mediante Core Tools, use los comandos siguientes en lugar de `func host start`: 

```command
npm install
npm start
```

El comando `npm start` es equivalente a los siguientes comandos:

- `npm run build`
- `func extensions install`
- `tsc`
- `func start`

#### <a name="publish-to-azure"></a>Publicación en Azure

Antes de usar el comando [`func azure functionapp publish`] para implementar en Azure, puede crear una compilación lista para producción de archivos de JavaScript a partir de los archivos de origen de TypeScript. 

Los siguientes comandos preparan y publican el proyecto de TypeScript mediante Core Tools: 

```command
npm run build:production 
func azure functionapp publish <APP_NAME>
```

En este comando, reemplace `<APP_NAME>` por el nombre de la aplicación de función.

## <a name="considerations-for-javascript-functions"></a>Consideraciones para las funciones de JavaScript

Cuando se trabaja con las funciones de JavaScript, tenga en cuenta las consideraciones de las secciones siguientes.

### <a name="choose-single-vcpu-app-service-plans"></a>Elección de los planes de App Service de una sola vCPU

Al crear una aplicación de función que usa el plan de App Service, se recomienda que seleccione un plan de una sola vCPU, en lugar de un plan con varias vCPU. En la actualidad, Functions ejecuta funciones de JavaScript con más eficacia en VM con una sola vCPU; el uso de máquinas virtuales más grandes no produce las mejoras de rendimiento esperadas. Cuando sea necesario, puede escalar horizontalmente de forma manual mediante la adición de más instancias de máquina virtual de una sola vCPU o bien puede habilitar el escalado automático. Para obtener más información, consulte [Escalación del recuento de instancias de forma manual o automática](../azure-monitor/platform/autoscale-get-started.md?toc=/azure/app-service/toc.json).

### <a name="cold-start"></a>Arranque en frío

Al desarrollar Azure Functions en el modelo de hospedaje sin servidor, los arranques en frío son una realidad. *Arranque en frío* hace referencia al hecho de que cuando se inicia Function App por primera vez tras un período de inactividad, tarda más tiempo en iniciarse. Especialmente para las funciones de JavaScript con árboles de dependencias de gran tamaño, el arranque en frío puede ser importante. Para acelerar el proceso de arranque en frío, [ejecute sus funciones como un archivo de paquete](run-functions-from-deployment-package.md) cuando sea posible. Muchos métodos de implementación utilizan el modelo de ejecución desde paquete de forma predeterminada, pero si experimenta arranques en frío prolongados y no ejecuta sus funciones de este modo, este cambio puede mejorar la situación considerablemente.

### <a name="connection-limits"></a>Límites de conexión

Cuando use un cliente específico del servicio en una aplicación de Azure Functions, no cree un cliente con cada invocación de función. En su lugar, cree un único cliente estático en el ámbito global. Para más información, consulte [Administrar conexiones en Azure Functions](manage-connections.md).

### <a name="use-async-and-await"></a>Uso de `async` y `await`

Al escribir funciones de Azure en JavaScript, debe escribir el código mediante las palabras clave `async` y `await`. El hecho de escribir el código con `async` y `await` en lugar de usar devoluciones de llamada o `.then` y `.catch` con promesas ayuda a evitar dos problemas comunes:
 - El inicio de excepciones no detectadas que [bloquean el proceso de Node.js](https://nodejs.org/api/process.html#process_warning_using_uncaughtexception_correctly), que puede afectar a la ejecución de otras funciones.
 - Un comportamiento inesperado, como la ausencia de registros de context.log debido a llamadas asincrónicas que no se esperan correctamente.

En el ejemplo siguiente, se invoca el método asincrónico `fs.readFile` con una función de devolución de llamada error-first como segundo parámetro. Este código provoca los dos problemas mencionados anteriormente. Una excepción no detectada explícitamente en el ámbito correcto bloquea todo el proceso (problema 1). Una llamada a `context.done()` fuera del ámbito de la función de devolución de llamada significa que la invocación de la función puede finalizar antes de que se lea el archivo (problema 2). En este ejemplo, una llamada a `context.done()` demasiado pronto provoca la ausencia de las entradas de registro que empiezan por `Data from file:`.

```javascript
// NOT RECOMMENDED PATTERN
const fs = require('fs');

module.exports = function (context) {
    fs.readFile('./hello.txt', (err, data) => {
        if (err) {
            context.log.error('ERROR', err);
            // BUG #1: This will result in an uncaught exception that crashes the entire process
            throw err;
        }
        context.log(`Data from file: ${data}`);
        // context.done() should be called here
    });
    // BUG #2: Data is not guaranteed to be read before the Azure Function's invocation ends
    context.done();
}
```

El uso de las palabras clave `async` y `await` ayuda a evitar ambos errores. Debe usar la función de utilidad [`util.promisify`](https://nodejs.org/api/util.html#util_util_promisify_original) de Node.js para convertir las funciones de estilo de devolución de llamada error-first en funciones que admiten await.

En el ejemplo siguiente, las excepciones no controladas iniciadas durante la ejecución de la función solo errarán la invocación individual que generó una excepción. La palabra clave `await` significa que los pasos después de `readFileAsync` solo se ejecutarán una vez finalizado `readFile`. Con `async` y `await`, tampoco es necesario llamar a la devolución de llamada `context.done()`.

```javascript
// Recommended pattern
const fs = require('fs');
const util = require('util');
const readFileAsync = util.promisify(fs.readFile);

module.exports = async function (context) {
    let data;
    try {
        data = await readFileAsync('./hello.txt');
    } catch (err) {
        context.log.error('ERROR', err);
        // This rethrown exception will be handled by the Functions Runtime and will only fail the individual invocation
        throw err;
    }
    context.log(`Data from file: ${data}`);
}
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte los siguientes recursos:

+ [Procedimientos recomendados para Azure Functions](functions-best-practices.md)
+ [Referencia para desarrolladores de Azure Functions](functions-reference.md)
+ [Enlaces y desencadenadores de Azure Functions](functions-triggers-bindings.md)

["func azure functionapp publish"]: functions-run-local.md#project-file-deployment