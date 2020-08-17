---
title: Compilación de la primera factoría de datos (REST)
description: En este tutorial se crea un ejemplo de canalización de Azure Data Factory con la API de REST de Data Factory.
services: data-factory
documentationcenter: ''
author: djpmsft
ms.author: daperlov
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.date: 11/01/2017
ms.openlocfilehash: 32705c37685ab03ffa68f805dedb64411d8d9c46
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543136"
---
# <a name="tutorial-build-your-first-azure-data-factory-using-data-factory-rest-api"></a>Tutorial: Compilación de la primera instancia de Azure Data Factory con la API de REST de Data Factory
> [!div class="op_single_selector"]
> * [Introducción y requisitos previos](data-factory-build-your-first-pipeline.md)
> * [Visual Studio](data-factory-build-your-first-pipeline-using-vs.md)
> * [PowerShell](data-factory-build-your-first-pipeline-using-powershell.md)
> * [Plantilla de Resource Manager](data-factory-build-your-first-pipeline-using-arm.md)
> * [REST API](data-factory-build-your-first-pipeline-using-rest-api.md)
>
>


> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si usa la versión actual del servicio Data Factory, consulte [Guía de inicio rápido: Creación de una factoría de datos mediante Data Factory](../quickstart-create-data-factory-rest-api.md).

En este artículo usará la API de REST de Data Factory para crear su primera instancia de Azure Data Factory. Para realizar el tutorial con otros SDK/herramientas, seleccione una de las opciones de la lista desplegable.

La canalización de este tutorial tiene una actividad: **actividad de HDInsight Hive**. Esta actividad ejecuta un script de Hive en un clúster de Azure HDInsight que transforma los datos de entrada para generar datos de salida. La canalización está programada para ejecutarse una vez al mes entre las horas de inicio y finalización especificadas.

> [!NOTE]
> Este artículo no abarca toda la API de REST. Para obtener documentación completa sobre la API de REST, consulte la [referencia de la API de REST de Data Factory](/rest/api/datafactory/).
> 
> pero cualquier canalización puede tener más de una actividad. También puede encadenar dos actividades (ejecutar una después de otra) haciendo que el conjunto de datos de salida de una actividad sea el conjunto de datos de entrada de la otra actividad. Para más información, consulte [Programación y ejecución en Data Factory](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline).


## <a name="prerequisites"></a>Prerequisites

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

* Lea el artículo [Tutorial: Compilación de la primera canalización para procesar datos mediante el clúster de Hadoop](data-factory-build-your-first-pipeline.md) y complete los pasos de los **requisitos previos** .
* Instale [Curl](https://curl.haxx.se/dlwiz/) en su máquina. Utilice la herramienta CURL con comandos de REST para crear una factoría de datos.
* Siga las instrucciones de [este artículo](../../active-directory/develop/howto-create-service-principal-portal.md) para:
  1. Cree una aplicación web denominada **ADFGetStartedApp** en Azure Active Directory.
  2. Obtenga el **Id. de cliente** y la **Clave secreta**.
  3. Obtenga el **Identificador de inquilino**.
  4. Asigne la aplicación **ADFGetStartedApp** al rol **Colaborador de Data Factory**.
* Instale [Azure PowerShell](/powershell/azure/).
* Inicie **PowerShell** y ejecute el comando siguiente. Mantenga Azure PowerShell abierto hasta el final de este tutorial. Si lo cierra y vuelve a abrirlo, deberá ejecutar los comandos de nuevo.
  1. Ejecute **Connect-AzAccount** y escriba el mismo nombre de usuario y contraseña que utiliza para iniciar sesión en Azure Portal.
  2. Ejecute **Get-AzSubscriptionn** para ver todas las suscripciones para esta cuenta.
  3. Ejecute **Get-AzSubscription -SubscriptionName NameOfAzureSubscription | Set-AzContext** para seleccionar la suscripción con la que desea trabajar. Reemplace **NameOfAzureSubscription** por el nombre de su suscripción de Azure.
* Cree un grupo de recursos de Azure denominado **ADFTutorialResourceGroup** , para lo que debe ejecutar el siguiente comando en PowerShell:

    ```powershell
    New-AzResourceGroup -Name ADFTutorialResourceGroup  -Location "West US"
    ```

   En algunos de los pasos de este tutorial se supone que se usa el grupo de recursos denominado ADFTutorialResourceGroup. Si usa un otro grupo de recursos, deberá usar su nombre en lugar de ADFTutorialResourceGroup en este tutorial.

## <a name="create-json-definitions"></a>Creación de definiciones de JSON
Cree los siguientes archivos JSON en la carpeta en la que se encuentra curl.exe.

### <a name="datafactoryjson"></a>datafactory.json
> [!IMPORTANT]
> El nombre debe ser único globalmente, por lo que quizás desee usar el prefijo/sufijo ADFCopyTutorialDF para que sea un nombre único.
>
>

```JSON
{
    "name": "FirstDataFactoryREST",
    "location": "WestUS"
}
```

### <a name="azurestoragelinkedservicejson"></a>azurestoragelinkedservice.json
> [!IMPORTANT]
> Reemplace **accountname** y **accountkey** por el nombre y la clave de su cuenta de almacenamiento de Azure. Para aprender a obtener la clave de acceso de almacenamiento, consulte [Administración de claves de acceso de la cuenta de almacenamiento](../../storage/common/storage-account-keys-manage.md).
>
>

```JSON
{
    "name": "AzureStorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
    }
}
```

### <a name="hdinsightondemandlinkedservicejson"></a>hdinsightondemandlinkedservice.json

```JSON
{
    "name": "HDInsightOnDemandLinkedService",
    "properties": {
        "type": "HDInsightOnDemand",
        "typeProperties": {
            "version": "3.5",
            "clusterSize": 1,
            "timeToLive": "00:05:00",
            "osType": "Linux",
            "linkedServiceName": "AzureStorageLinkedService"
        }
    }
}
```

En la siguiente tabla se ofrecen descripciones de las propiedades JSON que se usan en el fragmento de código:

| Propiedad | Descripción |
|:--- |:--- |
| clusterSize |Tamaño del clúster de HDInsight. |
| timeToLive |Especifica el tiempo de inactividad del clúster de HDInsight, antes de que se elimine. |
| linkedServiceName |Especifica la cuenta de almacenamiento que se usa para almacenar los registros que genera HDInsight. |

Tenga en cuenta los siguientes puntos:

* Data Factory crea un clúster de HDInsight **basado en Linux** con el código JSON anterior. Para más información, consulte la sección [Servicio vinculado a petición de HDInsight de Azure](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) .
* Puede usar **su propio clúster de HDInsight** en lugar de usar un clúster de HDInsight a petición. Para más información, consulte [Servicio vinculado de HDInsight de Azure](data-factory-compute-linked-services.md#azure-hdinsight-linked-service) .
* El clúster de HDInsight crea un **contenedor predeterminado** en el almacenamiento de blobs que especificó en JSON (**linkedServiceName**). HDInsight no elimina este contenedor cuando se elimina el clúster. Este comportamiento es así por diseño. Con el servicio vinculado de HDInsight a petición se crea un clúster de HDInsight cada vez se procesa un segmento, a menos que haya un clúster existente activo (**timeToLive**), que se elimina cuando finaliza el procesamiento.

    A medida que se procesen más segmentos, verá numerosos contenedores en su Almacenamiento de blobs de Azure. Si no los necesita para solucionar problemas de trabajos, puede eliminarlos para reducir el costo de almacenamiento. Los nombres de estos contenedores siguen este patrón: "adf**nombredefactoría dedatos**-**nombredelserviciovinculado**-marcadefechayhora". Use herramientas como el [Explorador de Microsoft Azure Storage](https://storageexplorer.com/) para eliminar contenedores de Azure Blob Storage.

Para más información, consulte la sección [Servicio vinculado a petición de HDInsight de Azure](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) .

### <a name="inputdatasetjson"></a>inputdataset.json

```JSON
{
    "name": "AzureBlobInput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
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

### <a name="outputdatasetjson"></a>outputdataset.json

```JSON
{
    "name": "AzureBlobOutput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
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

### <a name="pipelinejson"></a>pipeline.json
> [!IMPORTANT]
> Reemplace **storageaccountname** por el nombre de la cuenta de Almacenamiento de Azure.
>
>

```JSON
{
    "name": "MyFirstPipeline",
    "properties": {
        "description": "My first Azure Data Factory pipeline",
        "activities": [{
            "type": "HDInsightHive",
            "typeProperties": {
                "scriptPath": "adfgetstarted/script/partitionweblogs.hql",
                "scriptLinkedService": "AzureStorageLinkedService",
                "defines": {
                    "inputtable": "wasb://adfgetstarted@<storageaccountname>.blob.core.windows.net/inputdata",
                    "partitionedtable": "wasb://adfgetstarted@<storageaccountname>t.blob.core.windows.net/partitioneddata"
                }
            },
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
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
        }],
        "start": "2017-07-10T00:00:00Z",
        "end": "2017-07-11T00:00:00Z",
        "isPaused": false
    }
}
```

En este fragmento de código JSON, se crea una canalización formada por una sola actividad que usa Hive para procesar los datos en un clúster de HDInsight.

El archivo de script de Hive, **partitionweblogs.hql**, se almacena en la cuenta de Azure Storage (especificada mediante la propiedad scriptLinkedService, llamada **StorageLinkedService**) en una carpeta denominada **script** del contenedor **adfgetstarted**.

La sección **defines** especifica la configuración de tiempo de ejecución que se pasa al script de Hive como valores de configuración de Hive (por ejemplo, ${hiveconf:inputtable}, ${hiveconf:partitionedtable}).

Las propiedades **start** y **end** de la canalización especifican el período activo de esta canalización.

En el código JSON de la actividad, debe indicar que el script de Hive se ejecuta en el proceso especificado por **linkedServiceName** – **HDInsightOnDemandLinkedService**.

> [!NOTE]
> Consulte la sección "JSON de canalización" en [Canalizaciones y actividades en Azure Data Factory](data-factory-create-pipelines.md) para más información acerca de las propiedades de JSON utilizadas en el ejemplo anterior.
>
>

## <a name="set-global-variables"></a>Definición de variables globales
En Azure PowerShell, ejecute los comandos siguientes después de reemplazar los valores por los suyos propios:

> [!IMPORTANT]
> Consulte la sección [Requisitos previos](#prerequisites) para obtener instrucciones sobre cómo obtener el identificador de cliente, el secreto del cliente, el identificador de inquilino y el identificador de suscripción.
>
>

```powershell
$client_id = "<client ID of application in AAD>"
$client_secret = "<client key of application in AAD>"
$tenant = "<Azure tenant ID>";
$subscription_id="<Azure subscription ID>";

$rg = "ADFTutorialResourceGroup"
$adf = "FirstDataFactoryREST"
```


## <a name="authenticate-with-aad"></a>Autenticación con AAD

```powershell
$cmd = { .\curl.exe -X POST https://login.microsoftonline.com/$tenant/oauth2/token  -F grant_type=client_credentials  -F resource=https://management.core.windows.net/ -F client_id=$client_id -F client_secret=$client_secret };
$responseToken = Invoke-Command -scriptblock $cmd;
$accessToken = (ConvertFrom-Json $responseToken).access_token;

(ConvertFrom-Json $responseToken)
```


## <a name="create-data-factory"></a>Creación de Data Factory
En este paso, creará una instancia de Azure Data Factory llamada **LogProcessingFactory**. Una factoría de datos puede tener una o más canalizaciones. Una canalización puede tener una o más actividades. Por ejemplo, una actividad de copia para copiar datos desde un origen a un almacén de datos de destino o una actividad de Hive de HDInsight para ejecutar un script de Hive que transforme los datos de entrada. Ejecute los siguientes comandos para crear la factoría de datos:

1. Asigne el comando a la variable denominada **cmd**.

    Confirme que el nombre de la factoría de datos que especifique aquí (ADFCopyTutorialDF) coincide con el nombre especificado en el archivo **datafactory.json**.

    ```powershell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@datafactory.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/FirstDataFactoryREST?api-version=2015-10-01};
    ```
2. Ejecute el comando con **Invoke-Command**.

    ```powershell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Vea los resultados. Si la factoría de datos se ha creado correctamente, verá su JSON en los **resultados**; de lo contrario, verá un mensaje de error.

    ```powershell
    Write-Host $results
    ```

Tenga en cuenta los siguientes puntos:

* El nombre de Azure Data Factory debe ser único de forma global. Si ve el siguiente error en los resultados: **El nombre “ADFCopyTutorialDF” de factoría de datos no está disponible**, siga estos pasos:
  1. Cambie el nombre (por ejemplo, yournameFirstDataFactoryREST) en el archivo **datafactory.json** . Consulte el tema [Factoría de datos: reglas de nomenclatura](data-factory-naming-rules.md) para las reglas de nomenclatura para los artefactos de Factoría de datos.
  2. En el primer comando, donde se asigna un valor a la variable **$cmd** , reemplace FirstDataFactoryREST por el nuevo nombre y ejecute el comando.
  3. Ejecute los dos comandos siguientes para invocar la API de REST para crear la factoría de datos e imprimir los resultados de la operación.
* Para crear instancias de Data Factory, debe ser administrador o colaborador de la suscripción de Azure.
* El nombre de la factoría de datos se puede registrar como un nombre DNS en el futuro y, por lo tanto, que sea visible públicamente.
* Si recibe el error: "**La suscripción no está registrada para usar el espacio de nombres Microsoft.DataFactory**", realice una de las acciones siguientes e intente publicarla de nuevo:

  * En Azure PowerShell, ejecute el siguiente comando para registrar el proveedor de Data Factory:

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DataFactory
    ```

      Puede ejecutar el comando siguiente para confirmar que se ha registrado el proveedor de Data Factory:
    ```powershell
    Get-AzResourceProvider
    ```
  * Inicie sesión mediante la suscripción de Azure en el [Portal de Azure](https://portal.azure.com) y vaya a una hoja de Data Factory (o) cree una factoría de datos en el Portal de Azure. Esta acción registra automáticamente el proveedor.

Antes de crear una canalización, debe crear algunas entidades de factoría de datos primero. En primer lugar debe crear servicios vinculados para vincular los almacenes de datos y procesos con su almacén de datos, y, después, definir los conjuntos de datos de entrada y salida para representar los datos en los almacenes de datos vinculados.

## <a name="create-linked-services"></a>Crear servicios vinculados
En este paso, vinculará su cuenta de Azure Storage y el clúster de Azure HDInsight a petición con su factoría de datos. La cuenta de Azure Storage contiene los datos de entrada y salida de la canalización de este ejemplo. El servicio vinculado de HDInsight se usa para ejecutar un script de Hive especificado en la actividad de la canalización en este ejemplo.

### <a name="create-azure-storage-linked-service"></a>Creación de un servicio vinculado de Azure Storage
En este paso, vinculará su cuenta de Azure Storage con su factoría de datos. Con este tutorial, usará la misma cuenta de Azure Storage para almacenar los datos de entrada y salida y el archivo de script de HQL.

1. Asigne el comando a la variable denominada **cmd**.

    ```powershell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data “@azurestoragelinkedservice.json” https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/linkedservices/AzureStorageLinkedService?api-version=2015-10-01};
    ```
2. Ejecute el comando con **Invoke-Command**.

    ```powershell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Vea los resultados. Si el servicio vinculado se ha creado correctamente, verá su JSON en los **resultados**; de lo contrario, verá un mensaje de error.

    ```powershell
    Write-Host $results
    ```

### <a name="create-azure-hdinsight-linked-service"></a>Creación de un servicio vinculado de HDInsight de Azure
En este paso, vinculará un clúster de HDInsight a petición con la factoría de datos. El clúster de HDInsight se crea automáticamente en tiempo de ejecución y se elimina después de hacer el procesamiento y de permanecer inactivo durante el período de tiempo especificado. Puede usar su propio clúster de HDInsight en lugar de usar un clúster de HDInsight a petición. Consulte [Servicios vinculados de procesos](data-factory-compute-linked-services.md) para más información.

1. Asigne el comando a la variable denominada **cmd**.

    ```powershell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@hdinsightondemandlinkedservice.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/linkedservices/hdinsightondemandlinkedservice?api-version=2015-10-01};
    ```
2. Ejecute el comando con **Invoke-Command**.

    ```powershell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Vea los resultados. Si el servicio vinculado se ha creado correctamente, verá su JSON en los **resultados**; de lo contrario, verá un mensaje de error.

    ```powershell
    Write-Host $results
    ```

## <a name="create-datasets"></a>Creación de conjuntos de datos
En este paso, creará conjuntos de datos que representan los datos de entrada y salida para el procesamiento de Hive. Estos conjuntos de datos hacen referencia al servicio **StorageLinkedService** que ha creado anteriormente en este tutorial. El servicio vinculado apunta a una cuenta de Azure Storage y los conjuntos de datos especifican el contenedor, la carpeta y el nombre de archivo en el almacenamiento que contiene los datos de entrada y salida.

### <a name="create-input-dataset"></a>Creación de un conjunto de datos de entrada
En este paso, creará el conjunto de datos de entrada que representa los datos de entrada almacenados en Almacenamiento de blobs de Azure.

1. Asigne el comando a la variable denominada **cmd**.

    ```powershell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@inputdataset.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datasets/AzureBlobInput?api-version=2015-10-01};
    ```
2. Ejecute el comando con **Invoke-Command**.

    ```powershell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Vea los resultados. Si el conjunto de datos se ha creado correctamente, verá su JSON en los **resultados**; de lo contrario, verá un mensaje de error.

    ```powershell
    Write-Host $results
    ```

### <a name="create-output-dataset"></a>Creación del conjunto de datos de salida
En este paso, creará el conjunto de datos de salida que representa los datos de salida almacenados en Almacenamiento de blobs de Azure.

1. Asigne el comando a la variable denominada **cmd**.

    ```powershell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@outputdataset.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datasets/AzureBlobOutput?api-version=2015-10-01};
    ```
2. Ejecute el comando con **Invoke-Command**.

    ```powershell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Vea los resultados. Si el conjunto de datos se ha creado correctamente, verá su JSON en los **resultados**; de lo contrario, verá un mensaje de error.

    ```powershell
    Write-Host $results
    ```

## <a name="create-pipeline"></a>Creación de una canalización
En este paso, creará la primera canalización con una actividad **HDInsightHive** . El segmento de entrada está disponible mensualmente (frecuencia: mensual, intervalo: 1), el segmento de salida se genera mensualmente y la propiedad de programador de la actividad también se establece en mensual. La configuración del conjunto de datos de salida y la del programador de la actividad deben coincidir. Actualmente, el conjunto de datos de salida es lo que controla la programación, por lo que debe crear un conjunto de datos de salida aunque la actividad no genere ninguna salida. Si la actividad no toma ninguna entrada, puede omitir la creación del conjunto de datos de entrada.

Confirme que el archivo **input.log** aparece en la carpeta **adfgetstarted/inputdata** en Azure Blob Storage y ejecute el siguiente comando para implementar la canalización. Como las fechas de **inicio** y **finalización** están establecidas en el pasado e **isPaused** está establecido en false, la canalización (la actividad de la canalización) se ejecuta nada más realizar la implementación.

1. Asigne el comando a la variable denominada **cmd**.

    ```powershell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@pipeline.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datapipelines/MyFirstPipeline?api-version=2015-10-01};
    ```
2. Ejecute el comando con **Invoke-Command**.

    ```powershell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Vea los resultados. Si el conjunto de datos se ha creado correctamente, verá su JSON en los **resultados**; de lo contrario, verá un mensaje de error.

    ```powershell
    Write-Host $results
    ```
4. Enhorabuena, ya creó correctamente su primera canalización con Azure PowerShell.

## <a name="monitor-pipeline"></a>Supervisión de la canalización
En este paso, utilizará la API de REST de Data Factory para supervisar los segmentos que la canalización está produciendo.

```powershell
$ds ="AzureBlobOutput"

$cmd = {.\curl.exe -X GET -H "Authorization: Bearer $accessToken" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datasets/$ds/slices?start=1970-01-01T00%3a00%3a00.0000000Z"&"end=2016-08-12T00%3a00%3a00.0000000Z"&"api-version=2015-10-01};

$results2 = Invoke-Command -scriptblock $cmd;

IF ((ConvertFrom-Json $results2).value -ne $NULL) {
    ConvertFrom-Json $results2 | Select-Object -Expand value | Format-Table
} else {
        (convertFrom-Json $results2).RemoteException
}
```

> [!IMPORTANT]
> La creación de un clúster de HDInsight a petición normalmente tarda algún tiempo (20 minutos aproximadamente). Por tanto, cabe esperar que la canalización tarde **aproximadamente 30 minutos** en procesar el segmento.
>
>

Ejecute Invoke-Command y el siguiente hasta que vea el segmento en estado **Listo** o **Error**. Cuando el segmento tenga el estado Listo, busque los datos de salida en la carpeta **partitioneddata** del contenedor **adfgetstarted** del almacenamiento de blobs.  La creación de un clúster de HDInsight a petición normalmente requiere algo de tiempo.

![datos de salida](./media/data-factory-build-your-first-pipeline-using-rest-api/three-ouptut-files.png)

> [!IMPORTANT]
> El archivo de entrada se elimina cuando el segmento se procesa correctamente. Por lo tanto, si desea volver a ejecutar el segmento o volver a realizar el tutorial, cargue el archivo de entrada (input.log) en la carpeta inputdata del contenedor adfgetstarted.
>
>

También puede usar el Portal de Azure para supervisar los segmentos y solucionar cualquier problema. Consulte más detalles en la sección sobre la [supervisión de canalizaciones con el Portal de Azure](data-factory-monitor-manage-pipelines.md) .

## <a name="summary"></a>Resumen
En este tutorial, ha creado una instancia de Data Factory de Azure para procesar datos mediante la ejecución de un script de Hive en un clúster de Hadoop en HDInsight. Ha usado el Editor de Data Factory en el Portal de Azure para realizar los siguientes pasos:

1. Ha creado una **factoría de datos**de Azure.
2. Ha creado dos **servicios vinculados**:
   1. **Azure Storage** para vincular la instancia de Azure Blob Storage que contiene los archivos de entrada y salida a la factoría de datos.
   2. **HDInsight de Azure** para vincular un clúster de Hadoop en HDInsight a petición a la factoría de datos. Azure Data Factory crea un clúster de Hadoop en HDInsight justo a tiempo para procesar los datos de entrada y generar los datos de salida.
3. Ha creado dos **conjuntos de datos**, que describen los datos de entrada y salida para la actividad de Hive de HDInsight en la canalización.
4. Ha creado una **canalización** con una actividad de **Hive de HDInsight**.

## <a name="next-steps"></a>Pasos siguientes
En este artículo, creó una canalización con una actividad de transformación (actividad de HDInsight) que ejecuta un script de Hive en un clúster de HDInsight de Azure a petición. Para ver cómo se usa una actividad de copia para copiar datos de un blob de Azure en SQL Azure, consulte [Tutorial: Copia de datos de un blob de Azure a SQL Azure](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

## <a name="see-also"></a>Consulte también
| Tema | Descripción |
|:--- |:--- |
| [Referencia de API de REST de Data Factory](/rest/api/datafactory/) |Consulte la documentación completa sobre los cmdlets de Factoría de datos |
| [Canalizaciones](data-factory-create-pipelines.md) |Este artículo ayuda a conocer las canalizaciones y actividades de Azure Data Factory y cómo aprovecharlas para construir flujos de trabajo controlados por datos de un extremo a otro para su escenario o negocio. |
| [Conjuntos de datos](data-factory-create-datasets.md) |Este artículo le ayuda a comprender los conjuntos de datos de Azure Data Factory. |
| [Programación y ejecución](data-factory-scheduling-and-execution.md) |En este artículo se explican los aspectos de programación y ejecución del modelo de aplicación de Azure Data Factory. |
| [Supervisión y administración de canalizaciones de Data Factory de Azure mediante la nueva Aplicación de supervisión y administración](data-factory-monitor-manage-app.md) |En este artículo se describe cómo supervisar, administrar y depurar las canalizaciones mediante la aplicación de supervisión y administración. |
