---
title: 'Tutorial: Envío de correo electrónico con Logic Apps'
description: Aprenda a invocar procesos empresariales desde una aplicación de App Service. Envíe correos electrónicos, tweets y publicaciones de Facebook, incorpórese a listas de distribución de correo, etc.
ms.topic: tutorial
ms.date: 04/08/2020
ms.custom: devx-track-csharp, mvc
ms.openlocfilehash: ddd57bfe653766d133824f42098d9ad9adb1e56b
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962203"
---
# <a name="tutorial-send-email-and-invoke-other-business-processes-from-app-service"></a>Tutorial: Envío de correo electrónico e invocación de otros procesos empresariales desde App Service

En este tutorial, aprenderá a integrar una aplicación de App Service con los procesos empresariales. Esto es frecuente en algunos escenarios de aplicaciones web, como los siguientes:

- Envío de correos electrónicos de confirmación para una transacción
- Incorporación de un usuario a un grupo de Facebook
- Conexión a sistemas de terceros, como SAP, Salesforce, etc.
- Intercambio de mensajes B2B estándar

En este tutorial, enviará correos electrónicos con Gmail desde una aplicación de App Service mediante [Azure Logic Apps](../logic-apps/logic-apps-overview.md). Existen otros mecanismos para enviar correos electrónicos desde una aplicación web, como la configuración de SMTP proporcionada por el marco del lenguaje. Sin embargo, Logic Apps ofrece muchas más posibilidades a la aplicación de App Service sin aumentar la complejidad del código. Logic Apps cuenta con una sencilla interfaz de configuración para las integraciones empresariales más comunes, que la aplicación puede llamar en cualquier momento mediante una solicitud HTTP.

## <a name="prerequisite"></a>Requisito previo

Implemente una aplicación con el marco de lenguaje que prefiera para App Service. Para ver un tutorial acerca de cómo implementar una aplicación de ejemplo, consulte lo siguiente:

### <a name="aspnet"></a>[ASP.NET](#tab/dotnet)

[Tutorial: Compilación de una aplicación ASP.NET en Azure con SQL Database](app-service-web-tutorial-dotnet-sqldatabase.md)

### <a name="aspnet-core"></a>[ASP.NET Core](#tab/dotnetcore)

[Tutorial: Compilación de una aplicación ASP.NET Core y SQL Database en Azure App Service](tutorial-dotnetcore-sqldb-app.md)

### <a name="nodejs"></a>[Node.js](#tab/node)

[Tutorial: Compilación de una aplicación Node.js y MongoDB en Azure](tutorial-nodejs-mongodb-app.md)

### <a name="php"></a>[PHP](#tab/php)

[Tutorial: Creación de una aplicación PHP y MySQL en Azure](tutorial-php-mysql-app.md)

### <a name="python"></a>[Python](#tab/python)

[Tutorial: Ejecución de una aplicación web Python (Django) con PostgreSQL en Azure App Service](tutorial-python-postgresql-app.md)

### <a name="ruby"></a>[Ruby](#tab/ruby)

[Compilación de una aplicación Ruby y Postgres en Azure App Service en Linux](tutorial-ruby-postgres-app.md)

---

## <a name="create-the-logic-app"></a>Creación de la aplicación lógica

1. En [Azure Portal](https://portal.azure.com), cree una aplicación lógica vacía siguiendo las instrucciones de [Creación de una aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md#create-your-logic-app). Cuando aparezca el **diseñador de Logic Apps**, vuelva a este tutorial.
1. En la página de inicio del diseñador de Logic Apps, seleccione **Cuando se recibe una solicitud HTTP** en **Empezar con un desencadenador común**.

    ![Captura de pantalla que muestra la página inicial del diseñador de Logic Apps con la opción Cuando se recibe una solicitud HTTP resaltada.](./media/tutorial-send-email/receive-http-request.png)
1. En el cuadro de diálogo **Cuando se recibe una solicitud HTTP**, seleccione **Usar una carga de ejemplo para generar el esquema**.

    ![Captura de pantalla que muestra el cuadro de diálogo de la opción Cuando se recibe una solicitud HTTP y la opción Usar una carga de ejemplo para generar el esquema seleccionada. ](./media/tutorial-send-email/generate-schema-with-payload.png)

1. Copie el siguiente código JSON de ejemplo en el cuadro de texto y seleccione **Listo**.

    ```json
    {
        "task": "<description>",
        "due": "<date>",
        "email": "<email-address>"
    }
    ```

    Se generará el esquema con los datos de solicitud que desee. En la práctica, solo puede capturar los datos reales de la solicitud que genera el código de la aplicación, por lo que tendrá que dejar que Azure genere el esquema JSON. 
1. En la parte superior del diseñador de Logic Apps, seleccione **Guardar**. 

    Aparecerá la dirección URL del desencadenador de solicitudes HTTP. Cópiela utilizando el icono correspondiente para utilizarla más tarde.

    ![Captura de pantalla que resalta el icono de copia para copiar la URL del desencadenador de solicitud HTTP.](./media/tutorial-send-email/http-request-url.png)

    Esta definición de la solicitud HTTP le permitirá desencadenar lo que quiera en esta aplicación lógica, ya sea Gmail u otra aplicación. Más adelante, invocará esta dirección URL en la aplicación de App Service. Para más información sobre el desencadenador de la solicitud, consulte la [referencia sobre las solicitudes HTTP y su respuesta](../connectors/connectors-native-reqres.md).

1. En la parte inferior del diseñador, haga clic en **Nuevo paso** y, en el cuadro de búsqueda de acciones, escriba **Gmail**. Después, busque y seleccione **Enviar correo electrónico (V2)** .
    
    > [!TIP]
    > Puede buscar otros tipos de integraciones, como SendGrid, MailChimp, Office 365 y SalesForce. Para más información, consulte [Documentación de Azure Logic Apps](../logic-apps/index.yml).
1. En el cuadro de diálogo **Gmail**, seleccione **Iniciar sesión** e inicie sesión en la cuenta de Gmail desde la que desea enviar el correo electrónico.

    ![Captura de pantalla que muestra el cuadro de diálogo de Gmail que se usa para iniciar sesión en la cuenta de Gmail desde la que quiere enviar el correo electrónico.](./media/tutorial-send-email/gmail-sign-in.png)

1. Una vez que haya iniciado sesión, haga clic en el cuadro de texto **Para** y se abrirá automáticamente el cuadro de diálogo Contenido dinámico.

1. Junto a la acción **Cuando se recibe una solicitud HTTP**, seleccione **Ver más**.

    ![Captura de pantalla que muestra el botón Ver más junto a la acción Cuando se recibe una solicitud HTTP.](./media/tutorial-send-email/expand-dynamic-content.png)

    Ahora, debería ver las tres propiedades de los datos JSON de ejemplo que usó anteriormente. En este paso, utilizará estas propiedades de la solicitud HTTP para crear un correo electrónico.
1. Como está seleccionando el valor del campo **Para**, elija **correo electrónico**. Si lo desea, puede desactivar el cuadro de diálogo Contenido dinámico haciendo clic en **Agregar contenido dinámico**.

    ![Captura de pantalla que muestra la opción de correo electrónico y la opción Agregar contenido dinámico resaltada.](./media/tutorial-send-email/hide-dynamic-content.png)

1. En el menú desplegable **Agregar nuevo parámetro**, seleccione **Asunto** y **Cuerpo**.

1. Haga clic en el cuadro de texto **Asunto** y seleccione **tarea**. Con el cursor todavía en el cuadro **Asunto**, escriba *creado*. 

1. Haga clic en **Cuerpo** y elija **vencido**. Mueva el cursor a la izquierda de **vencido** y escriba *Este elemento de trabajo vence el*.

    > [!TIP]
    > Si desea editar el contenido HTML directamente en el cuerpo del correo electrónico, seleccione **Vista Código** en la parte superior de la ventana del diseñador de Logic Apps. No olvide conservar el código de Contenido dinámico (por ejemplo, `@{triggerBody()?['due']}`).
    >
    > ![Captura de pantalla que muestra la vista de código para visualizar el contenido HTML directamente en el cuerpo del correo electrónico.](./media/tutorial-send-email/edit-rich-html-email.png) 

1. A continuación, agregue una respuesta HTTP asincrónica al desencadenador HTTP. Entre el desencadenador HTTP y la acción de Gmail, haga clic en el signo **+** y seleccione **Agregar una rama paralela**.

    ![Captura de pantalla que muestra el signo + y la opción Agregar una rama paralela resaltada.](./media/tutorial-send-email/add-http-response.png)

1. En el cuadro de búsqueda, busque **respuesta** y seleccione la acción **Respuesta**.

    ![Captura de pantalla que muestra la barra de búsqueda y la acción Respuesta resaltada.](./media/tutorial-send-email/choose-response-action.png)

    De forma predeterminada, la acción Respuesta envía un estado HTTP 200. Ese estado es aceptable para este tutorial. Para más información, consulte esta [referencia sobre las solicitudes HTTP y su respuesta](../connectors/connectors-native-reqres.md).

1. En la parte superior del diseñador de Logic Apps, seleccione de nuevo **Guardar**. 

## <a name="add-http-request-code-to-app"></a>Incorporación del código de la solicitud HTTP en la aplicación

Asegúrese de que anteriormente copió la dirección URL del desencadenador de la solicitud HTTP. Como contiene información confidencial, no es conveniente que la incluya directamente en el código. Con App Service, puede hacer referencia a ella como una variable de entorno mediante la configuración de la aplicación. 

En [Cloud Shell](https://shell.azure.com), cree la configuración de la aplicación con el comando siguiente (sustituya *\<app-name>* , *\<resource-group-name>* y *\<logic-app-url>* ):

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings LOGIC_APP_URL="<your-logic-app-url>"
```

En el código, envíe una solicitud HTTP a la dirección URL utilizando cualquier lenguaje cliente de HTTP que esté disponible en el marco de trabajo con la siguiente configuración:

- El cuerpo de la solicitud debe tener el mismo formato JSON que utilizó con la aplicación lógica:

    ```json
    {
        "task": "<description>",
        "due": "<date>",
        "email": "<email-address>"
    }
    ```

- La solicitud debe contener el encabezado `Content-Type: application/json`. 
- Para optimizar el rendimiento, envíe la solicitud de forma asincrónica, si es posible.

Haga clic en la pestaña del lenguaje o marco que prefiera para ver un ejemplo.

### <a name="aspnet"></a>[ASP.NET](#tab/dotnet)

En ASP.NET, puede enviar la solicitud HTTP POST con la clase [System.Net.Http.HttpClient](/dotnet/api/system.net.http.httpclient). Por ejemplo:

```csharp
// requires using System.Net.Http;
var client = new HttpClient();
// requires using System.Text.Json;
var jsonData = JsonSerializer.Serialize(new
{
    email = "a-valid@emailaddress.com",
    due = "4/1/2020",
    task = "My new task!"
});

HttpResponseMessage result = await client.PostAsync(
    // requires using System.Configuration;
    ConfigurationManager.AppSettings["LOGIC_APP_URL"],
    new StringContent(jsonData, Encoding.UTF8, "application/json"));
    
var statusCode = result.StatusCode.ToString();
```

Si está probando este código en la aplicación de ejemplo de [Tutorial: Compilación de una aplicación ASP.NET en Azure con SQL Database](app-service-web-tutorial-dotnet-sqldatabase.md), podrá utilizarla para enviar una confirmación por correo electrónico en la [acción Create](https://github.com/Azure-Samples/dotnet-sqldb-tutorial/blob/master/DotNetAppSqlDb/Controllers/TodosController.cs#L52-L63) después de agregar el elemento `Todo`. Para usar el código asincrónico anterior, convierta la acción Create en asincrónica.

### <a name="aspnet-core"></a>[ASP.NET Core](#tab/dotnetcore)

En ASP.NET Core, puede enviar la solicitud HTTP POST con la clase [System.Net.Http.HttpClient](/dotnet/api/system.net.http.httpclient). Por ejemplo:

```csharp
// requires using System.Net.Http;
var client = new HttpClient();
// requires using System.Text.Json;
var jsonData = JsonSerializer.Serialize(new
{
    email = "a-valid@emailaddress.com",
    due = "4/1/2020",
    task = "My new task!"
});

HttpResponseMessage result = await client.PostAsync(
    // Requires DI configuration to access app settings. See https://docs.microsoft.com/azure/app-service/configure-language-dotnetcore#access-environment-variables
    _configuration["LOGIC_APP_URL"],
    new StringContent(jsonData, Encoding.UTF8, "application/json"));
    
var statusCode = result.StatusCode.ToString();
```

> [!NOTE]
> Este código se ha escrito así para simplificar la demostración. En la práctica, no cree instancias de un objeto `HttpClient` en cada solicitud. Siga las instrucciones que se indican en [Uso de IHttpClientFactory para implementar solicitudes HTTP resistentes](/dotnet/architecture/microservices/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests).

Si está probando este código en la aplicación de ejemplo de [Tutorial: Compilación de una aplicación ASP.NET Core y SQL Database en Azure App Service](tutorial-dotnetcore-sqldb-app.md), podrá utilizarla para enviar una confirmación por correo electrónico en la [acción Create](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial/blob/master/Controllers/TodosController.cs#L56-L65) después de agregar el elemento `Todo`.

### <a name="nodejs"></a>[Node.js](#tab/node)

En Node.js, puede enviar fácilmente la solicitud HTTP POST con un paquete npm como [axios](https://www.npmjs.com/package/axios). Por ejemplo:

```javascript
// Requires npm install --save axios
const axios = require('axios');

var jsonData = {
        email: "a-valid@emailaddress.com",
        due: "4/1/2020",
        task: "My new task!"
};

(async function(data) {
    try {
        const response = await axios.post(process.env.LOGIC_APP_URL, jsonData);
        console.log(response.status);
    } catch (error) {
        console.log(error);
    }
})(jsonData);

```

Si está probando este código en la aplicación de ejemplo de [Tutorial: Cree una aplicación Node.js y MongoDB en Azure](tutorial-nodejs-mongodb-app.md), puede usar esta aplicación para enviar un correo electrónico de confirmación en la [función Create](https://github.com/Azure-Samples/meanjs/blob/master/modules/articles/server/controllers/articles.server.controller.js#L14-L27), después de [the article is saved successfully](https://github.com/Azure-Samples/meanjs/blob/master/modules/articles/server/controllers/articles.server.controller.js#L24) (el artículo se guardó correctamente).

### <a name="php"></a>[PHP](#tab/php)

En PHP, puede enviar fácilmente la solicitud HTTP POST con [Guzzle](http://docs.guzzlephp.org/en/stable/index.html). Por ejemplo:

```php
// Requires composer require guzzlehttp/guzzle:~6.0
use GuzzleHttp\Client;
...
$client = new Client();
$options = [
    'json' => [ 
        'email' => 'a-valid@emailaddress.com',
        'due' => '4/1/2020',
        'task' => "My new task!"
    ]
];

$promise = $client-> postAsync(getenv($LOGIC_APP_URL), $options)->then( 
    function ($response) {
        return $response->getStatusCode();
    }, function ($exception) {
        return $exception->getResponse();
    }
);

$response = $promise->wait();
// Requires Laravel to run Log::info(). Check the documentation of your preferred framework for logging instructions.
Log::info(print_r($response, TRUE));
```

Si está probando este código en la aplicación de ejemplo de [Tutorial: Cree una aplicación PHP y MySQL en Azure](tutorial-php-mysql-app.md), puede utilizar esta aplicación para enviar un correo electrónico de confirmación en la [función Route::post](https://github.com/Azure-Samples/laravel-tasks/blob/master/routes/web.php#L30-L48), justo antes de la instrucción "return".

### <a name="python"></a>[Python](#tab/python)

En Python, puede enviar fácilmente una solicitud HTTP POST con [requests](https://pypi.org/project/requests/). Por ejemplo:

```python
# Requires pip install requests && pip freeze > requirements.txt
import requests
...
payload = {
    "email": "a-valid@emailaddress.com",
    "due": "4/1/2020",
    "task": "My new task!"
}
response = requests.post("https://prod-112.westeurope.logic.azure.com:443/workfl$
print(response.status_code)
```
<!-- ```python
# Requires pip install aiohttp && pip freeze > requirements.txt
import aiohttp
...
payload = {
        'email': 'a-valid@emailaddress.com',
        'due': '4/1/2020',
        'task': 'My new task!'
}
async with aiohttp.post('http://httpbin.org/post', data=json.dump(payload)) as resp:
    print(await resp.status())
``` -->

Si está probando este código en la aplicación de ejemplo de [Tutorial: Ejecute una aplicación web de Python (Django) con PostgreSQL en Azure App Service](tutorial-python-postgresql-app.md), puede utilizar esta aplicación para enviar un correo electrónico de confirmación en la [función Route::post](https://github.com/Azure-Samples/laravel-tasks/blob/master/routes/web.php#L30-L48) justo antes de la instrucción "return".

### <a name="ruby"></a>[Ruby](#tab/ruby)

En Ruby, puede enviar fácilmente una solicitud HTTP POST con [JSONClient](https://www.rubydoc.info/gems/httpclient/JSONClient). Por ejemplo:

```ruby
clnt = JSONClient.new
body = { 
    'email' => 'a-valid@emailaddress.com',
    'due' => '4/1/2020',
    'task' => "My new task!"
}

connection = clnt.post_async(ENV['LOGIC_APP_URL'], body)
```

Si está probando este código en la aplicación de ejemplo de [Compilación de una aplicación Ruby y Postgres en Azure App Service en Linux](tutorial-ruby-postgres-app.md), puede usarla para enviar una confirmación por correo electrónico en la acción [create](https://github.com/Azure-Samples/rubyrails-tasks/blob/master/app/controllers/tasks_controller.rb#L26-L38) [cuando @task.save se ejecute correctamente](https://github.com/Azure-Samples/rubyrails-tasks/blob/master/app/controllers/tasks_controller.rb#L30).

---

## <a name="more-resources"></a>Más recursos

[Tutorial: Hospedaje de una API RESTful con CORS en Azure App Service](app-service-web-tutorial-rest-api.md)  
[Referencia de solicitudes HTTP y sus respuestas en Logic Apps](../connectors/connectors-native-reqres.md)  
[Inicio rápido: Creación del primer flujo de trabajo mediante Azure Logic Apps: Azure Portal](../logic-apps/quickstart-create-first-logic-app-workflow.md)