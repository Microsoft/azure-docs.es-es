---
title: 'Tutorial: Publicación de un sitio de Gatsby en Azure Static Web Apps'
description: En este tutorial se muestra cómo implementar una aplicación de Gatsby en Azure Static Web Apps.
services: static-web-apps
author: aaronpowell
ms.service: static-web-apps
ms.topic: tutorial
ms.date: 05/08/2020
ms.author: aapowell
ms.custom: devx-track-javascript
ms.openlocfilehash: 03e9f99301444032d867bc1c2f854c807380844e
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88797741"
---
# <a name="tutorial-publish-a-gatsby-site-to-azure-static-web-apps-preview"></a>Tutorial: Publicación de un sitio de Gatsby en Azure Static Web Apps (versión preliminar)

En este artículo se muestra cómo crear e implementar una aplicación web de [Gatsby](https://gatsbyjs.org) en [Azure Static Web Apps](overview.md). El resultado final es un nuevo sitio de Static Web Apps (con las Acciones de GitHub asociadas) que le da control sobre cómo se compila y publica la aplicación.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> - Crear una aplicación de Gatsby
> - Configurar un sitio de Azure Static Web Apps
> - Implementar la aplicación de Gatsby en Azure

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. En caso de no tener una, puede [crear una cuenta gratuita](https://azure.microsoft.com/free/).
- Una cuenta de GitHub. En caso de no tener una, puede [crear una cuenta gratuita](https://github.com/join).
- [Node.js](https://nodejs.org) instalado.

## <a name="create-a-gatsby-app"></a>Creación de una aplicación de Gatsby

Cree una aplicación de Gatsby mediante la interfaz de la línea de comandos (CLI) de Gatsby:

1. Abra un terminal.
1. Use la herramienta [npx](https://www.npmjs.com/package/npx) para crear una aplicación con la CLI de Gatsby. Esta operación puede tardar unos minutos.

   ```bash
   npx gatsby new static-web-app
   ```

1. Vaya a la aplicación recién creada.

   ```bash
   cd static-web-app
   ```

1. Inicialice un repositorio de Git.

   ```bash
   git init
   git add -A
   git commit -m "initial commit"
   ```

## <a name="push-your-application-to-github"></a>Inserción de la aplicación en GitHub

Debe tener un repositorio en GitHub para crear un recurso de Azure Static Web Apps.

1. Cree un repositorio de GitHub en blanco (no cree un archivo Léame) desde [https://github.com/new](https://github.com/new), denominado **gatsby-static-web-app**.

1. A continuación, agregue el repositorio de GitHub que acaba de crear como remoto al repositorio local. Asegúrese de agregar el nombre de usuario de GitHub en lugar del marcador de posición `<YOUR_USER_NAME>` en el comando siguiente.

   ```bash
   git remote add origin https://github.com/<YOUR_USER_NAME>/gatsby-static-web-app
   ```

1. Inserte el repositorio local en GitHub.

   ```bash
   git push --set-upstream origin master
   ```

## <a name="deploy-your-web-app"></a>Implementación de la aplicación web

En los pasos siguientes se muestra cómo crear una aplicación de sitio estática e implementarla en un entorno de producción.

### <a name="create-the-application"></a>Creación de la aplicación

1. Vaya a [Azure Portal](https://portal.azure.com).
1. Haga clic en **Crear un recurso**.
1. Busque **Static Web Apps**.
1. Haga clic en **Static Web Apps (Preview)** (Static Web Apps [versión preliminar]).
1. Haga clic en **Crear**

   :::image type="content" source="./media/publish-gatsby/create-in-portal.png" alt-text="Creación de una aplicación de Static Web Apps (versión preliminar) en el portal":::

1. En _Suscripción_, acepte la suscripción que aparece o seleccione otra en la lista desplegable.

1. En _Grupo de recursos_, seleccione **Nuevo**. En _Nuevo nombre de grupo de recursos_, escriba **gatsby-static-web-app** y seleccione **Aceptar**.

1. Después, un nombre para la aplicación en el cuadro **Nombre**. Los caracteres válidos incluyen `a-z`, `A-Z`, `0-9` y `-`.

1. En _Región_, seleccione una región cercana disponible.

1. En _SKU_, seleccione **Gratis**.

   :::image type="content" source="./media/publish-gatsby/basic-app-details.png" alt-text="Detalles rellenados":::

1. Haga clic en el botón **Iniciar sesión con GitHub**.

1. Seleccione la **Organización** en la que creó el repositorio.

1. Seleccione **gatsby-static-web-app** como _Repositorio_.

1. En _Rama_, seleccione **maestra**.

   :::image type="content" source="./media/publish-gatsby/completed-github-info.png" alt-text="Información de GitHub completada":::

### <a name="build"></a>Build

A continuación, agregue los valores de configuración que el proceso de compilación usa para compilar la aplicación.

1. Haga clic en el botón **Siguiente: Compilar >** para editar la configuración de compilación.

1. Para configurar las opciones del paso en las Acciones de GitHub, establezca la _Ubicación de la aplicación_ en **/** .

1. Establezca _Ubicación del artefacto de la aplicación_ en **public**.

   No es necesario un valor para _Ubicación de la API_, ya que por el momento no está implementando una API.

   :::image type="content" source="./media/publish-gatsby/build-details.png" alt-text="Configuración de la compilación":::

### <a name="review-and-create"></a>Revisar y crear

1. Haga clic en el botón **Revisar y crear** para comprobar que todos los detalles sean correctos.

1. Haga clic en **Crear** para comenzar la creación de la aplicación de Static Web Apps de App Service y aprovisionar una Acción de GitHub para la implementación.

1. Cuando se complete la implementación, haga clic en **Ir al recurso**.

1. En la pantalla del recurso, haga clic en el vínculo _Dirección URL_ para abrir la aplicación implementada. Es posible que tenga que esperar uno o dos minutos para que se complete la Acción de GitHub.

   :::image type="content" source="./media/publish-gatsby/deployed-app.png" alt-text="Aplicación implementada":::

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cleanup-resource](../../includes/static-web-apps-cleanup-resource.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Agregar un dominio personalizado](custom-domain.md)
