---
title: 'Script de la CLI: Escalado de un servidor de Azure Database for MySQL'
description: Este script de la CLI de ejemplo escala el servidor de Azure Database for MySQL (Base de datos de Azure para MySQL) a un nivel de rendimiento diferente después de consultar las métricas.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.devlang: azurecli
ms.topic: sample
ms.custom: mvc, devx-track-azurecli
ms.date: 12/02/2019
ms.openlocfilehash: 318e41089f096f22d17faaa77863fe6af54a3553
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87483763"
---
# <a name="monitor-and-scale-an-azure-database-for-mysql-server-using-azure-cli"></a>Supervisión y escalado de un servidor de Azure Database for MySQL (Base de datos de Azure para MySQL) mediante la CLI de Azure
Este script de la CLI de ejemplo escala el proceso y el almacenamiento de un servidor individual de Azure Database for MySQL después de consultar las métricas. La capacidad de procesamiento se puede ampliar o reducir. El almacenamiento solo se puede ampliar.

[!INCLUDE [cloud-shell-try-it](../../../includes/cloud-shell-try-it.md)]

Si decide ejecutar la CLI localmente, este artículo necesita la CLI de Azure versión 2.0 o posterior. Ejecute `az --version` para comprobar la versión. Consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli) para instalar o actualizar su versión de la CLI de Azure. 

## <a name="sample-script"></a>Script de ejemplo
Actualice el script con el identificador de su suscripción.
[!code-azurecli-interactive[main](../../../cli_scripts/mysql/scale-mysql-server/scale-mysql-server.sh "Create and scale Azure Database for MySQL.")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación
Use el comando siguiente para quitar el grupo de recursos y todos los recursos asociados a él después de ejecutarse el script de ejemplo. 
[!code-azurecli-interactive[main](../../../cli_scripts/mysql/scale-mysql-server/delete-mysql.sh  "Delete the resource group.")]

## <a name="script-explanation"></a>Explicación del script
Este script usa los comandos que se describen en la tabla siguiente:

| **Comando** | **Notas** |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az mysql server create](/cli/azure/mysql/server#az-mysql-server-create) | Crea un servidor MySQL que hospeda las bases de datos. |
| [az mysql server update](/cli/azure/mysql/server#az-mysql-server-update) | Actualiza las propiedades del servidor MySQL. |
| [az monitor metrics list](/cli/azure/monitor/metrics#az-monitor-metrics-list) | Especifica el valor de métrica de los recursos. |
| [az group delete](/cli/azure/group#az-group-delete) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes
- Más información sobre el [proceso y el almacenamiento de Azure Database for MySQL](../concepts-pricing-tiers.md)
- Pruebe scripts adicionales en: [Ejemplos de la CLI de Azure para Azure Database for MySQL](../sample-scripts-azure-cli.md).
- Más información sobre la [CLI de Azure](/cli/azure)
