---
title: 'Tutorial: creación de una canalización mediante el uso de plantillas de Azure Resource Manager '
description: En este tutorial, creará una canalización de Azure Data Factory mediante una plantilla de Azure Resource Manager. Esta canalización copia datos desde una instancia de Azure Blob Storage hasta Azure SQL Database.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: ''
editor: ''
ms.assetid: 1274e11a-e004-4df5-af07-850b2de7c15e
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 831da4153eebc798265493441ee72c041901904f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87053896"
---
# <a name="tutorial-use-azure-resource-manager-template-to-create-a-data-factory-pipeline-to-copy-data"></a>Tutorial: Uso de plantillas de Azure Resource Manager para crear una canalización de Data Factory para copiar datos 
> [!div class="op_single_selector"]
> * [Introducción y requisitos previos](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [Asistente para copia](data-factory-copy-data-wizard-tutorial.md)
> * [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
> * [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
> * [Plantilla de Azure Resource Manager](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
> * [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
> * [API de .NET](data-factory-copy-activity-tutorial-using-dotnet-api.md)
> 

> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si utiliza la versión actual del servicio Data Factory, consulte el [tutorial de la actividad de copia](../quickstart-create-data-factory-dot-net.md). 

En este tutorial se muestra cómo usar una plantilla de Azure Resource Manager para crear una instancia de Azure Data Factory. La canalización de datos de este tutorial copia datos de un almacén de datos de origen a un almacén de datos de destino. No transforma los datos de entrada para generar datos de salida. Para ver un tutorial acerca de cómo transformar datos mediante Azure Data Factory, consulte [Tutorial: Compilación de una canalización para transformar datos mediante el clúster de Hadoop](data-factory-build-your-first-pipeline.md).

En este tutorial, creará una canalización con una actividad en ella: la actividad de copia. La actividad de copia realiza la copia de los datos de un almacén de datos admitido en un almacén de datos receptor. Para obtener una lista de almacenes de datos que se admiten como orígenes y receptores, consulte los [almacenes de datos admitidos](data-factory-data-movement-activities.md#supported-data-stores-and-formats). La actividad funciona con un servicio disponible de forma global que puede copiar datos entre varios almacenes de datos de forma segura, confiable y escalable. Para más información acerca de la actividad de copia, consulte las [actividades de movimiento de datos](data-factory-data-movement-activities.md).

pero cualquier canalización puede tener más de una actividad. También puede encadenar dos actividades (ejecutar una después de otra) haciendo que el conjunto de datos de salida de una actividad sea el conjunto de datos de entrada de la otra actividad. Para más información, consulte [Varias actividades en una canalización](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline). 

> [!NOTE] 
> La canalización de datos de este tutorial copia datos de un almacén de datos de origen a un almacén de datos de destino. Para ver un tutorial acerca de cómo transformar datos mediante Azure Data Factory, consulte [Tutorial: Compilación de una canalización para transformar datos mediante el clúster de Hadoop](data-factory-build-your-first-pipeline.md). 

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

* Consulte [el tutorial de introducción y de requisitos previos](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) y complete los pasos de los **requisitos previos**.
* Siga las instrucciones del artículo [Cómo instalar y configurar Azure PowerShell](/powershell/azure/) para instalar la última versión de Azure PowerShell en su equipo. En este tutorial, utiliza PowerShell para implementar las entidades de Data Factory. 
* (Opcional) Para más información sobre las plantillas de Azure Resource Manager (ARM), consulte [Creación de plantillas de Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md).

## <a name="in-this-tutorial"></a>Apartados de este tutorial
En este tutorial, va a crear una factoría de datos con las siguientes entidades de Data Factory:

| Entidad | Descripción |
| --- | --- |
| Servicio vinculado de Azure Storage |Vincula la cuenta de Azure Storage a la factoría de datos. Azure Storage es el almacén de datos de origen y Azure SQL Database es el almacén de datos receptor para la actividad de copia del tutorial. Especifica la cuenta de almacenamiento que contiene los datos de entrada para la actividad de copia. |
| Servicio vinculado de Azure SQL Database |Vincula la base de datos de Azure SQL Database a la factoría de datos. Especifica la base de datos que contiene los datos de salida para la actividad de copia. |
| Conjunto de datos de entrada de blob de Azure |Hace referencia al servicio vinculado de Azure Storage. El servicio vinculado hace referencia a una cuenta de Azure Storage y los conjuntos de datos del blob de Azure especifican el contenedor, la carpeta y el nombre de archivo del almacenamiento que contiene los datos de entrada. |
| Conjunto de datos de salida SQL de Azure |Hace referencia al servicio vinculado de Azure SQL. El servicio vinculado de SQL Azure hace referencia a un servidor SQL lógico y el conjunto de datos de Azure SQL especifica el nombre de la tabla que contiene los datos de salida. |
| Canalización de datos |La canalización tiene una actividad de tipo Copiar que toma el conjunto de datos del blob de Azure como entrada y el conjunto de datos de Azure SQL como salida. Esta actividad copia datos desde un blob de Azure hasta una tabla de Azure SQL Database. |

Una factoría de datos puede tener una o más canalizaciones. Una canalización puede tener una o más actividades. Hay dos tipos de actividades: [actividades de movimiento de datos](data-factory-data-movement-activities.md) y [actividades de transformación de datos](data-factory-data-transformation-activities.md). En este tutorial, va a crear una canalización con una actividad (actividad de copia).

![Copia de un blob de Azure a Azure SQL Database](media/data-factory-copy-activity-tutorial-using-azure-resource-manager-template/CopyBlob2SqlDiagram.png) 

En la siguiente sección se proporciona la plantilla completa de Resource Manager para definir las entidades de Data Factory para que pueda ejecutar el tutorial rápidamente y probar la plantilla. Para entender cómo se define cada entidad de Data Factory, consulte la sección [Entidades de Data Factory en la plantilla](#data-factory-entities-in-the-template).

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
            "type": "Microsoft.DataFactory/datafactories",
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
Cree un archivo JSON llamado **ADFCopyTutorialARM.json** en la carpeta **C:\ADFGetStarted** con el siguiente contenido:

```json
{
    "contentVersion": "1.0.0.0",
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "parameters": {
      "storageAccountName": { "type": "string", "metadata": { "description": "Name of the Azure storage account that contains the data to be copied." } },
      "storageAccountKey": { "type": "securestring", "metadata": { "description": "Key for the Azure storage account." } },
      "sourceBlobContainer": { "type": "string", "metadata": { "description": "Name of the blob container in the Azure Storage account." } },
      "sourceBlobName": { "type": "string", "metadata": { "description": "Name of the blob in the container that has the data to be copied to Azure SQL Database table" } },
      "sqlServerName": { "type": "string", "metadata": { "description": "Name of the logical SQL server that will hold the output/copied data." } },
      "databaseName": { "type": "string", "metadata": { "description": "Name of the Azure SQL Database in the logical SQL server." } },
      "sqlServerUserName": { "type": "string", "metadata": { "description": "Name of the user that has access to the logical SQL server." } },
      "sqlServerPassword": { "type": "securestring", "metadata": { "description": "Password for the user." } },
      "targetSQLTable": { "type": "string", "metadata": { "description": "Table in the Azure SQL Database that will hold the copied data." } 
      } 
    },
    "variables": {
      "dataFactoryName": "[concat('AzureBlobToAzureSQLDatabaseDF', uniqueString(resourceGroup().id))]",
      "azureSqlLinkedServiceName": "AzureSqlLinkedService",
      "azureStorageLinkedServiceName": "AzureStorageLinkedService",
      "blobInputDatasetName": "BlobInputDataset",
      "sqlOutputDatasetName": "SQLOutputDataset",
      "pipelineName": "Blob2SQLPipeline"
    },
    "resources": [
      {
        "name": "[variables('dataFactoryName')]",
        "apiVersion": "2015-10-01",
        "type": "Microsoft.DataFactory/datafactories",
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
            "name": "[variables('azureSqlLinkedServiceName')]",
            "dependsOn": [
              "[variables('dataFactoryName')]"
            ],
            "apiVersion": "2015-10-01",
            "properties": {
              "type": "AzureSqlDatabase",
              "description": "Azure SQL linked service",
              "typeProperties": {
                "connectionString": "[concat('Server=tcp:',parameters('sqlServerName'),'.database.windows.net,1433;Database=', parameters('databaseName'), ';User ID=',parameters('sqlServerUserName'),';Password=',parameters('sqlServerPassword'),';Trusted_Connection=False;Encrypt=True;Connection Timeout=30')]"
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
              "structure": [
                {
                  "name": "Column0",
                  "type": "String"
                },
                {
                  "name": "Column1",
                  "type": "String"
                }
              ],
              "typeProperties": {
                "folderPath": "[concat(parameters('sourceBlobContainer'), '/')]",
                "fileName": "[parameters('sourceBlobName')]",
                "format": {
                  "type": "TextFormat",
                  "columnDelimiter": ","
                }
              },
              "availability": {
                "frequency": "Hour",
                "interval": 1
              },
              "external": true
            }
          },
          {
            "type": "datasets",
            "name": "[variables('sqlOutputDatasetName')]",
            "dependsOn": [
              "[variables('dataFactoryName')]",
              "[variables('azureSqlLinkedServiceName')]"
            ],
            "apiVersion": "2015-10-01",
            "properties": {
              "type": "AzureSqlTable",
              "linkedServiceName": "[variables('azureSqlLinkedServiceName')]",
              "structure": [
                {
                  "name": "FirstName",
                  "type": "String"
                },
                {
                  "name": "LastName",
                  "type": "String"
                }
              ],
              "typeProperties": {
                "tableName": "[parameters('targetSQLTable')]"
              },
              "availability": {
                "frequency": "Hour",
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
              "[variables('azureSqlLinkedServiceName')]",
              "[variables('blobInputDatasetName')]",
              "[variables('sqlOutputDatasetName')]"
            ],
            "apiVersion": "2015-10-01",
            "properties": {
              "activities": [
                {
                  "name": "CopyFromAzureBlobToAzureSQL",
                  "description": "Copy data frm Azure blob to Azure SQL",
                  "type": "Copy",
                  "inputs": [
                    {
                      "name": "[variables('blobInputDatasetName')]"
                    }
                  ],
                  "outputs": [
                    {
                      "name": "[variables('sqlOutputDatasetName')]"
                    }
                  ],
                  "typeProperties": {
                    "source": {
                      "type": "BlobSource"
                    },
                    "sink": {
                      "type": "SqlSink",
                      "sqlWriterCleanupScript": "$$Text.Format('DELETE FROM {0}', 'emp')"
                    },
                    "translator": {
                      "type": "TabularTranslator",
                      "columnMappings": "Column0:FirstName,Column1:LastName"
                    }
                  },
                  "Policy": {
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst",
                    "retry": 3,
                    "timeout": "01:00:00"
                  }
                }
              ],
              "start": "2017-05-11T00:00:00Z",
              "end": "2017-05-12T00:00:00Z"
            }
          }
        ]
      }
    ]
  }
```

## <a name="parameters-json"></a>Parámetros JSON
Cree un archivo JSON denominado **ADFCopyTutorialARM Parameters.json** que contiene parámetros para la plantilla de Azure Resource Manager. 

> [!IMPORTANT]
> Especifique el nombre y la clave de la cuenta de Azure Storage para los parámetros storageAccountName y storageAccountKey.  
> 
> Especifique el servidor SQL lógico, la base de datos, el usuario y la contraseña de los parámetros sqlServerName, databaseName, sqlServerUserName y sqlServerPassword.  

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": { 
        "storageAccountName": { "value": "<Name of the Azure storage account>"    },
        "storageAccountKey": {
            "value": "<Key for the Azure storage account>"
        },
        "sourceBlobContainer": { "value": "adftutorial" },
        "sourceBlobName": { "value": "emp.txt" },
        "sqlServerName": { "value": "<Name of the logical SQL server>" },
        "databaseName": { "value": "<Name of the database>" },
        "sqlServerUserName": { "value": "<Name of the user who has access to the database>" },
        "sqlServerPassword": { "value": "<password for the user>" },
        "targetSQLTable": { "value": "emp" }
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
   * Ejecute el comando siguiente para seleccionar la suscripción con la que desea trabajar.
    
    ```PowerShell
    Get-AzSubscription -SubscriptionName <SUBSCRIPTION NAME> | Set-AzContext
    ```    
2. Ejecute el siguiente comando para implementar las entidades de Data Factory mediante la plantilla de Resource Manager que creó en el paso 1.

    ```PowerShell   
    New-AzResourceGroupDeployment -Name MyARMDeployment -ResourceGroupName ADFTutorialResourceGroup -TemplateFile C:\ADFGetStarted\ADFCopyTutorialARM.json -TemplateParameterFile C:\ADFGetStarted\ADFCopyTutorialARM-Parameters.json
    ```

## <a name="monitor-pipeline"></a>Supervisión de la canalización

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con su cuenta.
2. Haga clic en **Factorías de datos** en el menú de la izquierda (o bien) haga clic en **Todos los servicios** y después en **Factorías de datos** bajo la categoría **INTELIGENCIA Y ANÁLISIS**.
   
    ![Menú Factorías de datos](media/data-factory-copy-activity-tutorial-using-azure-resource-manager-template/data-factories-menu.png)
3. En la página **Factorías de datos**, busque y encuentre su factoría de datos (AzureBlobToAzureSQLDatabaseDF). 
   
    ![Búsqueda de la factoría de datos](media/data-factory-copy-activity-tutorial-using-azure-resource-manager-template/search-for-data-factory.png)  
4. Haga clic en su factoría de datos de Azure. Verá la página principal de la factoría de datos.
   
    ![Página principal de la factoría de datos](media/data-factory-copy-activity-tutorial-using-azure-resource-manager-template/data-factory-home-page.png)  
6. Siga las instrucciones que se indican en [Supervisión de conjuntos de datos y canalizaciones](data-factory-monitor-manage-pipelines.md) para supervisar la canalización y los conjuntos de datos que ha creado en este tutorial. Actualmente, Visual Studio no permite supervisar canalizaciones de Data Factory.
7. Cuando un segmento se encuentre en el estado **Listo**, compruebe que los datos se copian en la tabla **emp** de la base de datos de Azure SQL Database.


Para más información sobre cómo usar las hojas de Azure Portal para supervisar la canalización y los conjuntos de datos creados en este tutorial, consulte [Supervisión de conjuntos de datos y canalizaciones](data-factory-monitor-manage-pipelines.md).

Para más información sobre cómo usar la aplicación Supervisión y administración para supervisar sus canalizaciones de datos, consulte [Supervisión y administración de canalizaciones de Azure Data Factory mediante la aplicación de supervisión y administración](data-factory-monitor-manage-app.md).

## <a name="data-factory-entities-in-the-template"></a>Entidades de Data Factory en la plantilla
### <a name="define-data-factory"></a>Definición de factoría de datos
Puede definir una factoría de datos en la plantilla de Resource Manager como se muestra en el ejemplo siguiente:  

```json
"resources": [
{
    "name": "[variables('dataFactoryName')]",
    "apiVersion": "2015-10-01",
    "type": "Microsoft.DataFactory/datafactories",
    "location": "West US"
}
```

El valor de dataFactoryName se define como: 

```json
"dataFactoryName": "[concat('AzureBlobToAzureSQLDatabaseDF', uniqueString(resourceGroup().id))]"
```

Es una cadena única basada en el identificador del grupo de recursos.  

### <a name="defining-data-factory-entities"></a>Definición de las entidades de Data Factory
Las siguientes entidades de Data Factory se definen en la plantilla JSON: 

1. [Servicio vinculado de Azure Storage](#azure-storage-linked-service)
2. [Servicio vinculado SQL de Azure](#azure-sql-database-linked-service)
3. [Conjunto de datos del blob de Azure](#azure-blob-dataset)
4. [Conjunto de datos de Azure SQL](#azure-sql-dataset)
5. [Canalización de datos con una actividad de copia](#data-pipeline)

#### <a name="azure-storage-linked-service"></a>Servicio vinculado de Azure Storage
AzureStorageLinkedService vincula una cuenta de Azure Storage a la factoría de datos. Como parte de los [requisitos previos](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md), se creó un contenedor y se cargaron datos en esta cuenta de almacenamiento. Especifique el nombre y la clave de la cuenta de almacenamiento de Azure en esta sección. Consulte [Servicio vinculado de Azure Storage](data-factory-azure-blob-connector.md#azure-storage-linked-service) para más información sobre las propiedades JSON usadas para definir un servicio vinculado de Azure Storage. 

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

ConnectionString utiliza los parámetros storageAccountName y storageAccountKey. Los valores de estos parámetros se pasan mediante el uso de un archivo de configuración. La definición también usa las variables azureStroageLinkedService y dataFactoryName definidas en la plantilla. 

#### <a name="azure-sql-database-linked-service"></a>Servicio vinculado de Azure SQL Database
AzureSqlLinkedService vincula la base de datos de Azure SQL Database con la factoría de datos. Los datos que se copian desde Blob Storage se almacenan en esta base de datos. Como parte de los [requisitos previos](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md), se creó la tabla emp en esta base de datos. Especifique el nombre del servidor SQL lógico, el nombre de la base de datos, el nombre de usuario y la contraseña del usuario en esta sección. Consulte [Servicio vinculado de Azure SQL](data-factory-azure-sql-connector.md#linked-service-properties) para obtener información sobre las propiedades JSON usadas para definir un servicio vinculado de Azure SQL.  

```json
{
    "type": "linkedservices",
    "name": "[variables('azureSqlLinkedServiceName')]",
    "dependsOn": [
      "[variables('dataFactoryName')]"
    ],
    "apiVersion": "2015-10-01",
    "properties": {
          "type": "AzureSqlDatabase",
          "description": "Azure SQL linked service",
          "typeProperties": {
            "connectionString": "[concat('Server=tcp:',parameters('sqlServerName'),'.database.windows.net,1433;Database=', parameters('databaseName'), ';User ID=',parameters('sqlServerUserName'),';Password=',parameters('sqlServerPassword'),';Trusted_Connection=False;Encrypt=True;Connection Timeout=30')]"
          }
    }
}
```

ConnectionString usa los parámetros sqlServerName, databaseName, sqlServerUserName y sqlServerPassword cuyos valores se pasan mediante un archivo de configuración. La definición también usa las siguientes variables de la plantilla: azureSqlLinkedServiceName y dataFactoryName.

#### <a name="azure-blob-dataset"></a>Conjunto de datos del blob de Azure
El servicio vinculado Azure Storage especifica la cadena de conexión que el servicio Data Factory utiliza en tiempo de ejecución para conectarse a su cuenta de Azure Storage. En la definición del conjunto de datos de blob de Azure, especifique los nombres del contenedor de blobs, la carpeta y el archivo que contiene los datos de entrada. Consulte las [propiedades del conjunto de datos de Azure Blob](data-factory-azure-blob-connector.md#dataset-properties) para más información sobre las propiedades JSON usadas para definir un conjunto de datos de Azure Blob. 

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
        "structure": [
        {
              "name": "Column0",
              "type": "String"
        },
        {
              "name": "Column1",
              "type": "String"
        }
          ],
          "typeProperties": {
            "folderPath": "[concat(parameters('sourceBlobContainer'), '/')]",
            "fileName": "[parameters('sourceBlobName')]",
            "format": {
                  "type": "TextFormat",
                  "columnDelimiter": ","
            }
          },
          "availability": {
            "frequency": "Hour",
            "interval": 1
          },
          "external": true
    }
}
```

#### <a name="azure-sql-dataset"></a>Conjunto de datos de Azure SQL
Especifique el nombre de la tabla de Azure SQL Database que contiene los datos copiados de Azure Blob Storage. Consulte las [propiedades del conjunto de datos del blob de Azure](data-factory-azure-sql-connector.md#dataset-properties) para más información sobre las propiedades JSON usadas para definir un conjunto de datos de Azure SQL. 

```json
{
    "type": "datasets",
    "name": "[variables('sqlOutputDatasetName')]",
    "dependsOn": [
        "[variables('dataFactoryName')]",
          "[variables('azureSqlLinkedServiceName')]"
    ],
    "apiVersion": "2015-10-01",
    "properties": {
          "type": "AzureSqlTable",
          "linkedServiceName": "[variables('azureSqlLinkedServiceName')]",
          "structure": [
        {
              "name": "FirstName",
              "type": "String"
        },
        {
              "name": "LastName",
              "type": "String"
        }
          ],
          "typeProperties": {
            "tableName": "[parameters('targetSQLTable')]"
          },
          "availability": {
            "frequency": "Hour",
            "interval": 1
          }
    }
}
```

#### <a name="data-pipeline"></a>Canalización de datos
Va a definir una canalización que copia los datos del conjunto de datos de blob de Azure al conjunto de datos de Azure SQL. Consulte [JSON de canalización](data-factory-create-pipelines.md#pipeline-json) para obtener descripciones de elementos JSON que se usan para definir una canalización en este ejemplo. 

```json
{
    "type": "datapipelines",
    "name": "[variables('pipelineName')]",
    "dependsOn": [
        "[variables('dataFactoryName')]",
          "[variables('azureStorageLinkedServiceName')]",
          "[variables('azureSqlLinkedServiceName')]",
          "[variables('blobInputDatasetName')]",
          "[variables('sqlOutputDatasetName')]"
    ],
    "apiVersion": "2015-10-01",
    "properties": {
          "activities": [
        {
              "name": "CopyFromAzureBlobToAzureSQL",
              "description": "Copy data frm Azure blob to Azure SQL",
              "type": "Copy",
              "inputs": [
            {
                  "name": "[variables('blobInputDatasetName')]"
            }
              ],
              "outputs": [
            {
                  "name": "[variables('sqlOutputDatasetName')]"
            }
              ],
              "typeProperties": {
                "source": {
                      "type": "BlobSource"
                },
                "sink": {
                      "type": "SqlSink",
                      "sqlWriterCleanupScript": "$$Text.Format('DELETE FROM {0}', 'emp')"
                },
                "translator": {
                      "type": "TabularTranslator",
                      "columnMappings": "Column0:FirstName,Column1:LastName"
                }
              },
              "Policy": {
                "concurrency": 1,
                "executionPriorityOrder": "NewestFirst",
                "retry": 3,
                "timeout": "01:00:00"
              }
        }
          ],
          "start": "2017-05-11T00:00:00Z",
          "end": "2017-05-12T00:00:00Z"
    }
}
```

## <a name="reuse-the-template"></a>Reutilización de la plantilla
En el tutorial, ha creado una plantilla para definir las entidades de Data Factory y una plantilla para pasar valores de parámetros. La canalización copia los datos desde una cuenta de Azure Storage hasta la base de datos de Azure SQL especificada a través de parámetros. Para usar la misma plantilla para implementar las entidades de Data Factory en distintos entornos, cree un archivo de parámetros para cada entorno y utilícelo al implementarla en ese entorno.     

Ejemplo:  

```PowerShell
New-AzResourceGroupDeployment -Name MyARMDeployment -ResourceGroupName ADFTutorialResourceGroup -TemplateFile ADFCopyTutorialARM.json -TemplateParameterFile ADFCopyTutorialARM-Parameters-Dev.json
```
```PowerShell
New-AzResourceGroupDeployment -Name MyARMDeployment -ResourceGroupName ADFTutorialResourceGroup -TemplateFile ADFCopyTutorialARM.json -TemplateParameterFile ADFCopyTutorialARM-Parameters-Test.json
```
```PowerShell
New-AzResourceGroupDeployment -Name MyARMDeployment -ResourceGroupName ADFTutorialResourceGroup -TemplateFile ADFCopyTutorialARM.json -TemplateParameterFile ADFCopyTutorialARM-Parameters-Production.json
```

Tenga en cuenta que el primer comando usa el archivo de parámetros para el entorno de desarrollo, el segundo para el entorno de prueba y el tercero para el entorno de producción.  

También puede volver a usar la plantilla para llevar a cabo tareas repetidas. Por ejemplo, necesita crear muchas instancias de Data Factory con una o varias canalizaciones que implementen la misma lógica, pero cada instancia de Data Factory usa cuentas de Storage y SQL Database distintas. En este escenario, usa la misma plantilla en el mismo entorno (desarrollo, prueba o producción) con distintos archivos de parámetros para crear factorías de datos.   

## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha usado Azure Blob Storage como almacén de datos de origen y Azure SQL Database como almacén de datos de destino en una operación de copia. En la tabla siguiente se proporciona una lista de almacenes de datos que se admiten como orígenes y destinos de la actividad de copia: 

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

Para aprender a copiar datos hacia y desde un almacén de datos, haga clic en el vínculo del almacén de datos en la tabla.
