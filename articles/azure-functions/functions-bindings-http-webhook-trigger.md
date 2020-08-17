---
title: Desencadenador HTTP de Azure Functions
description: Obtenga información sobre cómo llamar a una función de Azure a través de HTTP.
author: craigshoemaker
ms.topic: reference
ms.date: 02/21/2020
ms.author: cshoe
ms.custom: devx-track-python
ms.openlocfilehash: 271787c47b29bc713cc923c7d27852a8e05703ef
ms.sourcegitcommit: 1a0dfa54116aa036af86bd95dcf322307cfb3f83
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88042554"
---
# <a name="azure-functions-http-trigger"></a>Desencadenador HTTP de Azure Functions

El desencadenador HTTP permite invocar una función con una solicitud HTTP. Puede usar un desencadenador HTTP para crear API sin servidor y responder a webhooks.

El valor devuelto predeterminado para una función desencadenada por HTTP es:

- `HTTP 204 No Content` con el cuerpo vacío en Functions 2.x y posterior
- `HTTP 200 OK` con el cuerpo vacío en Functions 1.x

Para modificar la respuesta HTTP, configure un [enlace de salida](./functions-bindings-http-webhook-output.md).

Para obtener más información sobre los enlaces HTTP, consulte la [información general](./functions-bindings-http-webhook.md) y la [referencia sobre enlaces de salida](./functions-bindings-http-webhook-output.md).

[!INCLUDE [HTTP client best practices](../../includes/functions-http-client-best-practices.md)]

## <a name="example"></a>Ejemplo

# <a name="c"></a>[C#](#tab/csharp)

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que busca un parámetro `name` en la cadena de consulta o en el cuerpo de la solicitud HTTP. Tenga en cuenta que el valor devuelto se utiliza para el enlace de salida, pero no se requiere un atributo de valor devuelto.

```cs
[FunctionName("HttpTriggerCSharp")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] 
    HttpRequest req, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    string name = req.Query["name"];

    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    dynamic data = JsonConvert.DeserializeObject(requestBody);
    name = name ?? data?.name;

    return name != null
        ? (ActionResult)new OkObjectResult($"Hello, {name}")
        : new BadRequestObjectResult("Please pass a name on the query string or in the request body");
}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

En el ejemplo siguiente se muestra un enlace de desencadenador en un archivo *function.json* y una [función de script de C#](functions-reference-csharp.md) que usa el enlace. La función busca un parámetro `name` en la cadena de consulta o en el cuerpo de la solicitud HTTP.

Este es el archivo *function.json*:

```json
{
    "disabled": false,
    "bindings": [
        {
            "authLevel": "function",
            "name": "req",
            "type": "httpTrigger",
            "direction": "in",
            "methods": [
                "get",
                "post"
            ]
        },
        {
            "name": "$return",
            "type": "http",
            "direction": "out"
        }
    ]
}
```

En la sección de [configuración](#configuration) se explican estas propiedades.

Este es el código de script de C# que se enlaza a `HttpRequest`:

```cs
#r "Newtonsoft.Json"

using System.Net;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Primitives;
using Newtonsoft.Json;

public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    string name = req.Query["name"];

    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    dynamic data = JsonConvert.DeserializeObject(requestBody);
    name = name ?? data?.name;

    return name != null
        ? (ActionResult)new OkObjectResult($"Hello, {name}")
        : new BadRequestObjectResult("Please pass a name on the query string or in the request body");
}
```

Puede enlazar a un objeto personalizado en lugar de `HttpRequest`. Este objeto se crea a partir del cuerpo de la solicitud y se analiza como JSON. Del mismo modo, puede pasarse un tipo al enlace de la salida de respuesta HTTP y se devuelve como el cuerpo de la respuesta, con el código de estado `200`.

```csharp
using System.Net;
using System.Threading.Tasks;
using Microsoft.Extensions.Logging;

public static string Run(Person person, ILogger log)
{   
    return person.Name != null
        ? (ActionResult)new OkObjectResult($"Hello, {person.Name}")
        : new BadRequestObjectResult("Please pass an instance of Person.");
}

public class Person {
     public string Name {get; set;}
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

En el ejemplo siguiente se muestra un enlace de desencadenador en un archivo *function.json* y una [función de JavaScript](functions-reference-node.md) que usa el enlace. La función busca un parámetro `name` en la cadena de consulta o en el cuerpo de la solicitud HTTP.

Este es el archivo *function.json*:

```json
{
    "disabled": false,    
    "bindings": [
        {
            "authLevel": "function",
            "type": "httpTrigger",
            "direction": "in",
            "name": "req"
        },
        {
            "type": "http",
            "direction": "out",
            "name": "res"
        }
    ]
}
```

En la sección de [configuración](#configuration) se explican estas propiedades.

Este es el código de JavaScript:

```javascript
module.exports = function(context, req) {
    context.log('Node.js HTTP trigger function processed a request. RequestUri=%s', req.originalUrl);

    if (req.query.name || (req.body && req.body.name)) {
        context.res = {
            // status defaults to 200 */
            body: "Hello " + (req.query.name || req.body.name)
        };
    }
    else {
        context.res = {
            status: 400,
            body: "Please pass a name on the query string or in the request body"
        };
    }
    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

En el ejemplo siguiente se muestra un enlace de desencadenador en un archivo *function.json* y una [función de Python](functions-reference-python.md) que usa el enlace. La función busca un parámetro `name` en la cadena de consulta o en el cuerpo de la solicitud HTTP.

Este es el archivo *function.json*:

```json
{
    "scriptFile": "__init__.py",
    "disabled": false,    
    "bindings": [
        {
            "authLevel": "function",
            "type": "httpTrigger",
            "direction": "in",
            "name": "req"
        },
        {
            "type": "http",
            "direction": "out",
            "name": "res"
        }
    ]
}
```

En la sección de [configuración](#configuration) se explican estas propiedades.

Este es el código de Python:

```python
import logging
import azure.functions as func


def main(req: func.HttpRequest) -> func.HttpResponse:
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
        return func.HttpResponse(f"Hello {name}!")
    else:
        return func.HttpResponse(
            "Please pass a name on the query string or in the request body",
            status_code=400
        )
```

# <a name="java"></a>[Java](#tab/java)

* [Leer el parámetro de la cadena de consulta](#read-parameter-from-the-query-string)
* [Leer el cuerpo de una solicitud POST](#read-body-from-a-post-request)
* [Leer el parámetro de una ruta](#read-parameter-from-a-route)
* [Leer el cuerpo POJO de una solicitud POST](#read-pojo-body-from-a-post-request)

En los ejemplos siguientes, se muestra el enlace del desencadenador HTTP.

#### <a name="read-parameter-from-the-query-string"></a>Lectura del parámetro desde la cadena de consulta

En este ejemplo se lee un parámetro, denominado `id`, desde la cadena de consulta, y se usa para generar un documento JSON devuelto al cliente, con el tipo de contenido `application/json`.

```java
@FunctionName("TriggerStringGet")
public HttpResponseMessage run(
        @HttpTrigger(name = "req", 
            methods = {HttpMethod.GET}, 
            authLevel = AuthorizationLevel.ANONYMOUS)
        HttpRequestMessage<Optional<String>> request,
        final ExecutionContext context) {
    
    // Item list
    context.getLogger().info("GET parameters are: " + request.getQueryParameters());

    // Get named parameter
    String id = request.getQueryParameters().getOrDefault("id", "");

    // Convert and display
    if (id.isEmpty()) {
        return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                        .body("Document not found.")
                        .build();
    } 
    else {
        // return JSON from to the client
        // Generate document
        final String name = "fake_name";
        final String jsonDocument = "{\"id\":\"" + id + "\", " + 
                                        "\"description\": \"" + name + "\"}";
        return request.createResponseBuilder(HttpStatus.OK)
                        .header("Content-Type", "application/json")
                        .body(jsonDocument)
                        .build();
    }
}
```

#### <a name="read-body-from-a-post-request"></a>Lectura del cuerpo de una solicitud POST

En este ejemplo se lee el cuerpo de una solicitud POST, como `String`, y se usa para generar un documento JSON devuelto al cliente, con el tipo de contenido `application/json`.

```java
    @FunctionName("TriggerStringPost")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req", 
              methods = {HttpMethod.POST}, 
              authLevel = AuthorizationLevel.ANONYMOUS)
            HttpRequestMessage<Optional<String>> request,
            final ExecutionContext context) {
        
        // Item list
        context.getLogger().info("Request body is: " + request.getBody().orElse(""));

        // Check request body
        if (!request.getBody().isPresent()) {
            return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                          .body("Document not found.")
                          .build();
        } 
        else {
            // return JSON from to the client
            // Generate document
            final String body = request.getBody().get();
            final String jsonDocument = "{\"id\":\"123456\", " + 
                                         "\"description\": \"" + body + "\"}";
            return request.createResponseBuilder(HttpStatus.OK)
                          .header("Content-Type", "application/json")
                          .body(jsonDocument)
                          .build();
        }
    }
```

#### <a name="read-parameter-from-a-route"></a>Lectura del parámetro de una ruta

En este ejemplo se lee un parámetro obligatorio, denominado `id`, y un parámetro opcional `name` de la ruta de acceso, y se usa para generar un documento JSON devuelto al cliente, con el tipo de contenido `application/json`. T

```java
@FunctionName("TriggerStringRoute")
public HttpResponseMessage run(
        @HttpTrigger(name = "req", 
            methods = {HttpMethod.GET}, 
            authLevel = AuthorizationLevel.ANONYMOUS,
            route = "trigger/{id}/{name=EMPTY}") // name is optional and defaults to EMPTY
        HttpRequestMessage<Optional<String>> request,
        @BindingName("id") String id,
        @BindingName("name") String name,
        final ExecutionContext context) {
    
    // Item list
    context.getLogger().info("Route parameters are: " + id);

    // Convert and display
    if (id == null) {
        return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                        .body("Document not found.")
                        .build();
    } 
    else {
        // return JSON from to the client
        // Generate document
        final String jsonDocument = "{\"id\":\"" + id + "\", " + 
                                        "\"description\": \"" + name + "\"}";
        return request.createResponseBuilder(HttpStatus.OK)
                        .header("Content-Type", "application/json")
                        .body(jsonDocument)
                        .build();
    }
}
```

#### <a name="read-pojo-body-from-a-post-request"></a>Lectura del cuerpo POJO de una solicitud POST

Este es el código para la clase `ToDoItem`, a la que se hace referencia en este ejemplo:

```java

public class ToDoItem {

  private String id;
  private String description;  

  public ToDoItem(String id, String description) {
    this.id = id;
    this.description = description;
  }

  public String getId() {
    return id;
  }

  public String getDescription() {
    return description;
  }
  
  @Override
  public String toString() {
    return "ToDoItem={id=" + id + ",description=" + description + "}";
  }
}

```

En este ejemplo se lee el cuerpo de una solicitud POST. El cuerpo de la solicitud se deserializa automáticamente en un objeto `ToDoItem` y se devuelve al cliente, con el tipo de contenido `application/json`. El parámetro `ToDoItem` se serializa en runtime de Functions, ya que está asignado a la propiedad `body` de la clase `HttpMessageResponse.Builder`.

```java
@FunctionName("TriggerPojoPost")
public HttpResponseMessage run(
        @HttpTrigger(name = "req", 
            methods = {HttpMethod.POST}, 
            authLevel = AuthorizationLevel.ANONYMOUS)
        HttpRequestMessage<Optional<ToDoItem>> request,
        final ExecutionContext context) {
    
    // Item list
    context.getLogger().info("Request body is: " + request.getBody().orElse(null));

    // Check request body
    if (!request.getBody().isPresent()) {
        return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                        .body("Document not found.")
                        .build();
    } 
    else {
        // return JSON from to the client
        // Generate document
        final ToDoItem body = request.getBody().get();
        return request.createResponseBuilder(HttpStatus.OK)
                        .header("Content-Type", "application/json")
                        .body(body)
                        .build();
    }
}
```

---

## <a name="attributes-and-annotations"></a>Atributos y anotaciones

En las [bibliotecas de clases de C#](functions-dotnet-class-library.md) y en Java, el atributo `HttpTrigger` está disponible para configurar la función.

Puede establecer el nivel de autorización y los métodos HTTP permitidos en los parámetros del constructor de atributo, así como la plantilla de ruta y el tipo de webhook. Para obtener más información sobre esta configuración, consulte [configuración](#configuration).

# <a name="c"></a>[C#](#tab/csharp)

En este ejemplo se muestra cómo usar el atributo [HttpTrigger](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/dev/src/WebJobs.Extensions.Http/HttpTriggerAttribute.cs).

```csharp
[FunctionName("HttpTriggerCSharp")]
public static Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Anonymous)] HttpRequest req)
{
    ...
}
```

Para obtener un ejemplo completo, vea el [ejemplo del desencadenador](#example).

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

El script de C# no admite atributos.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

JavaScript no admite atributos.

# <a name="python"></a>[Python](#tab/python)

Python no admite atributos.

# <a name="java"></a>[Java](#tab/java)

En este ejemplo se muestra cómo usar el atributo [HttpTrigger](https://github.com/Azure/azure-functions-java-library/blob/dev/src/main/java/com/microsoft/azure/functions/annotation/HttpTrigger.java).

```java
@FunctionName("HttpTriggerJava")
public HttpResponseMessage<String> HttpTrigger(
        @HttpTrigger(name = "req",
                     methods = {"get"},
                     authLevel = AuthorizationLevel.ANONYMOUS) HttpRequestMessage<String> request,
        final ExecutionContext context) {

    ...
}
```

Para obtener un ejemplo completo, vea el [ejemplo del desencadenador](#example).

---

## <a name="configuration"></a>Configuración

En la siguiente tabla se explican las propiedades de configuración de enlace que se definen en el archivo *function.json* y el atributo `HttpTrigger`.

|Propiedad de function.json | Propiedad de atributo |Descripción|
|---------|---------|----------------------|
| **type** | N/D| Requerida: se debe establecer en `httpTrigger`. |
| **direction** | N/D| Requerida: se debe establecer en `in`. |
| **name** | N/D| Requerida: nombre de variable que se usa en el código de la función para la solicitud o el cuerpo de la solicitud. |
| <a name="http-auth"></a>**authLevel** |  **AuthLevel** |Determina qué claves, si las hubiera, deben estar presentes en la solicitud para poder invocar a la función. El nivel de autorización puede ser uno de los siguientes: <ul><li><code>anonymous</code>: no se requiere ninguna clave de API.</li><li><code>function</code>: se requiere una clave de API específica de la función. Este es el valor predeterminado si no se proporciona ninguno.</li><li><code>admin</code>: se requiere la clave maestra.</li></ul> Para más información, consulte la sección sobre las [claves de autorización](#authorization-keys). |
| **methods** |**Métodos** | Una matriz de los métodos HTTP a los que responde la función. Si no se especifica, la función responde a todos los métodos HTTP. Consulte cómo [personalizar el punto de conexión HTTP](#customize-the-http-endpoint). |
| **route** | **Route** | Define la plantilla de ruta y controla las direcciones URL de solicitud a las que responde la función. El valor predeterminado es `<functionname>` si no se proporciona ninguno. Para más información, consulte cómo [personalizar el punto de conexión HTTP](#customize-the-http-endpoint). |
| **webHookType** | **WebHookType** | _Compatible solo con la versión 1.x del runtime._<br/><br/>Configura el desencadenador HTTP para que actúe como un receptor de [webhook](https://en.wikipedia.org/wiki/Webhook) para el proveedor especificado. No establezca la propiedad `methods` si establece esta propiedad. El tipo de webhook puede ser uno de los valores siguientes:<ul><li><code>genericJson</code>: un punto de conexión de webhook de uso general sin lógica para un proveedor concreto. Este valor restringe las solicitudes a solo aquellas que usan HTTP POST y con el tipo de contenido `application/json`.</li><li><code>github</code>&mdash;La función responde a [webhooks de GitHub](https://developer.github.com/webhooks/). No use la propiedad _authLevel_ con webhooks de GitHub. Para más información, consulte la sección sobre webhooks de GitHub que aparece más adelante en este artículo.</li><li><code>slack</code>&mdash;La función responde a [webhooks de Slack](https://api.slack.com/outgoing-webhooks). No use la propiedad _authLevel_ con webhooks de Slack. Para más información, consulte la sección sobre webhooks de Slack que aparece más adelante en este artículo.</li></ul>|

## <a name="payload"></a>Carga

El tipo de entrada del desencadenador se declara como `HttpRequest` o como un tipo personalizado. Si elige `HttpRequest`, obtiene acceso completo al objeto de solicitud. En un tipo personalizado, el runtime intenta analizar el cuerpo de la solicitud JSON para establecer las propiedades del objeto.

## <a name="customize-the-http-endpoint"></a>Personalización del punto de conexión HTTP

De forma predeterminada, al crear una función para un desencadenador HTTP, la función se puede dirigir con una ruta que tenga el siguiente formato:

```http
http://<APP_NAME>.azurewebsites.net/api/<FUNCTION_NAME>
```

Puede personalizar esta ruta mediante el parámetro opcional `route` del enlace de entrada del desencadenador HTTP. Por ejemplo, el siguiente archivo *function.json* define una propiedad `route` para un desencadenador HTTP:

```json
{
    "bindings": [
    {
        "type": "httpTrigger",
        "name": "req",
        "direction": "in",
        "methods": [ "get" ],
        "route": "products/{category:alpha}/{id:int?}"
    },
    {
        "type": "http",
        "name": "res",
        "direction": "out"
    }
    ]
}
```

Al usar esta configuración, ya se podrá dirigir la función con la ruta de abajo, en lugar de con la original.

```
http://<APP_NAME>.azurewebsites.net/api/products/electronics/357
```

Esta configuración permite que el código de la función admita dos parámetros en la dirección: _category_ e _id_.

# <a name="c"></a>[C#](#tab/csharp)

Puede usar cualquier [restricción de ruta de API web](https://www.asp.net/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2#constraints) con sus parámetros. El siguiente código de función de C# emplea los dos parámetros.

```csharp
using System.Net;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Primitives;

public static IActionResult Run(HttpRequest req, string category, int? id, ILogger log)
{
    var message = String.Format($"Category: {category}, ID: {id}");
    return (ActionResult)new OkObjectResult(message);
}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

Puede usar cualquier [restricción de ruta de API web](https://www.asp.net/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2#constraints) con sus parámetros. El siguiente código de función de C# emplea los dos parámetros.

```csharp
#r "Newtonsoft.Json"

using System.Net;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Primitives;

public static IActionResult Run(HttpRequest req, string category, int? id, ILogger log)
{
    var message = String.Format($"Category: {category}, ID: {id}");
    return (ActionResult)new OkObjectResult(message);
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

En el nodo, el entorno de ejecución de Functions proporciona el cuerpo de la solicitud a partir del objeto `context`. Para más información, consulte el [ejemplo de desencadenador de JavaScript](#example).

En el ejemplo siguiente se muestra cómo leer los parámetros de ruta desde `context.bindingData`.

```javascript
module.exports = function (context, req) {

    var category = context.bindingData.category;
    var id = context.bindingData.id;
    var message = `Category: ${category}, ID: ${id}`;

    context.res = {
        body: message;
    }

    context.done();
}
```

# <a name="python"></a>[Python](#tab/python)

El contexto de ejecución de la función se expone a través de un parámetro declarado como `func.HttpRequest`. Esta instancia permite que una función acceda a parámetros de rutas de datos, valores de cadenas de consulta y métodos que permiten devolver respuestas HTTP.

Una vez definidos, los parámetros de ruta están disponibles para la función mediante la llamada al método `route_params`.

```python
import logging

import azure.functions as func

def main(req: func.HttpRequest) -> func.HttpResponse:

    category = req.route_params.get('category')
    id = req.route_params.get('id')
    message = f"Category: {category}, ID: {id}"

    return func.HttpResponse(message)
```

# <a name="java"></a>[Java](#tab/java)

El contexto de ejecución de la función son las propiedades declaradas en el atributo `HttpTrigger`. El atributo permite definir parámetros de ruta, niveles de autorización, verbos HTTP y la instancia de solicitud entrante.

Los parámetros de ruta se definen mediante el atributo `HttpTrigger`.

```java
package com.function;

import java.util.*;
import com.microsoft.azure.functions.annotation.*;
import com.microsoft.azure.functions.*;

public class HttpTriggerJava {
    public HttpResponseMessage<String> HttpTrigger(
            @HttpTrigger(name = "req",
                         methods = {"get"},
                         authLevel = AuthorizationLevel.FUNCTION,
                         route = "products/{category:alpha}/{id:int}") HttpRequestMessage<String> request,
            @BindingName("category") String category,
            @BindingName("id") int id,
            final ExecutionContext context) {

        String message = String.format("Category  %s, ID: %d", category, id);
        return request.createResponseBuilder(HttpStatus.OK).body(message).build();
    }
}
```

---

De forma predeterminada, todas las rutas de la función tienen el prefijo *api*. También puede personalizar o quitar el prefijo con la propiedad `extensions.http.routePrefix` del archivo [host.json](functions-host-json.md). En el ejemplo siguiente, se quita el prefijo de ruta *api* utilizando una cadena vacía del prefijo del archivo *host.json*.

```json
{
    "extensions": {
        "http": {
            "routePrefix": ""
        }
    }
}
```

## <a name="using-route-parameters"></a>Uso de parámetros de ruta

Los parámetros de ruta que definían el patrón `route` de una función están disponibles para cada enlace. Por ejemplo, si tiene una ruta definida como `"route": "products/{id}"`, un enlace de almacenamiento de tabla puede utilizar el valor del parámetro `{id}` en la configuración de enlace.

La configuración siguiente muestra cómo se pasa el parámetro `{id}` al elemento `rowKey` del enlace.

```json
{
    "type": "table",
    "direction": "in",
    "name": "product",
    "partitionKey": "products",
    "tableName": "products",
    "rowKey": "{id}"
}
```

## <a name="working-with-client-identities"></a>Uso de identidades de cliente

Si la aplicación de función está usando la [autenticación/autorización de App Service](../app-service/overview-authentication-authorization.md), puede ver información sobre los clientes autenticados desde el código. Esta información está disponible como [encabezados de solicitud insertados por la plataforma](../app-service/app-service-authentication-how-to.md#access-user-claims). 

También puede leer esta información desde los datos de enlace. Esta funcionalidad solo está disponible para el entorno en tiempo de ejecución de Functions en la versión 2.x y posteriores. Actualmente, también está disponible para lenguajes .NET.

# <a name="c"></a>[C#](#tab/csharp)

La información relacionada con los clientes autenticados está disponible como [ClaimsPrincipal](/dotnet/api/system.security.claims.claimsprincipal). ClaimsPrincipal está disponible como parte del contexto de solicitud, tal como se muestra en el ejemplo siguiente:

```csharp
using System.Net;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;

public static IActionResult Run(HttpRequest req, ILogger log)
{
    ClaimsPrincipal identities = req.HttpContext.User;
    // ...
    return new OkObjectResult();
}
```

Como alternativa, ClaimsPrincipal se puede incluir como un parámetro adicional en la firma de función:

```csharp
using System.Net;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;
using Newtonsoft.Json.Linq;

public static void Run(JObject input, ClaimsPrincipal principal, ILogger log)
{
    // ...
    return;
}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

La información relacionada con los clientes autenticados está disponible como [ClaimsPrincipal](/dotnet/api/system.security.claims.claimsprincipal). ClaimsPrincipal está disponible como parte del contexto de solicitud, tal como se muestra en el ejemplo siguiente:

```csharp
using System.Net;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;

public static IActionResult Run(HttpRequest req, ILogger log)
{
    ClaimsPrincipal identities = req.HttpContext.User;
    // ...
    return new OkObjectResult();
}
```

Como alternativa, ClaimsPrincipal se puede incluir como un parámetro adicional en la firma de función:

```csharp
#r "Newtonsoft.Json"

using System.Net;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;
using Newtonsoft.Json.Linq;

public static void Run(JObject input, ClaimsPrincipal principal, ILogger log)
{
    // ...
    return;
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

El usuario autenticado está disponible a través de [encabezados HTTP](../app-service/app-service-authentication-how-to.md#access-user-claims).

# <a name="python"></a>[Python](#tab/python)

El usuario autenticado está disponible a través de [encabezados HTTP](../app-service/app-service-authentication-how-to.md#access-user-claims).

# <a name="java"></a>[Java](#tab/java)

El usuario autenticado está disponible a través de [encabezados HTTP](../app-service/app-service-authentication-how-to.md#access-user-claims).

---

## <a name="function-access-keys"></a><a name="authorization-keys"></a>Claves de acceso de función

[!INCLUDE [functions-authorization-keys](../../includes/functions-authorization-keys.md)]

## <a name="obtaining-keys"></a>Obtención de claves

Las claves se almacenan como parte de la aplicación de función en Azure y se cifran en reposo. Para ver las claves, crear nuevas o asignarles nuevos valores, vaya a una de las funciones desencadenadas por HTTP de [Azure Portal](https://portal.azure.com) y seleccione **Administrar**.

![Administre las claves de función en el portal.](./media/functions-bindings-http-webhook/manage-function-keys.png)

Puede obtener claves de función mediante programación con las [API de administración de claves](https://github.com/Azure/azure-functions-host/wiki/Key-management-API).

## <a name="api-key-authorization"></a>Autorización de la clave de API

La mayoría de las plantillas de desencadenador HTTP requieren una clave de API en la solicitud. Por lo tanto, la solicitud HTTP normalmente se parece a la siguiente dirección URL:

```http
https://<APP_NAME>.azurewebsites.net/api/<FUNCTION_NAME>?code=<API_KEY>
```

La clave se puede incluir en una variable de cadena de consulta denominada `code`, como arriba. También puede incluirse en un encabezado HTTP `x-functions-key`. El valor de la clave puede ser cualquier clave de función definida para la función o cualquier clave de host.

Puede permitir solicitudes anónimas, que no requieren claves. También puede exigir que se use la clave maestra. Cambie el nivel de autorización predeterminado mediante la propiedad `authLevel` en el JSON de enlace. Para más información, consulte [Desencadenador: configuración](#configuration).

> [!NOTE]
> Cuando las funciones se ejecutan localmente, la autorización se deshabilita independientemente del valor del nivel de autorización especificado. Después de publicar en Azure, se aplica el valor `authLevel` del desencadenador. Las claves siguen siendo necesarias cuando se ejecutan [localmente en un contenedor](functions-create-function-linux-custom-image.md#build-the-container-image-and-test-locally).


## <a name="secure-an-http-endpoint-in-production"></a>Proteger un punto de conexión HTTP en producción

Para proteger totalmente los puntos de conexión de función en producción, considere la posibilidad de implementar una de las opciones de seguridad siguientes de nivel de aplicación de función. Cuando use alguno de estos métodos de seguridad de nivel de aplicación de función, debe establecer el nivel de autorización de función desencadenado por HTTP en `anonymous`.

[!INCLUDE [functions-enable-auth](../../includes/functions-enable-auth.md)]

#### <a name="deploy-your-function-app-in-isolation"></a>Implementación de la aplicación de funciones en aislamiento

[!INCLUDE [functions-deploy-isolation](../../includes/functions-deploy-isolation.md)]

## <a name="webhooks"></a>webhooks

> [!NOTE]
> El modo de webhook solo está disponible para la versión 1.x del runtime de Functions. Este cambio se realizó para mejorar el rendimiento de los desencadenadores HTTP en la versión 2.x y posteriores.

En la versión 1.x, las plantillas de webhook proporcionan una validación adicional para las cargas de webhook. En la versión 2.x y posteriores, el desencadenador HTTP base todavía funciona y es el modo recomendado para webhooks. 

### <a name="github-webhooks"></a>Webhooks de GitHub

Para responder a webhooks de GitHub, primero cree la función con un desencadenador HTTP y establezca la propiedad **webHookType** en `github`. Luego copie su dirección URL y clave de API en la página **Agregar webhook** del repositorio GitHub. 

![Captura de pantalla que muestra cómo agregar un webhook para la función.](./media/functions-bindings-http-webhook/github-add-webhook.png)

### <a name="slack-webhooks"></a>Webhooks de Slack

El webhook de Slack genera un token en lugar de permitirle especificarlo, por lo que debe configurar una clave específica de función con el token desde Slack. Consulte [Claves de autorización](#authorization-keys).

## <a name="webhooks-and-keys"></a>Webhooks y claves

La autorización de webhook se controla mediante el componente receptor de webhook, parte del desencadenador HTTP; el mecanismo varía según el tipo de webhook. Cada mecanismo se basa en una clave. De forma predeterminada, se usa la clave de función denominada "default". Para usar otra clave, configure el proveedor de webhook de modo que envíe el nombre de la clave con la solicitud de una de las siguientes maneras:

* **Cadena de consulta**: el proveedor pasa el nombre de la clave en el parámetro de la cadena de consulta `clientid`, como `https://<APP_NAME>.azurewebsites.net/api/<FUNCTION_NAME>?clientid=<KEY_NAME>`.
* **Encabezado de solicitud**: el proveedor pasa el nombre de clave en el encabezado `x-functions-clientid`.

## <a name="content-types"></a>Tipos de contenido

Para pasar datos binarios y de formulario a una función que no es de C#, es necesario utilizar el encabezado Content-Type adecuado. Los tipos de contenido admitidos incluyen `octet-stream` para los datos binarios y [tipos de varias partes](https://www.iana.org/assignments/media-types/media-types.xhtml#multipart).

### <a name="known-issues"></a>Problemas conocidos

En las funciones que no son de C#, las solicitudes enviadas con el tipo de contenido `image/jpeg` producen un valor `string` pasado a la función. En casos como estos, puede convertir manualmente el valor `string` en una matriz de bytes para acceder a los datos binarios sin formato.

## <a name="limits"></a>Límites

La longitud de la solicitud HTTP está limitada a 100 MB (104 857 600 bytes) y la longitud de la dirección URL a 4 KB (4 096 bytes). El elemento `httpRuntime` del [archivo Web.config](https://github.com/Azure/azure-functions-host/blob/v3.x/src/WebJobs.Script.WebHost/web.config) especifica estos límites.

Si una función que usa el desencadenador HTTP no se completa en menos de 230 segundos, [Azure Load Balancer](../app-service/faq-availability-performance-application-issues.md#why-does-my-request-time-out-after-230-seconds) agotará el tiempo de espera y devolverá un error HTTP 502. La función seguirá ejecutándose, pero no podrá devolver una respuesta HTTP. En el caso de funciones de ejecución prolongada, se recomienda que siga patrones asincrónicos y que devuelva una ubicación donde pueda hacer ping con el estado de la solicitud. Para más información sobre cuánto tiempo se puede ejecutar una función, consulte [Escalado y hospedaje: Plan de consumo](functions-scale.md#timeout).


## <a name="next-steps"></a>Pasos siguientes

- [Devolución de una respuesta HTTP desde una función](./functions-bindings-http-webhook-output.md)
