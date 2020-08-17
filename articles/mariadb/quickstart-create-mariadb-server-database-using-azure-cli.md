---
title: 'Inicio rápido: Creación de un servidor: CLI de Azure (Azure Database for MariaDB)'
description: En esta guía de inicio rápido se describe cómo usar la CLI de Azure para crear un servidor de Azure Database for MariaDB en un grupo de recursos de Azure.
author: ajlam
ms.author: andrela
ms.service: mariadb
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 3/18/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 9f20f4cced74c5607fbfea2119e22cface0a8e03
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87499065"
---
# <a name="create-an-azure-database-for-mariadb-server-by-using-the-azure-cli"></a>Creación de un servidor de Azure Database for MariaDB mediante la CLI de Azure

Puede usar la CLI de Azure para crear y administrar recursos de Azure desde la línea de comandos o en scripts. En esta guía de inicio rápido se describe cómo usar la CLI de Azure para crear un servidor de Azure Database for MariaDB en un grupo de recursos de Azure en unos cinco minutos.

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

Si va a usar e instalar la CLI de forma local para esta guía de inicio rápido, deberá ejecutar la versión 2.0 de la CLI de Azure u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure 2.0]( /cli/azure/install-azure-cli).

Si tiene varias suscripciones, elija aquella que contiene el recurso o la suscripción que se le factura. Para seleccionar un identificador de suscripción específico en su cuenta, use el comando [az account set](/cli/azure/account#az-account-set):

```azurecli-interactive
az account set --subscription 00000000-0000-0000-0000-000000000000
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un [grupo de recursos de Azure](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) con el comando [az group create](/cli/azure/group#az-group-create). Un grupo de recursos es un contenedor lógico en el que se implementan y se administran recursos de Azure como un grupo.

En el ejemplo siguiente se crea un grupo de recursos denominado `myresourcegroup` en la ubicación `westus`:

```azurecli-interactive
az group create --name myresourcegroup --location westus
```

## <a name="create-an-azure-database-for-mariadb-server"></a>Creación de un servidor de Azure Database for MariaDB

Cree un servidor de Azure Database for MariaDB con el comando [az mariadb server create](/cli/azure/mariadb/server#az-mariadb-server-create). Un servidor puede administrar varias bases de datos. Normalmente se usa una base de datos independiente para cada proyecto o para cada usuario.

Configuración | Valor de ejemplo | Descripción
---|---|---
name | **mydemoserver** | Escriba un nombre único que identifique el servidor de Azure Database for MariaDB. El nombre del servidor solo puede contener letras minúsculas, números y el carácter de guion (-). Debe contener entre 3 y 63 caracteres.
resource-group | **myresourcegroup** | Escriba el nombre del grupo de recursos de Azure.
sku-name | **GP_Gen5_2** | El nombre de la SKU. Sigue la convención *plan de tarifa*\_*generación de procesos*\_*núcleos virtuales* en forma abreviada. Para más información acerca del parámetro **sku-name**, consulte la sección que sigue a esta tabla.
backup-retention | **7** | Cuánto tiempo deben conservarse las copias de seguridad. La unidad es días. Intervalo: de 7 a 35. 
geo-redundant-backup | **Deshabilitada** | Si se deben habilitar las copias de seguridad con redundancia geográfica en este servidor o no. Valores permitidos: **Enabled** (Habilitada), **Disabled** (Deshabilitada).
ubicación | **westus** | La ubicación de Azure para el servidor.
ssl-enforcement | **Enabled** | Si se debe habilitar SSL, o no, en este servidor. Valores permitidos: **Enabled** (Habilitada), **Disabled** (Deshabilitada).
storage-size | **51200** | La capacidad de almacenamiento del servidor (la unidad es megabytes). Los tamaños válidos de almacenamiento son a partir de 5120 MB con aumentos de 1024 MB. Consulte los [planes de tarifa](./concepts-pricing-tiers.md) para más información acerca de los límites de tamaño de almacenamiento. 
version | **10.2** | La versión del motor principal de MariaDB.
admin-user | **myadmin** | El nombre del usuario del inicio de sesión del administrador. El parámetro **admin-user** no puede ser **azure_superuser**, **admin**, **administrator**, **root**, **guest** ni **public**.
admin-password | *La contraseña* | La contraseña del usuario administrador. Debe tener entre 8 y 128 caracteres. Debe contener caracteres de tres de las siguientes categorías: Letras del alfabeto inglés mayúsculas y minúsculas, números y caracteres no alfanuméricos.

El valor del parámetro sku-name sigue la convención {plan de tarifa}\_{generación de proceso}\_{núcleos virtuales} como en los ejemplos siguientes:
+ `--sku-name B_Gen5_1` se asigna a Básico, Gen 5 y 1 núcleo virtual. Esta opción es la SKU más pequeña disponible.
+ `--sku-name GP_Gen5_32` se asigna a De uso general, Gen 5 y 32 núcleos virtuales.
+ `--sku-name MO_Gen5_2` se asigna a Optimizado para memoria, Gen 5 y 2 núcleos virtuales.

Para más información acerca de los valores válidos por región y sobre los planes, consulte [planes de tarifa](./concepts-pricing-tiers.md).

En el siguiente ejemplo, se crea un servidor denominado **mydemoserver** en la región Oeste de EE. UU. El servidor está en el grupo de recursos **myresourcegroup** y tiene el inicio de sesión de administrador del servidor **myadmin**. El servidor es un servidor Gen 5 del plan de tarifa de uso general y tiene 2 núcleos virtuales. Un nombre de servidor se asigna a un nombre DNS y debe ser único en todo el mundo en Azure. Sustituya `<server_admin_password>` por su propia contraseña de administrador del servidor.

```azurecli-interactive
az mariadb server create --resource-group myresourcegroup --name mydemoserver  --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen5_2 --version 10.2
```

> [!NOTE]
> Considere la posibilidad de usar el plan de tarifa Básico si menos proceso y E/S resultan adecuados para su carga de trabajo. Tenga en cuenta que los servidores que creó en el plan de tarifa Básico no se podrán escalar más adelante a De uso general u Optimizada para memoria. Consulte la [página de precios](https://azure.microsoft.com/pricing/details/mariadb/) para más información.

## <a name="configure-a-firewall-rule"></a>Configuración de una regla de firewall

Cree una regla de firewall de nivel de servidor de Azure Database for MariaDB mediante el comando [az mariadb server firewall-rule create](/cli/azure/mariadb/server/firewall-rule#az-mariadb-server-firewall-rule-create). Una regla de firewall de nivel de servidor permite que una aplicación externa, como la herramienta de línea de comandos mysql o MySQL Workbench, se conecte al servidor a través del firewall del servicio Azure Database for MariaDB.

En el ejemplo siguiente se crea una regla de firewall denominada `AllowMyIP` que permite las conexiones desde una dirección IP específica, 192.168.0.1. Sustituya por una dirección IP o intervalo de direcciones IP que se corresponda a la ubicación desde la que se conecta.

```azurecli-interactive
az mariadb server firewall-rule create --resource-group myresourcegroup --server mydemoserver --name AllowMyIP --start-ip-address 192.168.0.1 --end-ip-address 192.168.0.1
```

> [!NOTE]
> Las conexiones a Azure Database for MariaDB se comunican mediante puerto 3306. Si intenta conectarse desde una red corporativa, es posible que no se permita el tráfico saliente a través del puerto 3306. En este caso, solo podrá conectarse al servidor si el departamento de TI abre el puerto 3306.

## <a name="configure-ssl-settings"></a>Configuración de SSL

De manera predeterminada, se exigen conexiones SSL entre su servidor y las aplicaciones cliente. Este valor predeterminado garantiza la seguridad de los datos "en movimiento" al cifrar el flujo de datos que circula por Internet. Para esta guía de inicio rápido, deshabilite las conexiones SSL del servidor. Esto no se recomienda en servidores de producción. Para más información, consulte [Configuración de la conectividad SSL en la aplicación para conectarse de forma segura a Azure Database for MariaDB](./howto-configure-ssl.md).

En el ejemplo siguiente se deshabilita la obligación de SSL en el servidor de Azure Database for MariaDB:

```azurecli-interactive
az mariadb server update --resource-group myresourcegroup --name mydemoserver --ssl-enforcement Disabled
```

## <a name="get-connection-information"></a>Obtención de información sobre la conexión

Para conectarse al servidor, debe proporcionar las credenciales de acceso y la información del host. Para obtener la información de conexión, ejecute el siguiente comando:

```azurecli-interactive
az mariadb server show --resource-group myresourcegroup --name mydemoserver
```

El resultado está en formato JSON. Tome nota de los valores de **fullyQualifiedDomainName** y **administratorLogin**.

```json
{
  "administratorLogin": "myadmin",
  "earliestRestoreDate": null,
  "fullyQualifiedDomainName": "mydemoserver.mariadb.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforMariaDB/servers/mydemoserver",
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
  "type": "Microsoft.DBforMariaDB/servers",
  "userVisibleState": "Ready",
  "version": "10.2"
}
```

## <a name="connect-to-the-server-by-using-the-mysql-command-line-tool"></a>Conexión al servidor mediante la herramienta mysql de la línea de comandos

Conéctese al servidor mediante la herramienta mysql de la línea de comandos. Puede [descargar](https://dev.mysql.com/downloads/) la herramienta de la línea de comandos e instalarla en su equipo. También puede acceder a la herramienta de línea de comandos seleccionando el botón **Pruébelo** de un ejemplo de código de este artículo. Otra manera de acceder a la herramienta de línea de comandos consiste en seleccionar el botón **>_** en la barra de herramientas superior derecha de Azure Portal para abrir **Azure Cloud Shell**.

Para conectarse al servidor mediante la herramienta mysql de la línea de comandos:

1. Conéctese al servidor:

   ```azurecli-interactive
   mysql -h mydemoserver.mariadb.database.azure.com -u myadmin@mydemoserver -p
   ```

2. Vea el estado del servidor en el símbolo del sistema de `mysql>`:

   ```sql
   status
   ```

   Debería ver algo parecido al siguiente texto:

   ```cmd
   C:\Users\>mysql -h mydemoserver.mariadb.database.azure.com -u myadmin@mydemoserver -p
   Enter password: ***********
   Welcome to the MySQL monitor.  Commands end with ; or \g.
   Your MySQL connection id is 65512
   Server version: 5.6.39.0 MariaDB Server

   Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

   Oracle is a registered trademark of Oracle Corporation and/or its
   affiliates. Other names may be trademarks of their respective
   owners.

   Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

   mysql> status
   --------------
   mysql  Ver 14.14 Distrib 5.7.23, for Linux (x86_64)

   Connection id:          64681
   Current database:
   Current user:           myadmin@40.118.201.21
   SSL:                    Cipher in use is AES256-SHA
   Current pager:          stdout
   Using outfile:          ''
   Using delimiter:        ;
   Server version:         5.6.39.0 MariaDB Server
   Protocol version:       10
   Connection:             mydemoserver.mariadb.database.azure.com via TCP/IP
   Server characterset:    latin1
   Db     characterset:    latin1
   Client characterset:    utf8
   Conn.  characterset:    utf8
   TCP port:               3306
   Uptime:                 1 day 3 hours 28 min 50 sec

   Threads: 10  Questions: 29002  Slow queries: 0  Opens: 33  Flush tables: 3  Open tables: 1  Queries per second avg: 0.293
   --------------

   mysql>
   ```

> [!TIP]
> Para otros comandos, consulte el [capítulo 4.5.1 del Manual de referencia de MySQL 5.7](https://dev.mysql.com/doc/refman/5.7/en/mysql.html).

## <a name="connect-to-the-server-by-using-mysql-workbench"></a>Conexión al servidor con MySQL Workbench

1. Abra MySQL Workbench en el equipo cliente. Si aún no está instalado, [descargue](https://dev.mysql.com/downloads/workbench/) e instale la aplicación.

2. En el cuadro de diálogo **Setup New Connection** (Configurar nueva conexión), escriba la siguiente información en la pestaña **Parameters** (Parámetros):

   ![Configuración de una nueva conexión](./media/quickstart-create-mariadb-server-database-using-azure-cli/setup-new-connection.png)

   | Configuración | Valor sugerido | Descripción |
   |---|---|---|
   | Nombre de la conexión | **Conexión de demostración** | Especifique una etiqueta para esta conexión (el nombre de la conexión puede ser cualquiera) |
   | Método de conexión | **Estándar (TCP/IP)** | Use el protocolo TCP/IP para conectarse a Azure Database for MariaDB |
   | Hostname | **mydemoserver.mariadb.database.azure.com** | El nombre del servidor que anotó anteriormente. |
   | Port | **3306** | El puerto predeterminado para Azure Database for MariaDB. |
   | Nombre de usuario | **myadmin\@mydemoserver** | El inicio de sesión de administrador del servidor que anotó anteriormente. |
   | Contraseña | *La contraseña* | Use la contraseña de la cuenta de administrador que configuró anteriormente. |

3. Para comprobar si todos los parámetros están configurados correctamente, seleccione **Test Connection** (Probar conexión).

4. Seleccione la conexión para conectarse correctamente al servidor.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no necesita los recursos que ha usado en esta guía para otra guía de inicio rápido o tutorial, puede eliminarlos con el siguiente comando: 

```azurecli-interactive
az group delete --name myresourcegroup
```

Si solo desea eliminar el servidor que ha creado en esta guía de inicio rápido, ejecute el comando [az mariadb server delete](/cli/azure/mariadb/server#az-mariadb-server-delete):

```azurecli-interactive
az mariadb server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Diseño de una base de datos de MariaDB con la CLI de Azure](./tutorial-design-database-cli.md)
