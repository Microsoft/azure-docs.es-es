---
title: 'Inicio rápido: Creación de un servidor (CLI de Azure): Azure Database for PostgreSQL (servidor único)'
description: En esta guía de inicio rápido va a crear un servidor de Azure Database for PostgreSQL mediante la CLI de Azure.
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 06/25/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 182226a0bb97c9239f1c0d0dc6e7d4d2b8bdbb1c
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88798880"
---
# <a name="quickstart-create-an-azure-database-for-postgresql-server-by-using-the-azure-cli"></a>Inicio rápido: Creación de un servidor de Azure Database for PostgreSQL mediante la CLI de Azure

En este inicio rápido se muestra cómo usar los comandos de la [CLI de Azure](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli) en [Azure Cloud Shell](https://shell.azure.com) para crear un servidor de Azure Database for PostgreSQL único en cinco minutos. Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

> [!TIP]
> Considere la posibilidad de usar el comando de la CLI de Azure [az postgres up](/cli/azure/ext/db-up/postgres#ext-db-up-az-postgres-up), que es más sencillo y actualmente está en versión preliminar. Pruebe el [inicio rápido](./quickstart-create-server-up-azure-cli.md).

## <a name="prerequisites"></a>Requisitos previos
En este artículo es preciso que ejecute la versión 2.0 de la CLI de Azure, o cualquier versión posterior, en un entorno local. Para ver la versión instalada, ejecute el comando `az --version`. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

Es preciso que inicie sesión en su cuenta mediante el comando [az login](https://docs.microsoft.com/cli/azure/reference-index?view=azure-cli-latest#az-login). Tenga en cuenta la propiedad **id**, que hace referencia al **identificador de suscripción** de su cuenta de Azure. 

```azurecli-interactive
az login
```

Seleccione el identificador de suscripción específico en su cuenta mediante el comando [az account set](/cli/azure/account). Anote el valor de **id** de la salida de **az login** para usarlo como valor del argumento **subscription** del comando. 

```azurecli
az account set --subscription <subscription id>
```

Si tiene varias suscripciones, elija la suscripción adecuada en la que se debe facturar el recurso. Para obtener todas las suscripciones, use [az account list](https://docs.microsoft.com/cli/azure/account?view=azure-cli-latest#az-account-list).

## <a name="create-an-azure-database-for-postgresql-server"></a>Creación de un servidor de Azure Database for PostgreSQL

Cree un [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) mediante el comando [az group create](https://docs.microsoft.com/cli/azure/group?view=azure-cli-latest#az-group-create) y, después, cree un servidor PostgreSQL en el grupo de recursos. Debe proporcionar un nombre único. En el ejemplo siguiente, se crea un grupo de recursos denominado `myresourcegroup` en la ubicación `westus`.

```azurecli-interactive
az group create --name myresourcegroup --location westus
```

Cree un [servidor de Azure Database for PostgreSQL](overview.md) mediante el comando [az postgres server create](/cli/azure/postgres/server). Un servidor puede contener varias bases de datos.

```azurecli-interactive
az postgres server create --resource-group myresourcegroup --name mydemoserver  --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen5_2 
```
Estos son los detalles de los argumentos anteriores: 

**Configuración** | **Valor de ejemplo** | **Descripción**
---|---|---
name | mydemoserver | Nombre único que identifica al servidor de Azure Database for PostgreSQL. El nombre del servidor solo puede contener letras minúsculas, números y el carácter de guion (-). Debe contener entre 3 y 63 caracteres.
resource-group | myresourcegroup | Nombre del grupo de recursos de Azure.
ubicación | westus | Ubicación de Azure para el servidor.
admin-user | myadmin | Nombre del usuario del inicio de sesión de administrador. No pueden ser **azure_superuser**, **admin**, **administrator**, **root**, **guest** o **public**.
admin-password | *contraseña segura* | Contraseña del usuario administrador. Debe contener entre 8 y 128 caracteres de tres de las siguientes categorías: Letras del alfabeto inglés mayúsculas y minúsculas, números y caracteres no alfanuméricos.
sku-name|GP_Gen5_2| Nombre del plan de tarifa y la configuración del proceso. Siga la convención {plan de tarifa} _{generación de procesos}_ {núcleos virtuales} en forma abreviada. Para más información, consulte los [precios de Azure Database for PostgreSQL](https://azure.microsoft.com/pricing/details/postgresql/server/).

>[!IMPORTANT] 
>- La versión de PostgreSQL predeterminada en el servidor es la 9.6. Para ver todas versiones compatibles, consulte [Versiones principales de PostgreSQL admitidas](https://docs.microsoft.com/azure/postgresql/concepts-supported-versions).
>- Para ver todos los argumentos del comando **az postgres server create**, consulte este [documento de referencia](https://docs.microsoft.com/cli/azure/postgres/server?view=azure-cli-latest#az-postgres-server-create).
>- En el servidor, el protocolo SSL está habilitado de manera predeterminada. Para más información sobre SSL, consulte [Configuración de la conectividad de SSL](./concepts-ssl-connection-security.md).

## <a name="configure-a-server-level-firewall-rule"></a>Configuración de una regla de firewall de nivel de servidor 
De forma predeterminada, no se puede acceder públicamente al servidor que ha creado y está protegido con reglas de firewall. Puede configurar la regla de firewall en el servidor mediante el comando [az postgres server firewall-rule create](/cli/azure/postgres/server/firewall-rule) para dar acceso al entorno local para conectarse al servidor. 

En el ejemplo siguiente se crea una regla de firewall denominada `AllowMyIP` que permite las conexiones desde una dirección IP específica, 192.168.0.1. Reemplace la dirección IP o el intervalo de direcciones IP correspondientes al lugar desde el que se va a conectar. Si no conoce su dirección IP, vaya a [WhatIsMyIPAddress.com](https://whatismyipaddress.com/) para obtenerla.


```azurecli-interactive
az postgres server firewall-rule create --resource-group myresourcegroup --server mydemoserver --name AllowMyIP --start-ip-address 192.168.0.1 --end-ip-address 192.168.0.1
```

> [!NOTE]
> Para evitar problemas de conectividad, asegúrese de que el firewall de la red permite el puerto 5432. Los servidores de Azure Database for PostgreSQL usan ese puerto. 

## <a name="get-the-connection-information"></a>Obtención de la información de conexión

Para conectarse a su servidor, especifique las credenciales de acceso y la información del host.

```azurecli-interactive
az postgres server show --resource-group myresourcegroup --name mydemoserver
```

El resultado está en formato JSON. Tome nota de los valores de **administratorLogin** y **fullyQualifiedDomainName**.

```json
{
  "administratorLogin": "myadmin",
  "earliestRestoreDate": null,
  "fullyQualifiedDomainName": "mydemoserver.postgres.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforPostgreSQL/servers/mydemoserver",
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
  "type": "Microsoft.DBforPostgreSQL/servers",
  "userVisibleState": "Ready",
  "version": "9.6"
}
```

## <a name="connect-to-the-azure-database-for-postgresql-server-by-using-psql"></a>Conexión al servidor de Azure Database for PostgreSQL mediante psql
El cliente de [psql](https://www.postgresql.org/docs/current/static/app-psql.html) es una opción que se usa con frecuencia para conectarse a servidores PostgreSQL. Puede conectarse a un servidor mediante psql con [Azure Cloud Shell](../cloud-shell/overview.md). También puede usar psql en su entorno local, en caso de que esté disponible. Se crea automáticamente una base de datos vacía, **postgres**, con cada servidor PostgreSQL nuevo. Esa base de datos se puede usar para conectarse a psql, como se muestra en el código siguiente. 

   ```bash
 psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin@mydemoserver --dbname=postgres
   ```

> [!TIP]
> Si prefiere usar una ruta URL para conectarse a Postgres, codifique la URL con el signo @ en el nombre de usuario con `%40`. Por ejemplo, la cadena de conexión de psql sería:
>
> ```
> psql postgresql://myadmin%40mydemoserver@mydemoserver.postgres.database.azure.com:5432/postgres
> ```


## <a name="clean-up-resources"></a>Limpieza de recursos
Si no necesita estos recursos para otro inicio rápido o tutorial, se pueden eliminar. Para ello, debe ejecutar el siguiente comando. 

```azurecli-interactive
az group delete --name myresourcegroup
```

Si solo desea eliminar el servidor recién creado, puede ejecutar el comando [az postgres server delete](/cli/azure/postgres/server).

```azurecli-interactive
az postgres server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Migración de una base de datos mediante exportación e importación](./howto-migrate-using-export-and-import.md)
> 
> [Implementación de una aplicación web Django con PostgreSQL](../app-service/containers/tutorial-python-postgresql-app.md)
>
> [Conexión con una aplicación de Node.js](./connect-nodejs.md)

