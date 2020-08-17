---
title: 'Creación de una zona y un registro DNS para un nombre de dominio: CLI de Azure (Azure DNS)'
description: En este ejemplo de script de CLI de Azure se muestra cómo crear una zona DNS y un registro para un nombre de dominio
services: dns
author: rohinkoul
ms.service: dns
ms.topic: sample
ms.date: 09/20/2019
ms.author: rohink
ms.custom: devx-track-azurecli
ms.openlocfilehash: 1b0f08132ace5de7108702b61e89517fd28b4673
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87494917"
---
# <a name="azure-cli-script-example-create-a-dns-zone-and-record"></a>Ejemplo de script de CLI de Azure: crear una zona DNS y un registro

En este ejemplo de script de CLI de Azure se crea una zona DNS y un registro para un nombre de dominio. 

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/dns/create-dns-zone-and-record/create-dns-zone-and-record.sh "Create DNS zone and record")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación 

Ejecute el siguiente comando para quitar el grupo de recursos, la zona DNS y todos los recursos relacionados.

```azurecli
az group delete -n myResourceGroup
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un grupo de recursos, una máquina virtual, un conjunto de disponibilidad, un equilibrador de carga y todos los recursos relacionados. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az network dns zone create](/cli/azure/network/dns/zone#az-network-dns-zone-create) | Crea una zona de Azure DNS. |
| [az network dns record-set a add-record](/cli/azure/network/dns/record-set) | Agrega un registro *A* a una zona DNS. |
| [az network dns record-set list](/cli/azure/network/dns/record-set) | Muestra todos los conjuntos de registros *A* en una zona DNS. |
| [az group delete](https://docs.microsoft.com/cli/azure/vm/extension#az-vm-extension-set) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](https://docs.microsoft.com/cli/azure).

