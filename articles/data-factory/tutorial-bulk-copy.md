---
title: Copia masiva de datos con PowerShell
description: Utilice Azure Data Factory con la actividad de copia para copiar datos desde un almacén de datos de origen a un almacén de datos de destino de forma masiva.
services: data-factory
author: linda33wj
ms.author: jingwang
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.custom: seo-lt-2019
ms.date: 01/22/2018
ms.openlocfilehash: 4a9aaca8128570af74370213e9848e26dec25156
ms.sourcegitcommit: de2750163a601aae0c28506ba32be067e0068c0c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89490265"
---
# <a name="copy-multiple-tables-in-bulk-by-using-azure-data-factory-using-powershell"></a>Copia masiva de varias tablas mediante Azure Data Factory con PowerShell

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este tutorial se muestra cómo puede **copiar varias tablas de Azure SQL Database a Azure Synapse Analytics (anteriormente, SQL Data Warehouse)** . Además, puede aplicar el mismo patrón en otros escenarios de copia. Por ejemplo, para copiar tablas de SQL Server u Oracle a Azure SQL Database, Data Warehouse o el blob de Azure, o bien para copiar diferentes rutas de acceso de blob a tablas de Azure SQL Database.

A grandes rasgos, este tutorial incluye los pasos siguientes:

> [!div class="checklist"]
> * Creación de una factoría de datos.
> * Creación de los servicios vinculados de Azure SQL Database, Azure Synapse Analytics y Azure Storage.
> * Creación de los conjuntos de datos de Azure SQL Database y Azure Synapse Analytics.
> * Creación de una canalización para buscar las tablas que se deben copiar y otra canalización para realizar la operación de copia real. 
> * Inicio de la ejecución de una canalización.
> * Supervisión de las ejecuciones de canalización y actividad.

En este tutorial se usa Azure PowerShell. Para obtener información sobre el uso de otras herramientas o SDK para crear una factoría de datos, consulte los [inicios rápidos](quickstart-create-data-factory-dot-net.md). 

## <a name="end-to-end-workflow"></a>Flujo de trabajo de un extremo a otro
En este escenario, tenemos varias tablas en Azure SQL Database que queremos copiar en Azure Synapse Analytics. Esta es la secuencia lógica de pasos del flujo de trabajo que se realiza en las canalizaciones:

![Flujo de trabajo](media/tutorial-bulk-copy/tutorial-copy-multiple-tables.png)

* La primera canalización busca la lista de tablas que debe copiarse en los almacenes de datos del receptor.  También puede mantener una tabla de metadatos que muestre todas las tablas que se deben copiar en el almacén de datos receptor. A continuación, la canalización desencadena otra canalización, que itera en todas las tablas de la base de datos y realiza la operación de copia de datos.
* La segunda canalización realiza la copia real. Toma la lista de tablas como un parámetro. Para cada tabla de la lista, copie la tabla específica de Azure SQL Database a la tabla correspondiente de Azure Synapse Analytics con la [copia almacenada provisionalmente mediante Blob Storage y PolyBase](connector-azure-sql-data-warehouse.md#use-polybase-to-load-data-into-azure-synapse-analytics) a fin de obtener el mejor rendimiento. En este ejemplo, la primera canalización pasa la lista de tablas como un valor para el parámetro. 

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

* **Azure PowerShell**. Siga las instrucciones de [Instalación y configuración de Azure PowerShell](/powershell/azure/install-Az-ps).
* **Cuenta de Azure Storage**. La cuenta de Azure Storage se usa como almacenamiento de blobs de almacenamiento provisional en la operación de copia masiva. 
* **Azure SQL Database**. Esta base de datos contiene los datos de origen. 
* **Azure Synapse Analytics**. Este almacén de datos contiene los datos que se copian de SQL Database. 

### <a name="prepare-sql-database-and-azure-synapse-analytics"></a>Preparación de SQL Database y Azure Synapse Analytics

**Preparación de la base de datos de Azure SQL de origen**:

Cree una base de datos con los datos de ejemplo de Adventure Works LT en SQL Database siguiendo el artículo [Creación de una base de datos en Azure SQL Database](../azure-sql/database/single-database-create-quickstart.md). En este tutorial se copian todas las tablas de esta base de datos de ejemplo a Azure Synapse Analytics.

**Preparación de la instancia receptora de Azure Synapse Analytics**:

1. Si no tiene un área de trabajo de Azure Synapse Analytics, consulte el artículo [Introducción a Azure Synapse Analytics](..\synapse-analytics\get-started.md) para conocer los pasos para crear una.

2. Cree los esquemas de tabla correspondientes en Azure Synapse Analytics. Debe usar Azure Data Factory para migrar o copiar datos en un paso posterior.

## <a name="azure-services-to-access-sql-server"></a>Servicios de Azure para acceder a SQL Server

Para SQL Database y Azure Synapse Analytics, permita que los servicios de Azure accedan a SQL Server. Asegúrese de que la opción **Permitir el acceso a servicios de Azure** esté **activada** para el servidor. Esta configuración permite al servicio Data Factory leer los datos de Azure SQL Database y escribir los datos en Azure Synapse Analytics. Para comprobar y activar esta configuración, realice los siguientes pasos:

1. Haga clic en **Todos los servicios** a la izquierda y en **Servidores SQL**.
2. Seleccione el servidor y haga clic en **Firewall** en **CONFIGURACIÓN**.
3. En la hoja **Configuración de firewall**, haga clic en **Activar** para **Permitir el acceso a los servicios de Azure**.

## <a name="create-a-data-factory"></a>Crear una factoría de datos

1. Inicie **PowerShell**. Mantenga Azure PowerShell abierto hasta el final de este tutorial. Si lo cierra y vuelve a abrirlo, deberá ejecutar los comandos de nuevo.

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
2. Ejecute el cmdlet **Set-AzDataFactoryV2** para crear una factoría de datos. Reemplace los marcadores de posición con sus propios valores antes de ejecutar el comando. 

    ```powershell
    $resourceGroupName = "<your resource group to create the factory>"
    $dataFactoryName = "<specify the name of data factory to create. It must be globally unique.>"
    Set-AzDataFactoryV2 -ResourceGroupName $resourceGroupName -Location "East US" -Name $dataFactoryName
    ```

    Tenga en cuenta los siguientes puntos:

    * El nombre de la instancia de Azure Data Factory debe ser único de forma global. Si recibe el siguiente error, cambie el nombre y vuelva a intentarlo.

        ```
        The specified Data Factory name 'ADFv2QuickStartDataFactory' is already in use. Data Factory names must be globally unique.
        ```

    * Para crear instancias de Data Factory, debe ser administrador o colaborador de la suscripción de Azure.
    * Para una lista de las regiones de Azure en las que Data Factory está disponible actualmente, seleccione las regiones que le interesen en la página siguiente y expanda **Análisis** para poder encontrar **Data Factory**: [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/). Los almacenes de datos (Azure Storage, Azure SQL Database, etc.) y los procesos (HDInsight, etc.) que usa la factoría de datos pueden encontrarse en otras regiones.

## <a name="create-linked-services"></a>Crear servicios vinculados

En este tutorial, creará tres servicios vinculados para el origen, el receptor, y el blob de almacenamiento provisional respectivamente, lo que incluye conexiones con sus almacenes de datos:

### <a name="create-the-source-azure-sql-database-linked-service"></a>Creación del servicio vinculado de Azure SQL Database de origen

1. Cree un archivo JSON con el nombre **AzureSqlDatabaseLinkedService.json** en la carpeta **C:\ADFv2TutorialBulkCopy** con el siguiente contenido: Si todavía no existe, cree la carpeta ADFv2TutorialBulkCopy.

    > [!IMPORTANT]
    > Reemplace &lt;servername&gt;, &lt;databasename&gt;, &lt;username&gt;@&lt;servername&gt; y &lt;password&gt; por los nombres de su base de datos de Azure SQL antes de guardar el archivo.

    ```json
    {
        "name": "AzureSqlDatabaseLinkedService",
        "properties": {
            "type": "AzureSqlDatabase",
            "typeProperties": {
                "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
            }
        }
    }
    ```

2. En **Azure PowerShell**, cambie a la carpeta **ADFv2TutorialBulkCopy**.

3. Ejecute el cmdlet **Set-AzDataFactoryV2LinkedService** para crear el servicio vinculado: **AzureSqlDatabaseLinkedService**. 

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureSqlDatabaseLinkedService" -File ".\AzureSqlDatabaseLinkedService.json"
    ```

    Este es la salida de ejemplo:

    ```json
    LinkedServiceName : AzureSqlDatabaseLinkedService
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlDatabaseLinkedService
    ```

### <a name="create-the-sink-azure-synapse-analytics-linked-service"></a>Creación del servicio vinculado receptor de Azure Synapse Analytics

1. Cree un archivo JSON con el nombre **AzureSqlDWLinkedService.json** en la carpeta **C:\ADFv2TutorialBulkCopy** con el siguiente contenido:

    > [!IMPORTANT]
    > Reemplace &lt;servername&gt;, &lt;databasename&gt;, &lt;username&gt;@&lt;servername&gt; y &lt;password&gt; por los nombres de su base de datos de Azure SQL antes de guardar el archivo.

    ```json
    {
        "name": "AzureSqlDWLinkedService",
        "properties": {
            "type": "AzureSqlDW",
            "typeProperties": {
                "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
            }
        }
    }
    ```

2. Para crear el servicio vinculado: **AzureSqlDWLinkedService**, ejecute el cmdlet **Set-AzDataFactoryV2LinkedService**.

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureSqlDWLinkedService" -File ".\AzureSqlDWLinkedService.json"
    ```

    Este es la salida de ejemplo:

    ```json
    LinkedServiceName : AzureSqlDWLinkedService
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlDWLinkedService
    ```

### <a name="create-the-staging-azure-storage-linked-service"></a>Creación del servicio vinculado de Azure Storage de almacenamiento provisional

En este tutorial, debe usar Azure Blob Storage como un área de almacenamiento provisional para habilitar PolyBase a fin de mejorar el rendimiento de copia.

1. Cree un archivo JSON con el nombre **AzureStorageLinkedService.json** en la carpeta **C:\ADFv2TutorialBulkCopy** con el siguiente contenido:

    > [!IMPORTANT]
    > Reemplace &lt;accountName&gt; y &lt;accountKey&gt; por el nombre y la clave de su cuenta de Azure Storage antes de guardar el archivo.

    ```json
    {
        "name": "AzureStorageLinkedService",
        "properties": {
            "type": "AzureStorage",
            "typeProperties": {
                "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountName>;AccountKey=<accountKey>"
            }
        }
    }
    ```

2. Para crear el servicio vinculado: **AzureStorageLinkedService**, ejecute el cmdlet **Set-AzDataFactoryV2LinkedService**.

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureStorageLinkedService" -File ".\AzureStorageLinkedService.json"
    ```

    Este es la salida de ejemplo:

    ```json
    LinkedServiceName : AzureStorageLinkedService
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureStorageLinkedService
    ```

## <a name="create-datasets"></a>Creación de conjuntos de datos

En este tutorial, creará los conjuntos de datos de origen y recepción que especifican la ubicación donde se almacenan los datos:

### <a name="create-a-dataset-for-source-sql-database"></a>Creación de un conjunto de datos de la instancia de SQL Database de origen

1. Cree un archivo JSON con el nombre **AzureSqlDatabaseDataset.json** en la carpeta **C:\ADFv2TutorialBulkCopy** con el siguiente contenido. El valor de "tableName" es ficticio dado que más tarde debe usar la consulta SQL en la actividad de copia para recuperar los datos.

    ```json
    {
        "name": "AzureSqlDatabaseDataset",
        "properties": {
            "type": "AzureSqlTable",
            "linkedServiceName": {
                "referenceName": "AzureSqlDatabaseLinkedService",
                "type": "LinkedServiceReference"
            },
            "typeProperties": {
                "tableName": "dummy"
            }
        }
    }
    ```

2. Para crear el conjunto de datos: **AzureSqlDatabaseDataset**, ejecute el cmdlet **Set-AzDataFactoryV2Dataset**.

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureSqlDatabaseDataset" -File ".\AzureSqlDatabaseDataset.json"
    ```

    Este es la salida de ejemplo:

    ```json
    DatasetName       : AzureSqlDatabaseDataset
    ResourceGroupName : <resourceGroupname>
    DataFactoryName   : <dataFactoryName>
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlTableDataset
    ```

### <a name="create-a-dataset-for-sink-synapse-analytics"></a>Creación de un conjunto de datos para la instancia receptora de Synapse Analytics

1. Cree un archivo JSON con el nombre **AzureSqlDWDataset.json** en la carpeta **C:\ADFv2TutorialBulkCopy** con el siguiente contenido: El valor de "tableName" se establece como un parámetro, más adelante la actividad de copia que hace referencia a este conjunto de datos pasa el valor real en el conjunto de datos.

    ```json
    {
        "name": "AzureSqlDWDataset",
        "properties": {
            "type": "AzureSqlDWTable",
            "linkedServiceName": {
                "referenceName": "AzureSqlDWLinkedService",
                "type": "LinkedServiceReference"
            },
            "typeProperties": {
                "tableName": {
                    "value": "@{dataset().DWTableName}",
                    "type": "Expression"
                }
            },
            "parameters":{
                "DWTableName":{
                    "type":"String"
                }
            }
        }
    }
    ```

2. Para crear el conjunto de datos: **AzureSqlDWDataset**, ejecute el cmdlet **Set-AzDataFactoryV2Dataset**.

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureSqlDWDataset" -File ".\AzureSqlDWDataset.json"
    ```

    Este es la salida de ejemplo:

    ```json
    DatasetName       : AzureSqlDWDataset
    ResourceGroupName : <resourceGroupname>
    DataFactoryName   : <dataFactoryName>
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlDwTableDataset
    ```

## <a name="create-pipelines"></a>Creación de canalizaciones

En este tutorial, creará dos canalizaciones:

### <a name="create-the-pipeline-iterateandcopysqltables"></a>Creación de la canalización "IterateAndCopySQLTables"

Esta canalización toma la lista de tablas como un parámetro. Para cada tabla de la lista, copia los datos de la tabla de Azure SQL Database a Azure Synapse Analytics mediante la copia almacenada provisionalmente y PolyBase.

1. Cree un archivo JSON con el nombre **IterateAndCopySQLTables.json** en la carpeta **C:\ADFv2TutorialBulkCopy** con el siguiente contenido:

    ```json
    {
        "name": "IterateAndCopySQLTables",
        "properties": {
            "activities": [
                {
                    "name": "IterateSQLTables",
                    "type": "ForEach",
                    "typeProperties": {
                        "isSequential": "false",
                        "items": {
                            "value": "@pipeline().parameters.tableList",
                            "type": "Expression"
                        },
                        "activities": [
                            {
                                "name": "CopyData",
                                "description": "Copy data from Azure SQL Database to Azure Synapse Analytics",
                                "type": "Copy",
                                "inputs": [
                                    {
                                        "referenceName": "AzureSqlDatabaseDataset",
                                        "type": "DatasetReference"
                                    }
                                ],
                                "outputs": [
                                    {
                                        "referenceName": "AzureSqlDWDataset",
                                        "type": "DatasetReference",
                                        "parameters": {
                                            "DWTableName": "[@{item().TABLE_SCHEMA}].[@{item().TABLE_NAME}]"
                                        }
                                    }
                                ],
                                "typeProperties": {
                                    "source": {
                                        "type": "SqlSource",
                                        "sqlReaderQuery": "SELECT * FROM [@{item().TABLE_SCHEMA}].[@{item().TABLE_NAME}]"
                                    },
                                    "sink": {
                                        "type": "SqlDWSink",
                                        "preCopyScript": "TRUNCATE TABLE [@{item().TABLE_SCHEMA}].[@{item().TABLE_NAME}]",
                                        "allowPolyBase": true
                                    },
                                    "enableStaging": true,
                                    "stagingSettings": {
                                        "linkedServiceName": {
                                            "referenceName": "AzureStorageLinkedService",
                                            "type": "LinkedServiceReference"
                                        }
                                    }
                                }
                            }
                        ]
                    }
                }
            ],
            "parameters": {
                "tableList": {
                    "type": "Object"
                }
            }
        }
    }
    ```

2. Para crear la canalización: **IterateAndCopySQLTables**, ejecute el cmdlet **Set-AzDataFactoryV2Pipeline**.

    ```powershell
    Set-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "IterateAndCopySQLTables" -File ".\IterateAndCopySQLTables.json"
    ```

    Este es la salida de ejemplo:

    ```json
    PipelineName      : IterateAndCopySQLTables
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Activities        : {IterateSQLTables}
    Parameters        : {[tableList, Microsoft.Azure.Management.DataFactory.Models.ParameterSpecification]}
    ```

### <a name="create-the-pipeline-gettablelistandtriggercopydata"></a>Creación de la canalización "GetTableListAndTriggerCopyData"

Esta canalización lleva a cabo dos pasos:

* Busca la tabla del sistema de Azure SQL Database para obtener la lista de tablas que se copiará.
* Desencadena la canalización "IterateAndCopySQLTables" para realizar la copia de datos real.

1. Cree un archivo JSON con el nombre **GetTableListAndTriggerCopyData.json** en la carpeta **C:\ADFv2TutorialBulkCopy** con el siguiente contenido:

    ```json
    {
        "name":"GetTableListAndTriggerCopyData",
        "properties":{
            "activities":[
                { 
                    "name": "LookupTableList",
                    "description": "Retrieve the table list from Azure SQL dataabse",
                    "type": "Lookup",
                    "typeProperties": {
                        "source": {
                            "type": "SqlSource",
                            "sqlReaderQuery": "SELECT TABLE_SCHEMA, TABLE_NAME FROM information_schema.TABLES WHERE TABLE_TYPE = 'BASE TABLE' and TABLE_SCHEMA = 'SalesLT' and TABLE_NAME <> 'ProductModel'"
                        },
                        "dataset": {
                            "referenceName": "AzureSqlDatabaseDataset",
                            "type": "DatasetReference"
                        },
                        "firstRowOnly": false
                    }
                },
                {
                    "name": "TriggerCopy",
                    "type": "ExecutePipeline",
                    "typeProperties": {
                        "parameters": {
                            "tableList": {
                                "value": "@activity('LookupTableList').output.value",
                                "type": "Expression"
                            }
                        },
                        "pipeline": {
                            "referenceName": "IterateAndCopySQLTables",
                            "type": "PipelineReference"
                        },
                        "waitOnCompletion": true
                    },
                    "dependsOn": [
                        {
                            "activity": "LookupTableList",
                            "dependencyConditions": [
                                "Succeeded"
                            ]
                        }
                    ]
                }
            ]
        }
    }
    ```

2. Para crear la canalización: **GetTableListAndTriggerCopyData**, ejecute el cmdlet **Set-AzDataFactoryV2Pipeline**.

    ```powershell
    Set-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "GetTableListAndTriggerCopyData" -File ".\GetTableListAndTriggerCopyData.json"
    ```

    Este es la salida de ejemplo:

    ```json
    PipelineName      : GetTableListAndTriggerCopyData
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Activities        : {LookupTableList, TriggerCopy}
    Parameters        :
    ```

## <a name="start-and-monitor-pipeline-run"></a>Inicio y supervisión de la ejecución de la canalización

1. Inicie una ejecución para la canalización principal "GetTableListAndTriggerCopyData" y capture el identificador de ejecución de la canalización a fin de poder realizar una supervisión en el futuro. De forma subyacente, desencadena la ejecución de la canalización "IterateAndCopySQLTables" tal como se especifica en la actividad ExecutePipeline.

    ```powershell
    $runId = Invoke-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineName 'GetTableListAndTriggerCopyData'
    ```

2.  Ejecute el siguiente script para comprobar continuamente el estado de ejecución de la canalización **GetTableListAndTriggerCopyData** e imprima la ejecución de la canalización final y el resultado de la ejecución de actividad.

    ```powershell
    while ($True) {
        $run = Get-AzDataFactoryV2PipelineRun -ResourceGroupName $resourceGroupName -DataFactoryName $DataFactoryName -PipelineRunId $runId

        if ($run) {
            if ($run.Status -ne 'InProgress') {
                Write-Host "Pipeline run finished. The status is: " $run.Status -foregroundcolor "Yellow"
                Write-Host "Pipeline run details:" -foregroundcolor "Yellow"
                $run
                break
            }
            Write-Host  "Pipeline is running...status: InProgress" -foregroundcolor "Yellow"
        }

        Start-Sleep -Seconds 15
    }

    $result = Get-AzDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId $runId -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)
    Write-Host "Activity run details:" -foregroundcolor "Yellow"
    $result
    ```

    Este es el resultado de la ejecución de ejemplo:

    ```json
    Pipeline run details:
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    RunId             : 0000000000-00000-0000-0000-000000000000
    PipelineName      : GetTableListAndTriggerCopyData
    LastUpdated       : 9/18/2017 4:08:15 PM
    Parameters        : {}
    RunStart          : 9/18/2017 4:06:44 PM
    RunEnd            : 9/18/2017 4:08:15 PM
    DurationInMs      : 90637
    Status            : Succeeded
    Message           : 

    Activity run details:
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    ActivityName      : LookupTableList
    PipelineRunId     : 0000000000-00000-0000-0000-000000000000
    PipelineName      : GetTableListAndTriggerCopyData
    Input             : {source, dataset, firstRowOnly}
    Output            : {count, value, effectiveIntegrationRuntime}
    LinkedServiceName : 
    ActivityRunStart  : 9/18/2017 4:06:46 PM
    ActivityRunEnd    : 9/18/2017 4:07:09 PM
    DurationInMs      : 22995
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}

    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    ActivityName      : TriggerCopy
    PipelineRunId     : 0000000000-00000-0000-0000-000000000000
    PipelineName      : GetTableListAndTriggerCopyData
    Input             : {pipeline, parameters, waitOnCompletion}
    Output            : {pipelineRunId}
    LinkedServiceName : 
    ActivityRunStart  : 9/18/2017 4:07:11 PM
    ActivityRunEnd    : 9/18/2017 4:08:14 PM
    DurationInMs      : 62581
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}
    ```

3. Puede obtener el identificador de ejecución de la canalización "**IterateAndCopySQLTables**" y comprobar el resultado de la ejecución de actividad detallado como se muestra a continuación.

    ```powershell
    Write-Host "Pipeline 'IterateAndCopySQLTables' run result:" -foregroundcolor "Yellow"
    ($result | Where-Object {$_.ActivityName -eq "TriggerCopy"}).Output.ToString()
    ```

    Este es el resultado de la ejecución de ejemplo:

    ```json
    {
        "pipelineRunId": "7514d165-14bf-41fb-b5fb-789bea6c9e58"
    }
    ```

    ```powershell
    $result2 = Get-AzDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId <copy above run ID> -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)
    $result2
    ```

3. Conéctese a la instancia receptora de Azure Synapse Analytics y confirme que los datos se han copiado correctamente desde Azure SQL Database.

## <a name="next-steps"></a>Pasos siguientes
En este tutorial, realizó los pasos siguientes: 

> [!div class="checklist"]
> * Creación de una factoría de datos.
> * Creación de los servicios vinculados de Azure SQL Database, Azure Synapse Analytics y Azure Storage.
> * Creación de los conjuntos de datos de Azure SQL Database y Azure Synapse Analytics.
> * Creación de una canalización para buscar las tablas que se deben copiar y otra canalización para realizar la operación de copia real. 
> * Inicio de la ejecución de una canalización.
> * Supervisión de las ejecuciones de canalización y actividad.

Vaya al tutorial siguiente para obtener información sobre cómo copiar datos de forma incremental de un origen a un destino:
> [!div class="nextstepaction"]
>[Copia de datos de forma incremental](tutorial-incremental-copy-powershell.md)
