---
title: 'Inicio rápido: Conexión mediante PHP: Azure Database for MySQL'
description: En este tutorial rápido se proporcionan ejemplos de código de PHP que se pueden usar para conectarse a Azure Database for MySQL y consultar datos en este servicio.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.custom: mvc
ms.topic: quickstart
ms.date: 5/26/2020
ms.openlocfilehash: ec4b4d705e6cc8b9cfc26952083292b74b3d4abf
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88185883"
---
# <a name="quickstart-use-php-to-connect-and-query-data-in-azure-database-for-mysql"></a>Inicio rápido: Uso de PHP para conectarse y consultar datos en Azure Database for MySQL
En este tutorial rápido se muestra cómo conectarse a una instancia de Azure Database for MySQL mediante una aplicación de [PHP](https://secure.php.net/manual/intro-whatis.php). Se indica cómo usar instrucciones SQL para consultar, insertar, actualizar y eliminar datos en la base de datos. En este tema se da por hecho que está familiarizado con el desarrollo mediante PHP y que nunca ha usado Azure Database for MySQL.

## <a name="prerequisites"></a>Prerrequisitos
En este tutorial rápido se usan como punto de partida los recursos creados en una de estas guías:
- [Create an Azure Database for MySQL server using Azure Portal](./quickstart-create-mysql-server-database-using-azure-portal.md) (Creación de un servidor de Azure Database for MySQL mediante Azure Portal)
- [Create an Azure Database for MySQL server using Azure CLI](./quickstart-create-mysql-server-database-using-azure-cli.md) (Creación de un servidor de Azure Database for MySQL mediante la CLI de Azure)

> [!IMPORTANT] 
> Asegúrese de que a la dirección IP desde la que se conecta se le han agregado las reglas de firewall del servidor desde [Azure Portal](./howto-manage-firewall-using-portal.md) o la [CLI de Azure](./howto-manage-firewall-using-cli.md)

## <a name="install-php"></a>Instalación de PHP
Instale PHP en su propio servidor o cree una [aplicación web](../app-service/overview.md) de Azure que lo incluya.

### <a name="macos"></a>MacOS
- Descargue la [versión 7.1.4 de PHP](https://secure.php.net/downloads.php).
- Instale PHP y consulte el [manual de PHP](https://secure.php.net/manual/install.macosx.php) para continuar con la configuración.

### <a name="linux-ubuntu"></a>Linux (Ubuntu)
- Descargue la [versión segura 7.1.4 de PHP sin subprocesos (x64)](https://secure.php.net/downloads.php).
- Instale PHP y consulte el [manual de PHP](https://secure.php.net/manual/install.unix.php) para continuar con la configuración.

### <a name="windows"></a>Windows
- Descargue la [versión segura 7.1.4 de PHP sin subprocesos (x64)](https://windows.php.net/download#php-7.1).
- Instale PHP y consulte el [manual de PHP](https://secure.php.net/manual/install.windows.php) para continuar con la configuración.

## <a name="get-connection-information"></a>Obtención de información sobre la conexión
Obtenga la información de conexión necesaria para conectarse a Azure Database for MySQL. Necesitará el nombre completo del servidor y las credenciales de inicio de sesión.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. En el menú izquierdo de Azure Portal, haga clic en **Todos los recursos** y, luego, busque el servidor que ha creado, por ejemplo, **mydemoserver**.
3. Haga clic en el nombre del servidor.
4. En el panel **Información general** del servidor, anote el **nombre del servidor** y el **nombre de inicio de sesión del administrador del servidor**. Si olvida la contraseña, puede restablecerla en este panel.
 ![Nombre del servidor de Azure Database for MySQL](./media/connect-php/1_server-overview-name-login.png)

## <a name="connect-and-create-a-table"></a>Conexión y creación de una tabla
Use el código siguiente para crear una tabla y conectarla mediante la instrucción SQL **CREATE TABLE**. 

El código usa la clase **MySQL Improved extension** (mysqli) incluida en PHP. El código llama a los métodos [mysqli_init](https://secure.php.net/manual/mysqli.init.php) y [mysqli_real_connect](https://secure.php.net/manual/mysqli.real-connect.php) para conectarse a MySQL. A continuación, llama al método [mysqli_query](https://secure.php.net/manual/mysqli.query.php) para ejecutar la consulta. A continuación, llama al método [mysqli_close](https://secure.php.net/manual/mysqli.close.php) para cerrar la conexión.

Reemplace los parámetros de host, username, password y db_name con sus propios valores. 

```php
<?php
$host = 'mydemoserver.mysql.database.azure.com';
$username = 'myadmin@mydemoserver';
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
$username = 'myadmin@mydemoserver';
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
$username = 'myadmin@mydemoserver';
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
$username = 'myadmin@mydemoserver';
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
$username = 'myadmin@mydemoserver';
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
> [!div class="nextstepaction"]
> [Conexión a Azure Database for MySQL a través de SSL](howto-configure-ssl.md)
