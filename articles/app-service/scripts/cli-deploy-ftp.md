---
title: 'CLI: Implementación de archivos de aplicación con FTP'
description: Aprenda a usar la CLI de Azure para automatizar la implementación y administración de la aplicación App Service. En este ejemplo se indica cómo crear una aplicación e implementar archivos con FTP.
author: msangapu-msft
tags: azure-service-management
ms.devlang: azurecli
ms.topic: sample
ms.date: 12/12/2017
ms.author: msangapu
ms.custom: mvc, seodec18, devx-track-azurecli
ms.openlocfilehash: b4575c3227eefbb2a6bad3a8ba3c60bb9195b248
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87495586"
---
# <a name="create-an-app-service-app-and-deploy-files-with-ftp-using-azure-cli"></a>Creación de una aplicación de App Service e implementación de archivos con FTP mediante la CLI de Azure

Este script de ejemplo crea una aplicación en App Service con sus recursos relacionados y, después, implementa una página HTML estática mediante FTP. Para la carga de FTP, el script usa [cURL](https://en.wikipedia.org/wiki/CURL) como ejemplo. Puede usar cualquier herramienta FTP para cargar los archivos.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, necesitará la CLI de Azure versión 2.0 o posterior. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/app-service/deploy-ftp/deploy-ftp.sh "Create an app and deploy files with FTP")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explicación del script 

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [`az group create`](/cli/azure/group?view=azure-cli-latest#az-group-create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [`az appservice plan create`](/cli/azure/appservice/plan?view=azure-cli-latest#az-appservice-plan-create) | Crea un plan de App Service, |
| [`az webapp create`](/cli/azure/webapp?view=azure-cli-latest#az-webapp-create) | Crea una aplicación de App Service. |
| [`az webapp deployment list-publishing-profiles`](/cli/azure/webapp/deployment?view=azure-cli-latest#az-webapp-deployment-list-publishing-profiles) | Obtiene los detalles de los perfiles de implementación de aplicación disponibles. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](https://docs.microsoft.com/cli/azure).

Puede encontrar ejemplos de script adicionales de la CLI de App Service en la [documentación de Azure App Service](../samples-cli.md).
