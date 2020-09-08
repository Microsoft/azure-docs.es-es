---
title: Ejecución de trabajos de Apache Sqoop con .NET y HDInsight en Azure
description: Aprenda a usar el SDK de HDInsight para .NET para ejecutar la importación y exportación de Apache Sqoop entre un clúster de Apache Hadoop y una base de datos de Azure SQL.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, hdiseo17may2017, devx-track-csharp
ms.date: 01/14/2020
ms.openlocfilehash: bc4b3019cb8308ea19d86eea7bf886063ea28563
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89008291"
---
# <a name="run-apache-sqoop-jobs-by-using-net-sdk-for-apache-hadoop-in-hdinsight"></a>Ejecución de trabajos de Apache Sqoop con el SDK de .NET para Apache Hadoop en HDInsight

[!INCLUDE [sqoop-selector](../../../includes/hdinsight-selector-use-sqoop.md)]

Aprenda a usar el SDK de Azure HDInsight para .NET ejecutar trabajos de Apache Sqoop con el fin de llevar a cabo tareas de importación y exportación entre un clúster de HDInsight y una base de datos de Azure SQL o SQL Server.

## <a name="prerequisites"></a>Prerrequisitos

* Finalización de la [configuración de un entorno de prueba](./hdinsight-use-sqoop.md#create-cluster-and-sql-database) de [Uso de Apache Sqoop con Hadoop en HDInsight](./hdinsight-use-sqoop.md).

* [Visual Studio](https://visualstudio.microsoft.com/vs/community/).

* Familiaridad con Sqoop. Para más información, consulte la [guía de usuario de Sqoop](https://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html).

## <a name="use-sqoop-on-hdinsight-clusters-with-the-net-sdk"></a>Uso de Sqoop en clústeres de HDInsight con el SDK de .NET

El SDK .NET para HDInsight proporciona bibliotecas de cliente .NET que facilitan el uso de los clústeres de HDInsight en .NET. En esta sección se crea una aplicación de consola de C# para exportar `hivesampletable` a la tabla de Azure SQL Database creada anteriormente en los requisitos previos.

## <a name="set-up"></a>Configurar

1. Inicie Visual Studio y cree una aplicación de consola de C#.

1. Vaya a **Herramientas** > **Administrador de paquetes NuGet** > **Consola del administrador de paquetes** y ejecute el siguiente comando:

    ```
    Install-Package Microsoft.Azure.Management.HDInsight.Job
    ```

## <a name="sqoop-export"></a>Exportación de Sqoop

De Hive a SQL Server.  En este ejemplo se exportan datos de la tabla `hivesampletable` de Hive a la tabla `mobiledata` de SQL Database.

1. Use el siguiente código en el archivo Program.cs. Edite el código para establecer los valores de `ExistingClusterName` y `ExistingClusterPassword`.

    ```csharp
    using Microsoft.Azure.Management.HDInsight.Job;
    using Microsoft.Azure.Management.HDInsight.Job.Models;
    using Hyak.Common;
    
    namespace SubmitHDInsightJobDotNet
    {
        class Program
        {
            private static HDInsightJobManagementClient _hdiJobManagementClient;
    
            private const string ExistingClusterName = "<Your HDInsight Cluster Name>";
            private const string ExistingClusterPassword = "<Cluster User Password>";
            private const string ExistingClusterUri = ExistingClusterName + ".azurehdinsight.net";
            private const string ExistingClusterUsername = "admin";
    
            static void Main(string[] args)
            {
                System.Console.WriteLine("The application is running ...");
    
                var clusterCredentials = new BasicAuthenticationCloudCredentials { Username = ExistingClusterUsername, Password = ExistingClusterPassword };
                _hdiJobManagementClient = new HDInsightJobManagementClient(ExistingClusterUri, clusterCredentials);
    
                SubmitSqoopJob();
    
                System.Console.WriteLine("Press ENTER to continue ...");
                System.Console.ReadLine();
            }
    
            private static void SubmitSqoopJob()
            {
                var sqlDatabaseServerName = ExistingClusterName + "dbserver";
                var sqlDatabaseLogin = "sqluser";
                var sqlDatabaseLoginPassword = ExistingClusterPassword;
                var sqlDatabaseDatabaseName = ExistingClusterName + "db";
    
                // Connection string for using Azure SQL Database; Comment if using SQL Server
                var connectionString = "jdbc:sqlserver://" + sqlDatabaseServerName + ".database.windows.net;user=" + sqlDatabaseLogin + "@" + sqlDatabaseServerName + ";password=" + sqlDatabaseLoginPassword + ";database=" + sqlDatabaseDatabaseName;
    
                // Connection string for using SQL Server; Uncomment if using SQL Server
                // var connectionString = "jdbc:sqlserver://" + sqlDatabaseServerName + ";user=" + sqlDatabaseLogin + ";password=" + sqlDatabaseLoginPassword + ";database=" + sqlDatabaseDatabaseName;
    
                //sqoop start
                var tableName = "mobiledata";
    
                var parameters = new SqoopJobSubmissionParameters
                {
                     Command = "export --connect " + connectionString + " --table " + tableName + " --hcatalog-table hivesampletable"
                };
                //sqoop end
    
                System.Console.WriteLine("Submitting the Sqoop job to the cluster...");
                var response = _hdiJobManagementClient.JobManagement.SubmitSqoopJob(parameters);
                System.Console.WriteLine("Validating that the response is as expected...");
                System.Console.WriteLine("Response status code is " + response.StatusCode);
                System.Console.WriteLine("Validating the response object...");
                System.Console.WriteLine("JobId is " + response.JobSubmissionJsonResponse.Id);
            }
        }
    }
    ```

1. Para ejecutar el programa, seleccione la tecla **F5**.

## <a name="sqoop-import"></a>Importación de Sqoop

De SQL Server a Azure Storage. Este ejemplo depende de que se haya realizado la exportación anterior.  En este ejemplo se importan datos de la tabla `mobiledata` de SQL Database al directorio `wasb:///tutorials/usesqoop/importeddata` de la cuenta de almacenamiento predeterminada del clúster.

1. Reemplace el código anterior del bloque `//sqoop start //sqoop end` por el código siguiente:

    ```csharp
    var tableName = "mobiledata";
    var exportDir = "/tutorials/usesqoop/importeddata";
    
    var parameters = new SqoopJobSubmissionParameters
    {
        Command = "import --connect " + connectionString + " --table " + tableName + " --target-dir " +  exportDir + " --fields-terminated-by \\t --lines-terminated-by \\n -m 1"
    };
    ```

1. Para ejecutar el programa, seleccione la tecla **F5**.

## <a name="limitations"></a>Limitaciones

HDInsight basado en Linux presenta las siguientes limitaciones:

* Exportación masiva: el conector de Sqoop que se usa para exportar datos a Microsoft SQL Server o Azure SQL Database no admite actualmente inserciones masivas.

* Procesamiento por lotes: con el modificador `-batch`, Sqoop realiza inserciones múltiples en lugar de procesar por lotes las operaciones de inserción.

## <a name="next-steps"></a>Pasos siguientes

Ahora ya ha aprendido a usar Sqoop. Para obtener más información, consulte:

* [Uso de Apache Oozie con HDInsight](../hdinsight-use-oozie-linux-mac.md): use la acción Sqoop en un flujo de trabajo de Oozie.
* [Carga de datos en HDInsight](../hdinsight-upload-data.md): conozca otros métodos para cargar datos en HDInsight o Azure Blob Storage.
