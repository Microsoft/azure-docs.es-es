---
title: Acceso a registros de consultas lentas mediante la CLI de Azure en Azure Database for MySQL
description: En este artículo se describe cómo acceder a los registros de consultas lentas de Azure Database for MySQL mediante la CLI de Azure.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.devlang: azurecli
ms.topic: how-to
ms.date: 4/13/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: a662fe9300d78591cc7ae1fcff67f9b3f93ab8c8
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87501225"
---
# <a name="configure-and-access-slow-query-logs-by-using-azure-cli"></a>Configuración y acceso a los registros de consultas lentas con la CLI de Azure
Puede descargar los registros de consultas lentas de Azure Database for MySQL mediante la CLI de Azure, la utilidad de línea de comandos de Azure.

## <a name="prerequisites"></a>Prerrequisitos
Para seguir esta guía, necesitará:
- [Servidor de Azure Database for MySQL](quickstart-create-mysql-server-database-using-azure-cli.md)
- La [CLI de Azure](/cli/azure/install-azure-cli) o Azure Cloud Shell en el explorador

## <a name="configure-logging"></a>registro
Puede configurar el servidor para acceder al registro de consultas lentas de MySQL con los pasos siguientes:
1. Active el registro de consultas lentas al establecer el parámetro **slow\_query\_log** en ON.
2. Seleccione la ubicación en la que se van a generar los registros con **log\_output**. Para enviar registros al almacenamiento local y a los registros de diagnóstico de Azure Monitor, seleccione **Archivo**. Para enviar registros solo a los registros de Azure Monitor, seleccione **Ninguno**.
3. Ajuste otros parámetros, como **long\_query\_time** y **log\_slow\_admin\_statements**.

Para aprender a establecer el valor de estos parámetros mediante la CLI de Azure, consulte [Cómo configurar parámetros del servidor](howto-configure-server-parameters-using-cli.md).

Por ejemplo, el siguiente comando de la CLI activará el registro de consultas lentas, establecerá el tiempo de consultas largas en 10 segundos y desactivará el registro de la instrucción de administración lenta. Por último, se muestran las opciones de configuración para su revisión.
```azurecli-interactive
az mysql server configuration set --name slow_query_log --resource-group myresourcegroup --server mydemoserver --value ON
az mysql server configuration set --name log_output --resource-group myresourcegroup --server mydemoserver --value FILE
az mysql server configuration set --name long_query_time --resource-group myresourcegroup --server mydemoserver --value 10
az mysql server configuration set --name log_slow_admin_statements --resource-group myresourcegroup --server mydemoserver --value OFF
az mysql server configuration list --resource-group myresourcegroup --server mydemoserver
```

## <a name="list-logs-for-azure-database-for-mysql-server"></a>Enumeración de registros del servidor de Azure Database for MySQL
Si **log_output** está configurado en "Archivo", puede acceder a los registros directamente desde el almacenamiento local del servidor. Para mostrar la lista de archivos de registro de consultas lentas disponibles para el servidor, ejecute el comando [az mysql server-logs list](/cli/azure/mysql/server-logs#az-mysql-server-logs-list).

Puede enumerar archivos de registro del servidor **mydemoserver.mysql.database.azure.com** en el grupo de recursos **myresourcegroup**. A continuación, dirija la lista de archivos de registro a un archivo de texto denominado **log\_files\_list.txt**.
```azurecli-interactive
az mysql server-logs list --resource-group myresourcegroup --server mydemoserver > log_files_list.txt
```
## <a name="download-logs-from-the-server"></a>Descarga de registros del servidor
Si **log_output** está configurado como "Archivo", puede descargar archivos de registro individuales desde el servidor con el comando [az mysql server-logs download](/cli/azure/mysql/server-logs#az-mysql-server-logs-download).

Use el ejemplo siguiente para descargar el archivo de registro específico para el servidor **mydemoserver.mysql.database.azure.com** en el grupo de recursos **myresourcegroup** a su entorno local.
```azurecli-interactive
az mysql server-logs download --name 20170414-mydemoserver-mysql.log --resource-group myresourcegroup --server mydemoserver
```

## <a name="next-steps"></a>Pasos siguientes
- Más información sobre los [Registros de consultas lentas en Azure Database for MySQL](concepts-server-logs.md).
