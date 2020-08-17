---
title: Entornos de proceso compatibles con Azure Data Factory versión 1
description: Obtenga información sobre los entornos de proceso que puede usar en las canalizaciones de Azure Data Factory (como Azure HDInsight) para transformar o procesar datos.
services: data-factory
documentationcenter: ''
author: djpmsft
ms.author: daperlov
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: ac92e45e69522fe3de8abdb3afcf6049e5f07ac8
ms.sourcegitcommit: 97a0d868b9d36072ec5e872b3c77fa33b9ce7194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87563507"
---
# <a name="compute-environments-supported-by-azure-data-factory-version-1"></a>Entornos de proceso compatibles con Azure Data Factory versión 1
> [!NOTE]
> Este artículo se aplica a la versión 1 de Azure Data Factory. Si utiliza la versión actual del servicio Data Factory, consulte el [Servicios vinculados de Compute en](../compute-linked-services.md).

En este artículo se explican los entornos de procesos que se pueden usar para procesar o transformar datos. También se proporcionan detalles sobre las distintas configuraciones (a petición en comparación con traiga una propia) compatibles con Data Factory cuando se configuran servicios vinculados que vinculan estos entornos de procesos a una instancia de Azure Data Factory.

En la tabla siguiente se proporciona una lista de los entornos de proceso compatibles con Data Factory y las actividades que se pueden ejecutar en ellos. 

| Entorno de procesos                      | Actividades                               |
| ---------------------------------------- | ---------------------------------------- |
| [Clúster de Azure HDInsight a petición](#azure-hdinsight-on-demand-linked-service) o [clúster HDInsight propio](#azure-hdinsight-linked-service) | [DotNet](data-factory-use-custom-activities.md), [Hive](data-factory-hive-activity.md), [Pig](data-factory-pig-activity.md), [MapReduce](data-factory-map-reduce.md), [Hadoop Streaming](data-factory-hadoop-streaming-activity.md) |
| [Azure Batch](#azure-batch-linked-service) | [DotNet](data-factory-use-custom-activities.md) |
| [Azure Machine Learning](#azure-machine-learning-linked-service) | [Actividades de Machine Learning: ejecución de lotes y recurso de actualización](data-factory-azure-ml-batch-execution-activity.md) |
| [Análisis con Azure Data Lake](#azure-data-lake-analytics-linked-service) | [U-SQL de análisis con Data Lake](data-factory-usql-activity.md) |
| [Azure SQL](#azure-sql-linked-service), [Azure SQL Data Warehouse](#azure-sql-data-warehouse-linked-service), [SQL Server](#sql-server-linked-service) | [Actividad de procedimiento almacenado](data-factory-stored-proc-activity.md) |

## <a name="hdinsight-versions-supported-in-data-factory"></a><a name="supported-hdinsight-versions-in-azure-data-factory"></a>Versiones de HDInsight compatibles con Data Factory
Azure HDInsight admite varias versiones de clústeres de Hadoop que puede implementar en cualquier momento. Cada versión compatible crea una versión específica de la distribución Hortonworks Data Platform (HDP) y un conjunto de componentes en la distribución. 

Microsoft actualiza la lista de versiones compatibles de HDInsight con las correcciones y los componentes más recientes del ecosistema de Hadoop. Para más información, consulte las [versiones compatibles de HDInsight](../../hdinsight/hdinsight-component-versioning.md#supported-hdinsight-versions).

> [!IMPORTANT]
> La versión 3.3 de HDInsight basado en Linux se retiró el 31 de julio de 2017. A los clientes de servicios vinculados de HDInsight a petición de Data Factory versión 1 se les dio plazo hasta el 15 de diciembre de 2017 para probar y actualizar a una versión posterior de HDInsight. HDInsight basado en Windows se retirará el 31 de julio de 2018.
>
> 

### <a name="after-the-retirement-date"></a>Después de la fecha de retirada 

Después del 15 de diciembre de 2017:

- Ya no puede crear clústeres de la versión 3.3 (o versiones anteriores) de HDInsight basado en Linux mediante un servicio vinculado de HDInsight a petición en Data Factory versión 1. 
- Si no se especifican las propiedades [**osType** y **Version**](https://docs.microsoft.com/azure/data-factory/v1/data-factory-compute-linked-services#azure-hdinsight-on-demand-linked-service) en la definición de JSON para un servicio existente vinculado a HDInsight a petición de Data Factory versión 1, el valor predeterminado cambia de **Version=3.1, osType=Windows** a **Version=\<latest HDI default version\>(https://docs.microsoft.com/azure/hdinsight/hdinsight-component-versioning) , osType=Linux**.

Después del 31 de julio de 2018:

- Ya no puede crear ninguna versión de clústeres de HDInsight basado en Windows mediante un servicio vinculado de HDInsight a petición en Data Factory versión 1. 

### <a name="recommended-actions"></a>Acciones recomendadas 

- Para asegurarse de que puede usar las correcciones y los componentes más recientes del ecosistema de Hadoop, actualice las propiedades [**osType** y **Version**](https://docs.microsoft.com/azure/data-factory/v1/data-factory-compute-linked-services#azure-hdinsight-on-demand-linked-service) de las definiciones del servicio vinculado de HDInsight a petición de Data Factory versión 1 afectado a versiones de HDInsight basado en Linux más recientes (HDInsight 3.6). 
- Antes del 15 de diciembre de 2017, pruebe las actividades de streaming de Hive, Pig, MapReduce y Hadoop de Data Factory versión 1 que hacen referencia al servicio vinculado afectado. Asegúrese de que sean compatibles con los valores predeterminados **osType** y **Version** nuevos (**Version=3.6**, **osType=Linux**) o el tipo de sistema operativo y la versión de HDInsight explícitos a los que se está actualizando. 
  Para más información sobre la compatibilidad, consulte [Migración desde un clúster de HDInsight basado en Windows a un clúster basado en Linux](https://docs.microsoft.com/azure/hdinsight/hdinsight-migrate-from-windows-to-linux) y [¿Cuáles son los componentes y versiones de Hadoop disponibles con HDInsight?](https://docs.microsoft.com/azure/hdinsight/hdinsight-component-versioning#hortonworks-release-notes-associated-with-hdinsight-versions) 
- Para seguir usando un servicio vinculado de HDInsight a petición de Data Factory versión 1 para crear clústeres de HDInsight basados en Windows, establezca explícitamente la propiedad **osType** en **Windows** antes del 15 de diciembre de 2017. Se recomienda migrar a los clústeres de HDInsight basado en Linux antes del 31 de julio de 2018. 
- Si usa un servicio vinculado de HDInsight a petición para ejecutar la actividad personalizada de DotNet de Data Factory versión 1, actualice la definición JSON de actividad personalizada de DotNet para usar en su lugar un servicio vinculado de Azure Batch. Para más información, consulte [Uso de actividades personalizadas en una canalización de Data Factory](https://docs.microsoft.com/azure/data-factory/v1/data-factory-use-custom-activities). 

> [!Note]
> Si usa un dispositivo BYOD existente vinculado a un clúster de HDInsight en Data Factory versión 1 o un dispositivo BYOD bajo petición vinculado a HDInsight en Azure Data Factory, no se requiere ninguna acción. En esos escenarios, ya se aplica la directiva de compatibilidad de versión más reciente de los clústeres de HDInsight. 
>
> 


## <a name="on-demand-compute-environment"></a>Entorno de procesos a petición
En una configuración a petición, Data Factory administra completamente el entorno de proceso. Data Factory crea de manera automática el entorno de proceso antes de enviar un trabajo para el procesamiento de datos. Cuando se finaliza el trabajo, Data Factory quita el entorno de proceso. 

Puede crear un servicio vinculado para un entorno de proceso a petición. Use el servicio vinculado para configurar el entorno de proceso y para controlar la configuración granular de la ejecución del trabajo, la administración del clúster y las acciones de arranque.

> [!NOTE]
> Actualmente, la configuración a petición solo es compatible con los clústeres de HDInsight.
> 

## <a name="azure-hdinsight-on-demand-linked-service"></a>Servicio vinculado a petición de HDInsight de Azure
Data Factory puede crear automáticamente un clúster de HDInsight a petición basado en Windows o basado en Linux para procesar los datos. El clúster se crea en la misma región que la cuenta de almacenamiento asociada con el clúster. Use la propiedad **linkedServiceName** de JSON para crear el clúster.

Tenga en cuenta los puntos *clave* siguientes sobre el servicio vinculado de HDInsight a petición:

* El clúster de HDInsight a petición no aparece en la suscripción de Azure. El servicio Data Factory administra el clúster de HDInsight a petición en su nombre.
* Los registros de trabajos que se ejecutan en un clúster de HDInsight a petición se copian en la cuenta de almacenamiento que está asociada con el clúster de HDInsight. Para acceder a estos registros, en Azure Portal, vaya al panel **Detalles de ejecución de actividad**. Para más información, consulte el artículo sobre la [supervisión y administración de canalizaciones](data-factory-monitor-manage-pipelines.md).
* Se le cobrará solo por el tiempo en el que el clúster de HDInsight esté en ejecución y realizando trabajos.

> [!IMPORTANT]
> Aprovisionar un clúster de HDInsight a petición suele tardar *20 minutos* o más.
>
> 

### <a name="example"></a>Ejemplo
En el siguiente JSON se define un servicio vinculado de HDInsight a petición basado en Linux. Data Factory crea automáticamente un clúster de HDInsight *basado en Linux* cuando procesa un segmento de datos. 

```json
{
    "name": "HDInsightOnDemandLinkedService",
    "properties": {
        "type": "HDInsightOnDemand",
        "typeProperties": {
            "version": "3.6",
            "osType": "Linux",
            "clusterSize": 1,
            "timeToLive": "00:05:00",            
            "linkedServiceName": "AzureStorageLinkedService"
        }
    }
}
```

> [!IMPORTANT]
> El clúster de HDInsight crea un *contenedor predeterminado* en la instancia de Azure Blob Storage que especifica en la propiedad **linkedServiceName** de JSON. De manera predeterminada, HDInsight no elimina este contenedor cuando se elimina el clúster. En un servicio vinculado de HDInsight a petición, se crea un clúster de HDInsight cada vez que se debe procesar un segmento, a menos que haya un clúster activo (**timeToLive**). El clúster se elimina cuando finaliza el procesamiento. 
>
> A medida que se procesen más segmentos, verá numerosos contenedores en su almacenamiento de blobs. Si no necesita los contenedores para solucionar los problemas de los trabajos, puede desear eliminar los contenedores para reducir el costo de almacenamiento. Los nombres de estos contenedores siguen un patrón: `adf<your Data Factory name>-<linked service name>-<date and time>`. Puede usar una herramienta como [Explorador de Microsoft Azure Storage](https://storageexplorer.com/) para eliminar los contenedores en el almacenamiento de blobs.
>
> 

### <a name="properties"></a>Propiedades
| Propiedad                     | Descripción                              | Obligatorio |
| ---------------------------- | ---------------------------------------- | -------- |
| type                         | Establezca la propiedad type en **HDInsightOnDemand**. | Sí      |
| clusterSize                  | El número de nodos de trabajo y de datos en el clúster. El clúster de HDInsight se crea con dos nodos principales además del número de nodos de trabajo que especifica para esta propiedad. El tamaño de los nodos es Standard_D3, que tiene 4 núcleos. Un clúster de 4 nodos de trabajo necesita 24 núcleos, (4\*4 = 16 núcleos para nodos de trabajo, más 2\*4 = 8 núcleos para nodos principales). Para detalles sobre el nivel Standard_D3, consulte [Creación de clústeres de Hadoop basados en Linux en HDInsight](../../hdinsight/hdinsight-hadoop-provision-linux-clusters.md). | Sí      |
| timeToLive                   | El tiempo de inactividad permitido para el clúster de HDInsight a petición. Especifica cuánto tiempo permanece activo el clúster de HDInsight a petición cuando finaliza una ejecución de actividad si no hay ningún otro trabajo activo en el clúster.<br /><br />Por ejemplo, si una ejecución de actividad tarda 6 minutos y **timeToLive** está establecido en 5 minutos, el clúster permanece activo durante 5 minutos después de los 6 minutos de procesamiento de la ejecución de actividad. Si se ejecuta otra actividad en un margen de 6 minutos, la procesa el mismo clúster.<br /><br />Crear un clúster de HDInsight a petición es una operación costosa (podría tardar un tiempo). Use esta configuración según sea necesario para mejorar el rendimiento de una factoría de datos mediante la reutilización de un clúster de HDInsight a petición.<br /><br />Si establece el valor **timeToLive** en **0**, el clúster se elimina en cuenta se finaliza la ejecución de actividad. Sin embargo,si establece un valor alto, el clúster puede permanecer inactivo, lo que genera costos elevados innecesariamente. Es importante establecer el valor adecuado en función de sus necesidades.<br /><br />Varias canalizaciones pueden compartir la instancia del clúster de HDInsight a petición si el valor de la propiedad **timeToLive** está correctamente establecido. | Sí      |
| version                      | La versión del clúster de HDInsight. Para una lista de las versiones de HDInsight permitidas, consulte [Versiones compatibles de HDInsight](https://docs.microsoft.com/azure/hdinsight/hdinsight-component-versioning#supported-hdinsight-versions). Si no se especifica este valor, se usa la [versión predeterminada de HDI más reciente](https://docs.microsoft.com/azure/hdinsight/hdinsight-component-versioning). | No       |
| linkedServiceName            | El servicio vinculado de Azure Storage que el clúster a petición usará para almacenar y procesar datos. El clúster de HDInsight se crea en la misma región que esta cuenta de almacenamiento.<p>Actualmente, no puede crear un clúster de HDInsight a petición que use Azure Data Lake Store como el almacenamiento. Si desea almacenar los datos de resultados del procesamiento de HDInsight en Data Lake Store, use la actividad de copia para copiar los datos desde Blob Storage a Data Lake Store. </p> | Sí      |
| additionalLinkedServiceNames | Especifica cuentas de almacenamiento adicionales para el servicio vinculado de HDInsight. Data Factory registra las cuentas de almacenamiento en su nombre. Estas cuentas de almacenamiento deben estar en la misma región que el clúster de HDInsight. El clúster de HDInsight se crea en la misma región que la cuenta de almacenamiento que se especifica en la propiedad **linkedServiceName**. | No       |
| osType                       | El tipo de sistema operativo. Los valores permitidos son **Linux** y **Windows**. Si no se especifica este valor, se usa **Linux**.  <br /><br />Se recomienda usar los clústeres de HDInsight basado en Linux. La fecha de retirada de HDInsight en Windows es el 31 de julio de 2018. | No       |
| hcatalogLinkedServiceName    | El nombre del servicio vinculado de Azure SQL que apunta a la base de datos de HCatalog. El clúster de HDInsight a petición se crea mediante el uso de la base de datos SQL como metastore. | No       |

#### <a name="example-linkedservicenames-json"></a>Ejemplo: JSON LinkedServiceNames

```json
"additionalLinkedServiceNames": [
    "otherLinkedServiceName1",
    "otherLinkedServiceName2"
  ]
```

### <a name="advanced-properties"></a>Propiedades avanzadas
Para la configuración granular del clúster de HDInsight a petición, puede especificar las propiedades siguientes:

| Propiedad               | Descripción                              | Obligatorio |
| :--------------------- | :--------------------------------------- | :------- |
| coreConfiguration      | Especifica los parámetros de configuración Core (core-site.xml) para el clúster de HDInsight que se va a crear. | No       |
| hBaseConfiguration     | Especifica los parámetros de configuración HBase (como en hbase-site.xml) para el clúster de HDInsight. | No       |
| hdfsConfiguration      | Especifica los parámetros de configuración HDFS (hdfs-site.xml) para el clúster de HDInsight. | No       |
| hiveConfiguration      | Especifica los parámetros de configuración Hive (hive-site.xml) para el clúster de HDInsight. | No       |
| mapReduceConfiguration | Especifica los parámetros de configuración MapReduce (mapred-site.xml) para el clúster de HDInsight. | No       |
| oozieConfiguration     | Especifica los parámetros de configuración Oozie (oozie-site.xml) para el clúster de HDInsight. | No       |
| stormConfiguration     | Especifica los parámetros de configuración Storm (storm-site.xml) para el clúster de HDInsight. | No       |
| yarnConfiguration      | Especifica los parámetros de configuración YARN (yarn-site.xml) para el clúster de HDInsight. | No       |

#### <a name="example-on-demand-hdinsight-cluster-configuration-with-advanced-properties"></a>Ejemplo: Configuración del clúster de HDInsight a petición con propiedades avanzadas

```json
{
  "name": " HDInsightOnDemandLinkedService",
  "properties": {
    "type": "HDInsightOnDemand",
    "typeProperties": {
      "version": "3.6",
      "osType": "Linux",
      "clusterSize": 16,
      "timeToLive": "01:30:00",
      "linkedServiceName": "adfods1",
      "coreConfiguration": {
        "templeton.mapper.memory.mb": "5000"
      },
      "hiveConfiguration": {
        "templeton.mapper.memory.mb": "5000"
      },
      "mapReduceConfiguration": {
        "mapreduce.reduce.java.opts": "-Xmx4000m",
        "mapreduce.map.java.opts": "-Xmx4000m",
        "mapreduce.map.memory.mb": "5000",
        "mapreduce.reduce.memory.mb": "5000",
        "mapreduce.job.reduce.slowstart.completedmaps": "0.8"
      },
      "yarnConfiguration": {
        "yarn.app.mapreduce.am.resource.mb": "5000",
        "mapreduce.map.memory.mb": "5000"
      },
      "additionalLinkedServiceNames": [
        "datafeeds",
        "adobedatafeed"
      ]
    }
  }
}
```

### <a name="node-sizes"></a>Tamaño de nodo
Use estas propiedades para especificar el tamaño de los nodos principal, de datos y de ZooKeeper: 

| Propiedad          | Descripción                              | Obligatorio |
| :---------------- | :--------------------------------------- | :------- |
| headNodeSize      | Establece el tamaño del nodo principal. El valor predeterminado es **Standard_D3**. Para detalles, consulte cómo [especificar los tamaños de nodo](#specify-node-sizes). | No       |
| dataNodeSize      | Establece el tamaño del nodo de datos. El valor predeterminado es **Standard_D3**. | No       |
| zookeeperNodeSize | Establece el tamaño del nodo de ZooKeeper. El valor predeterminado es **Standard_D3**. | No       |

#### <a name="specify-node-sizes"></a>Especificación de los tamaños de nodo
Si desea conocer los valores de cadena que debe especificar para las propiedades descritas en la sección anterior, consulte cuáles son los [tamaños de máquina virtual](../../virtual-machines/linux/sizes.md). Los valores deben cumplir con las API y los cmdlets a los que se hace referencia en el artículo sobre los [tamaños de máquina virtual](../../virtual-machines/linux/sizes.md). El tamaño de nodo de datos Grande (valor predeterminado) tiene 7 GB de memoria. Podría no ser suficiente para su escenario. 

Si quiere crear nodos de trabajo y principales de tamaño D4, especifique **Standard_D4** como el valor de las propiedades **headNodeSize** y **dataNodeSize**: 

```json
"headNodeSize": "Standard_D4",    
"dataNodeSize": "Standard_D4",
```

Si establece un valor incorrecto para estas propiedades, podría ver el mensaje siguiente:

  Error al crear el clúster. Excepción: No se puede completar la operación de creación del clúster. Error en la operación con el código '400'. El clúster generó el estado: “Error”. Mensaje: “PreClusterCreationValidationFailure”. 
  
Si ve este mensaje, asegúrese de que usa los nombres de las API y los cmdlets de la tabla que aparece en el artículo sobre los [tamaños de máquina virtual](../../virtual-machines/linux/sizes.md).  

> [!NOTE]
> Actualmente, Data Factory no es compatible con los clústeres de HDInsight que usan Data Lake Store como el almacén principal. Use Azure Storage como el almacén principal de los clústeres de HDInsight. 
>
> 


## <a name="bring-your-own-compute-environment"></a>Traer su propio entorno de procesos
Puede registrar un entorno de proceso existente como un servicio vinculado en Data Factory. Usted administra el entorno de proceso. El servicio Data Factory usa el entorno de proceso para ejecutar actividades.

Este tipo de configuración se admite para los entornos de procesos siguientes:

* HDInsight de Azure
* Azure Batch
* Azure Machine Learning
* Análisis con Azure Data Lake
* Azure SQL Database, Azure SQL Data Warehouse, SQL Server

## <a name="azure-hdinsight-linked-service"></a>Servicio vinculado de HDInsight de Azure
Puede crear un servicio vinculado de HDInsight para registrar su propio clúster de HDInsight con Data Factory.

### <a name="example"></a>Ejemplo

```json
{
  "name": "HDInsightLinkedService",
  "properties": {
    "type": "HDInsight",
    "typeProperties": {
      "clusterUri": " https://<hdinsightclustername>.azurehdinsight.net/",
      "userName": "admin",
      "password": "<password>",
      "linkedServiceName": "MyHDInsightStoragelinkedService"
    }
  }
}
```

### <a name="properties"></a>Propiedades
| Propiedad          | Descripción                              | Obligatorio |
| ----------------- | ---------------------------------------- | -------- |
| type              | Establezca la propiedad type en **HDInsight**. | Sí      |
| clusterUri        | El URI del clúster de HDInsight.        | Sí      |
| username          | El nombre de la cuenta de usuario que se usará para conectarse a un clúster de HDInsight existente. | Sí      |
| password          | Contraseña para la cuenta de usuario.   | Sí      |
| linkedServiceName | El nombre del servicio vinculado de almacenamiento que hace referencia al almacenamiento de blobs que el clúster de HDInsight usa. <p>Actualmente, no puede especificar un servicio vinculado de Data Lake Store para esta propiedad. Si el clúster de HDInsight tiene acceso a Data Lake Store, puede acceder a los datos de Data Lake Store desde scripts de Hive o de Pig. </p> | Sí      |

## <a name="azure-batch-linked-service"></a>Servicio vinculado de Azure Batch
Puede crear un servicio vinculado de Batch para registrar un grupo de Batch de máquinas virtuales (VM) en una factoría de datos. Puede ejecutar actividades personalizadas de Microsoft .NET mediante el uso de Batch o de HDInsight.

Si es primera vez que usa el servicio de Batch:

* Obtenga información sobre los [conceptos básicos de Azure Batch](../../azure-sql/database/sql-database-paas-overview.md).
* Obtenga información sobre el cmdlet [New-AzureBatchAccount](https://msdn.microsoft.com/library/mt125880.aspx). Use este cmdlet para crear una cuenta de Batch. O bien, puede crear la cuenta de Batch con [Azure Portal](../../batch/batch-account-create-portal.md). Para información detallada sobre cómo usar el cmdlet, consulte el artículo sobre el [use de PowerShell para administrar una cuenta de Batch](https://blogs.technet.com/b/windowshpc/archive/2014/10/28/using-azure-powershell-to-manage-azure-batch-account.aspx).
* Obtenga información sobre el cmdlet [New-AzureBatchPool](https://msdn.microsoft.com/library/mt125936.aspx). Use este cmdlet para crear un grupo de Batch.

### <a name="example"></a>Ejemplo

```json
{
  "name": "AzureBatchLinkedService",
  "properties": {
    "type": "AzureBatch",
    "typeProperties": {
      "accountName": "<Azure Batch account name>",
      "accessKey": "<Azure Batch account key>",
      "poolName": "<Azure Batch pool name>",
      "linkedServiceName": "<Specify associated storage linked service reference here>"
    }
  }
}
```

Para la propiedad **accountName**, anexe **.\<region name\>** al nombre de la cuenta de Batch. Por ejemplo:

```json
"accountName": "mybatchaccount.eastus"
```

Otra opción es proporcionar el punto de conexión **batchUri**. Por ejemplo:

```json
"accountName": "adfteam",
"batchUri": "https://eastus.batch.azure.com",
```

### <a name="properties"></a>Propiedades
| Propiedad          | Descripción                              | Obligatorio |
| ----------------- | ---------------------------------------- | -------- |
| type              | Establezca la propiedad type en **AzureBatch**. | Sí      |
| accountName       | El nombre de la cuenta de Batch.         | Sí      |
| accessKey         | La clave de acceso de la cuenta de Batch.  | Sí      |
| poolName          | El nombre del grupo de máquinas virtuales.    | Sí      |
| linkedServiceName | El nombre del servicio vinculado de almacenamiento que está asociado con este servicio vinculado de Batch. Este servicio vinculado se usa para los archivos de almacenamiento provisional necesarios para ejecutar la actividad y para almacenar los registros de ejecución de actividad. | Sí      |

## <a name="azure-machine-learning-linked-service"></a>Servicio vinculado de Azure Machine Learning
Puede crear un servicio vinculado de Machine Learning para registrar un punto de conexión de puntuación de Batch de Machine Learning en una factoría de datos.

### <a name="example"></a>Ejemplo

```json
{
  "name": "AzureMLLinkedService",
  "properties": {
    "type": "AzureML",
    "typeProperties": {
      "mlEndpoint": "https://[batch scoring endpoint]/jobs",
      "apiKey": "<apikey>"
    }
  }
}
```

### <a name="properties"></a>Propiedades
| Propiedad   | Descripción                              | Obligatorio |
| ---------- | ---------------------------------------- | -------- |
| Tipo       | Establezca la propiedad type en **AzureML**. | Sí      |
| mlEndpoint | La dirección URL de puntuación por lotes.                   | Sí      |
| apiKey     | La API del modelo de área de trabajo publicado.     | Sí      |

## <a name="azure-data-lake-analytics-linked-service"></a>Servicio vinculado de Azure Data Lake Analytics
Puede crear un servicio vinculado de Data Lake Analytics para vincular un servicio de proceso de Data Lake Analytics a una instancia de Azure Data Factory. La actividad de U-SQL de Data Lake Analytics de la canalización hace referencia a este servicio vinculado. 

En la tabla siguiente se describen las propiedades genéricas que se usan en la definición JSON:

| Propiedad                 | Descripción                              | Obligatorio                                 |
| ------------------------ | ---------------------------------------- | ---------------------------------------- |
| type                 | Establezca la propiedad type en **AzureDataLakeAnalytics**. | Sí                                      |
| accountName          | El nombre de la cuenta de Data Lake Analytics.  | Sí                                      |
| dataLakeAnalyticsUri | El identificador URI de Data Lake Analytics.           | No                                       |
| subscriptionId       | El identificador de la suscripción de Azure.                    | No<br /><br />(Si no se especifica, se usa la suscripción de Data Factory). |
| resourceGroupName    | El nombre del grupo de recursos de Azure.                | No<br /><br /> (Si no se especifica, se usa el grupo de recursos de Data Factory). |

### <a name="authentication-options"></a>Opciones de autenticación
Para el servicio vinculado de Data Lake Analytics, puede elegir entre la autenticación mediante una entidad de servicio o una credencial de usuario.

#### <a name="service-principal-authentication-recommended"></a>Autenticación de la entidad de servicio (recomendada)
Para usar la autenticación de entidad de servicio, registre una entidad de aplicación en Azure Active Directory (Azure AD). Luego, conceda a Azure AD acceso a Data Lake Store. Consulte [Autenticación entre servicios](../../data-lake-store/data-lake-store-authenticate-using-active-directory.md) para ver los pasos detallados. Anote los siguientes valores; los usará para definir el servicio vinculado:
* Identificador de aplicación
* Clave de la aplicación 
* Id. de inquilino

Para usar la autenticación de la entidad de servicio, especifique las siguientes propiedades:

| Propiedad                | Descripción                              | Obligatorio |
| :---------------------- | :--------------------------------------- | :------- |
| servicePrincipalId  | El identificador de cliente de la aplicación.     | Sí      |
| servicePrincipalKey | La clave de la aplicación.           | Sí      |
| tenant              | La información del inquilino (nombre de dominio o identificador de inquilino) en el que se ubica la aplicación. Para obtener esta información, mantenga el puntero del mouse en la esquina superior derecha de Azure Portal. | Sí      |

**Ejemplo: Autenticación de entidad de servicio**
```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "adftestaccount",
            "dataLakeAnalyticsUri": "datalakeanalyticscompute.net",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<optional, subscription id of ADLA>",
            "resourceGroupName": "<optional, resource group name of ADLA>"
        }
    }
}
```

#### <a name="user-credential-authentication"></a>Autenticación de credenciales de usuario
Para la autenticación de credenciales de usuario de Data Lake Analytics, especifique las propiedades siguientes:

| Propiedad          | Descripción                              | Obligatorio |
| :---------------- | :--------------------------------------- | :------- |
| authorization | En el Editor de Data Factory, seleccione el botón **Autorizar**. Escriba la credencial que asigna la dirección URL de autorización generada automáticamente a esta propiedad. | Sí      |
| sessionID     | El identificador de sesión de OAuth de la sesión de autorización de OAuth. Cada id. de sesión es único y solo se puede usar una vez. Esta configuración se genera automáticamente al usar el Editor de Data Factory. | Sí      |

**Ejemplo: Autenticación de credenciales de usuario**
```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "adftestaccount",
            "dataLakeAnalyticsUri": "datalakeanalyticscompute.net",
            "authorization": "<authcode>",
            "sessionId": "<session ID>", 
            "subscriptionId": "<optional, subscription id of ADLA>",
            "resourceGroupName": "<optional, resource group name of ADLA>"
        }
    }
}
```

#### <a name="token-expiration"></a>Expiración del token
El código de autorización que genera al seleccionar el botón **Autorizar** expira después de un intervalo determinado. 

Puede ver el mensaje de error siguiente cuando expira el token de autenticación: 

  Error de operación de credencial: invalid_grant - AADSTS70002: error al validar las credenciales. AADSTS70008: la concesión de acceso proporcionada expiró o se revocó. Identificador de seguimiento: d18629e8-af88-43c5-88e3-d8419eb1fca1 Identificador de correlación: fac30a0c-6be6-4e02-8d69-a776d2ffefd7 Marca de tiempo: 2015-12-15 21:09:31Z

En la tabla siguiente se muestran las expiraciones por tipo de cuenta de usuario: 

| Tipo de usuario                                | Expira después de                            |
| :--------------------------------------- | :--------------------------------------- |
| Cuentas de usuario que Azure AD *no* administra (Hotmail, Live, etc.) | 12 horas.                                 |
| Cuentas de usuario que Azure AD *sí* administra | 14 días después de la ejecución del último segmento. <br /><br />90 días, si un segmento que se basa en un servicio vinculado basado en OAuth se ejecuta al menos una vez cada 14 días. |

Para evitar o resolver este error, vuelva a dar la autorización con el botón **Autorizar** cuando expire el token. Luego, vuelva a implementar el servicio vinculado. También puede generar valores para las propiedades **sessionId** y **authorization** mediante programación por medio del código siguiente:

```csharp
if (linkedService.Properties.TypeProperties is AzureDataLakeStoreLinkedService ||
    linkedService.Properties.TypeProperties is AzureDataLakeAnalyticsLinkedService)
{
    AuthorizationSessionGetResponse authorizationSession = this.Client.OAuth.Get(this.ResourceGroupName, this.DataFactoryName, linkedService.Properties.Type);

    WindowsFormsWebAuthenticationDialog authenticationDialog = new WindowsFormsWebAuthenticationDialog(null);
    string authorization = authenticationDialog.AuthenticateAAD(authorizationSession.AuthorizationSession.Endpoint, new Uri("urn:ietf:wg:oauth:2.0:oob"));

    AzureDataLakeStoreLinkedService azureDataLakeStoreProperties = linkedService.Properties.TypeProperties as AzureDataLakeStoreLinkedService;
    if (azureDataLakeStoreProperties != null)
    {
        azureDataLakeStoreProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeStoreProperties.Authorization = authorization;
    }

    AzureDataLakeAnalyticsLinkedService azureDataLakeAnalyticsProperties = linkedService.Properties.TypeProperties as AzureDataLakeAnalyticsLinkedService;
    if (azureDataLakeAnalyticsProperties != null)
    {
        azureDataLakeAnalyticsProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeAnalyticsProperties.Authorization = authorization;
    }
}
```

Para detalles sobre las clases de Data Factory que se usan en este ejemplo de código, consulte:
* [Clase AzureDataLakeStoreLinkedService](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuredatalakestorelinkedservice.aspx)
* [Clase AzureDataLakeAnalyticsLinkedService](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuredatalakeanalyticslinkedservice.aspx)
* [Clase AuthorizationSessionGetResponse](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.authorizationsessiongetresponse.aspx)

Agregue una referencia a Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms.dll para la clase **WindowsFormsWebAuthenticationDialog**. 

## <a name="azure-sql-linked-service"></a>Servicio vinculado de Azure SQL
Puede crear un servicio vinculado de SQL y usarlo con la [actividad de procedimiento almacenado](data-factory-stored-proc-activity.md) para invocar un procedimiento almacenado desde una canalización de Data Factory. Para más información, consulte el artículo sobre el [conector de Azure SQL](data-factory-azure-sql-connector.md#linked-service-properties).

## <a name="azure-sql-data-warehouse-linked-service"></a>Servicio vinculado de Azure SQL Data Warehouse
Puede crear un servicio vinculado de SQL Data Warehouse y usarlo con la [actividad de procedimiento almacenado](data-factory-stored-proc-activity.md) para invocar un procedimiento almacenado desde una canalización de Data Factory. Para más información, consulte el artículo sobre la [copia de datos con Azure SQL Data Warehouse como origen o destino mediante Azure Data Factory](data-factory-azure-sql-data-warehouse-connector.md#linked-service-properties).

## <a name="sql-server-linked-service"></a>Servicio vinculado de SQL Server
Puede crear un servicio vinculado de SQL Server y usarlo con la [actividad de procedimiento almacenado](data-factory-stored-proc-activity.md) para invocar un procedimiento almacenado desde una canalización de Data Factory. Para más información, consulte el artículo sobre el [conector de SQL Server](data-factory-sqlserver-connector.md#linked-service-properties).

