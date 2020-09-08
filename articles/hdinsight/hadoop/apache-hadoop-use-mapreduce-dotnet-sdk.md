---
title: Envío de trabajos de MapReduce mediante el SDK de HDInsight para .NET en Azure
description: Obtenga información sobre cómo enviar trabajos de MapReduce a HDInsight Apache Hadoop de Azure mediante el SDK de .NET de HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-csharp
ms.date: 01/15/2020
ms.openlocfilehash: 3de7101eb998d0f9b3b57afba6d19595738cbb63
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89020412"
---
# <a name="run-mapreduce-jobs-using-hdinsight-net-sdk"></a>Envío de trabajos de MapReduce mediante el SDK .NET de HDInsight

[!INCLUDE [mapreduce-selector](../../../includes/hdinsight-selector-use-mapreduce.md)]

Aprenda a enviar trabajos de MapReduce mediante el SDK .NET de HDInsight. Los clústeres de HDInsight vienen con un archivo .jar con varios ejemplos de MapReduce. El archivo jar es `/example/jars/hadoop-mapreduce-examples.jar`.  Uno de los ejemplos es **wordcount**. Desarrollará una aplicación de consola de C# para enviar un trabajo de recuento de palabras.  El trabajo lee el archivo `/example/data/gutenberg/davinci.txt` y envía los resultados a `/example/data/davinciwordcount`.  Si desea volver a ejecutar la aplicación, deberá limpiar la carpeta de salida.

> [!NOTE]  
> Los pasos de este artículo deben realizarse desde un cliente de Windows. Para obtener información sobre cómo usar un cliente Linux, OS X o Unix para trabajar con Hive, utilice el selector de pestañas que se muestra en la parte superior del artículo.

## <a name="prerequisites"></a>Prerequisites

* Un clúster de Apache Hadoop en HDInsight. Consulte [Creación de clústeres de Apache Hadoop mediante Azure Portal](../hdinsight-hadoop-create-linux-clusters-portal.md).

* [Visual Studio](https://visualstudio.microsoft.com/vs/community/).

## <a name="submit-mapreduce-jobs-using-hdinsight-net-sdk"></a>Envío de trabajos de MapReduce mediante el SDK .NET de HDInsight

El SDK de HDInsight para .NET ofrece bibliotecas cliente .NET que facilitan el trabajo con los clústeres de HDInsight desde .NET.

1. Inicie Visual Studio y cree una aplicación de consola de C#.

1. Vaya a **Herramientas** > **Administrador de paquetes NuGet** > **Consola del administrador de paquetes** y escriba el siguiente comando:

    ```   
    Install-Package Microsoft.Azure.Management.HDInsight.Job
    ```

1. Copie el código que se indica a continuación en **Program.cs**. A continuación, modifique el código mediante el establecimiento de los valores de `existingClusterName`, `existingClusterPassword`, `defaultStorageAccountName`, `defaultStorageAccountKey` y `defaultStorageContainerName`.

    ```csharp
    using System.Collections.Generic;
    using System.IO;
    using System.Text;
    using System.Threading;
    using Microsoft.Azure.Management.HDInsight.Job;
    using Microsoft.Azure.Management.HDInsight.Job.Models;
    using Hyak.Common;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Blob;
    
    namespace SubmitHDInsightJobDotNet
    {
        class Program
        {
            private static HDInsightJobManagementClient _hdiJobManagementClient;
    
            private const string existingClusterName = "<Your HDInsight Cluster Name>";
            private const string existingClusterPassword = "<Cluster User Password>";
            private const string defaultStorageAccountName = "<Default Storage Account Name>"; 
            private const string defaultStorageAccountKey = "<Default Storage Account Key>";
            private const string defaultStorageContainerName = "<Default Blob Container Name>";
    
            private const string existingClusterUsername = "admin";
            private const string existingClusterUri = existingClusterName + ".azurehdinsight.net";
            private const string sourceFile = "/example/data/gutenberg/davinci.txt";
            private const string outputFolder = "/example/data/davinciwordcount";
    
            static void Main(string[] args)
            {
                System.Console.WriteLine("The application is running ...");
    
                var clusterCredentials = new BasicAuthenticationCloudCredentials { Username = existingClusterUsername, Password = existingClusterPassword };
                _hdiJobManagementClient = new HDInsightJobManagementClient(existingClusterUri, clusterCredentials);
    
                SubmitMRJob();
    
                System.Console.WriteLine("Press ENTER to continue ...");
                System.Console.ReadLine();
            }
    
            private static void SubmitMRJob()
            {
                List<string> args = new List<string> { { "/example/data/gutenberg/davinci.txt" }, { "/example/data/davinciwordcount" } };
    
                var paras = new MapReduceJobSubmissionParameters
                {
                    JarFile = @"/example/jars/hadoop-mapreduce-examples.jar",
                    JarClass = "wordcount",
                    Arguments = args
                };
    
                System.Console.WriteLine("Submitting the MR job to the cluster...");
                var jobResponse = _hdiJobManagementClient.JobManagement.SubmitMapReduceJob(paras);
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
                System.Console.WriteLine("Job output is: ");
                var storageAccess = new AzureStorageAccess(defaultStorageAccountName, defaultStorageAccountKey,
                    defaultStorageContainerName);
    
                if (jobDetail.ExitValue == 0)
                {
                    // Create the storage account object
                    CloudStorageAccount storageAccount = CloudStorageAccount.Parse("DefaultEndpointsProtocol=https;AccountName=" +
                        defaultStorageAccountName +
                        ";AccountKey=" + defaultStorageAccountKey);
    
                    // Create the blob client.
                    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
    
                    // Retrieve reference to a previously created container.
                    CloudBlobContainer container = blobClient.GetContainerReference(defaultStorageContainerName);
    
                    CloudBlockBlob blockBlob = container.GetBlockBlobReference(outputFolder.Substring(1) + "/part-r-00000");
    
                    using (var stream = blockBlob.OpenRead())
                    {
                        using (StreamReader reader = new StreamReader(stream))
                        {
                            while (!reader.EndOfStream)
                            {
                                System.Console.WriteLine(reader.ReadLine());
                            }
                        }
                    }
                }
                else
                {
                    // fetch stderr output in case of failure
                    var output = _hdiJobManagementClient.JobManagement.GetJobErrorLogs(jobId, storageAccess);
    
                    using (var reader = new StreamReader(output, Encoding.UTF8))
                    {
                        string value = reader.ReadToEnd();
                        System.Console.WriteLine(value);
                    }
    
                }
            }
        }
    }

    ```

1. Presione **F5** para ejecutar la aplicación.

Para ejecutar el trabajo de nuevo, debe cambiar el nombre de la carpeta de salida del trabajo, que en el ejemplo es `/example/data/davinciwordcount`.

Cuando el trabajo se complete satisfactoriamente, la aplicación imprimirá el contenido del archivo de salida `part-r-00000`.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido varias maneras de crear un clúster de HDInsight. Para más información, vea los siguientes artículos:

* Para enviar un trabajo de Hive, consulte [Ejecución de consultas de Apache Hive mediante el SDK de .NET para HDInsight](apache-hadoop-use-hive-dotnet-sdk.md).
* Para más información sobre cómo crear clústeres de HDInsight, consulte [Creación de clústeres de Apache Hadoop basados en Linux en HDInsight](../hdinsight-hadoop-provision-linux-clusters.md).
* Para administrar clústeres de HDInsight, consulte [Administración de clústeres de Apache Haddop en HDInsight](../hdinsight-administer-use-portal-linux.md).
* Para obtener información sobre el SDK .NET de HDInsight, consulte ka [referencia del SDK de .NET de HDInsight](https://docs.microsoft.com/dotnet/api/overview/azure/hdinsight).
* Para usar la autenticación no interactiva en Azure, consulte [Creación de aplicaciones .NET para HDInsight de autenticación no interactiva](../hdinsight-create-non-interactive-authentication-dotnet-applications.md).
