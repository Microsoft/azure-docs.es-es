---
title: Copiar datos entre Data Lake Storage Gen1 y Azure SQL-Sqoop | Microsoft Docs
description: Uso de Sqoop para copiar datos entre Azure SQL Database y Azure Data Lake Storage Gen1
services: data-lake-store
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 07/30/2019
ms.author: twooley
ms.openlocfilehash: c61862ccc7bac839627e9e7a9fbff9859155c6a2
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87323084"
---
# <a name="copy-data-between-data-lake-storage-gen1-and-azure-sql-database-using-sqoop"></a>Copia de datos entre Data Lake Storage Gen1 y Azure SQL Database mediante Sqoop

Aprenda a usar Apache Sqoop para importar y exportar datos entre Azure SQL Database y Azure Data Lake Storage Gen1.

## <a name="what-is-sqoop"></a>¿Qué es Sqoop?

Las aplicaciones de macrodatos son una opción natural para procesar datos no estructurados y semiestructurados, como registros y archivos. Pero también puede que sea necesario procesar datos estructurados que se almacenan en bases de datos relacionales.

[Apache Sqoop](https://sqoop.apache.org/docs/1.4.4/SqoopUserGuide.html) es una herramienta diseñada para transferir datos entre bases de datos relacionales y un repositorio de macrodatos, como Data Lake Storage Gen1. Puede usarla para importar datos desde un sistema de administración de bases de datos relacionales (RDBMS) como Azure SQL Database a Data Lake Storage Gen1. Después, puede transformar y analizar los datos mediante cargas de trabajo de macrodatos y exportar los datos a un RDBMS. En este artículo, usa una base de datos de Azure SQL Database como la base de datos relacional para importar y exportar.

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar, debe disponer de lo siguiente:

* **Una suscripción de Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).
* **Una cuenta de Azure Data Lake Storage Gen1**. Para obtener instrucciones sobre cómo crear la cuenta, vea [Introducción a Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md)
* **Clúster de Azure HDInsight** con acceso a una cuenta de Data Lake Storage Gen1. Consulte [Creación de un clúster de HDInsight con Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md). En este artículo se supone que tiene un clúster de HDInsight Linux con acceso a Data Lake Storage Gen1.
* **Azure SQL Database**. Para obtener instrucciones sobre cómo crear una base de datos en Azure SQL, consulte [Creación de una base de datos de Azure SQL Database](../sql-database/sql-database-get-started.md).

## <a name="create-sample-tables-in-the-database"></a>Creación de tablas sencillas en la base de datos

1. Para empezar, cree dos tablas de ejemplo en la base de datos. Use [SQL Server Management Studio](../azure-sql/database/connect-query-ssms.md) o Visual Studio para conectarse a la base de datos y, después, ejecute las consultas siguientes.

    **Crear Tabla1**

    ```tsql
    CREATE TABLE [dbo].[Table1](
    [ID] [int] NOT NULL,
    [FName] [nvarchar](50) NOT NULL,
    [LName] [nvarchar](50) NOT NULL,
     CONSTRAINT [PK_Table_1] PRIMARY KEY CLUSTERED
           (
                  [ID] ASC
           )
    ) ON [PRIMARY]
    GO
    ```

    **Crear Tabla2**

    ```tsql
    CREATE TABLE [dbo].[Table2](
    [ID] [int] NOT NULL,
    [FName] [nvarchar](50) NOT NULL,
    [LName] [nvarchar](50) NOT NULL,
     CONSTRAINT [PK_Table_2] PRIMARY KEY CLUSTERED
           (
                  [ID] ASC
           )
    ) ON [PRIMARY]
    GO
    ```

1. Ejecute el siguiente comando para agregar algunos datos de ejemplo a **Tabla1**. Deje **Tabla2** vacía. Después, los datos de **Tabla1** se importarán a Data Lake Storage Gen1. Luego se exportarán los datos de Data Lake Storage Gen1 a **Tabla2**.

    ```tsql
    INSERT INTO [dbo].[Table1] VALUES (1,'Neal','Kell'), (2,'Lila','Fulton'), (3, 'Erna','Myers'), (4,'Annette','Simpson');
    ```

## <a name="use-sqoop-from-an-hdinsight-cluster-with-access-to-data-lake-storage-gen1"></a>Uso de Sqoop en un clúster de HDInsight con acceso a Data Lake Storage Gen1

Un clúster de HDInsight ya tiene los paquetes de Sqoop disponibles. Si ha configurado el clúster de HDInsight para utilizar Data Lake Storage Gen1 como almacenamiento adicional, puede usar Sqoop (sin cambios de configuración) para importar o exportar datos entre una base de datos relacional, como Azure SQL Database, y una cuenta de Data Lake Storage Gen1.

1. Para este artículo, se supone que ha creado un clúster de Linux, por lo que debe usar SSH para conectarse al clúster. Consulte [Conexión a un clúster de HDInsight basado en Linux](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).

1. Compruebe si puede tener acceso a la cuenta de Data Lake Storage Gen1 desde el clúster. Ejecute el siguiente comando desde el símbolo del sistema de SSH:

    ```console
    hdfs dfs -ls adl://<data_lake_storage_gen1_account>.azuredatalakestore.net/
    ```

   Este comando proporciona una lista de archivos o carpetas en la cuenta de Data Lake Storage Gen1.

### <a name="import-data-from-azure-sql-database-into-data-lake-storage-gen1"></a>Importación de datos de Azure SQL Database a Data Lake Storage Gen1

1. Navegue al directorio donde están disponibles los paquetes de Sqoop. Normalmente, esta ubicación es `/usr/hdp/<version>/sqoop/bin`.

1. Importe los datos de **Tabla1** a la cuenta de Data Lake Storage Gen1. Use la sintaxis siguiente:

    ```console
    sqoop-import --connect "jdbc:sqlserver://<sql-database-server-name>.database.windows.net:1433;username=<username>@<sql-database-server-name>;password=<password>;database=<sql-database-name>" --table Table1 --target-dir adl://<data-lake-storage-gen1-name>.azuredatalakestore.net/Sqoop/SqoopImportTable1
    ```

   El marcador de posición **sql-database-server-name** representa el nombre del servidor donde se ejecuta la base de datos. **sql-database-name** representa el nombre de la base de datos real.

   Por ejemplo,

    ```console
    sqoop-import --connect "jdbc:sqlserver://mysqoopserver.database.windows.net:1433;username=user1@mysqoopserver;password=<password>;database=mysqoopdatabase" --table Table1 --target-dir adl://myadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1
    ```

1. Compruebe que los datos se han transferido a la cuenta de Data Lake Storage Gen1. Ejecute el siguiente comando:

    ```console
    hdfs dfs -ls adl://hdiadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1/
    ```

   Debería ver la siguiente salida.

    ```console
    -rwxrwxrwx   0 sshuser hdfs          0 2016-02-26 21:09 adl://hdiadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1/_SUCCESS
    -rwxrwxrwx   0 sshuser hdfs         12 2016-02-26 21:09 adl://hdiadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1/part-m-00000
    -rwxrwxrwx   0 sshuser hdfs         14 2016-02-26 21:09 adl://hdiadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1/part-m-00001
    -rwxrwxrwx   0 sshuser hdfs         13 2016-02-26 21:09 adl://hdiadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1/part-m-00002
    -rwxrwxrwx   0 sshuser hdfs         18 2016-02-26 21:09 adl://hdiadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1/part-m-00003
    ```

   Cada archivo **part-m-** * corresponde a una fila de la tabla de origen, **Table1**. Puede ver el contenido de los archivos part-m-* para comprobarlo.

### <a name="export-data-from-data-lake-storage-gen1-into-azure-sql-database"></a>Exportación de datos de Data Lake Storage Gen1 a Azure SQL Database

1. Exporte los datos de la cuenta de Data Lake Storage Gen1 a la tabla vacía, **Tabla2**, en Azure SQL Database. Use la sintaxis siguiente.

    ```console
    sqoop-export --connect "jdbc:sqlserver://<sql-database-server-name>.database.windows.net:1433;username=<username>@<sql-database-server-name>;password=<password>;database=<sql-database-name>" --table Table2 --export-dir adl://<data-lake-storage-gen1-name>.azuredatalakestore.net/Sqoop/SqoopImportTable1 --input-fields-terminated-by ","
    ```

   Por ejemplo,

    ```console
    sqoop-export --connect "jdbc:sqlserver://mysqoopserver.database.windows.net:1433;username=user1@mysqoopserver;password=<password>;database=mysqoopdatabase" --table Table2 --export-dir adl://myadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1 --input-fields-terminated-by ","
    ```

1. Compruebe que los datos se han cargado en la tabla de SQL Database. Use [SQL Server Management Studio](../azure-sql/database/connect-query-ssms.md) o Visual Studio para conectarse a Azure SQL Database y ejecute la siguiente consulta.

    ```tsql
    SELECT * FROM TABLE2
    ```

   Este comando debe ofrecer la siguiente salida.

    ```output
     ID  FName    LName
    -------------------
    1    Neal     Kell
    2    Lila     Fulton
    3    Erna     Myers
    4    Annette  Simpson
    ```

## <a name="performance-considerations-while-using-sqoop"></a>Consideraciones de rendimiento sobre el uso de Sqoop

Para obtener información sobre cómo optimizar el rendimiento del trabajo de Scoop con el fin de copiar datos en Data Lake Storage Gen1, vea la [entrada de blog sobre el rendimiento de Sqoop](https://docs.microsoft.com/archive/blogs/shanyu/performance-tuning-for-hdinsight-storm-and-microsoft-azure-eventhubs).

## <a name="next-steps"></a>Pasos siguientes

* [Copia de datos de los blobs de Azure Storage en Data Lake Storage Gen1](data-lake-store-copy-data-azure-storage-blob.md)
* [Protección de datos en Data Lake Storage Gen1](data-lake-store-secure-data.md)
* [Use Azure Data Lake Analytics with Data Lake Storage Gen1](../data-lake-analytics/data-lake-analytics-get-started-portal.md) (Uso de Azure Data Lake Analytics con Data Lake Storage Gen1)
* [Use Azure HDInsight with Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md) (Uso de Azure HDInsight con Data Lake Storage Gen1)
