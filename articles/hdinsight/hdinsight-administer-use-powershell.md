---
title: 'Administración de clústeres de Apache Hadoop con PowerShell: Azure HDInsight'
description: Vea cómo realizar tareas administrativas para clústeres de Apache Hadoop en HDInsight mediante Azure PowerShell.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive
ms.date: 02/13/2020
ms.openlocfilehash: 1839cfe0f7295af583b7bd5397957a223e888d3e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87086513"
---
# <a name="manage-apache-hadoop-clusters-in-hdinsight-by-using-azure-powershell"></a>Administración de clústeres de Apache Hadoop en HDInsight con Azure PowerShell

[!INCLUDE [selector](../../includes/hdinsight-portal-management-selector.md)]

Se puede usar Azure PowerShell para controlar y automatizar la implementación y la administración de las cargas de trabajo en Azure. En este artículo aprenderá a administrar clústeres de [Apache Hadoop](https://hadoop.apache.org/) en Azure HDInsight mediante el módulo Az de Azure PowerShell. Para más información sobre los cmdlets de PowerShell de HDInsight, consulte la [referencia de Az.HDInsight](https://docs.microsoft.com/powershell/module/az.hdinsight).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Instalación del [módulo Az](https://docs.microsoft.com/powershell/azure/) de PowerShell.

## <a name="create-clusters"></a>Creación de clústeres

Consulte [Crear clústeres basados en Linux en HDInsight con Azure PowerShell](hdinsight-hadoop-create-linux-clusters-azure-powershell.md)

## <a name="list-clusters"></a>Lista de clústeres

Use el comando siguiente para enumerar todos los clústeres de la suscripción actual:

```powershell
Get-AzHDInsightCluster
```

## <a name="show-cluster"></a>Presentación de clústeres

Use el comando siguiente para mostrar los detalles de un clúster específico de la suscripción actual:

```powershell
Get-AzHDInsightCluster -ClusterName <Cluster Name>
```

## <a name="delete-clusters"></a>Eliminación de clústeres

Use el comando siguiente para eliminar un clúster:

```powershell
Remove-AzHDInsightCluster -ClusterName <Cluster Name>
```

También puede eliminar un clúster quitando el grupo de recursos que contiene el clúster. La eliminación de un grupo de recursos supone la eliminación de todos los recursos del grupo, incluida la cuenta de almacenamiento predeterminada.

```powershell
Remove-AzResourceGroup -Name <Resource Group Name>
```

## <a name="scale-clusters"></a>Escalado de clústeres

La característica de escalado de clústeres permite cambiar la cantidad de nodos de trabajo que usa un clúster que se ejecuta en HDInsight de Azure sin necesidad de volver a crear el clúster. Para cambiar el tamaño del clúster de Hadoop con Azure PowerShell, ejecute el siguiente comando desde un equipo cliente:

```powershell
Set-AzHDInsightClusterSize -ClusterName <Cluster Name> -TargetInstanceCount <NewSize>
```

 Para más información sobre la escalabilidad de los clústeres, consulte [Escalabilidad de clústeres de HDInsight](./hdinsight-scaling-best-practices.md).

## <a name="update-http-user-credentials"></a>Actualización de las credenciales de usuario HTTP

[Set-AzHDInsightGatewayCredential](https://docs.microsoft.com/powershell/module/az.hdinsight/set-azhdinsightgatewaycredential) establece las credenciales HTTP de puerta de enlace de un clúster de Azure HDInsight.

```powershell
$clusterName = "CLUSTERNAME"
$credential = Get-Credential -Message "Enter the HTTP username and password:" -UserName "admin"

Set-AzHDInsightGatewayCredential -ClusterName $clusterName -HttpCredential $credential
```

## <a name="find-the-default-storage-account"></a>Búsqueda de la cuenta de almacenamiento predeterminada

El siguiente script de PowerShell muestra cómo obtener el nombre de la cuenta de almacenamiento predeterminada y la información relacionada:

```powershell
#Connect-AzAccount
$clusterName = "<HDInsight Cluster Name>"

$clusterInfo = Get-AzHDInsightCluster -ClusterName $clusterName
$storageInfo = $clusterInfo.DefaultStorageAccount.split('.')
$defaultStoreageType = $storageInfo[1]
$defaultStorageName = $storageInfo[0]

echo "Default Storage account name: $defaultStorageName"
echo "Default Storage account type: $defaultStoreageType"

if ($defaultStoreageType -eq "blob")
{
    $defaultBlobContainerName = $cluster.DefaultStorageContainer
    $defaultStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $defaultStorageAccountName)[0].Value
    $defaultStorageAccountContext = New-AzStorageContext -StorageAccountName $defaultStorageAccountName -StorageAccountKey $defaultStorageAccountKey

    echo "Default Blob container name: $defaultBlobContainerName"
    echo "Default Storage account key: $defaultStorageAccountKey"
}
```

## <a name="find-the-resource-group"></a>Búsqueda del grupo de recursos

En el modo de Resource Manager, cada clúster de HDInsight pertenece a un grupo de recursos de Azure.  Para buscar el grupo de recursos:

```powershell
$clusterName = "<HDInsight Cluster Name>"

$cluster = Get-AzHDInsightCluster -ClusterName $clusterName
$resourceGroupName = $cluster.ResourceGroup
```

## <a name="submit-jobs"></a>Envío de trabajos

**Para enviar trabajos de MapReduce**

Vea [Ejecución de los ejemplos de MapReduce incluidos en HDInsight](hadoop/apache-hadoop-run-samples-linux.md).

**Para enviar trabajos de Apache Hive**

Consulte [Ejecución de consultas de Apache Hive con PowerShell](hadoop/apache-hadoop-use-hive-powershell.md).

**Para enviar trabajos de Apache Sqoop**

Consulte [Uso de Apache Sqoop con HDInsight](hadoop/hdinsight-use-sqoop.md).

**Para enviar trabajos de Apache Oozie**

Consulte [Uso de Apache Oozie con Apache Hadoop para definir y ejecutar un flujo de trabajo en HDInsight](hdinsight-use-oozie-linux-mac.md).

## <a name="upload-data-to-azure-blob-storage"></a>Carga de archivos de datos al almacenamiento de blobs de Azure

Consulte [Carga de datos en HDInsight](hdinsight-upload-data.md).

## <a name="see-also"></a>Consulte también

* [Cmdlets Az.HDInsight](https://docs.microsoft.com/powershell/module/az.hdinsight/?view=azps-3.1.0#hdinsight)
* [Administración de clústeres de Apache Hadoop en HDInsight mediante Azure Portal](hdinsight-administer-use-portal-linux.md)
* [Administración de HDInsight con la interfaz de la línea de comandos](hdinsight-administer-use-command-line.md)
* [Creación de clústeres de HDInsight](hdinsight-hadoop-provision-linux-clusters.md)
* [Envío de trabajos de Apache Hadoop mediante programación](hadoop/submit-apache-hadoop-jobs-programmatically.md)
* [Introducción a Azure HDInsight](hadoop/apache-hadoop-linux-tutorial-get-started.md)
