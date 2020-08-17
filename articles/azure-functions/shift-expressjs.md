---
title: Cambio de Express.js a Azure Functions
description: Aprenda a refactorizar los puntos de conexión de Express.js para Azure Functions.
author: craigshoemaker
ms.topic: conceptual
ms.date: 07/31/2020
ms.author: cshoe
ms.openlocfilehash: 266df5371ff5f47526fa9d6567c62e31d51ebb05
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87810231"
---
# <a name="shifting-from-expressjs-to-azure-functions"></a>Cambio de Express.js a Azure Functions

Express.js es uno de los marcos de Node.js más populares para desarrolladores web y sigue siendo una opción excelente para compilar aplicaciones que sirvan puntos de conexión de API.

Al migrar código a una arquitectura sin servidor, la refactorización de los puntos de conexión de Express.js afecta a las siguientes áreas:

- **Middleware**: Express.js presenta una sólida colección de middleware. Muchos módulos de middleware ya no son necesarios considerando las funcionalidades de [Azure API Management](../api-management/api-management-key-concepts.md) y Azure Functions. Asegúrese de que puede replicar o reemplazar cualquier lógica controlada por middleware esencial antes de migrar los puntos de conexión.

- **API diferentes**: La API usada para procesar las solicitudes y respuestas difiere entre Azure Functions y Express.js. En el ejemplo siguiente se detallan los cambios necesarios.

- **Ruta predeterminada**: De manera predeterminada, los puntos de conexión de Azure Functions se exponen en la ruta de `api`. Las reglas de enrutamiento se pueden configurar a través de [`routePrefix` en el archivo _host.json_](./functions-bindings-http-webhook-output.md#hostjson-settings).

- **Configuración y convenciones**: Una aplicación de Functions usa el archivo _function.json_ para definir verbos HTTP y directivas de seguridad, y puede configurar la [entrada y salida](./functions-triggers-bindings.md) de la función. De manera predeterminada, el nombre de la carpeta que contiene los archivos de función define el nombre del punto de conexión, pero puede cambiar dicho nombre mediante la propiedad `route` en el archivo [function.json](./functions-bindings-http-webhook-trigger.md#customize-the-http-endpoint).

> [!TIP]
> Obtenga más información a través del tutorial interactivo [Refactorización de las API de Node.js y Express a API sin servidor con Azure Functions](/learn/modules/shift-nodejs-express-apis-serverless/).

## <a name="example"></a>Ejemplo

### <a name="expressjs"></a>Express.js

En el ejemplo siguiente se muestra un punto de conexión `GET` de Express.js típico.

```javascript
// server.js
app.get('/hello', (req, res) => {
  try {
    res.send("Success!");
  } catch(error) {
    const err = JSON.stringify(error);
    res.status(500).send(`Request error. ${err}`);
  }
});
```

Cuando se envía una solicitud `GET` a `/hello`, se devuelve una respuesta `HTTP 200` que contiene `Success`. Si el punto de conexión encuentra un error, la respuesta es `HTTP 500` con los detalles del error.

### <a name="azure-functions"></a>Azure Functions

Azure Functions organiza la configuración y los archivos de código en una única carpeta para cada función. De manera predeterminada, el nombre de la carpeta dicta el nombre de la función.

Por ejemplo, una función denominada `hello` tiene una carpeta con los archivos siguientes.

``` files
| - hello
|  - function.json
|  - index.js
```

En el ejemplo siguiente se implementa el mismo resultado que en el punto de conexión de Express.js anterior, pero con Azure Functions.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
// hello/index.js
module.exports = async function (context, req) {
  try {
    context.res = { body: "Success!" };
  } catch(error) {
    const err = JSON.stringify(error);
    context.res = {
      status: 500,
      body: `Request error. ${err}`
    };
  }
};
```

# <a name="typescript"></a>[TypeScript](#tab/typescript)

```typescript
// hello/index.ts
import { AzureFunction, Context, HttpRequest } from "@azure/functions";

const httpTrigger: AzureFunction = async function (context: Context, req: HttpRequest): Promise<void> {
  try {
    context.res = { body: "Success!" };
  } catch (error) {
    const err = JSON.stringify(error);
    context.res = {
      status: 500,
      body: `Request error. ${err}`,
    };
  }
};

export default httpTrigger;
```

---

Al pasar a Functions, se realizan los siguientes cambios:

- **Módulo:** El código de la función se implementa como un módulo de JavaScript.

- **Objeto de respuesta y contexto**: [`context`](./functions-reference-node.md#context-object) le permite comunicarse con el entorno en tiempo de ejecución de la función. En el contexto, puede leer los datos de la solicitud y establecer la respuesta de la función. El código sincrónico requiere la llamada a `context.done()` para completar la ejecución, mientras que las funciones `asyc` resuelven la solicitud implícitamente.

- **Convención de nomenclatura**: El nombre de la carpeta que se usa para contener los archivos de Azure Functions se utiliza como nombre de punto de conexión de manera predeterminada (se puede invalidar en [function.json](./functions-bindings-http-webhook-trigger.md#customize-the-http-endpoint)).

- **Configuración**: Los verbos HTTP se definen en el archivo [function.jsonN](./functions-bindings-http-webhook-trigger.md#customize-the-http-endpoint) como `POST` o `PUT`.

El siguiente archivo _function.archivo_ contiene información de configuración para la función.

```json
{
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": ["get"]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    }
  ]
}
```

Al definir `get` en la matriz de `methods`, la función está disponible para las solicitudes `GET` HTTP. Si quiere que la API acepte las solicitudes `POST` de soporte, también puede agregar `post` a la matriz.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información con el tutorial interactivo [Refactorización de las API de Node.js y Express a API sin servidor con Azure Functions](/learn/modules/shift-nodejs-express-apis-serverless/).