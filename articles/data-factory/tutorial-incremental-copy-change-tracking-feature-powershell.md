---
title: Copia incremental de datos con Change Tracking mediante PowerShell
description: En este tutorial, creará una canalización de Azure Data Factory que copia los datos diferenciales de forma incremental de varias tablas de una base de datos de SQL Server a una base de datos de Azure SQL Database.
services: data-factory
ms.author: yexu
author: dearandyxu
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.custom: seo-lt-2019; seo-dt-2019, devx-track-azurepowershell
ms.date: 01/22/2018
ms.openlocfilehash: 6fab092b0467228fd4fcdbcf95f99fd0981fb1e4
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89079235"
---
# <a name="incrementally-load-data-from-azure-sql-database-to-azure-blob-storage-using-change-tracking-information-using-powershell"></a>Carga incremental de datos de Azure SQL Database a Azure Blob Storage mediante la información de control de cambios con PowerShell

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este tutorial, creará una factoría de datos de Azure con una canalización que carga los datos diferenciales según la información de **control de cambios** de la base de datos de origen de Azure SQL Database a una instancia de Azure Blob Storage.  

En este tutorial, realizará los siguientes pasos:

> [!div class="checklist"]
> * Preparación del almacenamiento de datos de origen
> * Creación de una factoría de datos.
> * Cree servicios vinculados.
> * Cree los conjuntos de datos de control de cambios, el origen y el receptor.
> * Creación, ejecución y supervisión de la canalización de copia completa
> * Adición o actualización de datos en la tabla de origen
> * Creación, ejecución y supervisión de la canalización de copia incremental

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="overview"></a>Información general
En una solución de integración de datos, la carga incremental de los datos después de cargas completas iniciales es un método ampliamente usado. En algunos casos, los datos modificados en un período en el almacén de datos de origen pueden ser fácilmente segmentados (por ejemplo, LastModifyTime o CreationTime). En algunos casos, no hay ninguna manera explícita de identificar las diferencias de datos desde la última vez que se procesaron. Para identificar los datos diferenciales, puede usarse la tecnología de control de cambios, admitida por almacenes de datos como Azure SQL Database y SQL Server.  Este tutorial describe cómo usar Azure Data Factory con la tecnología de control de cambios de SQL Server para cargar incrementalmente los datos diferenciales desde Azure SQL Database a Azure Blob Storage.  Para obtener información más concreta sobre la tecnología de control de cambios de SQL, consulte [Acerca del control de cambios (SQL Server)](/sql/relational-databases/track-changes/about-change-tracking-sql-server).

## <a name="end-to-end-workflow"></a>Flujo de trabajo de un extremo a otro
Estos son los pasos del flujo de trabajo completo típico para cargar incrementalmente los datos mediante la tecnología de control de cambios.

> [!NOTE]
> Tanto Azure SQL Database como SQL Server admiten la tecnología de control de cambios. Este tutorial utiliza Azure SQL Database como almacén de datos de origen. También puede usar una instancia de SQL Server.

1. **Carga inicial de datos históricos** (ejecutar una vez):
    1. Habilite la tecnología Change Tracking en la base de datos de origen de Azure SQL Database.
    2. Obtenga el valor inicial de SYS_CHANGE_VERSION en la base de datos como base de referencia para capturar los datos que han cambiado.
    3. Cargue todos los datos de la base de datos de origen en una instancia de Azure Blob Storage.
2. **Carga incremental de los datos diferenciales según una programación** (ejecutar periódicamente después de la carga inicial de datos):
    1. Obtenga los valores SYS_CHANGE_VERSION antiguos y nuevos.
    2. Cargue los datos diferenciales combinando las claves principales de las filas modificadas (entre dos valores SYS_CHANGE_VERSION) desde **sys.change_tracking_tables** con los datos de la **tabla de origen** y, a continuación, muévalos al destino.
    3. Actualice el valor SYS_CHANGE_VERSION para la carga diferencial la próxima vez.

## <a name="high-level-solution"></a>Solución de alto nivel
En este tutorial, creará dos canalizaciones que llevan a cabo las dos operaciones siguientes:  

1. **Carga inicial:** creará una canalización con la actividad de copia que copia todos los datos desde el almacén de datos de origen (Azure SQL Database) al almacén de datos de destino (Azure Blob Storage).

    ![Carga completa de datos](media/tutorial-incremental-copy-change-tracking-feature-powershell/full-load-flow-diagram.png)
1.  **Carga incremental:** creará una canalización con las siguientes actividades y la ejecutará con regularidad.
    1. Cree **dos actividades de búsqueda** para obtener los valores SYS_CHANGE_VERSION antiguo y nuevo desde Azure SQL Database y pasarlos a la actividad de copia.
    2. Cree **una actividad de copia** para copiar los datos insertados, actualizados o eliminados entre los dos valores SYS_CHANGE_VERSION de Azure SQL Database a Azure Blob Storage.
    3. Cree **una actividad de procedimiento almacenado** para actualizar el valor SYS_CHANGE_VERSION para la ejecución de la siguiente canalización.

    ![Diagrama de flujo de incremento de carga](media/tutorial-incremental-copy-change-tracking-feature-powershell/incremental-load-flow-diagram.png)


Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

* Azure PowerShell. Instale los módulos de Azure PowerShell siguiendo las instrucciones de [Cómo instalar y configurar Azure PowerShell](/powershell/azure/install-Az-ps).
* **Azure SQL Database**. La base de datos se usa como almacén de datos de **origen**. Si no tiene ninguna base de datos en Azure SQL Database, consulte el artículo [Creación de una base de datos en Azure SQL Database](../azure-sql/database/single-database-create-quickstart.md) para ver los pasos y crear una.
* **Cuenta de Azure Storage**. Blob Storage se usa como almacén de datos **receptor**. Si no tiene una cuenta de almacenamiento de Azure, consulte el artículo [Crear una cuenta de almacenamiento](../storage/common/storage-account-create.md) para ver los pasos para su creación. Cree un contenedor denominado **adftutorial**. 

### <a name="create-a-data-source-table-in-your-database"></a>Creación de una tabla de origen de datos en la base de datos

1. Abra **SQL Server Management Studio** y conéctese a SQL Database.
2. En el **Explorador de servidores**, haga clic con el botón derecho en la **base de datos** y elija la **Nueva consulta**.
3. Ejecute el siguiente comando SQL en su base de datos para crear una tabla denominada `data_source_table` como almacén de origen de datos.  

    ```sql
    create table data_source_table
    (
        PersonID int NOT NULL,
        Name varchar(255),
        Age int
        PRIMARY KEY (PersonID)
    );

    INSERT INTO data_source_table
        (PersonID, Name, Age)
    VALUES
        (1, 'aaaa', 21),
        (2, 'bbbb', 24),
        (3, 'cccc', 20),
        (4, 'dddd', 26),
        (5, 'eeee', 22);

    ```
4. Habilite el mecanismo de **control de cambios** en la base de datos y la tabla de origen (data_source_table) ejecutando la siguiente consulta SQL:

    > [!NOTE]
    > - Reemplace &lt;el nombre de la base de datos&gt; por el nombre de la base de datos que tiene la tabla data_source_table.
    > - Los datos modificados se mantienen durante dos días en el ejemplo actual. Si carga los datos cambiados para cada tres días o más, no se incluyen algunos que han cambiado.  Tiene que cambiar el valor de CHANGE_RETENTION por un número mayor. También puede asegurarse de que el período para cargar los datos cambiados es dentro de dos días. Para más información, vea [Habilitar el control de cambios para una base de datos](/sql/relational-databases/track-changes/enable-and-disable-change-tracking-sql-server#enable-change-tracking-for-a-database)

    ```sql
    ALTER DATABASE <your database name>
    SET CHANGE_TRACKING = ON  
    (CHANGE_RETENTION = 2 DAYS, AUTO_CLEANUP = ON)  

    ALTER TABLE data_source_table
    ENABLE CHANGE_TRACKING  
    WITH (TRACK_COLUMNS_UPDATED = ON)
    ```
5. Cree una tabla y almacene el valor predeterminado ChangeTracking_version ejecutando la consulta siguiente:

    ```sql
    create table table_store_ChangeTracking_version
    (
        TableName varchar(255),
        SYS_CHANGE_VERSION BIGINT,
    );

    DECLARE @ChangeTracking_version BIGINT
    SET @ChangeTracking_version = CHANGE_TRACKING_CURRENT_VERSION();  

    INSERT INTO table_store_ChangeTracking_version
    VALUES ('data_source_table', @ChangeTracking_version)
    ```

    > [!NOTE]
    > Si los datos no cambian después de haber habilitado el control de cambios para SQL Database, el valor de la versión de control de cambios es 0.
6. Ejecute la siguiente consulta para crear un procedimiento almacenado en su base de datos. La canalización invoca este procedimiento almacenado para actualizar la versión de control de cambios en la tabla que creó en el paso anterior.

    ```sql
    CREATE PROCEDURE Update_ChangeTracking_Version @CurrentTrackingVersion BIGINT, @TableName varchar(50)
    AS

    BEGIN

        UPDATE table_store_ChangeTracking_version
        SET [SYS_CHANGE_VERSION] = @CurrentTrackingVersion
    WHERE [TableName] = @TableName

    END    
    ```

### <a name="azure-powershell"></a>Azure PowerShell
Instale los módulos de Azure PowerShell siguiendo las instrucciones de [Cómo instalar y configurar Azure PowerShell](/powershell/azure/install-Az-ps).

## <a name="create-a-data-factory"></a>Crear una factoría de datos
1. Defina una variable para el nombre del grupo de recursos que usa en los comandos de PowerShell más adelante. Copie el texto del comando siguiente en PowerShell, especifique el nombre del [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) entre comillas dobles y ejecute el comando. Por ejemplo: `"adfrg"`. 
   
     ```powershell
    $resourceGroupName = "ADFTutorialResourceGroup";
    ```

    Si el grupo de recursos ya existe, puede que no desee sobrescribirlo. Asigne otro valor a la variable `$resourceGroupName` y vuelva a ejecutar el comando
2. Defina una variable para la ubicación de la factoría de datos:

    ```powershell
    $location = "East US"
    ```
3. Para crear el grupo de recursos de Azure, ejecute el comando siguiente:

    ```powershell
    New-AzResourceGroup $resourceGroupName $location
    ```
    Si el grupo de recursos ya existe, puede que no desee sobrescribirlo. Asigne otro valor a la variable `$resourceGroupName` y ejecute el comando de nuevo.
3. Defina una variable para el nombre de la factoría de datos.

    > [!IMPORTANT]
    >  Actualice el nombre de la factoría de datos para que sea globalmente único.  

    ```powershell
    $dataFactoryName = "IncCopyChgTrackingDF";
    ```
5. Para crear la factoría de datos, ejecute el siguiente cmdlet, **Set-AzDataFactoryV2**:

    ```powershell       
    Set-AzDataFactoryV2 -ResourceGroupName $resourceGroupName -Location $location -Name $dataFactoryName
    ```

Tenga en cuenta los siguientes puntos:

* El nombre de la instancia de Azure Data Factory debe ser único de forma global. Si recibe el siguiente error, cambie el nombre y vuelva a intentarlo.

    ```
    The specified Data Factory name 'ADFIncCopyChangeTrackingTestFactory' is already in use. Data Factory names must be globally unique.
    ```
* Para crear instancias de Data Factory, la cuenta de usuario que use para iniciar sesión en Azure debe ser un miembro de los roles **colaborador** o **propietario**, o de **administrador** de la suscripción de Azure.
* Para una lista de las regiones de Azure en las que Data Factory está disponible actualmente, seleccione las regiones que le interesen en la página siguiente y expanda **Análisis** para poder encontrar **Data Factory**: [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/). Los almacenes de datos (Azure Storage, Azure SQL Database, etc.) y los procesos (HDInsight, etc.) que usa la factoría de datos pueden encontrarse en otras regiones.


## <a name="create-linked-services"></a>Crear servicios vinculados
Los servicios vinculados se crean en una factoría de datos para vincular los almacenes de datos y los servicios de proceso con la factoría de datos. En esta sección, creará servicios vinculados en su cuenta de Azure Storage y en la base de datos de Azure SQL Database.

### <a name="create-azure-storage-linked-service"></a>Creación de un servicio vinculado de Azure Storage
En este paso, vincula su cuenta de Azure Storage a la factoría de datos.

1. Cree un archivo JSON llamado **AzureStorageLinkedService.json** en la carpeta **C:\ADFTutorials\IncCopyChangeTrackingTutorial** con el siguiente contenido: (cree la carpeta si aún no existe). Reemplace `<accountName>`, `<accountKey>` por el nombre y la clave de su cuenta de Azure Storage antes de guardar el archivo.

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
2. En **Azure PowerShell**, cambie a la carpeta **C:\ADFTutorials\IncCopyChangeTrackingTutorial**.
3. Ejecute el cmdlet **Set-AzDataFactoryV2LinkedService** para crear el servicio vinculado: **AzureStorageLinkedService**. En el ejemplo siguiente, debe pasar los valores de los parámetros **ResourceGroupName** y **DataFactoryName**.

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureStorageLinkedService" -File ".\AzureStorageLinkedService.json"
    ```

    Este es la salida de ejemplo:

    ```json
    LinkedServiceName : AzureStorageLinkedService
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : IncCopyChgTrackingDF
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureStorageLinkedService
    ```

### <a name="create-azure-sql-database-linked-service"></a>Creación de un servicio vinculado de Azure SQL Database
En este paso, vinculará la base de datos a la factoría de datos.

1. Cree un archivo JSON llamado **AzureSQLDatabaseLinkedService.json** en la carpeta **C:\ADFTutorials\IncCopyChangeTrackingTutorial** con el siguiente contenido: Antes de guardar el archivo, reemplace **&lt;server&gt; &lt;database name&gt;, &lt;user id&gt; y &lt;password&gt;** por el nombre del servidor, el nombre de la base de datos, el identificador de usuario y la contraseña.

    ```json
    {
        "name": "AzureSQLDatabaseLinkedService",
        "properties": {
            "type": "AzureSqlDatabase",
            "typeProperties": {
                "connectionString": "Server = tcp:<server>.database.windows.net,1433;Initial Catalog=<database name>; Persist Security Info=False; User ID=<user name>; Password=<password>; MultipleActiveResultSets = False; Encrypt = True; TrustServerCertificate = False; Connection Timeout = 30;"
            }
        }
    }
    ```
2. En **Azure PowerShell**, ejecute el cmdlet **Set-AzDataFactoryV2LinkedService** para crear el servicio vinculado: **AzureSQLDatabaseLinkedService**.

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureSQLDatabaseLinkedService" -File ".\AzureSQLDatabaseLinkedService.json"
    ```

    Este es la salida de ejemplo:

    ```json
    LinkedServiceName : AzureSQLDatabaseLinkedService
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : IncCopyChgTrackingDF
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlDatabaseLinkedService
    ```

## <a name="create-datasets"></a>Creación de conjuntos de datos
En este paso, creará conjuntos de datos para representar el origen de datos, el destino de datos. y el lugar donde almacenar el valor SYS_CHANGE_VERSION.

### <a name="create-a-source-dataset"></a>Creación de un conjunto de datos de origen
En este paso, creará conjuntos de datos para representar el origen de datos.

1. Cree un archivo JSON llamado SourceDataset.json en la misma carpeta con el siguiente contenido:

    ```json
    {
        "name": "SourceDataset",
        "properties": {
            "type": "AzureSqlTable",
            "typeProperties": {
                "tableName": "data_source_table"
            },
            "linkedServiceName": {
                "referenceName": "AzureSQLDatabaseLinkedService",
                "type": "LinkedServiceReference"
            }
        }
    }   
    ```

2.  Ejecute el cmdlet Set-AzDataFactoryV2Dataset para crear el conjunto de datos: SourceDataset

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SourceDataset" -File ".\SourceDataset.json"
    ```

    Esta es la salida de ejemplo del cmdlet:

    ```json
    DatasetName       : SourceDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : IncCopyChgTrackingDF
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlTableDataset
    ```

### <a name="create-a-sink-dataset"></a>Creación de un conjunto de datos receptor
En este paso, creará un conjunto de datos para representar los datos que se copian desde el almacén de datos de origen.

1. Cree un archivo JSON llamado SinkDataset.json en la misma carpeta con el siguiente contenido:

    ```json
    {
        "name": "SinkDataset",
        "properties": {
            "type": "AzureBlob",
            "typeProperties": {
                "folderPath": "adftutorial/incchgtracking",
                "fileName": "@CONCAT('Incremental-', pipeline().RunId, '.txt')",
                "format": {
                    "type": "TextFormat"
                }
            },
            "linkedServiceName": {
                "referenceName": "AzureStorageLinkedService",
                "type": "LinkedServiceReference"
            }
        }
    }
    ```

    Creará el contenedor de adftutorial en Azure Blob Storage como parte de los requisitos previos. Cree el contenedor si no existe (o) asígnele el nombre de uno existente. En este tutorial, el nombre de archivo de salida se genera dinámicamente mediante la expresión: @CONCAT('Incremental-', pipeline().RunId, '.txt').
2.  Ejecute el cmdlet Set-AzDataFactoryV2Dataset para crear el conjunto de datos: SinkDataset

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SinkDataset" -File ".\SinkDataset.json"
    ```

    Esta es la salida de ejemplo del cmdlet:

    ```json
    DatasetName       : SinkDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : IncCopyChgTrackingDF
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureBlobDataset
    ```

### <a name="create-a-change-tracking-dataset"></a>Creación de un conjunto de datos de control de cambios
En este paso, creará un conjunto de datos para almacenar la versión de control de cambios.  

1. Cree un archivo JSON llamado ChangeTrackingDataset.json en la misma carpeta con el siguiente contenido:

    ```json
    {
        "name": " ChangeTrackingDataset",
        "properties": {
            "type": "AzureSqlTable",
            "typeProperties": {
                "tableName": "table_store_ChangeTracking_version"
            },
            "linkedServiceName": {
                "referenceName": "AzureSQLDatabaseLinkedService",
                "type": "LinkedServiceReference"
            }
        }
    }
    ```

    Crea la tabla table_store_ChangeTracking_version como parte de los requisitos previos.
2.  Ejecute el cmdlet Set-AzDataFactoryV2Dataset para crear el conjunto de datos: ChangeTrackingDataset

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "ChangeTrackingDataset" -File ".\ChangeTrackingDataset.json"
    ```

    Esta es la salida de ejemplo del cmdlet:

    ```json
    DatasetName       : ChangeTrackingDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : IncCopyChgTrackingDF
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlTableDataset
    ```

## <a name="create-a-pipeline-for-the-full-copy"></a>Creación de una canalización para la copia completa
En este paso, va a crear una canalización con la actividad de copia que copia todos los datos desde el almacén de datos de origen (Azure SQL Database) al almacén de datos de destino (Azure Blob Storage).

1. Cree un archivo JSON llamado FullCopyPipeline.json en la misma carpeta con el siguiente contenido:

    ```json
    {
        "name": "FullCopyPipeline",
        "properties": {
            "activities": [{
                "name": "FullCopyActivity",
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "SqlSource"
                    },
                    "sink": {
                        "type": "BlobSink"
                    }
                },

                "inputs": [{
                    "referenceName": "SourceDataset",
                    "type": "DatasetReference"
                }],
                "outputs": [{
                    "referenceName": "SinkDataset",
                    "type": "DatasetReference"
                }]
            }]
        }
    }
    ```
2. Ejecute el cmdlet Set-AzDataFactoryV2Pipeline para crear la canalización: FullCopyPipeline.

   ```powershell
    Set-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "FullCopyPipeline" -File ".\FullCopyPipeline.json"
   ```

   Este es la salida de ejemplo:

   ```json
    PipelineName      : FullCopyPipeline
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : IncCopyChgTrackingDF
    Activities        : {FullCopyActivity}
    Parameters        :
   ```

### <a name="run-the-full-copy-pipeline"></a>Ejecución de la canalización de copia completa
Ejecución de la canalización **FullCopyPipeline** mediante el cmdlet **Invoke-AzDataFactoryV2Pipeline**.

```powershell
Invoke-AzDataFactoryV2Pipeline -PipelineName "FullCopyPipeline" -ResourceGroup $resourceGroupName -dataFactoryName $dataFactoryName        
```

### <a name="monitor-the-full-copy-pipeline"></a>Supervisión de la canalización de copia completa

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Haga clic en **Todos los servicios**, busque con la palabra clave `data factories` y seleccione **Factorías de datos**.

    ![Menú Factorías de datos](media/tutorial-incremental-copy-change-tracking-feature-powershell/monitor-data-factories-menu-1.png)
3. Busque **su factoría de datos** en la lista y selecciónela para iniciar la página Factoría de datos.

    ![Búsqueda de la factoría de datos](media/tutorial-incremental-copy-change-tracking-feature-powershell/monitor-search-data-factory-2.png)
4. En la página Factoría de datos, haga clic en el icono **Supervisión y administración**.

    ![Icono Supervisión y administración](media/tutorial-incremental-copy-change-tracking-feature-powershell/monitor-monitor-manage-tile-3.png)    
5. La **aplicación de integración de datos** se inicia en otra pestaña. Puede ver todas las **ejecuciones de canalización** y sus estados. Tenga en cuenta que, en el ejemplo siguiente, el estado de ejecución de la canalización es **Correcto**. Puede comprobar los parámetros pasados a la canalización si hace clic en la columna **Parámetros**. Si se ha producido un error, verá un vínculo en la columna **Error**. Haga clic en el vínculo de la columna **Acciones**.

    ![Ejecuciones de la canalización](media/tutorial-incremental-copy-change-tracking-feature-powershell/monitor-pipeline-runs-4.png)    
6. Al hacer clic en el vínculo de la columna **Acciones**, verá la página siguiente que muestra todas las **ejecuciones de actividad** de la canalización.

    ![Ejecuciones de actividad](media/tutorial-incremental-copy-change-tracking-feature-powershell/monitor-activity-runs-5.png)
7. Para volver a la vista de **ejecuciones de canalización**, haga clic en **Canalizaciones** como se muestra en la imagen.


### <a name="review-the-results"></a>Revisión del resultado
Verá un archivo denominado `incremental-<GUID>.txt` en la carpeta `incchgtracking` del contenedor `adftutorial`.

![Archivo de salida de una copia completa](media/tutorial-incremental-copy-change-tracking-feature-powershell/full-copy-output-file.png)

El archivo debe tener los datos de la base de datos:

```
1,aaaa,21
2,bbbb,24
3,cccc,20
4,dddd,26
5,eeee,22
```

## <a name="add-more-data-to-the-source-table"></a>Adición de más datos a la tabla de origen

Ejecute la siguiente consulta en la base de datos para agregar una fila y actualizarla.

```sql
INSERT INTO data_source_table
(PersonID, Name, Age)
VALUES
(6, 'new','50');


UPDATE data_source_table
SET [Age] = '10', [name]='update' where [PersonID] = 1

```

## <a name="create-a-pipeline-for-the-delta-copy"></a>Creación de una canalización para la copia diferencial
En este paso, creará una canalización con las siguientes actividades y la ejecutará con regularidad. Las **actividades de búsqueda** obtienen los valores SYS_CHANGE_VERSION antiguo y nuevo desde Azure SQL Database y los pasan a la actividad de copia. La **actividad de copia** copia los datos insertados, actualizados o eliminados entre los dos valores SYS_CHANGE_VERSION de Azure SQL Database a Azure Blob Storage. La **actividad de procedimiento almacenado** actualiza el valor SYS_CHANGE_VERSION para la ejecución de la siguiente canalización.

1. Cree un archivo JSON llamado IncrementalCopyPipeline.json en la misma carpeta con el siguiente contenido:

    ```json
    {
            "name": "IncrementalCopyPipeline",
            "properties": {
                "activities": [
                {
                        "name": "LookupLastChangeTrackingVersionActivity",
                        "type": "Lookup",
                        "typeProperties": {
                        "source": {
                            "type": "SqlSource",
                            "sqlReaderQuery": "select * from table_store_ChangeTracking_version"
                            },

                            "dataset": {
                            "referenceName": "ChangeTrackingDataset",
                            "type": "DatasetReference"
                            }
                        }
                    },
                    {
                        "name": "LookupCurrentChangeTrackingVersionActivity",
                        "type": "Lookup",
                        "typeProperties": {
                            "source": {
                                "type": "SqlSource",
                                "sqlReaderQuery": "SELECT CHANGE_TRACKING_CURRENT_VERSION() as CurrentChangeTrackingVersion"
                        },

                            "dataset": {
                            "referenceName": "SourceDataset",
                            "type": "DatasetReference"
                            }
                        }
                    },

                    {
                        "name": "IncrementalCopyActivity",
                        "type": "Copy",
                        "typeProperties": {
                            "source": {
                                "type": "SqlSource",
                                "sqlReaderQuery": "select data_source_table.PersonID,data_source_table.Name,data_source_table.Age, CT.SYS_CHANGE_VERSION, SYS_CHANGE_OPERATION from data_source_table RIGHT OUTER JOIN CHANGETABLE(CHANGES data_source_table, @{activity('LookupLastChangeTrackingVersionActivity').output.firstRow.SYS_CHANGE_VERSION}) as CT on data_source_table.PersonID = CT.PersonID where CT.SYS_CHANGE_VERSION <= @{activity('LookupCurrentChangeTrackingVersionActivity').output.firstRow.CurrentChangeTrackingVersion}"
                            },
                            "sink": {
                                "type": "BlobSink"
                            }
                        },
                        "dependsOn": [
                            {
                                "activity": "LookupLastChangeTrackingVersionActivity",
                                "dependencyConditions": [
                                    "Succeeded"
                                ]
                            },
                            {
                                "activity": "LookupCurrentChangeTrackingVersionActivity",
                                "dependencyConditions": [
                                    "Succeeded"
                                ]
                        }
                        ],

                        "inputs": [
                            {
                            "referenceName": "SourceDataset",
                                "type": "DatasetReference"
                        }
                        ],
                        "outputs": [
                            {
                                "referenceName": "SinkDataset",
                                "type": "DatasetReference"
                            }
                        ]
                    },

                {
                        "name": "StoredProceduretoUpdateChangeTrackingActivity",
                        "type": "SqlServerStoredProcedure",
                        "typeProperties": {

                            "storedProcedureName": "Update_ChangeTracking_Version",
                            "storedProcedureParameters": {
                            "CurrentTrackingVersion": {"value": "@{activity('LookupCurrentChangeTrackingVersionActivity').output.firstRow.CurrentChangeTrackingVersion}", "type": "INT64" },
                                "TableName":  { "value":"@{activity('LookupLastChangeTrackingVersionActivity').output.firstRow.TableName}", "type":"String"}
                            }
                    },

                        "linkedServiceName": {
                        "referenceName": "AzureSQLDatabaseLinkedService",
                            "type": "LinkedServiceReference"
                        },

                        "dependsOn": [
                        {
                                "activity": "IncrementalCopyActivity",
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
2. Ejecute el cmdlet Set-AzDataFactoryV2Pipeline para crear la canalización: FullCopyPipeline.

   ```powershell
    Set-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "IncrementalCopyPipeline" -File ".\IncrementalCopyPipeline.json"
   ```

   Este es la salida de ejemplo:

   ```json
    PipelineName      : IncrementalCopyPipeline
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : IncCopyChgTrackingDF
    Activities        : {LookupLastChangeTrackingVersionActivity, LookupCurrentChangeTrackingVersionActivity, IncrementalCopyActivity, StoredProceduretoUpdateChangeTrackingActivity}
    Parameters        :
   ```

### <a name="run-the-incremental-copy-pipeline"></a>Ejecución de la canalización de la copia incremental
Ejecución de la canalización **IncrementalCopyPipeline** mediante el cmdlet **Invoke-AzDataFactoryV2Pipeline**.

```powershell
Invoke-AzDataFactoryV2Pipeline -PipelineName "IncrementalCopyPipeline" -ResourceGroup $resourceGroupName -dataFactoryName $dataFactoryName     
```


### <a name="monitor-the-incremental-copy-pipeline"></a>Supervisión de la canalización de la copia incremental
1. En la **aplicación de integración de datos**, actualice la vista de **ejecuciones de canalización**. Confirme que ve IncrementalCopyPipeline en la lista. Haga clic en el vínculo de la columna **Acciones**.  

    ![Ejecuciones de la canalización](media/tutorial-incremental-copy-change-tracking-feature-powershell/monitor-pipeline-runs-6.png)    
2. Al hacer clic en el vínculo de la columna **Acciones**, verá la página siguiente que muestra todas las **ejecuciones de actividad** de la canalización.

    ![Ejecuciones de actividad](media/tutorial-incremental-copy-change-tracking-feature-powershell/monitor-activity-runs-7.png)
3. Para volver a la vista de **ejecuciones de canalización**, haga clic en **Canalizaciones** como se muestra en la imagen.

### <a name="review-the-results"></a>Revisión del resultado
Verá el segundo archivo `incchgtracking` en la carpeta `adftutorial` del contenedor.

![Archivo de salida de la copia incremental](media/tutorial-incremental-copy-change-tracking-feature-powershell/incremental-copy-output-file.png)

El archivo debe tener solo los datos diferenciales de la base de datos. El registro con `U` es la fila actualizada en la base de datos y `I` es la fila que se agrega.

```
1,update,10,2,U
6,new,50,1,I
```
Las tres primeras columnas son datos que han cambiado de data_source_table. Las dos últimas columnas son los metadatos de la tabla del sistema de control de cambios. La cuarta columna es el valor SYS_CHANGE_VERSION de cada fila modificada. La quinta columna es la operación:  U = actualización, I = inserción.  Para obtener más información acerca de la información de control de cambios, consulte [CHANGETABLE](/sql/relational-databases/system-functions/changetable-transact-sql).

```
==================================================================
PersonID Name    Age    SYS_CHANGE_VERSION    SYS_CHANGE_OPERATION
==================================================================
1        update  10     2                     U
6        new     50     1                     I
```


## <a name="next-steps"></a>Pasos siguientes
Pase al tutorial siguiente para obtener información acerca de cómo copiar archivos nuevos y modificados solo según el valor de LastModifiedDate:

> [!div class="nextstepaction"]
>[Copia de archivos nuevos por LastModifiedDate](tutorial-incremental-copy-lastmodified-copy-data-tool.md)
