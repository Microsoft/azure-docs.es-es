---
title: 'Ejemplo de script de la CLI de Azure: importación en un almacén de App Configuration'
titleSuffix: Azure App Configuration
description: Uso de un script de la CLI de Azure para importar una configuración a Azure App Configuration
services: azure-app-configuration
author: lisaguthrie
ms.service: azure-app-configuration
ms.devlang: azurecli
ms.topic: sample
ms.date: 02/19/2020
ms.author: lcozzens
ms.custom: devx-track-azurecli
ms.openlocfilehash: fe4c22042c31d55575320d9c3823783902700f9f
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87494781"
---
# <a name="import-to-an-azure-app-configuration-store"></a>Importación en un almacén de Azure App Configuration

Este script de ejemplo importa pares clave-valor en un almacén de Azure App Configuration.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI de Azure localmente, para este artículo es preciso que ejecute la versión 2.0 o posterior de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Para la instalación o la actualización, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="sample-script"></a>Script de ejemplo

```azurecli-interactive
#!/bin/bash

# Import key-values from a file
az appconfig kv import --name myTestAppConfigStore --source file --path ~/Import.json
```

[!INCLUDE [cli-script-cleanup](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para importar en un almacén de App Configuration. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az appconfig kv import](/cli/azure/appconfig/kv#az-appconfig-kv-import) | Importa en un recurso del almacén de App Configuration. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Puede encontrar ejemplos de script adicionales de la CLI y App Configuration en los [ejemplos de la CLI y Azure App Configuration](../cli-samples.md).
