---
title: Ejecución de trabajos de Apache Sqoop con PowerShell y Azure HDInsight
description: Aprenda a usar Azure PowerShell desde una estación de trabajo para ejecutar la importación y exportación de Apache Sqoop entre un clúster de Apache Hadoop y Azure SQL Database.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 05/14/2020
ms.openlocfilehash: 3f8a5bdc9e1240303216df1dec31c78a560c55eb
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87086501"
---
# <a name="run-apache-sqoop-jobs-with-azure-powershell-in-hdinsight"></a>Ejecución de trabajos de Apache Sqoop con Azure PowerShell en HDInsight

[!INCLUDE [sqoop-selector](../../../includes/hdinsight-selector-use-sqoop.md)]

Aprenda a usar Azure PowerShell para ejecutar trabajos de Apache Sqoop en Azure HDInsight con el fin de importar y exportar datos entre un clúster de HDInsight y Azure SQL Database o SQL Server.  Este artículo es una continuación de [Uso de Apache Sqoop con Hadoop en HDInsight](./hdinsight-use-sqoop.md).

## <a name="prerequisites"></a>Requisitos previos

* Una estación de trabajo con el [módulo AZ](https://docs.microsoft.com/powershell/azure/) de Azure PowerShell instalado.

* Finalización de la [configuración de un entorno de prueba](./hdinsight-use-sqoop.md#create-cluster-and-sql-database) de [Uso de Apache Sqoop con Hadoop en HDInsight](./hdinsight-use-sqoop.md).

* Familiaridad con Sqoop. Para más información, consulte la [guía de usuario de Sqoop](https://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html).

## <a name="sqoop-export"></a>Exportación de Sqoop

De Hive a SQL.

En este ejemplo se exportan datos de la tabla `hivesampletable` de Hive a la tabla `mobiledata` en SQL. Establezca los valores de las variables siguientes y, luego, ejecute el comando.

```powershell
$hdinsightClusterName = ""
$httpPassword = ''
$sqlDatabasePassword = ''

# These values only need to be changed if the template was not followed.
$httpUserName = "admin"
$sqlServerLogin = "sqluser"
$sqlServerName = $hdinsightClusterName + "dbserver"
$sqlDatabaseName = $hdinsightClusterName + "db"

$pw = ConvertTo-SecureString -String $httpPassword -AsPlainText -Force
$httpCredential = New-Object System.Management.Automation.PSCredential($httpUserName,$pw)

# Connection string
$connectionString = "jdbc:sqlserver://$sqlServerName.database.windows.net;user=$sqlServerLogin@$sqlServerName;password=$sqlDatabasePassword;database=$sqlDatabaseName"

# start export
New-AzHDInsightSqoopJobDefinition `
    -Command "export --connect $connectionString --table mobiledata --hcatalog-table hivesampletable" `
    | Start-AzHDInsightJob `
        -ClusterName $hdinsightClusterName `
        -HttpCredential $httpCredential
```

### <a name="alternative-execution"></a>Ejecución alternativa

1. Con el código siguiente se realiza la misma exportación; sin embargo, se proporciona una manera de leer los registros de salida. Ejecute el código para comenzar la exportación.

    ```powershell
    $sqoopCommand = "export --connect $connectionString --table mobiledata --hcatalog-table hivesampletable"
    
    $sqoopDef = New-AzHDInsightSqoopJobDefinition `
        -Command $sqoopCommand
    
    $sqoopJob = Start-AzHDInsightJob `
                    -ClusterName $hdinsightClusterName `
                    -HttpCredential $httpCredential `
                    -JobDefinition $sqoopDef
    ```

1. En el código siguiente se muestran los registros de salida. Ejecute el código siguiente:

    ```powershell
    Get-AzHDInsightJobOutput `
        -ClusterName $hdinsightClusterName `
        -HttpCredential $httpCredential `
        -JobId $sqoopJob.JobId `
        -DisplayOutputType StandardError
    
    Get-AzHDInsightJobOutput `
        -ClusterName $hdinsightClusterName `
        -HttpCredential $httpCredential `
        -JobId $sqoopJob.JobId `
        -DisplayOutputType StandardOutput
    ```

Si recibe el mensaje de error `The specified blob does not exist.`, inténtelo de nuevo después de unos minutos.

## <a name="sqoop-import"></a>Importación de Sqoop

De SQL a Azure Storage. En este ejemplo se importan datos de la tabla `mobiledata` de SQL al directorio `wasb:///tutorials/usesqoop/importeddata` de HDInsight. Los campos de los datos se separan mediante un carácter de tabulación y las líneas terminan con un carácter de nueva línea. En este ejemplo se da por supuesto que ha realizado el ejemplo anterior.

```powershell
$sqoopCommand = "import --connect $connectionString --table mobiledata --target-dir wasb:///tutorials/usesqoop/importeddata --fields-terminated-by '\t' --lines-terminated-by '\n' -m 1"


$sqoopDef = New-AzHDInsightSqoopJobDefinition `
    -Command $sqoopCommand

$sqoopJob = Start-AzHDInsightJob `
                -ClusterName $hdinsightClusterName `
                -HttpCredential $httpCredential `
                -JobDefinition $sqoopDef

Get-AzHDInsightJobOutput `
    -ClusterName $hdinsightClusterName `
    -HttpCredential $httpCredential `
    -JobId $sqoopJob.JobId `
    -DisplayOutputType StandardError

Get-AzHDInsightJobOutput `
    -ClusterName $hdinsightClusterName `
    -HttpCredential $httpCredential `
    -JobId $sqoopJob.JobId `
    -DisplayOutputType StandardOutput

```

## <a name="additional-sqoop-export-example"></a>Ejemplo adicional de exportación de Sqoop

En este ejemplo representativo se exportan datos de `/tutorials/usesqoop/data/sample.log` desde la cuenta de almacenamiento predeterminada y, luego, se importan a una tabla llamada `log4jlogs` de una base de datos de SQL Server. Este ejemplo no depende de los ejemplos anteriores.

El siguiente script de PowerShell procesa previamente el archivo de origen y lo exporta a la tabla `log4jlogs`. Reemplace `CLUSTERNAME`, `CLUSTERPASSWORD` y `SQLPASSWORD` por los valores que usó desde el requisito previo.

```powershell
<#------ BEGIN USER INPUT ------#>
$hdinsightClusterName = "CLUSTERNAME"
$httpUserName = "admin"  #default is admin, update as needed
$httpPassword = 'CLUSTERPASSWORD'
$sqlDatabasePassword = 'SQLPASSWORD'
<#------- END USER INPUT -------#>

# Other fixed variable that should be used as is
$sqlServerName = $hdinsightClusterName + "dbserver"
$sqlDatabaseName = $hdinsightClusterName + "db"
$tableName_log4j = "log4jlogs"
$exportDir_log4j = "/tutorials/usesqoop/data"
$sourceBlobName = "example/data/sample.log"
$destBlobName = "tutorials/usesqoop/data/sample.log"
$sqljdbcdriver = "/user/oozie/share/lib/sqoop/mssql-jdbc-7.0.0.jre8.jar"

$cluster = Get-AzHDInsightCluster -ClusterName $hdinsightClusterName
$defaultStorageAccountName = $cluster.DefaultStorageAccount -replace '.blob.core.windows.net'
$defaultStorageContainer = $cluster.DefaultStorageContainer
$resourceGroup = $cluster.ResourceGroup

$sqlServer = Get-AzSqlServer -ResourceGroupName $resourceGroup -ServerName $sqlServerName
$sqlServerLogin = $sqlServer.SqlAdministratorLogin
$sqlServerFQDN = $sqlServer.FullyQualifiedDomainName

#Connect to Azure subscription
Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
try{Get-AzContext}
catch{Connect-AzAccount}

#pre-process the source file
Write-Host "`nPreprocessing the source file ..." -ForegroundColor Green

# This procedure creates a new file with $destBlobName
# Define the connection string
$defaultStorageAccountKey = (Get-AzStorageAccountKey `
                                -ResourceGroupName $resourceGroup `
                                -Name $defaultStorageAccountName)[0].Value

# Create block blob objects referencing the source and destination blob.
$storageAccount = Get-AzStorageAccount `
    -ResourceGroupName $resourceGroup `
    -Name $defaultStorageAccountName

$storageContainer = ($storageAccount |Get-AzStorageContainer -Name $defaultStorageContainer).CloudBlobContainer

$sourceBlob = $storageContainer.GetBlockBlobReference($sourceBlobName)
$destBlob = $storageContainer.GetBlockBlobReference($destBlobName)

# Define a MemoryStream and a StreamReader for reading from the source file
$stream = New-Object System.IO.MemoryStream
$stream = $sourceBlob.OpenRead()
$sReader = New-Object System.IO.StreamReader($stream)

# Define a MemoryStream and a StreamWriter for writing into the destination file
$memStream = New-Object System.IO.MemoryStream
$writeStream = New-Object System.IO.StreamWriter $memStream

# Pre-process the source blob
$exString = "java.lang.Exception:"
while(-Not $sReader.EndOfStream){
    $line = $sReader.ReadLine()
    $split = $line.Split(" ")

    # remove the "java.lang.Exception" from the first element of the array
    # for example: java.lang.Exception: 2012-02-03 19:11:02 SampleClass8 [WARN] problem finding id 153454612
    if ($split[0] -eq $exString){
        #create a new ArrayList to remove $split[0]
        $newArray = [System.Collections.ArrayList] $split
        $newArray.Remove($exString)

        # update $split and $line
        $split = $newArray
        $line = $newArray -join(" ")
    }

    # remove the lines that has less than 7 elements
    if ($split.count -ge 7){
        write-host $line
        $writeStream.WriteLine($line)
    }
}

# Write to the destination blob
$writeStream.Flush()
$memStream.Seek(0, "Begin")
$destBlob.UploadFromStream($memStream)

#export the log file from the cluster to SQL
Write-Host "Exporting the log file ..." -ForegroundColor Green

$pw = ConvertTo-SecureString -String $httpPassword -AsPlainText -Force
$httpCredential = New-Object System.Management.Automation.PSCredential($httpUserName,$pw)

# Connection string
$connectionString = "jdbc:sqlserver://$sqlServerFQDN;user=$sqlServerLogin@$sqlServerName;password=$sqlDatabasePassword;database=$sqlDatabaseName"

# Submit a Sqoop job
$sqoopDef = New-AzHDInsightSqoopJobDefinition `
    -Command "export --connect $connectionString --table $tableName_log4j --export-dir $exportDir_log4j --input-fields-terminated-by \0x20 -m 1" `
    -Files $sqljdbcdriver

$sqoopJob = Start-AzHDInsightJob `
                -ClusterName $hdinsightClusterName `
                -HttpCredential $httpCredential `
                -JobDefinition $sqoopDef

Wait-AzHDInsightJob `
    -ResourceGroupName $resourceGroup `
    -ClusterName $hdinsightClusterName `
    -HttpCredential $httpCredential `
    -JobId $sqoopJob.JobId

Write-Host "Standard Error" -BackgroundColor Green
Get-AzHDInsightJobOutput `
    -ResourceGroupName $resourceGroup `
    -ClusterName $hdinsightClusterName `
    -DefaultStorageAccountName $defaultStorageAccountName `
    -DefaultStorageAccountKey $defaultStorageAccountKey `
    -DefaultContainer $defaultStorageContainer `
    -HttpCredential $httpCredential `
    -JobId $sqoopJob.JobId `
    -DisplayOutputType StandardError

Write-Host "Standard Output" -BackgroundColor Green
Get-AzHDInsightJobOutput `
    -ResourceGroupName $resourceGroupName `
    -ClusterName $hdinsightClusterName `
    -DefaultStorageAccountName $defaultStorageAccountName `
    -DefaultStorageAccountKey $defaultStorageAccountKey `
    -DefaultContainer $defaultStorageContainer `
    -HttpCredential $httpCredential `
    -JobId $sqoopJob.JobId `
    -DisplayOutputType StandardOutput
```

## <a name="limitations"></a>Limitaciones

HDInsight basado en Linux presenta las siguientes limitaciones:

* Exportación masiva: El conector Sqoop que se usa para exportar datos a SQL no admite actualmente inserciones masivas.

* Procesamiento por lotes: con el modificador `-batch` para realizar inserciones, Sqoop realiza inserciones múltiples en lugar de procesar por lotes las operaciones de inserción.

## <a name="next-steps"></a>Pasos siguientes

Ahora ya ha aprendido a usar Sqoop. Para obtener más información, consulte:

* [Uso de Apache Oozie con HDInsight](../hdinsight-use-oozie-linux-mac.md): use la acción Sqoop en un flujo de trabajo de Oozie.
* [Carga de datos en HDInsight](../hdinsight-upload-data.md): conozca otros métodos para cargar datos en HDInsight o Azure Blob Storage.
