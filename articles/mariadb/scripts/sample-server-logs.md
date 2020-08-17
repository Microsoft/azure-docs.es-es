---
title: 'Script de la CLI: Descarga de registros de consultas lentas de Azure Database for MariaDB'
description: En este script de ejemplo de la CLI de Azure se muestra cómo habilitar y descargar los registros de consultas lentas de un servidor de Azure Database for MariaDB.
author: ajlam
ms.author: andrela
ms.service: mariadb
ms.devlang: azurecli
ms.topic: sample
ms.custom: mvc, devx-track-azurecli
ms.date: 12/02/2019
ms.openlocfilehash: 5212f4eb17ef7b4ed1b89604fc12a1e13bae78f4
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87502195"
---
# <a name="enable-and-download-server-slow-query-logs-of-an-azure-database-for-mariadb-server-using-azure-cli"></a>Habilitar y descargar registros de consultas lentos de un servidor de Azure Database for MariaDB con la CLI de Azure
Este script de la CLI de ejemplo habilita y descarga los registros de consultas lentos de un servidor individual de Azure Database for MariaDB.

[!INCLUDE [cloud-shell-try-it](../../../includes/cloud-shell-try-it.md)]

Si decide ejecutar la CLI localmente, este artículo necesita la CLI de Azure versión 2.0 o posterior. Ejecute `az --version` para comprobar la versión. Consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli) para instalar o actualizar su versión de la CLI de Azure. 

## <a name="sample-script"></a>Script de ejemplo
En este script de ejemplo, va a modificar las líneas resaltadas para actualizar el nombre de usuario administrador y la contraseña a los suyos propios. Reemplace &lt;log_file_name&gt; en los comandos `az monitor` por su propio nombre del archivo de registro del servidor.
[!code-azurecli-interactive[main](../../../cli_scripts/mariadb/server-logs/server-logs.sh?highlight=15-16 "Manipulate with server logs.")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación
Use el comando siguiente para quitar el grupo de recursos y todos los recursos asociados a él después de ejecutarse el script de ejemplo. 
[!code-azurecli-interactive[main](../../../cli_scripts/mariadb/server-logs/delete-mariadb.sh  "Delete the resource group.")]

## <a name="script-explanation"></a>Explicación del script
Este script usa los comandos que se describen en la tabla siguiente:

| **Comando** | **Notas** |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az mariadb server create](/cli/azure/mariadb/server#az-mariadb-server-create) | Crea un servidor de MariaDB que hospeda las bases de datos. |
| [az mariadb server configuration list](/cli/azure/mariadb/server/configuration#az-mariadb-server-configuration-list) | Enumera los valores de configuración de un servidor. |
| [az mariadb server configuration set](/cli/azure/mariadb/server/configuration#az-mariadb-server-configuration-set) | Actualiza la configuración de un servidor. |
| [az mariadb server-logs list](/cli/azure/mariadb/server-logs#az-mariadb-server-logs-list) | Enumeración de los archivos de registro de un servidor. |
| [az mariadb server-logs download](/cli/azure/mariadb/server-logs#az-mariadb-server-logs-download) | Descargar los archivos de registro. |
| [az group delete](/cli/azure/group#az-group-delete) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes
- Para más información sobre la CLI de Azure: [Documentación de la CLI de Azure](/cli/azure).
- Pruebe otros scripts adicionales: [Ejemplos de la CLI de Azure para Azure Database for MariaDB](../sample-scripts-azure-cli.md)
