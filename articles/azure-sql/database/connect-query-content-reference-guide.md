---
title: Conexión y consultas
description: Vínculos a los inicios rápidos de Azure SQL Database que muestran cómo conectarse a Azure SQL Database y a Instancia administrada de Azure SQL.
titleSuffix: Azure SQL Database & SQL Managed Instance
services: sql-database
ms.service: sql-database
ms.subservice: service
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: quickstart
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 05/29/2020
ms.openlocfilehash: 2a6aa0aa22fce5ba0f763f98fec37f607d0ce8a1
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89441782"
---
# <a name="azure-sql-database-and-azure-sql-managed-instance-connect-and-query-articles"></a>Artículos acerca de la consulta y la conexión a Azure SQL Database e Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

En el documento siguiente se incluyen vínculos a ejemplos de Azure que muestran cómo conectarse a Azure SQL Database y a Instancia administrada de Azure SQL y realizar consultas en ellos. Para conocer algunas de las recomendaciones relacionadas con el protocolo Seguridad de la capa de transporte, consulte la sección sobre las [consideraciones de TLS para la conectividad de bases de datos](#tls-considerations-for-database-connectivity).

## <a name="quickstarts"></a>Guías de inicio rápido

| Guía de inicio rápido | Descripción |
|---|---|
|[SQL Server Management Studio](connect-query-ssms.md)|Este inicio rápido muestra cómo usar SSMS para conectarse a una base de datos y, posteriormente, usar instrucciones Transact-SQL para consultar, insertar, actualizar y eliminar datos en la base de datos.|
|[Azure Data Studio](https://docs.microsoft.com/sql/azure-data-studio/quickstart-sql-database?toc=/azure/sql-database/toc.json)|Este inicio rápido muestra cómo usar Azure Data Studio para conectarse a una base de datos y luego usar instrucciones Transact-SQL (T-SQL) para crear el elemento TutorialDB empleado en los tutoriales de Azure Data Studio.|
|[Azure Portal](connect-query-portal.md)|Este inicio rápido muestra cómo usar el Editor de consultas para conectarse a una base de datos (solo Azure SQL Database) y, después, usar instrucciones Transact-SQL para realizar consultas en datos, insertarlos, actualizarlos y eliminarlos de la base de datos.|
|[Visual Studio Code](connect-query-vscode.md)|Este inicio rápido muestra cómo usar Visual Studio Code para conectarse a una base de datos y después usar las instrucciones Transact-SQL para consultar, insertar, actualizar y eliminar datos en la base de datos.|
|[.NET con Visual Studio](connect-query-dotnet-visual-studio.md)|Este inicio rápido muestra cómo usar .NET Framework para crear un programa en C# con Visual Studio que se conecte a una base de datos y que use instrucciones Transact-SQL para consultar los datos.|
|[.NET Core](connect-query-dotnet-core.md)|Este inicio rápido muestra cómo se usa .NET Core en Windows, Linux o macOS para crear un programa de C# que se conecte a una base de datos y que use instrucciones Transact-SQL para consultar los datos.|
|[Go](connect-query-go.md)|En este inicio rápido se muestra cómo usar Go para conectarse a una base de datos. También se muestran las instrucciones Transact-SQL para consultar y modificar los datos.|
|[Java](connect-query-java.md)|Este inicio rápido muestra cómo utilizar Java para conectarse a una base de datos y luego utilizar instrucciones Transact-SQL para consultar los datos.|
|[Node.js](connect-query-nodejs.md)|En este inicio rápido se muestra cómo se usa Node.js para crear un programa que se conecte a una base de datos y que use instrucciones Transact-SQL para consultar los datos.|
|[PHP](connect-query-php.md)|En este inicio rápido se muestra cómo usar PHP para crear un programa que se conecte a una base de datos y que use instrucciones Transact-SQL para consultar los datos.|
|[Python](connect-query-python.md)|Este inicio rápido muestra cómo utilizar Python para conectarse a una base de datos y luego utilizar instrucciones Transact-SQL para consultar los datos. |
|[Ruby](connect-query-ruby.md)|En este inicio rápido se muestra cómo usar Ruby para crear un programa que se conecte a una base de datos y que use instrucciones Transact-SQL para consultar los datos.|
|[R](connect-query-r.md)|En este inicio rápido se muestra cómo se usa R con Azure SQL Database Machine Learning Services para crear un programa y que se conecte a una base de datos de Azure SQL Database y use instrucciones Transact-SQL para consultar los datos.|
|||

## <a name="tls-considerations-for-database-connectivity"></a>Consideraciones de TLS para la conectividad de bases de datos

El protocolo Seguridad de capa de transporte (TLS) lo usan todos los controladores que Microsoft proporciona o admite para conectarse a bases de datos de Azure SQL Database o Instancia administrada de Azure SQL. No es necesario realizar ninguna configuración especial. En todas las conexiones a una instancia de SQL Server, a una base de datos de Azure SQL Database o una instancia de Instancia administrada de Azure SQL, se recomienda que todas las aplicaciones establezcan las siguientes configuraciones o sus equivalentes:

- **Encrypt = On**
- **TrustServerCertificate = Off**

Algunos sistemas usan palabras clave diferentes pero equivalentes para esas palabras clave de configuración. Estas configuraciones garantizan que el controlador cliente comprueba la identidad del certificado TLS recibido del servidor.

También se recomienda deshabilitar TLS 1.1 y 1.0 en el cliente si debe cumplir con el estándar de seguridad de datos de la industria de tarjetas de pago, o PCI-DSS (Payment Card Industry - Data Security Standard).

Puede que los controladores que no son de Microsoft no usen TLS de forma predeterminada. Esto puede influir al conectarse a Azure SQL Database o a Instancia administrada de Azure SQL. Las aplicaciones con controladores insertados quizás no le permitan controlar estas configuraciones de conexión. Se recomienda que examine la seguridad de estos controladores y aplicaciones antes de usarlos en sistemas que interactúan con datos confidenciales.

## <a name="libraries"></a>Bibliotecas

Puede usar varias bibliotecas y marcos para conectarse a Azure SQL Database o Instancia administrada de Azure SQL. Vea nuestros [tutoriales de introducción](https://aka.ms/sqldev) para iniciarse rápidamente en los lenguajes de programación como C#, Java, Node.js, PHP y Python. A continuación, compile una aplicación mediante el uso de SQL Server en Linux o Windows o Docker en macOS.

En la siguiente tabla se enumeran las bibliotecas de conectividad o *controladores* que las aplicaciones cliente pueden utilizar desde una variedad de lenguajes para conectarse y que usan SQL Server de forma local o en la nube. Puede utilizarlas en Linux, Windows o Docker y usarlas para conectarse a Azure SQL Database, Azure SQL Managed Instance y Azure Synapse Analytics (anteriormente SQL Data Warehouse).

| Idioma | Plataforma | Recursos adicionales | Descargar | Introducción |
| :-- | :-- | :-- | :-- | :-- |
| C# | Windows, Linux, macOS | [Microsoft ADO.NET para SQL Server](https://docs.microsoft.com/sql/connect/ado-net/microsoft-ado-net-sql-server) | [Descargar](https://www.microsoft.com/net/download/) | [Introducción](https://www.microsoft.com/sql-server/developer-get-started/csharp/ubuntu)
| Java | Windows, Linux, macOS | [Controlador JDBC de Microsoft para SQL Server](/sql/connect/jdbc/microsoft-jdbc-driver-for-sql-server/) | [Descargar](https://go.microsoft.com/fwlink/?linkid=852460) |  [Introducción](https://www.microsoft.com/sql-server/developer-get-started/java/ubuntu)
| PHP | Windows, Linux, macOS| [Controlador SQL de PHP para SQL Server](https://docs.microsoft.com/sql/connect/php/microsoft-php-driver-for-sql-server) | [Descargar](https://docs.microsoft.com/sql/connect/php/download-drivers-php-sql-server) | [Introducción](https://www.microsoft.com/sql-server/developer-get-started/php/ubuntu/)
| Node.js | Windows, Linux, macOS | [Controlador de Node.js para SQL Server](/sql/connect/node-js/node-js-driver-for-sql-server/) | [Instalación](/sql/connect/node-js/step-1-configure-development-environment-for-node-js-development/) |  [Introducción](https://www.microsoft.com/sql-server/developer-get-started/node/ubuntu)
| Python | Windows, Linux, macOS | [Controlador de Python para SQL Server](/sql/connect/python/python-driver-for-sql-server/) | Opciones de instalación: <br/> \* [pymssql](/sql/connect/python/pymssql/step-1-configure-development-environment-for-pymssql-python-development/) <br/> \* [pyodbc](/sql/connect/python/pyodbc/step-1-configure-development-environment-for-pyodbc-python-development/) |  [Introducción](https://www.microsoft.com/sql-server/developer-get-started/python/ubuntu)
| Ruby | Windows, Linux, macOS | [Controlador de Ruby para SQL Server](/sql/connect/ruby/ruby-driver-for-sql-server/) | [Instalación](/sql/connect/ruby/step-1-configure-development-environment-for-ruby-development/) | [Introducción](https://www.microsoft.com/sql-server/developer-get-started/ruby/ubuntu)
| C++ | Windows, Linux, macOS | [Microsoft ODBC driver for SQL Server](/sql/connect/odbc/microsoft-odbc-driver-for-sql-server/) | [Descargar](/sql/connect/odbc/microsoft-odbc-driver-for-sql-server/) |  

En la tabla siguiente se muestran ejemplos de marcos de asignación relacional de objetos (ORM) y marcos web que las aplicaciones cliente pueden utilizar con SQL Server, Azure SQL Database, Instancia administrada de Azure SQL o Azure Synapse Analytics. Los marcos se pueden usar en Linux, Windows o Docker.

| Idioma | Plataforma | ORM |
| :-- | :-- | :-- |
| C# | Windows, Linux, macOS | [Entity Framework](https://docs.microsoft.com/ef)<br>[Entity Framework Core](https://docs.microsoft.com/ef/core/index) |
| Java | Windows, Linux, macOS |[Hibernate ORM](https://hibernate.org/orm)|
| PHP | Windows, Linux, macOS | [Laravel (Eloquent)](https://laravel.com/docs/eloquent)<br>[Doctrine](https://www.doctrine-project.org/projects/orm.html) |
| Node.js | Windows, Linux, macOS | [Sequelize ORM](https://sequelize.org/) |
| Python | Windows, Linux, macOS |[Django](https://www.djangoproject.com/) |
| Ruby | Windows, Linux, macOS | [Ruby on Rails](https://rubyonrails.org/) |
||||

## <a name="next-steps"></a>Pasos siguientes

- Para información sobre la arquitectura de conectividad, consulte [Arquitectura de conectividad de Azure SQL Database](connectivity-architecture.md).
- Busque los [controladores de SQL Server](/sql/connect/sql-connection-libraries/) que se usan para conectarse desde aplicaciones cliente.
- Conéctese a Azure SQL Database o Instancia administrada de Azure SQL:
  - [Conexión y consulta mediante .NET (C#)](connect-query-dotnet-core.md)
  - [Conexión y consulta mediante PHP](connect-query-php.md)
  - [Conexión y consulta mediante Node.js](connect-query-nodejs.md)
  - [Conexión y consulta mediante Java](connect-query-java.md)
  - [Conexión y consulta mediante Python](connect-query-python.md)
  - [Conexión y consulta mediante Ruby](connect-query-ruby.md)
- Ejemplos de código de la lógica de reintento:
  - [Conexión resistente con ADO.NET][step-4-connect-resiliently-to-sql-with-ado-net-a78n]
  - [Conexión resistente con PHP][step-4-connect-resiliently-to-sql-with-php-p42h]

<!-- Link references. -->

[step-4-connect-resiliently-to-sql-with-ado-net-a78n]: https://docs.microsoft.com/sql/connect/ado-net/step-4-connect-resiliently-sql-ado-net

[step-4-connect-resiliently-to-sql-with-php-p42h]: https://docs.microsoft.com/sql/connect/php/step-4-connect-resiliently-to-sql-with-php
