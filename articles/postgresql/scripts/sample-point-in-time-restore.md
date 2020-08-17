---
title: 'Script de la CLI de Azure: restauración de un servidor de Azure Database for PostgreSQL'
description: En este script de la CLI de Azure de ejemplo se muestra cómo restaurar un servidor de Azure Database for PostgreSQL y sus bases de datos a un momento anterior.
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.devlang: azurecli
ms.topic: sample
ms.custom: mvc, devx-track-azurecli
ms.date: 02/28/2018
ms.openlocfilehash: 7b02b81e650eabea6f3f5f09347dc4aa2382671a
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87496515"
---
# <a name="restore-an-azure-database-for-postgresql-server-using-azure-cli"></a>Restauración de un servidor de Azure Database for PostgreSQL mediante la CLI de Azure
Este script de la CLI de ejemplo restaura un único servidor de Azure Database for PostgreSQL a un momento anterior.

[!INCLUDE [cloud-shell-try-it](../../../includes/cloud-shell-try-it.md)]

Si decide ejecutar la CLI localmente, este artículo necesita la CLI de Azure versión 2.0 o posterior. Ejecute `az --version` para comprobar la versión. Consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli) para instalar o actualizar su versión de la CLI de Azure.

## <a name="sample-script"></a>Script de ejemplo
En este script de ejemplo, va a modificar las líneas resaltadas para actualizar el nombre de usuario administrador y la contraseña a los suyos propios. Reemplace el identificador de suscripción que se usa en los comandos `az monitor` por su propio identificador de suscripción.
[!code-azurecli-interactive[main](../../../cli_scripts/postgresql/backup-restore/backup-restore.sh?highlight=15-16 "Restore Azure Database for PostgreSQL.")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación
Use el comando siguiente para quitar el grupo de recursos y todos los recursos asociados a él después de ejecutarse el script de ejemplo. 
[!code-azurecli-interactive[main](../../../cli_scripts/postgresql/backup-restore/delete-postgresql.sh  "Delete the resource group.")]

## <a name="script-explanation"></a>Explicación del script
Este script usa los comandos que se describen en la tabla siguiente:

| **Comando** | **Notas** |
|---|---|
| [az group create](/cli/azure/group) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az postgresql server create](/cli/azure/postgres/server#az-postgres-server-create) | Crea un servidor de PostgreSQL que hospeda las bases de datos. |
| [az postgresql server restore](/cli/azure/postgres/server#az-postgres-server-restore) | Restaura un servidor desde una copia de seguridad. |
| [az group delete](/cli/azure/group) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes
- Para más información sobre la CLI de Azure: [Documentación de la CLI de Azure](/cli/azure).
- Pruebe otros scripts adicionales: [Azure CLI samples for Azure Database for PostgreSQL](../sample-scripts-azure-cli.md) (Ejemplos de la CLI de Azure para Azure Database for PostgreSQL)
- [Copia de seguridad y restauración de un servidor en Azure Database for PostgreSQL mediante Azure Portal](../howto-restore-server-portal.md)
