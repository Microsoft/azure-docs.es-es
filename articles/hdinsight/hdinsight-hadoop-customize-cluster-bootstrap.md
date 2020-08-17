---
title: Personalización de configuraciones de clúster de Azure HDInsight mediante el arranque
description: Obtenga información sobre cómo personalizar la configuración del clúster de HDInsight mediante programación con .NET, PowerShell y plantillas de Resource Manager.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 04/01/2020
ms.openlocfilehash: eaddae04c198742007947831046139d80828534b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87006592"
---
# <a name="customize-hdinsight-clusters-using-bootstrap"></a>Personalización de los clústeres de HDInsight con Bootstrap

Los scripts de arranque permiten instalar y configurar componentes en Azure HDInsight mediante programación.

Hay tres enfoques para establecer opciones de archivos de configuración al crear el clúster de HDInsight:

* Uso de Azure PowerShell
* Uso del SDK de .NET
* Usar plantillas de Azure Resource Manager

Por ejemplo, mediante estos métodos de programación, puede configurar opciones en estos archivos:

* clusterIdentity.xml
* core-site.xml
* gateway.xml
* hbase-env.xml
* hbase-site.xml
* hdfs-site.xml
* hive-env.xml
* hive-site.xml
* mapred-site
* oozie-site.xml
* oozie-env.xml
* storm-site.xml
* tez-site.xml
* webhcat-site.xml
* yarn-site.xml
* server.properties (kafka-broker configuration)

Para obtener información sobre cómo instalar componentes adicionales en el clúster de HDInsight en tiempo de creación, vea [Personalización de clústeres de HDInsight de Azure mediante el uso de las acciones de script (Linux)](hdinsight-hadoop-customize-cluster-linux.md).

## <a name="prerequisites"></a>Prerrequisitos

* Si usa PowerShell, necesitará el [Módulo Az](https://docs.microsoft.com/powershell/azure/).

## <a name="use-azure-powershell"></a>Uso de Azure PowerShell

El siguiente código de PowerShell personaliza una configuración de [Apache Hive](https://hive.apache.org/):

> [!IMPORTANT]  
> Es posible que tenga que usar el parámetro `Spark2Defaults` con [Add-AzHDInsightConfigValue](https://docs.microsoft.com/powershell/module/az.hdinsight/add-azhdinsightconfigvalue). Puede pasar valores vacíos al parámetro, como se muestra en el ejemplo de código siguiente.

```powershell
# hive-site.xml configuration
$hiveConfigValues = @{ "hive.metastore.client.socket.timeout"="90s" }

$config = New-AzHDInsightClusterConfig `
    | Set-AzHDInsightDefaultStorage `
        -StorageAccountName "$defaultStorageAccountName.blob.core.windows.net" `
        -StorageAccountKey $defaultStorageAccountKey `
    | Add-AzHDInsightConfigValue `
        -HiveSite $hiveConfigValues `
        -Spark2Defaults @{}

New-AzHDInsightCluster `
    -ResourceGroupName $existingResourceGroupName `
    -ClusterName $clusterName `
    -Location $location `
    -ClusterSizeInNodes $clusterSizeInNodes `
    -ClusterType Hadoop `
    -OSType Linux `
    -Version "3.6" `
    -HttpCredential $httpCredential `
    -Config $config
```

En el [Anexo](#appendix-powershell-sample) se incluye un script de PowerShell completamente en uso.

**Para comprobar el cambio:**

1. Vaya a `https://CLUSTERNAME.azurehdinsight.net/`, donde `CLUSTERNAME` es el nombre del clúster.
1. En el menú de la izquierda, vaya a **Hive** > **Configs (Configuraciones)**  > **Advanced (Avanzado)** .
1. Expanda **Advanced hive-site** (Sitio de Hive avanzado).
1. Busque **hive.metastore.client.socket.timeout** y confirme que el valor es **90s**.

Otros ejemplos de cómo personalizar otros archivos de configuración:

```xml
# hdfs-site.xml configuration
$HdfsConfigValues = @{ "dfs.blocksize"="64m" } #default is 128MB in HDI 3.0 and 256MB in HDI 2.1

# core-site.xml configuration
$CoreConfigValues = @{ "ipc.client.connect.max.retries"="60" } #default 50

# mapred-site.xml configuration
$MapRedConfigValues = @{ "mapreduce.task.timeout"="1200000" } #default 600000

# oozie-site.xml configuration
$OozieConfigValues = @{ "oozie.service.coord.normal.default.timeout"="150" }  # default 120
```

## <a name="use-net-sdk"></a>Uso del SDK de .NET

Vea [SDK de Azure HDInsight para .NET](https://docs.microsoft.com/dotnet/api/overview/azure/hdinsight?view=azure-dotnet).

## <a name="use-resource-manager-template"></a>Uso de plantillas de Resource Manager

Puede usar Bootstrap en la plantilla de Resource Manager:

```json
"configurations": {
    "hive-site": {
        "hive.metastore.client.connect.retry.delay": "5",
        "hive.execution.engine": "mr",
        "hive.security.authorization.manager": "org.apache.hadoop.hive.ql.security.authorization.DefaultHiveAuthorizationProvider"
    }
}
```

![Hadoop personaliza la plantilla de Azure Resource Manager de arranque del clúster](./media/hdinsight-hadoop-customize-cluster-bootstrap/hdinsight-customize-cluster-bootstrap-arm.png)

Ejemplo de fragmento de código de plantilla de Resource Manager para cambiar la configuración en spark2-defaults para limpiar periódicamente los registros de eventos del almacenamiento.  

```json
"configurations": {
    "spark2-defaults": {
        "spark.history.fs.cleaner.enabled": "true",
        "spark.history.fs.cleaner.interval": "7d",
        "spark.history.fs.cleaner.maxAge": "90d"
    }
}
```

## <a name="see-also"></a>Consulte también

* En [Creación de clústeres de Apache Hadoop en HDInsight](hdinsight-hadoop-provision-linux-clusters.md) se proporcionan instrucciones sobre cómo crear un clúster de HDInsight con otras opciones personalizadas.
* [Desarrollo de la acción de script con HDInsight](hdinsight-hadoop-script-actions-linux.md)
* [Instalación y uso de Apache Spark en clústeres de HDInsight](spark/apache-spark-jupyter-spark-sql-use-portal.md)
* [Instalación y uso de Apache Giraph en clústeres de HDInsight](hdinsight-hadoop-giraph-install.md).

## <a name="appendix-powershell-sample"></a>Apéndice: Ejemplo de PowerShell

Este script de PowerShell crea un clúster de HDInsight y personaliza una configuración de Hive. Asegúrese de especificar valores para `$nameToken`, `$httpPassword` y `$sshPassword`.

```powershell
####################################
# Set these variables
####################################
#region - used for creating Azure service names
$nameToken = "<ENTER AN ALIAS>"
#endregion

#region - cluster user accounts
$httpUserName = "admin"  #HDInsight cluster username
$httpPassword = '<ENTER A PASSWORD>'

$sshUserName = "sshuser" #HDInsight ssh user name
$sshPassword = '<ENTER A PASSWORD>'
#endregion

####################################
# Service names and varialbes
####################################
#region - service names
$namePrefix = $nameToken.ToLower() + (Get-Date -Format "MMdd")

$resourceGroupName = $namePrefix + "rg"
$hdinsightClusterName = $namePrefix + "hdi"
$defaultStorageAccountName = $namePrefix + "store"
$defaultBlobContainerName = $hdinsightClusterName

$location = "East US"
#endregion


####################################
# Connect to Azure
####################################
#region - Connect to Azure subscription
Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
$sub = Get-AzSubscription -ErrorAction SilentlyContinue
if(-not($sub))
{
    Connect-AzAccount
}

# If you have multiple subscriptions, set the one to use
# Select-AzSubscription -SubscriptionId "<SUBSCRIPTIONID>"

#endregion

#region - Create an HDInsight cluster
####################################
# Create dependent components
####################################
Write-Host "Creating a resource group ..." -ForegroundColor Green
New-AzResourceGroup `
    -Name  $resourceGroupName `
    -Location $location

Write-Host "Creating the default storage account and default blob container ..."  -ForegroundColor Green
New-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $defaultStorageAccountName `
    -Location $location `
    -SkuName Standard_LRS `
    -Kind StorageV2 `
    -EnableHttpsTrafficOnly 1

# Note: Storage account kind BlobStorage cannot be used as primary storage.

$defaultStorageAccountKey = (Get-AzStorageAccountKey `
                                -ResourceGroupName $resourceGroupName `
                                -Name $defaultStorageAccountName)[0].Value

$defaultStorageContext = New-AzStorageContext `
                                -StorageAccountName $defaultStorageAccountName `
                                -StorageAccountKey $defaultStorageAccountKey

New-AzStorageContainer `
    -Name $defaultBlobContainerName `
    -Context $defaultStorageContext #use the cluster name as the container name

####################################
# Create a configuration object
####################################
$hiveConfigValues = @{"hive.metastore.client.socket.timeout"="90s"}

$config = New-AzHDInsightClusterConfig `
    | Set-AzHDInsightDefaultStorage `
        -StorageAccountName "$defaultStorageAccountName.blob.core.windows.net" `
        -StorageAccountKey $defaultStorageAccountKey `
    | Add-AzHDInsightConfigValue `
        -HiveSite $hiveConfigValues `
        -Spark2Defaults @{}

####################################
# Create an HDInsight cluster
####################################
$httpPW = ConvertTo-SecureString -String $httpPassword -AsPlainText -Force
$httpCredential = New-Object System.Management.Automation.PSCredential($httpUserName,$httpPW)

$sshPW = ConvertTo-SecureString -String $sshPassword -AsPlainText -Force
$sshCredential = New-Object System.Management.Automation.PSCredential($sshUserName,$sshPW)

New-AzHDInsightCluster `
    -ResourceGroupName $resourceGroupName `
    -ClusterName $hdinsightClusterName `
    -Location $location `
    -ClusterSizeInNodes 1 `
    -ClusterType Hadoop `
    -OSType Linux `
    -Version "3.6" `
    -HttpCredential $httpCredential `
    -SshCredential $sshCredential `
    -Config $config

####################################
# Verify the cluster
####################################
Get-AzHDInsightCluster `
    -ClusterName $hdinsightClusterName

#endregion
```
