---
title: Cómo agregar fuentes de datos de diferentes orígenes a Metrics Advisor
titleSuffix: Azure Cognitive Services
description: Incorporación de fuentes de datos diferentes a Metrics Advisor
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: ''
ms.topic: conceptual
ms.date: 09/04/2020
ms.author: aahi
ms.openlocfilehash: f9ab340e73ce8d58da63a0089073ac4770bf2d52
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90973382"
---
# <a name="add-data-feeds-from-different-data-sources-to-metrics-advisor"></a>Incorporación de fuentes de datos de diferentes orígenes de datos a Metrics Advisor

Use este artículo para encontrar la configuración y los requisitos para conectar distintos tipos de orígenes de datos a Metrics Advisor. Asegúrese de leer cómo [incorporar los datos](how-tos/onboard-your-data.md) para obtener información sobre los conceptos clave para usar los datos con Metrics Advisor. 

## <a name="supported-authentication-types"></a>Tipos de autenticación que se admiten

| Tipos de autenticación | Descripción |
| ---------------------|-------------|
|**Basic** | Tendrá que ser capaz de proporcionar los parámetros básicos para acceder a los orígenes de datos. Por ejemplo, una cadena de conexión o clave. Los administradores de fuentes de distribución de datos pueden ver estas credenciales. |
| **AzureManagedIdentity** | [Identidades administradas](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview) para recursos de Azure es una característica de Azure Active Directory. Proporciona a los servicios de Azure una identidad de sistema administrada automáticamente en Azure AD. Puede usar la identidad para autenticarse en cualquier servicio que admita la autenticación de Azure AD.|
| **AzureSQLConnectionString**| Almacene la cadena de conexión de AzureSQL como una **entidad de credencial** en Metrics Advisor y úsela directamente cada vez que se incorporen los datos de métricas. Solo los administradores de la entidad de credencial pueden ver estas credenciales, pero permite que los espectadores autorizados creen fuentes de distribución de datos sin necesidad de conocer los detalles de las credenciales. |
| **DataLakeGen2SharedKey**| Almacene la clave de cuenta del lago de datos como una **entidad de credencial** en Metrics Advisor y úsela directamente cada vez que se incorporen los datos de métricas. Solo los administradores de la entidad de credencial pueden ver estas credenciales, pero permite que los espectadores autorizados creen fuentes de distribución de datos sin necesidad de conocer los detalles de las credenciales.|
| **Entidad de servicio**| Almacene la entidad de servicio como una **entidad de credencial** en Metrics Advisor y úsela directamente cada vez que se incorporen los datos de métricas. Solo los administradores de la entidad de credencial pueden ver las credenciales, pero permite que los espectadores autorizados creen fuentes de distribución de datos sin necesidad de conocer los detalles de las credenciales.|
| **Entidad de servicio de Key Vault**|Almacene la entidad de servicio en KeyVault como una **entidad de credencial** en Metrics Advisor y úsela directamente cada vez que se incorporen los datos de métricas. Solo los administradores de la **entidad de credencial** pueden ver las credenciales, pero también deja que los espectadores puedan crear fuentes de distribución de datos sin necesidad de conocer las credenciales detalladas. |

## <a name="data-sources-supported-and-corresponding-authentication-types"></a>Orígenes de datos admitidos y tipos de autenticación correspondientes


| Orígenes de datos | Tipos de autenticación |
|-------------| ---------------------|
|[**Azure Application Insights**](#appinsights)|  Básico |
|[**Azure Blob Storage (JSON)** ](#blob) | Básico<br>ManagedIdentity|
|[**Azure Cosmos DB (SQL)** ](#cosmosdb) | Básico |
|[**Azure Data Explorer (Kusto)**](#kusto) | Básico<br>ManagedIdentity|
|[**Azure Data Lake Storage Gen2**](#adl) | Básico<br>DataLakeGen2SharedKey<br>Entidad de servicio<br>Entidad de servicio de Key Vault<br> |
|[**Azure SQL Database/SQL Server**](#sql) | Básico<br>ManagedIdentity<br>Entidad de servicio<br>Entidad de servicio de Key Vault<br>AzureSQLConnectionString
|[**Azure Table Storage**](#table) | Básico | 
|[**ElasticSearch**](#es) | Básico |
|[**Solicitud Http**](#http) | Básico | 
|[**InfluxDB (InfluxQL)** ](#influxdb) | Básico |
|[**MongoDB**](#mongodb) | Básico |
|[**MySQL**](#mysql) | Básico |
|[**PostgreSQL**](#pgsql)| Básico|

Cree una **entidad de credencial** y úsela para autenticarse en los orígenes de datos. En las secciones siguientes se especifican los parámetros necesarios para la autenticación *básica*. 

## <a name="span-idappinsightsazure-application-insightsspan"></a><span id="appinsights">Azure Application Insights</span>

* **Identificador de aplicación**: Se usa para identificar esta aplicación cuando se utiliza la API de Application Insights. Para obtener el id. de la aplicación, haga lo siguiente:

    1. Desde el recurso de Application Insights, haga clic en Acceso de API.

    2. Copie el identificador de aplicación generado en el campo **Id. de la aplicación** en Metrics Advisor. 
    
    Consulte la [documentación de Azure Bot Service](https://docs.microsoft.com/azure/bot-service/bot-service-resources-app-insights-keys#application-id) para más información.

* **Clave de API**: Las aplicaciones de fuera del explorador usan la claves de API para acceder a este recurso. Para obtener la clave de API, haga lo siguiente:

    1. Desde el recurso de Application Insights, haga clic en Acceso de API.

    2. Haga clic en Crear clave de API.

    3. Escriba una descripción breve, active la opción Leer telemetría y haga clic en el botón Generar clave.

    4. Copie la clave de API en el campo **Clave de la API** en Metrics Advisor.

* **Consultar** Los registros de Azure Application Insights se crean en Azure Data Explorer y las consultas de registros de Azure Monitor usan una versión del mismo lenguaje de consulta de Kusto. La [documentación del lenguaje de consulta de Kusto](https://docs.microsoft.com/azure/data-explorer/kusto/query/) incluye todos los detalles del lenguaje y debe ser el principal recurso para escribir una consulta en Application Insights. 


## <a name="span-idblobazure-blob-storage-jsonspan"></a><span id="blob">Azure Blob Storage (JSON)</span>

* **Cadena de conexión**: Consulte el artículo sobre la [cadena de conexión](https://docs.microsoft.com/azure/storage/common/storage-configure-connection-string#view-and-copy-a-connection-string) de Azure Blob Storage para obtener información sobre cómo recuperar esta cadena.

* **Contenedor**: Metrics Advisor espera datos de series temporales almacenados como archivos de blob (un blob por marca de tiempo) en un solo contenedor. Este es el campo de nombre de contenedor.

* **Plantilla de blob**: Esta es la plantilla de los nombres de archivo de blob. Por ejemplo: `/%Y/%m/X_%Y-%m-%d-%h-%M.json`. Se admiten los siguientes parámetros:
  * `%Y` es el año con formato `yyyy`
  * `%m` es el mes con formato `MM`
  * `%d` es el día con formato `dd`
  * `%h` es la hora con formato `HH`
  * `%M` es el minuto con formato `mm`

* **Versión de formato JSON**: Define el esquema de datos en los archivos JSON. Actualmente, Metrics Advisor admite dos versiones:
  
  * v1 (valor predeterminado)

      Solo se aceptan las métricas *Nombre* y *Valor*. Por ejemplo:
    
      ``` JSON
      {"count":11, "revenue":1.23}
      ```

  * v2

      También se aceptan las métricas *Dimensiones* y *Marca de tiempo*. Por ejemplo:
      
      ``` JSON
      [
        {"date": "2018-01-01T00:00:00Z", "market":"en-us", "count":11, "revenue":1.23},
        {"date": "2018-01-01T00:00:00Z", "market":"zh-cn", "count":22, "revenue":4.56}
      ]
      ```

Solo se permite una marca de tiempo por cada archivo JSON. 

## <a name="span-idcosmosdbazure-cosmos-db-sqlspan"></a><span id="cosmosdb">Azure Cosmos DB (SQL)</span>

* **Cadena de conexión**: Cadena de conexión para acceder a Azure Cosmos DB. Esto se puede encontrar en el recurso Cosmos DB, en **Claves**. 
* **Base de datos**: Base de datos que se va a consultar. Esto se puede encontrar en la página **Examinar** en la sección **Contenedores**.
* **Id. de colección**: Identificador de la colección con el que se va a realizar la consulta. Esto se puede encontrar en la página **Examinar** en la sección **Contenedores**.
* **Consulta SQL**: Consulta SQL para obtener y formular datos en datos de series temporales multidimensionales. Puede usar las variables `@StartTime` y `@EndTime` en la consulta. Deben tener el formato `yyyy-MM-dd HH:mm:ss`.

    Consulta de ejemplo:
    
    ``` mssql
    select StartDate, JobStatusId, COUNT(*) AS JobNumber from IngestionJobs WHERE and StartDate = @StartTime
    ```
    
    Consulta de ejemplo para un segmento de datos de 2019/12/12:
    
    ``` mssql
    select StartDate, JobStatusId, COUNT(*) AS JobNumber from IngestionJobs WHERE and StartDate = '2019-12-12 00:00:00'
    ```

## <a name="span-idkustoazure-data-explorer-kustospan"></a><span id="kusto">Azure Data Explorer (Kusto)</span>

* **Cadena de conexión**: Consulte [Ver y copiar una cadena de conexión](https://docs.microsoft.com/azure/data-explorer/kusto/api/connection-strings/kusto) para obtener información sobre cómo recuperar la cadena de conexión de Azure Data Explorer (Kusto).

* **Consultar** Consulte [Lenguaje de consulta Kusto](https://docs.microsoft.com/azure/data-explorer/kusto/query) para obtener y formular datos en datos de series temporales multidimensionales. Puede usar las variables `@StartTime` y `@EndTime` en la consulta. Deben tener el formato `yyyy-MM-dd HH:mm:ss`.

## <a name="span-idadlazure-data-lake-storage-gen2span"></a><span id="adl">Azure Data Lake Storage Gen2</span>

* **Nombre de cuenta**: Nombre de la cuenta de Azure Data Lake Storage Gen2. Se puede encontrar en el recurso de la cuenta de Azure Storage (Azure Data Lake Storage Gen2) en **Claves de acceso**.

* **Clave de cuenta**: Especifique el nombre de la cuenta para acceder a Azure Data Lake Storage Gen2. Se podría encontrar en el recurso de la cuenta de Azure Storage (Azure Data Lake Storage Gen2) en **Claves de acceso**.

* **Nombre del sistema de archivos (contenedor)** : Metrics Advisor esperará los datos de series temporales almacenados como archivos de blob (un blob por marca de tiempo) en un solo contenedor. Este es el campo de nombre de contenedor. Se puede encontrar en la instancia de la cuenta de almacenamiento de Azure (Azure Data Lake Storage Gen2) y haciendo clic en "Contenedores" en la sección "Blob service".

* **Plantilla de directorio**: Esta es la plantilla de directorio del archivo de blob. Por ejemplo: */%Y/%m/%d*. Se admiten los siguientes parámetros:
  * `%Y` es el año con formato `yyyy`
  * `%m` es el mes con formato `MM`
  * `%d` es el día con formato `dd`
  * `%h` es la hora con formato `HH`
  * `%M` es el minuto con formato `mm`

* **Plantilla de archivo**: Esta es la plantilla de archivo del archivo de blob. Por ejemplo: *X_%Y-%m-%d-%h-%M.json*. Se admiten los siguientes parámetros:
  * `%Y` es el año con formato `yyyy`
  * `%m` es el mes con formato `MM`
  * `%d` es el día con formato `dd`
  * `%h` es la hora con formato `HH`
  * `%M` es el minuto con formato `mm`

Actualmente, Metrics Advisor admite el esquema de datos en los archivos JSON de la manera siguiente. Por ejemplo:

``` JSON
[
  {"date": "2018-01-01T00:00:00Z", "market":"en-us", "count":11, "revenue":1.23},
  {"date": "2018-01-01T00:00:00Z", "market":"zh-cn", "count":22, "revenue":4.56}
]
```
<!--
## <span id="eventhubs">Azure Event Hubs</span>

* **Connection String**: This can be found in 'Shared access policies' in your Event Hubs instance. Also for the 'EntityPath', it could be found by clicking into your Event Hubs instance and clicking at 'Event Hubs' in 'Entities' blade. Items that listed can be input as EntityPath. 

* **Consumer Group**: A [consumer group](https://docs.microsoft.com/azure/event-hubs/event-hubs-features#consumer-groups) is a view (state, position, or offset) of an entire event hub.
Event Hubs use the latest offset of a consumer group to consume (subscribe from) the data from data source. Therefore a dedicated consumer group should be created for one data feed in your Metrics Advisor instance.

* **Timestamp**: Metrics Advisor uses the Event Hubs timestamp as the event timestamp if the user data source does not contain a timestamp field.
The timestamp field must match one of these two formats:

    * "YYYY-MM-DDTHH:MM:SSZ" format;

    * Number of seconds or milliseconds from the epoch of 1970-01-01T00:00:00Z.

    No matter which timestamp field it left aligns to granularity.For example, if timestamp is "2019-01-01T00:03:00Z", granularity is 5 minutes, then Metrics Advisor aligns the timestamp to "2019-01-01T00:00:00Z". If the event timestamp is "2019-01-01T00:10:00Z",  Metrics Advisor uses the timestamp directly without any alignment.
-->
## <a name="span-idsqlazure-sql-database--sql-serverspan"></a><span id="sql">Azure SQL Database | SQL Server</span>

* **Cadena de conexión**: Metrics Advisor acepta una [cadena de conexión de estilo ADO.NET](https://docs.microsoft.com/dotnet/framework/data/adonet/connection-string-syntax) para el origen de datos de SQL Server.

    Cadena de conexión de ejemplo:

    ```
    Data Source=db-server.database.windows.net:[port];initial catalog=[database];User ID=[username];Password=[password];Connection Timeout=10ms;
    ```

* **Consultar** Consulta SQL para obtener y formular datos en datos de series temporales multidimensionales. Puede usar una variable `@StartTime` en la consulta para ayudar a obtener el valor de las métricas esperado.

  * `@StartTime`: un valor de fecha y hora con el formato `yyyy-MM-dd HH:mm:ss`

    Consulta de ejemplo:
    
    ``` mssql
    select StartDate, JobStatusId, COUNT(*) AS JobNumber from IngestionJobs WHERE and StartDate = @StartTime
    ```
    
    Consulta real ejecutada para el segmento de datos de 2019/12/12:
    
    ``` mssql
    select StartDate, JobStatusId, COUNT(*) AS JobNumber from IngestionJobs WHERE and StartDate = '2019-12-12 00:00:00'
    ```

## <a name="span-idtableazure-table-storagespan"></a><span id="table">Azure Table Storage</span>

* **Cadena de conexión**: Consulte [Ver y copiar una cadena de conexión](https://docs.microsoft.com/azure/storage/common/storage-account-keys-manage?toc=%2Fazure%2Fstorage%2Ftables%2Ftoc.json&tabs=azure-portal#view-account-access-keys) para obtener información sobre cómo recuperar la cadena de conexión de Azure Table Storage.

* **Nombre de la tabla**: Especifique una tabla en la que realizar la consulta. Se puede encontrar en la instancia de la cuenta de Azure Storage. Haga clic en **Tablas** en la sección **Table Service**.

* **Consulta**: puede usar `@StartTime` en la consulta. `@StartTime` se reemplaza por una cadena con formato aaaa-MM-ddTHH:mm:ss en el script.

    ``` mssql
    let StartDateTime = datetime(@StartTime); let EndDateTime = StartDateTime + 1d; 
    SampleTable | where Timestamp >= StartDateTime and Timestamp < EndDateTime | project Timestamp, Market, RPM
    ```

## <a name="span-ideselasticsearchspan"></a><span id="es">Elasticsearch</span>

* **Host**: especifique el host principal del clúster de Elasticsearch.
* **Puerto**: especifique el puerto principal del clúster de Elasticsearch.
* **Encabezado de autorización**: especifique el valor del encabezado de autorización del clúster de Elasticsearch.
* **Consulta**: especifique la consulta para obtener datos. Se admite el marcador de posición @StartTime (por ejemplo, cuando se ingieren los datos de 2020-06-21T00:00:00Z, @StartTime = 2020-06-21T00:00:00)

## <a name="span-idhttphttp-requestspan"></a><span id="http">Solicitud HTTP</span>

* **URL de la solicitud**: Una dirección URL HTTP que puede devolver un JSON. Se admiten los marcadores de posición %Y,%m,%d,%h,%M: %Y=año en formato aaaa, %m=mes en formato MM, %d=día en formato dd, %h=hora en formato HH, % M=minuto en formato mm. Por ejemplo: `http://microsoft.com/ProjectA/%Y/%m/X_%Y-%m-%d-%h-%M`.
* **Método HTTP de solicitud**: Use GET o POST.
* **Encabezado de solicitud**: Podría agregar la autenticación básica. 
* **Carga de la solicitud**: Solo se admite la carga de JSON. El marcador de posición @StartTime se admite en la carga. La respuesta debe tener el siguiente formato JSON: [{"timestamp": "2018-01-01T00:00:00Z", "market":"en-us", "count":11, "revenue":1.23}, {"timestamp": "2018-01-01T00:00:00Z", "market":"zh-cn", "count":22, "revenue":4.56}]. (Por ejemplo, cuando se ingieren los datos de 2020-06-21T00:00:00Z, @StartTime = 2020-06-21T00:00:00.0000000+00:00)

## <a name="span-idinfluxdbinfluxdb-influxqlspan"></a><span id="influxdb">InfluxDB (InfluxQL)</span>

* **Cadena de conexión**: Cadena de conexión para acceder a InfluxDB.
* **Base de datos**: Base de datos que se va a consultar.
* **Consultar** Consulta para obtener y formular datos en datos de series temporales multidimensionales para la ingesta.
* **Nombre de usuario**: Opcional para la autenticación. 
* **Contraseña**: Opcional para la autenticación. 

## <a name="span-idmongodbmongodbspan"></a><span id="mongodb">MongoDB</span>

* **Cadena de conexión**: Cadena de conexión para acceder a MongoDB.
* **Base de datos**: Base de datos que se va a consultar.
* **Comando**: Comando para obtener y formular datos en datos de series temporales multidimensionales para la ingesta.

## <a name="span-idmysqlmysqlspan"></a><span id="mysql">MySQL</span>

* **Cadena de conexión**: Cadena de conexión para acceder a la base de datos de MySQL.
* **Consultar** Consulta para obtener y formular datos en datos de series temporales multidimensionales para la ingesta.

## <a name="span-idpgsqlpostgresqlspan"></a><span id="pgsql">PostgreSQL</span>

* **Cadena de conexión**: Cadena de conexión para acceder a la base de datos de PostgreSQL.
* **Consultar** Consulta para obtener y formular datos en datos de series temporales multidimensionales para la ingesta.

## <a name="next-steps"></a>Pasos siguientes

* Mientras espera a que se ingieran los datos de métricas en el sistema, lea sobre [cómo administrar las configuraciones de fuentes de distribución de datos](how-tos/manage-data-feeds.md).
* Cuando se ingieren los datos de métricas, puede [configurar métricas y ajustar la configuración de detección de forma precisa](how-tos/configure-metrics.md).