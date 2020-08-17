---
title: Configuración de parámetros de servidor mediante la CLI de Azure en Azure Database for MySQL
description: En este artículo se describe cómo configurar los parámetros de servicio de Azure Database for MySQL mediante la utilidad de línea de comandos de la CLI de Azure.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.devlang: azurecli
ms.topic: how-to
ms.date: 6/11/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 43562454e8ddbeb3e674cbdbace508ed9ca1d549
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87501177"
---
# <a name="configure-server-parameters-in-azure-database-for-mysql-using-the-azure-cli"></a>Configuración de parámetros del servidor en Azure Database for MySQL mediante la CLI de Azure
Puede enumerar, mostrar y actualizar los parámetros de configuración de un servidor de Azure Database for MySQL con la CLI de Azure, la utilidad de línea de comandos de Azure. En el nivel del servidor, se expone y se puede modificar un subconjunto de las opciones de configuración del motor. 

## <a name="prerequisites"></a>Requisitos previos
Para seguir esta guía, necesitará:
- [Un servidor de Azure Database for MySQL](quickstart-create-mysql-server-database-using-azure-cli.md)
- La utilidad de línea de comandos [CLI de Azure](/cli/azure/install-azure-cli) o usar Azure Cloud Shell en el explorador.

## <a name="list-server-configuration-parameters-for-azure-database-for-mysql-server"></a>Lista de los parámetros de configuración del servidor de Azure Database for MySQL
Para obtener una lista de todos los parámetros modificables en un servidor y sus valores, ejecute el comando [az mysql server configuration list](/cli/azure/mysql/server/configuration#az-mysql-server-configuration-list).

Puede enumerar los parámetros de configuración del servidor **mydemoserver.mysql.database.azure.com** en el grupo de recursos **myresourcegroup**.
```azurecli-interactive
az mysql server configuration list --resource-group myresourcegroup --server mydemoserver
```
Para ver la definición de cada uno de los parámetros enumerados, consulte la sección de referencia de MySQL en [Server System Variables](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html) (Variables del sistema del servidor).

## <a name="show-server-configuration-parameter-details"></a>Presentación de los detalles de los parámetros de configuración del servidor
Para mostrar los detalles de un parámetro de configuración específico de un servidor, ejecute el comando [az mysql server configuration show](/cli/azure/mysql/server/configuration#az-mysql-server-configuration-show).

En este ejemplo se muestran detalles del parámetro de configuración **slow\_query\_log** del servidor **mydemoserver.mysql.database.azure.com** en el grupo de recursos **myresourcegroup.**
```azurecli-interactive
az mysql server configuration show --name slow_query_log --resource-group myresourcegroup --server mydemoserver
```
## <a name="modify-a-server-configuration-parameter-value"></a>Modificación de un valor de los parámetros de configuración del servidor
También puede modificar el valor de un determinado parámetro de configuración del servidor; esta acción actualizará el valor de configuración subyacente del motor del servidor de MySQL. Para actualizar la configuración, use el comando [az mysql server configuration set](/cli/azure/mysql/server/configuration#az-mysql-server-configuration-set). 

Para actualizar el parámetro de configuración **slow\_query\_log** del servidor **mydemoserver.mysql.database.azure.com** en el grupo de recursos **myresourcegroup.**
```azurecli-interactive
az mysql server configuration set --name slow_query_log --resource-group myresourcegroup --server mydemoserver --value ON
```
Si desea restablecer el valor de un parámetro de configuración, omita el parámetro opcional `--value` y el servicio aplicará el valor predeterminado. Para el ejemplo anterior, sería:
```azurecli-interactive
az mysql server configuration set --name slow_query_log --resource-group myresourcegroup --server mydemoserver
```
Este código restablece la configuración **slow\_query\_log** en el valor predeterminado **Apagado**. 

## <a name="setting-parameters-not-listed"></a>Valor de parámetros que no aparecen en la lista
Si el parámetro de servidor que desea actualizar no aparece en Azure Portal, también puede establecer el parámetro en el nivel de conexión mediante `init_connect`. De este modo, se establecen los parámetros del servidor para cada cliente con conexión al servidor. 

Actualice el parámetro de configuración del servidor **init\_connect** del servidor **mydemoserver.mysql.database.azure.com** en el grupo de recursos **myresourcegroup** para establecer valores como el juego de caracteres.
```azurecli-interactive
az mysql server configuration set --name init_connect --resource-group myresourcegroup --server mydemoserver --value "SET character_set_client=utf8;SET character_set_database=utf8mb4;SET character_set_connection=latin1;SET character_set_results=latin1;"
```

## <a name="working-with-the-time-zone-parameter"></a>Trabajo con el parámetro de zona horaria

### <a name="populating-the-time-zone-tables"></a>Relleno de las tablas de la zona horaria

Las tablas de la zona horaria del servidor se pueden rellenar mediante una llamada al procedimiento almacenado `mysql.az_load_timezone` desde una herramienta como la línea de comandos de MySQL o MySQL Workbench.

> [!NOTE]
> Si ejecuta el comando `mysql.az_load_timezone` de MySQL Workbench, es posible que primero tenga que desactivar el modo de actualización segura mediante `SET SQL_SAFE_UPDATES=0;`.

```sql
CALL mysql.az_load_timezone();
```

> [!IMPORTANT]
> Debe reiniciar el servidor para asegurarse de que las tablas de zona horaria se rellenen correctamente. Para reiniciar el servidor, use [Azure Portal](howto-restart-server-portal.md) o la [CLI](howto-restart-server-cli.md).

Para ver los valores de zonas horarias disponibles, ejecute el comando siguiente:

```sql
SELECT name FROM mysql.time_zone_name;
```

### <a name="setting-the-global-level-time-zone"></a>Establecimiento de la zona horaria de nivel global

La zona horaria de nivel global se puede establecer mediante el comando [az mysql server configuration set](/cli/azure/mysql/server/configuration#az-mysql-server-configuration-set).

El siguiente comando actualiza el parámetro de configuración **time\_zone** del servidor **mydemoserver.mysql.database.azure.com** en el grupo de recursos **myresourcegroup** a **US/Pacific**.

```azurecli-interactive
az mysql server configuration set --name time_zone --resource-group myresourcegroup --server mydemoserver --value "US/Pacific"
```

### <a name="setting-the-session-level-time-zone"></a>Establecimiento de la zona horaria de nivel de sesión

La zona horaria de nivel de sesión se puede establecer mediante la ejecución del comando `SET time_zone` desde una herramienta como la línea de comandos de MySQL o MySQL Workbench. En el ejemplo siguiente se establece la zona horaria en **US/Pacific**.  

```sql
SET time_zone = 'US/Pacific';
```

Consulte [Date and Time Functions](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_convert-tz) (Funciones de fecha y hora) en la documentación de MySQL.


## <a name="next-steps"></a>Pasos siguientes

- Cómo configurar [parámetros del servidor en Azure Portal](howto-server-parameters.md)
