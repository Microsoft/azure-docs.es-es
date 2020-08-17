---
title: Compilación de la primera factoría de datos (PowerShell)
description: En este tutorial, creará un de ejemplo de canalización de Azure Data Factory mediante Azure PowerShell.
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
ms.openlocfilehash: 3f388937c43c9c6a2b9e4700768d4af9cdcb39de
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543119"
---
# <a name="tutorial-build-your-first-azure-data-factory-using-azure-powershell"></a>Tutorial: Compilación de la primera factoría de datos de Azure mediante Azure PowerShell
> [!div class="op_single_selector"]
> * [Introducción y requisitos previos](data-factory-build-your-first-pipeline.md)
> * [Visual Studio](data-factory-build-your-first-pipeline-using-vs.md)
> * [PowerShell](data-factory-build-your-first-pipeline-using-powershell.md)
> * [Plantilla de Resource Manager](data-factory-build-your-first-pipeline-using-arm.md)
> * [REST API](data-factory-build-your-first-pipeline-using-rest-api.md)


> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si utiliza la versión actual del servicio Data Factory, consulte el artículo [Inicio rápido: Creación de una factoría de datos con Azure Data Factory](../quickstart-create-data-factory-powershell.md).

En este artículo, usará Azure PowerShell para crear su primera instancia de Azure Data Factory. Para realizar el tutorial con otros SDK/herramientas, seleccione una de las opciones de la lista desplegable.

La canalización de este tutorial tiene solo una actividad: **Actividad de Hive de HDInsight**. Esta actividad ejecuta un script de Hive en un clúster de Azure HDInsight que transforma los datos de entrada para generar datos de salida. La canalización está programada para ejecutarse una vez al mes entre las horas de inicio y finalización especificadas. 

> [!NOTE]
> En este tutorial, la canalización de datos transforma los datos de entrada para generar datos de salida. No copia los datos de un almacén de datos de origen a un almacén de datos de destino. Para ver un tutorial acerca de cómo copiar datos mediante Azure Data Factory, consulte [Tutorial: Copia de datos de Blob Storage en SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).
> 
> pero cualquier canalización puede tener más de una actividad. También puede encadenar dos actividades (ejecutar una después de otra) haciendo que el conjunto de datos de salida de una actividad sea el conjunto de datos de entrada de la otra actividad. Para más información, consulte [Programación y ejecución en Data Factory](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline).

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

* Lea el artículo [Tutorial: Compilación de la primera canalización para procesar datos mediante el clúster de Hadoop](data-factory-build-your-first-pipeline.md) y complete los pasos de los **requisitos previos** .
* Siga las instrucciones del artículo [Cómo instalar y configurar Azure PowerShell](/powershell/azure/) para instalar la última versión de Azure PowerShell en su equipo.
* (opcional) Este artículo no abarca todos los cmdlets de Factoría de datos. Vea [Referencia de cmdlets de factoría de datos](/powershell/module/az.datafactory) para obtener la documentación completa sobre los cmdlets de la factoría de datos.

## <a name="create-data-factory"></a>Creación de Data Factory
En este paso, utilice Azure PowerShell para crear una Azure Data Factory llamada **FirstDataFactoryPSH**. Una factoría de datos puede tener una o más canalizaciones. Una canalización puede tener una o más actividades. Por ejemplo, una actividad de copia para copiar datos desde un origen hasta un almacén de datos de destino o una actividad de Hive de HDInsight para ejecutar un script de Hive que transforme los datos de entrada. Comencemos con la creación de la factoría de datos en este paso.

1. Inicie Azure PowerShell y ejecute el comando siguiente. Mantenga Azure PowerShell abierto hasta el final de este tutorial. Si lo cierra y vuelve a abrirlo, deberá ejecutar los comandos de nuevo.
   * Ejecute el siguiente comando y escriba el nombre de usuario y la contraseña que utiliza para iniciar sesión en el Portal de Azure.
     ```PowerShell
     Connect-AzAccount
     ```    
   * Ejecute el siguiente comando para ver todas las suscripciones para esta cuenta.
     ```PowerShell
     Get-AzSubscription  
     ```
   * Ejecute el comando siguiente para seleccionar la suscripción con la que desea trabajar. Esta suscripción debe ser la misma que la que usó en el Portal de Azure.
     ```PowerShell
     Get-AzSubscription -SubscriptionName <SUBSCRIPTION NAME> | Set-AzContext
     ```     
2. Cree un grupo de recursos de Azure con el nombre: **ADFTutorialResourceGroup** mediante la ejecución del siguiente comando:
    
    ```PowerShell
    New-AzResourceGroup -Name ADFTutorialResourceGroup  -Location "West US"
    ```
    En algunos de los pasos de este tutorial se supone que se usa el grupo de recursos denominado ADFTutorialResourceGroup. Si usa un otro grupo de recursos, deberá usarlo en lugar de ADFTutorialResourceGroup en este tutorial.
3. Ejecute el cmdlet **New-AzDataFactory** que crea una factoría de datos denominada **FirstDataFactoryPSH**.

    ```PowerShell
    New-AzDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name FirstDataFactoryPSH –Location "West US"
    ```
   Tenga en cuenta los siguientes puntos:

* El nombre de Azure Data Factory debe ser único de forma global. Si recibe el error: **El nombre de la factoría de datos "FirstDataFactoryPSH" no está disponible**, cambie el nombre (por ejemplo, suNombreFirstDataFactoryPSH). Use este nombre en lugar de ADFTutorialFactoryPSH mientras lleva a cabo los pasos de este tutorial. Consulte el tema [Factoría de datos: reglas de nomenclatura](data-factory-naming-rules.md) para las reglas de nomenclatura para los artefactos de Factoría de datos.
* Para crear instancias de Data Factory, debe ser administrador o colaborador de la suscripción de Azure.
* El nombre de la factoría de datos se puede registrar como un nombre DNS en el futuro y, por lo tanto, que sea visible públicamente.
* Si recibe el error: "**La suscripción no está registrada para usar el espacio de nombres Microsoft.DataFactory**", realice una de las acciones siguientes e intente publicarla de nuevo:

  * En Azure PowerShell, ejecute el siguiente comando para registrar el proveedor de Data Factory:

    ```PowerShell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DataFactory
    ```
      Puede ejecutar el comando siguiente para confirmar que se ha registrado el proveedor de Data Factory:

    ```PowerShell
    Get-AzResourceProvider
    ```
  * Inicie sesión mediante la suscripción de Azure en el [Portal de Azure](https://portal.azure.com) y vaya a una hoja de Data Factory (o) cree una factoría de datos en el Portal de Azure. Esta acción registra automáticamente el proveedor.

Antes de crear una canalización, debe crear algunas entidades de factoría de datos primero. En primer lugar, cree servicios vinculados para vincular los almacenes de datos y procesos con su almacén de datos, luego defina los conjuntos de datos de entrada y salida para representar los datos de entrada/salida en los almacenes de datos vinculados y, finalmente, cree la canalización con una actividad que utilice estos conjuntos de datos.

## <a name="create-linked-services"></a>Crear servicios vinculados
En este paso, vinculará su cuenta de Azure Storage y el clúster de Azure HDInsight a petición con su factoría de datos. La cuenta de Azure Storage contiene los datos de entrada y salida de la canalización de este ejemplo. El servicio vinculado de HDInsight se usa para ejecutar un script de Hive especificado en la actividad de la canalización en este ejemplo. Identifique qué datos de almacén o servicios de proceso se usan en el escenario y vincular dichos servicios con la factoría de datos mediante la creación de servicios vinculados.

### <a name="create-azure-storage-linked-service"></a>Creación de un servicio vinculado de Azure Storage
En este paso, vinculará su cuenta de Azure Storage con su factoría de datos. Tiene que usar la misma cuenta de Azure Storage para almacenar los datos de entrada y salida y el archivo de script de HQL.

1. Cree un archivo JSON con el nombre StorageLinkedService.json en la carpeta C:\ADFGetStarted con el siguiente contenido. Si todavía no existe, cree la carpeta ADFGetStarted.

    ```json
    {
        "name": "StorageLinkedService",
        "properties": {
            "type": "AzureStorage",
            "description": "",
            "typeProperties": {
                "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
            }
        }
    }
    ```
    Reemplace **account name** por el nombre de la cuenta de Azure Storage y **account key** por la clave de acceso de la cuenta de Azure Storage. Para aprender a obtener la clave de acceso de almacenamiento, consulte [Administración de claves de acceso de la cuenta de almacenamiento](../../storage/common/storage-account-keys-manage.md).
2. En Azure PowerShell, cambie a la carpeta ADFGetStarted.
3. Puede usar el cmdlet **New-AzDataFactoryLinkedService** que crea un servicio vinculado. Tanto cmdlet como otros cmdlets de Data Factory que usa en este tutorial requieren que se pasen los valores de los parámetros *ResourceGroupName* y *DataFactoryName*. Como alternativa, puede usar **Get-AzDataFactory** para obtener un objeto **DataFactory** y pasarlo sin necesidad de escribir *ResourceGroupName* y *DataFactoryName* cada vez que ejecuta un cmdlet. Ejecute el comando siguiente para asignar el resultado del cmdlet **Get-AzDataFactory** a una variable **$df**.

    ```PowerShell
    $df=Get-AzDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name FirstDataFactoryPSH
    ```
4. Ahora, ejecute el cmdlet **New-AzDataFactoryLinkedService** que crea el servicio vinculado **StorageLinkedService**.

    ```PowerShell
    New-AzDataFactoryLinkedService $df -File .\StorageLinkedService.json
    ```
    Si no hubiera ejecutado el cmdlet**Get-AzDataFactory** y asignado la salida a la variable **$df**, tendría que especificar valores para los parámetros *ResourceGroupName* y *DataFactoryName* de la siguiente forma.

    ```PowerShell
    New-AzDataFactoryLinkedService -ResourceGroupName ADFTutorialResourceGroup -DataFactoryName FirstDataFactoryPSH -File .\StorageLinkedService.json
    ```
    Si cierra Azure PowerShell en mitad del tutorial, tendrá que ejecutar el cmdlet **Get-AzDataFactory** la próxima vez que inicie Azure PowerShell para completar el tutorial.

### <a name="create-azure-hdinsight-linked-service"></a>Creación de un servicio vinculado de HDInsight de Azure
En este paso, vinculará un clúster de HDInsight a petición con la factoría de datos. El clúster de HDInsight se crea automáticamente en tiempo de ejecución y se elimina después de hacer el procesamiento y de permanecer inactivo durante el período de tiempo especificado. Puede usar su propio clúster de HDInsight en lugar de usar un clúster de HDInsight a petición. Consulte [Servicios vinculados de procesos](data-factory-compute-linked-services.md) para más información.

1. Cree un archivo JSON con el nombre **HDInsightOnDemandLinkedService**.json en la carpeta **C:\ADFGetStarted** con el siguiente contenido.

    ```json
    {
        "name": "HDInsightOnDemandLinkedService",
        "properties": {
            "type": "HDInsightOnDemand",
            "typeProperties": {
                "version": "3.5",
                "clusterSize": 1,
                "timeToLive": "00:05:00",
                "osType": "Linux",
                "linkedServiceName": "StorageLinkedService"
            }
        }
    }
    ```
    En la siguiente tabla se ofrecen descripciones de las propiedades JSON que se usan en el fragmento de código:

   | Propiedad | Descripción |
   |:--- |:--- |
   | clusterSize |Especifica el tamaño del clúster de HDInsight. |
   | timeToLive |Especifica el tiempo de inactividad del clúster de HDInsight, antes de que se elimine. |
   | linkedServiceName |Especifica la cuenta de almacenamiento que se usa para almacenar los registros que genera HDInsight. |

    Tenga en cuenta los siguientes puntos:

   * Data Factory crea un clúster de HDInsight **basado en Linux** con el código JSON. Para más información, consulte la sección [Servicio vinculado a petición de HDInsight de Azure](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) .
   * Puede usar **su propio clúster de HDInsight** en lugar de usar un clúster de HDInsight a petición. Para más información, consulte [Servicio vinculado de HDInsight de Azure](data-factory-compute-linked-services.md#azure-hdinsight-linked-service) .
   * El clúster de HDInsight crea un **contenedor predeterminado** en el almacenamiento de blobs que especificó en JSON (**linkedServiceName**). HDInsight no elimina este contenedor cuando se elimina el clúster. Este comportamiento es así por diseño. Con el servicio vinculado de HDInsight a petición se crea un clúster de HDInsight cada vez se procesa un segmento, a menos que haya un clúster existente activo (**timeToLive**). El clúster se elimina automáticamente cuando se realiza el procesamiento.

       A medida que se procesen más segmentos, verá numerosos contenedores en su Almacenamiento de blobs de Azure. Si no los necesita para solucionar problemas de trabajos, puede eliminarlos para reducir el costo de almacenamiento. Los nombres de estos contenedores siguen este patrón: "adf**nombredefactoría dedatos**-**nombredelserviciovinculado**-marcadefechayhora". Use herramientas como el [Explorador de Microsoft Azure Storage](https://storageexplorer.com/) para eliminar contenedores de Azure Blob Storage.

     Para más información, consulte la sección [Servicio vinculado a petición de HDInsight de Azure](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) .
2. Ejecute el cmdlet **New-AzDataFactoryLinkedService** que crea el servicio vinculado llamado HDInsightOnDemandLinkedService.
    
    ```PowerShell
    New-AzDataFactoryLinkedService $df -File .\HDInsightOnDemandLinkedService.json
    ```

## <a name="create-datasets"></a>Creación de conjuntos de datos
En este paso, creará conjuntos de datos que representan los datos de entrada y salida para el procesamiento de Hive. Estos conjuntos de datos hacen referencia al servicio **StorageLinkedService** que ha creado anteriormente en este tutorial. El servicio vinculado apunta a una cuenta de Azure Storage y los conjuntos de datos especifican el contenedor, la carpeta y el nombre de archivo en el almacenamiento que contiene los datos de entrada y salida.

### <a name="create-input-dataset"></a>Creación de un conjunto de datos de entrada
1. Cree un archivo JSON con el nombre **InputTable.json** en la carpeta **C:\ADFGetStarted** con el siguiente contenido:

    ```json
    {
        "name": "AzureBlobInput",
        "properties": {
            "type": "AzureBlob",
            "linkedServiceName": "StorageLinkedService",
            "typeProperties": {
                "fileName": "input.log",
                "folderPath": "adfgetstarted/inputdata",
                "format": {
                    "type": "TextFormat",
                    "columnDelimiter": ","
                }
            },
            "availability": {
                "frequency": "Month",
                "interval": 1
            },
            "external": true,
            "policy": {}
        }
    }
    ```
    Este código JSON define un conjunto de datos denominado **AzureBlobInput**, que representa los datos de entrada de una actividad en la canalización. Además, especifica que los datos de entrada se colocan en el contenedor de blobs **adfgetstarted** y en la carpeta **inputdata**.

    En la siguiente tabla se ofrecen descripciones de las propiedades JSON que se usan en el fragmento de código:

   | Propiedad | Descripción |
   |:--- |:--- |
   | type |La propiedad type se establece en AzureBlob porque los datos residen en Almacenamiento de blobs de Azure. |
   | linkedServiceName |hace referencia a StorageLinkedService que creó anteriormente. |
   | fileName |Esta propiedad es opcional. Si omite esta propiedad, se seleccionan todos los archivos de folderPath. En este caso, se procesa solo el archivo input.log. |
   | type |Los archivos de registro están en formato de texto, por lo que usaremos TextFormat. |
   | columnDelimiter |Las columnas de los archivos de registro están delimitadas por una coma (,). |
   | frequency/interval |La frecuencia está establecida en Mes y el intervalo es 1, lo que significa que los segmentos de entrada estarán disponibles cada mes. |
   | external |Esta propiedad se establece en true si el servicio Factoría de datos no ha generado los datos de entrada. |
2. Ejecute el comando siguiente en Azure PowerShell para crear el conjunto de datos de Data Factory:

    ```PowerShell
    New-AzDataFactoryDataset $df -File .\InputTable.json
    ```

### <a name="create-output-dataset"></a>Creación del conjunto de datos de salida
Ahora, se crea el conjunto de datos de salida que representa los datos de salida almacenados en Almacenamiento de blobs de Azure.

1. Cree un archivo JSON con el nombre **OutputTable.json** en la carpeta **C:\ADFGetStarted** con el siguiente contenido:

    ```json
    {
      "name": "AzureBlobOutput",
      "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
          "folderPath": "adfgetstarted/partitioneddata",
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
    Este código JSON define un conjunto de datos denominado **AzureBlobOutput**, que representa los datos de salida de una actividad en la canalización. Además, especifica que los resultados se almacenan en el contenedor de blobs **adfgetstarted** y en la carpeta **partitioneddata**. La sección **availability** especifica que el conjunto de datos de salida se genera mensualmente.
2. Ejecute el comando siguiente en Azure PowerShell para crear el conjunto de datos de Data Factory:

    ```PowerShell
    New-AzDataFactoryDataset $df -File .\OutputTable.json
    ```

## <a name="create-pipeline"></a>Creación de una canalización
En este paso, creará la primera canalización con una actividad **HDInsightHive** . El segmento de entrada está disponible de forma mensual (frecuencia: mensual, intervalo: 1), el segmento de salida se genera mensualmente y la propiedad de programador de la actividad también se establece en mensual. La configuración del conjunto de datos de salida y la del programador de la actividad deben coincidir. Actualmente, el conjunto de datos de salida es lo que controla la programación, por lo que debe crear un conjunto de datos de salida aunque la actividad no genere ninguna salida. Si la actividad no toma ninguna entrada, puede omitir la creación del conjunto de datos de entrada. Al final de esta sección se explican las propiedades usadas en el siguiente JSON.

1. Cree un archivo JSON con el nombre MyFirstPipelinePSH.json en la carpeta C:\ADFGetStarted con el siguiente contenido:

   > [!IMPORTANT]
   > Reemplace **storageaccountname** por el nombre de la cuenta de almacenamiento en JSON.
   >
   >

    ```json
    {
        "name": "MyFirstPipeline",
        "properties": {
            "description": "My first Azure Data Factory pipeline",
            "activities": [
                {
                    "type": "HDInsightHive",
                    "typeProperties": {
                        "scriptPath": "adfgetstarted/script/partitionweblogs.hql",
                        "scriptLinkedService": "StorageLinkedService",
                        "defines": {
                            "inputtable": "wasb://adfgetstarted@<storageaccountname>.blob.core.windows.net/inputdata",
                            "partitionedtable": "wasb://adfgetstarted@<storageaccountname>.blob.core.windows.net/partitioneddata"
                        }
                    },
                    "inputs": [
                        {
                            "name": "AzureBlobInput"
                        }
                    ],
                    "outputs": [
                        {
                            "name": "AzureBlobOutput"
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
                    "linkedServiceName": "HDInsightOnDemandLinkedService"
                }
            ],
            "start": "2017-07-01T00:00:00Z",
            "end": "2017-07-02T00:00:00Z",
            "isPaused": false
        }
    }
    ```
    En el fragmento de código JSON, se crea una canalización que consta de una sola actividad que usa Hive para procesar los datos en un clúster de HDInsight.

    El archivo de script de Hive, **partitionweblogs.hql**, se almacena en la cuenta de Azure Storage (especificada mediante la propiedad scriptLinkedService, llamada **StorageLinkedService**) en una carpeta denominada **script** del contenedor **adfgetstarted**.

    La sección **defines** se usa para especificar la configuración de tiempo de ejecución que se debe pasar al script de Hive como valores de configuración de Hive (por ejemplo, ${hiveconf:inputtable}, ${hiveconf:partitionedtable}).

    Las propiedades **start** y **end** de la canalización especifican el período activo de esta canalización.

    En el código JSON de la actividad, debe indicar que el script de Hive se ejecuta en el proceso especificado por **linkedServiceName** – **HDInsightOnDemandLinkedService**.

   > [!NOTE]
   > Consulte la sección "JSON de canalización" en [Canalizaciones y actividades en Azure Data Factory](data-factory-create-pipelines.md) para más información acerca de las propiedades de JSON utilizadas en el ejemplo.

2. Confirme que el archivo **input.log** aparece en la carpeta **adfgetstarted/inputdata** en Azure Blob Storage y ejecute el siguiente comando para implementar la canalización. Como las fechas de **inicio** y **finalización** están establecidas en el pasado e **isPaused** está establecido en false, la canalización (la actividad de la canalización) se ejecuta nada más realizar la implementación.

    ```PowerShell
    New-AzDataFactoryPipeline $df -File .\MyFirstPipelinePSH.json
    ```
3. Enhorabuena, ya creó correctamente su primera canalización con Azure PowerShell.

## <a name="monitor-pipeline"></a>Supervisión de la canalización
En este paso, se usa Azure PowerShell para supervisar lo que ocurre en una Data Factory de Azure.

1. Ejecute **Get-AzDataFactory** y asigne el resultado a la variable **$df**.

    ```PowerShell
    $df=Get-AzDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name FirstDataFactoryPSH
    ```
2. Ejecute **Get-AzDataFactorySlice** para obtener la información sobre todos los segmentos de **EmpSQLTable**, que es la tabla de salida de la canalización.

    ```PowerShell
    Get-AzDataFactorySlice $df -DatasetName AzureBlobOutput -StartDateTime 2017-07-01
    ```
    Observe que la StartDateTime que especifique aquí es la misma hora de inicio especificada en el JSON de la canalización. Este es la salida de ejemplo:

    ```PowerShell
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : FirstDataFactoryPSH
    DatasetName       : AzureBlobOutput
    Start             : 7/1/2017 12:00:00 AM
    End               : 7/2/2017 12:00:00 AM
    RetryCount        : 0
    State             : InProgress
    SubState          :
    LatencyStatus     :
    LongRetryCount    : 0
    ```
3. Ejecute **Get-AzDataFactoryRun** para más información de las ejecuciones de actividad para un segmento específico.

    ```PowerShell
    Get-AzDataFactoryRun $df -DatasetName AzureBlobOutput -StartDateTime 2017-07-01
    ```

    Este es la salida de ejemplo: 

    ```PowerShell
    Id                  : 0f6334f2-d56c-4d48-b427-d4f0fb4ef883_635268096000000000_635292288000000000_AzureBlobOutput
    ResourceGroupName   : ADFTutorialResourceGroup
    DataFactoryName     : FirstDataFactoryPSH
    DatasetName         : AzureBlobOutput
    ProcessingStartTime : 12/18/2015 4:50:33 AM
    ProcessingEndTime   : 12/31/9999 11:59:59 PM
    PercentComplete     : 0
    DataSliceStart      : 7/1/2017 12:00:00 AM
    DataSliceEnd        : 7/2/2017 12:00:00 AM
    Status              : AllocatingResources
    Timestamp           : 12/18/2015 4:50:33 AM
    RetryAttempt        : 0
    Properties          : {}
    ErrorMessage        :
    ActivityName        : RunSampleHiveActivity
    PipelineName        : MyFirstPipeline
    Type                : Script
    ```
    Puede seguir ejecutando este cmdlet hasta que vea que el segmento se encuentra en el estado **Listo** o **Con error**. Cuando el segmento tenga el estado Listo, busque los datos de salida en la carpeta **partitioneddata** del contenedor **adfgetstarted** del almacenamiento de blobs.  La creación de un clúster de HDInsight a petición normalmente requiere algo de tiempo.

    ![datos de salida](./media/data-factory-build-your-first-pipeline-using-powershell/three-ouptut-files.png)

> [!IMPORTANT]
> La creación de un clúster de HDInsight a petición normalmente tarda algún tiempo (20 minutos aproximadamente). Por tanto, cabe esperar que la canalización tarde **aproximadamente 30 minutos** en procesar el segmento.
>
> El archivo de entrada se elimina cuando el segmento se procesa correctamente. Por lo tanto, si desea volver a ejecutar el segmento o volver a realizar el tutorial, cargue el archivo de entrada (input.log) en la carpeta inputdata del contenedor adfgetstarted.
>
>

## <a name="summary"></a>Resumen
En este tutorial, ha creado una instancia de Data Factory de Azure para procesar datos mediante la ejecución de un script de Hive en un clúster de Hadoop en HDInsight. Ha usado el Editor de Data Factory en el Portal de Azure para realizar los siguientes pasos:

1. Ha creado una **factoría de datos**de Azure.
2. Ha creado dos **servicios vinculados**:
   1. **Azure Storage** para vincular la instancia de Azure Blob Storage que contiene los archivos de entrada y salida a la factoría de datos.
   2. **HDInsight de Azure** para vincular un clúster de Hadoop en HDInsight a petición a la factoría de datos. Azure Data Factory crea un clúster de Hadoop en HDInsight justo a tiempo para procesar los datos de entrada y generar los datos de salida.
3. Ha creado dos **conjuntos de datos**, que describen los datos de entrada y salida para la actividad de Hive de HDInsight en la canalización.
4. Ha creado una **canalización** con una actividad de **Hive de HDInsight**.

## <a name="next-steps"></a>Pasos siguientes
En este artículo, creó una canalización con una actividad de transformación (actividad de HDInsight) que ejecuta un script de Hive en un clúster de HDInsight de Azure a petición. Para ver cómo se usa una actividad de copia para copiar datos de un blob de Azure en SQL Azure, consulte [Tutorial: Copia de datos de un blob de Azure a Azure SQL](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

## <a name="see-also"></a>Consulte también

| Tema | Descripción |
|:--- |:--- |
| [Referencia para cmdlets de Factoría de datos](/powershell/module/az.datafactory) |Consulte la documentación completa sobre los cmdlets de Factoría de datos |
| [Canalizaciones](data-factory-create-pipelines.md) |Este artículo ayuda a conocer las canalizaciones y actividades de Azure Data Factory y cómo aprovecharlas para construir flujos de trabajo controlados por datos de un extremo a otro para su escenario o negocio. |
| [Conjuntos de datos](data-factory-create-datasets.md) |Este artículo le ayuda a comprender los conjuntos de datos de Azure Data Factory. |
| [Programación y ejecución](data-factory-scheduling-and-execution.md) |En este artículo se explican los aspectos de programación y ejecución del modelo de aplicación de Azure Data Factory. |
| [Supervisión y administración de canalizaciones de Data Factory de Azure mediante la nueva Aplicación de supervisión y administración](data-factory-monitor-manage-app.md) |En este artículo se describe cómo supervisar, administrar y depurar las canalizaciones mediante la aplicación de supervisión y administración. |
