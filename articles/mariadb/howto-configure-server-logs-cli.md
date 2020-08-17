---
title: Acceso a registros de consultas lentos de Azure Database for MariaDB mediante la CLI de Azure
description: En este artículo se describe cómo acceder a los registros lentos de Azure Database for MariaDB mediante la utilidad de la línea de comandos de la CLI de Azure.
author: ajlam
ms.author: andrela
ms.service: mariadb
ms.devlang: azurecli
ms.topic: how-to
ms.date: 4/13/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: cd74feaebe5a89667668c05e332ed9d3c7cdad5d
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87490258"
---
# <a name="configure-and-access-azure-database-for-maria-db-slow-query-logs-by-using-azure-cli"></a>Configuración y acceso a los registros de consultas lentas de Azure Database for Maria DB mediante CLI de Azure.

Puede descargar los registros de consultas lentos de Azure Database for MySQL mediante la CLI de Azure, la utilidad de línea de comandos de Azure.

## <a name="prerequisites"></a>Requisitos previos
Para seguir esta guía, necesitará:
- [Servidor de Azure Database for MariaDB](quickstart-create-mariadb-server-database-using-azure-cli.md)
- La [CLI de Azure](/cli/azure/install-azure-cli) o Azure Cloud Shell en el explorador

## <a name="configure-logging"></a>registro
Puede configurar el servidor para acceder al registro de consultas lentas de MySQL con los pasos siguientes:
1. Active el registro de consultas lentas al establecer el parámetro **slow\_query\_log** en ON.
2. Seleccione la ubicación en la que se van a generar los registros con **log\_output**. Para enviar registros al almacenamiento local y a los registros de diagnóstico de Azure Monitor, seleccione **Archivo**. Para enviar registros solo a los registros de Azure Monitor, seleccione **Ninguno**.
3. Ajuste otros parámetros, como **long\_query\_time** y **log\_slow\_admin\_statements**.

Para aprender a establecer el valor de estos parámetros mediante la CLI de Azure, consulte [Cómo configurar parámetros del servidor](howto-configure-server-parameters-cli.md).

Por ejemplo, el siguiente comando de la CLI activará el registro de consultas lentas, establecerá el tiempo de consultas largas en 10 segundos y desactivará el registro de la instrucción de administración lenta. Por último, se muestran las opciones de configuración para su revisión.
```azurecli-interactive
az mariadb server configuration set --name slow_query_log --resource-group myresourcegroup --server mydemoserver --value ON
az mariadb server configuration set --name log_output --resource-group myresourcegroup --server mydemoserver --value FILE
az mariadb server configuration set --name long_query_time --resource-group myresourcegroup --server mydemoserver --value 10
az mariadb server configuration set --name log_slow_admin_statements --resource-group myresourcegroup --server mydemoserver --value OFF
az mariadb server configuration list --resource-group myresourcegroup --server mydemoserver
```

## <a name="list-logs-for-azure-database-for-mariadb-server"></a>Lista de registros del servidor de Azure Database for MariaDB
Si **log_output** está configurado en "File" (Archivo), puede acceder a los registros directamente desde el almacenamiento local del servidor. Para mostrar la lista de archivos de registro de consultas lentas disponibles para el servidor, ejecute el comando [az mariadb server-logs list](/cli/azure/mariadb/server-logs#az-mariadb-server-logs-list).

Puede enumerar los archivos de registro del servidor **mydemoserver.mariadb.database.azure.com** en el grupo de recursos **myresourcegroup**. A continuación, dirija la lista de archivos de registro a un archivo de texto denominado **log\_files\_list.txt**.
```azurecli-interactive
az mariadb server-logs list --resource-group myresourcegroup --server mydemoserver > log_files_list.txt
```
## <a name="download-logs-from-the-server"></a>Descarga de registros del servidor
Si **log_output** está configurado como "File" (Archivo), puede descargar archivos de registro individuales desde el servidor con el comando [az mariadb server-logs download](/cli/azure/mariadb/server-logs#az-mariadb-server-logs-download).

Use el ejemplo siguiente para descargar el archivo de registro específico del servidor **mydemoserver.mariadb.database.azure.com** en el grupo de recursos **myresourcegroup** a su entorno local.
```azurecli-interactive
az mariadb server-logs download --name mysql-slow-mydemoserver-2018110800.log --resource-group myresourcegroup --server mydemoserver
```

## <a name="next-steps"></a>Pasos siguientes
- Más información sobre los [registros de consultas lentas en Azure Database for MariaDB](concepts-server-logs.md).
