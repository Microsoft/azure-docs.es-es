---
title: 'Inicio rápido: creación de la primera aplicación web estática con Azure Static Web Apps mediante Azure Portal'
description: Aprenda a crear una instancia de Azure Static Web Apps con Azure Portal.
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: quickstart
ms.date: 09/03/2020
ms.author: cshoe
ms.openlocfilehash: 66335548381a6f7e3ebf93c97eb212255639644c
ms.sourcegitcommit: 80b9c8ef63cc75b226db5513ad81368b8ab28a28
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2020
ms.locfileid: "90604094"
---
# <a name="quickstart-building-your-first-static-web-app-in-the-azure-portal"></a>Inicio rápido: creación de la primera aplicación web en Azure Portal

Azure Static Web Apps publica sitios web en entornos de producción mediante la creación de aplicaciones desde un repositorio de GitHub. En este inicio rápido, se implementa una aplicación web en Azure Static Web Apps mediante Azure Portal.

Si no tiene ninguna suscripción a Azure, [cree una cuenta de evaluación gratuita](https://azure.microsoft.com/free).

## <a name="prerequisites"></a>Requisitos previos

- [GitHub](https://github.com)
- Cuenta de [Azure](https://portal.azure.com)

[!INCLUDE [create repository from template](../../includes/static-web-apps-get-started-create-repo.md)]

## <a name="create-a-static-web-app"></a>Creación de una aplicación web estática

Ahora que se ha creado el repositorio, puede crear una aplicación web estática desde Azure Portal.

1. Vaya a [Azure Portal](https://portal.azure.com).
1. Seleccione **Crear un recurso**
1. Busque **Static Web Apps**.
1. Seleccione **Static Web Apps (Preview)** (Static Web Apps [versión preliminar]).
1. Seleccione **Crear**

En la pestaña _Conceptos básicos_, configure la nueva aplicación y vincúlela a un repositorio de GitHub.

:::image type="content" source="media/getting-started-portal/basics-tab.png" alt-text="Pestaña Aspectos básicos":::

1. Seleccione su _suscripción a Azure_.
1. Seleccione o cree un nuevo _grupo de recursos_.
1. Asigne el nombre **my-first-static-web-app** a la aplicación.
      1. Los caracteres válidos son `a-z` (no distingue mayúsculas de minúsculas), `0-9` y `-`.
1. Seleccione la _región_ más cercana a la suya.
1. Seleccione la _SKU_ **gratis**.
1. Seleccione el botón **Sign-in with GitHub** (Iniciar sesión con GitHub) y autentíquese con GitHub

Tras iniciar sesión con GitHub, escriba la información del repositorio.

:::image type="content" source="media/getting-started-portal/repository-details.png" alt-text="Detalles del repositorio":::

1. Seleccione su _organización_ preferida.
1. Seleccione **my-first-web-static-app** en el menú desplegable _Repositorio_.
1. Seleccione **master** en el menú desplegable _Rama_.

> [!NOTE]
> Si no ve ningún repositorio, es posible que deba autorizar Azure Static Web Apps en GitHub. Vaya al repositorio de GitHub y, a continuación, a **Configuración > Aplicaciones > Aplicaciones de OAuth autorizadas**, seleccione **Azure Static Web Apps** y, después, **Conceder**. En el caso de los repositorios de la organización, debe ser propietario de la organización para conceder los permisos.

1. En la sección _Build Details_ (Detalles de la compilación), agregue los detalles de configuración específicos del marco de front-end que prefiera.

    # <a name="no-framework"></a>[Ningún marco](#tab/vanilla-javascript)

    1. Seleccione **Custom** (Personalizado) en la lista desplegable _Build Presets_ (Valores preestablecidos de compilación).
    1. Deje el valor predeterminado en el cuadro _App location_ (Ubicación de la aplicación).
    1. Borre el valor predeterminado del cuadro _Ubicación de la API_.
    1. Deje el cuadro _App artifact location_ (Ubicación del artefacto de la aplicación) en blanco.

    # <a name="angular"></a>[Angular](#tab/angular)

    1. Seleccione **Angular** en la lista desplegable _Build Presets_ (Valores preestablecidos de compilación).
    1. Deje el valor predeterminado en el cuadro _App location_ (Ubicación de la aplicación).
    1. Borre el valor predeterminado del cuadro _Ubicación de la API_.
    1. Escriba **dist/angular-basic** en el cuadro _App artifact location_ (Ubicación del artefacto de la aplicación).

    # <a name="react"></a>[React](#tab/react)

    1. Seleccione **React** en la lista desplegable _Build Presets_ (Valores preestablecidos de compilación).
    1. Deje el valor predeterminado en el cuadro _App location_ (Ubicación de la aplicación).
    1. Borre el valor predeterminado del cuadro _Ubicación de la API_.
    1. Escriba **build** en el cuadro _App artifact location_ (Ubicación del artefacto de la aplicación).

    # <a name="vue"></a>[Vue](#tab/vue)

    1. Seleccione **Vue.js** en la lista desplegable _Build Presets_ (Valores preestablecidos de compilación).
    1. Deje el valor predeterminado en el cuadro _App location_ (Ubicación de la aplicación).
    1. Borre el valor predeterminado del cuadro _Ubicación de la API_.
    1. Deje el valor predeterminado en el cuadro _App artifact location_ (Ubicación del artefacto de la aplicación).

    ---

1. Seleccione **Revisar + crear**.

    :::image type="content" source="media/getting-started-portal/review-create.png" alt-text="Botón Revisar y crear":::

    > [!NOTE]
    > Puede editar el [archivo de flujo de trabajo](github-actions-workflow.md) para cambiar estos valores después de crear la aplicación.

1. Seleccione **Crear**.

    :::image type="content" source="media/getting-started-portal/create-button.png" alt-text="Botón Crear":::

1. Seleccione **Ir al recurso**.

    :::image type="content" source="media/getting-started-portal/resource-button.png" alt-text="Botón Ir al recurso":::

[!INCLUDE [view website](../../includes/static-web-apps-get-started-view-website.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, puede eliminar la instancia de Azure Static Web Apps mediante los siguientes pasos:

1. Abra [Azure Portal](https://portal.azure.com).
1. Busque **my-first-web-static-app** en la barra de búsqueda superior.
1. Selección del nombre de la aplicación
1. Seleccione el botón **Eliminar**.
1. Seleccione **Sí** para confirmar la acción de eliminación.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Adición de una API](add-api.md)
