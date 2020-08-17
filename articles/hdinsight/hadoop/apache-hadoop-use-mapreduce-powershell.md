---
title: Uso de MapReduce y PowerShell con Apache Hadoop en Azure HDInsight
description: Obtenga información sobre cómo usar PowerShell para ejecutar trabajos de MapReduce de forma remota con Apache Hadoop en HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 01/08/2020
ms.openlocfilehash: ff0c7001184af58432a7915b3a7df29f900bedec
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87006609"
---
# <a name="run-mapreduce-jobs-with-apache-hadoop-on-hdinsight-using-powershell"></a>Ejecución de trabajos de MapReduce con Apache Hadoop en HDInsight con PowerShell

[!INCLUDE [mapreduce-selector](../../../includes/hdinsight-selector-use-mapreduce.md)]

Este documento proporciona un ejemplo de uso de Azure PowerShell para ejecutar un trabajo de MapReduce en un Hadoop de un clúster de HDInsight.

## <a name="prerequisites"></a>Prerequisites

* Un clúster de Apache Hadoop en HDInsight. Consulte [Creación de clústeres de Apache Hadoop mediante Azure Portal](../hdinsight-hadoop-create-linux-clusters-portal.md).

* Instalación del [módulo Az](https://docs.microsoft.com/powershell/azure/) de PowerShell.

## <a name="run-a-mapreduce-job"></a>Ejecución de un trabajo de MapReduce

Azure PowerShell proporciona *cmdlets* que le permiten ejecutar de manera remota trabajos de MapReduce en HDInsight. Internamente, PowerShell hace llamadas de REST a un objeto [WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat) (anteriormente llamado Templeton) que se ejecuta en el clúster de HDInsight.

Los siguientes cmdlets se utilizan al ejecutar trabajos de MapReduce en un clúster de HDInsight remoto.

|Cmdlet | Descripción |
|---|---|
|Connect-AzAccount|autentica a Azure PowerShell en la suscripción de Azure.|
|New-AzHDInsightMapReduceJobDefinition|crea una *definición de trabajo* mediante la información especificada de MapReduce.|
|Start-AzHDInsightJob|envía la definición del trabajo a HDInsight e inicia el trabajo. Se devuelve un objeto *job*.|
|Wait-AzHDInsightJob|usa el objeto del trabajo para comprobar el estado del trabajo. Esperará hasta que el trabajo se complete o se supere el tiempo de espera.|
|Get-AzHDInsightJobOutput|se utiliza para recuperar la salida del trabajo.|

Los pasos siguientes muestran cómo usar estos cmdlets para ejecutar un trabajo en el clúster de HDInsight.

1. Mediante un editor, guarde el código siguiente como **mapreducejob.ps1**.

    [!code-powershell[main](../../../powershell_scripts/hdinsight/use-mapreduce/use-mapreduce.ps1?range=5-69)]

2. Abra un nuevo símbolo del sistema de **Azure PowerShell** . Cambie los directorios a la ubicación del archivo **mapreducejob.ps1** y, a continuación, utilice el siguiente comando para ejecutar el script:

    ```azurepowershell
    .\mapreducejob.ps1
    ```

    Si ejecuta el script, se le pedirá el nombre del clúster de HDInsight y el inicio de sesión del clúster. Puede que también se le pida que autentique su suscripción de Azure.

3. Cuando se complete el trabajo, obtendrá un texto similar al siguiente:

    ```output
    Cluster         : CLUSTERNAME
    ExitCode        : 0
    Name            : wordcount
    PercentComplete : map 100% reduce 100%
    Query           :
    State           : Completed
    StatusDirectory : f1ed2028-afe8-402f-a24b-13cc17858097
    SubmissionTime  : 12/5/2014 8:34:09 PM
    JobId           : job_1415949758166_0071
    ```

    Esto indica que el trabajo se ha completado correctamente.

    > [!NOTE]  
    > Si **ExitCode** es un valor distinto de 0, consulte [Solución de problemas](#troubleshooting).

    En este ejemplo también se almacenarán los archivos descargados en el archivo **output.txt**, en el directorio desde el que se ejecuta el script.

### <a name="view-output"></a>Ver salida

Para ver las palabras y los recuentos generados por el trabajo, abra el archivo **output.txt** en un editor de texto.

> [!NOTE]  
> Los archivos de salida de un trabajo de MapReduce no se pueden mover. Por lo tanto, si vuelve a ejecutar esta muestra, debe cambiar el nombre del archivo de salida.

## <a name="troubleshooting"></a>Solución de problemas

Si una vez completado el trabajo no se devuelve información, consulte los errores del trabajo. Para ver información de error de este trabajo, agregue el siguiente comando al final del archivo **mapreducejob.ps1**. Después, guarde el archivo y vuelva a ejecutar el script.

```powershell
# Print the output of the WordCount job.
Write-Host "Display the standard output ..." -ForegroundColor Green
Get-AzHDInsightJobOutput `
        -Clustername $clusterName `
        -JobId $wordCountJob.JobId `
        -HttpCredential $creds `
        -DisplayOutputType StandardError
```

Este cmdlet devuelve la información que se escribió en STDERR mientras se ejecutaba el trabajo.

## <a name="next-steps"></a>Pasos siguientes

Como puede ver, Azure PowerShell proporciona una manera fácil de ejecutar trabajos de MapReduce en un clúster de HDInsight, de supervisar el estado del trabajo y de recuperar el resultado. Para obtener información sobre otras maneras de trabajar con Hadoop en HDInsight:

* [Uso de MapReduce en Hadoop de HDInsight](hdinsight-use-mapreduce.md)
* [Uso de Apache Hive con Apache Hadoop en HDInsight](hdinsight-use-hive.md)
