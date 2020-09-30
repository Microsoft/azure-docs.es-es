---
title: 'Inicio rápido: Conexión mediante PHP: Azure Database for MySQL con la opción Servidor flexible'
description: En este inicio rápido se proporcionan ejemplos de código de PHP que se pueden usar para conectarse a Azure Database for MySQL con la opción Servidor flexible y consultar datos en este servicio.
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.custom: mvc
ms.topic: quickstart
ms.date: 9/21/2020
ms.openlocfilehash: 77e4e2e1548beaa840f46953ef5bb4e94345416f
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945195"
---
# <a name="quickstart-use-php-to-connect-and-query-data-in-azure-database-for-mysql---flexible-server"></a>Inicio rápido: Uso de PHP para conectarse y consultar datos en Azure Database for MySQL con la opción Servidor flexible

> [!IMPORTANT]
> Actualmente, Azure Database for MySQL con la opción Servidor flexible se encuentra en versión preliminar pública.

En este inicio rápido se muestra cómo puede conectarse a una instancia de Azure Database for MySQL con la opción Servidor flexible mediante una aplicación de [PHP](https://secure.php.net/manual/intro-whatis.php). Se indica cómo usar instrucciones SQL para consultar, insertar, actualizar y eliminar datos en la base de datos. En este artículo se da por hecho que está familiarizado con el desarrollo mediante PHP, pero que nunca ha usado Azure Database for MySQL con la opción Servidor flexible.

## <a name="prerequisites"></a>Prerrequisitos
En este tutorial rápido se usan como punto de partida los recursos creados en una de estas guías:

- [Creación de una instancia de Azure Database for MySQL con la opción Servidor flexible mediante Azure Portal](./quickstart-create-server-portal.md).
- [Creación de una instancia de Azure Database for MySQL con opción Servidor flexible mediante la CLI de Azure](./quickstart-create-server-cli.md)

## <a name="preparing-your-client-workstation"></a>Preparación de la estación de trabajo de cliente
1. Si creó el servidor flexible con la opción *Acceso privado (integración con red la virtual)* , deberá conectarse a su servidor desde un recurso de la misma red virtual que el servidor. Puede crear una máquina virtual y agregarla a la red virtual creada con el servidor flexible. Consulte [Creación y administración de la red virtual de Azure Database for MySQL con la opción Servidor flexible mediante la CLI de Azure](./how-to-manage-virtual-network-cli.md).

2. Si creó el servidor flexible con la opción *Acceso público (direcciones IP permitidas)* , puede agregar la dirección IP local a la lista de reglas de firewall del servidor. Consulte [Creación y administración de reglas de firewall de Azure Database for MySQL con la opción Servidor flexible mediante la CLI de Azure](./how-to-manage-firewall-cli.md).

### <a name="install-php"></a>Instalación de PHP

Instale PHP en su propio servidor o cree una [aplicación web](https://docs.microsoft.com/azure/app-service/overview) de Azure que lo incluya.  Consulte [Creación y administración de reglas de firewall](./how-to-manage-firewall-portal.md) para obtener información sobre cómo crear las reglas de firewall.

#### <a name="macos"></a>macOS

1. Descargue la [versión 7.1.4 de PHP](https://secure.php.net/downloads.php).
2. Instale PHP y consulte el [manual de PHP](https://secure.php.net/manual/install.macosx.php) para continuar con la configuración.

#### <a name="linux-ubuntu"></a>Linux (Ubuntu)

1. Descargue la [versión segura 7.1.4 de PHP sin subprocesos (x64)](https://secure.php.net/downloads.php).
2. Instale PHP y consulte el [manual de PHP](https://secure.php.net/manual/install.unix.php) para continuar con la configuración.

#### <a name="windows"></a>Windows

1. Descargue la [versión segura 7.1.4 de PHP sin subprocesos (x64)](https://windows.php.net/download#php-7.1).
2. Instale PHP y consulte el [manual de PHP](https://secure.php.net/manual/install.windows.php) para continuar con la configuración.

## <a name="get-connection-information"></a>Obtención de información sobre la conexión

Obtenga la información de conexión necesaria para conectarse a Azure Database for MySQL con la opción Servidor flexible. Necesitará el nombre completo del servidor y las credenciales de inicio de sesión.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. En el menú izquierdo de Azure Portal, seleccione **Todos los recursos** y, luego, busque el servidor que ha creado, por ejemplo, **mydemoserver**.
3. Seleccione el nombre del servidor.
4. En el panel **Información general** del servidor, anote el **nombre del servidor** y el **nombre de inicio de sesión del administrador del servidor**. Si olvida la contraseña, puede restablecerla en este panel.
 <!---:::image type="content" source="./media/connect-php/1_server-overview-name-login.png" alt-text="Azure Database for MySQL Flexible Server name":::--->

## <a name="connecting-to-flexible-server-using-tlsssl-in-php"></a>Conexión a un servidor flexible mediante TLS/SSL en PHP

Para establecer una conexión cifrada a su servidor flexible a través de TLS/SSL desde la aplicación, consulte los siguientes ejemplos de código: Puede descargar el certificado necesario para comunicarse a través de TLS/SSL desde [https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem](https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem).

```php using SSL
$conn = mysqli_init();
mysqli_ssl_set($conn,NULL,NULL, "/var/www/html/DigiCertGlobalRootCA.crt.pem", NULL, NULL);
mysqli_real_connect($conn, 'mydemoserver.mysql.database.azure.com', 'myadmin', 'yourpassword', 'quickstartdb', 3306, MYSQLI_CLIENT_SSL);
if (mysqli_connect_errno($conn)) {
die('Failed to connect to MySQL: '.mysqli_connect_error());
}
```

## <a name="connect-and-create-a-table"></a>Conexión y creación de una tabla

Use el código siguiente para crear una tabla y conectarla mediante la instrucción SQL **CREATE TABLE**.

El código usa la clase **MySQL Improved extension** (mysqli) incluida en PHP. El código llama a los métodos [mysqli_init](https://secure.php.net/manual/mysqli.init.php) y [mysqli_real_connect](https://secure.php.net/manual/mysqli.real-connect.php) para conectarse a MySQL. A continuación, llama al método [mysqli_query](https://secure.php.net/manual/mysqli.query.php) para ejecutar la consulta. A continuación, llama al método [mysqli_close](https://secure.php.net/manual/mysqli.close.php) para cerrar la conexión.

Reemplace los parámetros de host, username, password y db_name con sus propios valores.

```php
<?php
$host = 'mydemoserver.mysql.database.azure.com';
$username = 'myadmin';
$password = 'your_password';
$db_name = 'your_database';

//Establishes the connection
$conn = mysqli_init();
mysqli_real_connect($conn, $host, $username, $password, $db_name, 3306);
if (mysqli_connect_errno($conn)) {
die('Failed to connect to MySQL: '.mysqli_connect_error());
}

// Run the create table query
if (mysqli_query($conn, '
CREATE TABLE Products (
`Id` INT NOT NULL AUTO_INCREMENT ,
`ProductName` VARCHAR(200) NOT NULL ,
`Color` VARCHAR(50) NOT NULL ,
`Price` DOUBLE NOT NULL ,
PRIMARY KEY (`Id`)
);
')) {
printf("Table created\n");
}

//Close the connection
mysqli_close($conn);
?>
```

## <a name="insert-data"></a>Insertar datos

Use el código siguiente para conectarse e insertar datos mediante la instrucción SQL **INSERT**.

El código usa la clase **MySQL Improved extension** (mysqli) incluida en PHP. El código usa el método [mysqli_prepare](https://secure.php.net/manual/mysqli.prepare.php) para crear una instrucción insert preparada y enlaza los parámetros para cada valor de columna insertado mediante el método [mysqli_stmt_bind_param](https://secure.php.net/manual/mysqli-stmt.bind-param.php). El código ejecuta la instrucción mediante el método [mysqli_stmt_execute](https://secure.php.net/manual/mysqli-stmt.execute.php) y después la cierra mediante el método [mysqli_stmt_close](https://secure.php.net/manual/mysqli-stmt.close.php).

Reemplace los parámetros de host, username, password y db_name con sus propios valores.

```php
<?php
$host = 'mydemoserver.mysql.database.azure.com';
$username = 'myadmin';
$password = 'your_password';
$db_name = 'your_database';

//Establishes the connection
$conn = mysqli_init();
mysqli_real_connect($conn, $host, $username, $password, $db_name, 3306);
if (mysqli_connect_errno($conn)) {
die('Failed to connect to MySQL: '.mysqli_connect_error());
}

//Create an Insert prepared statement and run it
$product_name = 'BrandNewProduct';
$product_color = 'Blue';
$product_price = 15.5;
if ($stmt = mysqli_prepare($conn, "INSERT INTO Products (ProductName, Color, Price) VALUES (?, ?, ?)")) {
mysqli_stmt_bind_param($stmt, 'ssd', $product_name, $product_color, $product_price);
mysqli_stmt_execute($stmt);
printf("Insert: Affected %d rows\n", mysqli_stmt_affected_rows($stmt));
mysqli_stmt_close($stmt);
}

// Close the connection
mysqli_close($conn);
?>
```

## <a name="read-data"></a>Lectura de datos

Use el código siguiente para conectarse y leer los datos mediante la instrucción SQL **SELECT**.  El código usa la clase **MySQL Improved extension** (mysqli) incluida en PHP. El código usa el método [mysqli_query](https://secure.php.net/manual/mysqli.query.php) para realizar la consulta sql y el método [mysqli_fetch_assoc](https://secure.php.net/manual/mysqli-result.fetch-assoc.php) para capturar las filas resultantes.

Reemplace los parámetros de host, username, password y db_name con sus propios valores.

```php
<?php
$host = 'mydemoserver.mysql.database.azure.com';
$username = 'myadmin';
$password = 'your_password';
$db_name = 'your_database';

//Establishes the connection
$conn = mysqli_init();
mysqli_real_connect($conn, $host, $username, $password, $db_name, 3306);
if (mysqli_connect_errno($conn)) {
die('Failed to connect to MySQL: '.mysqli_connect_error());
}

//Run the Select query
printf("Reading data from table: \n");
$res = mysqli_query($conn, 'SELECT * FROM Products');
while ($row = mysqli_fetch_assoc($res)) {
var_dump($row);
}

//Close the connection
mysqli_close($conn);
?>
```

## <a name="update-data"></a>Actualización de datos

Use el código siguiente para conectarse y actualizar los datos mediante la instrucción SQL **UPDATE**.

El código usa la clase **MySQL Improved extension** (mysqli) incluida en PHP. El código usa el método [mysqli_prepare](https://secure.php.net/manual/mysqli.prepare.php) para crear una instrucción update preparada y enlaza los parámetros para cada valor de columna actualizado mediante el método [mysqli_stmt_bind_param](https://secure.php.net/manual/mysqli-stmt.bind-param.php). El código ejecuta la instrucción mediante el método [mysqli_stmt_execute](https://secure.php.net/manual/mysqli-stmt.execute.php) y después la cierra mediante el método [mysqli_stmt_close](https://secure.php.net/manual/mysqli-stmt.close.php).

Reemplace los parámetros de host, username, password y db_name con sus propios valores.

```php
<?php
$host = 'mydemoserver.mysql.database.azure.com';
$username = 'myadmin';
$password = 'your_password';
$db_name = 'your_database';

//Establishes the connection
$conn = mysqli_init();
mysqli_real_connect($conn, $host, $username, $password, $db_name, 3306);
if (mysqli_connect_errno($conn)) {
die('Failed to connect to MySQL: '.mysqli_connect_error());
}

//Run the Update statement
$product_name = 'BrandNewProduct';
$new_product_price = 15.1;
if ($stmt = mysqli_prepare($conn, "UPDATE Products SET Price = ? WHERE ProductName = ?")) {
mysqli_stmt_bind_param($stmt, 'ds', $new_product_price, $product_name);
mysqli_stmt_execute($stmt);
printf("Update: Affected %d rows\n", mysqli_stmt_affected_rows($stmt));

//Close the connection
mysqli_stmt_close($stmt);
}

mysqli_close($conn);
?>
```


## <a name="delete-data"></a>Eliminación de datos
Use el código siguiente para conectarse y leer los datos mediante la instrucción SQL **DELETE**.

El código usa la clase **MySQL Improved extension** (mysqli) incluida en PHP. El código usa el método [mysqli_prepare](https://secure.php.net/manual/mysqli.prepare.php) para crear una instrucción delete preparada y enlaza los parámetros para la cláusula where de la instrucción mediante el método [mysqli_stmt_bind_param](https://secure.php.net/manual/mysqli-stmt.bind-param.php). El código ejecuta la instrucción mediante el método [mysqli_stmt_execute](https://secure.php.net/manual/mysqli-stmt.execute.php) y después la cierra mediante el método [mysqli_stmt_close](https://secure.php.net/manual/mysqli-stmt.close.php).

Reemplace los parámetros de host, username, password y db_name con sus propios valores.

```php
<?php
$host = 'mydemoserver.mysql.database.azure.com';
$username = 'myadmin';
$password = 'your_password';
$db_name = 'your_database';

//Establishes the connection
$conn = mysqli_init();
mysqli_real_connect($conn, $host, $username, $password, $db_name, 3306);
if (mysqli_connect_errno($conn)) {
die('Failed to connect to MySQL: '.mysqli_connect_error());
}

//Run the Delete statement
$product_name = 'BrandNewProduct';
if ($stmt = mysqli_prepare($conn, "DELETE FROM Products WHERE ProductName = ?")) {
mysqli_stmt_bind_param($stmt, 's', $product_name);
mysqli_stmt_execute($stmt);
printf("Delete: Affected %d rows\n", mysqli_stmt_affected_rows($stmt));
mysqli_stmt_close($stmt);
}

//Close the connection
mysqli_close($conn);
?>
```
## <a name="next-steps"></a>Pasos siguientes
- [Conectividad cifrada con Seguridad de la capa de transporte (TLS 1.2) en Azure Database for PostgreSQL con la opción Servidor flexible](./how-to-connect-tls-ssl.md).
- Obtenga más información sobre las [redes de Azure Database for MySQL con la opción Servidor flexible](./concepts-networking.md).
- [Creación y administración de reglas de firewall de Azure Database for MySQL con la opción Servidor flexible mediante Azure Portal](./how-to-manage-firewall-portal.md).
- [Creación y administración de la red virtual de Azure Database for MySQL con la opción Servidor flexible mediante Azure Portal](./how-to-manage-virtual-network-portal.md).
