---
title: 'Transformación de datos mediante Spark en Azure Data Factory '
description: Este tutorial proporciona instrucciones detalladas para transformar los datos mediante el uso de la actividad de Spark en Azure Data Factory.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.date: 01/22/2018
author: nabhishek
ms.author: abnarain
manager: anandsub
ms.openlocfilehash: bef80cdeab32d14aeaae350adda869a8ea7b05c7
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "81409082"
---
# <a name="transform-data-in-the-cloud-by-using-spark-activity-in-azure-data-factory"></a>Transformación de datos en la nube mediante la actividad de Spark en Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este tutorial, se usa Azure PowerShell para crear una canalización de Data Factory que transforma los datos mediante la actividad de Spark y un servicio vinculado a HDInsight a petición. En este tutorial, realizará los siguientes pasos:

> [!div class="checklist"]
> * Creación de una factoría de datos. 
> * Creación e implementación de servicios vinculados
> * Creación e implementación de una canalización 
> * Inicio de la ejecución de una canalización.
> * Supervisión de la ejecución de la canalización

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

* **Cuenta de Azure Storage**. Debe crear un script de Python y un archivo de entrada y cargarlos en Azure Storage. La salida del programa Spark se almacena en esta cuenta de almacenamiento. El clúster de Spark a petición usa la misma cuenta de almacenamiento que el almacenamiento principal.  
* **Azure PowerShell**. Siga las instrucciones de [Instalación y configuración de Azure PowerShell](/powershell/azure/install-Az-ps).


### <a name="upload-python-script-to-your-blob-storage-account"></a>Carga del script de Python en la cuenta de Blob Storage
1. Cree un archivo de Python denominado **WordCount_Spark.py** con el siguiente contenido: 

    ```python
    import sys
    from operator import add
    
    from pyspark.sql import SparkSession
    
    def main():
        spark = SparkSession\
            .builder\
            .appName("PythonWordCount")\
            .getOrCreate()
            
        lines = spark.read.text("wasbs://adftutorial@<storageaccountname>.blob.core.windows.net/spark/inputfiles/minecraftstory.txt").rdd.map(lambda r: r[0])
        counts = lines.flatMap(lambda x: x.split(' ')) \
            .map(lambda x: (x, 1)) \
            .reduceByKey(add)
        counts.saveAsTextFile("wasbs://adftutorial@<storageaccountname>.blob.core.windows.net/spark/outputfiles/wordcount")
        
        spark.stop()
    
    if __name__ == "__main__":
        main()
    ```
2. Reemplace **&lt;storageaccountname&gt;** por el nombre de la cuenta de Azure Storage. A continuación, guarde el archivo. 
3. En Azure Blob Storage, cree un contenedor denominado **adftutorial** si no existe. 
4. Cree una carpeta llamada **spark**.
5. Cree una subcarpeta denominada **script** en la carpeta **spark**. 
6. Cargue el archivo **WordCount_Spark.py** a la subcarpeta **script**. 


### <a name="upload-the-input-file"></a>Carga del archivo de entrada
1. Cree un archivo denominado **minecraftstory.txt** con algo de texto. El programa Spark contará el número de palabras de este texto. 
2. Cree una subcarpeta denominada `inputfiles` en la carpeta `spark`. 
3. Cargue `minecraftstory.txt` a la subcarpeta `inputfiles`. 

## <a name="author-linked-services"></a>Creación de servicios vinculados
En esta sección, deberá crear dos servicios vinculados: 
    
- Un servicio vinculado a Azure Storage que vincule una cuenta de Azure Storage con la factoría de datos. Este almacenamiento lo usa el clúster HDInsight a petición. También contiene el script de Spark que se ejecutará. 
- Un servicio vinculado de HDInsight a petición. Azure Data Factory crea automáticamente un clúster de HDInsight, ejecuta el programa Spark y, luego, eliminar el clúster HDInsight después de estar inactivo durante un período de configuración previa. 

### <a name="azure-storage-linked-service"></a>Servicio vinculado de Azure Storage
Cree un archivo JSON con su editor preferido, copie la siguiente definición de JSON de un servicio vinculado de Azure Storage y, a continuación, guarde el archivo como **MyStorageLinkedService.json**.  

```json
{
    "name": "MyStorageLinkedService",
    "properties": {
      "type": "AzureStorage",
      "typeProperties": {
        "connectionString": "DefaultEndpointsProtocol=https;AccountName=<storageAccountName>;AccountKey=<storageAccountKey>"
      }
    }
}
```
Actualice &lt;storageAccountName&gt; y &lt;storageAccountKey&gt; con el nombre y la clave de su cuenta de Azure Storage. 


### <a name="on-demand-hdinsight-linked-service"></a>Servicio vinculado de HDInsight a petición
Cree un archivo JSON con su editor preferido, copie la siguiente definición de JSON de un servicio vinculado de Azure HDInsight y, a continuación, guarde el archivo como **MyOnDemandSparkLinkedService.json**.  

```json
{
    "name": "MyOnDemandSparkLinkedService",
    "properties": {
      "type": "HDInsightOnDemand",
      "typeProperties": {
        "clusterSize": 2,
        "clusterType": "spark",
        "timeToLive": "00:15:00",
        "hostSubscriptionId": "<subscriptionID> ",
        "servicePrincipalId": "<servicePrincipalID>",
        "servicePrincipalKey": {
          "value": "<servicePrincipalKey>",
          "type": "SecureString"
        },
        "tenant": "<tenant ID>",
        "clusterResourceGroup": "<resourceGroupofHDICluster>",
        "version": "3.6",
        "osType": "Linux",
        "clusterNamePrefix":"ADFSparkSample",
        "linkedServiceName": {
          "referenceName": "MyStorageLinkedService",
          "type": "LinkedServiceReference"
        }
      }
    }
}
```
Actualice los valores de las siguientes propiedades en la definición de servicio vinculado: 

- **hostSubscriptionId**. Reemplace &lt;subscriptionID&gt; con el identificador de la suscripción de Azure. El clúster de HDInsight a petición se crea en esta suscripción de Azure. 
- **tenant**. Reemplace &lt;tenantID&gt; con el identificador de su inquilino de Azure. 
- **servicePrincipalId**, **servicePrincipalKey**. Reemplace &lt;servicePrincipalID&gt; y &lt;servicePrincipalKey&gt; por el identificador y la clave de la entidad de servicio de Azure Active Directory. Esta entidad de servicio debe ser miembro del rol de colaborador de la suscripción o del grupo de recursos en el que se crea el clúster. Consulte [Creación de una aplicación de Azure Active Directory y una entidad de servicio](../active-directory/develop/howto-create-service-principal-portal.md) para obtener detalles. El **Id. de entidad de servicio** es equivalente al *Id. de aplicación* y una **Clave de entidad de servicio** es equivalente al valor de un *Secreto de cliente*.
- **clusterResourceGroup**. Reemplace &lt;resourceGroupOfHDICluster&gt; por el nombre del grupo de recursos en el que se debe crear el clúster de HDInsight. 

> [!NOTE]
> Azure HDInsight tiene limitaciones en el número total de núcleos que se pueden utilizar en cada región de Azure que admite. Para el servicio vinculado de HDInsight a petición, el clúster de HDInsight se creará en la misma ubicación que Azure Storage usó como almacenamiento principal. Asegúrese de que dispone de suficientes cuotas de núcleo para que el clúster se cree correctamente. Para obtener más información, consulte [Configuración de clústeres en HDInsight con Hadoop, Spark, Kafka, etc](../hdinsight/hdinsight-hadoop-provision-linux-clusters.md). 


## <a name="author-a-pipeline"></a>Creación de una canalización 
En este paso, se crea una canalización con una actividad de Spark. La actividad usa el ejemplo de **recuento de palabras**. Descargue el contenido desde esta ubicación si aún no lo ha hecho.

Cree un archivo JSON en el editor que prefiera, copie la siguiente definición de JSON de una definición de canalización y guárdela como **MySparkOnDemandPipeline.json**. 

```json
{
  "name": "MySparkOnDemandPipeline",
  "properties": {
    "activities": [
      {
        "name": "MySparkActivity",
        "type": "HDInsightSpark",
        "linkedServiceName": {
            "referenceName": "MyOnDemandSparkLinkedService",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
          "rootPath": "adftutorial/spark",
          "entryFilePath": "script/WordCount_Spark.py",
          "getDebugInfo": "Failure",
          "sparkJobLinkedService": {
            "referenceName": "MyStorageLinkedService",
            "type": "LinkedServiceReference"
          }
        }
      }
    ]
  }
}
```

Tenga en cuenta los siguientes puntos: 

- rootPathz apunta a la carpeta de Spark del contenedor adftutorial. 
- entryFilePath apunta al archivo WordCount_Spark.py de la subcarpeta de script de la carpeta de Spark. 


## <a name="create-a-data-factory"></a>Crear una factoría de datos 
Ha creado el servicio vinculado y definiciones de canalización en archivos JSON. Ahora, vamos a crear una factoría de datos y a implementar el servicio vinculado y los archivos JSON de canalización mediante cmdlets de PowerShell. Ejecute los comandos de PowerShell siguientes uno por uno: 

1. Establezca las variables una a una.

    **Nombre del grupo de recursos**
    ```powershell
    $resourceGroupName = "ADFTutorialResourceGroup" 
    ```

    **Nombre de la factoría de datos. Debe ser globalmente único** 
    ```powershell
    $dataFactoryName = "MyDataFactory09102017"
    ```
    
    **Nombre de la canalización**
    ```powershell
    $pipelineName = "MySparkOnDemandPipeline" # Name of the pipeline
    ```
2. Inicie **PowerShell**. Mantenga Azure PowerShell abierto hasta el final de esta guía de inicio rápido. Si lo cierra y vuelve a abrirlo, deberá ejecutar los comandos de nuevo. Para una lista de las regiones de Azure en las que Data Factory está disponible actualmente, seleccione las regiones que le interesen en la página siguiente y expanda **Análisis** para poder encontrar **Data Factory**: [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/). Los almacenes de datos (Azure Storage, Azure SQL Database, etc.) y los procesos (HDInsight, etc.) que usa la factoría de datos pueden encontrarse en otras regiones.

    Ejecute el siguiente comando y escriba el nombre de usuario y la contraseña que utiliza para iniciar sesión en Azure Portal:
        
    ```powershell
    Connect-AzAccount
    ```        
    Ejecute el siguiente comando para ver todas las suscripciones de esta cuenta:

    ```powershell
    Get-AzSubscription
    ```
    Ejecute el comando siguiente para seleccionar la suscripción con la que desea trabajar. Reemplace **SubscriptionId** con el identificador de la suscripción de Azure:

    ```powershell
    Select-AzSubscription -SubscriptionId "<SubscriptionId>"    
    ```  
3. Creación del grupo de recursos: ADFTutorialResourceGroup. 

    ```powershell
    New-AzResourceGroup -Name $resourceGroupName -Location "East Us" 
    ```
4. Cree la factoría de datos. 

    ```powershell
     $df = Set-AzDataFactoryV2 -Location EastUS -Name $dataFactoryName -ResourceGroupName $resourceGroupName
    ```

    Ejecute el comando siguiente para ver la salida: 

    ```powershell
    $df
    ```
5. Cambie a la carpeta donde creó los archivos JSON y ejecute el siguiente comando para implementar un servicio vinculado a Azure Storage: 
       
    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "MyStorageLinkedService" -File "MyStorageLinkedService.json"
    ```
6. Ejecute el siguiente comando para implementar un servicio vinculado de Spark a petición: 
       
    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "MyOnDemandSparkLinkedService" -File "MyOnDemandSparkLinkedService.json"
    ```
7. Ejecute el siguiente comando para implementar una canalización: 
       
    ```powershell
    Set-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name $pipelineName -File "MySparkOnDemandPipeline.json"
    ```
    
## <a name="start-and-monitor-a-pipeline-run"></a>Inicio y supervisión de una ejecución de la canalización  

1. Inicio de la ejecución de una canalización. También se captura el id. de ejecución de la canalización para poder realizar una supervisión en un futuro.

    ```powershell
    $runId = Invoke-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineName $pipelineName  
    ```
2. Ejecute el script siguiente para comprobar continuamente el estado de ejecución de la canalización hasta que termine.

    ```powershell
    while ($True) {
        $result = Get-AzDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId $runId -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)
    
        if(!$result) {
            Write-Host "Waiting for pipeline to start..." -foregroundcolor "Yellow"
        }
        elseif (($result | Where-Object { $_.Status -eq "InProgress" } | Measure-Object).count -ne 0) {
            Write-Host "Pipeline run status: In Progress" -foregroundcolor "Yellow"
        }
        else {
            Write-Host "Pipeline '"$pipelineName"' run finished. Result:" -foregroundcolor "Yellow"
            $result
            break
        }
        ($result | Format-List | Out-String)
        Start-Sleep -Seconds 15
    }

    Write-Host "Activity `Output` section:" -foregroundcolor "Yellow"
    $result.Output -join "`r`n"

    Write-Host "Activity `Error` section:" -foregroundcolor "Yellow"
    $result.Error -join "`r`n" 
    ```  
3. Este es el resultado de la ejecución de ejemplo: 

    ```
    Pipeline run status: In Progress
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : 
    ActivityName      : MySparkActivity
    PipelineRunId     : 94e71d08-a6fa-4191-b7d1-cf8c71cb4794
    PipelineName      : MySparkOnDemandPipeline
    Input             : {rootPath, entryFilePath, getDebugInfo, sparkJobLinkedService}
    Output            : 
    LinkedServiceName : 
    ActivityRunStart  : 9/20/2017 6:33:47 AM
    ActivityRunEnd    : 
    DurationInMs      : 
    Status            : InProgress
    Error             :
    …
    
    Pipeline ' MySparkOnDemandPipeline' run finished. Result:
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : MyDataFactory09102017
    ActivityName      : MySparkActivity
    PipelineRunId     : 94e71d08-a6fa-4191-b7d1-cf8c71cb4794
    PipelineName      : MySparkOnDemandPipeline
    Input             : {rootPath, entryFilePath, getDebugInfo, sparkJobLinkedService}
    Output            : {clusterInUse, jobId, ExecutionProgress, effectiveIntegrationRuntime}
    LinkedServiceName : 
    ActivityRunStart  : 9/20/2017 6:33:47 AM
    ActivityRunEnd    : 9/20/2017 6:46:30 AM
    DurationInMs      : 763466
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}
    
    Activity Output section:
    "clusterInUse": "https://ADFSparkSamplexxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.azurehdinsight.net/"
    "jobId": "0"
    "ExecutionProgress": "Succeeded"
    "effectiveIntegrationRuntime": "DefaultIntegrationRuntime (East US)"
    Activity Error section:
    "errorCode": ""
    "message": ""
    "failureType": ""
    "target": "MySparkActivity"
    ```
4. Confirme que se haya creado una carpeta denominada `outputfiles` en la carpeta `spark` del contenedor adftutorial con la salida del programa de Spark. 


## <a name="next-steps"></a>Pasos siguientes
La canalización de este ejemplo copia los datos de una ubicación a otra en una instancia de Azure Blob Storage. Ha aprendido a: 

> [!div class="checklist"]
> * Creación de una factoría de datos. 
> * Creación e implementación de servicios vinculados
> * Creación e implementación de una canalización 
> * Inicio de la ejecución de una canalización.
> * Supervisión de la ejecución de la canalización

Pase al siguiente tutorial para obtener información sobre cómo puede transformar datos mediante la ejecución del script de Hive en un clúster de Azure HDInsight que se encuentra en una red virtual. 

> [!div class="nextstepaction"]
> [Tutorial: transformación de datos con Hive en Azure Virtual Network](tutorial-transform-data-hive-virtual-network.md).





