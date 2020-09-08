---
title: Incorporación de una API en Azure Static Web Apps con Azure Functions
description: Para empezar a usar Azure Static Web Apps, agregue una API sin servidor a la aplicación web estática mediante Azure Functions.
services: static-web-apps
author: manekinekko
ms.service: static-web-apps
ms.topic: how-to
ms.date: 05/29/2020
ms.author: wachegha
ms.custom: devx-track-javascript
ms.openlocfilehash: 4dd253aa5dc1b8b89b1b387a555dde416c601f9c
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88691503"
---
# <a name="add-an-api-to-azure-static-web-apps-preview-with-azure-functions"></a>Incorporación de una API en la versión preliminar de Azure Static Web Apps con Azure Functions

Puede agregar API sin servidor a Azure Static Web Apps a través de la integración con Azure Functions. En este artículo se muestra cómo agregar e implementar una API en un sitio de Azure Static Web Apps.

## <a name="prerequisites"></a>Requisitos previos

- Cuenta de Azure con una suscripción activa.
  - Si no tiene una cuenta, puede [crear una gratis](https://azure.microsoft.com/free).
- [Visual Studio Code](https://code.visualstudio.com/)
- [Extensión de Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) para Visual Studio Code
- Extensión [Live Server de Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer).
- [Node.js](https://nodejs.org/download/) para ejecutar la aplicación de API localmente

## <a name="create-a-git-repository"></a>Crear un repositorio de Git

En los pasos siguientes se muestra cómo crear un nuevo repositorio y clonar los archivos en el equipo.

1. Asegúrese de que ha iniciado sesión en GitHub y, a continuación, vaya a https://github.com/staticwebdev/vanilla-basic/generate para crear un nuevo repositorio.
1. En el cuadro _Nombre del repositorio_, escriba **my-vanilla-api**.
1. Haga clic en **Create repository from template** (Crear repositorio a partir de plantilla).

   :::image type="content" source="media/add-api/create-repository.png" alt-text="Creación de un nuevo repositorio a partir de vanilla-basic":::

Una vez creado el proyecto, copie la dirección URL en el explorador del nuevo repositorio. Esta dirección URL se usa en Visual Studio Code para clonar el repositorio de Git.

1. Presione **F1** para abrir un comando en la paleta de comandos.
1. Pegue la dirección URL en el cuadro _Git: Clone_ (Git: Clonar) y presione **Entrar**.

   :::image type="content" source="media/add-api/vscode-git-0.png" alt-text="Clonación de un proyecto de GitHub con Visual Studio Code":::

    Siga las indicaciones para seleccionar una ubicación del repositorio para clonar el proyecto.

## <a name="create-the-api"></a>Creación de la API

A continuación, cree un proyecto de Azure Functions como la API de la aplicación. 

1. Dentro del proyecto _my-vanilla-api_, cree una subcarpeta denominada **api**.
1. Presione **F1** para abrir la paleta de comandos.
1. Escriba **Azure Functions: Crear nuevo proyecto...**
1. Presione **Entrar**.
1. Seleccione **Examinar**.
1. Seleccione la carpeta **api** como directorio del área de trabajo del proyecto.
1. Elija **Seleccionar**.

   :::image type="content" source="media/add-api/create-azure-functions-vscode-1.png" alt-text="Creación de una nueva instancia de Azure Functions mediante Visual Studio Code":::

1. Escriba la siguiente información cuando se le indique:

    - _Seleccione un lenguaje_: Elija **JavaScript**.
    - _Seleccione una plantilla para la primera función del proyecto_: Elija **desencadenador HTTP**.
    - _Especifique un nombre de función_: escriba **GetMessage**.
    - _Nivel de autorización_: Elija **Anonymous** (anónimo), que permite que cualquiera llame al punto de conexión de la función.
        - Para obtener información sobre los niveles de autorización, consulte [Claves de autorización](../azure-functions/functions-bindings-http-webhook-trigger.md#authorization-keys).

Visual Studio Code genera un proyecto de Azure Functions con una función de desencadenador de HTTP.

Ahora la aplicación tiene una estructura de proyecto similar a la del ejemplo siguiente.

```files
├── api
│   ├── GetMessage
│   │   ├── function.json
│   │   ├── index.js
│   │   └── sample.dat
│   ├── host.json
│   ├── local.settings.json
│   ├── package.json
│   └── proxies.json
├── index.html
├── readme.md
└── styles.css
```

A continuación, cambiará la función `GetMessage` para devolver un mensaje al front-end.

1. Actualice la función `GetMessage` en _api/GetMessage/index.js_ con el código siguiente.

    ```javascript
    module.exports = async function (context, req) {
      context.res = {
        body: {
          text: "Hello from the API"
        }
      };
    };
    ```

1. Actualice la configuración de `GetMessage` en `api/GetMessage/function.json` con las siguientes opciones.

    ```json
    {
      "bindings": [
        {
          "authLevel": "anonymous",
          "type": "httpTrigger",
          "direction": "in",
          "name": "req",
          "methods": [
            "get"
          ],
          "route": "message"
        },
        {
          "type": "http",
          "direction": "out",
          "name": "res"
        }
      ]
    }
    ```

Con la configuración anterior, el punto de conexión de la API:

- Se desencadena cuando se realiza una solicitud HTTP a la función
- Está disponible para todas las solicitudes, independientemente del estado de autenticación
- Está expuesta a través de la ruta _/api/message_

## <a name="run-the-api-locally"></a>Ejecución local de la API

Visual Studio Code se integra con [Azure Functions Core Tools](../azure-functions/functions-run-local.md) para que pueda ejecutar este proyecto en el equipo de desarrollo local antes de publicarlo en Azure.

> [!TIP]
> Asegúrese de que tiene todos los recursos que se mencionan en la sección de [requisitos previos](#prerequisites) instalados antes de continuar.

1. Ejecute la función presionando **F5** para iniciar la aplicación de Functions.

1. Si Azure Functions Core Tools aún no está instalado, seleccione **Instalar** en la ventana.

    Core Tools muestra la salida de la aplicación en ejecución en el panel _Terminal_. Como parte de la salida, puede ver el punto de conexión de la dirección URL de la función desencadenada por HTTP que se ejecuta localmente.

    :::image type="content" source="media/add-api/create-azure-functions-vscode-2.png" alt-text="Creación de una nueva instancia de Azure Functions mediante Visual Studio Code":::

1. Con las herramientas principales en ejecución, vaya a la siguiente dirección URL para comprobar que la API se está ejecutando correctamente: `http://localhost:7071/api/message`.

   La respuesta en el explorador debe ser similar a la del ejemplo siguiente:

   :::image type="content" source="media/add-api/create-azure-functions-vscode-3.png" alt-text="Creación de una nueva instancia de Azure Functions mediante Visual Studio Code":::

1. Para detener la sesión de depuración, presione **Mayús+F5**.

### <a name="call-the-api-from-the-application"></a>Llamada a la API desde la aplicación

Cuando se implementan en Azure, las solicitudes a la API se enrutan automáticamente a la aplicación de Functions para las solicitudes enviadas a la ruta de `api`. Al trabajar localmente, debe configurar las opciones de la aplicación para las solicitudes de proxy a la API local.

[!INCLUDE [static-web-apps-local-proxy](../../includes/static-web-apps-local-proxy.md)]

#### <a name="update-html-files-to-access-the-api"></a>Actualización de archivos HTML para acceder a la API

1. A continuación, actualice el contenido del archivo _index.html_ con el código siguiente para capturar el texto de la función de la API y mostrarlo en la pantalla:

   ```html
   <!DOCTYPE html>
   <html lang="en">

   <head>
     <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
     <link rel="stylesheet" href="styles.css">
     <title>Vanilla JavaScript App</title>
   </head>

   <body>
     <main>
       <h1>Vanilla JavaScript App</h1>
       <p>Loading content from the API: <b id="name">...</b></p>
     </main>

     <script>
       (async function() {
         let { text } = await( await fetch(`/api/message`)).json();
         document.querySelector('#name').textContent = text;
       }())
     </script>
   </body>

   </html>
   ```

1. Presione **F5** para iniciar el proyecto de la API.

1. Presione **F1** y elija **Live Server: Open with Live Server** (Abrir con Live Server).

    Ahora debería ver el mensaje de la API en la página web.

   :::image type="content" source="media/add-api/create-azure-functions-vscode-4.png" alt-text="Creación de una nueva instancia de Azure Functions mediante Visual Studio Code":::

   > [!NOTE]
   > Puede usar otros servidores proxy o HTTP para proporcionar el archivo `index.html`. No podrá acceder a `index.html` desde `file:///`.

1. Presione **Mayús + F5** para detener el proyecto de API.

### <a name="commit-and-push-your-changes-to-github"></a>Confirmación y envío de los cambios a GitHub

Con Visual Studio Code, confirme ("commit") e inserte los cambios ("push") en el repositorio Git remoto.

1. Presione **F1** para abrir la paleta de comandos.
1. Escriba **Git: Commit All** (Git: Confirmar todo)
1. Agregue un mensaje de confirmación y presione **Entrar**.
1. Presione **F1**.
1. Escriba **Git: push** y presione **Entrar**.

## <a name="create-a-static-web-app"></a>Creación de una aplicación web estática

1. Vaya a [Azure Portal](https://portal.azure.com).
1. Haga clic en **Crear un recurso**.
1. Busque **Aplicación web estática**.
1. Haga clic en **Aplicación web estática (versión preliminar)** .
1. Haga clic en **Crear**

A continuación, agregue la configuración específica de la aplicación.

1. Seleccione su _suscripción a Azure_.
1. Seleccione o cree un nuevo _grupo de recursos_.
1. Asigne a la aplicación el nombre **my-vanilla-api**.
1. Seleccione la _región_ más cercana a la suya.
1. Seleccione la **SKU** _gratis_.
1. Haga clic en el botón **Iniciar sesión con GitHub** y realice la autenticación con GitHub.
1. Seleccione su _organización_ preferida.
1. Seleccione **my-vanilla-api** del menú desplegable _Repositorio_.
1. Seleccione **master** en el menú desplegable _Rama_.
1. Haga clic en el botón **Siguiente: Compilar >** para editar la configuración de compilación.

A continuación, agregue los siguientes detalles de compilación.

1. Escriba **/** como _Ubicación de la aplicación_.
1. Escriba **api** en el cuadro _Ubicación de la API_.
1. Borre el valor predeterminado de la _Ubicación del artefacto de la aplicación_ y deje el cuadro vacío.
1. Haga clic en **Revisar + crear**.
1. Haga clic en el botón **Crear**.

    Una vez que haga clic en el botón _Crear_, Azure realizará dos acciones. En primer lugar, se crean los servicios en la nube subyacentes para admitir la aplicación. Después, un proceso en segundo plano comienza a compilar e implementar la aplicación.

1. Haga clic en el botón **Ir al recurso** para acceder a la página _Información general_ de la aplicación web.

    Mientras que la aplicación se está generando en segundo plano, puede hacer clic en el banner que contiene un vínculo para ver el estado de la compilación.

    :::image type="content" source="media/add-api/github-action-flag.png" alt-text="Flujo de trabajo de GitHub":::

1. Una vez completada la implementación, puede navegar a la aplicación web haciendo clic en el vínculo _URL_ que se muestra en la página _Información general_.

    :::image type="content" source="media/add-api/static-app-url-from-portal.png" alt-text="Acceso a la dirección URL de la aplicación estática desde Azure Portal":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no quiere conservar esta aplicación para usos adicionales, puede usar los pasos siguientes para eliminar la aplicación de Azure Static Web App y sus recursos relacionados.

1. Vaya a [Azure Portal](https://portal.azure.com).
1. En la barra de búsqueda de la parte superior, escriba **Grupos de recursos**.
1. Haga clic en **Grupos de recursos**.
1. Seleccione **myResourceGroup**.
1. En la página _myResourceGroup_, asegúrese de que los recursos enumerados sean los que desea eliminar.
1. Seleccionar **Eliminar**
1. Escriba **myResourceGroup** en el cuadro de texto.
1. Seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Configuración de aplicaciones](./application-settings.md)
