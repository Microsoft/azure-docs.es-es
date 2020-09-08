---
title: 'Ejecución de consultas de Apache Hive mediante el SDK de HDInsight para .NET: Azure'
description: Obtenga información sobre cómo enviar trabajos de Apache Hadoop a HDInsight Apache Hadoop de Azure mediante el SDK de .NET de HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-csharp
ms.date: 12/24/2019
ms.openlocfilehash: cdb2207ab834ab84cedd2ad23e5e26186ac44039
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89020480"
---
# <a name="run-apache-hive-queries-using-hdinsight-net-sdk"></a>Ejecución de consultas de Apache Hive mediante el SDK de .NET para HDInsight

[!INCLUDE [hive-selector](../../../includes/hdinsight-selector-use-hive.md)]

Obtenga información sobre cómo enviar consultas de Apache Hive mediante el SDK de .NET para HDInsight. Escriba un programa C# para enviar una consulta de Hive para enumerar tablas de Hive y mostrar los resultados.

> [!NOTE]  
> Los pasos de este artículo deben realizarse desde un cliente de Windows. Para obtener información sobre cómo usar un cliente Linux, OS X o Unix para trabajar con Hive, utilice el selector de pestañas que se muestra en la parte superior del artículo.

## <a name="prerequisites"></a>Prerequisites

Antes de empezar este artículo, debe tener los siguientes elementos:

* Un clúster de Apache Hadoop en HDInsight. Consulte [Tutorial de Hadoop: Introducción al uso de Hadoop en HDInsight basado en Linux](apache-hadoop-linux-tutorial-get-started.md).

    > [!IMPORTANT]  
    > A partir del 15 de septiembre de 2017, el SDK de .NET para HDInsight solo admite devolver resultados de consulta de Hive desde cuentas de Azure Storage. Si usa este ejemplo con un clúster de HDInsight que utilice Azure Data Lake Storage como almacenamiento principal, no se pueden recuperar resultados de búsqueda mediante .NET SDK.

* [Visual Studio](https://visualstudio.microsoft.com/vs/community/) 2013 y versiones posteriores. Se debe instalar al menos la carga de trabajo **Desarrollo de escritorio de .NET**.

## <a name="run-a-hive-query"></a>Ejecución de una consulta de Hive

El SDK .NET de HDInsight ofrece bibliotecas de cliente .NET que facilitan el trabajo con los clústeres de HDInsight de .NET.

1. Cree una aplicación de consola en C# mediante Visual Studio.

1. En la Consola del Administrador de paquetes NuGet, ejecute el siguiente comando:

    ```console
    Install-Package Microsoft.Azure.Management.HDInsight.Job
    ```

1. Edite el código siguiente para inicializar los valores de las variables: `ExistingClusterName, ExistingClusterUsername, ExistingClusterPassword,DefaultStorageAccountName,DefaultStorageAccountKey,DefaultStorageContainerName`. A continuación, use el código revisado como todo el contenido de **Program.cs** en Visual Studio.

    ```csharp
    using System.Collections.Generic;
    using System.IO;
    using System.Text;
    using System.Threading;
    using Microsoft.Azure.Management.HDInsight.Job;
    using Microsoft.Azure.Management.HDInsight.Job.Models;
    using Hyak.Common;

    namespace SubmitHDInsightJobDotNet
    {
        class Program
        {
            private static HDInsightJobManagementClient _hdiJobManagementClient;

            private const string ExistingClusterName = "<Your HDInsight Cluster Name>";
            private const string ExistingClusterUsername = "<Cluster Username>";
            private const string ExistingClusterPassword = "<Cluster User Password>";

            // Only Azure Storage accounts are supported by the SDK
            private const string DefaultStorageAccountName = "<Default Storage Account Name>";
            private const string DefaultStorageAccountKey = "<Default Storage Account Key>";
            private const string DefaultStorageContainerName = "<Default Blob Container Name>";

            private const string ExistingClusterUri = ExistingClusterName + ".azurehdinsight.net";

            static void Main(string[] args)
            {
                System.Console.WriteLine("The application is running ...");

                var clusterCredentials = new BasicAuthenticationCloudCredentials { Username = ExistingClusterUsername, Password = ExistingClusterPassword };
                _hdiJobManagementClient = new HDInsightJobManagementClient(ExistingClusterUri, clusterCredentials);

                SubmitHiveJob();

                System.Console.WriteLine("Press ENTER to continue ...");
                System.Console.ReadLine();
            }

            private static void SubmitHiveJob()
            {
                Dictionary<string, string> defines = new Dictionary<string, string> { { "hive.execution.engine", "tez" }, { "hive.exec.reducers.max", "1" } };
                List<string> args = new List<string> { { "argA" }, { "argB" } };
                var parameters = new HiveJobSubmissionParameters
                {
                    Query = "SHOW TABLES",
                    Defines = defines,
                    Arguments = args
                };

                System.Console.WriteLine("Submitting the Hive job to the cluster...");
                var jobResponse = _hdiJobManagementClient.JobManagement.SubmitHiveJob(parameters);
                var jobId = jobResponse.JobSubmissionJsonResponse.Id;
                System.Console.WriteLine("Response status code is " + jobResponse.StatusCode);
                System.Console.WriteLine("JobId is " + jobId);

                System.Console.WriteLine("Waiting for the job completion ...");

                // Wait for job completion
                var jobDetail = _hdiJobManagementClient.JobManagement.GetJob(jobId).JobDetail;
                while (!jobDetail.Status.JobComplete)
                {
                    Thread.Sleep(1000);
                    jobDetail = _hdiJobManagementClient.JobManagement.GetJob(jobId).JobDetail;
                }

                // Get job output
                var storageAccess = new AzureStorageAccess(DefaultStorageAccountName, DefaultStorageAccountKey,
                    DefaultStorageContainerName);
                var output = (jobDetail.ExitValue == 0)
                    ? _hdiJobManagementClient.JobManagement.GetJobOutput(jobId, storageAccess) // fetch stdout output in case of success
                    : _hdiJobManagementClient.JobManagement.GetJobErrorLogs(jobId, storageAccess); // fetch stderr output in case of failure

                System.Console.WriteLine("Job output is: ");

                using (var reader = new StreamReader(output, Encoding.UTF8))
                {
                    string value = reader.ReadToEnd();
                    System.Console.WriteLine(value);
                }
            }
        }
    }
    ```

1. Presione **F5** para ejecutar la aplicación.

La salida de la aplicación debe ser similar a:

![Salida de trabajo de Hive para Hadoop en HDInsight](./media/apache-hadoop-use-hive-dotnet-sdk/hdinsight-hadoop-use-hive-net-sdk-output.png)

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a enviar consultas de Apache Hive mediante el SDK para .NET de HDInsight. Para más información, vea los siguientes artículos:

* [Introducción a Azure HDInsight](apache-hadoop-linux-tutorial-get-started.md)
* [Creación de clústeres de Apache Hadoop en HDInsight](../hdinsight-hadoop-provision-linux-clusters.md)
* [Referencia del SDK de .NET de HDInsight](https://docs.microsoft.com/dotnet/api/overview/azure/hdinsight)
* [Uso de Apache Sqoop con HDInsight](apache-hadoop-use-sqoop-mac-linux.md)
* [Crear aplicaciones .NET para HDInsight de autenticación no interactiva](../hdinsight-create-non-interactive-authentication-dotnet-applications.md)
