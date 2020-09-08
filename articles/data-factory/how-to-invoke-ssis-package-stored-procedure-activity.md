---
title: 'Ejecución de paquetes de SSIS con una actividad de procedimiento almacenado: Azure'
description: En este artículo se describe cómo ejecutar un paquete de SQL Server Integration Services (SSIS) desde una canalización de Azure Data Factory mediante la actividad de procedimiento almacenado.
services: data-factory
documentationcenter: ''
author: swinarko
manager: anandsub
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: ''
ms.devlang: powershell
ms.topic: conceptual
ms.date: 07/09/2020
ms.author: sawinark
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 05ec19fd2351b3a9ac1f383ce4747404eeead936
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89067831"
---
# <a name="run-an-ssis-package-with-the-stored-procedure-activity-in-azure-data-factory"></a>Ejecución de un paquete de SSIS mediante una actividad de procedimiento almacenado de Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este artículo se describe cómo ejecutar un paquete de SSIS desde una canalización de Azure Data Factory mediante una actividad de procedimiento almacenado. 

## <a name="prerequisites"></a>Requisitos previos

### <a name="azure-sql-database"></a>Azure SQL Database 
En el tutorial de este artículo se usa Azure SQL Database para hospedar el catálogo de SSIS. También puede usar Instancia administrada de Azure SQL.

## <a name="create-an-azure-ssis-integration-runtime"></a>Creación de un entorno de ejecución de integración de SSIS para Azure
Cree una instancia de Integration Runtime de SSIS de Azure si no tiene ninguna. Para ello, siga las instrucciones paso a paso del [tutorial: Implementación de paquetes de SSIS](tutorial-create-azure-ssis-runtime-portal.md).

## <a name="data-factory-ui-azure-portal"></a>Interfaz de usuario de Data Factory (Azure Portal)
En esta sección, usará la interfaz de usuario de Data Factory para crear una canalización de Data Factory con una actividad de procedimiento almacenado que invoca un paquete SSIS.

### <a name="create-a-data-factory"></a>Crear una factoría de datos
El primer paso es crear una factoría de datos con Azure Portal. 

1. Inicie el explorador web **Microsoft Edge** o **Google Chrome**. Actualmente, la interfaz de usuario de Data Factory solo se admite en los exploradores web Microsoft Edge y Google Chrome.
2. Acceda a [Azure Portal](https://portal.azure.com). 
3. En el menú de la izquierda, haga clic en **Nuevo**, **Datos y análisis** y **Factoría de datos**. 
   
   ![New->DataFactory](./media/how-to-invoke-ssis-package-stored-procedure-activity/new-azure-data-factory-menu.png)
2. En la página **New data factory** (Nueva factoría de datos), escriba **ADFTutorialDataFactory** en **Name** (Nombre). 
      
     ![Página New data factory (Nueva factoría de datos)](./media/how-to-invoke-ssis-package-stored-procedure-activity/new-azure-data-factory.png)
 
   El nombre de la instancia de Azure Data Factory debe ser **único de forma global**. Si ve el siguiente error en el campo del nombre, cambie el nombre de la factoría de datos (por ejemplo, yournameADFTutorialDataFactory). Consulte el artículo [Azure Data Factory: reglas de nomenclatura](naming-rules.md) para conocer las reglas de nomenclatura de los artefactos de Data Factory.
  
     ![Error de nombre no disponible](./media/how-to-invoke-ssis-package-stored-procedure-activity/name-not-available-error.png)
3. Seleccione la **suscripción** de Azure donde desea crear la factoría de datos. 
4. Para el **grupo de recursos**, realice uno de los siguientes pasos:
     
   - Seleccione en primer lugar **Usar existente**y después un grupo de recursos de la lista desplegable. 
   - Seleccione **Crear nuevo**y escriba el nombre de un grupo de recursos.   
         
     Para obtener más información sobre los grupos de recursos, consulte [Uso de grupos de recursos para administrar los recursos de Azure](../azure-resource-manager/management/overview.md).  
4. Seleccione **V2** para la **versión**.
5. Seleccione la **ubicación** de Data Factory. En la lista desplegable solo se muestran las ubicaciones que admite Data Factory. Los almacenes de datos (Azure Storage, Azure SQL Database, etc.) y los procesos (HDInsight, etc.) utilizados por la factoría de datos pueden encontrarse en otras ubicaciones.
6. Seleccione **Anclar al panel**.     
7. Haga clic en **Crear**.
8. En el panel, verá el icono siguiente con el estado: **Deploying data factory** (Implementación de la factoría de datos). 

     ![icono implementando factoría de datos](media//how-to-invoke-ssis-package-stored-procedure-activity/deploying-data-factory.png)
9. Una vez completada la creación, verá la página **Data Factory** tal como se muestra en la imagen.
   
     ![Página principal Factoría de datos](./media/how-to-invoke-ssis-package-stored-procedure-activity/data-factory-home-page.png)
10. Haga clic en el icono **Author & Monitor** (Creación y supervisión) para iniciar la aplicación de interfaz de usuario de Azure Data Factory en una pestaña independiente. 

### <a name="create-a-pipeline-with-stored-procedure-activity"></a>Crear una canalización con una actividad de procedimiento almacenado
En este paso, usa la interfaz de Data Factory para crear una canalización. Agrega una actividad de procedimiento almacenado a la canalización y lo configura para ejecutar el paquete SSIS mediante el uso del procedimiento almacenado sp_executesql. 

1. En la página de introducción, haga clic en **Create pipeline** (Crear canalización): 

    ![Página de introducción](./media/how-to-invoke-ssis-package-stored-procedure-activity/get-started-page.png)
2. En el cuadro de herramientas **Activities** (Actividades), expanda **General**, arrastre la actividad **Store Procedure** (Procedimiento almacenado) y colóquela en la superficie del diseñador de canalizaciones. 

    ![Operación de arrastrar y colocar la actividad de procedimiento almacenado](./media/how-to-invoke-ssis-package-stored-procedure-activity/drag-drop-sproc-activity.png)
3. En la ventana de propiedades de la actividad de procedimiento almacenado, cambie a la pestaña **SQL Account** (Cuenta de SQL) y haga clic en **+ New** (+ Nuevo). Crea una conexión a la base de datos de Azure SQL Database que hospeda el catálogo de SSIS (base de datos SSIDB). 
   
    ![Botón New linked service (Nuevo servicio vinculado)](./media/how-to-invoke-ssis-package-stored-procedure-activity/new-linked-service-button.png)
4. En la ventana **New Linked Service** (Nuevo servicio vinculado), realice los pasos siguientes: 

    1. Seleccione **Azure SQL Database** para **Type** (Tipo).
    2. Seleccione el entorno de ejecución de Azure **Default** para conectarse a la instancia de Azure SQL Database que hospeda la base de datos `SSISDB`.
    3. Seleccione la base de datos de Azure SQL que hospeda la base de datos SSISDB para el campo **Server name** (Nombre del servidor).
    4. Seleccione **SSISDB** para el campo **Database name** (Nombre de la base de datos).
    5. En **User name** (Nombre de usuario), escriba el nombre del usuario que tiene acceso a la base de datos.
    6. En **Password** (Contraseña), escriba la contraseña del usuario. 
    7. Para probar la conexión con la base de datos, haga clic en el botón **Test connection** (Prueba de conexión).
    8. Guarde el servicio vinculado con un clic en el botón **Save** (Guardar). 

        ![Servicio vinculado de Azure SQL Database](./media/how-to-invoke-ssis-package-stored-procedure-activity/azure-sql-database-linked-service-settings.png)
5. En la ventana de propiedades, cambie a la pestaña **Stored Procedure** (Procedimiento almacenado) de la pestaña **SQL Account** (Cuenta de SQL) y lleve a cabo estos pasos: 

    1. Seleccione **Editar**. 
    2. En el campo **Stored procedure name** (Nombre del procedimiento almacenado), escriba `sp_executesql`. 
    3. Haga clic en **+ New** (+ Nuevo) en la sección **Stored procedure parameters** (Parámetros del procedimiento almacenado). 
    4. En el **nombre** del parámetro, escriba **stmt**. 
    5. En el **tipo** de parámetro, escriba **Cadena**. 
    6. En el **valor** del parámetro, escriba la consulta SQL siguiente:

        En la consulta SQL, especifique los valores correctos para los parámetros **folder_name**, **project_name** y **package_name**. 

        ```sql
        DECLARE @return_value INT, @exe_id BIGINT, @err_msg NVARCHAR(150)    EXEC @return_value=[SSISDB].[catalog].[create_execution] @folder_name=N'<FOLDER name in SSIS Catalog>', @project_name=N'<PROJECT name in SSIS Catalog>', @package_name=N'<PACKAGE name>.dtsx', @use32bitruntime=0, @runinscaleout=1, @useanyworker=1, @execution_id=@exe_id OUTPUT    EXEC [SSISDB].[catalog].[set_execution_parameter_value] @exe_id, @object_type=50, @parameter_name=N'SYNCHRONIZED', @parameter_value=1    EXEC [SSISDB].[catalog].[start_execution] @execution_id=@exe_id, @retry_count=0    IF(SELECT [status] FROM [SSISDB].[catalog].[executions] WHERE execution_id=@exe_id)<>7 BEGIN SET @err_msg=N'Your package execution did not succeed for execution ID: ' + CAST(@exe_id AS NVARCHAR(20)) RAISERROR(@err_msg,15,1) END
        ```

        ![Servicio vinculado de Azure SQL Database](./media/how-to-invoke-ssis-package-stored-procedure-activity/stored-procedure-settings.png)
6. Para validar la configuración de la canalización, haga clic en **Validate** (Validar) en la barra de herramientas. Para cerrar **Pipeline Validation Report** (Informe de comprobación de la canalización), haga clic en **>>** .

    ![Comprobar la canalización](./media/how-to-invoke-ssis-package-stored-procedure-activity/validate-pipeline.png)
7. Publique la canalización en Data Factory con un clic en el botón **Publish All** (Publicar todo). 

    ![Publicar](./media/how-to-invoke-ssis-package-stored-procedure-activity/publish-all-button.png)    

### <a name="run-and-monitor-the-pipeline"></a>Ejecución y supervisión de la canalización
En esta sección, desencadena una ejecución de canalización y luego la supervisa. 

1. Para desencadenar una ejecución de canalización, haga clic en **Trigger** (Desencadenar) en la barra de herramientas y en **Trigger now** (Desencadenar ahora). 

    ![Trigger now (Desencadenar ahora)](media/how-to-invoke-ssis-package-stored-procedure-activity/trigger-now.png)

2. En la ventana **Pipeline Run** (Ejecución de canalización), seleccione **Finish** (Finalizar). 
3. Cambie a la pestaña **Monitor** (Supervisar) de la izquierda. Verá la ejecución de canalización y su estado junto con otro tipo de información (como la hora de inicio de la ejecución). Para actualizar la vista, haga clic en **Refresh** (Actualizar).

    ![Ejecuciones de la canalización](./media/how-to-invoke-ssis-package-stored-procedure-activity/pipeline-runs.png)

3. Haga clic en el vínculo **View Activity Runs** (Ver ejecuciones de actividad) de la columna **Actions** (Acciones). Solo verá una ejecución de actividad porque la canalización solo tiene una actividad (actividad de procedimiento almacenado).

    ![Ejecuciones de actividad](./media/how-to-invoke-ssis-package-stored-procedure-activity/activity-runs.png)

4. Puede ejecutar la **consulta** siguiente en la base de datos de SSISDB en SQL Database para comprobar la ejecución del paquete. 

    ```sql
    select * from catalog.executions
    ```

    ![Comprobación de las ejecuciones del paquete](./media/how-to-invoke-ssis-package-stored-procedure-activity/verify-package-executions.png)


> [!NOTE]
> También puede crear un desencadenador programado para la canalización de manera que esta se ejecute según una programación (por hora, cada día, etc.). Para ver un ejemplo, consulte [Create a data factory - Data Factory UI](quickstart-create-data-factory-portal.md#trigger-the-pipeline-on-a-schedule) (Creación de una factoría de datos: interfaz de usuario de Data Factory).

## <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

En esta sección, usará Azure PowerShell para crear una canalización de Data Factory con una actividad de procedimiento almacenado que invoca un paquete SSIS. 

Instale los módulos de Azure PowerShell siguiendo las instrucciones de [Cómo instalar y configurar Azure PowerShell](/powershell/azure/install-az-ps). 

### <a name="create-a-data-factory"></a>Crear una factoría de datos
Puede usar la misma factoría de datos que tiene el IR de SSIS de Azure o crear una factoría de datos independiente. El siguiente procedimiento detalla los pasos para crear una factoría de datos. Debe crear una canalización con una actividad de procedimiento almacenado en esta factoría de datos. La actividad de procedimiento almacenado ejecuta un procedimiento almacenado en la base de datos SSISDB para ejecutar el paquete de SSIS. 

1. Defina una variable para el nombre del grupo de recursos que usa en los comandos de PowerShell más adelante. Copie el texto del comando siguiente en PowerShell, especifique el nombre del [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) entre comillas dobles y ejecute el comando. Por ejemplo: `"adfrg"`. 
   
     ```powershell
    $resourceGroupName = "ADFTutorialResourceGroup";
    ```

    Si el grupo de recursos ya existe, puede que no desee sobrescribirlo. Asigne otro valor a la variable `$ResourceGroupName` y vuelva a ejecutar el comando
2. Para crear el grupo de recursos de Azure, ejecute el comando siguiente: 

    ```powershell
    $ResGrp = New-AzResourceGroup $resourceGroupName -location 'eastus'
    ``` 
    Si el grupo de recursos ya existe, puede que no desee sobrescribirlo. Asigne otro valor a la variable `$ResourceGroupName` y ejecute el comando de nuevo. 
3. Defina una variable para el nombre de la factoría de datos. 

    > [!IMPORTANT]
    >  Actualice el nombre de la factoría de datos para que sea globalmente único. 

    ```powershell
    $DataFactoryName = "ADFTutorialFactory";
    ```

5. Para crear la factoría de datos, ejecute el siguiente cmdlet **Set-AzDataFactoryV2** con las propiedades ResourceGroupName y Location de la variable $ResGrp: 
    
    ```powershell       
    $DataFactory = Set-AzDataFactoryV2 -ResourceGroupName $ResGrp.ResourceGroupName -Location $ResGrp.Location -Name $dataFactoryName 
    ```

Tenga en cuenta los siguientes puntos:

* El nombre de la instancia de Azure Data Factory debe ser único de forma global. Si recibe el siguiente error, cambie el nombre y vuelva a intentarlo.

    ```
    The specified Data Factory name 'ADFv2QuickStartDataFactory' is already in use. Data Factory names must be globally unique.
    ```
* Para crear instancias de Data Factory, la cuenta de usuario que use para iniciar sesión en Azure debe ser un miembro de los roles **colaborador** o **propietario**, o de **administrador** de la suscripción de Azure.
* Para una lista de las regiones de Azure en las que Data Factory está disponible actualmente, seleccione las regiones que le interesen en la página siguiente y expanda **Análisis** para poder encontrar **Data Factory**: [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/). Los almacenes de datos (Azure Storage, Azure SQL Database, etc.) y los procesos (HDInsight, etc.) que usa la factoría de datos pueden encontrarse en otras regiones.

### <a name="create-an-azure-sql-database-linked-service"></a>Creación de un servicio vinculado de Azure SQL Database
Cree un servicio vinculado para vincular su base de datos que hospeda el catálogo de SSIS con la factoría de datos. Data Factory usa la información de este servicio vinculado para conectarse a la base de datos SSISDB y ejecuta un procedimiento almacenado para ejecutar un paquete de SSIS. 

1. Cree un archivo JSON denominado **AzureSQLDatabaseLinkedService.json** en la carpeta **C:\ADF\RunSSISPackage** con el siguiente contenido: 

    > [!IMPORTANT]
    > Reemplace &lt;servername&gt;, &lt;username&gt; y &lt;password&gt; por los nombres de su base de datos de Azure SQL antes de guardar el archivo.

    ```json
    {
        "name": "AzureSqlDatabaseLinkedService",
        "properties": {
            "type": "AzureSqlDatabase",
            "typeProperties": {
                "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=SSISDB;User ID=<username>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
            }
        }
    }
    ```

2. En **Azure PowerShell**, cambie a la carpeta **C:\ADF\RunSSISPackage**.

3. Ejecute el cmdlet **Set-AzDataFactoryV2LinkedService** para crear el servicio vinculado: **AzureSqlDatabaseLinkedService**. 

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $DataFactory.DataFactoryName -ResourceGroupName $ResGrp.ResourceGroupName -Name "AzureSqlDatabaseLinkedService" -File ".\AzureSqlDatabaseLinkedService.json"
    ```

### <a name="create-a-pipeline-with-stored-procedure-activity"></a>Crear una canalización con una actividad de procedimiento almacenado 
En este paso, debe crear una canalización con una actividad de procedimiento almacenado. La actividad invoca el procedimiento almacenado sp_executesql para ejecutar su paquete de SSIS. 

1. Cree un archivo JSON con el nombre **RunSSISPackagePipeline.json** en la carpeta **C:\ADF\RunSSISPackage** con el siguiente contenido:

    > [!IMPORTANT]
    > Reemplace &lt;FOLDER NAME&gt;, &lt;PROJECT NAME&gt; y &lt;PACKAGE NAME&gt; por los nombres de carpeta, proyecto y paquete del catálogo de SSIS antes de guardar el archivo. 

    ```json
    {
        "name": "RunSSISPackagePipeline",
        "properties": {
            "activities": [
                {
                    "name": "My SProc Activity",
                    "description":"Runs an SSIS package",
                    "type": "SqlServerStoredProcedure",
                    "linkedServiceName": {
                        "referenceName": "AzureSqlDatabaseLinkedService",
                        "type": "LinkedServiceReference"
                    },
                    "typeProperties": {
                        "storedProcedureName": "sp_executesql",
                        "storedProcedureParameters": {
                            "stmt": {
                                "value": "DECLARE @return_value INT, @exe_id BIGINT, @err_msg NVARCHAR(150)    EXEC @return_value=[SSISDB].[catalog].[create_execution] @folder_name=N'<FOLDER NAME>', @project_name=N'<PROJECT NAME>', @package_name=N'<PACKAGE NAME>', @use32bitruntime=0, @runinscaleout=1, @useanyworker=1, @execution_id=@exe_id OUTPUT    EXEC [SSISDB].[catalog].[set_execution_parameter_value] @exe_id, @object_type=50, @parameter_name=N'SYNCHRONIZED', @parameter_value=1    EXEC [SSISDB].[catalog].[start_execution] @execution_id=@exe_id, @retry_count=0    IF(SELECT [status] FROM [SSISDB].[catalog].[executions] WHERE execution_id=@exe_id)<>7 BEGIN SET @err_msg=N'Your package execution did not succeed for execution ID: ' + CAST(@exe_id AS NVARCHAR(20)) RAISERROR(@err_msg,15,1) END"
                            }
                        }
                    }
                }
            ]
        }
    }
    ```

2. Para crear la canalización: **RunSSISPackagePipeline**, ejecute el cmdlet **Set-AzDataFactoryV2Pipeline**.

    ```powershell
    $DFPipeLine = Set-AzDataFactoryV2Pipeline -DataFactoryName $DataFactory.DataFactoryName -ResourceGroupName $ResGrp.ResourceGroupName -Name "RunSSISPackagePipeline" -DefinitionFile ".\RunSSISPackagePipeline.json"
    ```

    Este es la salida de ejemplo:

    ```
    PipelineName      : Adfv2QuickStartPipeline
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Activities        : {CopyFromBlobToBlob}
    Parameters        : {[inputPath, Microsoft.Azure.Management.DataFactory.Models.ParameterSpecification], [outputPath, Microsoft.Azure.Management.DataFactory.Models.ParameterSpecification]}
    ```

### <a name="create-a-pipeline-run"></a>Creación de una ejecución de canalización
Use el cmdlet **Invoke-AzDataFactoryV2Pipeline** para ejecutar la canalización. El cmdlet devuelve el identificador de ejecución de la canalización para realizar una supervisión en un futuro.

```powershell
$RunId = Invoke-AzDataFactoryV2Pipeline -DataFactoryName $DataFactory.DataFactoryName -ResourceGroupName $ResGrp.ResourceGroupName -PipelineName $DFPipeLine.Name
```

### <a name="monitor-the-pipeline-run"></a>Supervisión de la ejecución de la canalización

Ejecute el script de PowerShell siguiente para comprobar continuamente el estado de ejecución de la canalización hasta que termine de copiar los datos. Copie y pegue el siguiente script en la ventana de PowerShell y presione ENTRAR. 

```powershell
while ($True) {
    $Run = Get-AzDataFactoryV2PipelineRun -ResourceGroupName $ResGrp.ResourceGroupName -DataFactoryName $DataFactory.DataFactoryName -PipelineRunId $RunId

    if ($Run) {
        if ($run.Status -ne 'InProgress') {
            Write-Output ("Pipeline run finished. The status is: " +  $Run.Status)
            $Run
            break
        }
        Write-Output  "Pipeline is running...status: InProgress"
    }

    Start-Sleep -Seconds 10
}   
```

### <a name="create-a-trigger"></a>Crear un desencadenador
En el paso anterior, invocó la canalización a petición. También puede crear un desencadenador de programación para ejecutar la canalización en una programación (cada hora, día, etc.).

1. Cree un archivo JSON con el nombre **MyTrigger.json** en la carpeta **C:\ADF\RunSSISPackage** con el siguiente contenido: 

    ```json
    {
        "properties": {
            "name": "MyTrigger",
            "type": "ScheduleTrigger",
            "typeProperties": {
                "recurrence": {
                    "frequency": "Hour",
                    "interval": 1,
                    "startTime": "2017-12-07T00:00:00-08:00",
                    "endTime": "2017-12-08T00:00:00-08:00"
                }
            },
            "pipelines": [{
                    "pipelineReference": {
                        "type": "PipelineReference",
                        "referenceName": "RunSSISPackagePipeline"
                    },
                    "parameters": {}
                }
            ]
        }
    }    
    ```
2. En **Azure PowerShell**, cambie a la carpeta **C:\ADF\RunSSISPackage**.
3. Ejecute el cmdlet **Set-AzDataFactoryV2Trigger**, que crea el desencadenador. 

    ```powershell
    Set-AzDataFactoryV2Trigger -ResourceGroupName $ResGrp.ResourceGroupName -DataFactoryName $DataFactory.DataFactoryName -Name "MyTrigger" -DefinitionFile ".\MyTrigger.json"
    ```
4. De manera predeterminada, el desencadenador está en estado detenido. Inicie el desencadenador al ejecutar el cmdlet **Start-AzDataFactoryV2Trigger**. 

    ```powershell
    Start-AzDataFactoryV2Trigger -ResourceGroupName $ResGrp.ResourceGroupName -DataFactoryName $DataFactory.DataFactoryName -Name "MyTrigger" 
    ```
5. Confirme que el desencadenador se ha iniciado al ejecutar el cmdlet **Get-AzDataFactoryV2Trigger**. 

    ```powershell
    Get-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name "MyTrigger"     
    ```    
6. Ejecute el comando siguiente al comenzar la hora siguiente. Por ejemplo, si la hora actual es 15:25 UTC, ejecute el comando a las 16:00 UTC. 
    
    ```powershell
    Get-AzDataFactoryV2TriggerRun -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -TriggerName "MyTrigger" -TriggerRunStartedAfter "2017-12-06" -TriggerRunStartedBefore "2017-12-09"
    ```

    Puede ejecutar la consulta siguiente en la base de datos de SSISDB en SQL Database para comprobar la ejecución del paquete. 

    ```sql
    select * from catalog.executions
    ```


## <a name="next-steps"></a>Pasos siguientes
También puede supervisar la canalización mediante Azure Portal. Para ver instrucciones paso a paso, consulte [Supervisar la canalización](quickstart-create-data-factory-resource-manager-template.md#monitor-the-pipeline).
