---
title: Cadenas de conexión en Azure Database for MariaDB
description: En este documento se enumeran todas las cadenas de conexión admitidas actualmente para que las aplicaciones se conecten con Azure Database for MariaDB, incluidas ADO.NET (C#), JDBC, Node.js, ODBC, PHP, Python y Ruby.
author: ajlam
ms.author: andrela
ms.service: mariadb
ms.topic: how-to
ms.date: 3/18/2020
ms.custom: devx-track-python, devx-track-javascript, devx-track-csharp
ms.openlocfilehash: 1d24f06631bd3fd52061637892423c94e12127af
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89021364"
---
# <a name="how-to-connect-applications-to-azure-database-for-mariadb"></a>Conexión de aplicaciones a Azure Database for MariaDB
En este documento se enumeran todos los tipos de cadena de conexión que Azure Database for MariaDB admite, junto con plantillas y ejemplos. Su cadena de conexión puede tener otros parámetros y configuraciones.

- Para obtener el certificado, consulte [Configuración de SSL](./howto-configure-ssl.md).
- {su_host} = [nombredeservidor].mariadb.database.azure.com
- {su_usuario}@{nombredeservidor} = formato de userID para una autenticación correcta.  Si solo usa userID, se producirá un error en la autenticación.

## <a name="adonet"></a>ADO.NET
```csharp
Server={your_host}; Port=3306; Database={your_database}; Uid={username@servername}; Pwd={your_password}; SslMode=Preferred;
```

En este ejemplo, el nombre del servidor es `mydemoserver`, el nombre de la base de datos es `wpdb`, el nombre de usuario es `WPAdmin` y la contraseña es `mypassword!2`. En consecuencia, la cadena de conexión debería ser:

```csharp
Server= "mydemoserver.mariadb.database.azure.com"; Port=3306; Database= "wpdb"; Uid= "WPAdmin@mydemoserver"; Pwd="mypassword!2"; SslMode=Required;
```

## <a name="jdbc"></a>JDBC
```java
String url ="jdbc:mariadb://{your_host}:3306/{your_database}?useSSL=true&trustServerCertificate=true"; myDbConn = DriverManager.getConnection(url, "{username@servername}", {your_password});
```

## <a name="nodejs"></a>Node.js
```javascript
var conn = mysql.createConnection({host: "{your_host}", user: "{your_username}", password: {your_password}, database: {your_database}, port: 3306, ssl:{ca:fs.readFileSync({ca-cert filename})}});
```

## <a name="odbc"></a>ODBC
```cpp
DRIVER={MARIADB ODBC 3.0 Driver}; Server="{your_host}"; Port=3306; Database={your_database}; Uid="{username@servername}"; Pwd={your_password}; sslca={ca-cert filename}; sslverify=1;
```

## <a name="php"></a>PHP
```php
$con=mysqli_init(); mysqli_ssl_set($con, NULL, NULL, {ca-cert filename}, NULL, NULL); mysqli_real_connect($con, "{your_host}", "{username@servername}", {your_password}, {your_database}, 3306);
```

## <a name="python"></a>Python
```python
cnx = mysql.connector.connect(user="{username@servername}", password={your_password}, host="{your_host}", port=3306, database={your_database}, ssl_ca={ca-cert filename}, ssl_verify_cert=true)
```

## <a name="ruby"></a>Ruby
```ruby
client = Mysql2::Client.new(username: "{username@servername}", password: {your_password}, database: {your_database}, host: "{your_host}", port: 3306, sslca:{ca-cert filename}, sslverify:false, sslcipher:'AES256-SHA')
```

## <a name="get-the-connection-string-details-from-the-azure-portal"></a>Obtención de los detalles de la cadena de conexión de Azure Portal
En [Azure Portal](https://portal.azure.com), vaya al servidor de Azure Database for MariaDB y haga clic en **Cadenas de conexión** para obtener la lista de cadenas para la instancia: ![Panel Cadenas de conexión en Azure Portal](./media/howto-connection-strings/connection-strings-on-portal.png)

La cadena proporciona detalles como el controlador, el servidor y otros parámetros de conexión de base de datos. Modifique estos ejemplos para usar sus propios parámetros, como un nombre de base de datos, una contraseña, etc. Luego puede usar esta cadena para conectarse al servidor desde su código y sus aplicaciones.

<!-- 
## Next steps
- For more information about connection libraries, see [Concepts - Connection libraries](./concepts-connection-libraries.md).
- -->
