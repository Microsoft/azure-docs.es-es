---
title: 'Administración de reglas de firewall de Azure Database for PostgreSQL: servidor único mediante la CLI de Azure'
description: En este artículo se describe cómo crear y administrar reglas de firewall de Azure Database for PostgreSQL con un único servidor mediante la línea de comandos de la CLI de Azure.
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.devlang: azurecli
ms.topic: how-to
ms.date: 5/6/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: 17a41030adeb302f2c85b1bd4576caad9e723544
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87489765"
---
# <a name="create-and-manage-firewall-rules-in-azure-database-for-postgresql---single-server-using-azure-cli"></a>Creación y administración de reglas de firewall en Azure Database for PostgreSQL con un único servidor mediante la CLI de Azure
Las reglas de firewall de nivel de servidor se pueden usar para administrar el acceso a un servidor de Azure Database for PostgreSQL desde una dirección IP específica o desde un intervalo de direcciones IP. Con los comandos de la CLI de Azure adecuados, puede crear, actualizar, eliminar, enumerar y mostrar reglas de firewall para administrar el servidor. Para obtener información general sobre las reglas de firewall de Azure Database for PostgreSQL, consulte [Reglas de firewall del servidor de Azure Database for PostgreSQL](concepts-firewall-rules.md).

Las reglas de Virtual Network (VNet) también se pueden usar para proteger el acceso a su servidor. Más información sobre cómo [crear y administrar puntos de conexión de servicio y reglas de Virtual Network mediante la CLI de Azure](howto-manage-vnet-using-cli.md).

## <a name="prerequisites"></a>Requisitos previos
Para seguir esta guía, necesitará:
- Instale la utilidad de línea de comandos [CLI de Azure](/cli/azure/install-azure-cli) o use Azure Cloud Shell en el explorador.
- Un [servidor y una base de datos de Azure Database for PostgreSQL](quickstart-create-server-database-azure-cli.md).

## <a name="configure-firewall-rules-for-azure-database-for-postgresql"></a>Configuración de reglas de firewall para Azure Database for PostgreSQL
Los comandos [az postgres server firewall-rule](/cli/azure/postgres/server/firewall-rule) se usan para configurar reglas de firewall.

## <a name="list-firewall-rules"></a>Enumerar reglas de firewall 
Para enumerar las reglas de firewall de servidor existentes, ejecute el comando [az postgres server firewall-rule list](/cli/azure/postgres/server/firewall-rule).
```azurecli-interactive
az postgres server firewall-rule list --resource-group myresourcegroup --server-name mydemoserver
```
La salida enumera las reglas de firewall, si existen, en formato JSON de forma predeterminada. Puede usar el modificador `--output table` para obtener una salida en formato de tabla más legible.
```azurecli-interactive
az postgres server firewall-rule list --resource-group myresourcegroup --server-name mydemoserver --output table
```
## <a name="create-firewall-rule"></a>Crear reglas de firewall
Para crear una regla de firewall en el servidor, ejecute el comando [az postgres server firewall-rule create](/cli/azure/postgres/server/firewall-rule). 

```
To allow access to a singular IP address, provide the same address in the `--start-ip-address` and `--end-ip-address`, as in this example, replacing the IP shown here with your specific IP.
```azurecli-interactive
az postgres server firewall-rule create --resource-group myresourcegroup --server-name mydemoserver --name AllowSingleIpAddress --start-ip-address 13.83.152.1 --end-ip-address 13.83.152.1
```
Para permitir que las aplicaciones de las direcciones IP de Azure se conecten al servidor Azure Database for PostgreSQL, proporcione la dirección IP 0.0.0.0 como IP inicial e IP final, como en este ejemplo.
```azurecli-interactive
az postgres server firewall-rule create --resource-group myresourcegroup --server-name mydemoserver --name AllowAllAzureIps --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

> [!IMPORTANT]
> Esta opción configura el firewall para permitir todas las conexiones de Azure, incluidas las de las suscripciones de otros clientes. Al seleccionar esta opción, asegúrese de que los permisos de usuario y el inicio de sesión limiten el acceso solamente a los usuarios autorizados.
> 

Si se realiza correctamente, la salida del comando muestra los detalles de la regla de firewall que ha creado, de forma predeterminada en formato JSON. Si se produce un error, la salida muestra un mensaje de error en su lugar.

## <a name="update-firewall-rule"></a>Actualizar reglas de firewall 
Se puede actualizar una regla de firewall existente en el servidor con el comando [az postgres server firewall-rule update](/cli/azure/postgres/server/firewall-rule). Proporcione como entrada el nombre de una regla de firewall existente y los atributos de dirección IP de inicio y final que se van a actualizar.
```azurecli-interactive
az postgres server firewall-rule update --resource-group myresourcegroup --server-name mydemoserver --name AllowIpRange --start-ip-address 13.83.152.0 --end-ip-address 13.83.152.0
```
Cuando se realiza correctamente, la salida del comando muestra los detalles de la regla de firewall que ha actualizado, de forma predeterminada en formato JSON. Si se produce un error, la salida muestra un mensaje de error en su lugar.
> [!NOTE]
> Si la regla de firewall no existe, el comando update la crea.

## <a name="show-firewall-rule-details"></a>Mostrar los detalles de la regla de firewall
También puede mostrar los detalles de una regla de firewall existente a nivel de servidor mediante la ejecución del comando [az postgres server firewall-rule show](/cli/azure/postgres/server/firewall-rule).
```azurecli-interactive
az postgres server firewall-rule show --resource-group myresourcegroup --server-name mydemoserver --name AllowIpRange
```
Cuando se realiza correctamente, la salida del comando muestra los detalles de la regla de firewall que ha especificado, de forma predeterminada en formato JSON. Si se produce un error, la salida muestra un mensaje de error en su lugar.

## <a name="delete-firewall-rule"></a>Eliminar reglas de firewall
Para revocar el acceso de un intervalo IP al servidor, elimine una regla de firewall existente mediante la ejecución del comando [az postgres server firewall-rule delete](/cli/azure/postgres/server/firewall-rule). Proporcione el nombre de una regla de firewall existente.
```azurecli-interactive
az postgres server firewall-rule delete --resource-group myresourcegroup --server-name mydemoserver --name AllowIpRange
```
Cuando se realiza correctamente, no hay ninguna salida. En caso de error, se devuelve el texto del mensaje de error.

## <a name="next-steps"></a>Pasos siguientes
- De forma similar, puede utilizar un explorador web para [crear y administrar reglas de firewall de Azure Database for PostgreSQL mediante Azure Portal](howto-manage-firewall-using-portal.md).
- Para más información, consulte [Reglas de firewall del servidor de Azure Database for PostgreSQL](concepts-firewall-rules.md).
- Proteja más aun el acceso al servidor con la [creación y la administración puntos de conexión de servicio y reglas de Virtual Network mediante la CLI de Azure](howto-manage-vnet-using-cli.md).
- Para obtener ayuda para la conexión a un servidor de Azure Database for PostgreSQL, consulte [Bibliotecas de conexión para Azure Database for PostgreSQL](concepts-connection-libraries.md).
