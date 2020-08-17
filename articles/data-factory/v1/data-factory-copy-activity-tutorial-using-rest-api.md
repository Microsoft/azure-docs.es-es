---
title: 'Tutorial: Uso de API REST para crear una canalización de Azure Data Factory '
description: En este tutorial, usará una API REST para crear una canalización de Azure Data Factory con una actividad de copia para copiar datos de Azure Blob Storage a Azure SQL Database.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: ''
editor: ''
ms.assetid: 1704cdf8-30ad-49bc-a71c-4057e26e7350
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 91a92f9dd0eaf55b8ba35f38102ee30b8cda4bfa
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87053815"
---
# <a name="tutorial-use-rest-api-to-create-an-azure-data-factory-pipeline-to-copy-data"></a>Tutorial: Uso de API REST para crear una canalización de Azure Data Factory 
> [!div class="op_single_selector"]
> * [Introducción y requisitos previos](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [Asistente para copia](data-factory-copy-data-wizard-tutorial.md)
> * [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
> * [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
> * [Plantilla de Azure Resource Manager](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
> * [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
> * [API de .NET](data-factory-copy-activity-tutorial-using-dotnet-api.md)
> 
> 

> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si utiliza la versión actual del servicio Data Factory, consulte el [tutorial de la actividad de copia](../quickstart-create-data-factory-rest-api.md). 

En este artículo, aprenderá a usar una API REST para crear una factoría de datos con una canalización que copia datos desde una instancia de Azure Blob Storage hasta Azure SQL Database. Si no está familiarizado con Azure Data Factory, lea el artículo [Introducción a Azure Data Factory](data-factory-introduction.md) antes de realizar este tutorial.   

En este tutorial, creará una canalización con una actividad en ella: la actividad de copia. La actividad de copia realiza la copia de los datos de un almacén de datos admitido en un almacén de datos receptor. Para obtener una lista de almacenes de datos que se admiten como orígenes y receptores, consulte los [almacenes de datos admitidos](data-factory-data-movement-activities.md#supported-data-stores-and-formats). La actividad funciona con un servicio disponible de forma global que puede copiar datos entre varios almacenes de datos de forma segura, confiable y escalable. Para más información acerca de la actividad de copia, consulte las [actividades de movimiento de datos](data-factory-data-movement-activities.md).

pero cualquier canalización puede tener más de una actividad. También puede encadenar dos actividades (ejecutar una después de otra) haciendo que el conjunto de datos de salida de una actividad sea el conjunto de datos de entrada de la otra actividad. Para más información, consulte [Varias actividades en una canalización](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline).

> [!NOTE]
> Este artículo no abarca toda la API de REST de Data Factory. Consulte [Referencia de API de REST de Data Factory](/rest/api/datafactory/) para ver la documentación completa sobre los cmdlets de Data Factory.
>  
> La canalización de datos de este tutorial copia datos de un almacén de datos de origen a un almacén de datos de destino. Para ver un tutorial acerca de cómo transformar datos mediante Azure Data Factory, consulte [Tutorial: Compilación de una canalización para transformar datos mediante el clúster de Hadoop](data-factory-build-your-first-pipeline.md).

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

* Consulte [Copia de datos de Almacenamiento de blobs en Base de datos SQL mediante Data Factory](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) y complete los pasos de los **requisitos previos** .
* Instale [Curl](https://curl.haxx.se/dlwiz/) en su máquina. Utilice la herramienta CURL con comandos de REST para crear una factoría de datos. 
* Siga las instrucciones de [este artículo](../../active-directory/develop/howto-create-service-principal-portal.md) para: 
  1. Cree una aplicación web denominada **ADFCopyTutorialApp** en Azure Active Directory.
  2. Obtenga el **Id. de cliente** y la **Clave secreta**. 
  3. Obtenga el **Identificador de inquilino**. 
  4. Asigne la aplicación **ADFCopyTutorialApp** al rol **Colaborador de Data Factory**.  
* Instale [Azure PowerShell](/powershell/azure/).  
* Inicie **PowerShell** y realice los pasos siguientes. Mantenga Azure PowerShell abierto hasta el final de este tutorial. Si lo cierra y vuelve a abrirlo, deberá ejecutar los comandos de nuevo.
  
  1. Ejecute el siguiente comando y escriba el nombre de usuario y la contraseña que utiliza para iniciar sesión en Azure Portal:
    
     ```PowerShell 
     Connect-AzAccount
     ```   
  2. Ejecute el siguiente comando para ver todas las suscripciones de esta cuenta:

     ```PowerShell     
     Get-AzSubscription
     ``` 
  3. Ejecute el comando siguiente para seleccionar la suscripción con la que desea trabajar. Reemplace **&lt;NameOfAzureSubscription**&gt; por el nombre de su suscripción de Azure. 
     
     ```PowerShell
     Get-AzSubscription -SubscriptionName <NameOfAzureSubscription> | Set-AzContext
     ```
  4. Cree un grupo de recursos de Azure denominado **ADFTutorialResourceGroup** , para lo que debe ejecutar el siguiente comando en PowerShell:  

     ```PowerShell     
      New-AzResourceGroup -Name ADFTutorialResourceGroup  -Location "West US"
     ```
     
      Si el grupo de recursos ya existe, especifique si desea actualizarlo (Y) o mantenerlo como está (N). 
     
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
    "name": "ADFCopyTutorialDF",  
    "location": "WestUS"
}  
```

### <a name="azurestoragelinkedservicejson"></a>azurestoragelinkedservice.json
> [!IMPORTANT]
> Reemplace **accountname** y **accountkey** por el nombre y la clave de su cuenta de almacenamiento de Azure. Para aprender a obtener la clave de acceso de almacenamiento, consulte [Administración de claves de acceso de la cuenta de almacenamiento](../../storage/common/storage-account-keys-manage.md).

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

Para más información acerca de las propiedades JSON, consulte [servicio vinculado Azure Storage](data-factory-azure-blob-connector.md#azure-storage-linked-service).

### <a name="azuresqllinkedservicejson"></a>azuresqllinkedservice.json
> [!IMPORTANT]
> Reemplace **servername**, **databasename**, **username** y **password** por los nombres del servidor, de la base de datos SQL, de la cuenta de usuario y de la contraseña para la cuenta.  
> 
>

```JSON
{
    "name": "AzureSqlLinkedService",
    "properties": {
        "type": "AzureSqlDatabase",
        "description": "",
        "typeProperties": {
            "connectionString": "Data Source=tcp:<servername>.database.windows.net,1433;Initial Catalog=<databasename>;User ID=<username>;Password=<password>;Integrated Security=False;Encrypt=True;Connect Timeout=30"
        }
    }
}
```

Para más información acerca de las propiedades JSON, consulte [servicio vinculado Azure SQL](data-factory-azure-sql-connector.md#linked-service-properties).

### <a name="inputdatasetjson"></a>inputdataset.json

```JSON
{
  "name": "AzureBlobInput",
  "properties": {
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
    "type": "AzureBlob",
    "linkedServiceName": "AzureStorageLinkedService",
    "typeProperties": {
      "folderPath": "adftutorial/",
      "fileName": "emp.txt",
      "format": {
        "type": "TextFormat",
        "columnDelimiter": ","
      }
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

En la siguiente tabla se ofrecen descripciones de las propiedades JSON que se usan en el fragmento de código:

| Propiedad | Descripción |
|:--- |:--- |
| type | La propiedad type se establece en **AzureBlob** porque los datos residen en una instancia de Azure Blob Storage. |
| linkedServiceName | Hace referencia al servicio **AzureStorageLinkedService** que creó anteriormente. |
| folderPath | Especifica el **contenedor** de blobs y la **carpeta** que contiene los blobs de entrada. En este tutorial, adftutorial es el contenedor de blobs y folder es la carpeta raíz. | 
| fileName | Esta propiedad es opcional. Si omite esta propiedad, se seleccionan todos los archivos de folderPath. En este tutorial, se especifica **emp.txt** en fileName, por lo que solo se selecciona ese archivo para su procesamiento. |
| format -> type |El archivo de entrada tiene formato de texto, por lo que usamos **TextFormat**. |
| columnDelimiter | Las columnas del archivo de entrada están delimitadas por **coma (`,`)** . |
| frequency/interval | La frecuencia está establecida en **Hour** y el intervalo es **1**, lo que significa que los segmentos de entrada estarán disponibles **cada hora**. En otras palabras, el servicio Data Factory busca los datos de entrada cada hora en la carpeta raíz del contenedor de blobs (**adftutorial**) que se ha especificado. Busca los datos entre las horas de inicio y finalización de la canalización, no antes ni después de esas horas.  |
| external | Esta propiedad se establece en **true** si esta canalización no ha generado los datos. Los datos de entrada de este tutorial están en el archivo emp.txt, que no lo generó esta canalización, por lo que establecemos esta propiedad en true. |

Para más información acerca de estas propiedades JSON, consulte el [artículo sobre el conector de blob de Azure](data-factory-azure-blob-connector.md#dataset-properties).

### <a name="outputdatasetjson"></a>outputdataset.json

```JSON
{
  "name": "AzureSqlOutput",
  "properties": {
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
    "type": "AzureSqlTable",
    "linkedServiceName": "AzureSqlLinkedService",
    "typeProperties": {
      "tableName": "emp"
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```
En la siguiente tabla se ofrecen descripciones de las propiedades JSON que se usan en el fragmento de código:

| Propiedad | Descripción |
|:--- |:--- |
| type | La propiedad type se establece en **AzureSqlTable** porque los datos se copian en una tabla de una base de Azure SQL Database. |
| linkedServiceName | Hace referencia al servicio **AzureSqlLinkedService** que creó anteriormente. |
| tableName | Especifica la **tabla** en la que se copian los datos. | 
| frequency/interval | La frecuencia se establece en **Hour** y el intervalo es **1**, lo que significa que los sectores de salida se producen **cada hora** entre las horas de inicio y finalización de la canalización, no antes ni después de esas horas.  |

En la tabla emp de la base de datos hay tres columnas: **ID**, **FirstName** y **LastName**. ID es una columna de identidad, por lo que deberá especificar solo **FirstName** y **LastName** aquí.

Para más información acerca de estas propiedades JSON, consulte el artículo sobre el [conector de Azure SQL Database](data-factory-azure-sql-connector.md#dataset-properties).

### <a name="pipelinejson"></a>pipeline.json

```JSON
{
  "name": "ADFTutorialPipeline",
  "properties": {
    "description": "Copy data from a blob to Azure SQL table",
    "activities": [
      {
        "name": "CopyFromBlobToSQL",
        "description": "Push Regional Effectiveness Campaign data to Azure SQL Database",
        "type": "Copy",
        "inputs": [
          {
            "name": "AzureBlobInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureSqlOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "SqlSink",
            "writeBatchSize": 10000,
            "writeBatchTimeout": "60:00:00"
          }
        },
        "Policy": {
          "concurrency": 1,
          "executionPriorityOrder": "NewestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
    ],
    "start": "2017-05-11T00:00:00Z",
    "end": "2017-05-12T00:00:00Z"
  }
}
```

Tenga en cuenta los siguientes puntos:

- En la sección de actividades, solo hay una actividad con **type** establecido en **Copy**. Para más información acerca de la actividad de copia, consulte las [actividades de movimiento de datos](data-factory-data-movement-activities.md). En las soluciones de Data Factory, también puede usar [actividades de transformación de datos](data-factory-data-transformation-activities.md).
- La entrada de la actividad está establecida en **AzureBlobInput** y la salida de la actividad está establecida en **AzureSqlOutput**. 
- En la sección **typeProperties**, **BlobSource** se especifica como el tipo de origen y **SqlSink** como el tipo de receptor. Consulte la lista de [almacenes de datos que se admiten](data-factory-data-movement-activities.md#supported-data-stores-and-formats) como orígenes y receptores de la actividad de copia. Para más información sobre cómo usar un almacén de datos admitido específico como receptor de origen, haga clic en el vínculo en la tabla.  
 
Reemplace el valor de la propiedad **start** por el día actual y el valor **end** por el próximo día. Puede especificar solo la parte de fecha y omitir la parte de hora de la fecha y hora. Por ejemplo, "03-02-2017", que es equivalente a "03-02-2017T00:00:00Z"
 
Las fechas y horas de inicio y de finalización deben estar en [formato ISO](https://en.wikipedia.org/wiki/ISO_8601). Por ejemplo: 2016-10-14T16:32:41Z. La hora de finalización ( **end** ) es opcional, pero se utilizará en este tutorial. 
 
Si no especifica ningún valor para la propiedad **end**, se calcula como "**start + 48 horas**". Para ejecutar la canalización indefinidamente, especifique **9999-09-09** como valor de propiedad **end**.
 
En el ejemplo anterior hay 24 segmentos de datos, ya que cada segmento de datos se produce cada hora.

Para obtener descripciones de las propiedades JSON en una definición de canalización, consulte cómo [crear canalizaciones](data-factory-create-pipelines.md). Para obtener descripciones de las propiedades JSON en una definición de actividad de copia, consulte las [actividades de movimiento de datos](data-factory-data-movement-activities.md). Para ver las descripciones de las propiedades JSON admitidas por BlobSource, consulte el artículo sobre el [conector de Azure Blob](data-factory-azure-blob-connector.md). Para ver las descripciones de las propiedades JSON admitidas por SqlSink, consulte el artículo sobre el [conector de Azure SQL Database](data-factory-azure-sql-connector.md).

## <a name="set-global-variables"></a>Definición de variables globales
En Azure PowerShell, ejecute los comandos siguientes después de reemplazar los valores por los suyos propios:

> [!IMPORTANT]
> Consulte la sección [Requisitos previos](#prerequisites) para obtener instrucciones sobre cómo obtener el identificador de cliente, el secreto del cliente, el identificador de inquilino y el identificador de suscripción.   
> 
> 

```JSON
$client_id = "<client ID of application in AAD>"
$client_secret = "<client key of application in AAD>"
$tenant = "<Azure tenant ID>";
$subscription_id="<Azure subscription ID>";

$rg = "ADFTutorialResourceGroup"
```

Después de actualizar el nombre de la factoría de datos que usa, ejecute el siguiente comando: 

```
$adf = "ADFCopyTutorialDF"
```

## <a name="authenticate-with-aad"></a>Autenticación con AAD
Ejecute el siguiente comando para autenticarse con Azure Active Directory (AAD): 

```PowerShell
$cmd = { .\curl.exe -X POST https://login.microsoftonline.com/$tenant/oauth2/token  -F grant_type=client_credentials  -F resource=https://management.core.windows.net/ -F client_id=$client_id -F client_secret=$client_secret };
$responseToken = Invoke-Command -scriptblock $cmd;
$accessToken = (ConvertFrom-Json $responseToken).access_token;

(ConvertFrom-Json $responseToken) 
```

## <a name="create-data-factory"></a>Creación de Data Factory
En este paso, creará una instancia de Azure Data Factory llamada **ADFCopyTutorialDF**. Una factoría de datos puede tener una o más canalizaciones. Una canalización puede tener una o más actividades. Por ejemplo, una actividad de copia para copiar datos de un almacén de datos de origen a uno de destino. Una actividad de Hive de HDInsight para ejecutar un script de Hive y convertir los datos de entrada en datos de salida del producto. Ejecute los siguientes comandos para crear la factoría de datos: 

1. Asigne el comando a la variable denominada **cmd**. 
   
    > [!IMPORTANT]
    > Confirme que el nombre de la factoría de datos que especifique aquí (ADFCopyTutorialDF) coincide con el nombre especificado en el archivo **datafactory.json**. 
   
    ```PowerShell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@datafactory.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/ADFCopyTutorialDF0411?api-version=2015-10-01};
    ```
2. Ejecute el comando con **Invoke-Command**.
   
    ```PowerShell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Vea los resultados. Si la factoría de datos se ha creado correctamente, verá su JSON en los **resultados**; de lo contrario, verá un mensaje de error.  
   
    ```
    Write-Host $results
    ```

Tenga en cuenta los siguientes puntos:

* El nombre de Azure Data Factory debe ser único de forma global. Si ve el siguiente error en los resultados, **El nombre “ADFCopyTutorialDF” de factoría de datos no está disponible**, siga estos pasos:  
  
  1. Cambie el nombre (por ejemplo, suNombreADFCopyTutorialDF) en el archivo **datafactory.json** .
  2. En el primer comando donde se asigna un valor a la variable **$cmd** , reemplace ADFCopyTutorialDF por el nuevo nombre y ejecute el comando. 
  3. Ejecute los dos comandos siguientes para invocar la API de REST para crear la factoría de datos e imprimir los resultados de la operación. 
     
     Consulte el tema [Factoría de datos: reglas de nomenclatura](data-factory-naming-rules.md) para las reglas de nomenclatura para los artefactos de Factoría de datos.
* Para crear instancias de Data Factory, debe ser administrador o colaborador de la suscripción de Azure.
* El nombre de la factoría de datos se puede registrar como un nombre DNS en el futuro y, por lo tanto, que sea visible públicamente.
* Si recibe el error: "**La suscripción no está registrada para usar el espacio de nombres Microsoft.DataFactory**", realice una de las acciones siguientes e intente publicarla de nuevo: 
  
  * En Azure PowerShell, ejecute el siguiente comando para registrar el proveedor de Data Factory: 

    ```PowerShell    
    Register-AzResourceProvider -ProviderNamespace Microsoft.DataFactory
    ```
    Puede ejecutar el comando siguiente para confirmar que se ha registrado el proveedor de Data Factory. 
    
    ```PowerShell
    Get-AzResourceProvider
    ```
  * Inicie sesión mediante la suscripción de Azure en el [Portal de Azure](https://portal.azure.com) y vaya a una hoja de Data Factory (o) cree una factoría de datos en el Portal de Azure. Esta acción registra automáticamente el proveedor.

Antes de crear una canalización, debe crear algunas entidades de factoría de datos primero. Cree unos servicios vinculados para vincular almacenes de datos de origen y destino a su almacén de datos. A continuación, defina los conjuntos de datos de entrada y salida para representar datos en almacenes de datos vinculados. Por último, cree la canalización con una actividad que utilice estos conjuntos de datos.

## <a name="create-linked-services"></a>Crear servicios vinculados
Los servicios vinculados se crean en una factoría de datos para vincular los almacenes de datos y los servicios de proceso con la factoría de datos. En este tutorial, no se usa ningún servicio de proceso, como Azure HDInsight o Azure Data Lake Analytics. Se usan dos almacenes de datos del tipo Azure Storage (origen) y Azure SQL Database (destino). Por lo tanto, se crean dos servicios vinculados llamados AzureStorageLinkedService y AzureSqlLinkedService de los tipos: AzureStorage y AzureSqlDatabase.  

AzureStorageLinkedService vincula una cuenta de Azure Storage a la factoría de datos. Esta cuenta de almacenamiento es la que se usó para crear un contenedor y con la que se cargaron los datos como parte de los [requisitos previos](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).   

AzureSqlLinkedService vincula Azure SQL Database con la factoría de datos. Los datos que se copian desde Blob Storage se almacenan en esta base de datos. Como parte de los [requisitos previos](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md), se creó la tabla emp en esta base de datos.  

### <a name="create-azure-storage-linked-service"></a>Creación de un servicio vinculado de Azure Storage
En este paso, vinculará su cuenta de Azure Storage con su factoría de datos. Especifique el nombre y la clave de la cuenta de almacenamiento de Azure en esta sección. Consulte [Servicio vinculado de Azure Storage](data-factory-azure-blob-connector.md#azure-storage-linked-service) para más información sobre las propiedades JSON usadas para definir un servicio vinculado de Azure Storage.  

1. Asigne el comando a la variable denominada **cmd**. 

    ```PowerShell   
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@azurestoragelinkedservice.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/linkedservices/AzureStorageLinkedService?api-version=2015-10-01};
    ```
2. Ejecute el comando con **Invoke-Command**.

    ```PowerShell   
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Vea los resultados. Si el servicio vinculado se ha creado correctamente, verá su JSON en los **resultados**; de lo contrario, verá un mensaje de error.

    ```PowerShell   
    Write-Host $results
    ```

### <a name="create-azure-sql-linked-service"></a>Creación de un servicio vinculado SQL de Azure
En este paso, vinculará Azure SQL Database con su factoría de datos. Especifique el nombre del servidor SQL lógico, el nombre de la base de datos, el nombre de usuario y la contraseña del usuario en esta sección. Consulte [Servicio vinculado de Azure SQL](data-factory-azure-sql-connector.md#linked-service-properties) para obtener información sobre las propiedades JSON usadas para definir un servicio vinculado de Azure SQL.

1. Asigne el comando a la variable denominada **cmd**. 
   
    ```PowerShell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@azuresqllinkedservice.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/linkedservices/AzureSqlLinkedService?api-version=2015-10-01};
    ```
2. Ejecute el comando con **Invoke-Command**.
   
    ```PowerShell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Vea los resultados. Si el servicio vinculado se ha creado correctamente, verá su JSON en los **resultados**; de lo contrario, verá un mensaje de error.
   
    ```PowerShell
    Write-Host $results
    ```

## <a name="create-datasets"></a>Creación de conjuntos de datos
En el paso anterior, creó servicios vinculados para vincular una cuenta de Azure Storage y Azure SQL Database con la factoría de datos. En este paso, se definen dos conjuntos de datos llamados AzureBlobInput y AzureSqlOutput, que representan los datos de entrada y salida que se almacenan en los almacenes de datos a los que hacen referencia AzureStorageLinkedService y AzureSqlLinkedService, respectivamente.

El servicio vinculado Azure Storage especifica la cadena de conexión que el servicio Data Factory utiliza en tiempo de ejecución para conectarse a su cuenta de Azure Storage. Además, el conjunto de datos de blobs de entrada (AzureBlobInput) especifica el contenedor y la carpeta que contiene los datos de entrada.  

De forma similar, el servicio vinculado de Azure SQL Database especifica la cadena de conexión que el servicio Data Factory utiliza en tiempo de ejecución para conectarse a Azure SQL Database. Además, el conjunto de datos de la tabla SQL de salida (OutputDataset) especifica la tabla de la base de datos en la que se copian los datos de Blob Storage. 

### <a name="create-input-dataset"></a>Creación de un conjunto de datos de entrada
En este paso, se crea un conjunto de datos llamado AzureBlobInput, que apunta a un archivo de blobs (emp.txt) en la carpeta raíz de un contenedor de blobs (adftutorial), en la instancia de Azure Storage representada por el servicio vinculado AzureStorageLinkedService. Si no especifica un valor para fileName (o puede omitirlo), los datos de todos los blobs en la carpeta de entrada se copian en el destino. En este tutorial, especifique un valor para fileName. 

1. Asigne el comando a la variable denominada **cmd**. 

    ```PowerSHell   
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@inputdataset.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datasets/AzureBlobInput?api-version=2015-10-01};
    ```
2. Ejecute el comando con **Invoke-Command**.
   
    ```PowerShell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Vea los resultados. Si el conjunto de datos se ha creado correctamente, verá su JSON en los **resultados**; de lo contrario, verá un mensaje de error.
   
    ```PowerShell
    Write-Host $results
    ```

### <a name="create-output-dataset"></a>Creación del conjunto de datos de salida
El servicio vinculado de Azure SQL Database especifica la cadena de conexión que el servicio Data Factory usa en tiempo de ejecución para conectarse a Azure SQL Database. El conjunto de datos de la tabla SQL de salida (OutputDataset) que crea en este paso especifica la tabla de la base de datos en la que se copian los datos de Blob Storage.

1. Asigne el comando a la variable denominada **cmd**.

    ```PowerShell   
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@outputdataset.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datasets/AzureSqlOutput?api-version=2015-10-01};
    ```
2. Ejecute el comando con **Invoke-Command**.
    
    ```PowerShell   
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Vea los resultados. Si el conjunto de datos se ha creado correctamente, verá su JSON en los **resultados**; de lo contrario, verá un mensaje de error.
   
    ```PowerShell
    Write-Host $results
    ``` 

## <a name="create-pipeline"></a>Creación de una canalización
En este paso, creará una canalización con una **actividad de copia** que utiliza **AzureBlobInput** como entrada y **AzureSqlOutput** como salida.

Actualmente, el conjunto de datos de salida es lo que impulsa la programación. En este tutorial, el conjunto de datos de salida está configurado para generar un segmento una vez cada hora. La canalización tiene una hora de inicio y una hora de finalización con un día de diferencia, es decir, 24 horas. Por lo tanto, la canalización produce 24 segmentos del conjunto de datos de salida. 

1. Asigne el comando a la variable denominada **cmd**.

    ```PowerShell   
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@pipeline.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datapipelines/MyFirstPipeline?api-version=2015-10-01};
    ```
2. Ejecute el comando con **Invoke-Command**.

    ```PowerShell   
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Vea los resultados. Si el conjunto de datos se ha creado correctamente, verá su JSON en los **resultados**; de lo contrario, verá un mensaje de error.  

    ```PowerShell   
    Write-Host $results
    ```

**¡Enhorabuena!** Ha creado correctamente una factoría de datos de Azure, con una canalización que copia datos desde Azure Blob Storage hasta Azure SQL Database.

## <a name="monitor-pipeline"></a>Supervisión de la canalización
En este paso, utilizará la API de REST de Data Factory para supervisar los segmentos que la canalización está produciendo.

```PowerShell
$ds ="AzureSqlOutput"
```

> [!IMPORTANT] 
> Asegúrese de que las horas de inicio y finalización especificadas en el siguiente comando coinciden con las horas de inicio y finalización de la canalización. 

```PowerShell
$cmd = {.\curl.exe -X GET -H "Authorization: Bearer $accessToken" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datasets/$ds/slices?start=2017-05-11T00%3a00%3a00.0000000Z"&"end=2017-05-12T00%3a00%3a00.0000000Z"&"api-version=2015-10-01};
```

```PowerShell
$results2 = Invoke-Command -scriptblock $cmd;
```

```PowerShell
IF ((ConvertFrom-Json $results2).value -ne $NULL) {
    ConvertFrom-Json $results2 | Select-Object -Expand value | Format-Table
} else {
        (convertFrom-Json $results2).RemoteException
}
```

Ejecute el comando Invoke y el siguiente hasta que vea el segmento en estado **Listo** o **Error**. Cuando el segmento esté en estado Listo, compruebe la tabla **emp** de Azure SQL Database para ver los datos de salida. 

Por cada segmento, se copian dos filas de datos del archivo de origen en la tabla emp de Azure SQL Database. Por tanto, podrá ver 24 nuevos registros en la tabla emp cuando todos los segmentos se procesan correctamente (en estado Listo). 

## <a name="summary"></a>Resumen
En este tutorial, ha usado una API REST para crear una factoría de datos de Azure y copiar datos desde una instancia de Azure Bob Storage hasta Azure SQL Database. Estos son los pasos de alto nivel que realizó en este tutorial:  

1. Ha creado una **factoría de datos**de Azure.
2. Ha creado **servicios vinculados**.
   1. Un servicio vinculado Azure Storage para vincular la cuenta de Azure Storage que contiene datos de entrada.     
   2. Un servicio vinculado de Azure SQL para vincular la base de datos que contiene los datos de salida. 
3. Ha creado **conjuntos de datos**que describen los datos de entrada y salida para las canalizaciones.
4. Ha creado una **canalización** con una actividad de copia con un origen BlobSource y un receptor SqlSink. 

## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha usado Azure Blob Storage como almacén de datos de origen y Azure SQL Database como almacén de datos de destino en una operación de copia. En la tabla siguiente se proporciona una lista de almacenes de datos que se admiten como orígenes y destinos de la actividad de copia: 

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

Para aprender a copiar datos hacia y desde un almacén de datos, haga clic en el vínculo del almacén de datos en la tabla.
