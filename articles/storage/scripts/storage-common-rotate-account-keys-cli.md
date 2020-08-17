---
title: 'Ejemplo de script de la CLI de Azure: rotar las claves de acceso de la cuenta de almacenamiento | Microsoft Docs'
description: Cree una cuenta de Azure Storage y luego recupere y rote sus claves de acceso de cuenta.
services: storage
author: tamram
ms.service: storage
ms.subservice: blobs
ms.devlang: cli
ms.topic: sample
ms.date: 06/22/2017
ms.author: tamram
ms.custom: devx-track-azurecli
ms.openlocfilehash: bfda6c5315e11a4a924b82dc3eacdf692357827b
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87503691"
---
# <a name="create-a-storage-account-and-rotate-its-account-access-keys"></a>Creación de una cuenta de almacenamiento y rotación de sus claves de acceso de cuenta

Este script crea una cuenta de Azure Storage, muestra las claves de acceso de la nueva cuenta de almacenamiento y, por último, renueva (rota) las claves.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/storage/rotate-storage-account-keys/rotate-storage-account-keys.sh "Rotate storage account keys")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Ejecute el siguiente comando para quitar el grupo de recursos, la cuenta de almacenamiento y todos los recursos relacionados.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear la cuenta de almacenamiento y recuperar y rotar sus claves de acceso. Cada elemento de la tabla incluye vínculos a la documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az storage account create](/cli/azure/storage/account) | Crea una cuenta de Azure Storage en el grupo de recursos especificado. |
| [az storage account keys list](/cli/azure/storage/account/keys) | Muestra las claves de acceso de la cuenta de almacenamiento para la cuenta especificada. |
| [az storage account keys renew](/cli/azure/storage/account/keys) | Vuelve a generar la clave de acceso de la cuenta de almacenamiento primaria o secundaria. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Puede encontrar ejemplos de script adicionales de la CLI de almacenamiento en los [ejemplos de la CLI de Azure para Azure Blob Storage](../blobs/storage-samples-blobs-cli.md).
