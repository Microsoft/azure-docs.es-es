---
title: Requisitos previos de la CLI de Azure para Azure HPC Cache
description: Pasos de configuración para usar la CLI de Azure para crear o modificar Azure HPC Cache
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 07/08/2020
ms.author: v-erkel
ms.openlocfilehash: 7d0844e699c012d987c23a75e2b0874005cf535a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095434"
---
# <a name="set-up-azure-cli-for-azure-hpc-cache"></a>Configuración de la CLI de Azure para Azure HPC Cache

Siga estos pasos para preparar el entorno antes de usar la CLI de Azure para crear o administrar la extensión Azure HPC Cache.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="install-azure-cli"></a>Instalación de la CLI de Azure

Azure HPC Cache requiere la versión 2.7 o posterior de la CLI de Azure. Ejecute `az --version` para buscar cuál es la versión y las bibliotecas dependientes que están instaladas. Para la instalación o la actualización, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

```azurecli-interactive
az --version
```

### <a name="install-the-azure-hpc-cache-extension"></a>Instalación de la extensión Azure HPC Cache

Las funciones de Azure HPC Cache no forman parte de la base de código principal, por lo que también debe instalar la referencia de la extensión. Siga las instrucciones que se describen a continuación.

1. Iniciar sesión

   Si usa una versión local de la CLI, inicie sesión con el comando [az login](/cli/azure/reference-index#az-login).

    ```azurecli
    az login
    ```

    Siga los pasos que se muestran en el terminal para completar el proceso de autenticación.

   > [!TIP]
   > Si tiene varias suscripciones, debe elegir una. Selecciónela al iniciar una sesión de Cloud Shell en Azure Portal o siga las instrucciones de [Introducción a la CLI de Azure](/cli/azure/get-started-with-azure-cli#sign-in) para establecer la suscripción desde la línea de comandos.

2. Instalación de la extensión de la CLI de Azure

   Las funciones de Azure HPC Cache se proporcionan como una extensión de la CLI de Azure, ya que aún no forma parte del paquete de la CLI principal. Debe instalar la referencia de la extensión para poder usarla.

   Las extensiones de la CLI de Azure proporcionan acceso a comandos experimentales y en versión preliminar. Para más información acerca de las extensiones, incluida la actualización y la desinstalación, consulte [Uso de extensiones con la CLI de Azure](/cli/azure/azure-cli-extensions-overview).

   Ejecute el siguiente comando para instalar la extensión Azure HPC Cache:

    ```azurecli-interactive
    az extension add --name hpc-cache
   ```

## <a name="set-default-resource-group-optional"></a>Configuración del grupo de recursos predeterminado (opcional)

La mayoría de los comandos hpc-cache requieren que se pase el grupo de recursos de la memoria caché. Puede establecer el grupo de recursos predeterminado mediante [az configure](/cli/azure/reference-index#az-configure).

## <a name="next-steps"></a>Pasos siguientes

Después de instalar la extensión de la CLI de Azure e iniciar sesión, puede usar la CLI de Azure para crear y administrar sistemas de Azure HPC Cache.

* [Creación de una instancia de Azure HPC Cache](hpc-cache-create.md)
* [Documentación de hpc-cache de la CLI de Azure](/cli/azure/ext/hpc-cache/hpc-cache)
