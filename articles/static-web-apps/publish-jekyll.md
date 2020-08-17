---
title: 'Tutorial: Publicación de un sitio de Jekyll en Azure Static Web Apps'
description: Aprenda a implementar una aplicación de Jekyll en Azure Static Web Apps.
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: tutorial
ms.date: 06/08/2020
ms.author: cshoe
ms.openlocfilehash: c4211f21d78ac0e06743c97f1081804fd641f9d2
ms.sourcegitcommit: 97a0d868b9d36072ec5e872b3c77fa33b9ce7194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87563582"
---
# <a name="tutorial-publish-a-jekyll-site-to-azure-static-web-apps-preview"></a>Tutorial: Publicación de un sitio de Jekyll en Azure Static Web Apps, versión preliminar

En este artículo se muestra cómo crear e implementar una aplicación web de [Jekyll](https://jekyllrb.com/) en [Azure Static Web Apps](overview.md).

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> - Crear un sitio web de Jekyll.
> - Configurar Azure Static Web Apps.
> - Implementar la aplicación de Jekyll en Azure.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

- Instalar [Jekyll](https://jekyllrb.com/docs/installation/)
  - Puede usar el subsistema de Windows para Linux y seguir las instrucciones de Ubuntu si fuera necesario.
- Una cuenta de Azure con una suscripción activa. En caso de no tener una, puede [crear una cuenta gratuita](https://azure.microsoft.com/free/).
- Una cuenta de GitHub. En caso de no tener una, puede [crear una cuenta gratuita](https://github.com/join).

## <a name="create-jekyll-app"></a>Creación de una aplicación de Jekyll

Cree una aplicación de Jekyll mediante la interfaz de la línea de comandos (CLI) de Jekyll:

1. En el terminal, ejecute Jekyll CLI para crear una aplicación.

   ```bash
   jekyll new static-app
   ```

1. Vaya a la aplicación recién creada.

   ```bash
   cd static-app
   ```

1. Inicialice un nuevo repositorio de Git.

   ```bash
    git init
   ```

1. Confirme los cambios.

   ```bash
   git add -A
   git commit -m "initial commit"
   ```

## <a name="push-your-application-to-github"></a>Inserción de la aplicación en GitHub

Azure Static Web Apps usa GitHub para publicar su sitio web. En los pasos siguientes se muestra cómo crear un repositorio de GitHub.

1. Cree un repositorio de GitHub en blanco (no cree un archivo Léame) desde [https://github.com/new](https://github.com/new), denominado **jekyll-azure-static**.

1. Agregue el repositorio de GitHub como remoto al repositorio local. Asegúrese de agregar el nombre de usuario de GitHub en lugar del marcador de posición `<YOUR_USER_NAME>` en el comando siguiente.

   ```bash
   git remote add origin https://github.com/<YOUR_USER_NAME>/jekyll-static-app
   ```

1. Inserte el repositorio local en GitHub.

   ```bash
   git push --set-upstream origin master
   ```

## <a name="deploy-your-web-app"></a>Implementación de la aplicación web

En los pasos siguientes se muestra cómo crear una aplicación de sitio estática e implementarla en un entorno de producción.

### <a name="create-the-application"></a>Creación de la aplicación

1. Acceda a [Azure Portal](https://portal.azure.com).

1. Haga clic en **Crear un recurso**.

1. Busque **Static Web Apps**.

1. Haga clic en **Static Web App (Preview)** (Static Web Apps [versión preliminar]).

1. Haga clic en **Crear**.

1. En **Suscripción**, acepte la suscripción que aparece o seleccione otra en la lista desplegable.

1. En _Grupo de recursos_, seleccione **Nuevo**. En _Nuevo nombre de grupo de recursos_, escriba **jekyll-static-app** y seleccione **Aceptar**.

1. Después, especifique un nombre para la aplicación en el cuadro _Nombre_. Los caracteres válidos incluyen `a-z`, `A-Z`, `0-9` y `-`.

1. En _Región_, seleccione una región cercana disponible.

1. En _SKU_, seleccione **Gratis**.

    :::image type="content" source="./media/publish-jekyll/basic-app-details.png" alt-text="Detalles rellenados":::

1. Haga clic en el botón **Iniciar sesión con GitHub**.

1. Seleccione la **Organización** en la que creó el repositorio.

1. Seleccione **jekyll-static-app** como _Repositorio_.

1. En _Rama_, seleccione **maestra**.

    :::image type="content" source="./media/publish-jekyll/completed-github-info.png" alt-text="Información de GitHub completada":::

### <a name="build"></a>Build

A continuación, agregue los valores de configuración que el proceso de compilación usa para compilar la aplicación. La configuración siguiente define el archivo de flujo de trabajo de la Acción de GitHub.

1. Haga clic en el botón **Siguiente: Compilar >** para editar la configuración de compilación.

1. Establezca _Ubicación de la aplicación_ en **/_site**.

1. Deje _Ubicación del artefacto de la aplicación_ en blanco.

   No es necesario un valor para _Ubicación de la API_, ya que por el momento no está implementando una API.

### <a name="review-and-create"></a>Revisar y crear

1. Haga clic en el botón **Revisar y crear** para comprobar que todos los detalles sean correctos.

1. Haga clic en **Crear** para comenzar la creación de la aplicación de Azure Static Web Apps y aprovisionar una Acción de GitHub para la implementación.

1. En primer lugar, la implementación generará un error porque el archivo de flujo de trabajo necesita algunos valores específicos de Jekyll. Para agregarlos, vaya a su terminal y extraiga la confirmación con la acción de GitHub en la máquina.

   ```bash
   git pull
   ```

1. Abra la aplicación de Jekyll en un editor de texto y abra el archivo _.github/workflows/azure-pages-<NOMBRE_DE_FLUJO_DE_TRABAJO>.yml_.

1. Agregue varias líneas después del bloque `- uses: actions/checkout@v2` en el siguiente bloque de configuración.

    ```yml
    - uses: actions/checkout@v2
      with:
        submodules: true
    - name: Set up Ruby
      uses: ruby/setup-ruby@ec106b438a1ff6ff109590de34ddc62c540232e0
      with:
        ruby-version: 2.6
    - name: Install dependencies
      run: bundle install
    - name: Jekyll build
      run: jekyll build
    ```

1. Confirme el flujo de trabajo actualizado e inserte en GitHub.

    ```bash
    git add -A
    git commit -m "Updating GitHub Actions workflow"
    git push
    ```

1. Espere a que se complete la Acción de GitHub.

1. En la ventana _Información general_ de Azure Portal, haga clic en el vínculo _Dirección URL_ para abrir la aplicación implementada.

   :::image type="content" source="./media/publish-jekyll/deployed-app.png" alt-text="Aplicación implementada":::

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cleanup-resource](../../includes/static-web-apps-cleanup-resource.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Agregar un dominio personalizado](custom-domain.md)
