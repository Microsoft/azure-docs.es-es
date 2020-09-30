---
title: 'Administración de un servidor mediante la CLI de Azure en Azure Database for PostgreSQL: Servidor flexible'
description: Obtenga información sobre cómo administrar un Servidor flexible de Azure Database for PostgreSQL desde la CLI de Azure.
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: 8f078c2c7b22a27c1fd87030361fc56d4495177c
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932585"
---
# <a name="manage-an-azure-database-for-postgresql---flexible-server-using-the-azure-cli"></a>Administración de un Servidor flexible de Azure Database for PostgreSQL mediante la CLI de Azure

> [!IMPORTANT]
> Azure Database for PostgreSQL: Servidor flexible en versión preliminar

En este artículo se muestra cómo administrar el Servidor flexible implementado en Azure. Entre las tareas de administración se incluyen el escalado de proceso y almacenamiento, el restablecimiento de contraseñas de administración y la visualización de detalles del servidor.

## <a name="prerequisites"></a>Requisitos previos
Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar. En este artículo es necesario que ejecute la versión 2.0 de la CLI de Azure, o cualquier versión posterior, de forma local. Para ver la versión instalada, ejecute el comando `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

Será preciso que inicie sesión en su cuenta mediante el comando [az login](https://docs.microsoft.com/cli/azure/reference-index#az-login). Tenga en cuenta la propiedad **id**, que hace referencia al **identificador de suscripción** para su cuenta de Azure.

```azurecli-interactive
az login
```

Seleccione la suscripción específica en su cuenta mediante el comando [az account set](/cli/azure/account). Anote el valor de **id** de la salida de **az login** para usarlo como valor del argumento **subscription** del comando. Si tiene varias suscripciones, elija la suscripción adecuada en la que se debe facturar el recurso. Para obtener todas las suscripciones, use [az account list](https://docs.microsoft.com/cli/azure/account#az-account-list).

```azurecli
az account set --subscription <subscription id>
```

> [!Important]
> Si aún no ha creado un servidor flexible, créelo para empezar a trabajar con esta guía de procedimientos.

## <a name="scale-compute-and-storage"></a>Escalado de proceso y almacenamiento

El nivel de proceso, los núcleos virtuales y el almacenamiento se pueden escalar verticalmente de forma sencilla con el siguiente comando. Puede ver toda la operación del servidor que puede ejecutar [az postgres flexible-server server overview](/cli/azure/PostgreSQL/server).

```azurecli-interactive
az postgres flexible-server update --resource-group myresourcegroup --name mydemoserver --sku-name Standard_D4ds_v3 --storage-size 6144
```

Estos son los detalles de los argumentos anteriores:

**Configuración** | **Valor de ejemplo** | **Descripción**
---|---|---
name | mydemoserver | Escriba un nombre único para el servidor. El nombre del servidor solo puede contener letras minúsculas, números y el carácter de guion (-). Debe contener entre 3 y 63 caracteres.
resource-group | myresourcegroup | Especifique el nombre del grupo de recursos de Azure.
sku-name|Standard_D4ds_v3|Escriba el nombre del nivel de proceso y el tamaño. Sigue la convención Standard_ {tamaño de máquina virtual} en abreviatura. Para más información, consulte los [planes de tarifa](../concepts-pricing-tiers.md).
storage-size | 6144 | La capacidad de almacenamiento del servidor (la unidad es megabytes). El mínimo es 5120 y aumenta en incrementos de 1024.

> [!IMPORTANT]
> El almacenamiento no se puede reducir verticalmente. 

## <a name="manage-postgresql-databases-on-a-server"></a>Administración de bases de datos de PostgreSQL en un servidor

Hay una serie de aplicaciones que se pueden usar para conectarse al servidor de Azure Database for PostgreSQL. Si el equipo cliente tiene PostgreSQL instalado, puede usar una instancia local de [psql](https://www.postgresql.org/docs/current/static/app-psql.html) para conectarse a un servidor Azure PostgreSQL. Ahora vamos a usar la utilidad de línea de comandos psql para conectarnos al servidor Azure PostgreSQL.

1. Ejecute el comando psql siguiente para conectarse a un servidor de Azure Database for PostgreSQL

   ```bash
   psql --host=<servername> --port=<port> --username=<user> --dbname=<dbname>
   ```

   Por ejemplo, el siguiente comando se conecta a la base de datos predeterminada llamada **postgres** en el servidor PostgreSQL **mydemoserver.postgres.database.azure.com** con las credenciales de acceso. Escriba el valor de `<server_admin_password>` que eligió cuando se le solicitó una contraseña.
  
   ```bash
   psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin --dbname=postgres
   ```

   Tras conectarse, la utilidad psql muestra un símbolo del sistema de postgres donde escribir comandos sql. En la salida de la conexión inicial, puede aparecer una advertencia, ya que la versión de psql que usa puede diferir de la versión del servidor de Azure Database for PostgreSQL.

   Ejemplo de salida de psql:

   ```bash
   psql (11.3, server 12.1)
   WARNING: psql major version 11, server major version 12.
            Some psql features might not work.
   SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
   Type "help" for help.

   postgres=>
   ```

   > [!TIP]
   > Si el firewall no está configurado para permitir la dirección IP de su cliente, se produce el siguiente error:
   >
   > "psql: FATAL:  no pg_hba.conf entry for host `<IP address>`, user "myadmin", database "postgres", SSL on FATAL: SSL connection is required. Specify SSL options and retry.
   >
   > Confirme que la IP de su cliente se permite en el paso anterior de las reglas de firewall.

2. Escriba el comando siguiente para crear una base de datos en blanco llamada "postgresdb" en el símbolo del sistema:

    ```bash
    CREATE DATABASE postgresdb;
    ```

3. En el símbolo del sistema, ejecute el comando siguiente para cambiar las conexiones a la base de datos **postgresdb** recientemente creada:

    ```bash
    \c postgresdb
    ```

4. Escriba `\q` y presione la tecla Entrar para salir de psql.

Ya está conectado al servidor de Azure Database for PostgreSQL a través de psql y ha creado una base de datos de usuarios en blanco.

## <a name="reset-admin-password"></a>Restablecimiento de la contraseña del administrador
La contraseña del rol Administrador se puede cambiar con este comando.
```azurecli-interactive
az postgres flexible-server update --resource-group myresourcegroup --name mydemoserver --admin-password <new-password>
```

> [!IMPORTANT]
> Asegúrese de que tenga un mínimo de 8 caracteres y un máximo de 128 caracteres de longitud.
> La contraseña debe contener caracteres de tres de las categorías siguientes: Letras del alfabeto inglés mayúsculas y minúsculas, números y caracteres no alfanuméricos.

## <a name="delete-a-server"></a>Eliminación de un servidor

Si solo quiere eliminar el Servidor flexible de PostgreSQL, puede ejecutar el comando [az postgres flexible-server delete](/cli/azure/PostgreSQL/server#az-PostgreSQL-flexible-server-delete).

```azurecli-interactive
az postgres flexible-server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Pasos siguientes

- [Descripción de los conceptos de copia de seguridad y restauración](concepts-backup-restore.md)
- [Ajuste y supervisión del servidor](concepts-monitoring.md)
