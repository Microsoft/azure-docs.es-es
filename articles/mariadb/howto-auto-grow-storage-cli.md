---
title: Crecimiento automático del almacenamiento mediante la CLI de Azure en Azure Database for MariaDB
description: En este artículo se describe cómo habilitar el crecimiento automático de almacenamiento en Azure Database for MariaDB mediante la CLI de Azure.
author: ambhatna
ms.author: ambhatna
ms.service: mariadb
ms.topic: how-to
ms.date: 3/18/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 2f4ebe13560da58685ac352a102042b9312d4efd
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87497144"
---
# <a name="auto-grow-azure-database-for-mariadb-storage-using-the-azure-cli"></a>Crecimiento automático del almacenamiento de Azure Database for MariaDB mediante la CLI de Azure
En este artículo se describe cómo configurar el almacenamiento en el servidor Azure Database for MariaDB para que aumente sin que ello afecte a la carga de trabajo.

El servidor [que alcanza el límite de almacenamiento](https://docs.microsoft.com/azure/mariadb/concepts-pricing-tiers#reaching-the-storage-limit) se establece en solo lectura. Si el crecimiento automático del almacenamiento está habilitado, para servidores con menos de 100 GB de almacenamiento aprovisionado, el tamaño del almacenamiento aprovisionado se incrementa en 5 GB tan pronto como el almacenamiento disponible se encuentre por debajo de 1 GB o el 10 % del almacenamiento aprovisionado. En cuanto a servidores con más de 100 GB de almacenamiento aprovisionado, el tamaño del almacenamiento aprovisionado se incrementa en un 5 % cuando el espacio de almacenamiento disponible es inferior al 5 % del tamaño de almacenamiento aprovisionado. Se aplican los límites máximos de almacenamiento según lo especificado [aquí](https://docs.microsoft.com/azure/mariadb/concepts-pricing-tiers#storage).

## <a name="prerequisites"></a>Requisitos previos
Para completar esta guía, necesita:
- Un [servidor de Azure Database for MariaDB](quickstart-create-mariadb-server-database-using-azure-cli.md)

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

> [!IMPORTANT]
> Esta guía de procedimientos requiere el uso de la CLI de Azure versión 2.0 o posterior. Para confirmar la versión, en el símbolo del sistema de la CLI de Azure, escriba `az --version`. Para la instalación o la actualización, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="enable-mariadb-server-storage-auto-grow"></a>Habilitar el crecimiento automático del almacenamiento de servidores MariaDB

Habilite el almacenamiento de crecimiento automático de servidor en un servidor existente con el siguiente comando:

```azurecli-interactive
az mariadb server update --name mydemoserver --resource-group myresourcegroup --auto-grow Enabled
```

Habilite el almacenamiento de crecimiento automático de servidor mientras crea un nuevo servidor con el siguiente comando:

```azurecli-interactive
az mariadb server create --resource-group myresourcegroup --name mydemoserver  --auto-grow Enabled --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen5_2 --version 10.3
```

## <a name="next-steps"></a>Pasos siguientes

Aprenda sobre la [creación de alertas de métricas](howto-alert-metric.md).
