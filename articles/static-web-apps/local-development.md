---
title: Configuración del desarrollo local para Azure Static Web Apps
description: Aprenda a configurar su entorno de desarrollo local para Azure Static Web Apps
services: static-web-apps
author: burkeholland
ms.service: static-web-apps
ms.topic: how-to
ms.date: 05/08/2020
ms.author: buhollan
ms.custom: devx-track-javascript
ms.openlocfilehash: d9988adea8e4d2dab39de64f9fd155a9d0b02cae
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87432965"
---
# <a name="set-up-local-development-for-azure-static-web-apps-preview"></a>Configuración del desarrollo local para Azure Static Web Apps (versión preliminar)

Una instancia de Azure Static Web Apps se compone de dos tipos distintos de aplicación. La primera es una aplicación web para el contenido estático. A menudo, las aplicaciones web se crean con marcos y bibliotecas front-end o con generadores de sitios estáticos. El segundo aspecto es la API, que es una aplicación de Azure Functions que proporciona un entorno de desarrollo de back-end enriquecido.

Cuando se ejecuta en la nube, Azure Static Web Apps asigna fácilmente las solicitudes a la ruta de `api` desde la aplicación web a la aplicación de Azure Functions sin requerir la configuración de CORS. Localmente, debe configurar la aplicación para que imite este comportamiento.

En este artículo se muestran los procedimientos recomendados para el desarrollo local, incluidos los siguientes conceptos:

- Configuración de la aplicación web para contenido estático
- Configuración de la aplicación de Azure Functions para la API de la aplicación
- Depuración y ejecución de la aplicación
- Procedimientos recomendados para la estructura de archivos y carpetas de la aplicación

## <a name="prerequisites"></a>Requisitos previos

- [Visual Studio Code](https://code.visualstudio.com/)
- [Extensión de Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) para Visual Studio Code
- [Extensión Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) para Visual Studio Code
  - Solo es necesario si no usa un marco de trabajo de JavaScript de front-end o la CLI del generador de sitios estáticos.

## <a name="run-projects-locally"></a>Ejecución local de proyectos

La ejecución local de una aplicación de Azure Static Web Apps implica tres procesos, en función de si el proyecto contiene una API o no.

- Ejecución de un servidor web local
- Ejecución de la API
- Conexión del proyecto web a la API

En función de cómo se ha creado un sitio web, es posible que se necesite un servidor web local o no para ejecutar la aplicación en el explorador. Al usar marcos de JavaScript de front-end y generadores de sitios estáticos, esta funcionalidad está integrada en sus respectivas CLI (interfaces de línea de comandos). Los siguientes vínculos apuntan a la referencia de la CLI para una selección de marcos, bibliotecas y generadores.

### <a name="javascript-frameworks-and-libraries"></a>Marcos y bibliotecas de JavaScript

- [Angular CLI](https://angular.io/cli)
- [Vue CLI](https://cli.vuejs.org/guide/creating-a-project.html)
- [React CLI](https://create-react-app.dev/)

### <a name="static-site-generators"></a>Generadores de sitios estáticos

- [Gatsby CLI](https://www.gatsbyjs.org/docs/gatsby-cli/)
- [Hugo](https://gohugo.io/getting-started/quick-start/)
- [Jekyll](https://jekyllrb.com/docs/usage/)

Si usa una herramienta de la CLI para entregar su sitio, puede ir directamente a la sección [Ejecución de la API](#run-api-locally).

### <a name="running-a-local-web-server-with-live-server"></a>Ejecución de un servidor web local con Live Server

La extensión Live Server para Visual Studio Code ofrece un servidor web de desarrollo local que entrega el contenido estático.

#### <a name="create-a-repository"></a>Creación de un repositorio

1. Asegúrese de que ha iniciado sesión en GitHub, vaya a [https://github.com/staticwebdev/vanilla-api/generate](https://github.com/staticwebdev/vanilla-api/generate) y cree un proyecto de GitHub denominado **vanilla-api** con esta plantilla.

    :::image type="content" source="media/local-development/vanilla-api.png" alt-text="Ventana Nuevo repositorio de GitHub":::

1. Abra Visual Studio Code.

1. Presione **F1** para abrir la paleta de comandos.

1. Escriba **clone** en el cuadro de búsqueda y seleccione **Git: Clone**.

    :::image type="content" source="media/local-development/command-palette-git-clone.png" alt-text="Opción git clone en Visual Studio Code":::

1. Escriba el siguiente valor en **Dirección URL del repositorio**.

   ```http
   git@github.com:<YOUR_GITHUB_ACCOUNT>/vanilla-api.git
   ```

1. Seleccione la ubicación de la carpeta para el nuevo proyecto.

1. Cuando se le pida que abra el repositorio clonado, seleccione **Abrir**.

    :::image type="content" source="media/local-development/open-new-window.png" alt-text="Abrir en nueva ventana":::

Visual Studio Code abre el proyecto clonado en el editor.

### <a name="run-the-website-locally-with-live-server"></a>Ejecución del sitio web localmente con Live Server

1. Presione **F1** para abrir la paleta de comandos.

1. Escriba **Live Server** en el cuadro de búsqueda y seleccione **Live Server: Open with Live Server** (Abrir con Live Server).

    Se abre una pestaña del explorador para mostrar la aplicación.

    :::image type="content" source="media/local-development/vanilla-api-site.png" alt-text="Sitio estático simple que se ejecuta en el explorador":::

    Esta aplicación realiza una solicitud HTTP al punto de conexión `api/message`. En este momento, se produce un error en la solicitud porque es necesario iniciar la parte de la API de esta aplicación.

### <a name="run-api-locally"></a>Ejecución local de la API

Las API de Azure Static Web Apps se basan en tecnología de Azure Functions. Consulte [Incorporación de una API en Azure Static Web Apps con Azure Functions](add-api.md) para obtener más información sobre cómo agregar una API a un proyecto de Azure Static Web Apps.

Como parte del proceso de creación de la API, se crea una configuración de inicio para Visual Studio Code. Esta configuración se encuentra en la carpeta _.vscode_. Esta carpeta contiene toda la configuración necesaria para compilar y ejecutar la API de forma local.

1. En Visual Studio Code, presione **F5** para iniciar la API.

1. Se abre una nueva instancia del terminal que muestra la salida del proceso de compilación de la API.

    :::image type="content" source="media/local-development/terminal-api-debug.png" alt-text="API en ejecución en el terminal de Visual Studio Code":::

   La barra de estado de Visual Studio Code ahora está naranja. Este color indica que la API se está ejecutando ahora y que el depurador está conectado.

1. A continuación, presione **CTRL/Cmd** y haga clic en la dirección URL en el terminal para abrir una ventana del explorador que llama a la API.

    :::image type="content" source="media/local-development/hello-from-api-endpoint.png" alt-text="Resultado de visualización del explorador de la llamada a la API":::

### <a name="debugging-the-api"></a>Depuración de la API

1. Abra el archivo _api/GetMessage/index.js_ en Visual Studio Code.

1. Haga clic en el margen izquierdo de la línea 2 para establecer un punto de interrupción. Aparece un punto rojo que indica que el punto de interrupción está establecido.

    :::image type="content" source="media/local-development/breakpoint-set.png" alt-text="Punto de interrupción en Visual Studio Code":::

1. En el explorador, actualice la página en ejecución en <http://127.0.0.1:7071/api/message>.

1. Se alcanza el punto de interrupción en Visual Studio Code y se pausa la ejecución del programa.

   :::image type="content" source="media/local-development/breakpoint-hit.png" alt-text="Punto de interrupción alcanzado en Visual Studio Code":::

   Hay disponible una [experiencia de depuración completa en Visual Studio Code](https://code.visualstudio.com/Docs/editor/debugging) para la API.

1. Presione el botón **Continuar** de la barra de depuración para continuar con la ejecución.

    :::image type="content" source="media/local-development/continue-button.png" alt-text="Botón Continuar en Visual Studio Code":::

### <a name="calling-the-api-from-the-application"></a>Llamada a la API desde la aplicación

Tras la implementación, Azure Static Web Apps asigna automáticamente estas solicitudes a los puntos de conexión en la carpeta de _api_. Esta asignación se asegura de que las solicitudes de la aplicación a la API sean similares a las del ejemplo siguiente.

```javascript
let response = await fetch("/api/message");
```

En función de si la aplicación se ha compilado con una CLI con marco de JavaScript, hay dos maneras de configurar la ruta de acceso a la ruta `api` cuando la aplicación se ejecuta localmente.

- Archivos de configuración del entorno (recomendado para los marcos y bibliotecas de JavaScript)
- Proxy local

### <a name="environment-configuration-files"></a>Archivos de configuración del entorno

Si va a compilar la aplicación con marcos de front-end que tienen una CLI, debe usar los archivos de configuración del entorno. Cada marco o biblioteca tiene una forma diferente de administrar estos archivos de configuración del entorno. Es habitual tener un archivo de configuración para desarrollo, que se usa cuando la aplicación se ejecuta localmente, y otro para producción, que se usa cuando la aplicación se ejecuta en producción. La CLI para el marco de JavaScript o el generador de sitios estáticos que uses sabrá automáticamente usar el archivo de desarrollo localmente y el archivo de producción cuando Azure Static Web Apps compile la aplicación.

En el archivo de configuración de desarrollo, puede especificar la ruta de acceso a la API, que apunta a la ubicación local de `http:127.0.0.1:7071` donde la API de su sitio se ejecuta localmente.

```
API=http:127.0.0.1:7071/api
```

En el archivo de configuración de producción, especifique la ruta de acceso a la API como `api`. De esta manera, la aplicación llamará a la API a través de "susitio.com/api" cuando se ejecute en producción.

```
API=api
```

Se puede hacer referencia a estos valores de configuración como variables de entorno de Node en JavaScript de la aplicación web.

```js
let response = await fetch(`${process.env.API}/message`);
```

Cuando se usa la CLI para ejecutar el sitio en modo de desarrollo o para compilar el sitio para producción, el valor `process.env.API` se reemplaza por el valor del archivo de configuración adecuado.

Para obtener más información sobre la configuración de archivos de entorno para los marcos y bibliotecas de JavaScript para front-end, consulte estos artículos:

- [Variable de entorno de Angular](https://angular.io/guide/build#configuring-application-environments)
- [React: Adding Custom Environment Variables](https://create-react-app.dev/docs/adding-custom-environment-variables/)
- [Vue: Modes and Environment Variables](https://cli.vuejs.org/guide/mode-and-env.html)

[!INCLUDE [static-web-apps-local-proxy](../../includes/static-web-apps-local-proxy.md)]

##### <a name="restart-live-server"></a>Reinicio de Live Server

1. Presione **F1** para abrir la paleta de comandos en Visual Studio Code.

1. Escriba **Live Server** y seleccione **Live Server: Stop Live Server** (Detener Live Server).

    :::image type="content" source="media/local-development/stop-live-server.png" alt-text="Comando Stop Live Server en la paleta de comandos de Visual Studio":::

1. Presione **F1** para abrir la paleta de comandos.

1. Escriba **Live Server** y seleccione **Live Server: Open with Live Server** (Abrir con Live Server).

1. Actualice la aplicación que se ejecuta en `http://locahost:3000`. Ahora, el explorador muestra el mensaje devuelto desde la API.

    :::image type="content" source="media/local-development/hello-from-api.png" alt-text="Hello from the API en el explorador":::

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Configuración de aplicaciones](application-settings.md)
