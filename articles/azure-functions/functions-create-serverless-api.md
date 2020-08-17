---
title: Personalización de un punto de conexión HTTP en Azure Functions
description: Obtenga información sobre cómo personalizar un punto de conexión de desencadenador HTTP en Azure Functions.
author: mattchenderson
ms.topic: conceptual
ms.date: 04/27/2020
ms.author: mahender
ms.custom: mvc
ms.openlocfilehash: 440eb1f39284f8d99a8d6b9067b018c4a54fcd27
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87083028"
---
# <a name="customize-an-http-endpoint-in-azure-functions"></a>Personalización de un punto de conexión HTTP en Azure Functions

En este artículo, aprenderá cómo Azure Functions permite crear API de gran escalabilidad. Azure Functions incluye una colección de enlaces y desencadenadores HTTP integrados que facilitan la creación de un punto de conexión en diversos lenguajes, como Node.js, C# y muchos más. En este artículo, va a personalizar un desencadenador HTTP para administrar acciones específicas en el diseño de la API. También va a prepararse para ampliar su API integrándola con Azure Functions Proxies y configurando API simuladas. Estas tareas se logran en un entorno de proceso sin servidor de Functions, por lo que no tiene que preocuparse de escalar los recursos; puede centrarse simplemente en la lógica de la API.

## <a name="prerequisites"></a>Prerrequisitos 

[!INCLUDE [Previous quickstart note](../../includes/functions-quickstart-previous-topics.md)]

La función resultante se usará durante el resto de este artículo.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com) con su cuenta de Azure.

## <a name="customize-your-http-function"></a>Personalización de la función HTTP

De forma predeterminada, la función de desencadenador HTTP está configurada para aceptar cualquier método HTTP. También se puede utilizar la dirección URL predeterminada, `http://<yourapp>.azurewebsites.net/api/<funcname>?code=<functionkey>`. En esta sección, modificará la función para que solo responda a solicitudes GET con `/api/hello`. 

1. Vaya a su función en Azure Portal. Seleccione **Integración** en el menú de la izquierda y, luego, seleccione **HTTP (req)** en **Desencadenador**.

    :::image type="content" source="./media/functions-create-serverless-api/customizing-http.png" alt-text="Personalización de una función HTTP":::

1. Use la configuración de desencadenador HTTP como se especifica en la tabla siguiente.

    | Campo | Valor de ejemplo | Descripción |
    |---|---|---|
    | Plantilla de ruta | /hello | Determina qué ruta se utiliza para invocar esta función. |
    | Nivel de autorización | Anónimas | Opcional: Hace que la función sea accesible sin una clave de API |
    | Métodos HTTP seleccionados | GET | Permite que solo se usen los métodos HTTP seleccionados para invocar esta función. |

    No incluyó el prefijo de ruta de acceso base `/api` en la plantilla de ruta, ya que esto se controla mediante una configuración global.

1. Seleccione **Guardar**.

Para obtener más información sobre cómo personalizar funciones HTTP, consulte [Enlaces HTTP de Azure Functions](./functions-bindings-http-webhook.md).

### <a name="test-your-api"></a>Prueba de la API

A continuación, pruebe la función para verla en acción con la nueva superficie de la API:
1. En la página de función, seleccione **Código y prueba** en el menú de la izquierda.

1. Seleccione **Obtener la dirección URL de la función** en el menú superior y copie la dirección URL. Confirme que ahora usa la ruta de acceso `/api/hello`.
 
1. Copie la dirección URL en una nueva pestaña del explorador o el cliente de REST que prefiera. 

   Los exploradores usan GET de forma predeterminada.
 
1. Agregue parámetros a la cadena de consulta en la dirección URL. 

   Por ejemplo, `/api/hello/?name=John`.
 
1. Presione Entrar para confirmar que funciona. Debería ver la respuesta "*Hello John*".

1. También puede intentar llamar al punto de conexión con otro método HTTP para confirmar que no se ejecuta la función. Para ello, use un cliente de REST, como cURL, Postman o Fiddler.

## <a name="proxies-overview"></a>Introducción a Servidores proxy

En la siguiente sección, expondrá la API a través de un servidor proxy. Azure Functions Proxies le permite reenviar solicitudes a otros recursos. Un punto de conexión HTTP se define igual que con el desencadenador HTTP. Sin embargo, en lugar de escribir código que se ejecute cuando se llame a ese punto de conexión, se proporciona una dirección URL a una implementación remota. Esto le permite crear varios orígenes de API dentro de una sola superficie de API que es fácil de consumir para los clientes, lo que resulta útil si desea compilar la API como microservicios.

Un servidor proxy puede apuntar a cualquier recurso HTTP, como:
- Azure Functions 
- Aplicaciones de API en [Azure App Service](../app-service/overview.md)
- Contenedores de Docker en [App Service en Linux](../app-service/containers/app-service-linux-intro.md)
- Cualquier otra API hospedada

Para obtener más información sobre Proxies, consulte [Uso de Azure Functions Proxies].

## <a name="create-your-first-proxy"></a>Creación de su primer proxy

En esta sección, creará un proxy que actúa como front-end de la API en general. 

### <a name="setting-up-the-frontend-environment"></a>Configuración del entorno de front-end

Repita los pasos de [Creación de una aplicación de función](./functions-create-first-azure-function.md#create-a-function-app) para crear una aplicación de función en la que creará el proxy. Esta nueva dirección URL de la aplicación sirve como front-end para la API, mientras que la aplicación de función que editó antes actúa como back-end.

1. Vaya a la nueva aplicación de función de front-end en el portal.
1. Seleccione **Características de la plataforma** y elija **Configuración de la aplicación**.
1. Desplácese hacia abajo hasta **Configuración de la aplicación**, donde se almacenan los pares clave-valor y cree una nueva configuración con la clave `HELLO_HOST`. Establezca su valor en el host de la aplicación de función de back-end, como `<YourBackendApp>.azurewebsites.net`. Este valor forma parte de la dirección URL que copió antes al probar la función HTTP. Hará referencia a este valor en la configuración más adelante.

    > [!NOTE] 
    > Se recomienda la configuración de la aplicación para que la configuración del host evite una dependencia de entorno codificada de forma rígida para el servidor proxy. Al usarse la configuración de la aplicación, puede mover la configuración del proxy entre entornos, y se aplicará la configuración de la aplicación específica del entorno.

1. Seleccione **Guardar**.

### <a name="creating-a-proxy-on-the-frontend"></a>Creación de un proxy en el front-end

1. Vuelva a la aplicación de función de front-end en el portal.

1. En el menú izquierdo, seleccione **Proxies** y, luego, **Agregar**. 

1. En la página **Nuevo proxy**, use la configuración de la tabla siguiente y, luego, seleccione **Crear**.

    | Campo | Valor de ejemplo | Descripción |
    |---|---|---|
    | Nombre | HelloProxy | Un nombre descriptivo que se usa solo para la administración |
    | Plantilla de ruta | /api/remotehello | Determina qué ruta se utiliza para invocar este proxy. |
    | Dirección URL de back-end | https://%HELLO_HOST%/api/hello | Especifica el punto de conexión al que la solicitud debe remitir a través del proxy. |

    
    :::image type="content" source="./media/functions-create-serverless-api/creating-proxy.png" alt-text="Creación de un proxy":::

    En Azure Functions Proxies no se proporciona el prefijo de ruta de acceso base `/api`, que debe incluirse en la plantilla de ruta. La sintaxis `%HELLO_HOST%` hace referencia a la configuración de la aplicación que creó antes. La dirección URL resuelta apuntará a la función original.

1. Pruebe el nuevo proxy; para ello, copie la dirección URL del proxy y pruébela en el explorador o con su cliente HTTP favorito:
    - Para una función anónima, use:   `https://YOURPROXYAPP.azurewebsites.net/api/remotehello?name="Proxies"`.
    - Para una función con autorización, use:   `https://YOURPROXYAPP.azurewebsites.net/api/remotehello?code=YOURCODE&name="Proxies"`.

## <a name="create-a-mock-api"></a>Creación de una API simulada

A continuación, utilizará un proxy para crear una API simulada para su solución. Este proxy permite que avance el desarrollo del cliente, sin necesidad de que el back-end esté totalmente implementado. Más adelante en el desarrollo, puede crear una aplicación de función que admita esta lógica y redirigir al proxy a ella.

Para crear esta API simulada, se va a crear un proxy, esta vez mediante el [Editor de App Service](https://github.com/projectkudu/kudu/wiki/App-Service-Editor). Para empezar, vaya a la aplicación de función en el portal. Seleccione **Características de la plataforma** y, en **Herramientas de desarrollo**, busque **Editor de App Service**. El Editor de App Service se abre en una nueva pestaña.

Seleccione `proxies.json` en el panel de navegación izquierdo. Este archivo almacena la configuración para todos los servidores proxy. Si utiliza uno de los [métodos de implementación de Functions](./functions-continuous-deployment.md), mantendrá este archivo en el control de código fuente. Para aprender más sobre este archivo, consulte [Configuración avanzada de servidores proxy](./functions-proxies.md#advanced-configuration).

Si ha seguido los pasos hasta ahora, su archivo proxies.json debería tener el siguiente aspecto:

```json
{
    "$schema": "http://json.schemastore.org/proxies",
    "proxies": {
        "HelloProxy": {
            "matchCondition": {
                "route": "/api/remotehello"
            },
            "backendUri": "https://%HELLO_HOST%/api/hello"
        }
    }
}
```

A continuación, va a agregar su API simulada. Reemplace su archivo proxies.json por el siguiente código:

```json
{
    "$schema": "http://json.schemastore.org/proxies",
    "proxies": {
        "HelloProxy": {
            "matchCondition": {
                "route": "/api/remotehello"
            },
            "backendUri": "https://%HELLO_HOST%/api/hello"
        },
        "GetUserByName" : {
            "matchCondition": {
                "methods": [ "GET" ],
                "route": "/api/users/{username}"
            },
            "responseOverrides": {
                "response.statusCode": "200",
                "response.headers.Content-Type" : "application/json",
                "response.body": {
                    "name": "{username}",
                    "description": "Awesome developer and master of serverless APIs",
                    "skills": [
                        "Serverless",
                        "APIs",
                        "Azure",
                        "Cloud"
                    ]
                }
            }
        }
    }
}
```

Este código agrega un nuevo proxy, `GetUserByName`, sin la propiedad `backendUri`. En lugar de llamar a otro recurso, modifica la respuesta predeterminada desde Servidores proxy mediante una invalidación de la respuesta. Las invalidaciones de solicitud y respuesta también pueden utilizarse junto con una dirección URL de back-end. Esta técnica resulta especialmente útil cuando se remite a través de un proxy a un sistema heredado, donde podría necesitar modificar los encabezados, los parámetros de consulta, etc. Para aprender más sobre las invalidaciones de solicitud y respuesta, consulte [Modificación de solicitudes y respuestas en Servidores proxy](./functions-proxies.md).

Pruebe la API simulada mediante una llamada al punto de conexión `<YourProxyApp>.azurewebsites.net/api/users/{username}` con un explorador o el cliente de REST que prefiera. Asegúrese de reemplazar _{username}_ por un valor de cadena que represente un nombre de usuario.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a compilar y personalizar una API en Azure Functions. También ha aprendido a reunir varias API, incluidas las simuladas, como una superficie de API unificada. Puede utilizar estas técnicas para crear API de cualquier complejidad, todo ello mientras ejecuta el modelo de proceso sin servidor proporcionado por Azure Functions.

Las siguientes referencias pueden serle de ayuda a medida que sigue desarrollando su API:

- [Enlaces HTTP de Azure Functions](./functions-bindings-http-webhook.md)
- [Uso de Azure Functions Proxies]
- [Documentación de una API de Azure Functions (versión preliminar)](./functions-openapi-definition.md)


[Create your first function]: ./functions-create-first-azure-function.md
[Uso de Azure Functions Proxies]: ./functions-proxies.md
