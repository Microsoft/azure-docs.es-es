---
title: 'Montaje de un recurso compartido de archivos en una aplicación de funciones de Python: CLI de Azure'
description: Cree una aplicación de funciones de Python sin servidor y montaría un recurso compartido de archivos existente mediante la CLI de Azure.
ms.topic: sample
ms.date: 03/01/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 63ab9ba3219dc600187e73bbf124d62d3f51317a
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87498215"
---
# <a name="mount-a-file-share-to-a-python-function-app-using-azure-cli"></a>Montaje de un recurso compartido de archivos en una aplicación de funciones de Python mediante la CLI de Azure

Este script de ejemplo de Azure Functions crea una aplicación de funciones y crea un recurso compartido en Azure Files. Luego monta el recurso compartido, con el fin de que sus funciones puedan acceder a los datos.  

>[!NOTE]
>La aplicación de funciones creada se ejecuta en la versión 3.7 de Python. Azure Functions también [admite las versiones 3.6 y 3.8 de Python](../functions-reference-python.md#python-version).

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para este artículo es preciso que ejecute la versión 2.0 o posterior de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli). Los ejemplos se escriben para la shell de Bash y se debe modificar para ejecutarse en un símbolo del sistema de Windows. 

## <a name="sample-script"></a>Script de ejemplo

Este script crea una aplicación de funciones de Azure con el [Plan de consumo](../functions-scale.md#consumption-plan).

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/functions-cli-mount-files-storage-linux/functions-cli-mount-files-storage-linux.sh "Create an Azure Function on a Consumption plan")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explicación del script

Cada comando de la tabla crea un vínculo a documentación específica del comando. Este script usa los siguientes comandos:

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az storage account create](/cli/azure/storage/account#az-storage-account-create) | Crea una cuenta de Azure Storage. |
| [az functionapp create](/cli/azure/functionapp#az-functionapp-create) | Crea una aplicación de función. |
| [az storage share create](/cli/azure/storage/share#az-storage-share-create) | Crea un recurso compartido de archivos de Azure Files en una cuenta de almacenamiento. | 
| [az storage directory create](/cli/azure/storage/directory#az-storage-directory-create) | Crea un directorio en el recurso compartido de archivos. |
| [az webapp config storage-account add](/cli/azure/webapp/config/storage-account#az-webapp-config-storage-account-add) | Monta el recurso compartido en la aplicación de funciones. |
| [az webapp config storage-account list](/cli/azure/webapp/config/storage-account#az-webapp-config-storage-account-list) | Muestra los recursos compartidos de archivos montados en la aplicación de funciones. | 

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Encontrará más ejemplos de scripts de CLI para Azure Functions en la [documentación de Azure Functions](../functions-cli-samples.md).
