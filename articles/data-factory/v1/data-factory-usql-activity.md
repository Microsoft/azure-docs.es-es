---
title: 'Transformación de datos mediante un script de U-SQL: Azure'
description: Aprenda a procesar o transformar datos mediante la ejecución de scripts de U-SQL en el servicio de proceso Azure Data Lake Analytics.
services: data-factory
documentationcenter: ''
ms.assetid: e17c1255-62c2-4e2e-bb60-d25274903e80
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 10/01/2017
author: nabhishek
ms.author: abnarain
ms.custom: devx-track-csharp
manager: anandsub
robots: noindex
ms.openlocfilehash: a4ae575984badb2b03f72a77aaf580012a1fc002
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88997139"
---
# <a name="transform-data-by-running-u-sql-scripts-on-azure-data-lake-analytics"></a>Transformación de datos mediante la ejecución de scripts de U-SQL en Azure Data Lake Analytics 
> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](data-factory-usql-activity.md)
> * [Versión 2 (versión actual)](../transform-data-using-data-lake-analytics.md)

> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si utiliza la versión actual del servicio Data Factory, consulte [Actividad de U-SQL en V2](../transform-data-using-data-lake-analytics.md).

Una canalización en una factoría de datos de Azure procesa los datos de los servicios de almacenamiento vinculados mediante el uso de servicios de proceso vinculados. Contiene una secuencia de actividades donde cada actividad realiza una operación de procesamiento específica. En este artículo se describe la **actividad U-SQL de Data Lake Analytics** que ejecuta un script de **U-SQL** en un servicio vinculado de proceso de **Azure Data Lake Analytics**. 

Debe crear una cuenta de Azure Data Lake Analytics antes de crear una canalización con una actividad de U-SQL de este servicio. Para obtener más información sobre Azure Data Lake Analytics, consulte el artículo de [introducción a Azure Data Lake Analytics](../../data-lake-analytics/data-lake-analytics-get-started-portal.md).

Revise el [tutorial sobre la compilación de la primera canalización](data-factory-build-your-first-pipeline.md) para ver los pasos detallados para crear una factoría de datos, servicios vinculados, conjuntos de datos y una canalización. Use los fragmentos de código JSON con el Editor de Data Factory, Visual Studio o Azure PowerShell para crear las entidades de Data Factory.

## <a name="supported-authentication-types"></a>Tipos de autenticación que se admiten
La actividad de U-SQL admite siguientes tipos de autenticación frente a Data Lake Analytics:
* Autenticación de entidad de servicio
* Autenticación de credenciales de usuario (OAuth) 

Se recomienda usar la autenticación de la entidad de servicio, en especial para una ejecución de U-SQL. Con la autenticación se credenciales de usuario puede darse la situación de que expiren los tokens. Para información sobre los detalles de configuración, consulte la sección [Propiedades del servicio vinculado](#azure-data-lake-analytics-linked-service).

## <a name="azure-data-lake-analytics-linked-service"></a>Servicio vinculado con el Análisis con Azure Data Lake
Cree un servicio vinculado de **Azure Data Lake Analytics** para vincular un servicio de proceso de Azure Data Lake Analytics a una instancia de Azure Data Factory. La actividad de U-SQL de Data Lake Analytics de la canalización hace referencia a este servicio vinculado. 

En la siguiente tabla se ofrecen descripciones de las propiedades genéricas que se usan en la definición de JSON. Puede elegir entre la autenticación de la entidad de servicio y la autenticación de credenciales de usuario.

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| **type** |La propiedad type se debe establecer en: **AzureDataLakeAnalytics**. |Sí |
| **accountName** |Nombre de la cuenta de Análisis de Azure Data Lake |Sí |
| **dataLakeAnalyticsUri** |Identificador URI de Análisis de Azure Data Lake. |No |
| **subscriptionId** |Identificador de suscripción de Azure |No (si no se especifica, se usa la suscripción de Data Factory). |
| **resourceGroupName** |Nombre del grupo de recursos de Azure |No (si no se especifica, se usa el grupo de recursos de la factoría de datos). |

### <a name="service-principal-authentication-recommended"></a>Autenticación de la entidad de servicio (recomendada)
Para usar la autenticación de la entidad de servicio, registre una entidad de aplicación en Azure Active Directory (AAD) y concédale acceso a Data Lake Store. Consulte [Autenticación entre servicios](../../data-lake-store/data-lake-store-authenticate-using-active-directory.md) para ver los pasos detallados. Anote los siguientes valores; los usará para definir el servicio vinculado:
* Identificador de aplicación
* Clave de la aplicación 
* Id. de inquilino

Para usar la autenticación de la entidad de servicio, especifique las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| **servicePrincipalId** | Especifique el id. de cliente de la aplicación. | Sí |
| **servicePrincipalKey** | Especifique la clave de la aplicación. | Sí |
| **tenant** | Especifique la información del inquilino (nombre de dominio o identificador de inquilino) en el que reside la aplicación. Para recuperarlo, mantenga el puntero del mouse en la esquina superior derecha de Azure Portal. | Sí |

**Ejemplo: autenticación de la entidad de servicio**
```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "adftestaccount",
            "dataLakeAnalyticsUri": "azuredatalakeanalytics.net",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<optional, subscription id of ADLA>",
            "resourceGroupName": "<optional, resource group name of ADLA>"
        }
    }
}
```

### <a name="user-credential-authentication"></a>Autenticación de credenciales de usuario
También puede utilizar la autenticación de credenciales de usuario para Data Lake Analytics mediante la especificación de las propiedades siguientes:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| **authorization** | Haga clic en el botón **Autorizar** de Data Factory Editor y escriba sus credenciales, que asignan la dirección URL de autorización generada automáticamente a esta propiedad. | Sí |
| **sessionId** | Id. de sesión de OAuth de la sesión de autorización de OAuth. Cada id. de sesión es único y solo se puede usar una vez. Esta configuración se genera automáticamente al usar Data Factory Editor. | Sí |

**Ejemplo: autenticación de credenciales de usuario**
```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "adftestaccount",
            "dataLakeAnalyticsUri": "azuredatalakeanalytics.net",
            "authorization": "<authcode>",
            "sessionId": "<session ID>", 
            "subscriptionId": "<optional, subscription id of ADLA>",
            "resourceGroupName": "<optional, resource group name of ADLA>"
        }
    }
}
```

#### <a name="token-expiration"></a>Expiración del token
El código de autorización que se generó al hacer clic en el botón **Autorizar** expira poco tiempo después. Consulte la tabla siguiente para conocer el momento en que expiran los distintos tipos de cuentas de usuario. Puede ver el siguiente mensaje de error cuando el **token de autenticación expira**: Error de operación de credencial: invalid_grant - AADSTS70002: error al validar las credenciales. AADSTS70008: la concesión de acceso proporcionada expiró o se revocó. Id. de seguimiento: d18629e8-af88-43c5-88e3-d8419eb1fca1 Id. de correlación: fac30a0c-6be6-4e02-8d69-a776d2ffefd7 Marca de tiempo: 2015-12-15 21:09:31Z

| Tipo de usuario | Expira después de |
|:--- |:--- |
| Cuentas de usuario NO administradas por Azure Active Directory (@hotmail.com, @live.com, etc.) |12 horas |
| Cuentas de usuario administradas por Azure Active Directory (AAD) |14 días después de la ejecución del último segmento. <br/><br/>Noventa días, si un segmento basado en el servicio vinculado basado en OAuth se ejecuta al menos una vez cada catorce días. |

Para evitar o resolver este error, vuelva a dar la autorización con el botón **Autorizar** cuando el **token expire** e implemente de nuevo el servicio vinculado. También puede generar valores para las propiedades **sessionId** y **authorization** mediante programación, para lo que usará el código siguiente:

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

Para más información sobre las clases de Data Factory que se usan en el código, consulte los temas [AzureDataLakeStoreLinkedService Class](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuredatalakestorelinkedservice.aspx), [AzureDataLakeAnalyticsLinkedService Class](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuredatalakeanalyticslinkedservice.aspx) y [AuthorizationSessionGetResponse Class](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.authorizationsessiongetresponse.aspx). Agregue una referencia a Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms.dll para la clase WindowsFormsWebAuthenticationDialog. 

## <a name="data-lake-analytics-u-sql-activity"></a>Actividad U-SQL de Análisis de Data Lake
El siguiente fragmento JSON define una canalización con una actividad U-SQL de Análisis de Data Lake. La definición de actividad tiene una referencia al servicio vinculado de Análisis de Azure Data Lake que creó anteriormente.   

```json
{
    "name": "ComputeEventsByRegionPipeline",
    "properties": {
        "description": "This is a pipeline to compute events for en-gb locale and date less than 2012/02/19.",
        "activities": 
        [
            {
                "type": "DataLakeAnalyticsU-SQL",
                "typeProperties": {
                    "scriptPath": "scripts\\kona\\SearchLogProcessing.txt",
                    "scriptLinkedService": "StorageLinkedService",
                    "degreeOfParallelism": 3,
                    "priority": 100,
                    "parameters": {
                        "in": "/datalake/input/SearchLog.tsv",
                        "out": "/datalake/output/Result.tsv"
                    }
                },
                "inputs": [
                    {
                        "name": "DataLakeTable"
                    }
                ],
                "outputs": 
                [
                    {
                        "name": "EventsByRegionTable"
                    }
                ],
                "policy": {
                    "timeout": "06:00:00",
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst",
                    "retry": 1
                },
                "scheduler": {
                    "frequency": "Day",
                    "interval": 1
                },
                "name": "EventsByRegion",
                "linkedServiceName": "AzureDataLakeAnalyticsLinkedService"
            }
        ],
        "start": "2015-08-08T00:00:00Z",
        "end": "2015-08-08T01:00:00Z",
        "isPaused": false
    }
}
```

En la tabla siguiente se describen los nombres y descripciones de las propiedades que son específicas de esta actividad. 

| Propiedad            | Descripción                              | Obligatorio                                 |
| :------------------ | :--------------------------------------- | :--------------------------------------- |
| type                | La propiedad type debe establecerse en **DataLakeAnalyticsU-SQL**. | Sí                                      |
| linkedServiceName   | Referencia a la instancia de Azure Data Lake Analytics registrada como servicio vinculado en Data Factory. | Sí                                      |
| scriptPath          | Ruta de acceso a la carpeta que contiene el script U-SQL. El nombre del archivo distingue mayúsculas de minúsculas. | No (si se utiliza el script)                   |
| scriptLinkedService | Servicio vinculado que se vincula al almacenamiento que contiene el script para la factoría de datos | No (si se utiliza el script)                   |
| script              | Especifique el script en línea en lugar de scriptPath y scriptLinkedService. Por ejemplo: `"script": "CREATE DATABASE test"`. | No (si usa scriptPath y scriptLinkedService) |
| degreeOfParallelism | Número máximo de nodos que se usará de forma simultánea para ejecutar el trabajo. | No                                       |
| priority            | Determina qué trabajos de todos los están en cola deben seleccionarse para ejecutarse primero. Cuanto menor sea el número, mayor será la prioridad. | No                                       |
| parámetros          | Parámetros del script SQL U          | No                                       |
| runtimeVersion      | Versión en tiempo de ejecución del motor de U-SQL que se usa | No                                       |
| compilationMode     | <p>Modo de compilación de U-SQL. Debe ser uno de los valores siguientes:</p> <ul><li>**Semantic:** solo realiza comprobaciones semánticas y comprobaciones de integridad necesarias.</li><li>**Full:** realiza la compilación completa (comprobación de sintaxis, optimización, generación de código, etc.).</li><li>**SingleBox:** realiza la compilación completa, con la opción TargetType en SingleBox.</li></ul><p>Si no se especifica ningún valor para esta propiedad, el servidor determina el modo de compilación óptimo. </p> | No                                       |

Para ver la definición del script, vea [Definición del script SearchLogProcessing.txt](#sample-u-sql-script) . 

## <a name="sample-input-and-output-datasets"></a>Conjuntos de datos de entrada y salida de ejemplo
### <a name="input-dataset"></a>Conjunto de datos de entrada
En este ejemplo, los datos de entrada residen en Almacén de Azure Data Lake (archivo SearchLog.tsv en la carpeta de datalake/input). 

```json
{
    "name": "DataLakeTable",
    "properties": {
        "type": "AzureDataLakeStore",
        "linkedServiceName": "AzureDataLakeStoreLinkedService",
        "typeProperties": {
            "folderPath": "datalake/input/",
            "fileName": "SearchLog.tsv",
            "format": {
                "type": "TextFormat",
                "rowDelimiter": "\n",
                "columnDelimiter": "\t"
            }
        },
        "availability": {
            "frequency": "Day",
            "interval": 1
        }
    }
}    
```

### <a name="output-dataset"></a>Conjunto de datos de salida
En este ejemplo, los datos de salida generados por el script U-SQL se almacenan en Almacén de Azure Data Lake (carpeta datalake/output). 

```json
{
    "name": "EventsByRegionTable",
    "properties": {
        "type": "AzureDataLakeStore",
        "linkedServiceName": "AzureDataLakeStoreLinkedService",
        "typeProperties": {
            "folderPath": "datalake/output/"
        },
        "availability": {
            "frequency": "Day",
            "interval": 1
        }
    }
}
```

### <a name="sample-data-lake-store-linked-service"></a>Ejemplo de servicio vinculado de Data Lake Store
Aquí está la definición del servicio vinculado de Azure Data Lake Store de ejemplo que usan los conjuntos de datos de entrada y salida. 

```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
        }
    }
}
```

Consulte el artículo sobre cómo [mover datos a Azure Data Lake Store como origen y destino](data-factory-azure-datalake-connector.md) para ver descripciones de las propiedades JSON. 

## <a name="sample-u-sql-script"></a>Script U-SQL

```
@searchlog =
    EXTRACT UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
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

ADF pasa dinámicamente los valores de los parámetros de **\@entrada** y **\@salida** del script de U-SQL usando la sección "parámetros". Consulte la sección parameters de la definición de canalización.

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
    "in": "$$Text.Format('/datalake/input/{0:yyyy-MM-dd HH:mm:ss}.tsv', SliceStart)",
    "out": "$$Text.Format('/datalake/output/{0:yyyy-MM-dd HH:mm:ss}.tsv', SliceStart)"
}
```

En este caso, los archivos de entrada se siguen tomando de la carpeta /datalake/input; los de salida se generan en la carpeta /datalake/output. Sin embargo, los nombres de archivo son dinámicos según la hora de inicio del segmento.  


