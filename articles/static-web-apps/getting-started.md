---
title: 'Inicio rápido: creación de su primera aplicación web estática con Azure Static Web Apps'
description: Aprenda a crear una instancia de Azure Static Web Apps con su marco de front-end preferido.
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: quickstart
ms.date: 05/08/2020
ms.author: cshoe
ms.openlocfilehash: 6738f598275e91ce8a811c3ef6bcc6d5dc84e0bd
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87089505"
---
# <a name="quickstart-building-your-first-static-web-app"></a>Inicio rápido: creación de la primera aplicación web estática

Azure Static Web Apps publica sitios web en un entorno de producción mediante la creación de aplicaciones desde un repositorio de GitHub. En este inicio rápido, va a compilar una aplicación web con su marco de front-end preferido desde un repositorio de GitHub.

Si no tiene ninguna suscripción a Azure, [cree una cuenta de evaluación gratuita](https://azure.microsoft.com/free).

## <a name="prerequisites"></a>Requisitos previos

- [GitHub](https://github.com)
- Cuenta de [Azure](https://portal.azure.com)

## <a name="create-a-repository"></a>Creación de un repositorio

En este artículo se usan repositorios de plantillas de GitHub para facilitar la creación de un nuevo repositorio. Las plantillas incluyen aplicaciones de inicio compiladas con diferentes marcos de front-end.

# <a name="angular"></a>[Angular](#tab/angular)

- Asegúrese de que ha iniciado sesión en GitHub y, a continuación, vaya a la siguiente ubicación para crear un nuevo repositorio.
  - https://github.com/staticwebdev/angular-basic/generate
- Asigne el nombre **my-first-static-web-app** al repositorio.

# <a name="react"></a>[React](#tab/react)

- Asegúrese de que ha iniciado sesión en GitHub y, a continuación, vaya a la siguiente ubicación para crear un nuevo repositorio.
  - https://github.com/staticwebdev/react-basic/generate
- Asigne el nombre **my-first-static-web-app** al repositorio.

# <a name="vue"></a>[Vue](#tab/vue)

- Asegúrese de que ha iniciado sesión en GitHub y, a continuación, vaya a la siguiente ubicación para crear un nuevo repositorio.
  - https://github.com/staticwebdev/vue-basic/generate
- Asigne el nombre **my-first-static-web-app** al repositorio.

# <a name="no-framework"></a>[Ningún marco](#tab/vanilla-javascript)

- Asegúrese de que ha iniciado sesión en GitHub y, a continuación, vaya a la siguiente ubicación para crear un nuevo repositorio.
  - https://github.com/staticwebdev/vanilla-basic/generate
- Asigne el nombre **my-first-static-web-app** al repositorio.

> [!NOTE]
> Azure Static Web Apps requiere al menos un archivo HTML para crear una aplicación web. El repositorio que se crea en este paso incluye un solo archivo _index.html_.

---

Haga clic en **Crear repositorio a partir de plantilla**.

:::image type="content" source="media/getting-started/create-template.png" alt-text="Crear repositorio a partir de plantilla":::

## <a name="create-a-static-web-app"></a>Creación de una aplicación web estática

Ahora que se ha creado el repositorio, puede crear una aplicación web estática desde Azure Portal.

- Vaya a [Azure Portal](https://portal.azure.com).
- Haga clic en **Crear un recurso**.
- Busque **Static Web Apps**.
- Haga clic en **Static Web Apps (Preview)** (Static Web Apps (versión preliminar)).
- Haga clic en **Crear**

### <a name="basics"></a>Aspectos básicos

Para empezar, configure la nueva aplicación y vincúlela a un repositorio de GitHub.

:::image type="content" source="media/getting-started/basics-tab.png" alt-text="Pestaña Aspectos básicos":::

- Seleccione su _suscripción a Azure_.
- Seleccione o cree un nuevo _grupo de recursos_.
- Asigne el nombre **my-first-static-web-app** a la aplicación.
  - Los caracteres válidos son `a-z` (no distingue mayúsculas de minúsculas), `0-9` y `-`.
- Seleccione la _región_ más cercana a la suya.
- Seleccione la _SKU_ **gratis**.
- Haga clic en el botón **Iniciar sesión con GitHub** y realice la autenticación con GitHub.

Una vez que inicie sesión con GitHub, escriba la información del repositorio.

:::image type="content" source="media/getting-started/repository-details.png" alt-text="Detalles del repositorio":::

- Seleccione su _organización_ preferida.
- Seleccione **my-first-web-static-app** en el menú desplegable _Repositorio_.
- Seleccione **master** en el menú desplegable _Rama_.
- Haga clic en el botón **Siguiente: Compilar >** para editar la configuración de compilación.

:::image type="content" source="media/getting-started/next-build-button.png" alt-text="Botón Siguiente: Compilar":::

> [!NOTE]
>  Si no ve ningún repositorio, es posible que deba autorizar Azure Static Web Apps en GitHub. Vaya al repositorio de GitHub y, a continuación, a **Configuración > Aplicaciones > Aplicaciones de OAuth autorizadas**, seleccione **Azure Static Web Apps** y, después, **Conceder**. En el caso de los repositorios de la organización, debe ser propietario de la organización para conceder los permisos.

### <a name="build"></a>Build

A continuación, agregue los detalles de configuración específicos de su plataforma de front-end preferida.

# <a name="angular"></a>[Angular](#tab/angular)

- Escriba **/** en el cuadro _Ubicación de la aplicación_.
- Borre el valor predeterminado del cuadro _Ubicación de la API_.
- Escriba **dist/angular-basic** en el cuadro _Ubicación del artefacto de la aplicación_.

# <a name="react"></a>[React](#tab/react)

- Escriba **/** en el cuadro _Ubicación de la aplicación_.
- Borre el valor predeterminado del cuadro _Ubicación de la API_.
- Escriba **build** en el cuadro _Ubicación del artefacto de la aplicación_.

# <a name="vue"></a>[Vue](#tab/vue)

- Escriba **/** en el cuadro _Ubicación de la aplicación_.
- Borre el valor predeterminado del cuadro _Ubicación de la API_.
- Escriba **dist** en el cuadro _Ubicación del artefacto de la aplicación_.

# <a name="no-framework"></a>[Ningún marco](#tab/vanilla-javascript)

- Escriba **/** en el cuadro _Ubicación de la aplicación_.
- Borre el valor predeterminado del cuadro _Ubicación de la API_.
- Borre el valor predeterminado del cuadro _Ubicación del artefacto de la aplicación_.

---

Haga clic en el botón **Revisar y Crear**.

:::image type="content" source="media/getting-started/review-create.png" alt-text="Botón Revisar y crear":::

Para cambiar estos valores después de crear la aplicación, puede editar el [archivo de flujo de trabajo](github-actions-workflow.md).

### <a name="review--create"></a>Revisar y crear

Después de validar la solicitud, puede continuar con la creación de la aplicación.

Haga clic en el botón **Crear**.

:::image type="content" source="media/getting-started/create-button.png" alt-text="Botón Crear":::

Una vez creado el recurso, haga clic en el botón **Ir al recurso**.

:::image type="content" source="media/getting-started/resource-button.png" alt-text="Botón Ir al recurso":::

## <a name="view-the-website"></a>Vista del sitio web

Hay dos aspectos para implementar una aplicación estática. El primero aprovisiona los recursos de Azure subyacentes que componen la aplicación. El segundo es un flujo de trabajo de Acciones de GitHub que crea y publica la aplicación.

Para poder navegar a un nuevo sitio estático, la compilación de implementación debe terminar de ejecutarse.

En la ventana Información general de Static Web Apps se muestran una serie de vínculos que le ayudarán a interactuar con la aplicación web.

:::image type="content" source="media/getting-started/overview-window.png" alt-text="Ventana Información general":::

1. Al hacer clic en el banner con el texto "Click here to check the status of your GitHub Actions runs" (Haga clic aquí para comprobar el estado de las ejecuciones de las Acciones de GitHub"), se le dirigirá a las Acciones de GitHub que se ejecutan en el repositorio. Una vez que haya verificado que el trabajo de implementación se ha completado, puede ir a su sitio web a través de la URL generada.

2. Una vez completado el flujo de trabajo de Acciones de GitHub, puede hacer clic en el vínculo con la _dirección URL_ para abrir el sitio web en una pestaña nueva.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, puede eliminar la instancia de Azure Static Web Apps mediante los siguientes pasos:

1. Abra [Azure Portal](https://portal.azure.com).
1. Busque **my-first-web-static-app** en la barra de búsqueda superior.
1. Haga clic en el nombre de la aplicación.
1. Haga clic en el botón **Eliminar**.
1. Haga clic en **Sí** para confirmar la acción de eliminación.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Adición de una API](add-api.md)
