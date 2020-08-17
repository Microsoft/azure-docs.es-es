---
title: Procedimiento para actualizar las herramientas de Azure Dev Spaces
services: azure-dev-spaces
ms.date: 07/03/2018
ms.topic: conceptual
ms.custom: devx-track-azurecli
description: Obtenga información sobre cómo actualizar las herramientas de línea de comandos de Azure Dev Spaces, la extensión de Visual Studio Code y la extensión de Visual Studio.
keywords: Docker, Kubernetes, Azure, AKS, Azure Container Service, contenedores
ms.openlocfilehash: aebe81d58a1bcd44f5766d368dbafa53c81ed2b8
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87504337"
---
# <a name="how-to-upgrade-azure-dev-spaces-tools"></a>Procedimiento para actualizar las herramientas de Azure Dev Spaces

Si hay una nueva versión y ya está usando Azure Dev Spaces, es posible que deba actualizar sus herramientas de cliente.

## <a name="update-the-azure-cli"></a>Actualización de la CLI de Azure

Al actualizar a la CLI de Azure más reciente, también obtendrá la versión más reciente de la extensión de la CLI de Dev Spaces.

No es necesario desinstalar la versión anterior, basta con buscar la descarga correspondiente en la [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest).


## <a name="update-the-dev-spaces-cli-extension-and-command-line-tools"></a>Actualización de la extensión de la CLI de Dev Spaces y las herramientas de línea de comandos

Ejecute el siguiente comando:

```azurecli
az aks use-dev-spaces -n <your-aks-cluster> -g <your-aks-cluster-resource-group> --update
```

## <a name="update-the-vs-code-extension"></a>Actualización de la extensión de VS Code

Una vez instalada, la extensión se actualiza automáticamente. Es posible que deba volver a cargar la extensión para usar las nuevas características. En Visual Studio Code, abra el panel **Extensiones**, elija las extensiones de **Azure Dev Spaces** y elija **Volver a cargar**.

## <a name="update-visual-studio"></a>Actualización de Visual Studio

Azure Dev Spaces forma parte de la carga de trabajo de desarrollo de Azure y se incluye en todas las actualizaciones de Visual Studio.

## <a name="next-steps"></a>Pasos siguientes

Pruebe las nuevas herramientas creando un clúster. Pruebe las guías de inicio rápido y los tutoriales en [Azure Dev Spaces](../index.yml).
