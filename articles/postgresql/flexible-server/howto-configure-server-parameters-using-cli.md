---
title: 'Configuración de parámetros de Azure Database for PostgreSQL: servidor flexible'
description: 'En este artículo se describe cómo configurar los parámetros de Postgres en Azure Database for PostgreSQL: servidor flexible con la CLI de Azure.'
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.devlang: azurecli
ms.topic: how-to
ms.date: 9/22/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 43b14858cc99cac41e277b03171fd4cac4d6eafa
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932588"
---
# <a name="customize-server-parameters-for-azure-database-for-postgresql---flexible-server-using-azure-cli"></a>Personalización de los parámetros de Azure Database for PostgreSQL con la CLI de Azure: servidor flexible

Puede enumerar, mostrar y actualizar los parámetros de configuración de un servidor Azure PostgreSQL con la interfaz de la línea de comandos (CLI de Azure). En el nivel del servidor, se expone un subconjunto de los parámetros del motor que es posible modificar. 

## <a name="prerequisites"></a>Requisitos previos

Para seguir esta guía, necesitará:
- Crear una base de datos y un servidor Azure Database for PostgreSQL siguiendo las instrucciones de [Creación de una instancia de Azure Database for PostgreSQL](quickstart-create-server-cli.md)
- Instalar la interfaz de la línea de comandos [CLI de Azure](/cli/azure/install-azure-cli) en la máquina o usar [Azure Cloud Shell](../../cloud-shell/overview.md) en Azure Portal mediante el explorador.

## <a name="list-server-parameters-for-a-flexible-server"></a>Lista de los parámetros de servidor para un servidor flexible

Para obtener una lista de todos los parámetros modificables en un servidor y sus valores, ejecute el comando [az postgres flexible-server parameter list](/cli/azure/postgres/flexible-server/parameter).

Puede enumerar los parámetros del servidor **mydemoserver.postgres.database.azure.com** en el grupo de recursos **myresourcegroup**.

```azurecli-interactive
az postgres flexible-server parameter list --resource-group myresourcegroup --server-name mydemoserver
```

## <a name="show-server-parameter-details"></a>Visualización de los detalles de los parámetros de servidor

Para mostrar los detalles de un parámetro específico de un servidor, ejecute el comando [az postgres flexible-server parameter show](/cli/azure/postgres/flexible-server/parameter).

En este ejemplo se muestran detalles del parámetro**log\_min\_messages** del servidor **mydemoserver.postgres.database.azure.com** en el grupo de recursos **myresourcegroup**.

```azurecli-interactive
az postgres flexible-server parameter show --name log_min_messages --resource-group myresourcegroup --server-name mydemoserver
```

## <a name="modify-server-parameter-value"></a>Modificación del valor de los parámetros del servidor

También puede modificar el valor de un determinado parámetro del servidor; esta acción actualizará el valor de configuración subyacente del motor del servidor PostgreSQL. Para actualizar el parámetro, use el comando [az postgres flexible-server parameter set](/cli/azure/postgres/flexible-server/parameter). 

Para actualizar el parámetro **log\_min\_messages** del servidor **mydemoserver.postgres.database.azure.com** en el grupo de recursos **myresourcegroup**.

```azurecli-interactive
az postgres flexible-server parameter set --name log_min_messages --value INFO --resource-group myresourcegroup --server-name mydemoserver
```

Si quiere restablecer el valor de un parámetro, basta con no incluir el parámetro opcional `--value` y el servicio aplicará el valor predeterminado. En el ejemplo anterior, sería:

```azurecli-interactive
az postgres flexible-server parameter set --name log_min_messages --resource-group myresourcegroup --server-name mydemoserver
```

El parámetro **log\_min\_messages** se restablecerá al valor predeterminado **WARNING**. Para obtener más información sobre los parámetros del servidor y los valores permitidos, consulte la documentación de PostgreSQL sobre la [configuración de parámetros](https://www.postgresql.org/docs/12/config-setting.html).

## <a name="next-steps"></a>Pasos siguientes

- Para configurar y obtener acceso a los registros del servidor, consulte [Registros del servidor en Azure Database for PostgreSQL](concepts-logging.md).
