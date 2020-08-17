---
title: Administración de puntos de conexión de red virtual de Azure Database for MariaDB mediante la CLI de Azure
description: En este artículo se describe cómo crear y administrar reglas y puntos de conexión de servicio de red virtual de Azure Database for MariaDB mediante la línea de comandos de la CLI de Azure.
author: ajlam
ms.author: andrela
ms.service: mariadb
ms.devlang: azurecli
ms.topic: how-to
ms.date: 3/18/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 8c57013249d614b8933455c6b4d04d012ca87500
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87502297"
---
# <a name="create-and-manage-azure-database-for-mariadb-vnet-service-endpoints-using-azure-cli"></a>Creación y administración de puntos de conexión de servicio de red virtual de Azure Database for MariaDB mediante la CLI de Azure

Las reglas y los puntos de conexión de los servicios de red virtual (VNet) amplían el espacio de direcciones privadas de una red virtual al servidor de Azure Database for MariaDB. Con los comandos de la interfaz de la línea de comandos (CLI) de Azure adecuados, puede crear, actualizar, eliminar, enumerar y mostrar reglas y puntos de conexión de servicio de red virtual para administrar el servidor. Para obtener información general sobre los puntos de conexión de servicio de red virtual de Azure Database for MariaDB, incluidas las limitaciones, consulte [Puntos de conexión de servicio de red virtual del servidor de Azure Database for MariaDB](concepts-data-access-security-vnet.md). Los puntos de conexión del servicio de red virtual están disponibles en todas las regiones admitidas para Azure Database for MariaDB.

## <a name="prerequisites"></a>Prerequisites
Para seguir esta guía, necesitará:
- Instale [la CLI de Azure](/cli/azure/install-azure-cli) o use Azure Cloud Shell en el explorador.
- Un [servidor y una base de datos de Azure Database for MariaDB](quickstart-create-mariadb-server-database-using-azure-cli.md).

> [!NOTE]
> La compatibilidad con puntos de conexión de servicio de red virtual solo existe para servidores de uso general y optimizados para memoria.

## <a name="configure-vnet-service-endpoints"></a>Configuración de los puntos de conexión de servicio de red virtual
Los comandos [az network vnet](https://docs.microsoft.com/cli/azure/network/vnet?view=azure-cli-latest) se usan para configurar redes virtuales.

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para este artículo es preciso que ejecute la versión 2.0 o posterior de la CLI de Azure. Para ver la versión instalada, ejecute el comando `az --version`. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli). 

Si ejecuta localmente la CLI, debe iniciar sesión en su cuenta mediante el comando [az login](https://docs.microsoft.com/cli/azure/authenticate-azure-cli?view=azure-cli-latest). Anote la propiedad **id** de la salida del comando para el nombre de la suscripción correspondiente.
```azurecli-interactive
az login
```

Si tiene varias suscripciones, elija la suscripción adecuada en la que se debe facturar el recurso. Seleccione el identificador de suscripción específico en su cuenta mediante el comando [az account set](https://docs.microsoft.com/cli/azure/account?view=azure-cli-latest#az-account-set). Sustituya la propiedad **id** de la salida **az login** para su suscripción en el marcador de posición de id. de suscripción.

- La cuenta debe tener todos los permisos necesarios para crear una red virtual y un punto de conexión de servicio.

Los puntos de conexión de servicio pueden configurarse en redes virtuales de forma independiente por un usuario con acceso de escritura a la red virtual.

Para proteger los recursos de servicio de Azure en una red virtual, el usuario debe tener permiso en "Microsoft.Network/virtualNetworks/subnets/joinViaServiceEndpoint/" para las subredes que se agregan. De forma predeterminada, este permiso se incluye en los roles de administrador de servicios integrado y puede modificarse mediante la creación de roles personalizados.

Obtenga más información sobre los [roles integrados](https://docs.microsoft.com/azure/active-directory/role-based-access-built-in-roles) y la asignación de permisos específicos a [roles personalizados](https://docs.microsoft.com/azure/active-directory/role-based-access-control-custom-roles).

Las redes virtuales y los recursos de servicio de Azure pueden encontrarse en la misma o en diferentes suscripciones. Si los recursos de servicio de Azure y de red virtual se encuentran en distintas suscripciones, los recursos deben estar en el mismo inquilino de Active Directory (AD). Asegúrese de que ambas suscripciones tengan registrado el proveedor de recursos **Microsoft.Sql**. Para más información, consulte [resource-manager-registration][resource-manager-portal].

> [!IMPORTANT]
> Se recomienda encarecidamente leer este artículo sobre las configuraciones y las consideraciones de puntos de conexión de servicio antes de configurarlos. **punto de conexión de servicio de red virtual:** un [punto de conexión de servicio de red virtual](../virtual-network/virtual-network-service-endpoints-overview.md) es una subred cuyos valores de propiedad incluyen uno o más nombres formales de tipo de servicio de Azure. Los puntos de conexión de servicio de red virtual usan el nombre de tipo de servicio **Microsoft.Sql**, que hace referencia al servicio de Azure denominado SQL Database. Esta etiqueta de servicio también se aplica a los servicios Azure SQL Database, Azure Database for MariaDB, PostgreSQL y MySQL. Es importante tener en cuenta que, al aplicar la etiqueta de servicio **Microsoft.Sql** a un punto de conexión del servicio de red virtual, se configura el tráfico del punto de conexión de servicio de todos los servicios de Azure Database, incluidos los servidores de Azure SQL Database, Azure Database for PostgreSQL, Azure Database for MariaDB y Azure Database for MySQL de la subred.

### <a name="sample-script"></a>Script de ejemplo

Este script de ejemplo se usa para crear una base de datos de Azure Database for MariaDB, crear una red virtual, un punto de conexión de servicio de red virtual y proteger el servidor a la subred con una regla de red virtual. En este script de ejemplo, cambie el nombre de usuario de administrador y la contraseña. Reemplace el identificador de suscripción usado en el comando `az account set --subscription` por su propio identificador de suscripción.

```azurecli-interactive
# To find the name of an Azure region in the CLI run this command: az account list-locations
# Substitute  <subscription id> with your identifier
az account set --subscription <subscription id>

# Create a resource group
az group create \
--name myresourcegroup \
--location westus

# Create a MariaDB server in the resource group
# Name of a server maps to DNS name and is thus required to be globally unique in Azure.
# Substitute the <server_admin_password> with your own value.
az mariadb server create \
--name mydemoserver \
--resource-group myresourcegroup \
--location westus \
--admin-user mylogin \
--admin-password <server_admin_password> \
--sku-name GP_Gen5_2

# Get available service endpoints for Azure region output is JSON
# Use the command below to get the list of services supported for endpoints, for an Azure region, say "westus".
az network vnet list-endpoint-services \
-l westus

# Add Azure SQL service endpoint to a subnet *mySubnet* while creating the virtual network *myVNet* output is JSON
az network vnet create \
-g myresourcegroup \
-n myVNet \
--address-prefixes 10.0.0.0/16 \
-l westus

# Creates the service endpoint
az network vnet subnet create \
-g myresourcegroup \
-n mySubnet \
--vnet-name myVNet \
--address-prefix 10.0.1.0/24 \
--service-endpoints Microsoft.SQL

# View service endpoints configured on a subnet
az network vnet subnet show \
-g myresourcegroup \
-n mySubnet \
--vnet-name myVNet

# Create a VNet rule on the sever to secure it to the subnet Note: resource group (-g) parameter is where the database exists. VNet resource group if different should be specified using subnet id (URI) instead of subnet, VNet pair.
az mariadb server vnet-rule create \
-n myRule \
-g myresourcegroup \
-s mydemoserver \
--vnet-name myVNet \
--subnet mySubnet
```

<!-- 
In this sample script, change the highlighted lines to customize the admin username and password. Replace the SubscriptionID used in the `az account set --subscription` command with your own subscription identifier.
[!code-azurecli-interactive[main](../../cli_scripts/mariadb/create-mysql-server-vnet/create-mysql-server.sh?highlight=5,20 "Create an Azure Database for MariaDB, VNet, VNet service endpoint, and VNet rule.")]
-->

## <a name="clean-up-deployment"></a>Limpieza de la implementación
Después de ejecutar el script de ejemplo, se puede usar el comando siguiente para quitar el grupo de recursos y todos los recursos asociados.

```azurecli-interactive
az group delete --name myresourcegroup
```


<!--
[!code-azurecli-interactive[main](../../cli_scripts/mysql/create-mysql-server-vnet/delete-mysql.sh "Delete the resource group.")]
-->

<!-- Link references, to text, Within this same GitHub repo. --> 
[resource-manager-portal]: ../azure-resource-manager/management/resource-providers-and-types.md
