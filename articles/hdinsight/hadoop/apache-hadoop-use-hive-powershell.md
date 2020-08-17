---
title: 'Uso de Apache Hive con PowerShell en HDInsight: Azure'
description: Utilice PowerShell para ejecutar consultas de Apache Hive en Apache Hadoop con Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/24/2019
ms.openlocfilehash: 327a8a0de0d144a5c1d8494a6dd22a8b89a7bd93
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87081056"
---
# <a name="run-apache-hive-queries-using-powershell"></a>Ejecución de consultas de Apache Hive con PowerShell

[!INCLUDE [hive-selector](../../../includes/hdinsight-selector-use-hive.md)]

En este documento se proporciona un ejemplo de uso de Azure PowerShell para ejecutar consultas de Apache Hive a un clúster de Apache Hadoop en HDInsight.

> [!NOTE]  
> Este documento no ofrece una descripción detallada de las instrucciones de HiveQL que se usan en los ejemplos. Para obtener información sobre el lenguaje HiveQL que se usa en este ejemplo, vea [Uso de Apache Hive con Apache Hadoop en HDInsight](hdinsight-use-hive.md).

## <a name="prerequisites"></a>Prerequisites

* Un clúster de Apache Hadoop en HDInsight. Consulte [Introducción a HDInsight en Linux](./apache-hadoop-linux-tutorial-get-started.md).

* Instalación del [módulo Az](https://docs.microsoft.com/powershell/azure/) de PowerShell.

## <a name="run-a-hive-query"></a>Ejecución de una consulta de Hive

Azure PowerShell proporciona *cmdlets* que le permiten ejecutar de manera remota las consultas de Hive en HDInsight. Internamente, los cmdlets realizan llamadas de REST a la instancia de [WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat) del clúster de HDInsight.

Los siguientes cmdlets se utilizan al ejecutar las consultas de Hive en un clúster de HDInsight remoto:

* `Connect-AzAccount`: autentica a Azure PowerShell en la suscripción de Azure.
* `New-AzHDInsightHiveJobDefinition`: crea una *definición de trabajo* con las instrucciones de HiveQL especificadas.
* `Start-AzHDInsightJob`: envía la definición del trabajo a HDInsight e inicia el trabajo. Se devuelve un objeto *job*.
* `Wait-AzHDInsightJob`: usa el objeto del trabajo para comprobar el estado del trabajo. Esperará hasta que el trabajo se complete o se supere el tiempo de espera.
* `Get-AzHDInsightJobOutput`: se utiliza para recuperar la salida del trabajo.
* `Invoke-AzHDInsightHiveJob`: se utiliza para ejecutar instrucciones de HiveQL. Este cmdlet bloquea toda la consulta y devuelve los resultados.
* `Use-AzHDInsightCluster`: establece el clúster actual que se utilizará para el comando `Invoke-AzHDInsightHiveJob`.

Los pasos siguientes muestran cómo usar estos cmdlets para ejecutar un trabajo en el clúster de HDInsight:

1. Mediante un editor, guarde el código siguiente como `hivejob.ps1`.

    [!code-powershell[main](../../../powershell_scripts/hdinsight/use-hive/use-hive.ps1?range=5-42)]

2. Abra un nuevo símbolo del sistema de **Azure PowerShell** . Cambie los directorios a la ubicación del archivo `hivejob.ps1` y, a continuación, utilice el siguiente comando para ejecutar el script:

    ```azurepowershell
    .\hivejob.ps1
    ```

    Cuando se ejecuta el script, se le pide que escriba el nombre del clúster y las credenciales de la cuenta de administrador HTTPS o del clúster. También se le puede solicitar que inicie la sesión en su suscripción de Azure.

3. Cuando el trabajo se complete, la información devuelta será similar a la siguiente:

    ```output
    Display the standard output...
    2012-02-03      18:35:34        SampleClass0    [ERROR] incorrect       id
    2012-02-03      18:55:54        SampleClass1    [ERROR] incorrect       id
    2012-02-03      19:25:27        SampleClass4    [ERROR] incorrect       id
    ```

4. Como se mencionó anteriormente, `Invoke-Hive` se puede utilizar para ejecutar una consulta y esperar la respuesta. Use el siguiente script para ver cómo funciona Invoke-Hive:

    [!code-powershell[main](../../../powershell_scripts/hdinsight/use-hive/use-hive.ps1?range=50-71)]

    La salida tendrá un aspecto similar al siguiente:

    ```output
    2012-02-03    18:35:34    SampleClass0    [ERROR]    incorrect    id
    2012-02-03    18:55:54    SampleClass1    [ERROR]    incorrect    id
    2012-02-03    19:25:27    SampleClass4    [ERROR]    incorrect    id
    ```

   > [!NOTE]  
   > Para consultas de HiveQL más extensas, puede usar el cmdlet **Here-Strings** de Azure PowerShell o archivos de script de HiveQL. El siguiente fragmento de código indica cómo usar el cmdlet `Invoke-Hive` para ejecutar un archivo de script de HiveQL. El archivo de script de HiveQL debe cargarse en wasbs://.
   >
   > `Invoke-AzHDInsightHiveJob -File "wasbs://<ContainerName>@<StorageAccountName>/<Path>/query.hql"`
   >
   > Para más información acerca de **Here-Strings**, consulte <a href="https://technet.microsoft.com/library/ee692792.aspx" target="_blank">Using Windows PowerShell Here-Strings</a> (Uso de Here-Strings de Windows PowerShell).

## <a name="troubleshooting"></a>Solución de problemas

Si una vez completado el trabajo no se devuelve información, consulte los registros de errores. Para ver información de error para este trabajo, agregue lo siguiente al final del archivo `hivejob.ps1`, guárdelo y, a continuación, ejecútelo de nuevo.

```powershell
# Print the output of the Hive job.
Get-AzHDInsightJobOutput `
        -Clustername $clusterName `
        -JobId $job.JobId `
        -HttpCredential $creds `
        -DisplayOutputType StandardError
```

Este cmdlet devuelve la información que se escribió en STDERR durante el procesamiento del trabajo.

## <a name="summary"></a>Resumen

Como puede ver, Azure PowerShell proporciona una manera fácil de ejecutar consultas de Hive en un clúster de HDInsight, de supervisar el estado del trabajo y de recuperar la salida.

## <a name="next-steps"></a>Pasos siguientes

Para obtener información general acerca de Hive en HDInsight:

* [Uso de Apache Hive con Apache Hadoop en HDInsight](hdinsight-use-hive.md)

Para obtener información sobre otras maneras de trabajar con Hadoop en HDInsight:

* [Uso de MapReduce con Apache Hadoop en HDInsight](hdinsight-use-mapreduce.md)
