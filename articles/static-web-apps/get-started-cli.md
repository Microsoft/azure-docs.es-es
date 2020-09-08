---
title: 'Inicio rápido: creación de la primera aplicación web estática con Azure Static Web Apps mediante la CLI de Azure'
description: Aprenda a crear una instancia de Azure Static Web Apps con la CLI de Azure Static Web Apps.
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: quickstart
ms.date: 08/13/2020
ms.author: cshoe
ms.openlocfilehash: 7e0fdbc50dd36e4ea23903a5929735c1c83bd394
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88752874"
---
# <a name="quickstart-building-your-first-static-web-app-using-the-azure-cli"></a>Inicio rápido: creación de la primera aplicación web estática con la CLI de Azure

Azure Static Web Apps publica sitios web en entornos de producción mediante la creación de aplicaciones desde un repositorio de GitHub. En este inicio rápido, se implementa una aplicación web en Azure Static Web Apps mediante la CLI de Azure.

Si no tiene ninguna suscripción a Azure, [cree una cuenta de evaluación gratuita](https://azure.microsoft.com/free).

## <a name="prerequisites"></a>Requisitos previos

- [GitHub](https://github.com)
- [Token de acceso personal de GitHub](https://docs.github.com/github/authenticating-to-github/creating-a-personal-access-token)
- Cuenta de [Azure](https://portal.azure.com)
- [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) instalada (versión 2.8.0 y superiores)

[!INCLUDE [create repository from template](../../includes/static-web-apps-get-started-create-repo.md)]

[!INCLUDE [clone the repository](../../includes/static-web-apps-get-started-clone-repo.md)]

A continuación, use el siguiente comando para cambiar a la nueva carpeta.

```bash
cd my-first-static-web-app
```

## <a name="create-a-static-web-app"></a>Creación de una aplicación web estática

Ahora que se ha creado el repositorio, puede crear una aplicación web estática desde la CLI de Azure.

> [!IMPORTANT]
> Asegúrese de que se encuentre en la carpeta _my-first-static-web-app_ de su terminal.

1. Inicie sesión en la CLI de Azure mediante el siguiente comando.

    ```bash
    az login
    ```

1. Cree una aplicación web estática desde el repositorio.

    # <a name="no-framework"></a>[Ningún marco](#tab/vanilla-javascript)

    ```bash
    az staticwebapp create \
        -n my-first-static-web-app \
        -g <RESOURCE_GROUP_NAME> \
        -s https://github.com/<YOUR_GITHUB_ACCOUNT_NAME>/my-first-static-web-app \
        -l <LOCATION> \
        -b master \
        --token <YOUR_GITHUB_PERSONAL_ACCESS_TOKEN>
    ```

    # <a name="angular"></a>[Angular](#tab/angular)

    ```bash
    az staticwebapp create \
        -n my-first-static-web-app \
        -g <RESOURCE_GROUP_NAME> \
        -s https://github.com/<YOUR_GITHUB_ACCOUNT_NAME>/my-first-static-web-app \
        -l <LOCATION> \
        -b master \
        --app-artifact-location "dist/angular-basic" \
        --token <YOUR_GITHUB_PERSONAL_ACCESS_TOKEN>
    ```

    # <a name="react"></a>[React](#tab/react)

    ```bash
    az staticwebapp create \
        -n my-first-static-web-app \
        -g <RESOURCE_GROUP_NAME> \
        -s https://github.com/<YOUR_GITHUB_ACCOUNT_NAME>/my-first-static-web-app \
        -l <LOCATION> \
        -b master \
        --app-artifact-location "build" \
        --token <YOUR_GITHUB_PERSONAL_ACCESS_TOKEN>
    ```

    # <a name="vue"></a>[Vue](#tab/vue)

    ```bash
    az staticwebapp create \
        -n my-first-static-web-app \
        -g <RESOURCE_GROUP_NAME> \
        -s https://github.com/<YOUR_GITHUB_ACCOUNT_NAME>/my-first-static-web-app \
        -l <LOCATION> \
        -b master \
        --app-artifact-location "dist" \
        --token <YOUR_GITHUB_PERSONAL_ACCESS_TOKEN>
    ```

    ---

    - `<RESOURCE_GROUP_NAME>`: Reemplace este valor por un nombre de grupo de recursos de Azure existente.

    - `<YOUR_GITHUB_ACCOUNT_NAME>`: Reemplace este valor por el nombre de usuario de GitHub.

    - `<LOCATION>`: Reemplace este valor por la ubicación más cercana. Las opciones incluyen: _CentralUS_, _EastAsia_, _EastUS2_, _WestEurope_ y _WestUS2_.

    - `<YOUR_GITHUB_PERSONAL_ACCESS_TOKEN>`: Reemplace este valor por el [token de acceso personal de GitHub](https://docs.github.com/github/authenticating-to-github/creating-a-personal-access-token) que ha generado anteriormente.

    Ya puede ver la aplicación creada en Azure.

1. Abra [Azure Portal](https://portal.azure.com).

1. Busque **my-first-web-static-app** en la barra de búsqueda superior.

1. Seleccione **my-first-web-static-app**.

[!INCLUDE [view website](../../includes/static-web-apps-get-started-view-website.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, puede para eliminar la instancia de Azure Static Web Apps ejecute el siguiente comando:

```bash
az staticwebapp delete \
    --name my-first-static-web-app
    --resource-group my-first-static-web-app
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Adición de una API](add-api.md)
