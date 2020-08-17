---
title: 'Tutorial: Publicación de un sitio de VuePress en Azure Static Web Apps'
description: En este tutorial se muestra cómo implementar una aplicación de VuePress en Azure Static Web Apps.
services: static-web-apps
author: aaronpowell
ms.service: static-web-apps
ms.topic: tutorial
ms.date: 05/08/2020
ms.author: aapowell
ms.custom: devx-track-javascript
ms.openlocfilehash: 6f5e05e9bfea6b14e46bbde8d17fba2d2a3b7b07
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87429495"
---
# <a name="tutorial-publish-a-vuepress-site-to-azure-static-web-apps-preview"></a>Tutorial: Publicación de un sitio de VuePress en Azure Static Web Apps (versión preliminar)

En este artículo se muestra cómo crear e implementar una aplicación web de [VuePress](https://vuepress.vuejs.org/) en [Azure Static Web Apps](overview.md). El resultado final es una nueva aplicación de Azure Static Web Apps, con las Acciones de GitHub asociadas, que le da control sobre cómo se compila y publica la aplicación.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> - Crear una aplicación de VuePress
> - Configurar Azure Static Web Apps
> - Implementar la aplicación de VuePress en Azure

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. En caso de no tener una, puede [crear una cuenta gratuita](https://azure.microsoft.com/free/).
- Una cuenta de GitHub. En caso de no tener una, puede [crear una cuenta gratuita](https://github.com/join).
- [Node.js](https://nodejs.org) instalado.

## <a name="create-a-vuepress-app"></a>Creación de una aplicación de VuePress

Cree una aplicación de VuePress desde la interfaz de la línea de comandos (CLI):

1. Cree una carpeta para la aplicación de VuePress.

   ```bash
   mkdir static-site
   ```

1. Agregue un archivo _README.md_ a la carpeta.

   ```bash
   echo '# Hello From VuePress' > README.md
   ```

1. Inicialice el archivo _package.json_.

   ```bash
   npm init -y
   ```

1. Agregue VuePress como `devDependency`.

   ```bash
   npm install --save-dev vuepress
   ```

1. Abra el archivo _package.json_ en un editor de texto y agregue un comando de compilación a la sección [`scripts`](https://docs.npmjs.com/cli-commands/run-script.html).

   ```json
   ...
   "scripts": {
       "build": "vuepress build"
   }
   ...
   ```

1. Cree un archivo _.gitignore_ para excluir la carpeta _node\_modules_.

    ```bash
    echo 'node_modules' > .gitignore
    ```

1. Inicialice un repositorio GIT.

   ```bash
    git init
    git add -A
    git commit -m "initial commit"
   ```

## <a name="push-your-application-to-github"></a>Inserción de la aplicación en GitHub

Necesita un repositorio en GitHub para conectarse a Azure Static Web Apps. En los pasos siguientes se muestra cómo crear un repositorio para el sitio.

1. Cree un repositorio de GitHub en blanco (no cree un archivo Léame) desde [https://github.com/new](https://github.com/new), denominado **vuepress-static-app**.

1. Agregue el repositorio de GitHub como remoto al repositorio local. Asegúrese de agregar el nombre de usuario de GitHub en lugar del marcador de posición `<YOUR_USER_NAME>` en el comando siguiente.

   ```bash
   git remote add origin https://github.com/<YOUR_USER_NAME>/vuepress-static-app
   ```

1. Inserte el repositorio local en GitHub.

   ```bash
   git push --set-upstream origin master
   ```

## <a name="deploy-your-web-app"></a>Implementación de la aplicación web

En los pasos siguientes se muestra cómo crear una aplicación de Static Web Apps e implementarla en un entorno de producción.

### <a name="create-the-application"></a>Creación de la aplicación

1. Vaya a [Azure Portal](https://portal.azure.com).
1. Haga clic en **Crear un recurso**.
1. Busque **Static Web Apps**.
1. Haga clic en **Static Web Apps (Preview)** (Static Web Apps [versión preliminar]).
1. Haga clic en **Crear**

   :::image type="content" source="./media/publish-vuepress/create-in-portal.png" alt-text="Creación de una aplicación de Static Web Apps (versión preliminar) en el portal":::

1. En **Suscripción**, acepte la suscripción que aparece o seleccione otra en la lista desplegable.

1. En _Grupo de recursos_, seleccione **Nuevo**. En _Nuevo nombre de grupo de recursos_, escriba **vuepress-static-app** y seleccione **Aceptar**.

1. Después, un nombre para la aplicación en el cuadro **Nombre**. Los caracteres válidos incluyen `a-z`, `A-Z`, `0-9` y `-`.

1. En _Región_, seleccione una región cercana disponible.

1. En _SKU_, seleccione **Gratis**.

   :::image type="content" source="./media/publish-vuepress/basic-app-details.png" alt-text="Detalles rellenados":::

1. Haga clic en el botón **Iniciar sesión con GitHub**.

1. Seleccione la **Organización** en la que creó el repositorio.

1. Seleccione **vuepress-static-app** como _Repositorio_.

1. En _Rama_, seleccione **maestra**.

   :::image type="content" source="./media/publish-vuepress/completed-github-info.png" alt-text="Información de GitHub completada":::

### <a name="build"></a>Build

A continuación, agregue los valores de configuración que el proceso de compilación usa para compilar la aplicación. La configuración siguiente define el archivo de flujo de trabajo de la Acción de GitHub.

1. Haga clic en el botón **Siguiente: Compilar >** para editar la configuración de compilación.

1. Establezca _Ubicación de la aplicación_ en **/**

1. Establezca _Ubicación del artefacto de la aplicación_ en **.vuepress/dist**.

No es necesario un valor para _Ubicación de la API_, ya que por el momento no está implementando una API.

   :::image type="content" source="./media/publish-vuepress/build-details.png" alt-text="Configuración de la compilación":::

### <a name="review-and-create"></a>Revisar y crear

1. Haga clic en el botón **Revisar y crear** para comprobar que todos los detalles sean correctos.

1. Haga clic en **Crear** para comenzar la creación de la aplicación de Azure Static Web Apps y aprovisionar una Acción de GitHub para la implementación.

1. Cuando se complete la implementación, haga clic en **Ir al recurso**.

1. En la pantalla del recurso, haga clic en el vínculo _Dirección URL_ para abrir la aplicación implementada. Es posible que tenga que esperar uno o dos minutos para que se complete la Acción de GitHub.

   :::image type="content" source="./media/publish-vuepress/deployed-app.png" alt-text="Aplicación implementada":::

### <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cleanup-resource](../../includes/static-web-apps-cleanup-resource.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Agregar un dominio personalizado](custom-domain.md)
