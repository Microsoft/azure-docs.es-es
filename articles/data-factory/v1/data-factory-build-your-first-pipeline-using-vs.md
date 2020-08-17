---
title: Compilación de la primera factoría de datos (Visual Studio)
description: En este tutorial, creará un de ejemplo de canalización de Azure Data Factory mediante Visual Studio.
services: data-factory
documentationcenter: ''
author: djpmsft
ms.author: daperlov
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.custom: vs-azure
ms.date: 01/22/2018
ms.openlocfilehash: f3c6364226288ffb6796664e89d0b7c2cc4dadf6
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543153"
---
# <a name="tutorial-create-a-data-factory-by-using-visual-studio"></a>Tutorial: Creación de una instancia de Data Factory mediante Visual Studio
> [!div class="op_single_selector" title="Tools/SDKs"]
> * [Introducción y requisitos previos](data-factory-build-your-first-pipeline.md)
> * [Visual Studio](data-factory-build-your-first-pipeline-using-vs.md)
> * [PowerShell](data-factory-build-your-first-pipeline-using-powershell.md)
> * [Plantilla de Resource Manager](data-factory-build-your-first-pipeline-using-arm.md)
> * [REST API](data-factory-build-your-first-pipeline-using-rest-api.md)


> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si usa la versión actual del servicio Data Factory, consulte [Guía de inicio rápido: Creación de una factoría de datos mediante Data Factory](../quickstart-create-data-factory-dot-net.md).

Este tutorial muestra cómo crear una instancia de Azure Data Factory mediante Visual Studio. Va a crear un proyecto de Visual Studio mediante la plantilla de proyectos de Data Factory, a definir entidades de Data Factory (servicios vinculados, conjuntos de datos y canalización) en formato JSON y, finalmente, a publicar o implementar estas entidades en la nube. 

La canalización de este tutorial tiene una actividad: **actividad de HDInsight Hive**. Esta actividad ejecuta un script de Hive en un clúster de Azure HDInsight que transforma los datos de entrada para generar datos de salida. La canalización está programada para ejecutarse una vez al mes entre las horas de inicio y finalización especificadas. 

> [!NOTE]
> Este tutorial no muestra cómo copiar datos mediante Azure Data Factory. Para ver un tutorial acerca de cómo copiar datos mediante Azure Data Factory, consulte [Copia de datos de Blob Storage en SQL Database mediante Data Factory](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).
> 
> pero cualquier canalización puede tener más de una actividad. También puede encadenar dos actividades (ejecutar una después de otra) haciendo que el conjunto de datos de salida de una actividad sea el conjunto de datos de entrada de la otra actividad. Para más información, consulte [Programación y ejecución en Data Factory](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline).


## <a name="walkthrough-create-and-publish-data-factory-entities"></a>Tutorial: Creación y publicación de entidades de Data Factory
Estos son los pasos que debe realizar en este tutorial:

1. Cree dos servicios vinculados: **AzureStorageLinkedService1** y **HDInsightOnDemandLinkedService1**. 
   
    En este tutorial, los datos de entrada y los de salida de la actividad de Hive están en la misma instancia de Azure Blob Storage. Va a usar un clúster de HDInsight a petición para procesar los datos de entrada existentes para generar datos de salida. El clúster de HDInsight a petición se crea automáticamente mediante Azure Data Factory en tiempo de ejecución cuando los datos de entrada están listos para su procesamiento. Debe vincular los almacenes o procesos de datos a la factoría de datos para que el servicio de Data Factory pueda conectarse a ellos en tiempo de ejecución. Por tanto, va a vincular su cuenta de Azure Storage a la factoría de datos mediante AzureStorageLinkedService1 y a vincular un clúster de HDInsight a petición mediante HDInsightOnDemandLinkedService1. Al publicar, especifique el nombre de la factoría de datos que va a crear o de una ya existente.  
2. Cree dos conjuntos de datos: **InputDataset** y **OutputDataset**, que representan los datos de entrada y salida que se almacenan en Azure Blob Storage. 
   
    Estas definiciones de conjunto de datos hacen referencia al servicio vinculado de Azure Storage que creó en el paso anterior. Para InputDataset, debe especificar el contenedor de blobs (adfgetstarted) y la carpeta (inptutdata) que contiene un blob con los datos de entrada. Para OutputDataset, debe especificar el contenedor de blobs (adfgetstarted) y la carpeta (partitioneddata) que contiene los datos de salida. Especifique también otras propiedades como estructura, disponibilidad y directiva.
3. Cree una canalización llamada **MyFirstPipeline**. 
  
    En este tutorial, la canalización tiene una actividad: **actividad de HDInsight Hive**. Esta actividad transforma los datos de entrada para generar datos de salida mediante la ejecución de un script de Hive en un clúster de HDInsight a petición. Para más información acerca de la actividad de Hive, consulte [Actividad de Hive](data-factory-hive-activity.md) 
4. Cree una instancia de Data Factory denominada **DataFactoryUsingVS**. Implemente tanto la instancia de Data Factory como todas las entidades de Data Factory (servicios vinculados, tablas y la canalización).
5. Después de publicar, use las hojas de Azure Portal y la aplicación de supervisión y administración para supervisar la canalización. 
  
### <a name="prerequisites"></a>Prerequisites

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

1. Lea el artículo [Tutorial: Compilación de la primera canalización para procesar datos mediante el clúster de Hadoop](data-factory-build-your-first-pipeline.md) y complete los pasos de los **requisitos previos** . También puede seleccionar la opción **Información general y requisitos previos** en la lista desplegable de la parte superior para cambiar al artículo. Después de completar los requisitos previos, vuelva a este artículo mediante la selección de la opción **Visual Studio** de la lista desplegable.
2. Para crear instancias de Data Factory, es preciso ser miembro del rol [Colaborador de Data Factory](../../role-based-access-control/built-in-roles.md#data-factory-contributor) en el nivel de grupo de recursos o suscripción.  
3. Debe tener lo siguiente instalado en el equipo:
   * Visual Studio 2013 o Visual Studio 2015.
   * Descargue el SDK de Azure para Visual Studio 2013 o Visual Studio 2015. Vaya a la [página Descargas de Azure](https://azure.microsoft.com/downloads/) y haga clic en **VS 2013** o **VS 2015** en la sección **.NET**.
   * Descargue el último complemento de Azure Data Factory para Visual Studio: [VS 2013](https://visualstudiogallery.msdn.microsoft.com/754d998c-8f92-4aa7-835b-e89c8c954aa5) o [VS 2015](https://visualstudiogallery.msdn.microsoft.com/371a4cf9-0093-40fa-b7dd-be3c74f49005). También puede actualizar el complemento, para lo que debe seguir estos pasos: en el menú, haga clic en **Herramientas** -> **Extensiones y actualizaciones** -> **En línea** -> **Galería de Visual Studio** -> **Herramientas de Microsoft Azure Data Factory para Visual Studio** -> **Actualizar**.

Vamos a usa Visual Studio para crear una instancia de Azure Data Factory.

### <a name="create-visual-studio-project"></a>Creación de un proyecto de Visual Studio
1. Inicie **Visual Studio 2013** o **Visual Studio 2015**. Haga clic en **Archivo**, seleccione **Nuevo** y, luego, haga clic en **Proyecto**. Debería ver el cuadro de diálogo **Nuevo proyecto** .  
2. En el cuadro de diálogo **Nuevo proyecto**, seleccione la plantilla **DataFactory** y haga clic en **Proyecto vacío de factoría de datos**.   

    ![Cuadro de diálogo Nuevo proyecto](./media/data-factory-build-your-first-pipeline-using-vs/new-project-dialog.png)
3. Escriba un **nombre** para el proyecto, la **ubicación** y un nombre para la **solución**; a continuación, haga clic en **Aceptar**.

    ![Explorador de soluciones](./media/data-factory-build-your-first-pipeline-using-vs/solution-explorer.png)

### <a name="create-linked-services"></a>Crear servicios vinculados
En este paso se crean dos servicios vinculados: **Azure Storage** y **HDInsight a petición**. 

El servicio vinculado de Azure Storage vincula una cuenta de Azure Storage a Data Factory proporcionando la información de conexión. El servicio Data Factory utiliza la cadena de conexión del valor de configuración del servicio vinculado para conectarse a Azure Storage en tiempo de ejecución. Este almacenamiento contiene los datos de entrada y salida de la canalización y el archivo de script de Hive utilizado por la actividad de Hive. 

Con el servicio vinculado de HDInsight a petición, se crea automáticamente el clúster de HDInsight en tiempo de ejecución cuando los datos de entrada están listos para su procesamiento. El clúster se elimina después de hacer el procesamiento y de permanecer inactivo durante el período de tiempo especificado. 

> [!NOTE]
> Puede crear una instancia de Data Factory especificando su nombre y la configuración en el momento de la publicación de la solución de Data Factory.

#### <a name="create-azure-storage-linked-service"></a>Creación de un servicio vinculado de Azure Storage
1. Haga clic con el botón derecho en **Servicios vinculados** en el Explorador de soluciones, seleccione **Agregar** y haga clic en **Nuevo elemento**.      
2. En el cuadro de diálogo **Agregar nuevo elemento**, seleccione **Servicios vinculados de Azure Storage** en la lista y haga clic en **Agregar**.
    ![Servicio vinculado de Azure Storage](./media/data-factory-build-your-first-pipeline-using-vs/new-azure-storage-linked-service.png)
3. Reemplace `<accountname>` y `<accountkey>` por el nombre de la cuenta de Azure Storage y su clave. Para aprender a obtener la clave de acceso de almacenamiento, consulte [Administración de claves de acceso de la cuenta de almacenamiento](../../storage/common/storage-account-keys-manage.md).
    ![Servicio vinculado de Azure Storage](./media/data-factory-build-your-first-pipeline-using-vs/azure-storage-linked-service.png)
4. Guarde el archivo **AzureStorageLinkedService1.json** .

#### <a name="create-azure-hdinsight-linked-service"></a>Creación de un servicio vinculado de HDInsight de Azure
1. En el **Explorador de soluciones**, haga clic con el botón derecho en **Servicios vinculados**, seleccione **Agregar** y haga clic en **Nuevo elemento**.
2. Seleccione **HDInsight On Demand Linked Service** (Servicio vinculado de HDInsight a petición) y haga clic en **Agregar**.
3. Reemplace el código **JSON** por el siguiente:

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
                "linkedServiceName": "AzureStorageLinkedService1"
            }
        }
    }
    ```

    En la siguiente tabla se ofrecen descripciones de las propiedades JSON que se usan en el fragmento de código:

    Propiedad | Descripción
    -------- | ----------- 
    clusterSize | Especifica el tamaño del clúster de HDInsight Hadoop.
    timeToLive | Especifica el tiempo de inactividad del clúster de HDInsight, antes de que se elimine.
    linkedServiceName | Especifica la cuenta de almacenamiento que se usa para almacenar los registros que genera el clúster de HDInsight Hadoop. 

    > [!IMPORTANT]
    > El clúster de HDInsight crea un **contenedor predeterminado** en el almacenamiento de blobs que especificó en JSON (linkedServiceName). HDInsight no elimina este contenedor cuando se elimina el clúster. Este comportamiento es así por diseño. Con el servicio vinculado de HDInsight a petición se crea un clúster de HDInsight cada vez que se procesa un segmento, a menos que haya un clúster existente activo (timeToLive). El clúster se elimina automáticamente cuando se realiza el procesamiento.
    > 
    > A medida que se procesen más segmentos, verá numerosos contenedores en su Almacenamiento de blobs de Azure. Si no los necesita para solucionar problemas de trabajos, puede eliminarlos para reducir el costo de almacenamiento. Los nombres de estos contenedores siguen un patrón: `adf<yourdatafactoryname>-<linkedservicename>-datetimestamp`. Use herramientas como el [Explorador de Microsoft Azure Storage](https://storageexplorer.com/) para eliminar contenedores de Azure Blob Storage.

    Para más información sobre las propiedades JSON, consulte [Servicios vinculados de procesos](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service). 
4. Guarde el archivo **HDInsightOnDemandLinkedService1.json** .

### <a name="create-datasets"></a>Creación de conjuntos de datos
En este paso, creará conjuntos de datos que representan los datos de entrada y salida para el procesamiento de Hive. Estos conjuntos de datos hacen referencia al servicio **AzureStorageLinkedService1** que creó anteriormente en este tutorial. El servicio vinculado apunta a una cuenta de Azure Storage y los conjuntos de datos especifican el contenedor, la carpeta y el nombre de archivo en el almacenamiento que contiene los datos de entrada y salida.   

#### <a name="create-input-dataset"></a>Creación de un conjunto de datos de entrada
1. En el **Explorador de soluciones**, haga clic con el botón derecho en **Tablas**, seleccione **Agregar** y haga clic en **Nuevo elemento**.
2. Seleccione **Blob de Azure** en la lista, cambie el nombre del archivo a **InputDataSet.json** y haga clic en **Agregar**.
3. Reemplace el código **JSON** del editor por el siguiente fragmento de código JSON:

    ```json
    {
        "name": "AzureBlobInput",
        "properties": {
            "type": "AzureBlob",
            "linkedServiceName": "AzureStorageLinkedService1",
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
    Este fragmento de código JSON define un conjunto de datos denominado **AzureBlobInput** que representa los datos de entrada de la actividad de Hive de la canalización. Debe indicar que los datos de entrada se encuentran en el contenedor de blobs llamado `adfgetstarted` y en la carpeta `inputdata`.

    En la siguiente tabla se ofrecen descripciones de las propiedades JSON que se usan en el fragmento de código:

    Propiedad | Descripción |
    -------- | ----------- |
    type |La propiedad type se establece en **AzureBlob** porque los datos residen en Azure Blob Storage.
    linkedServiceName | Hace referencia al servicio AzureStorageLinkedService1 que creó anteriormente.
    fileName |Esta propiedad es opcional. Si omite esta propiedad, se seleccionan todos los archivos de folderPath. En este caso, se procesa solo el archivo input.log.
    type | Los archivos de registro están en formato de texto, por lo que usaremos TextFormat. |
    columnDelimiter | Las columnas de los archivos de registro están delimitadas por una coma (`,`).
    frequency/interval | La frecuencia está establecida en Mes y el intervalo es 1, lo que significa que los segmentos de entrada estarán disponibles cada mes.
    external | Esta propiedad se establece en true si los datos de entrada de la actividad no se han generado en la canalización. Esta propiedad solo se especifica en los conjuntos de datos de entrada. Para el conjunto de datos de entrada de la primera actividad, establézcala siempre en true.
4. Guarde el archivo **InputDataset.json** .

#### <a name="create-output-dataset"></a>Creación del conjunto de datos de salida
Ahora, se crea el conjunto de datos de salida que representa los datos de salida almacenados en Azure Blob Storage.

1. En el **Explorador de soluciones**, haga clic con el botón derecho en **Tablas**, seleccione **Agregar** y haga clic en **Nuevo elemento**.
2. Seleccione **Blob de Azure** en la lista, cambie el nombre del archivo a **OutputDataset.json** y haga clic en **Agregar**.
3. Reemplace el código **JSON** en el editor por el siguiente:
    
    ```json
    {
        "name": "AzureBlobOutput",
        "properties": {
            "type": "AzureBlob",
            "linkedServiceName": "AzureStorageLinkedService1",
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
    Este fragmento de código JSON define un conjunto de datos denominado **AzureBlobOutput** que representa los datos de salida generados por la actividad de Hive de la canalización. Debe especificar que los datos de salida generados por la actividad de Hive se encuentran en el contenedor de blobs llamado `adfgetstarted` y en la carpeta `partitioneddata`. 
    
    La sección **availability** especifica que el conjunto de datos de salida se genera mensualmente. El conjunto de datos de salida controla la programación de la canalización. La canalización se ejecuta mensualmente a una hora específica comprendida entre sus horas de inicio y finalización. 

    Consulte la sección **Creación del conjunto de datos de entrada** para obtener las descripciones de estas propiedades. No establezca la propiedad externa en un conjunto de datos ya que este es generado por la canalización.
4. Guarde el archivo **OutputDataset.json** .

### <a name="create-pipeline"></a>Creación de una canalización
Hasta ahora se han creado el servicio vinculado de Azure Storage y los conjuntos de datos de entrada y salida. Ahora, cree una canalización con una actividad **HDInsightHive**. La **entrada** de la actividad de Hive está establecida en **AzureBlobInput** y la **salida** de la actividad está establecida en **AzureBlobOutput**. Un segmento de un conjunto de datos de entrada está disponible mensualmente (frecuencia: mes, intervalo: 1), y también se genera mensualmente un segmento de salida. 

1. En el **Explorador de soluciones**, haga clic con el botón derecho en **Canalizaciones**, seleccione **Agregar** y haga clic en **Nuevo elemento**.
2. Seleccione **Hive Transformation Pipeline** (Canalización de transformación de Hive) en la lista y haga clic en **Agregar**.
3. Reemplace el código **JSON** por el siguiente fragmento de código:

    > [!IMPORTANT]
    > Reemplace `<storageaccountname>` por el nombre de la cuenta de almacenamiento.

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
                        "scriptLinkedService": "AzureStorageLinkedService1",
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
            "start": "2016-04-01T00:00:00Z",
            "end": "2016-04-02T00:00:00Z",
            "isPaused": false
        }
    }
    ```

    > [!IMPORTANT]
    > Reemplace `<storageaccountname>` por el nombre de la cuenta de almacenamiento.

    El fragmento de código JSON define una canalización que consta de una sola actividad (actividad de Hive). Esta actividad ejecuta un script de Hive en un clúster de Azure HDInsight que procesa los datos de entrada para generar datos de salida. En la sección de actividades de la canalización JSON, solo puede ver una actividad de la matriz con el tipo establecido en **HDInsightHive**. 

    En las propiedades de tipo que son específicas de la actividad de HDInsight Hive, debe especificar qué servicio vinculado de Azure Storage tiene el archivo de script de Hive, la ruta de acceso a este y sus parámetros. 

    El archivo de script de Hive, **partitionweblogs.hql**, se guarda en la cuenta de almacenamiento de Azure (que especifica scriptLinkedService) y en la carpeta `script` del contenedor `adfgetstarted`.

    La sección `defines` se usa para especificar la configuración de tiempo de ejecución que se pasa al script de Hive como valores de configuración de Hive (por ejemplo, `${hiveconf:inputtable}`, `${hiveconf:partitionedtable})`).

    Las propiedades **start** y **end** de la canalización especifican el período activo de esta canalización. Ha configurado el conjunto de datos para que se genere mensualmente y, por tanto, la canalización solo generará un segmento (ya que el mes es el mismo en la fecha de inicio y en la de finalización).

    En el código JSON de la actividad, debe indicar que el script de Hive se ejecuta en el proceso especificado por **linkedServiceName** – **HDInsightOnDemandLinkedService**.
4. Guarde el archivo **HiveActivity1.json** .

### <a name="add-partitionweblogshql-and-inputlog-as-a-dependency"></a>Adición de partitionweblogs.hql e input.log como una dependencia
1. En la ventana del **Explorador de soluciones**, haga clic con el botón derecho en **Dependencias**, seleccione **Agregar** y haga clic en **Elemento existente**.  
2. Acceda a **C:\ADFGettingStarted**, seleccione los archivos **partitionweblogs.hql** e **input.log** y haga clic en **Agregar**. Se crearon estos dos archivos como parte de los requisitos previos de la sección [Información general del tutorial](data-factory-build-your-first-pipeline.md).

Cuando publique la solución en el paso siguiente, el archivo **partitionweblogs.hql** se cargará en la carpeta de **script** del contenedor de blobs `adfgetstarted`.   

### <a name="publishdeploy-data-factory-entities"></a>Publicación e implementación de las entidades de Data Factory
En este paso, va a publicar las entidades de Data Factory (servicios vinculados, conjuntos de datos y canalización) del proyecto en el servicio Azure Data Factory. En el proceso de publicación, debe especificar el nombre de la instancia de Data Factory. 

1. Haga clic con el botón derecho en el proyecto en el Explorador de soluciones y haga clic en **Publicar**.
2. Si ve el cuadro de diálogo **Iniciar sesión en tu cuenta Microsoft**, escriba sus credenciales para la cuenta que tiene la suscripción de Azure y haga clic en **Iniciar sesión**.
3. Debería ver el siguiente cuadro de diálogo:

   ![Cuadro de diálogo Publicar](./media/data-factory-build-your-first-pipeline-using-vs/publish.png)
4. En la página **Configure data factory** (Configurar Data Factory), siga estos pasos:

    ![Publicar: nueva configuración de Data Factory](media/data-factory-build-your-first-pipeline-using-vs/publish-new-data-factory.png)

   1. Seleccione la opción **Crear la nueva factoría de datos** .
   2. Escriba un **nombre** único para la factoría de datos. Por ejemplo: **DataFactoryUsingVS09152016**. El nombre debe ser único globalmente.
   3. Seleccione la suscripción correcta en el campo **Suscripción** . 
        > [!IMPORTANT]
        > Si no ve ninguna suscripción, asegúrese de que ha iniciado sesión con una cuenta que sea administrador o coadministrador de la suscripción.
   4. Seleccione el **grupo de recursos** para la factoría de datos que se va a crear.
   5. Seleccione la **Región** de la factoría de datos.
   6. Haga clic en **Siguiente** para cambiar a la página **Publicar elementos**. (Si el botón **Siguiente** está deshabilitado, presione la tecla **TAB** para salir del campo).

      > [!IMPORTANT]
      > Si recibe el error **El nombre "DataFactoryUsingVS" de la factoría de datos no está disponible** al realizar la publicación, cambie el nombre (por ejemplo, a suNombreDataFactoryUsingVS). Consulte el tema [Factoría de datos: reglas de nomenclatura](data-factory-naming-rules.md) para las reglas de nomenclatura para los artefactos de Factoría de datos.   
1. En la página **Publicar elementos**, asegúrese de que todas las factorías de datos están seleccionadas y haga clic en **Siguiente** para cambiar a la página **Resumen**.

    ![Página Publicar elementos](media/data-factory-build-your-first-pipeline-using-vs/publish-items-page.png)     
2. Revise el resumen y haga clic en **Siguiente** para iniciar el proceso de implementación y ver el **Estado de implementación**.

    ![Página de resumen](media/data-factory-build-your-first-pipeline-using-vs/summary-page.png)
3. En la página **Estado de implementación** , debería ver el estado del proceso de implementación. Cuando se haya completado la implementación, haga clic en Finalizar.

Puntos importantes que hay que tener en cuenta:

- Si recibe el error: **La suscripción no está registrada para usar el espacio de nombres Microsoft.DataFactory**, realice una de las acciones siguientes e intente publicarla de nuevo:
    - En Azure PowerShell, ejecute el siguiente comando para registrar el proveedor de Data Factory.
        ```powershell   
        Register-AzResourceProvider -ProviderNamespace Microsoft.DataFactory
        ```
        Puede ejecutar el comando siguiente para confirmar que se ha registrado el proveedor de Data Factory.

        ```powershell
        Get-AzResourceProvider
        ```
    - Inicie sesión mediante la suscripción de Azure en el [Portal de Azure](https://portal.azure.com) y vaya a una hoja de Data Factory (o) cree una factoría de datos en el Portal de Azure. Esta acción registra automáticamente el proveedor.
- El nombre de la factoría de datos se puede registrar como un nombre DNS en el futuro y, por lo tanto, que sea visible públicamente.
- Para crear instancias de Data Factory, es preciso ser administrador o coadministrador de la suscripción de Azure

### <a name="monitor-pipeline"></a>Supervisión de la canalización
En este paso, va a supervisar la canalización mediante la vista de diagrama de Data Factory. 

#### <a name="monitor-pipeline-using-diagram-view"></a>Supervisión de una canalización desde la vista de diagrama
1. Inicie sesión en [Azure Portal](https://portal.azure.com/) y realice los siguientes pasos:
   1. Haga clic en **More services** (Más servicios) y en **Fábricas de datos**.
       
        ![Examinar factorías de datos](./media/data-factory-build-your-first-pipeline-using-vs/browse-datafactories.png)
   2. Seleccione el nombre de la factoría de datos (por ejemplo: **DataFactoryUsingVS09152016**) en la lista de factorías de datos.
   
       ![Seleccione su factoría de datos](./media/data-factory-build-your-first-pipeline-using-vs/select-first-data-factory.png)
2. En la página principal de la factoría de datos, haga clic en **Diagrama**.

    ![Icono Diagrama](./media/data-factory-build-your-first-pipeline-using-vs/diagram-tile.png)
3. En la Vista de diagrama, se ve información general de las canalizaciones y los conjuntos de datos empleados en este tutorial.

    ![Vista de diagrama](./media/data-factory-build-your-first-pipeline-using-vs/diagram-view-2.png)
4. Para ver todas las actividades de la canalización, haga clic con el botón derecho en la canalización en el diagrama y haga clic en Abrir canalización.

    ![Menú Abrir canalización](./media/data-factory-build-your-first-pipeline-using-vs/open-pipeline-menu.png)
5. Confirme que la actividad de HDInsightHive está en la canalización.

    ![Vista Abrir canalización](./media/data-factory-build-your-first-pipeline-using-vs/open-pipeline-view.png)

    Para volver a la vista anterior, haga clic en **Data Factory** en el menú de la ruta de navegación de la parte superior.
6. En la **Vista de diagrama**, haga doble clic en el conjunto de datos **AzureBlobInput**. Confirme que el estado del segmento es **Listo** . Puede que el segmento tarde un par de minutos en aparecer con ese estado. Si no aparece después de un tiempo, compruebe si el archivo de entrada (input.log) está en el contenedor (`adfgetstarted`) y en la carpeta (`inputdata`) correctos. Y asegúrese de que la propiedad **external** del conjunto de datos de entrada se establece en **true**. 

   ![Segmento de entrada con estado Listo](./media/data-factory-build-your-first-pipeline-using-vs/input-slice-ready.png)
7. Haga clic en el botón **X** para cerrar la hoja **AzureBlobInput**.
8. En la **Vista de diagrama**, haga doble clic en el conjunto de datos **AzureBlobOutput**. Se ve que el segmento se está procesando.

   ![Dataset](./media/data-factory-build-your-first-pipeline-using-vs/dataset-blade.png)
9. Cuando finalice el procesamiento, el segmento aparece con el estado **Listo** .

   > [!IMPORTANT]
   > La creación de un clúster de HDInsight a petición normalmente tarda algún tiempo (20 minutos aproximadamente). Por tanto, cabe esperar que la canalización tarde **aproximadamente 30 minutos** en procesar el segmento.  
   
    ![Dataset](./media/data-factory-build-your-first-pipeline-using-vs/dataset-slice-ready.png)    
10. Cuando el segmento se encuentre en el estado **Listo**, busque los datos de salida en la carpeta `partitioneddata` del contenedor `adfgetstarted` del almacenamiento de blobs.  

    ![datos de salida](./media/data-factory-build-your-first-pipeline-using-vs/three-ouptut-files.png)
11. Haga clic en el segmento para ver sus detalles en una hoja **Segmento de datos** .

    ![Detalles de segmento de datos](./media/data-factory-build-your-first-pipeline-using-vs/data-slice-details.png)  
12. Haga clic en una de las entradas de **Activity runs list** (Lista de ejecuciones de actividad) para ver los detalles de la ejecución de la actividad (en nuestro caso, la actividad de Hive) en la ventana **Detalles de la ejecución de actividad**. 
  
    ![Detalles de ejecución de actividad](./media/data-factory-build-your-first-pipeline-using-vs/activity-window-blade.png)    

    En los archivos de registro, puede ver la consulta de Hive que se ejecutó y la información sobre el estado. Estos registros son útiles para solucionar cualquier problema.  

Consulte [Supervisión de conjuntos de datos y canalizaciones](data-factory-monitor-manage-pipelines.md) para obtener instrucciones sobre cómo usar el Portal de Azure para supervisar la canalización y los conjuntos de datos creados en este tutorial.

#### <a name="monitor-pipeline-using-monitor--manage-app"></a>Supervisión de una canalización con la Aplicación de supervisión y administración
La Aplicación de supervisión y administración también se puede usar para supervisar las canalizaciones. Para más información acerca del uso de esta aplicación, consulte [Supervisión y administración de canalizaciones de Azure Data Factory mediante la nueva Aplicación de supervisión y administración](data-factory-monitor-manage-app.md).

1. Haga clic en el icono Supervisión y administración.

    ![Icono Supervisión y administración](./media/data-factory-build-your-first-pipeline-using-vs/monitor-and-manage-tile.png)
2. Debería ver la Aplicación de supervisión y administración. Cambie el valor de **Hora de inicio** y **Hora de finalización** para que coincidan con la hora de inicio (01-04-2016 12:00 a. m.) y la hora de finalización (02-04-2016 12:00 a. m.) de la canalización y haga clic en **Aplicar**.

    ![Aplicación de supervisión y administración](./media/data-factory-build-your-first-pipeline-using-vs/monitor-and-manage-app.png)
3. Para ver los detalles de una ventana de actividad, selecciónela en la **lista de ventanas de actividad**.
    ![Detalles de ventana de actividad](./media/data-factory-build-your-first-pipeline-using-vs/activity-window-details.png)

> [!IMPORTANT]
> El archivo de entrada se elimina cuando el segmento se procesa correctamente. Por lo tanto, si desea volver a ejecutar el segmento o volver a realizar el tutorial, cargue el archivo de entrada (input.log) en la carpeta `inputdata` del contenedor `adfgetstarted`.

### <a name="additional-notes"></a>Notas adicionales
- Una factoría de datos puede tener una o más canalizaciones. Una canalización puede tener una o más actividades. Por ejemplo, una actividad de copia para copiar datos desde un origen hasta un almacén de datos de destino o una actividad de Hive de HDInsight para ejecutar un script de Hive que transforme los datos de entrada. Consulte los [almacenes de datos compatibles](data-factory-data-movement-activities.md#supported-data-stores-and-formats) para ver todos orígenes y receptores que admite la actividad de copia. Consulte los [servicios vinculados de procesos](data-factory-compute-linked-services.md) para ver la lista de servicios de proceso compatibles con Data Factory.
- Los servicios vinculados vinculan almacenes de datos o servicios de proceso con una factoría de datos de Azure. Consulte los [almacenes de datos compatibles](data-factory-data-movement-activities.md#supported-data-stores-and-formats) para ver todos orígenes y receptores que admite la actividad de copia. Consulte los [servicios vinculados de procesos](data-factory-compute-linked-services.md) para ver la lista de servicios de proceso compatibles con Data Factory y las [actividades de transformación](data-factory-data-transformation-activities.md) que se pueden ejecutar en ellos.
- Consulte [Movimiento de datos hacia y desde Azure Blob mediante Azure Data Factory](data-factory-azure-blob-connector.md#azure-storage-linked-service) para más información acerca de las propiedades JSON usadas en la definición del servicio vinculado de Azure Storage.
- Puede usar su propio clúster de HDInsight en lugar de usar un clúster de HDInsight a petición. Consulte [Servicios vinculados de procesos](data-factory-compute-linked-services.md) para más información.
-  Data Factory crea un clúster de HDInsight **basado en Linux** con el código JSON anterior. Para más información, consulte la sección [Servicio vinculado a petición de HDInsight de Azure](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) .
- El clúster de HDInsight crea un **contenedor predeterminado** en el almacenamiento de blobs que especificó en JSON (linkedServiceName). HDInsight no elimina este contenedor cuando se elimina el clúster. Este comportamiento es así por diseño. Con el servicio vinculado de HDInsight a petición se crea un clúster de HDInsight cada vez que se procesa un segmento, a menos que haya un clúster existente activo (timeToLive). El clúster se elimina automáticamente cuando se realiza el procesamiento.
    
    A medida que se procesen más segmentos, verá numerosos contenedores en su Almacenamiento de blobs de Azure. Si no los necesita para solucionar problemas de trabajos, puede eliminarlos para reducir el costo de almacenamiento. Los nombres de estos contenedores siguen un patrón: `adf**yourdatafactoryname**-**linkedservicename**-datetimestamp`. Use herramientas como el [Explorador de Microsoft Azure Storage](https://storageexplorer.com/) para eliminar contenedores de Azure Blob Storage.
- Actualmente, el conjunto de datos de salida es lo que controla la programación, por lo que debe crear un conjunto de datos de salida aunque la actividad no genere ninguna salida. Si la actividad no toma ninguna entrada, puede omitir la creación del conjunto de datos de entrada. 
- Este tutorial no muestra cómo copiar datos mediante Azure Data Factory. Para ver un tutorial acerca de cómo copiar datos mediante Azure Data Factory, consulte [Copia de datos de Blob Storage en SQL Database mediante Data Factory](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).


## <a name="use-server-explorer-to-view-data-factories"></a>Uso del Explorador de servidores para ver las entidades de Data Factory
1. En **Visual Studio** haga clic en **Vista** en el menú y haga clic en **Explorador de servidores**.
2. En la ventana Explorador de servidores, expanda **Azure** y **Factoría de datos**. Si aparece **Iniciar sesión en Visual Studio**, escriba la **cuenta** asociada a su suscripción de Azure y haga clic en **Continuar**. Escriba la **contraseña** y haga clic en **Iniciar sesión**. Visual Studio intenta obtener información acerca de todas las factorías de datos de Azure de su suscripción. Puede ver el estado de esta operación en la ventana **Data Factory Task List** (Lista de tareas de Data Factory).

    ![Explorador de servidores](./media/data-factory-build-your-first-pipeline-using-vs/server-explorer.png)
3. Puede hacer clic con el botón derecho en una factoría de datos y seleccionar **Export Data Factory to New Project** (Exportar Data Factory al nuevo proyecto) para crear un proyecto de Visual Studio basado en una factoría de datos existente.

    ![Exportación de la factoría de datos](./media/data-factory-build-your-first-pipeline-using-vs/export-data-factory-menu.png)

## <a name="update-data-factory-tools-for-visual-studio"></a>Actualización de herramientas de Factoría de datos para Visual Studio
Para actualizar las herramientas de Azure Data Factory para Visual Studio, siga estos pasos:

1. Haga clic en el menú **Herramientas** y seleccione **Extensiones y actualizaciones**.
2. Seleccione **Actualizaciones** en el panel izquierdo y, luego, seleccione **Galería de Visual Studio**.
3. Seleccione **Herramientas de Azure Data Factory para Visual Studio** y haga clic en **Actualizar**. Si no ve esta entrada, ya tiene la versión más reciente de las herramientas.

## <a name="use-configuration-files"></a>Uso de archivos de configuración
Puede usar archivos de configuración de Visual Studio para configurar propiedades para servicios vinculados/tablas/canalizaciones distintos para cada entorno.

Considere la siguiente definición de JSON para un servicio vinculado de Azure Storage. Con ella, podrá especificar la propiedad **connectionString** con distintos valores para accountname y accountkey basados en el entorno (desarrollo, pruebas y producción) en el que va a implementar las entidades de Data Factory. Puede conseguir este comportamiento mediante el uso del archivo de configuración independiente para cada entorno.

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

### <a name="add-a-configuration-file"></a>Adición de un archivo de configuración
Agregue un archivo de configuración a cada entorno realizando los pasos siguientes:   

1. En la solución de Visual Studio, haga clic con el botón derecho en el proyecto de Data Factory, seleccione **Agregar** y haga clic en **Nuevo elemento**.
2. Seleccione **Config** en la lista de plantillas instaladas que encontrará a la izquierda, elija **Archivo de configuración**, especifique un **nombre** para este archivo y haga clic en **Agregar**.

    ![Adición de archivo de configuración](./media/data-factory-build-your-first-pipeline-using-vs/add-config-file.png)
3. Agregue parámetros de configuración y sus valores en el formato siguiente:

    ```json
    {
        "$schema": "http://datafactories.schema.management.azure.com/vsschemas/V1/Microsoft.DataFactory.Config.json",
        "AzureStorageLinkedService1": [
            {
                "name": "$.properties.typeProperties.connectionString",
                "value": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
            }
        ],
        "AzureSqlLinkedService1": [
            {
                "name": "$.properties.typeProperties.connectionString",
                "value":  "Server=tcp:<logical SQL server name>.database.windows.net,1433;Database=<Azure Sql database>;User ID=<user name>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
            }
        ]
    }
    ```

    En este ejemplo se configura la propiedad connectionString de un servicio vinculado de Azure Storage y un servicio vinculado de SQL de Azure. Observe que la sintaxis para especificar el nombre es [JsonPath](https://goessner.net/articles/JsonPath/).   

    Si JSON tiene una propiedad con una matriz de valores como se muestra en el código siguiente:  

    ```json
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
    ```

    Configure las propiedades tal como se muestra en el siguiente archivo de configuración (use la indexación de base cero):

    ```json
    {
        "name": "$.properties.structure[0].name",
        "value": "FirstName"
    }
    {
        "name": "$.properties.structure[0].type",
        "value": "String"
    }
    {
        "name": "$.properties.structure[1].name",
        "value": "LastName"
    }
    {
        "name": "$.properties.structure[1].type",
        "value": "String"
    }
    ```

### <a name="property-names-with-spaces"></a>Nombres de propiedad con espacios
Si un nombre de propiedad contiene espacios, utilice corchetes como se muestra en el ejemplo siguiente (nombre de servidor de base de datos):

```json
 {
     "name": "$.properties.activities[1].typeProperties.webServiceParameters.['Database server name']",
     "value": "MyAsqlServer.database.windows.net"
 }
```

### <a name="deploy-solution-using-a-configuration"></a>Implementación de la solución mediante una configuración
Al publicar las entidades de Azure Data Factory en Visual Studio, puede especificar la configuración que desea usar para esa operación de publicación.

Para publicar entidades en un proyecto de Azure Data Factory mediante el archivo de configuración:   

1. Haga clic con el botón derecho en el proyecto de Data Factory y después haga clic en **Publicar** para abrir el cuadro de diálogo **Publish Items** (Publicar elementos).
2. Seleccione una factoría de datos existente o especifique los valores necesarios para crear una en la página **Configure data factory** (Configurar Data Factory) y haga clic en **Siguiente**.   
3. En la página **Publish Items** (Publicar elementos), verá una lista desplegable con las configuraciones disponibles para el campo **Select Deployment Config** (Seleccionar configuración de implementación).

    ![Selección de archivo de configuración](./media/data-factory-build-your-first-pipeline-using-vs/select-config-file.png)
4. Seleccione el **archivo de configuración** que desea usar y haga clic en **Siguiente**.
5. Confirme que ve el nombre del archivo JSON en la página **Resumen** y haga clic en **Siguiente**.
6. Cuando finalice la operación de implementación, haga clic en **Finalizar** .

Cuando realiza la implementación, se usan los valores del archivo de configuración para establecer los valores de las propiedades de los archivos JSON antes de que las entidades se implementen en el servicio Azure Data Factory.   

## <a name="use-azure-key-vault"></a>Uso de Azure Key Vault
No es aconsejable y, en ocasiones, contraviene la directiva de seguridad, confirmar información confidencial, como las cadenas de conexión, en el repositorio de código. Vea el ejemplo [ADF Secure Publish](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/ADFSecurePublish) en GitHub para obtener información sobre cómo almacenar información confidencial en Azure Key Vault y usarla al publicar entidades de Data Factory. La extensión Secure Publish de Visual Studio permite almacenar los secretos en Key Vault, y solo las referencias a ellos se especifican en las configuraciones de implementación o en los servicios vinculados. Estas referencias se resuelven al publicar las entidades de Data Factory en Azure. Estos archivos se pueden confirmar en el repositorio de origen sin exponer los secretos.

## <a name="summary"></a>Resumen
En este tutorial, ha creado una instancia de Data Factory de Azure para procesar datos mediante la ejecución de un script de Hive en un clúster de Hadoop en HDInsight. Ha usado el Editor de Data Factory en el Portal de Azure para realizar los siguientes pasos:  

1. Ha creado una **factoría de datos**de Azure.
2. Ha creado dos **servicios vinculados**:
   1. **Azure Storage** para vincular la instancia de Azure Blob Storage que contiene los archivos de entrada y salida a la factoría de datos.
   2. **HDInsight de Azure** para vincular un clúster de Hadoop en HDInsight a petición a la factoría de datos. Azure Data Factory crea un clúster de Hadoop en HDInsight justo a tiempo para procesar los datos de entrada y generar los datos de salida.
3. Ha creado dos **conjuntos de datos**, que describen los datos de entrada y salida para la actividad de Hive de HDInsight en la canalización.
4. Ha creado una **canalización** con una actividad de **Hive de HDInsight**.  

## <a name="next-steps"></a>Pasos siguientes
En este artículo, creó una canalización con una actividad de transformación (actividad de HDInsight) que ejecuta un script de Hive en un clúster de HDInsight a petición. Para ver cómo se usa una actividad de copia para copiar datos de un blob de Azure en SQL Azure, consulte [Tutorial: Copia de datos de un blob de Azure a SQL Azure](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

Puede encadenar dos actividades (ejecutar una después de otra) haciendo que el conjunto de datos de salida de una actividad sea el conjunto de datos de entrada de la otra actividad. Para más información, consulte [Programación y ejecución en Data Factory](data-factory-scheduling-and-execution.md). 


## <a name="see-also"></a>Consulte también

| Tema | Descripción |
|:--- |:--- |
| [Canalizaciones](data-factory-create-pipelines.md) |Este artículo ayuda a conocer las canalizaciones y actividades de Azure Data Factory y cómo aprovecharlas para construir flujos de trabajo controlados por datos para su escenario o negocio. |
| [Conjuntos de datos](data-factory-create-datasets.md) |Este artículo le ayuda a comprender los conjuntos de datos de Azure Data Factory. |
| [Transformación y análisis con Data Factory de Azure](data-factory-data-transformation-activities.md) |En este artículo se proporciona una lista de actividades de transformación de datos (por ejemplo, la transformación de Hive para HDInsight usada en este tutorial), que se admiten en Azure Data Factory. |
| [Programación y ejecución con Data Factory](data-factory-scheduling-and-execution.md) |En este artículo se explican los aspectos de programación y ejecución del modelo de aplicación de Azure Data Factory. |
| [Supervisión y administración de canalizaciones de Data Factory de Azure mediante la nueva Aplicación de supervisión y administración](data-factory-monitor-manage-app.md) |En este artículo se describe cómo supervisar, administrar y depurar las canalizaciones mediante la aplicación de supervisión y administración. |
