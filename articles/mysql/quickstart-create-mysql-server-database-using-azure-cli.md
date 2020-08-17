---
title: 'Inicio rápido: Creación de un servidor: de CLI de Azure (Azure Database for MySQL)'
description: En esta guía de inicio rápido se describe cómo usar la CLI de Azure para crear una Base de datos de Azure para el servidor MySQL en un grupo de recursos de Azure.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 07/15/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: a7d69ae15e72133e08b3e66de607aa06fefdbd32
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87495394"
---
# <a name="quickstart-create-an-azure-database-for-mysql-server-using-azure-cli"></a>Inicio rápido: Creación de una Base de datos de Azure para el servidor MySQL con la CLI de Azure

> [!TIP]
> Considere la posibilidad de usar un comando de la CLI de Azure [az mysql up](/cli/azure/ext/db-up/mysql#ext-db-up-az-mysql-up) más sencillo (actualmente en versión preliminar). Pruebe el [inicio rápido](./quickstart-create-server-up-azure-cli.md).

En este inicio rápido se muestra cómo usar los comandos de la [CLI de Azure](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli) en [Azure Cloud Shell](https://shell.azure.com) para crear un servidor de Azure Database for MySQL en cinco minutos. Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

> [!NOTE]
> Si decide instalar y usar la CLI localmente, para este artículo es preciso que ejecute la versión 2.0 o posterior de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli). 

## <a name="prerequisites"></a>Requisitos previos
En este artículo es necesario que ejecute la versión 2.0 de la CLI de Azure, o cualquier versión posterior, de forma local. Para ver la versión instalada, ejecute el comando `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

Será preciso que inicie sesión en su cuenta mediante el comando [az login](https://docs.microsoft.com/cli/azure/reference-index?view=azure-cli-latest#az-login). Tenga en cuenta la propiedad **id**, que hace referencia al **identificador de suscripción** para su cuenta de Azure. 

```azurecli-interactive
az login
```

Seleccione la suscripción específica en su cuenta mediante el comando [az account set](/cli/azure/account). Anote el valor de **id** de la salida de **az login** para usarlo como valor del argumento **subscription** del comando. Si tiene varias suscripciones, elija la suscripción adecuada en la que se debe facturar el recurso. Para obtener todas las suscripciones, use [az account list](https://docs.microsoft.com/cli/azure/account?view=azure-cli-latest#az-account-list).

```azurecli
az account set --subscription <subscription id>
```

## <a name="create-an-azure-database-for-mysql-server"></a>Creación de un servidor de Azure Database for MySQL
Cree un [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) mediante el comando [az group create](/cli/azure/group) y, después, cree un servidor MySQL en el grupo de recursos. Debe proporcionar un nombre único. En el ejemplo siguiente, se crea un grupo de recursos denominado `myresourcegroup` en la ubicación `westus`.

```azurecli-interactive
az group create --name myresourcegroup --location westus
```

Cree una Base de datos de Azure para el servidor MySQL con el comando [az mysql server create](/cli/azure/mysql/server#az-mysql-server-create). Un servidor puede contener varias bases de datos.

```azurecli
az mysql server create --resource-group myresourcegroup --name mydemoserver --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen5_2 
```

Estos son los detalles de los argumentos anteriores: 

**Configuración** | **Valor de ejemplo** | **Descripción**
---|---|---
name | mydemoserver | Escriba un nombre único para el servidor de Azure Database for MySQL. El nombre del servidor solo puede contener letras minúsculas, números y el carácter de guion (-). Debe contener entre 3 y 63 caracteres.
resource-group | myresourcegroup | Especifique el nombre del grupo de recursos de Azure.
ubicación | westus | La ubicación de Azure para el servidor.
admin-user | myadmin | El nombre del usuario del inicio de sesión del administrador. No puede ser **azure_superuser**, **admin**, **administrator**, **root**, **guest** o **public**.
admin-password | *contraseña segura* | La contraseña del usuario administrador. Debe tener entre 8 y 128 caracteres. La contraseña debe contener caracteres de tres de las siguientes categorías: Letras del alfabeto inglés mayúsculas y minúsculas, números y caracteres no alfanuméricos.
sku-name|GP_Gen5_2|Escriba el nombre del plan de tarifa y la configuración del proceso. Sigue la convención {plan de tarifa} _{generación de procesos}_ {núcleos virtuales} en forma abreviada. Para más información, consulte los [planes de tarifa](./concepts-pricing-tiers.md).

>[!IMPORTANT] 
>- La versión de MySQL predeterminada en el servidor es la 5.7. Actualmente también están disponibles las versiones 5.6 y 8.0.
>- Para ver todos los argumentos del comando **az mysql server create**, consulte este [documento de referencia](/cli/azure/mysql/server#az-mysql-server-create).
>- En el servidor, el protocolo SSL está habilitado de forma predeterminada. Para más información sobre SSL, consulte [Configuración de la conectividad de SSL](howto-configure-ssl.md)

## <a name="configure-a-server-level-firewall-rule"></a>Configuración de una regla de firewall de nivel de servidor 
De forma predeterminada, el servidor que se ha creado está protegido mediante reglas de firewall y no se puede acceder a él públicamente. Para configurar la regla de firewall en su servidor, utilice el comando [az mysql server firewall-rule create](/cli/azure/mysql/server/firewall-rule). De esta forma podrá conectar con el servidor de forma local.

En el ejemplo siguiente se crea una regla de firewall denominada `AllowMyIP` que permite las conexiones desde una dirección IP específica, 192.168.0.1. Reemplace la dirección IP desde la que se va a conectar. Puede usar un intervalo de direcciones IP si es necesario. Si no sabe cómo buscar su dirección IP, vaya a [https://whatismyipaddress.com/](https://whatismyipaddress.com/) para obtenerla.

```azurecli-interactive
az mysql server firewall-rule create --resource-group myresourcegroup --server mydemoserver --name AllowMyIP --start-ip-address 192.168.0.1 --end-ip-address 192.168.0.1
```

> [!NOTE]
> Las conexiones a Azure Database for MySQL se comunican a través del puerto 3306. Si intenta conectarse desde una red corporativa, es posible que no se permita el tráfico saliente a través del puerto 3306. En ese caso no podrá conectarse al servidor, salvo que el departamento de TI abra el puerto 3306.

## <a name="get-the-connection-information"></a>Obtención de la información de conexión

Para conectarse al servidor, debe proporcionar las credenciales de acceso y la información del host.

```azurecli-interactive
az mysql server show --resource-group myresourcegroup --name mydemoserver
```

El resultado está en formato JSON. Tome nota de los valores de **fullyQualifiedDomainName** y **administratorLogin**.
```json
{
  "administratorLogin": "myadmin",
  "earliestRestoreDate": null,
  "fullyQualifiedDomainName": "mydemoserver.mysql.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforMySQL/servers/mydemoserver",
  "location": "westus",
  "name": "mydemoserver",
  "resourceGroup": "myresourcegroup",
  "sku": {
    "capacity": 2,
    "family": "Gen5",
    "name": "GP_Gen5_2",
    "size": null,
    "tier": "GeneralPurpose"
  },
  "sslEnforcement": "Enabled",
  "storageProfile": {
    "backupRetentionDays": 7,
    "geoRedundantBackup": "Disabled",
    "storageMb": 5120
  },
  "tags": null,
  "type": "Microsoft.DBforMySQL/servers",
  "userVisibleState": "Ready",
  "version": "5.7"
}
```

## <a name="connect-to-azure-database-for-mysql-server-using-mysql-command-line-client"></a>Conexión a un servidor de Azure Database for MySQL mediante el cliente de línea de comandos de mysql
Puede conectarse al servidor mediante una herramienta cliente popular, la herramienta de línea de comandos **[mysql.exe](https://dev.mysql.com/downloads/)** con [Azure Cloud Shell](../cloud-shell/overview.md). Como alternativa, puede usar la línea de comandos de mysql en su entorno local.
```bash
 mysql -h mydemoserver.mysql.database.azure.com -u myadmin@mydemoserver -p
```

## <a name="clean-up-resources"></a>Limpieza de recursos
Si no necesita estos recursos para otra guía de inicio rápido o tutorial, puede eliminarlos con el siguiente comando: 

```azurecli-interactive
az group delete --name myresourcegroup
```

Si solo desea eliminar el servidor recién creado, puede ejecutar el comando [az mysql server delete](/cli/azure/mysql/server#az-mysql-server-delete).

```azurecli-interactive
az mysql server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
>[Compilación de una aplicación PHP en Windows con MySQL](../app-service/app-service-web-tutorial-php-mysql.md)
>[Compilación de una aplicación PHP en Linux con MySQL](../app-service/containers/tutorial-php-mysql-app.md)
>[Compilación de una aplicación Spring basada en Java con MySQL](https://docs.microsoft.com/azure/developer/java/spring-framework/spring-app-service-e2e?tabs=bash)
