---
title: 'Configuración de parámetros de Azure Database for PostgreSQL: servidor único'
description: 'En este artículo se describe cómo configurar los parámetros de Postgres de Azure Database for PostgreSQL: servidor único mediante la CLI de Azure.'
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.devlang: azurecli
ms.topic: how-to
ms.date: 06/19/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: e0d19720df6f3bea8d65c24076709e84c585c638
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87493506"
---
# <a name="customize-server-configuration-parameters-for-azure-database-for-postgresql---single-server-using-azure-cli"></a>Personalización de los parámetros de configuración del servidor de Azure Database for PostgreSQL: Single Server mediante la CLI de Azure
Puede enumerar, mostrar y actualizar los parámetros de configuración de un servidor Azure PostgreSQL con la interfaz de la línea de comandos (CLI de Azure). Sin embargo, en el nivel del servidor, solo se expone y se puede modificar un subconjunto de las opciones de configuración del motor. 

## <a name="prerequisites"></a>Requisitos previos
Para seguir esta guía, necesitará:
- Crear una base de datos y un servidor Azure Database for PostgreSQL siguiendo las instrucciones de [Creación de una instancia de Azure Database for PostgreSQL](quickstart-create-server-database-azure-cli.md)
- Instalar la interfaz de la línea de comandos [CLI de Azure](/cli/azure/install-azure-cli) en la máquina o usar [Azure Cloud Shell](../cloud-shell/overview.md) en Azure Portal mediante el explorador.

## <a name="list-server-configuration-parameters-for-azure-database-for-postgresql-server"></a>Lista de los parámetros de configuración del servidor de Azure Database for PostgreSQL
Para obtener una lista de todos los parámetros modificables en un servidor y sus valores, ejecute el comando [az postgres server configuration list](/cli/azure/postgres/server/configuration).

Puede enumerar los parámetros de configuración del servidor **mydemoserver.postgres.database.azure.com** en el grupo de recursos **myresourcegroup**.
```azurecli-interactive
az postgres server configuration list --resource-group myresourcegroup --server mydemoserver
```
## <a name="show-server-configuration-parameter-details"></a>Presentación de los detalles de los parámetros de configuración del servidor
Para mostrar los detalles de un parámetro de configuración específico de un servidor, ejecute el comando [az postgres server configuration show](/cli/azure/postgres/server/configuration).

En este ejemplo se muestran detalles del parámetro de configuración **log\_min\_messages** del servidor **mydemoserver.postgres.database.azure.com** en el grupo de recursos **myresourcegroup.**
```azurecli-interactive
az postgres server configuration show --name log_min_messages --resource-group myresourcegroup --server mydemoserver
```
## <a name="modify-server-configuration-parameter-value"></a>Modificación del valor de los parámetros de configuración del servidor
También puede modificar el valor de un determinado parámetro de configuración del servidor; esta acción actualizará el valor de configuración subyacente del motor del servidor PostgreSQL. Para actualizar la configuración, use el comando [az postgres server configuration set](/cli/azure/postgres/server/configuration). 

Para actualizar el parámetro de configuración **log\_min\_messages** del servidor **mydemoserver.postgres.database.azure.com** en el grupo de recursos **myresourcegroup.**
```azurecli-interactive
az postgres server configuration set --name log_min_messages --resource-group myresourcegroup --server mydemoserver --value INFO
```
Si desea restablecer el valor de un parámetro de configuración, basta con no incluir el parámetro opcional `--value` y el servicio aplicará el valor predeterminado. En el ejemplo anterior, sería:
```azurecli-interactive
az postgres server configuration set --name log_min_messages --resource-group myresourcegroup --server mydemoserver
```
La configuración de **log\_min\_messages** se restablecerá al valor predeterminado **WARNING**. Para obtener más información sobre la configuración del servidor y los valores permitidos, consulte la documentación de PostgreSQL en [Server Configuration](https://www.postgresql.org/docs/9.6/static/runtime-config.html) (Configuración del servidor).

## <a name="next-steps"></a>Pasos siguientes
- [Aprenda a reiniciar un servidor](howto-restart-server-cli.md)
- Para configurar y obtener acceso a los registros del servidor, consulte [Registros del servidor en Azure Database for PostgreSQL](concepts-server-logs.md).
