---
title: Uso del conector de Spark con Microsoft Azure SQL y SQL Server
description: Obtenga información sobre el uso del conector de Spark con Azure SQL Database, Azure SQL Managed Instance y SQL Server.
services: sql-database
ms.service: sql-db-mi
ms.subservice: development
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
author: denzilribeiro
ms.author: denzilr
ms.reviewer: carlrab
ms.date: 09/25/2018
ms.openlocfilehash: e7ce11a5fdba5c7d98cc331d581e827d34f7af42
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89318852"
---
# <a name="accelerate-real-time-big-data-analytics-using-the-spark-connector"></a>Aceleración de análisis de macrodatos en tiempo real mediante el conector de Spark 
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

El conector de Spark permite a las bases de datos de Azure SQL Database, Azure SQL Managed Instance y SQL Server actuar como orígenes de datos de entrada o receptores de datos de salida para los trabajos de Spark. Permite usar datos transaccionales en tiempo real en análisis de macrodatos y conservar los resultados para informes o consultas ad hoc. En comparación con el conector JDBC integrado, este conector proporciona la capacidad para insertar datos de forma masiva en la base de datos. Puede mejorar el rendimiento de la inserción de fila en fila, ya que puede insertar datos entre 10 y 20 veces más rápido. El conector de Spark admite la autenticación de Azure Active Directory (Azure AD) para conectarse a Azure SQL Database y Azure SQL Managed Instance, lo que le permite conectar la base de datos desde Azure Databricks mediante su cuenta de Azure AD. Proporciona interfaces similares con el conector JDBC integrado. Es fácil migrar los trabajos de Spark existentes para usar este nuevo conector.

## <a name="download-and-build-a-spark-connector"></a>Descarga y compilación de un conector de Spark

Para comenzar, descargue el conector de Spark del [repositorio azure-sqldb-spark](https://github.com/microsoft/sql-spark-connector) en GitHub.

### <a name="official-supported-versions"></a>Versiones oficiales compatibles

| Componente                             | Versión                  |
| :-----------------------------------  | :----------------------- |
| Spark de Apache                          | 2.0.2 o posterior           |
| Scala                                 | 2.10 o posterior            |
| Microsoft JDBC Driver para SQL Server  | 6.2 o posterior             |
| Microsoft SQL Server                  | SQL Server 2008 o posterior |
| Azure SQL Database                    | Compatible                |
| Instancia administrada de Azure SQL            | Compatible                |

El conector de Spark usa Microsoft JDBC Driver para SQL Server para mover datos entre los nodos de trabajo de Spark y las bases de datos:

El flujo de datos es el siguiente:

1. El nodo principal de Spark se conecta a bases de datos de SQL Database o SQL Server y carga los datos desde una tabla específica o mediante una consulta SQL específica.
2. El nodo principal de Spark distribuye los datos a los nodos de trabajo para la transformación.
3. El nodo de trabajo se conecta a bases de datos que se conectan a SQL Database y SQL Server y escribe los datos en la base de datos. El usuario puede optar por usar la inserción de fila en fila o la inserción de forma masiva.

En el siguiente diagrama se ilustra el flujo de datos.

   ![arquitectura](./media/spark-connector/architecture.png)

### <a name="build-the-spark-connector"></a>Compilación del conector de Spark

Actualmente, el proyecto del conector usa Maven. Para crear el conector sin dependencias, puede ejecutar:

- mvn clean package
- Descargar las últimas versiones del archivo JAR desde la carpeta de versión
- Incluir el archivo JAR de Spark de SQL Database

## <a name="connect-and-read-data-using-the-spark-connector"></a>Conexión y lectura de datos mediante el conector de Spark

Puede conectarse a bases de datos de SQL Database y SQL Server desde un trabajo de Spark para leer o escribir datos. También puede ejecutar una consulta DML o DDL en bases de datos de SQL Database y SQL Server.

### <a name="read-data-from-azure-sql-and-sql-server"></a>Lectura de datos desde Azure SQL o SQL Server

```scala
import com.microsoft.azure.sqldb.spark.config.Config
import com.microsoft.azure.sqldb.spark.connect._

val config = Config(Map(
  "url"            -> "mysqlserver.database.windows.net",
  "databaseName"   -> "MyDatabase",
  "dbTable"        -> "dbo.Clients",
  "user"           -> "username",
  "password"       -> "*********",
  "connectTimeout" -> "5", //seconds
  "queryTimeout"   -> "5"  //seconds
))

val collection = sqlContext.read.sqlDB(config)
collection.show()
```

### <a name="read-data-from-azure-sql-and-sql-server-with-specified-sql-query"></a>Lectura de datos desde Azure SQL o SQL Server con la consulta SQL especificada

```scala
import com.microsoft.azure.sqldb.spark.config.Config
import com.microsoft.azure.sqldb.spark.connect._

val config = Config(Map(
  "url"          -> "mysqlserver.database.windows.net",
  "databaseName" -> "MyDatabase",
  "queryCustom"  -> "SELECT TOP 100 * FROM dbo.Clients WHERE PostalCode = 98074" //Sql query
  "user"         -> "username",
  "password"     -> "*********",
))

//Read all data in table dbo.Clients
val collection = sqlContext.read.sqlDB(config)
collection.show()
```

### <a name="write-data-to-azure-sql-and-sql-server"></a>Escritura de datos en Azure SQL o SQL Server

```scala
import com.microsoft.azure.sqldb.spark.config.Config
import com.microsoft.azure.sqldb.spark.connect._

// Aquire a DataFrame collection (val collection)

val config = Config(Map(
  "url"          -> "mysqlserver.database.windows.net",
  "databaseName" -> "MyDatabase",
  "dbTable"      -> "dbo.Clients",
  "user"         -> "username",
  "password"     -> "*********"
))

import org.apache.spark.sql.SaveMode
collection.write.mode(SaveMode.Append).sqlDB(config)
```

### <a name="run-dml-or-ddl-query-in-azure-sql-and-sql-server"></a>Ejecución de una consulta DML o DDL en Azure SQL o SQL Server

```scala
import com.microsoft.azure.sqldb.spark.config.Config
import com.microsoft.azure.sqldb.spark.query._
val query = """
              |UPDATE Customers
              |SET ContactName = 'Alfred Schmidt', City = 'Frankfurt'
              |WHERE CustomerID = 1;
            """.stripMargin

val config = Config(Map(
  "url"          -> "mysqlserver.database.windows.net",
  "databaseName" -> "MyDatabase",
  "user"         -> "username",
  "password"     -> "*********",
  "queryCustom"  -> query
))

sqlContext.sqlDBQuery(config)
```

## <a name="connect-from-spark-using-azure-ad-authentication"></a>Conexión desde Spark mediante la autenticación de Azure AD

Puede conectarse a Azure SQL Database y SQL Managed Instance con la autenticación de Azure AD. Use la autenticación de Azure AD para administrar identidades de usuarios de base de datos de forma centralizada y como alternativa a la autenticación de SQL Server.

### <a name="connecting-using-activedirectorypassword-authentication-mode"></a>Conexión con el modo de autenticación ActiveDirectoryPassword

#### <a name="setup-requirement"></a>Requisito de instalación

Si usa el modo de autenticación ActiveDirectoryPassword, necesitará descargar [azure-activedirectory-library-for-java](https://github.com/AzureAD/azure-activedirectory-library-for-java) y sus dependencias e incluirlos en la ruta de compilación Java.

```scala
import com.microsoft.azure.sqldb.spark.config.Config
import com.microsoft.azure.sqldb.spark.connect._

val config = Config(Map(
  "url"            -> "mysqlserver.database.windows.net",
  "databaseName"   -> "MyDatabase",
  "user"           -> "username",
  "password"       -> "*********",
  "authentication" -> "ActiveDirectoryPassword",
  "encrypt"        -> "true"
))

val collection = sqlContext.read.sqlDB(config)
collection.show()
```

### <a name="connecting-using-an-access-token"></a>Conexión mediante un token de acceso

#### <a name="setup-requirement"></a>Requisito de instalación

Si usa el modo de autenticación basado en token de acceso, necesitará descargar [azure-activedirectory-library-for-java](https://github.com/AzureAD/azure-activedirectory-library-for-java) y sus dependencias e incluirlos en la ruta de compilación Java.

Consulte [Uso de la autenticación de Azure Active Directory](authentication-aad-overview.md) para obtener información sobre cómo obtener un token de acceso para la base de datos en Azure SQL Database o Azure SQL Managed Instance.

```scala
import com.microsoft.azure.sqldb.spark.config.Config
import com.microsoft.azure.sqldb.spark.connect._

val config = Config(Map(
  "url"                   -> "mysqlserver.database.windows.net",
  "databaseName"          -> "MyDatabase",
  "accessToken"           -> "access_token",
  "hostNameInCertificate" -> "*.database.windows.net",
  "encrypt"               -> "true"
))

val collection = sqlContext.read.sqlDB(config)
collection.show()
```

## <a name="write-data-using-bulk-insert"></a>Escritura de datos mediante inserción masiva

El conector JDBC tradicional escribe datos en la base de datos mediante la inserción de fila en fila. Puede usar el conector de Spark para escribir datos en Azure SQL y SQL Server mediante la inserción masiva. Mejora considerablemente el rendimiento de escritura al cargar grandes conjuntos de datos o cargar datos en tablas donde se usa un índice de almacenamiento de columnas.

```scala
import com.microsoft.azure.sqldb.spark.bulkcopy.BulkCopyMetadata
import com.microsoft.azure.sqldb.spark.config.Config
import com.microsoft.azure.sqldb.spark.connect._

/**
  Add column Metadata.
  If not specified, metadata is automatically added
  from the destination table, which may suffer performance.
*/
var bulkCopyMetadata = new BulkCopyMetadata
bulkCopyMetadata.addColumnMetadata(1, "Title", java.sql.Types.NVARCHAR, 128, 0)
bulkCopyMetadata.addColumnMetadata(2, "FirstName", java.sql.Types.NVARCHAR, 50, 0)
bulkCopyMetadata.addColumnMetadata(3, "LastName", java.sql.Types.NVARCHAR, 50, 0)

val bulkCopyConfig = Config(Map(
  "url"               -> "mysqlserver.database.windows.net",
  "databaseName"      -> "MyDatabase",
  "user"              -> "username",
  "password"          -> "*********",
  "dbTable"           -> "dbo.Clients",
  "bulkCopyBatchSize" -> "2500",
  "bulkCopyTableLock" -> "true",
  "bulkCopyTimeout"   -> "600"
))

df.bulkCopyToSqlDB(bulkCopyConfig, bulkCopyMetadata)
//df.bulkCopyToSqlDB(bulkCopyConfig) if no metadata is specified.
```

## <a name="next-steps"></a>Pasos siguientes

Si aún no lo ha hecho, descargue el conector de Spark del [repositorio azure-sqldb-spark en GitHub](https://github.com/Azure/azure-sqldb-spark) y explore los recursos adicionales del repositorio:

- [Blocs de notas de ejemplo de Azure Databricks](https://github.com/Azure/azure-sqldb-spark/tree/master/samples/notebooks)
- [Scripts de ejemplo (Scala)](https://github.com/Azure/azure-sqldb-spark/tree/master/samples/scripts)

Es posible que también quiera consultar [Apache Spark SQL, DataFrames, and Datasets Guide](https://spark.apache.org/docs/latest/sql-programming-guide.html) (Guía de Apache Spark SQL, DataFrames y conjuntos de datos) y la[documentación de Azure Databricks](https://docs.microsoft.com/azure/azure-databricks/).
