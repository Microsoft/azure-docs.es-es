---
title: 'Script de la CLI de Azure: descarga de registros de Azure Database for PostgreSQL'
description: Este script de la CLI de Azure de ejemplo muestra cómo habilitar y descargar los registros del servidor de Azure Database for PostgreSQL.
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.devlang: azurecli
ms.topic: sample
ms.custom: mvc, devx-track-azurecli
ms.date: 02/28/2018
ms.openlocfilehash: 11940cce5ea4bf6265c451ee765c385a18b956b9
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87496488"
---
# <a name="enable-and-download-server-slow-query-logs-of-an-azure-database-for-postgresql-server-using-azure-cli"></a>Habilitar y descargar registros de consultas lentos de un servidor de Azure Database for PostgreSQL con la CLI de Azure
Este script de la CLI de ejemplo habilita y descarga los registros de consultas lentos de un único servidor de Azure Database for PostgreSQL.

[!INCLUDE [cloud-shell-try-it](../../../includes/cloud-shell-try-it.md)]

Si decide ejecutar la CLI localmente, este artículo necesita la CLI de Azure versión 2.0 o posterior. Ejecute `az --version` para comprobar la versión. Consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli) para instalar o actualizar su versión de la CLI de Azure.

## <a name="sample-script"></a>Script de ejemplo
En este script de ejemplo, va a modificar las líneas resaltadas para actualizar el nombre de usuario administrador y la contraseña a los suyos propios. Reemplace &lt;log_file_name&gt; en los comandos `az monitor` por su propio nombre del archivo de registro del servidor.
[!code-azurecli-interactive[main](../../../cli_scripts/postgresql/server-logs/server-logs.sh?highlight=15-16 "Manipulate with server logs.")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación
Use el comando siguiente para quitar el grupo de recursos y todos los recursos asociados a él después de ejecutarse el script de ejemplo. 
[!code-azurecli-interactive[main](../../../cli_scripts/postgresql/server-logs/delete-postgresql.sh  "Delete the resource group.")]

## <a name="script-explanation"></a>Explicación del script
Este script usa los comandos que se describen en la tabla siguiente:

| **Comando** | **Notas** |
|---|---|
| [az group create](/cli/azure/group) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az postgres server create](/cli/azure/postgres/server) | Crea un servidor de PostgreSQL que hospeda las bases de datos. |
| [az postgres server configuration list](/cli/azure/postgres/server/configuration) | Enumera los valores de configuración de un servidor. |
| [az postgres server configuration set](/cli/azure/postgres/server/configuration) | Actualiza la configuración de un servidor. |
| [az postgres server-logs list](/cli/azure/postgres/server-logs) | Enumeración de los archivos de registro de un servidor. |
| [az postgres server-logs download](/cli/azure/postgres/server-logs) | Descargar los archivos de registro. |
| [az group delete](/cli/azure/group) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes
- Para más información sobre la CLI de Azure: [Documentación de la CLI de Azure](/cli/azure).
- Pruebe otros scripts adicionales: [Azure CLI samples for Azure Database for PostgreSQL](../sample-scripts-azure-cli.md) (Ejemplos de la CLI de Azure para Azure Database for PostgreSQL)
- [Configuración y acceso a los registros del servidor en Azure Portal](../howto-configure-server-logs-in-portal.md)
