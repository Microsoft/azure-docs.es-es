---
title: 'Inicio rápido: Creación de servidor: CLI de Azure: Azure Database for PostgreSQL (servidor único)'
description: Guía de inicio rápido para crear una instancia de Azure Database for PostgreSQL (servidor único) con la CLI (interfaz de la línea de comandos) de Azure.
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 06/25/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: cb88f085e18526a17621d3c4960a5aad6db727ad
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87496600"
---
# <a name="quickstart-create-an-azure-database-for-postgresql---single-server-using-the-azure-cli"></a>Inicio rápido: Crear una instancia de Azure Database for PostgreSQL: servidor único con la CLI de Azure

> [!TIP]
> Considere la posibilidad de usar un comando de la CLI de Azure [az postgres up](/cli/azure/ext/db-up/postgres#ext-db-up-az-postgres-up) más sencillo (actualmente en versión preliminar). Pruebe el [inicio rápido](./quickstart-create-server-up-azure-cli.md).

En este inicio rápido se muestra cómo usar los comandos de la [CLI de Azure](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli) en [Azure Cloud Shell](https://shell.azure.com) para crear un servidor de Azure Database for PostgreSQL en cinco minutos.  Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

>[!Note]
>Si decide instalar y usar la CLI localmente, para este artículo es preciso que ejecute la versión 2.0 o posterior de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest). 

## <a name="prerequisites"></a>Requisitos previos
En este artículo es necesario que ejecute la versión 2.0 de la CLI de Azure, o cualquier versión posterior, de forma local. Para ver la versión instalada, ejecute el comando `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

Será preciso que inicie sesión en su cuenta mediante el comando [az login](https://docs.microsoft.com/cli/azure/reference-index?view=azure-cli-latest#az-login). Tenga en cuenta la propiedad **id**, que hace referencia a **Id. de suscripción** para su cuenta de Azure. 

```azurecli-interactive
az login
```

Seleccione el identificador de suscripción específico en su cuenta mediante el comando [az account set](/cli/azure/account). Anote el valor de **id** de la salida de **az login** para usarlo como valor del argumento **subscription** del comando. Si tiene varias suscripciones, elija la suscripción adecuada en la que se debe facturar el recurso. Para obtener todas las suscripciones, use [az account list](https://docs.microsoft.com/cli/azure/account?view=azure-cli-latest#az-account-list).

```azurecli
az account set --subscription <subscription id>
```
## <a name="create-an-azure-database-for-postgresql-server"></a>Creación de un servidor de Azure Database for PostgreSQL

Cree un [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) mediante el comando [az group create](https://docs.microsoft.com/cli/azure/group?view=azure-cli-latest#az-group-create) y, después, cree un servidor PostgreSQL en el grupo de recursos. Debe proporcionar un nombre único. En el ejemplo siguiente, se crea un grupo de recursos denominado `myresourcegroup` en la ubicación `westus`.
```azurecli-interactive
az group create --name myresourcegroup --location westus
```

Cree un [servidor de Azure Database for PostgreSQL](overview.md) con el comando [az postgres server create](/cli/azure/postgres/server). Un servidor puede contener varias bases de datos.

```azurecli-interactive
az postgres server create --resource-group myresourcegroup --name mydemoserver  --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen5_2 
```
Estos son los datos de los argumentos anteriores: 

**Configuración** | **Valor de ejemplo** | **Descripción**
---|---|---
name | mydemoserver | Elija un nombre único que identifique al servidor de Azure Database for PostgreSQL. El nombre del servidor solo puede contener letras minúsculas, números y el carácter de guion (-). Debe contener entre 3 y 63 caracteres.
resource-group | myresourcegroup | Especifique el nombre del grupo de recursos de Azure.
ubicación | westus | La ubicación de Azure para el servidor.
admin-user | myadmin | El nombre del usuario del inicio de sesión del administrador. No puede ser **azure_superuser**, **admin**, **administrator**, **root**, **guest** o **public**.
admin-password | *contraseña segura* | La contraseña del usuario administrador. Debe tener entre 8 y 128 caracteres. La contraseña debe contener caracteres de tres de las siguientes categorías: Letras del alfabeto inglés mayúsculas y minúsculas, números y caracteres no alfanuméricos.
sku-name|GP_Gen5_2|Escriba el nombre del plan de tarifa y la configuración del proceso. Sigue la convención {plan de tarifa} _{generación de procesos}_ {núcleos virtuales} en forma abreviada. Para más información, consulte el artículo [Azure Database for PostgreSQL](https://azure.microsoft.com/pricing/details/postgresql/server/).

>[!IMPORTANT] 
>- La versión de PostgreSQL predeterminada en el servidor es la 9.6. [Aquí](https://docs.microsoft.com/azure/postgresql/concepts-supported-versions) puede ver todas las versiones admitidas.
>- Para ver todos los argumentos del comando **az postgres server create**, consulte este [documento de referencia](https://docs.microsoft.com/cli/azure/postgres/server?view=azure-cli-latest#az-postgres-server-create)
>- En el servidor, el protocolo SSL está habilitado de manera predeterminada. Para más información sobre SSL, consulte [Configuración de la conectividad de SSL](./concepts-ssl-connection-security.md)

## <a name="configure-a-server-level-firewall-rule"></a>Configuración de una regla de firewall de nivel de servidor 
De forma predeterminada, no se puede acceder públicamente al servidor creado, que está protegido con reglas de firewall. Puede configurar la regla de firewall en el servidor mediante el comando [az postgres server firewall-rule create](/cli/azure/postgres/server/firewall-rule) para dar acceso al entorno local para conectarse al servidor. 

En el ejemplo siguiente se crea una regla de firewall denominada `AllowMyIP` que permite las conexiones desde una dirección IP específica, 192.168.0.1. Reemplace la dirección IP o el intervalo de direcciones IP correspondientes al lugar desde el que se va a conectar.  Si no sabe cómo buscar su IP, vaya a [https://whatismyipaddress.com/](https://whatismyipaddress.com/) para obtener la dirección IP.


```azurecli-interactive
az postgres server firewall-rule create --resource-group myresourcegroup --server mydemoserver --name AllowMyIP --start-ip-address 192.168.0.1 --end-ip-address 192.168.0.1
```

> [!NOTE]
>  Asegúrese de que el firewall de la red permite el puerto 5432, que es el que utilizan los servidores de Azure Database for PostgreSQL para evitar problemas de conectividad. 

## <a name="get-the-connection-information"></a>Obtención de la información de conexión

Para conectarse al servidor, debe proporcionar las credenciales de acceso y la información del host.
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

## <a name="connect-to-azure-database-for-postgresql-server-using-psql"></a>Conexión a Azure Database for PostgreSQL mediante psql
[**psql**](https://www.postgresql.org/docs/current/static/app-psql.html) es un cliente que se usa frecuentemente para conectarse a servidores de PostgreSQL. Puede conectarse a un servidor mediante **psql** con [Azure Cloud Shell](../cloud-shell/overview.md). Como alternativa, puede usar psql en su entorno local, siempre que esté disponible. Una base de datos vacía, "postgres", se ha creado con el nuevo servidor de PostgreSQL y se puede usar para conectarse con psql, como se muestra a continuación 

   ```bash
 psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin@mydemoserver --dbname=postgres
   ```

   > [!TIP]
   > Si prefiere usar una ruta URL para conectarse a Postgres, codifique la URL con el signo @ en el nombre de usuario con `%40`. Por ejemplo, la cadena de conexión de psql sería:
   > ```
   > psql postgresql://myadmin%40mydemoserver@mydemoserver.postgres.database.azure.com:5432/postgres
   > ```


## <a name="clean-up-resources"></a>Limpieza de recursos
Si no necesita estos recursos para otro inicio rápido o tutorial, se pueden eliminar. Para ello, debe ejecutar el siguiente comando: 

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

