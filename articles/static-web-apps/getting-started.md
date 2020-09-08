---
title: 'Inicio rápido: creación de la primera aplicación web estática con Azure Static Web Apps'
description: Aprenda a crear un sitio web con Azure Static Web Apps.
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: quickstart
ms.date: 08/13/2020
ms.author: cshoe
ms.openlocfilehash: db3836e6171d187539b8615efcb5ab782c368020
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88752447"
---
# <a name="quickstart-building-your-first-static-web-app"></a>Inicio rápido: creación de la primera aplicación web estática

Azure Static Web Apps publica sitios web en entornos de producción mediante la creación de aplicaciones desde un repositorio de GitHub. En este inicio rápido, se implementa una aplicación web en Azure Static Web Apps mediante la extensión de Visual Studio Code.

Si no tiene ninguna suscripción a Azure, [cree una cuenta de evaluación gratuita](https://azure.microsoft.com/free).

## <a name="prerequisites"></a>Requisitos previos

- [GitHub](https://github.com)
- Cuenta de [Azure](https://portal.azure.com)
- [Visual Studio Code](https://code.visualstudio.com)
- [Extensión de Azure Static Web Apps para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurestaticwebapps)

[!INCLUDE [create repository from template](../../includes/static-web-apps-get-started-create-repo.md)]

[!INCLUDE [clone the repository](../../includes/static-web-apps-get-started-clone-repo.md)]

Después, abra Visual Studio Code y vaya a **Archivo > Abrir carpeta** para abrir el repositorio que acaba de clonar en la máquina en el editor.

## <a name="create-a-static-web-app"></a>Creación de una aplicación web estática

1. En Visual Studio Code, seleccione el logotipo de Azure en la barra de actividades para abrir la ventana extensiones de Azure.

    :::image type="content" source="media/getting-started/extension-azure-logo.png" alt-text="Logotipo de Azure":::

    > [!NOTE]
    > Es necesario iniciar sesión tanto en Azure como en GitHub. Si aún no ha iniciado sesión en Azure y GitHub desde Visual Studio Code, la extensión le solicitará que lo haga en ambos durante el proceso de creación.

1. Coloque el mouse sobre la etiqueta _Static Web Apps_ y seleccione el **signo más**.

    :::image type="content" source="media/getting-started/extension-create-button.png" alt-text="Nombre de la aplicación":::

1. La paleta de comandos se abre en la parte superior del editor y le pide que asigne un nombre a la aplicación.

    Escriba **my-first-static-web-app** y presione **Entrar**.

    :::image type="content" source="media/getting-started/extension-create-app.png" alt-text="Creación de la aplicación web estática":::

1. Seleccione la rama **master** y presione **Entrar**.

    :::image type="content" source="media/getting-started/extension-branch.png" alt-text="Nombre de la rama":::

1. Seleccione **/** como ubicación del código de la aplicación y presione **Entrar**.

    :::image type="content" source="media/getting-started/extension-app-location.png" alt-text="Ubicación del código de la aplicación":::

1. La extensión busca la ubicación de la API en la aplicación. En este artículo no se implementa ninguna API.

    Seleccione **Omitir por ahora** y presione **Entrar**.

    :::image type="content" source="media/getting-started/extension-api-location.png" alt-text="Ubicación de la API":::

1. Seleccione la ubicación en la que se compilan los archivos para producción en la aplicación.

    # <a name="no-framework"></a>[Ningún marco](#tab/vanilla-javascript)

    Desactive la casilla y presione **Entrar**.

    :::image type="content" source="media/getting-started/extension-artifact-no-framework.png" alt-text="Ruta de acceso de los archivos de la aplicación":::

    # <a name="angular"></a>[Angular](#tab/angular)

    Escriba **dist/angular-basic** y presione **Entrar**.

    :::image type="content" source="media/getting-started/extension-artifact-angular.png" alt-text="Ruta de acceso de los archivos de la aplicación Angular":::

    # <a name="react"></a>[React](#tab/react)

    Escriba **build** y presione **Entrar**.

    :::image type="content" source="media/getting-started/extension-artifact-react.png" alt-text="Ruta de acceso de los archivos de la aplicación React":::

    # <a name="vue"></a>[Vue](#tab/vue)

    Escriba **build** y presione **Entrar**.

    :::image type="content" source="media/getting-started/extension-artifact-vue.png" alt-text="Ruta de acceso de los archivos de la aplicación Vue":::

    ---

1. Seleccione la ubicación más cercana y presione **Entrar**.

    :::image type="content" source="media/getting-started/extension-location.png" alt-text="Ubicación de los recursos":::

1. Una vez creada la aplicación, se muestra una notificación de confirmación en Visual Studio Code.

    :::image type="content" source="media/getting-started/extension-confirmation.png" alt-text="Confirmación creada":::

1. En la ventana Explorador de Visual Studio Code, vuelva a la sección _Web Apps estática,_ haga clic con el botón derecho en **Producción** y seleccione **Abrir en el portal** para ver la aplicación en Azure Portal.

    :::image type="content" source="media/getting-started/extension-open-in-portal.png" alt-text="Abrir portal":::

[!INCLUDE [view website](../../includes/static-web-apps-get-started-view-website.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, puede eliminar la instancia de Azure Static Web Apps mediante la extensión.

En la ventana Explorador de Visual Studio Code, vuelva a la sección _Static Web Apps_ y haga clic con el botón derecho en **my-first-static-web-app** y seleccione **Eliminar**.

:::image type="content" source="media/getting-started/extension-delete.png" alt-text="Eliminar la aplicación":::

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Adición de una API](add-api.md)
