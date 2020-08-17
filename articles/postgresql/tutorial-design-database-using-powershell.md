---
title: 'Tutorial: Diseño de una instancia de Azure Database for PostgreSQL: servidor único con Azure PowerShell'
description: 'En este tutorial se muestra cómo crear, configurar y consultar su primera instancia de Azure Database for PostgreSQL: servidor único con Azure PowerShell.'
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.devlang: azurepowershell
ms.topic: tutorial
ms.date: 06/08/2020
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: b424213a1041c426653e5cb455615111c265746a
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87500068"
---
# <a name="tutorial-design-an-azure-database-for-postgresql---single-server-using-powershell"></a>Tutorial: Diseño de una instancia de Azure Database for PostgreSQL: servidor único con PowerShell

Azure Database for PostgreSQL es un servicio de base de datos relacional de la nube de Microsoft basado en el motor de base de datos de PostgreSQL Community Edition. En este tutorial, usará PowerShell y otras utilidades para aprender lo siguiente:

> [!div class="checklist"]
> - Creación de una instancia de Azure Database for PostgreSQL
> - Configuración del firewall del servidor
> - Uso de la utilidad [**psql**](https://www.postgresql.org/docs/9.6/static/app-psql.html) para crear una base de datos
> - Carga de datos de muestra
> - Consultar datos
> - Actualización de datos
> - Restauración de datos

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

Si decide usar PowerShell de forma local, para este artículo es preciso que instale el módulo Az PowerShell y que se conecte a su cuenta de Azure con el cmdlet [Connect-AzAccount](https://docs.microsoft.com/powershell/module/az.accounts/connect-azaccount). Para más información sobre cómo instalar el módulo Az PowerShell, consulte [Instalación de Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps).

> [!IMPORTANT]
> Mientras el módulo Az.PostgreSql PowerShell se encuentre en versión preliminar, debe instalarlo por separado del módulo Az PowerShell con el siguiente comando: `Install-Module -Name Az.PostgreSql -AllowPrerelease`.
> Una vez que el módulo Az.PostgreSql PowerShell esté disponible con carácter general, formará parte de las futuras versiones del módulo Az PowerShell y estará disponible de forma nativa en Azure Cloud Shell.

Si esta es la primera vez que usa el servicio Azure Database for PostgreSQL, debe registrar el proveedor de recursos **Microsoft.DBforPostgreSQL**.

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.DBforPostgreSQL
```

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

Si tiene varias suscripciones a Azure, elija la suscripción adecuada en la que se debe facturar el recurso. Seleccione un identificador de suscripción específico con el cmdlet [Set-AzContext](https://docs.microsoft.com/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un [grupo de recursos de Azure](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) con el cmdlet [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup). Un grupo de recursos es un contenedor lógico en el que se implementan y se administran recursos de Azure como un grupo.

En el ejemplo siguiente, se crea un grupo de recursos denominado **myResourceGroup** en la región **Oeste de EE. UU.**

```azurepowershell-interactive
New-AzResourceGroup -Name myresourcegroup -Location westus
```

## <a name="create-an-azure-database-for-postgresql-server"></a>Creación de un servidor de Azure Database for PostgreSQL

Cree un servidor Azure Database for PostgreSQL con el cmdlet `New-AzPostgreSqlServer`. Un servidor puede administrar varias bases de datos. Normalmente se usa una base de datos independiente para cada proyecto o para cada usuario.

En el ejemplo siguiente se crea un servidor PostgreSQL en la región **Oeste de EE. UU.** denominado **mydemoserver** en el grupo de recursos **myresourcegroup** con el inicio de sesión del administrador del servidor de **myadmin**. Es un servidor de generación 5 en el plan de tarifa de uso general con dos núcleos virtuales y con copias de seguridad con redundancia geográfica habilitadas. Documente la contraseña usada en la primera línea del ejemplo, ya que se trata de la contraseña de la cuenta de administrador del servidor PostgreSQL.

> [!TIP]
> Un nombre de servidor se asigna a un nombre DNS y debe ser único en todo el mundo en Azure.

```azurepowershell-interactive
$Password = Read-Host -Prompt 'Please enter your password' -AsSecureString
New-AzPostgreSqlServer -Name mydemoserver -ResourceGroupName myresourcegroup -Sku GP_Gen5_2 -GeoRedundantBackup Enabled -Location westus -AdministratorUsername myadmin -AdministratorLoginPassword $Password
```

El valor del parámetro **Sku** sigue la convención **plan de tarifa\_generación de proceso\_núcleos virtuales** como en los ejemplos siguientes.

- `-Sku B_Gen5_1` se asigna a Básico, Gen 5 y 1 núcleo virtual. Esta opción es la SKU más pequeña disponible.
- `-Sku GP_Gen5_32` se asigna a De uso general, Gen 5 y 32 núcleos virtuales.
- `-Sku MO_Gen5_2` se asigna a Optimizado para memoria, Gen 5 y 2 núcleos virtuales.

Para más información sobre los valores válidos de **Sku** por región y sobre los planes, consulte [Planes de tarifa de Azure Database for PostgreSQL](./concepts-pricing-tiers.md).

Considere la posibilidad de usar el plan de tarifa básico si para su carga de trabajo es conveniente menos proceso y E/S.

> [!IMPORTANT]
> Los servidores que creó en el plan de tarifa Básico no se podrán escalar más adelante a servidores de uso general u optimizados para memoria y no se pueden replicar geográficamente.

## <a name="configure-a-firewall-rule"></a>Configuración de una regla de firewall

Cree una regla de firewall de nivel de servidor de Azure Database for PostgreSQL con el cmdlet `New-AzPostgreSqlFirewallRule`. Una regla de firewall de nivel de servidor permite que una aplicación externa, como la herramienta de línea de comandos `psql` o PostgreSQL Workbench, se conecte al servidor mediante el firewall del servicio Azure Database for PostgreSQL.

En el ejemplo siguiente se crea una regla de firewall denominada **AllowMyIP** que permite las conexiones desde una dirección IP específica, 192.168.0.1. Sustituya por una dirección IP o intervalo de direcciones IP que se corresponda a la ubicación desde la que se conecta.

```azurepowershell-interactive
New-AzPostgreSqlFirewallRule -Name AllowMyIP -ResourceGroupName myresourcegroup -ServerName mydemoserver -StartIPAddress 192.168.0.1 -EndIPAddress 192.168.0.1
```

> [!NOTE]
> Las conexiones a Azure Database for PostgreSQL se comunican a través del puerto 3306. Si intenta conectarse desde una red corporativa, es posible que no se permita el tráfico saliente a través del puerto 3306. En este escenario, solo podrá conectarse al servidor si el departamento de TI abre el puerto 3306.

## <a name="get-the-connection-information"></a>Obtención de la información de conexión

Para conectarse al servidor, debe proporcionar las credenciales de acceso y la información del host. Use el ejemplo siguiente para determinar la información de conexión. Tome nota de los valores de **FullyQualifiedDomainName** y **AdministratorLogin**.

```azurepowershell-interactive
Get-AzPostgreSqlServer -Name mydemoserver -ResourceGroupName myresourcegroup |
  Select-Object -Property FullyQualifiedDomainName, AdministratorLogin
```

```Output
FullyQualifiedDomainName                    AdministratorLogin
------------------------                    ------------------
mydemoserver.postgresql.database.azure.com       myadmin
```

## <a name="connect-to-postgresql-database-using-psql"></a>Conectarse a la base de datos de PostgreSQL mediante psql

Si el equipo cliente tiene PostgreSQL instalado, puede usar una instancia local de [psql](https://www.postgresql.org/docs/current/static/app-psql.html) para conectarse a un servidor Azure PostgreSQL. También puede acceder a una versión preinstalada de la herramienta de línea de comandos `psql` en Azure Cloud Shell si selección el botón **Pruébelo** en un código de ejemplo de este artículo. Otras maneras de acceder a Azure Cloud Shell consisten en seleccionar el botón **>_** en la barra de herramientas superior derecha de Azure Portal o visitar [shell.azure.com](https://shell.azure.com/).

1. Conéctese al servidor Azure PostgreSQL mediante la utilidad de línea de comandos `psql`.

   ```azurepowershell-interactive
   psql --host=<servername> --port=<port> --username=<user@servername> --dbname=<dbname>
   ```

   Por ejemplo, el siguiente comando se conecta a la base de datos predeterminada denominada **postgres** en el servidor PostgreSQL `mydemoserver.postgres.database.azure.com` con las credenciales de acceso. Escriba el valor de `<server_admin_password>` que eligió cuando se le solicitó una contraseña.

   ```azurepowershell-interactive
   psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin@mydemoserver --dbname=postgres
   ```

   > [!TIP]
   > Si prefiere usar una ruta URL para conectarse a Postgres, codifique la URL con el signo @ en el nombre de usuario con `%40`. Por ejemplo, la cadena de conexión de psql sería `psql postgresql://myadmin%40mydemoserver@mydemoserver.postgres.database.azure.com:5432/postgres`.

1. Una vez conectado al servidor, cree una base de datos vacía en el símbolo del sistema.

   ```sql
   CREATE DATABASE mypgsqldb;
   ```

1. En el símbolo del sistema, ejecute el siguiente comando para cambiar la conexión a la base de datos **mypgsqldb** recién creada:

   ```sql
   \c mypgsqldb
   ```

## <a name="create-tables-in-the-database"></a>Creación de tablas en la base de datos

Ahora que sabe cómo conectarse a la base de datos de Azure Database for PostgreSQL, realice algunas tareas básicas.

En primer lugar, cree una tabla y cárguela con algunos datos. Vamos a crear una tabla que almacena la información del inventario.

```sql
CREATE TABLE inventory (
  id serial PRIMARY KEY,
  name VARCHAR(50),
  quantity INTEGER
);
```

## <a name="load-data-into-the-tables"></a>Carga de datos en las tablas

Ahora que tiene una tabla, inserte algunos datos en ella. En la ventana de símbolo del sistema abierta, ejecute la consulta siguiente para insertar algunas filas de datos.

```sql
INSERT INTO inventory (id, name, quantity) VALUES (1, 'banana', 150);
INSERT INTO inventory (id, name, quantity) VALUES (2, 'orange', 154);
```

Ahora tiene dos filas de datos de ejemplo en la tabla que creó anteriormente.

## <a name="query-and-update-the-data-in-the-tables"></a>Consulta y actualización de los datos en las tablas

Ejecute la consulta siguiente para recuperar información de la tabla de base de datos.

```sql
SELECT * FROM inventory;
```

También puede actualizar los datos en las tablas.

```sql
UPDATE inventory SET quantity = 200 WHERE name = 'banana';
```

La fila se actualiza en consecuencia cuando se recuperan los datos.

```sql
SELECT * FROM inventory;
```

## <a name="restore-a-database-to-a-previous-point-in-time"></a>Restauración de una base de datos a un momento anterior en el tiempo

Puede restaurar el servidor a un momento dado anterior. Los datos restaurados se copian en un nuevo servidor y el existente no cambia. Por ejemplo, si una tabla se ha eliminado por error, puede restaurar al momento justo antes de la eliminación. Así podrá recuperar la tabla y los datos que faltan de la copia restaurada del servidor.

Para restaurar el servidor, use el cmdlet de PowerShell de `Restore-AzPostgreSqlServer`.

### <a name="run-the-restore-command"></a>Ejecutar el comando restore

Para restaurar el servidor, ejecute el ejemplo siguiente en PowerShell.

```azurepowershell-interactive
$restorePointInTime = (Get-Date).AddMinutes(-10)
Get-AzPostgreSqlServer -Name mydemoserver -ResourceGroupName myresourcegroup |
  Restore-AzPostgreSqlServer -Name mydemoserver-restored -ResourceGroupName myresourcegroup -RestorePointInTime $restorePointInTime -UsePointInTimeRestore
```

Cuando se restaura un servidor a un momento dado anterior, se crea un servidor. El servidor de origen y las bases de datos de ese momento dado anterior se copian en el servidor nuevo.

Los valores de ubicación y plan de tarifa del servidor restaurado son los mismos que los del servidor de origen.

Una vez finalizada la restauración, busque el servidor nuevo y compruebe que los datos se restauraron según lo previsto. El nuevo servidor tiene el mismo nombre de inicio de sesión y contraseña de administrador del servidor que el servidor existente tenía cuando se inició la restauración. La contraseña se puede cambiar en la página **Información general** del nuevo servidor.

El servidor creado durante una restauración no tiene los puntos de conexión de servicio de red virtual que existían en el servidor original. Estas reglas deben configurarse por separado para el nuevo servidor. Se restauran las reglas de firewall del servidor original.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Copia de seguridad y restauración de un servidor Azure Database for PostgreSQL mediante PowerShell](howto-restore-server-powershell.md)
