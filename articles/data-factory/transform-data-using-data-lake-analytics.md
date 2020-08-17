---
title: Transformación de datos mediante un script de U-SQL
description: Aprenda a procesar o transformar datos mediante la ejecución de scripts de U-SQL en el servicio de proceso Azure Data Lake Analytics.
services: data-factory
documentationcenter: ''
ms.author: abnarain
author: nabhishek
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 08/01/2018
ms.openlocfilehash: f99bc5b07f3749fac28f60d28b040a7e0682971b
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86522421"
---
# <a name="process-data-by-running-u-sql-scripts-on-azure-data-lake-analytics"></a>Procesamiento de datos mediante la ejecución de scripts de U-SQL en Azure Data Lake Analytics 
> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](v1/data-factory-usql-activity.md)
> * [Versión actual](transform-data-using-data-lake-analytics.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Una canalización en una factoría de datos de Azure procesa los datos de los servicios de almacenamiento vinculados mediante el uso de servicios de proceso vinculados. Contiene una secuencia de actividades donde cada actividad realiza una operación de procesamiento específica. En este artículo se describe la **actividad U-SQL de Data Lake Analytics** que ejecuta un script de **U-SQL** en un servicio vinculado de proceso de **Azure Data Lake Analytics**. 

Debe crear una cuenta de Azure Data Lake Analytics antes de crear una canalización con una actividad de U-SQL de este servicio. Para obtener más información sobre Azure Data Lake Analytics, consulte el artículo de [introducción a Azure Data Lake Analytics](../data-lake-analytics/data-lake-analytics-get-started-portal.md).


## <a name="azure-data-lake-analytics-linked-service"></a>Servicio vinculado de Azure Data Lake Analytics
Cree un servicio vinculado de **Azure Data Lake Analytics** para vincular un servicio de proceso de Azure Data Lake Analytics a una instancia de Azure Data Factory. La actividad de U-SQL de Data Lake Analytics de la canalización hace referencia a este servicio vinculado. 

En la siguiente tabla se ofrecen descripciones de las propiedades genéricas que se usan en la definición de JSON. 

| Propiedad                 | Descripción                              | Obligatorio                                 |
| ------------------------ | ---------------------------------------- | ---------------------------------------- |
| **type**                 | La propiedad type se debe establecer en: **AzureDataLakeAnalytics**. | Sí                                      |
| **accountName**          | Nombre de la cuenta de Análisis de Azure Data Lake  | Sí                                      |
| **dataLakeAnalyticsUri** | Identificador URI de Análisis de Azure Data Lake.           | No                                       |
| **subscriptionId**       | Identificador de suscripción de Azure                    | No                                       |
| **resourceGroupName**    | Nombre del grupo de recursos de Azure                | No                                       |

### <a name="service-principal-authentication"></a>Autenticación de entidad de servicio
El servicio vinculado de Azure Data Lake Analytics requiere una autenticación de entidad de servicio para conectarse al servicio Azure Data Lake Analytics. Para usar la autenticación de la entidad de servicio, registre una entidad de aplicación en Azure Active Directory (Azure AD) y concédale acceso a Data Lake Analytics y al almacén de Data Lake Store que utiliza. Consulte [Autenticación entre servicios](../data-lake-store/data-lake-store-authenticate-using-active-directory.md) para ver los pasos detallados. Anote los siguientes valores; los usará para definir el servicio vinculado:

* Identificador de aplicación
* Clave de la aplicación 
* Id. de inquilino

Conceda permiso de entidad de seguridad de servicio a su instancia de Azure Data Lake Analytics mediante el [Asistente para agregar usuarios](../data-lake-analytics/data-lake-analytics-manage-use-portal.md#add-a-new-user).

Para usar la autenticación de la entidad de servicio, especifique las siguientes propiedades:

| Propiedad                | Descripción                              | Obligatorio |
| :---------------------- | :--------------------------------------- | :------- |
| **servicePrincipalId**  | Especifique el id. de cliente de la aplicación.     | Sí      |
| **servicePrincipalKey** | Especifique la clave de la aplicación.           | Sí      |
| **tenant**              | Especifique la información del inquilino (nombre de dominio o identificador de inquilino) en el que reside la aplicación. Para recuperarlo, mantenga el puntero del mouse en la esquina superior derecha de Azure Portal. | Sí      |

**Ejemplo: Autenticación de entidad de servicio**
```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "<account name>",
            "dataLakeAnalyticsUri": "<azure data lake analytics URI>",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": {
                "value": "<service principal key>",
                "type": "SecureString"
            },
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<optional, subscription id of ADLA>",
            "resourceGroupName": "<optional, resource group name of ADLA>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }       
    }
}
```

Para más información sobre el servicio vinculado, consulte [Servicios vinculados de Compute](compute-linked-services.md).

## <a name="data-lake-analytics-u-sql-activity"></a>Actividad U-SQL de Análisis de Data Lake
El siguiente fragmento JSON define una canalización con una actividad U-SQL de Análisis de Data Lake. La definición de actividad tiene una referencia al servicio vinculado de Análisis de Azure Data Lake que creó anteriormente. Para ejecutar un script U-SQL de Data Lake Analytics, Data Factory envía el script especificado a Data Lake Analytics y las entradas y salidas necesarias se definen en el script para que Data Lake Analytics realice la captura y resultado. 

```json
{
    "name": "ADLA U-SQL Activity",
    "description": "description",
    "type": "DataLakeAnalyticsU-SQL",
    "linkedServiceName": {
        "referenceName": "<linked service name of Azure Data Lake Analytics>",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "scriptLinkedService": {
            "referenceName": "<linked service name of Azure Data Lake Store or Azure Storage which contains the U-SQL script>",
            "type": "LinkedServiceReference"
        },
        "scriptPath": "scripts\\kona\\SearchLogProcessing.txt",
        "degreeOfParallelism": 3,
        "priority": 100,
        "parameters": {
            "in": "/datalake/input/SearchLog.tsv",
            "out": "/datalake/output/Result.tsv"
        }
    }   
}
```

En la tabla siguiente se describen los nombres y descripciones de las propiedades que son específicas de esta actividad. 

| Propiedad            | Descripción                              | Obligatorio |
| :------------------ | :--------------------------------------- | :------- |
| name                | Nombre de la actividad en la canalización     | Sí      |
| description         | Texto que describe para qué se usa la actividad.  | No       |
| type                | Para la actividad U-SQL de Data Lake Analytics, el tipo de actividad es **DataLakeAnalyticsU-SQL**. | Sí      |
| linkedServiceName   | Servicio vinculado a Azure Data Lake Analytics. Para obtener más información sobre este servicio vinculado, vea el artículo [Compute linked services](compute-linked-services.md) (Servicios vinculados de procesos).  |Sí       |
| scriptPath          | Ruta de acceso a la carpeta que contiene el script U-SQL. El nombre del archivo distingue mayúsculas de minúsculas. | Sí      |
| scriptLinkedService | Servicio vinculado que vincula la instancia de **Azure Data Lake Store** o **Azure Storage** que contiene el script a la factoría de datos. | Sí      |
| degreeOfParallelism | Número máximo de nodos que se usará de forma simultánea para ejecutar el trabajo. | No       |
| priority            | Determina qué trabajos de todos los están en cola deben seleccionarse para ejecutarse primero. Cuanto menor sea el número, mayor será la prioridad. | No       |
| parámetros          | Parámetros para pasar el script de U-SQL.    | No       |
| runtimeVersion      | Versión en tiempo de ejecución del motor de U-SQL que se usará. | No       |
| compilationMode     | <p>Modo de compilación de U-SQL. Debe ser uno de los valores siguientes: **Semantic:** solo realiza comprobaciones semánticas y comprobaciones de integridad necesarias. **Full:** realiza la compilación completa (comprobación de sintaxis, optimización, generación de código, etc.), **SingleBox:** realiza la compilación completa, con la opción TargetType establecida en SingleBox. Si no se especifica ningún valor para esta propiedad, el servidor determina el modo de compilación óptimo. | No |

Para ver la definición del script, consulte [SearchLogProcessing.txt](#sample-u-sql-script). 

## <a name="sample-u-sql-script"></a>Script U-SQL de ejemplo

```
@searchlog =
    EXTRACT UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int,
            Urls            string,
            ClickedUrls     string
    FROM @in
    USING Extractors.Tsv(nullEscape:"#NULL#");

@rs1 =
    SELECT Start, Region, Duration
    FROM @searchlog
WHERE Region == "en-gb";

@rs1 =
    SELECT Start, Region, Duration
    FROM @rs1
    WHERE Start <= DateTime.Parse("2012/02/19");

OUTPUT @rs1   
    TO @out
      USING Outputters.Tsv(quoting:false, dateTimeFormat:null);
```

En el anterior script de ejemplo, la entrada y salida del script se define en los parámetros **\@in** y **\@out**. Data Factory pasa dinámicamente los valores de los parámetros **\@in** y **\@out** del script U-SQL usando la sección "parameters". 

También puede especificar otras propiedades como degreeOfParallelism y priority en la definición de canalización de los trabajos que se ejecutan en el servicio Azure Data Lake Analytics.

## <a name="dynamic-parameters"></a>Parámetros dinámicos
En la definición de canalización de ejemplo, se asignan los parámetros in y out con valores codificados de forma rígida. 

```json
"parameters": {
    "in": "/datalake/input/SearchLog.tsv",
    "out": "/datalake/output/Result.tsv"
}
```

Es posible usar los parámetros dinámicos en su lugar. Por ejemplo: 

```json
"parameters": {
    "in": "/datalake/input/@{formatDateTime(pipeline().parameters.WindowStart,'yyyy/MM/dd')}/data.tsv",
    "out": "/datalake/output/@{formatDateTime(pipeline().parameters.WindowStart,'yyyy/MM/dd')}/result.tsv"
}
```

En este caso, los archivos de entrada se siguen tomando de la carpeta /datalake/input; los de salida se generan en la carpeta /datalake/output. Los nombres de archivo son dinámicos en función del tiempo de inicio de ventana que se pasa cuando se desencadena la canalización.  

## <a name="next-steps"></a>Pasos siguientes
Vea los siguientes artículos, en los que se explica cómo transformar datos de otras maneras: 

* [Actividad de Hive](transform-data-using-hadoop-hive.md)
* [Actividad de Pig](transform-data-using-hadoop-pig.md)
* [Actividad de MapReduce](transform-data-using-hadoop-map-reduce.md)
* [Actividad de streaming de Hadoop](transform-data-using-hadoop-streaming.md)
* [Actividad de Spark](transform-data-using-spark.md)
* [Actividad personalizada de .NET](transform-data-using-dotnet-custom-activity.md)
* [Actividad de ejecución de Batch de Machine Learning](transform-data-using-machine-learning.md)
* [Actividad de procedimiento almacenado](transform-data-using-stored-procedure.md)
