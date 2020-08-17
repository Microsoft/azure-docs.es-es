---
title: Script de PowerShell para actualizar las regiones de una cuenta de Azure Cosmos
description: 'Ejemplo de script de Azure PowerShell: actualización de las regiones de una cuenta de Azure Cosmos'
author: markjbrown
ms.service: cosmos-db
ms.topic: sample
ms.date: 05/01/2020
ms.author: mjbrown
ms.openlocfilehash: c58f59a08b99c3ce9f69e0c6b0633ac8d0d9c00b
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87505206"
---
# <a name="update-an-azure-cosmos-accounts-regions-using-powershell"></a>Actualización de las regiones de una cuenta de Azure Cosmos mediante PowerShell

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="sample-script"></a>Script de ejemplo

> [!NOTE]
> No es posible modificar regiones y cambiar otras propiedades de una cuenta de Cosmos en la misma operación. Deben realizarse dos operaciones independientes.
> [!NOTE]
> En este ejemplo se muestra el uso de una cuenta de SQL (Core) API. Para usar este ejemplo para otras API, copie las propiedades relacionadas y aplíquelas a su script específico de la API.

[!code-powershell[main](../../../../../powershell_scripts/cosmosdb/common/ps-account-update-region.ps1 "Update Azure Cosmos account regions")]

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
| [Get-AzCosmosDBAccount](https://docs.microsoft.com/powershell/module/az.cosmosdb/get-azcosmosdbaccount) | Enumera cuentas de Cosmos DB u obtiene una cuenta de Cosmos DB especificada. |
| [New-AzCosmosDBLocationObject](https://docs.microsoft.com/powershell/module/az.cosmosdb/new-azcosmosdblocationobject) | Crea un objeto de tipo PSLocation que se usará como parámetro para Update-AzCosmosDBAccountRegion. |
| [Update-AzCosmosDBAccountRegion](https://docs.microsoft.com/powershell/module/az.cosmosdb/update-azcosmosdbaccountregion) | Actualice las regiones de una cuenta de Cosmos DB. |
|**Grupos de recursos de Azure**| |
| [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |
|||

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](https://docs.microsoft.com/powershell/).

Encontrará más ejemplos de scripts de PowerShell de Azure Cosmos DB en los [scripts de PowerShell de Azure Cosmos DB](../../../powershell-samples.md).
