---
title: Compilación de la primera factoría de datos (plantilla de Resource Manager)
description: En este tutorial, creará una canalización de Azure Data Factory de ejemplo con la plantilla de Azure Resource Manager.
services: data-factory
documentationcenter: ''
author: djpmsft
ms.author: daperlov
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.date: 01/22/2018
ms.openlocfilehash: 5024b1091e84c0d1583ce3e455f29e0d9b9c2fc4
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543272"
---
# <a name="tutorial-build-your-first-azure-data-factory-using-azure-resource-manager-template"></a>Tutorial: Compilación de la primera Data Factory de Azure con la plantilla de Azure Resource Manager
> [!div class="op_single_selector"]
> * [Introducción y requisitos previos](data-factory-build-your-first-pipeline.md)
> * [Visual Studio](data-factory-build-your-first-pipeline-using-vs.md)
> * [PowerShell](data-factory-build-your-first-pipeline-using-powershell.md)
> * [Plantilla de Resource Manager](data-factory-build-your-first-pipeline-using-arm.md)
> * [REST API](data-factory-build-your-first-pipeline-using-rest-api.md)
> 
 
> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si utiliza la versión actual del servicio Data Factory, consulte el artículo [Inicio rápido: Creación de una factoría de datos con Azure Data Factory](../quickstart-create-data-factory-dot-net.md).

En este artículo, usará una plantilla de Azure Resource Manager para crear su primera instancia de Azure Data Factory. Para realizar el tutorial con otros SDK/herramientas, seleccione una de las opciones de la lista desplegable.

La canalización de este tutorial tiene solo una actividad: **Actividad de Hive de HDInsight**. Esta actividad ejecuta un script de Hive en un clúster de Azure HDInsight que transforma los datos de entrada para generar datos de salida. La canalización está programada para ejecutarse una vez al mes entre las horas de inicio y finalización especificadas. 

> [!NOTE]
> En este tutorial, la canalización de datos transforma los datos de entrada para generar datos de salida. Para ver un tutorial acerca de cómo copiar datos mediante Azure Data Factory, consulte [Tutorial: Copia de datos de Blob Storage en SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).
> 
> La canalización de este tutorial tiene solo una actividad de tipo: HDInsightHive. pero cualquier canalización puede tener más de una actividad. También puede encadenar dos actividades (ejecutar una después de otra) haciendo que el conjunto de datos de salida de una actividad sea el conjunto de datos de entrada de la otra actividad. Para más información, consulte [Programación y ejecución en Data Factory](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline). 

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

* Lea el artículo [Tutorial: Compilación de la primera canalización para procesar datos mediante el clúster de Hadoop](data-factory-build-your-first-pipeline.md) y complete los pasos de los **requisitos previos** .
* Siga las instrucciones del artículo [Cómo instalar y configurar Azure PowerShell](/powershell/azure/) para instalar la última versión de Azure PowerShell en su equipo.
* Para más información sobre las plantillas de Azure Resource Manager (ARM), consulte [Creación de plantillas del Administrador de recursos de Azure](../../azure-resource-manager/templates/template-syntax.md) . 

## <a name="in-this-tutorial"></a>Apartados de este tutorial

| Entidad | Descripción |
| --- | --- |
| Servicio vinculado de Azure Storage |Vincula la cuenta de Azure Storage a la factoría de datos. La cuenta de Azure Storage contiene los datos de entrada y salida de la canalización de este ejemplo. |
| Servicio vinculado a petición de HDInsight |Vincula un clúster de HDInsight a petición a la factoría de datos. El clúster se crea automáticamente para procesar los datos y se elimina después de que termina el procesamiento. |
| Conjunto de datos de entrada de blob de Azure |Hace referencia al servicio vinculado de Azure Storage. El servicio vinculado hace referencia a una cuenta de Azure Storage y los conjuntos de datos del blob de Azure especifican el contenedor, la carpeta y el nombre de archivo del almacenamiento que contiene los datos de entrada. |
| Conjunto de datos de salida de blob de Azure |Hace referencia al servicio vinculado de Azure Storage. El servicio vinculado hace referencia a una cuenta de Azure Storage y el conjunto de datos de Azure Blob especifica el contenedor, la carpeta y el nombre de archivo del almacenamiento que contiene los datos de salida. |
| Canalización de datos |La canalización tiene una actividad de tipo HDInsightHive, que consume el conjunto de datos de entrada y genera el conjunto de datos de salida. |

Una factoría de datos puede tener una o más canalizaciones. Una canalización puede tener una o más actividades. Hay dos tipos de actividades: [actividades de movimiento de datos](data-factory-data-movement-activities.md) y [actividades de transformación de datos](data-factory-data-transformation-activities.md). En este tutorial se crea una canalización con una actividad (actividad de Hive).

En la siguiente sección se proporciona la plantilla completa de Resource Manager para definir las entidades de Data Factory para que pueda ejecutar el tutorial rápidamente y probar la plantilla. Para entender cómo se define cada entidad de Data Factory, consulte la sección [Entidades de Data Factory en la plantilla](#data-factory-entities-in-the-template). Para obtener información sobre la sintaxis y las propiedades de JSON para los recursos de la factoría de datos en una plantilla, consulte [Tipos de recurso Microsoft.DataFactory](/azure/templates/microsoft.datafactory/allversions).

## <a name="data-factory-json-template"></a>Plantilla JSON de Data Factory
La plantilla de Resource Manager de nivel superior para definir una factoría de datos es: 

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": { ...
    },
    "variables": { ...
    },
    "resources": [
        {
            "name": "[parameters('dataFactoryName')]",
            "apiVersion": "[variables('apiVersion')]",
            "type": "Microsoft.DataFactory/factories",
            "location": "westus",
            "resources": [
                { ... },
                { ... },
                { ... },
                { ... }
            ]
        }
    ]
}
```
Cree un archivo JSON llamado **ADFTutorialARM.json** en la carpeta **C:\ADFGetStarted** con el siguiente contenido:

```json
{
    "contentVersion": "1.0.0.0",
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "parameters": {
        "storageAccountName": { "type": "string", "metadata": { "description": "Name of the Azure storage account that contains the input/output data." } },
          "storageAccountKey": { "type": "securestring", "metadata": { "description": "Key for the Azure storage account." } },
          "blobContainer": { "type": "string", "metadata": { "description": "Name of the blob container in the Azure Storage account." } },
          "inputBlobFolder": { "type": "string", "metadata": { "description": "The folder in the blob container that has the input file." } },
          "inputBlobName": { "type": "string", "metadata": { "description": "Name of the input file/blob." } },
          "outputBlobFolder": { "type": "string", "metadata": { "description": "The folder in the blob container that will hold the transformed data." } },
          "hiveScriptFolder": { "type": "string", "metadata": { "description": "The folder in the blob container that contains the Hive query file." } },
          "hiveScriptFile": { "type": "string", "metadata": { "description": "Name of the hive query (HQL) file." } }
    },
    "variables": {
          "dataFactoryName": "[concat('HiveTransformDF', uniqueString(resourceGroup().id))]",
          "azureStorageLinkedServiceName": "AzureStorageLinkedService",
          "hdInsightOnDemandLinkedServiceName": "HDInsightOnDemandLinkedService",
          "blobInputDatasetName": "AzureBlobInput",
          "blobOutputDatasetName": "AzureBlobOutput",
          "pipelineName": "HiveTransformPipeline"
    },
    "resources": [
      {
        "name": "[variables('dataFactoryName')]",
        "apiVersion": "2015-10-01",
        "type": "Microsoft.DataFactory/factories",
        "location": "West US",
        "resources": [
          {
            "type": "linkedservices",
            "name": "[variables('azureStorageLinkedServiceName')]",
            "dependsOn": [
                  "[variables('dataFactoryName')]"
            ],
            "apiVersion": "2015-10-01",
            "properties": {
                  "type": "AzureStorage",
                  "description": "Azure Storage linked service",
                  "typeProperties": {
                    "connectionString": "[concat('DefaultEndpointsProtocol=https;AccountName=',parameters('storageAccountName'),';AccountKey=',parameters('storageAccountKey'))]"
                  }
            }
          },
          {
            "type": "linkedservices",
            "name": "[variables('hdInsightOnDemandLinkedServiceName')]",
            "dependsOn": [
                  "[variables('dataFactoryName')]",
                  "[variables('azureStorageLinkedServiceName')]"
            ],
            "apiVersion": "2015-10-01",
            "properties": {
                  "type": "HDInsightOnDemand",
                  "typeProperties": {
                    "version": "3.5",
                    "clusterSize": 1,
                    "timeToLive": "00:05:00",
                    "osType": "Linux",
                    "linkedServiceName": "[variables('azureStorageLinkedServiceName')]"
                  }
            }
          },
          {
            "type": "datasets",
            "name": "[variables('blobInputDatasetName')]",
            "dependsOn": [
                  "[variables('dataFactoryName')]",
                  "[variables('azureStorageLinkedServiceName')]"
            ],
            "apiVersion": "2015-10-01",
            "properties": {
                  "type": "AzureBlob",
                  "linkedServiceName": "[variables('azureStorageLinkedServiceName')]",
                  "typeProperties": {
                    "fileName": "[parameters('inputBlobName')]",
                    "folderPath": "[concat(parameters('blobContainer'), '/', parameters('inputBlobFolder'))]",
                    "format": {
                          "type": "TextFormat",
                          "columnDelimiter": ","
                    }
                  },
                  "availability": {
                    "frequency": "Month",
                    "interval": 1
                  },
                  "external": true
            }
          },
          {
            "type": "datasets",
            "name": "[variables('blobOutputDatasetName')]",
            "dependsOn": [
                  "[variables('dataFactoryName')]",
                  "[variables('azureStorageLinkedServiceName')]"
            ],
            "apiVersion": "2015-10-01",
            "properties": {
                  "type": "AzureBlob",
                  "linkedServiceName": "[variables('azureStorageLinkedServiceName')]",
                  "typeProperties": {
                    "folderPath": "[concat(parameters('blobContainer'), '/', parameters('outputBlobFolder'))]",
                    "format": {
                          "type": "TextFormat",
                          "columnDelimiter": ","
                    }
                  },
                  "availability": {
                    "frequency": "Month",
                    "interval": 1
                  }
            }
          },
          {
            "type": "datapipelines",
            "name": "[variables('pipelineName')]",
            "dependsOn": [
                  "[variables('dataFactoryName')]",
                  "[variables('azureStorageLinkedServiceName')]",
                  "[variables('hdInsightOnDemandLinkedServiceName')]",
                  "[variables('blobInputDatasetName')]",
                  "[variables('blobOutputDatasetName')]"
            ],
            "apiVersion": "2015-10-01",
            "properties": {
                  "description": "Pipeline that transforms data using Hive script.",
                  "activities": [
                {
                      "type": "HDInsightHive",
                      "typeProperties": {
                        "scriptPath": "[concat(parameters('blobContainer'), '/', parameters('hiveScriptFolder'), '/', parameters('hiveScriptFile'))]",
                        "scriptLinkedService": "[variables('azureStorageLinkedServiceName')]",
                        "defines": {
                              "inputtable": "[concat('wasb://', parameters('blobContainer'), '@', parameters('storageAccountName'), '.blob.core.windows.net/', parameters('inputBlobFolder'))]",
                              "partitionedtable": "[concat('wasb://', parameters('blobContainer'), '@', parameters('storageAccountName'), '.blob.core.windows.net/', parameters('outputBlobFolder'))]"
                        }
                      },
                      "inputs": [
                        {
                              "name": "[variables('blobInputDatasetName')]"
                        }
                      ],
                      "outputs": [
                        {
                              "name": "[variables('blobOutputDatasetName')]"
                        }
                      ],
                      "policy": {
                        "concurrency": 1,
                        "retry": 3
                      },
                      "scheduler": {
                        "frequency": "Month",
                        "interval": 1
                      },
                      "name": "RunSampleHiveActivity",
                      "linkedServiceName": "[variables('hdInsightOnDemandLinkedServiceName')]"
                }
                  ],
                  "start": "2017-07-01T00:00:00Z",
                  "end": "2017-07-02T00:00:00Z",
                  "isPaused": false
              }
          }
        ]
      }
    ]
}
```

> [!NOTE]
> Puede encontrar otro ejemplo de plantilla de Resource Manager para crear una instancia de Data Factory de Azure en [Tutorial: Creación de una canalización para copiar datos mediante el uso de plantillas de Azure Resource Manager](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md).  
> 
> 

## <a name="parameters-json"></a>Parámetros JSON
Cree un archivo JSON denominado **ADFTutorialARM Parameters.json** que contenga parámetros para la plantilla de Azure Resource Manager.  

> [!IMPORTANT]
> Especifique el nombre y la clave de la cuenta de Azure Storage para los parámetros **storageAccountName** y **storageAccountKey** en este archivo de parámetros. 
> 
> 

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storageAccountName": {
            "value": "<Name of your Azure Storage account>"
        },
        "storageAccountKey": {
            "value": "<Key of your Azure Storage account>"
        },
        "blobContainer": {
            "value": "adfgetstarted"
        },
        "inputBlobFolder": {
            "value": "inputdata"
        },
        "inputBlobName": {
            "value": "input.log"
        },
        "outputBlobFolder": {
            "value": "partitioneddata"
        },
        "hiveScriptFolder": {
              "value": "script"
        },
        "hiveScriptFile": {
              "value": "partitionweblogs.hql"
        }
    }
}
```

> [!IMPORTANT]
> Puede tener archivos JSON de parámetros independientes para los entornos de desarrollo, pruebas y producción, que puede usar con la misma plantilla JSON de Data Factory. Al usar un script de PowerShell, puede automatizar la implementación de las entidades de Data Factory en estos entornos. 
> 
> 

## <a name="create-data-factory"></a>Creación de Data Factory
1. Inicie **Azure PowerShell** y ejecute el siguiente comando: 
   * Ejecute el siguiente comando y escriba el nombre de usuario y la contraseña que utiliza para iniciar sesión en el Portal de Azure.
     ```PowerShell
     Connect-AzAccount
     ```  
   * Ejecute el siguiente comando para ver todas las suscripciones para esta cuenta.
     ```PowerShell
     Get-AzSubscription
     ``` 
   * Ejecute el comando siguiente para seleccionar la suscripción con la que desea trabajar. Esta suscripción debe ser la misma que la que usó en el Portal de Azure.
     ```
     Get-AzSubscription -SubscriptionName <SUBSCRIPTION NAME> | Set-AzContext
     ```   
2. Ejecute el siguiente comando para implementar las entidades de Data Factory mediante la plantilla de Resource Manager que creó en el paso 1. 

    ```PowerShell
    New-AzResourceGroupDeployment -Name MyARMDeployment -ResourceGroupName ADFTutorialResourceGroup -TemplateFile C:\ADFGetStarted\ADFTutorialARM.json -TemplateParameterFile C:\ADFGetStarted\ADFTutorialARM-Parameters.json
    ```

## <a name="monitor-pipeline"></a>Supervisión de la canalización
1. Después de iniciar sesión en [Azure Portal](https://portal.azure.com/), haga clic en **Examinar** y seleccione **Factorías de datos**.
     ![Examinar-&gt;Factorías de datos](./media/data-factory-build-your-first-pipeline-using-arm/BrowseDataFactories.png)
2. En la hoja **Factorías de datos**, haga clic en la factoría de datos (**TutorialFactoryARM**) que creó.    
3. En la hoja **Factoría de datos** de su factoría de datos, haga clic en **Diagrama**.

     ![Icono Diagrama](./media/data-factory-build-your-first-pipeline-using-arm/DiagramTile.png)
4. En la **Vista de diagrama**, se ve información general de las canalizaciones y los conjuntos de datos empleados en este tutorial.
   
   ![Vista de diagrama](./media/data-factory-build-your-first-pipeline-using-arm/DiagramView.png) 
5. En la Vista de diagrama, haga doble clic en el conjunto de datos **AzureBlobOutput**. Se ve que el segmento se está procesando.
   
    ![Dataset](./media/data-factory-build-your-first-pipeline-using-arm/AzureBlobOutput.png)
6. Cuando finalice el procesamiento, el segmento aparece con el estado **Listo** . La creación de un clúster de HDInsight a petición normalmente tarda algún tiempo (20 minutos aproximadamente). Por tanto, cabe esperar que la canalización tarde **aproximadamente 30 minutos** en procesar el segmento.
   
    ![Dataset](./media/data-factory-build-your-first-pipeline-using-arm/SliceReady.png)    
7. Cuando el segmento tenga el estado **Listo**, busque los datos de salida en la carpeta **partitioneddata** del contenedor **adfgetstarted** del almacenamiento de blobs.  

Para obtener instrucciones sobre cómo usar las hojas del Portal de Azure para supervisar la canalización y los conjuntos de datos creados en este tutorial, consulte [Supervisión de conjuntos de datos y canalizaciones](data-factory-monitor-manage-pipelines.md) .

También puede supervisar y administrar la aplicación para supervisar las canalizaciones de datos. Para más información acerca del uso de la aplicación, consulte [Supervisión y administración de canalizaciones de Azure Data Factory mediante la nueva Aplicación de supervisión y administración](data-factory-monitor-manage-app.md) . 

> [!IMPORTANT]
> El archivo de entrada se elimina cuando el segmento se procesa correctamente. Por lo tanto, si desea volver a ejecutar el segmento o volver a realizar el tutorial, cargue el archivo de entrada (input.log) en la carpeta inputdata del contenedor adfgetstarted.
> 
> 

## <a name="data-factory-entities-in-the-template"></a>Entidades de Data Factory en la plantilla
### <a name="define-data-factory"></a>Definición de factoría de datos
Puede definir una factoría de datos en la plantilla de Resource Manager como se muestra en el ejemplo siguiente:  

```json
"resources": [
{
    "name": "[variables('dataFactoryName')]",
    "apiVersion": "2015-10-01",
    "type": "Microsoft.DataFactory/factories",
    "location": "West US"
}
```
El valor de dataFactoryName se define como: 

```json
"dataFactoryName": "[concat('HiveTransformDF', uniqueString(resourceGroup().id))]",
```
Es una cadena única basada en el identificador del grupo de recursos.  

### <a name="defining-data-factory-entities"></a>Definición de las entidades de Data Factory
Las siguientes entidades de Data Factory se definen en la plantilla JSON: 

* [Servicio vinculado de Azure Storage](#azure-storage-linked-service)
* [Servicio vinculado a petición de HDInsight](#hdinsight-on-demand-linked-service)
* [Conjunto de datos de entrada de blob de Azure:](#azure-blob-input-dataset)
* [Conjunto de datos de salida de blob de Azure](#azure-blob-output-dataset)
* [Canalización de datos con una actividad de copia](#data-pipeline)

#### <a name="azure-storage-linked-service"></a>Servicio vinculado de Azure Storage
Especifique el nombre y la clave de la cuenta de almacenamiento de Azure en esta sección. Consulte [Servicio vinculado de Azure Storage](data-factory-azure-blob-connector.md#azure-storage-linked-service) para más información sobre las propiedades JSON usadas para definir un servicio vinculado de Azure Storage. 

```json
{
    "type": "linkedservices",
    "name": "[variables('azureStorageLinkedServiceName')]",
    "dependsOn": [
        "[variables('dataFactoryName')]"
    ],
    "apiVersion": "2015-10-01",
    "properties": {
        "type": "AzureStorage",
        "description": "Azure Storage linked service",
        "typeProperties": {
            "connectionString": "[concat('DefaultEndpointsProtocol=https;AccountName=',parameters('storageAccountName'),';AccountKey=',parameters('storageAccountKey'))]"
        }
    }
}
```
**connectionString** usa los parámetros storageAccountName y storageAccountKey. Los valores de estos parámetros se pasan mediante el uso de un archivo de configuración. La definición también usa las variables azureStroageLinkedService y dataFactoryName definidas en la plantilla. 

#### <a name="hdinsight-on-demand-linked-service"></a>Servicio vinculado a petición de HDInsight
Consulte el artículo [Servicios vinculados de proceso](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) para más información sobre las propiedades JSON que se usan para definir un servicio vinculado a petición de HDInsight.  

```json
{
    "type": "linkedservices",
    "name": "[variables('hdInsightOnDemandLinkedServiceName')]",
    "dependsOn": [
        "[variables('dataFactoryName')]"
    ],
    "apiVersion": "2015-10-01",
    "properties": {
        "type": "HDInsightOnDemand",
        "typeProperties": {
            "version": "3.5",
            "clusterSize": 1,
            "timeToLive": "00:05:00",
            "osType": "Linux",
            "linkedServiceName": "[variables('azureStorageLinkedServiceName')]"
        }
    }
}
```

Tenga en cuenta los siguientes puntos: 

* Data Factory crea un clúster de HDInsight **basado en Linux** con el código JSON anterior. Para más información, consulte la sección [Servicio vinculado a petición de HDInsight de Azure](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) . 
* Puede usar **su propio clúster de HDInsight** en lugar de usar un clúster de HDInsight a petición. Para más información, consulte [Servicio vinculado de HDInsight de Azure](data-factory-compute-linked-services.md#azure-hdinsight-linked-service) .
* El clúster de HDInsight crea un **contenedor predeterminado** en el almacenamiento de blobs que especificó en JSON (**linkedServiceName**). HDInsight no elimina este contenedor cuando se elimina el clúster. Este comportamiento es así por diseño. Con el servicio vinculado de HDInsight a petición se crea un clúster de HDInsight cada vez tenga que procesarse un segmento, a menos que haya un clúster existente activo (**timeToLive**), que se elimina cuando finaliza el procesamiento.
  
    A medida que se procesen más segmentos, verá numerosos contenedores en su Almacenamiento de blobs de Azure. Si no los necesita para solucionar problemas de trabajos, puede eliminarlos para reducir el costo de almacenamiento. Los nombres de estos contenedores siguen este patrón: "adf**nombredefactoría dedatos**-**nombredelserviciovinculado**-marcadefechayhora". Use herramientas como el [Explorador de Microsoft Azure Storage](https://storageexplorer.com/) para eliminar contenedores de Azure Blob Storage.

Para más información, consulte la sección [Servicio vinculado a petición de HDInsight de Azure](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) .

#### <a name="azure-blob-input-dataset"></a>Conjunto de datos de entrada de blob de Azure
Especifique los nombres del contenedor de blobs, la carpeta y el archivo que contiene los datos de entrada. Consulte las [propiedades del conjunto de datos de Azure Blob](data-factory-azure-blob-connector.md#dataset-properties) para más información sobre las propiedades JSON usadas para definir un conjunto de datos de Azure Blob. 

```json
{
    "type": "datasets",
    "name": "[variables('blobInputDatasetName')]",
    "dependsOn": [
        "[variables('dataFactoryName')]",
        "[variables('azureStorageLinkedServiceName')]"
    ],
    "apiVersion": "2015-10-01",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "[variables('azureStorageLinkedServiceName')]",
        "typeProperties": {
            "fileName": "[parameters('inputBlobName')]",
            "folderPath": "[concat(parameters('blobContainer'), '/', parameters('inputBlobFolder'))]",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        },
        "availability": {
            "frequency": "Month",
            "interval": 1
        },
        "external": true
    }
}
```
Esta definición usa los siguientes parámetros definidos en la plantilla de parámetros: blobContainer, inputBlobFolder y inputBlobName. 

#### <a name="azure-blob-output-dataset"></a>Conjunto de datos de salida de blob de Azure
Especifique los nombres del contenedor de blobs y la carpeta que contiene los datos de salida. Consulte las [propiedades del conjunto de datos de Azure Blob](data-factory-azure-blob-connector.md#dataset-properties) para más información sobre las propiedades JSON usadas para definir un conjunto de datos de Azure Blob.  

```json
{
    "type": "datasets",
    "name": "[variables('blobOutputDatasetName')]",
    "dependsOn": [
        "[variables('dataFactoryName')]",
        "[variables('azureStorageLinkedServiceName')]"
    ],
    "apiVersion": "2015-10-01",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "[variables('azureStorageLinkedServiceName')]",
        "typeProperties": {
            "folderPath": "[concat(parameters('blobContainer'), '/', parameters('outputBlobFolder'))]",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        },
        "availability": {
            "frequency": "Month",
            "interval": 1
        }
    }
}
```

Esta definición usa los siguientes parámetros definidos en la plantilla de parámetros: blobContainer y outputBlobFolder. 

#### <a name="data-pipeline"></a>Canalización de datos
Defina una canalización que transforme los datos mediante la ejecución de un script de Hive en un clúster a petición de Azure HDInsight. Consulte [JSON de canalización](data-factory-create-pipelines.md#pipeline-json) para obtener descripciones de elementos JSON que se usan para definir una canalización en este ejemplo. 

```json
{
    "type": "datapipelines",
    "name": "[variables('pipelineName')]",
    "dependsOn": [
        "[variables('dataFactoryName')]",
        "[variables('azureStorageLinkedServiceName')]",
        "[variables('hdInsightOnDemandLinkedServiceName')]",
        "[variables('blobInputDatasetName')]",
        "[variables('blobOutputDatasetName')]"
    ],
    "apiVersion": "2015-10-01",
    "properties": {
        "description": "Pipeline that transforms data using Hive script.",
        "activities": [
        {
            "type": "HDInsightHive",
            "typeProperties": {
                "scriptPath": "[concat(parameters('blobContainer'), '/', parameters('hiveScriptFolder'), '/', parameters('hiveScriptFile'))]",
                "scriptLinkedService": "[variables('azureStorageLinkedServiceName')]",
                "defines": {
                    "inputtable": "[concat('wasb://', parameters('blobContainer'), '@', parameters('storageAccountName'), '.blob.core.windows.net/', parameters('inputBlobFolder'))]",
                    "partitionedtable": "[concat('wasb://', parameters('blobContainer'), '@', parameters('storageAccountName'), '.blob.core.windows.net/', parameters('outputBlobFolder'))]"
                }
            },
            "inputs": [
            {
                "name": "[variables('blobInputDatasetName')]"
            }
            ],
            "outputs": [
            {
                "name": "[variables('blobOutputDatasetName')]"
            }
            ],
            "policy": {
                "concurrency": 1,
                "retry": 3
            },
            "scheduler": {
                "frequency": "Month",
                "interval": 1
            },
            "name": "RunSampleHiveActivity",
            "linkedServiceName": "[variables('hdInsightOnDemandLinkedServiceName')]"
        }
        ],
        "start": "2017-07-01T00:00:00Z",
        "end": "2017-07-02T00:00:00Z",
        "isPaused": false
    }
}
```

## <a name="reuse-the-template"></a>Reutilización de la plantilla
En el tutorial, ha creado una plantilla para definir las entidades de Data Factory y una plantilla para pasar valores de parámetros. Para usar la misma plantilla para implementar las entidades de Data Factory en distintos entornos, cree un archivo de parámetros para cada entorno y utilícelo al implementarla en ese entorno.     

Ejemplo:  

```PowerShell
New-AzResourceGroupDeployment -Name MyARMDeployment -ResourceGroupName ADFTutorialResourceGroup -TemplateFile ADFTutorialARM.json -TemplateParameterFile ADFTutorialARM-Parameters-Dev.json

New-AzResourceGroupDeployment -Name MyARMDeployment -ResourceGroupName ADFTutorialResourceGroup -TemplateFile ADFTutorialARM.json -TemplateParameterFile ADFTutorialARM-Parameters-Test.json

New-AzResourceGroupDeployment -Name MyARMDeployment -ResourceGroupName ADFTutorialResourceGroup -TemplateFile ADFTutorialARM.json -TemplateParameterFile ADFTutorialARM-Parameters-Production.json
```
Tenga en cuenta que el primer comando usa el archivo de parámetros para el entorno de desarrollo, el segundo para el entorno de prueba y el tercero para el entorno de producción.  

También puede volver a usar la plantilla para llevar a cabo tareas repetidas. Por ejemplo, necesita crear muchas factorías de datos con una o varias canalizaciones que implementen la misma lógica, pero cada factoría de datos usa cuentas de Azure Storage y Azure SQL Database. En este escenario, usa la misma plantilla en el mismo entorno (desarrollo, prueba o producción) con distintos archivos de parámetros para crear factorías de datos. 

## <a name="resource-manager-template-for-creating-a-gateway"></a>Plantilla de Resource Manager para crear una puerta de enlace
Aquí aparece una plantilla de Resource Manager de ejemplo para crear una puerta de enlace lógica en la parte posterior. Instale una puerta de enlace en el equipo local o en la máquina virtual de IaaS de Azure y registrar la puerta de enlace con el servicio Data Factory mediante una clave. Para más información, consulte [Movimiento de datos entre orígenes locales y la nube con Data Management Gateway](data-factory-move-data-between-onprem-and-cloud.md) .

```json
{
    "contentVersion": "1.0.0.0",
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "parameters": {
    },
    "variables": {
        "dataFactoryName":  "GatewayUsingArmDF",
        "apiVersion": "2015-10-01",
        "singleQuote": "'"
    },
    "resources": [
        {
            "name": "[variables('dataFactoryName')]",
            "apiVersion": "[variables('apiVersion')]",
            "type": "Microsoft.DataFactory/factories",
            "location": "eastus",
            "resources": [
                {
                    "dependsOn": [ "[concat('Microsoft.DataFactory/factories/', variables('dataFactoryName'))]" ],
                    "type": "gateways",
                    "apiVersion": "[variables('apiVersion')]",
                    "name": "GatewayUsingARM",
                    "properties": {
                        "description": "my gateway"
                    }
                }            
            ]
        }
    ]
}
```
Esta plantilla crea una factoría de datos denominada GatewayUsingArmDF con una puerta de enlace llamada: GatewayUsingARM. 

## <a name="see-also"></a>Consulte también

| Tema | Descripción |
|:--- |:--- |
| [Canalizaciones](data-factory-create-pipelines.md) |Este artículo ayuda a conocer las canalizaciones y actividades de Azure Data Factory y cómo aprovecharlas para construir flujos de trabajo controlados por datos de un extremo a otro para su escenario o negocio. |
| [Conjuntos de datos](data-factory-create-datasets.md) |Este artículo le ayuda a comprender los conjuntos de datos de Azure Data Factory. |
| [Programación y ejecución con Data Factory](data-factory-scheduling-and-execution.md) |En este artículo se explican los aspectos de programación y ejecución del modelo de aplicación de Azure Data Factory. |
| [Supervisión y administración de canalizaciones de Data Factory de Azure mediante la nueva Aplicación de supervisión y administración](data-factory-monitor-manage-app.md) |En este artículo se describe cómo supervisar, administrar y depurar las canalizaciones mediante la aplicación de supervisión y administración. |

