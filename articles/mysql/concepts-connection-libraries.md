---
title: Bibliotecas de conexiones en Azure Database for MySQL
description: En este artículo se enumera cada biblioteca o controlador que pueden usar los programas cliente al conectarse a Azure Database for MySQL.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: conceptual
ms.date: 8/3/2020
ms.openlocfilehash: 9d4d862389a607b18ee5f2440069fa2eb6cf7bc4
ms.sourcegitcommit: 1b2d1755b2bf85f97b27e8fbec2ffc2fcd345120
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87553043"
---
# <a name="connection-libraries-for-azure-database-for-mysql"></a>Bibliotecas de conexiones de Azure Database for MySQL
En este artículo se enumera cada biblioteca o controlador que pueden usar los programas cliente al conectarse a Azure Database for MySQL.

## <a name="client-interfaces"></a>Interfaces de cliente
MySQL ofrece conectividad de controlador de base de datos estándar para el uso de MySQL con aplicaciones y herramientas que son compatibles con los estándares del sector ODBC y JDBC. Cualquier sistema que funciona con ODBC o JDBC puede usar MySQL.

| **Lenguaje** | **Plataforma** | **Recursos adicionales** | **Descargar** |
| :----------- | :------------| :-----------------------| :------------|
| PHP | Windows, Linux | [Controlador nativo de MySQL para PHP: mysqlnd](https://dev.mysql.com/downloads/connector/php-mysqlnd/) | [Descargar](https://secure.php.net/downloads.php) |
| ODBC | Plataformas Windows, Linux, Mac OS X y Unix | [MySQL Connector/ODBC Developer Guide](https://dev.mysql.com/doc/connector-odbc/en/) (Guía del desarrollador del conector de MySQL/ODBC) | [Descargar](https://dev.mysql.com/downloads/connector/odbc/) |
| ADO.NET | Windows | [MySQL Connector/Net Developer Guide](https://dev.mysql.com/doc/connector-net/en/) (Guía del desarrollador del conector de MySQL/Net) | [Descargar](https://dev.mysql.com/downloads/connector/net/) |
| JDBC | Independiente de la plataforma | [MySQL Connector/J 5.1 Developer Guide](https://dev.mysql.com/doc/connector-j/5.1/en/) (Guía del desarrollador del conector de MySQL/J 5.1) | [Descargar](https://dev.mysql.com/downloads/connector/j/) |
| Node.js | Windows, Linux, Mac OS X | [sidorares/node-mysql2](https://github.com/sidorares/node-mysql2/tree/master/documentation) | [Descargar](https://github.com/sidorares/node-mysql2) |
| Python | Windows, Linux, Mac OS X | [MySQL Connector/Python Developer Guide](https://dev.mysql.com/doc/connector-python/en/) (Guía del desarrollador del conector de MySQL/Python) | [Descargar](https://dev.mysql.com/downloads/connector/python/) |
| C++ | Windows, Linux, Mac OS X | [MySQL Connector/C++ Developer Guide](https://dev.mysql.com/doc/connector-cpp/en/) (Guía del desarrollador del conector de MySQL/C++) | [Descargar](https://dev.mysql.com/downloads/connector/python/) |
| C | Windows, Linux, Mac OS X | [MySQL Connector/C Developer Guide](https://dev.mysql.com/doc/c-api/8.0/en/) (Guía del desarrollador del conector MySQL/C) | [Descargar](https://dev.mysql.com/downloads/connector/c/)
| Perl | Plataformas Windows, Linux, Mac OS X y Unix | [DBD::MySQL](https://metacpan.org/pod/DBD::mysql) | [Descargar](https://metacpan.org/pod/DBD::mysql) |


## <a name="next-steps"></a>Pasos siguientes
Lea estos tutoriales rápidos sobre cómo conectarse a Azure Database for MySQL y realizar consultas ahí mediante el lenguaje de su elección:

- [PHP](./connect-php.md)
- [Java](./connect-java.md)
- [.NET (C#)](./connect-csharp.md)
- [Python](./connect-python.md)
- [Node.JS](./connect-nodejs.md)
- [Ruby](./connect-ruby.md)
- [C++](connect-cpp.md)
- [Go](./connect-go.md)
