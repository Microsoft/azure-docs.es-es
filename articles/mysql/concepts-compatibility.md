---
title: Compatibilidad de controladores y herramientas en Azure Database for MySQL
description: En este artículo se describen las herramientas de administración y los controladores de MySQL compatibles con Azure Database for MySQL.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: conceptual
ms.date: 3/18/2020
ms.openlocfilehash: 8d222eca76f020703f89dd1bdd5848cec4bc52ca
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89006540"
---
# <a name="mysql-drivers-and-management-tools-compatible-with-azure-database-for-mysql"></a>Herramientas de administración y controladores de MySQL compatibles con Azure Database for MySQL
En este artículo se describen las herramientas de administración y los controladores compatibles con Azure Database for MySQL.

## <a name="mysql-drivers"></a>Controladores de MySQL
Azure Database for MySQL usa la edición comunitaria más popular del mundo de la base de datos MySQL. Por lo tanto, es compatible con una amplia variedad de controladores y lenguajes de programación. El objetivo es la compatibilidad con las tres versiones más recientes de los controladores MySQL, y continúan los esfuerzos con los autores de la comunidad de código abierto para mejorar constantemente la funcionalidad y la facilidad de uso de los controladores de MySQL. En la tabla siguiente se proporciona una lista de controladores que se han probado y son compatibles con Azure Database for MySQL 5.6 y 5.7:

| **Lenguaje de programación** | **Controlador** | **Vínculos** | **Versiones compatibles** | **Versiones incompatibles** | **Notas** |
| :----------------------- | :--------- | :-------- | :---------------------- | :------------------------ | :-------- |
| PHP | mysqli, pdo_mysql, mysqlnd | https://secure.php.net/downloads.php | 5.5, 5.6, 7.x | 5.3 | Para la conexión PHP 7.0 con SSL MySQLi, agregue MYSQLI_CLIENT_SSL_DONT_VERIFY_SERVER_CERT en la cadena de conexión. <br> ```mysqli_real_connect($conn, $host, $username, $password, $db_name, 3306, NULL, MYSQLI_CLIENT_SSL_DONT_VERIFY_SERVER_CERT);```<br> PDO: establezca la opción ```PDO::MYSQL_ATTR_SSL_VERIFY_SERVER_CERT``` en false.|
| .NET | Conector de MySQL asincrónico para .NET | https://github.com/mysql-net/MySqlConnector <br> [Paquete de instalación de NuGet](https://www.nuget.org/packages/MySqlConnector/) | 0.27 y posterior | 0.26.5 y anterior | |
| .NET | Conector MySQL/NET | https://github.com/mysql/mysql-connector-net | 6.6.3 ,7.0 ,8.0 |  | Un error de codificación puede provocar errores de conexión en algunos sistemas Windows que no sean UTF8. |
| Node.js | mysqljs | https://github.com/mysqljs/mysql/ <br> Paquete de instalación de NPM:<br> Ejecute `npm install mysql` en NPM | 2.15 | 2.14.1 y anterior | |
| Node.js | node-mysql2 | https://github.com/sidorares/node-mysql2 | 1.3.4+ | | |
| Go | Controlador de MySQL de Go | https://github.com/go-sql-driver/mysql/releases | 1.3, 1.4 | 1.2 y anterior | Use `allowNativePasswords=true` en la cadena de conexión para la versión 1.3. La versión 1.4 contiene una corrección y `allowNativePasswords=true` ya no es necesario. |
| Python | Conector de MySQL/Python | https://pypi.python.org/pypi/mysql-connector-python | 1.2.3, 2.0, 2.1, 2.2, use 8.0.16+ con MySQL 8.0  | 1.2.2 y anterior | |
| Python | PyMySQL | https://pypi.org/project/PyMySQL/ | 0.7.11, 0.8.0, 0.8.1, 0.9.3+ | 0.9.0-0.9.2 (regresión en web2py) | |
| Java | Conector de MariaDB/J | https://downloads.mariadb.org/connector-java/ | 2.1, 2.0, 1.6 | 1.5.5 y anterior | | 
| Java | Conector de MySQL/J | https://github.com/mysql/mysql-connector-j | 5.1.21+, use 8.0.17+ con MySQL 8.0 | 5.1.20 y anteriores | |
| C | Conector de MySQL/C (libmysqlclient) | https://dev.mysql.com/doc/refman/5.7/en/c-api-implementations.html | 6.0.2+ | | |
| C | Conector de MySQL/ODBC (myodbc) | https://github.com/mysql/mysql-connector-odbc | 3.51.29+ | | |
| C++ | Conector de MySQL/C++ | https://github.com/mysql/mysql-connector-cpp | 1.1.9+ | 1.1.3 y anteriores | | 
| C++ | MySQL++| https://github.com/tangentsoft/mysqlpp | 3.2.3+ | | |
| Ruby | mysql2 | https://github.com/brianmario/mysql2 | 0.4.10+ | | |
| R | RMySQL | https://github.com/rstats-db/RMySQL | 0.10.16+ | | |
| Swift | mysql-swift | https://github.com/novi/mysql-swift | 0.7.2+ | | |
| Swift | vapor/mysql | https://github.com/vapor/mysql-kit | 2.0.1+ | | |

## <a name="management-tools"></a>Herramientas de administración
La ventaja de compatibilidad se amplía también a las herramientas de administración de base de datos. Sus herramientas actuales deben continuar funcionando con Azure Database for MySQL, siempre y cuando la manipulación de la base de datos tenga lugar dentro de los límites establecidos por los permisos de usuario. En la tabla siguiente se indican las tres herramientas comunes de administración de base de datos que se han probado y son compatibles con Azure Database for MySQL 5.6 y 5.7:

|                                     | **MySQL Workbench 6.x y superiores** | **Navicat 12** | **PHPMyAdmin 4.x y superiores** |
| :---------------------------------- | :----------------------------- | :------------- | :-------------------------|
| **Crear, actualizar, leer, escribir, eliminar** | X | X | X |
| **Conexión SSL** | X | X | X |
| **Completado automático de consultas SQL** | X | X |  |
| **Importar y exportar datos** | X | X | X |
| **Exportar a varios formatos** | X | X | X |
| **Copia de seguridad y restauración** |  | X |  |
| **Mostrar parámetros del servidor** | X | X | X |
| **Mostrar conexiones de cliente** | X | X | X |

## <a name="next-steps"></a>Pasos siguientes

- [Troubleshoot connection issues to Azure Database for MySQL](howto-troubleshoot-common-connection-issues.md) (Solución de problemas de conexión a Azure Database for MySQL)