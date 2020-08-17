---
title: 'Script de la CLI de Azure: cambio de configuraciones del servidor (PostgreSQL)'
description: Este script de la CLI de ejemplo enumera todas las opciones de configuración del servidor disponibles y actualiza el valor de una de las opciones.
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.devlang: azurecli
ms.topic: sample
ms.custom: mvc, devx-track-azurecli
ms.date: 02/28/2018
ms.openlocfilehash: 097b7692c5088a0860b8778fe8f315a84cf6b147
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87496566"
---
# <a name="list-and-update-configurations-of-an-azure-database-for-postgresql-server-using-azure-cli"></a>Enumeración y actualización de configuraciones de un servidor de Azure Database for PostgreSQL mediante la CLI de Azure
Este script de la CLI de ejemplo enumera todos los parámetros de configuración disponibles, así como sus valores permitidos para el servidor de Azure Database for PostgreSQL y establece *log_retention_days* en un valor distinto del predeterminado.

[!INCLUDE [cloud-shell-try-it](../../../includes/cloud-shell-try-it.md)]

Si decide ejecutar la CLI localmente, este artículo necesita la CLI de Azure versión 2.0 o posterior. Ejecute `az --version` para comprobar la versión. Consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli) para instalar o actualizar su versión de la CLI de Azure. 

## <a name="sample-script"></a>Script de ejemplo
En este script de ejemplo, va a modificar las líneas resaltadas para actualizar el nombre de usuario administrador y la contraseña a los suyos propios.
[!code-azurecli-interactive[main](../../../cli_scripts/postgresql/change-server-configurations/change-server-configurations.sh?highlight=15-16 "List and update configurations of Azure Database for PostgreSQL.")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación
Use el comando siguiente para quitar el grupo de recursos y todos los recursos asociados a él después de ejecutarse el script de ejemplo. 
[!code-azurecli-interactive[main](../../../cli_scripts/postgresql/change-server-configurations/delete-postgresql.sh  "Delete the resource group.")]

## <a name="script-explanation"></a>Explicación del script
Este script usa los comandos que se describen en la tabla siguiente:

| **Comando** | **Notas** |
|---|---|
| [az group create](/cli/azure/group) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az postgres server create](/cli/azure/postgres/server) | Crea un servidor de PostgreSQL que hospeda las bases de datos. |
| [az postgres server configuration list](/cli/azure/postgres/server/configuration) | Enumera las configuraciones de un servidor de Azure Database for PostgreSQL. |
| [az postgres server configuration set](/cli/azure/postgres/server/configuration) | Actualiza la configuración de un servidor de Azure Database for PostgreSQL. |
| [az postgres server configuration show](/cli/azure/postgres/server/configuration) | Muestra la configuración de un servidor de Azure Database for PostgreSQL. |
| [az group delete](/cli/azure/group) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes
- Para más información sobre la CLI de Azure: [Documentación de la CLI de Azure](/cli/azure).
- Pruebe otros scripts adicionales: [Azure CLI samples for Azure Database for PostgreSQL](../sample-scripts-azure-cli.md) (Ejemplos de la CLI de Azure para Azure Database for PostgreSQL)
- Para más información sobre los parámetros del servidor, consulte [Cómo configurar los parámetros del servidor en Azure Portal](../howto-configure-server-parameters-using-portal.md).
