---
title: 'Ejemplo de script de la CLI de Azure: eliminación de contenedores por prefijo | Microsoft Docs'
description: Elimine los contenedores de blobs de Azure Storage en función de un prefijo de nombre de contenedor y, después, limpie la implementación. Vea los vínculos de ayuda para los comandos usados en el ejemplo de script.
services: storage
author: tamram
ms.service: storage
ms.subservice: blobs
ms.devlang: cli
ms.topic: sample
ms.date: 06/22/2017
ms.author: tamram
ms.custom: devx-track-azurecli
ms.openlocfilehash: aeccf255004cd4512fbc591942324341504b20f7
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87901902"
---
# <a name="use-an-azure-cli-script-to-delete-containers-based-on-container-name-prefix"></a>Uso de un script de la CLI de Azure para eliminar contenedores según el prefijo de nombre del contenedor

Este script primero crea algunos contenedores de ejemplo en Azure Blob Storage, luego elimina algunos de los contenedores según un prefijo del nombre de contenedor.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/storage/delete-containers-by-prefix/delete-containers-by-prefix.sh?highlight=2-3 "Delete containers by prefix")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Ejecute el siguiente comando para quitar el grupo de recursos, contenedores restantes y todos los recursos relacionados.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para eliminar contenedores según el prefijo del nombre de contenedor. Cada elemento de la tabla incluye vínculos a la documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az storage account create](/cli/azure/storage/account) | Crea una cuenta de Azure Storage en el grupo de recursos especificado. |
| [az storage container create](/cli/azure/storage/container) | Crea un contenedor en Azure Blob Storage. |
| [az storage container list](/cli/azure/storage/container) | Enumera los contenedores de una cuenta de Azure Storage. |
| [az storage container delete](/cli/azure/storage/container) | Elimina contenedores de una cuenta de Azure Storage. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Puede encontrar ejemplos de script adicionales de la CLI de almacenamiento en los [ejemplos de la CLI de Azure para Azure Storage](../blobs/storage-samples-blobs-cli.md).
