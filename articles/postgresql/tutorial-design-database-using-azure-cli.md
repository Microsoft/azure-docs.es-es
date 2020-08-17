---
title: 'Tutorial: Diseño de un servidor único de Azure Database for PostgreSQL con la CLI de Azure'
description: 'En este tutorial se muestra cómo crear, configurar y consultar su primera instancia de Azure Database for PostgreSQL: servidor único con la CLI de Azure.'
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.custom: mvc, devx-track-azurecli
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 06/25/2019
ms.openlocfilehash: c1ab057c34a89bfa39a298805216af89b2327622
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87500783"
---
# <a name="tutorial-design-an-azure-database-for-postgresql---single-server-using-azure-cli"></a>Tutorial: Diseño de una instancia de Azure Database for PostgreSQL: servidor único con la CLI de Azure 
En este tutorial, usa la CLI (interfaz de la línea de comandos) de Azure y otras utilidades para aprender a hacer lo siguiente:
> [!div class="checklist"]
> * Creación de un servidor de Azure Database for PostgreSQL
> * Configuración del firewall del servidor
> * Uso de la utilidad [**psql**](https://www.postgresql.org/docs/9.6/static/app-psql.html) para crear una base de datos
> * Carga de datos de muestra
> * Consultar datos
> * Actualización de datos
> * Restauración de datos

Puede usar Azure Cloud Shell en el explorador, o bien [instalar la CLI de Azure]( /cli/azure/install-azure-cli) en su propio equipo para ejecutar los comandos de este tutorial.

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para este artículo es preciso que ejecute la versión 2.0 o posterior de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli). 

Si tiene varias suscripciones, elija la adecuada donde se encuentre el recurso o para la cual se facture. Seleccione un identificador de suscripción específico en su cuenta mediante el comando [az account set](/cli/azure/account).
```azurecli-interactive
az account set --subscription 00000000-0000-0000-0000-000000000000
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos
Cree un [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) con el comando [az group create](/cli/azure/group). Un grupo de recursos es un contenedor lógico en el que se implementan y se administran recursos de Azure como un grupo. En el ejemplo siguiente, se crea un grupo de recursos denominado `myresourcegroup` en la ubicación `westus`.
```azurecli-interactive
az group create --name myresourcegroup --location westus
```

## <a name="create-an-azure-database-for-postgresql-server"></a>Creación de un servidor de Azure Database for PostgreSQL
Cree un [servidor de Azure Database for PostgreSQL](overview.md) con el comando [az postgres server create](/cli/azure/postgres/server). Un servidor contiene un conjunto de bases de datos administradas como un grupo. 

En el ejemplo siguiente se crea un servidor denominado `mydemoserver` en el grupo de recursos `myresourcegroup` con el inicio de sesión de administrador de servidor `myadmin`. El nombre de un servidor se asigna al nombre DNS y, por tanto, debe ser único a nivel global en Azure. Sustituya `<server_admin_password>` por su propio valor. Se trata de un servidor Gen 5 de uso general con 2 núcleos virtuales.
```azurecli-interactive
az postgres server create --resource-group myresourcegroup --name mydemoserver --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen5_2 --version 9.6
```
El valor del parámetro sku-name sigue la convención {plan de tarifa}\_{generación de proceso}\_{núcleos virtuales} como en los ejemplos siguientes:
+ `--sku-name B_Gen5_2` se asigna a Básico, Gen 5 y 2 núcleos virtuales.
+ `--sku-name GP_Gen5_32` se asigna a De uso general, Gen 5 y 32 núcleos virtuales.
+ `--sku-name MO_Gen5_2` se asigna a Optimizado para memoria, Gen 5 y 2 núcleos virtuales.

Para comprender cuáles son los valores válidos por región y nivel consulte la documentación sobre [planes de tarifa](./concepts-pricing-tiers.md).

> [!IMPORTANT]
> El inicio de sesión y la contraseña de administrador del servidor que especifique aquí serán necesarios para iniciar sesión más adelante en ese servidor y en las bases de datos que se especificarán en esta guía de inicio rápido. Recuerde o grabe esta información para su uso posterior.

De forma predeterminada, la base de datos de **postgres** se crea en el servidor. La base de datos [postgres](https://www.postgresql.org/docs/9.6/static/app-initdb.html) es una base de datos predeterminada pensada para que la usen los usuarios, las utilidades y aplicaciones de otros fabricantes. 


## <a name="configure-a-server-level-firewall-rule"></a>Configuración de una regla de firewall de nivel de servidor

Cree una regla de firewall de nivel de servidor de Azure PostgreSQL con el comando [az postgres server firewall-rule create](/cli/azure/postgres/server/firewall-rule). Una regla de firewall de nivel de servidor permite que una aplicación externa, como [psql](https://www.postgresql.org/docs/9.2/static/app-psql.html) o [PgAdmin](https://www.pgadmin.org/), se conecte al servidor a través del firewall del servicio Azure PostgreSQL. 

Puede establecer una regla de firewall que abarque un intervalo de IP para poder conectarse desde la red. En el ejemplo siguiente se usa [az postgres server firewall-rule create](/cli/azure/postgres/server/firewall-rule) para crear una regla de firewall `AllowMyIP` que permite la conexión desde una única dirección IP.

```azurecli-interactive
az postgres server firewall-rule create --resource-group myresourcegroup --server mydemoserver --name AllowMyIP --start-ip-address 192.168.0.1 --end-ip-address 192.168.0.1
```

Para restringir el acceso a su servidor de Azure PostgreSQL solamente a su red, puede establecer la regla de firewall para que solo cubra el intervalo de direcciones IP de su red corporativa.

> [!NOTE]
> El servidor Azure PostgreSQL se comunica a través de puerto 5432. Al conectarse desde una red corporativa, es posible que el firewall de la red no permita el tráfico saliente a través del puerto 5432. Haga que el departamento de TI abra el puerto 5432 para conectarse al servidor de Azure SQL Database.
>

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

## <a name="connect-to-azure-database-for-postgresql-database-using-psql"></a>Conexión a la base de datos de Azure Database for PostgreSQL mediante psql
Si el equipo cliente tiene PostgreSQL instalado, puede usar una instancia local de [psql](https://www.postgresql.org/docs/9.6/static/app-psql.html) o usar la consola de Azure Cloud para conectarse a un servidor Azure PostgreSQL. Ahora vamos a usar la utilidad de línea de comandos psql para conectarnos al servidor de Azure Database for PostgreSQL.

1. Ejecute el comando psql siguiente para conectarse a una base de datos Azure Database for PostgreSQL:
   ```
   psql --host=<servername> --port=<port> --username=<user@servername> --dbname=<dbname>
   ```

   Por ejemplo, el siguiente comando se conecta a la base de datos predeterminada llamada **postgres** en el servidor PostgreSQL **mydemoserver.postgres.database.azure.com** con las credenciales de acceso. Escriba el valor de `<server_admin_password>` que eligió cuando se le solicitó una contraseña.
  
   ```
   psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin@mydemoserver --dbname=postgres
   ```

   > [!TIP]
   > Si prefiere usar una ruta URL para conectarse a Postgres, codifique la URL con el signo @ en el nombre de usuario con `%40`. Por ejemplo, la cadena de conexión de psql sería:
   > ```
   > psql postgresql://myadmin%40mydemoserver@mydemoserver.postgres.database.azure.com:5432/postgres
   > ```

2. Una vez conectado al servidor, cree una base de datos vacía en el símbolo del sistema:
   ```sql
   CREATE DATABASE mypgsqldb;
   ```

3. En el símbolo del sistema, ejecute el siguiente comando para cambiar la conexión a la base de datos **mypgsqldb** recién creada:
   ```sql
   \c mypgsqldb
   ```

## <a name="create-tables-in-the-database"></a>Creación de tablas en la base de datos
Ahora que sabe cómo conectarse a la base de datos de Azure Database for PostgreSQL, puede completar algunas tareas básicas:

En primer lugar, cree una tabla y cárguela con algunos datos. Por ejemplo, vamos a crear una tabla que haga un seguimiento de la información del inventario:
```sql
CREATE TABLE inventory (
    id serial PRIMARY KEY, 
    name VARCHAR(50), 
    quantity INTEGER
);
```

Puede ver la tabla recién creada ahora en la lista de tablas si escribe lo siguiente:
```sql
\dt
```

## <a name="load-data-into-the-table"></a>Carga de datos en la tabla
Ahora que ya hay una tabla creada, inserte algunos datos en ella. En la ventana de símbolo del sistema abierta, ejecute la consulta siguiente para insertar algunas filas de datos:
```sql
INSERT INTO inventory (id, name, quantity) VALUES (1, 'banana', 150); 
INSERT INTO inventory (id, name, quantity) VALUES (2, 'orange', 154);
```

Ahora ha agregado dos filas de datos de ejemplo en la tabla que creó anteriormente.

## <a name="query-and-update-the-data-in-the-tables"></a>Consulta y actualización de los datos en las tablas
Ejecute la siguiente consulta para recuperar información de la tabla del inventario: 
```sql
SELECT * FROM inventory;
```

También puede actualizar los datos en la tabla del inventario:
```sql
UPDATE inventory SET quantity = 200 WHERE name = 'banana';
```

Puede ver los valores actualizados cuando recupera los datos:
```sql
SELECT * FROM inventory;
```

## <a name="restore-a-database-to-a-previous-point-in-time"></a>Restauración de una base de datos a un momento anterior en el tiempo
Imagine que ha eliminado accidentalmente una tabla. No se puede recuperar con facilidad. Azure Database for PostgreSQL permite volver a cualquier momento dado para el cual haya copias de seguridad del servidor (en función del período de retención de copia de seguridad que se configuró) y restaurar este momento dado en un nuevo servidor. Puede usar este servidor nuevo para recuperar los datos eliminados. 

El comando siguiente restaura el servidor de ejemplo a un punto antes de que se agregara la tabla:
```azurecli-interactive
az postgres server restore --resource-group myresourcegroup --name mydemoserver-restored --restore-point-in-time 2017-04-13T13:59:00Z --source-server mydemoserver
```

El comando `az postgres server restore` necesita los parámetros siguientes:

| Configuración | Valor sugerido | Descripción  |
| --- | --- | --- |
| resource-group |  myresourcegroup |  Grupo de recursos en el que existe el servidor de origen.  |
| name | mydemoserver-restored | Nombre del nuevo servidor que se crea mediante el comando de restauración. |
| restore-point-in-time | 2017-04-13T13:59:00Z | Seleccione un momento dado en el que quiere restaurar. Esta fecha y hora debe estar dentro del período de retención de copia de seguridad del servidor de origen. Use el formato de fecha y hora ISO8601. Por ejemplo, puede usar su propia zona horaria local, como `2017-04-13T05:59:00-08:00`, o usar el formato de hora Zulú UTC `2017-04-13T13:59:00Z`. |
| source-server | mydemoserver | Nombre o identificador del servidor de origen desde el que se va a restaurar. |

Al restaurar un servidor a un momento dado, se crea un servidor que se copia como servidor original a un momento dado que especifique. Los valores de ubicación y plan de tarifa del servidor restaurado son los mismos que los del servidor de origen.

El comando es sincrónico y se devolverá después de que se haya restaurado el servidor. Una vez finalizada la restauración, busque el servidor que se ha creado. Compruebe que los datos se han restaurado del modo esperado.


## <a name="next-steps"></a>Pasos siguientes
En este tutorial, aprendió a usar la CLI (interfaz de la línea de comandos) de Azure y otras utilidades para hacer lo siguiente:
> [!div class="checklist"]
> * Creación de un servidor de Azure Database for PostgreSQL
> * Configuración del firewall del servidor
> * Uso de la utilidad [**psql**](https://www.postgresql.org/docs/9.6/static/app-psql.html) para crear una base de datos
> * Carga de datos de muestra
> * Consultar datos
> * Actualización de datos
> * Restauración de datos

Después, para aprender a usar Azure Portal para realizar tareas similares, consulte este tutorial: [Diseño de la primera base de datos de Azure Database for PostgreSQL con Azure Portal](tutorial-design-database-using-azure-portal.md)
