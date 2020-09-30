---
title: 'Cadenas de conexión: Azure Database for MySQL'
description: En este documento se enumeran todas las cadenas de conexión admitidas actualmente para que las aplicaciones se conecten con Azure Database for MySQL, incluidas ADO.NET (C#), JDBC, Node.js, ODBC, PHP, Python y Ruby.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: how-to
ms.date: 3/18/2020
ms.custom: devx-track-python, devx-track-javascript
ms.openlocfilehash: 289a33d6544e3b8930059ad58115673193a1c89b
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90905807"
---
# <a name="how-to-connect-applications-to-azure-database-for-mysql"></a>Conexión de aplicaciones a Azure Database for MySQL
En este documento se enumeran todos los tipos de cadena de conexión que Azure Database for MySQL admite, junto con plantillas y ejemplos. Su cadena de conexión puede tener otros parámetros y configuraciones.

- Para obtener el certificado, consulte [Configuración de SSL](./howto-configure-ssl.md).
- {su_host} = \<servername>.mysql.database.azure.com
- {su_usuario}@{nombredeservidor} = formato de userID para una autenticación correcta.  Si solo usa userID, se producirá un error en la autenticación.

## <a name="adonet"></a>ADO.NET
```ado.net
Server={your_host};Port={your_port};Database={your_database};Uid={username@servername};Pwd={your_password};[SslMode=Required;]
```

En este ejemplo, el nombre del servidor es `mydemoserver`, el nombre de la base de datos es `wpdb`, el nombre de usuario es `WPAdmin` y la contraseña es `mypassword!2`. En consecuencia, la cadena de conexión debería ser:

```ado.net
Server= "mydemoserver.mysql.database.azure.com"; Port=3306; Database= "wpdb"; Uid= "WPAdmin@mydemoserver"; Pwd="mypassword!2"; SslMode=Required;
```

## <a name="jdbc"></a>JDBC
```jdbc
String url ="jdbc:mysql://%s:%s/%s[?verifyServerCertificate=true&useSSL=true&requireSSL=true]",{your_host},{your_port},{your_database}"; myDbConn = DriverManager.getConnection(url, {username@servername}, {your_password}";
```

## <a name="nodejs"></a>Node.js
```node.js
var conn = mysql.createConnection({host: {your_host}, user: {username@servername}, password: {your_password}, database: {your_database}, Port: {your_port}[, ssl:{ca:fs.readFileSync({ca-cert filename})}}]);
```

## <a name="odbc"></a>ODBC
```odbc
DRIVER={MySQL ODBC 5.3 UNICODE Driver};Server={your_host};Port={your_port};Database={your_database};Uid={username@servername};Pwd={your_password}; [sslca={ca-cert filename}; sslverify=1; Option=3;]
```

## <a name="php"></a>PHP
```php
$con=mysqli_init(); [mysqli_ssl_set($con, NULL, NULL, {ca-cert filename}, NULL, NULL);] mysqli_real_connect($con, {your_host}, {username@servername}, {your_password}, {your_database}, {your_port});
```

## <a name="python"></a>Python
```python
cnx = mysql.connector.connect(user={username@servername}, password={your_password}, host={your_host}, port={your_port}, database={your_database}[, ssl_ca={ca-cert filename}, ssl_verify_cert=true])
```

## <a name="ruby"></a>Ruby
```ruby
client = Mysql2::Client.new(username: {username@servername}, password: {your_password}, database: {your_database}, host: {your_host}, port: {your_port}[, sslca:{ca-cert filename}, sslverify:false, sslcipher:'AES256-SHA'])
```

## <a name="get-the-connection-string-details-from-the-azure-portal"></a>Obtención de los detalles de la cadena de conexión de Azure Portal
En [Azure Portal](https://portal.azure.com), vaya al servidor de Azure Database for MySQL y haga clic en **Cadenas de conexión** para obtener la lista de cadenas para la instancia: :::image type="content" source="./media/howto-connection-strings/connection-strings-on-portal.png" alt-text="Panel Cadenas de conexión en Azure Portal":::

La cadena proporciona detalles como el controlador, el servidor y otros parámetros de conexión de base de datos. Modifique estos ejemplos para usar sus propios parámetros, como un nombre de base de datos, una contraseña, etc. Luego puede usar esta cadena para conectarse al servidor desde su código y sus aplicaciones.

## <a name="next-steps"></a>Pasos siguientes
- Para más información sobre las bibliotecas de conexión, consulte [Conceptos: bibliotecas de conexiones](./concepts-connection-libraries.md).
