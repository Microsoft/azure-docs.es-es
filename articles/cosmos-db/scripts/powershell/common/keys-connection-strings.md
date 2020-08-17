---
title: Script de PowerShell para las operaciones de obtención de clave y cadena de conexión para una cuenta de Azure Cosmos DB
description: 'Ejemplo de script de Azure PowerShell: operaciones de clave de cuenta y cadena de conexión para una cuenta de Azure Cosmos DB'
author: markjbrown
ms.service: cosmos-db
ms.topic: sample
ms.date: 03/18/2020
ms.author: mjbrown
ms.openlocfilehash: f6dd9d1290ea9d18fc6a5f18196585926b2ab91a
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87505217"
---
# <a name="connection-string-and-account-key-operations-for-an-azure-cosmos-db-account-using-powershell"></a>Operaciones de clave de cuenta y cadena de conexión para una cuenta de Azure Cosmos DB mediante PowerShell

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="sample-script"></a>Script de ejemplo

> [!NOTE]
> En este ejemplo se muestra el uso de una cuenta de SQL API. Para usar este ejemplo para otras API, copie las propiedades relacionadas y aplíquelas al script específico de la API

[!code-powershell[main](../../../../../powershell_scripts/cosmosdb/common/ps-account-keys-connection-strings.ps1 "Connection strings and account keys for Azure Cosmos account")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Después de ejecutar el script de ejemplo, se puede usar el comando siguiente para quitar el grupo de recursos y todos los recursos asociados.

```powershell
Remove-AzResourceGroup -ResourceGroupName "myResourceGroup"
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
|**Azure Cosmos DB**| |
| [Get-AzCosmosDBAccountKey](https://docs.microsoft.com/powershell/module/az.cosmosdb/get-azcosmosdbaccountkey) | Obtiene la cadena de conexión o la clave (lectura y escritura o solo lectura ) de una cuenta de Cosmos DB. |
| [New-AzCosmosDBAccountKey](https://docs.microsoft.com/powershell/module/az.cosmosdb/new-azcosmosdbaccountkey) | Vuelve a generar la clave especificada de una cuenta de Cosmos DB. |
|**Grupos de recursos de Azure**| |
| [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |
|||

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](https://docs.microsoft.com/powershell/).

Encontrará más ejemplos de scripts de PowerShell de Azure Cosmos DB en los [scripts de PowerShell de Azure Cosmos DB](../../../powershell-samples.md).
