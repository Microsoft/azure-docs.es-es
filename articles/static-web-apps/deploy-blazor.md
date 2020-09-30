---
title: 'Tutorial: Creación de una aplicación web estática con Blazor en Azure Static Web Apps'
description: Aprenda a crear un sitio web con Azure Static Web Apps con Blazor.
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: tutorial
ms.date: 09/10/2020
ms.author: cshoe
ms.openlocfilehash: 2a8e0ec113b4d008f759c7d199c4dab823576e16
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90929745"
---
# <a name="tutorial-building-a-static-web-app-with-blazor-in-azure-static-web-apps"></a>Tutorial: Creación de una aplicación web estática con Blazor en Azure Static Web Apps

Azure Static Web Apps publica sitios web en entornos de producción mediante la creación de aplicaciones desde un repositorio de GitHub. En este tutorial, se implementa una aplicación web en Azure Static Web Apps mediante Azure Portal.

Si no tiene ninguna suscripción a Azure, [cree una cuenta de evaluación gratuita](https://azure.microsoft.com/free).

## <a name="prerequisites"></a>Requisitos previos

- [GitHub](https://github.com)
- Cuenta de [Azure](https://portal.azure.com)

## <a name="application-overview"></a>Información general de la aplicación

Azure Static Web Apps permite crear aplicaciones web estáticas compatibles con un back-end sin servidor. En el siguiente tutorial se muestra cómo implementar una aplicación web C# de Blazor que devuelve datos meteorológicos.

:::image type="content" source="./media/deploy-blazor/blazor-app-complete.png" alt-text="Aplicación Blazor completa":::

La aplicación que se incluye en este tutorial se compone de tres proyectos de Visual Studio diferentes:

- **API**: la aplicación C# de Azure Functions que implementa el punto de conexión de API que proporciona información meteorológica a la aplicación estática. La función [`WeatherForecastFunction`](https://github.com/staticwebev/blazor-starter/blob/main/Api/WeatherForecastFunction.cs) devuelve una matriz de objetos `WeatherForecast`.

- **Cliente**: el proyecto de ensamblado web Blazor del servidor front-end. Se implementa una [ruta de reserva](#fallback-route) para garantizar que todas las rutas reciben el archivo _index.html_.

- **Shared:** contiene clases comunes a las que hacen referencia los proyectos de API y cliente, lo que permite que los datos fluyan desde el punto de conexión de la API a la aplicación web de front-end. La clase [`WeatherForecast`](https://github.com/staticwebdev/blazor-starter/blob/main/Shared/WeatherForecast.cs) se comparte entre ambas aplicaciones.

Juntos, estos proyectos constituyen los elementos necesarios para crear una aplicación de ensamblado web de Blazor que se ejecuta en el explorador con el respaldo de un back-end de API.

## <a name="fallback-route"></a>Ruta de reserva

La aplicación expone direcciones URL como _/counter_ y _/fetchdata_ que se asignan a rutas específicas de la aplicación. Puesto que esta aplicación se implementa como una aplicación de página única, cada ruta recibe el archivo _index.html_. Para asegurarse de que la solicitud de cualquier ruta de acceso devuelva _index.html_, se implementa una [ruta de reserva](./routes.md#fallback-routes) en el archivo _routes.json_ que se encuentra en la carpeta _wwwroot_ del proyecto Client.

```json
{
  "routes": [
    {
      "route": "/*",
      "serve": "/index.html",
      "statusCode": 200
    }
  ]
}
```

La configuración anterior garantiza que las solicitudes a cualquier ruta de la aplicación devuelvan la página _index.html_.

## <a name="create-a-repository"></a>Creación de un repositorio

En este artículo se usa un repositorio de plantillas de GitHub para facilitar los primeros pasos. La plantilla incluye una aplicación de inicio que se implementa en Azure Static Web Apps.

1. Asegúrese de que ha iniciado sesión en GitHub y, a continuación, vaya a la siguiente ubicación para crear un repositorio:
    - https://github.com/staticwebdev/blazor-starter/generate
1. Asigne el nombre **my-first-static-blazor-app** al repositorio.

## <a name="create-a-static-web-app"></a>Creación de una aplicación web estática

Ahora que se ha creado el repositorio, cree una aplicación web estática desde Azure Portal.

1. Vaya a [Azure Portal](https://portal.azure.com).
1. Seleccione **Crear un recurso**
1. Busque **Static Web Apps**.
1. Seleccione **Static Web Apps (Preview)** (Static Web Apps [versión preliminar]).
1. Seleccione **Crear**

En la sección _Aspectos básicos_, configure la nueva aplicación y vincúlela a un repositorio de GitHub.

:::image type="content" source="media/deploy-blazor/basics.png" alt-text="Pestaña Aspectos básicos":::

1. Seleccione su _suscripción a Azure_.
1. Seleccione o cree un nuevo _grupo de recursos_.
1. Asigne el nombre **my-first-static-blazor-app** a la aplicación.
    - Los caracteres válidos son `a-z` (no distingue mayúsculas de minúsculas), `0-9` y `-`.
1. Seleccione la _región_ más cercana a la suya.
1. Seleccione la _SKU_ **gratis**.
1. Seleccione el botón **Sign-in with GitHub** (Iniciar sesión con GitHub) y autentíquese con GitHub

Tras iniciar sesión con GitHub, escriba la información del repositorio.

:::image type="content" source="media/deploy-blazor/repository-details.png" alt-text="Detalles del repositorio":::

1. Seleccione su _organización_ preferida.
1. Seleccione **my-first-static-blazor-app** en el menú desplegable _Repositorio_.
1. Seleccione **main** en el menú desplegable _Rama_.

    Si no ve ningún repositorio, es posible que deba autorizar Azure Static Web Apps en GitHub. Vaya al repositorio de GitHub y, a continuación, a **Configuración > Aplicaciones > Aplicaciones de OAuth autorizadas**, seleccione **Azure Static Web Apps** y, después, **Conceder**. En el caso de los repositorios de la organización, debe ser propietario de la organización para conceder los permisos.

1. En la sección _Detalles de la compilación_, agregue información de configuración específicos de Blazor.

    - Seleccione **Blazor** en la lista desplegable _Build Presets_ (Valores prestablecidos de compilación) y mantenga todos los valores predeterminados.

1. Seleccione **Revisar + crear**.

    :::image type="content" source="media/deploy-blazor/review-create.png" alt-text="Botón Revisar y crear":::

1. Seleccione **Crear**.

    :::image type="content" source="media/deploy-blazor/create-button.png" alt-text="Botón Crear":::

1. Seleccione **Ir al recurso**.

    :::image type="content" source="media/deploy-blazor/resource-button.png" alt-text="Botón Ir al recurso":::

## <a name="view-the-website"></a>Vista del sitio web

Hay dos aspectos para implementar una aplicación estática. El primero aprovisiona los recursos de Azure subyacentes que componen la aplicación. El segundo es un flujo de trabajo de Acciones de GitHub que crea y publica la aplicación.

Para poder navegar a un nuevo sitio estático, la compilación de implementación debe terminar de ejecutarse.

En la ventana Información general de Static Web Apps se muestran una serie de vínculos que le ayudarán a interactuar con la aplicación web.

:::image type="content" source="./media/deploy-blazor/overview-window.png" alt-text="Ventana Información general":::

1. Al hacer clic en el banner con el texto, _Click here to check the status of your GitHub Actions runs_ (Haga clic aquí para comprobar el estado de las ejecuciones de Acciones de GitHub en su repositorio). Una vez que haya verificado que el trabajo de implementación se ha completado, puede ir a su sitio web a través de la URL generada.

2. Una vez completado el flujo de trabajo de Acciones de GitHub, puede seleccionar el vínculo con la _dirección URL_ para abrir el sitio web en una pestaña nueva.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, puede eliminar la instancia de Azure Static Web Apps mediante los siguientes pasos:

1. Abra [Azure Portal](https://portal.azure.com).
1. Busque **my-first-static-blazor-app** en la barra de búsqueda superior.
1. Selección del nombre de la aplicación
1. Seleccione el botón **Eliminar**.
1. Seleccione **Sí** para confirmar la acción de eliminación.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Autenticación y autorización](./authentication-authorization.md)
