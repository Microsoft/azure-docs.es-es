---
title: Referencia de datos de supervisión de Azure Storage | Microsoft Docs
description: Referencia de registro y métricas para la supervisión de datos de Azure Storage.
author: normesta
services: azure-monitor
ms.service: azure-monitor
ms.topic: reference
ms.date: 05/01/2020
ms.author: normesta
ms.subservice: logs
ms.custom: monitoring
ms.openlocfilehash: 28a127b4debeacd2562867008bc594897558d50d
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87446834"
---
# <a name="azure-storage-monitoring-data-reference"></a>Referencia de datos de supervisión de Azure Storage

Consulte [Supervisión de Azure Storage](monitor-storage.md) para más información sobre la recopilación y el análisis de datos de supervisión de Azure Storage.

## <a name="metrics"></a>Métricas

En las tablas siguientes se indican las métricas de plataforma recopiladas de Azure Storage. 

### <a name="capacity-metrics"></a>Métricas de capacidad

Los valores de las métricas de capacidad se envían a Azure Monitor cada hora. Los valores se actualizan a diario. El intervalo de agregación define el intervalo de tiempo para el que se presentan los valores de las métricas. El intervalo de agregación que se admite en las métricas de capacidad es una hora (PT1H).

Azure Storage proporciona las siguientes métricas de capacidad en Azure Monitor.

#### <a name="account-level"></a>Nivel de cuenta

En esta tabla se muestran [métricas de nivel de cuenta](https://docs.microsoft.com/azure/azure-monitor/platform/metrics-supported#microsoftstoragestorageaccounts).

| Métrica | Descripción |
| ------------------- | ----------------- |
| UsedCapacity | La cantidad de almacenamiento que utiliza la cuenta de almacenamiento. En las cuentas de almacenamiento estándar, es la suma de la capacidad usada por un blob, una tabla, un archivo y una cola. Tanto en las cuentas de almacenamiento Premium como en las cuentas de Blob Storage, coincide con BlobCapacity. <br/><br/> Unidad: Bytes <br/> Tipo de agregación: Average <br/> Ejemplo de valor: 1024 |

#### <a name="blob-storage"></a>Blob Storage

En esta tabla se muestran [métricas de Blob Storage](https://docs.microsoft.com/azure/azure-monitor/platform/metrics-supported#microsoftstoragestorageaccountsblobservices).

| Métrica | Descripción |
| ------------------- | ----------------- |
| BlobCapacity | El total de almacenamiento de blobs que se utiliza en la cuenta de almacenamiento. <br/><br/> Unidad: Bytes <br/> Tipo de agregación: Average <br/> Ejemplo de valor: 1024 <br/> Dimensiones: **BlobType** y **BlobTier** ([Definition](#metrics-dimensions)) |
| BlobCount    | El número de objetos BLOB almacenados en la cuenta de almacenamiento. <br/><br/> Unidad: Count <br/> Tipo de agregación: Average <br/> Ejemplo de valor: 1024 <br/> Dimensiones: **BlobType** y **BlobTier** ([Definition](#metrics-dimensions)) |
| BlobProvisionedSize | Cantidad de almacenamiento aprovisionado en la cuenta de Storage. Esta métrica solo se puede aplicar a las cuentas de Storage Premium. <br/><br/> Unidad: bytes <br/> Tipo de agregación: Average |
| ContainerCount    | El número de contenedores que hay en la cuenta de almacenamiento. <br/><br/> Unidad: Count <br/> Tipo de agregación: Average <br/> Ejemplo de valor: 1024 |
| IndexCapacity     | Cantidad de almacenamiento que usa el índice jerárquico de ADLS Gen2. <br/><br/> Unidad: Bytes <br/> Tipo de agregación: Average <br/> Ejemplo de valor: 1024 |

#### <a name="table-storage"></a>Almacenamiento de tablas

En esta tabla se muestran [métricas de almacenamiento de tablas](https://docs.microsoft.com/azure/azure-monitor/platform/metrics-supported#microsoftstoragestorageaccountstableservices).

| Métrica | Descripción |
| ------------------- | ----------------- |
| TableCapacity | La cantidad de almacenamiento de tablas que utiliza la cuenta de almacenamiento. <br/><br/> Unidad: Bytes <br/> Tipo de agregación: Average <br/> Ejemplo de valor: 1024 |
| TableCount   | El número de tablas de la cuenta de almacenamiento. <br/><br/> Unidad: Count <br/> Tipo de agregación: Average <br/> Ejemplo de valor: 1024 |
| TableEntityCount | El número de entidades de tabla que hay en la cuenta de almacenamiento. <br/><br/> Unidad: Count <br/> Tipo de agregación: Average <br/> Ejemplo de valor: 1024 |

#### <a name="queue-storage"></a>Queue Storage

En esta tabla se muestran [métricas de Queue Storage](https://docs.microsoft.com/azure/azure-monitor/platform/metrics-supported#microsoftstoragestorageaccountsfileservices).

| Métrica | Descripción |
| ------------------- | ----------------- |
| QueueCapacity | La cantidad de almacenamiento de colas que utiliza la cuenta de almacenamiento. <br/><br/> Unidad: Bytes <br/> Tipo de agregación: Average <br/> Ejemplo de valor: 1024 |
| QueueCount   | El número de colas que hay en la cuenta de almacenamiento. <br/><br/> Unidad: Count <br/> Tipo de agregación: Average <br/> Ejemplo de valor: 1024 |
| QueueMessageCount | Número aproximado de mensajes de cola en la instancia de Queue service de la cuenta de almacenamiento. <br/><br/>Unidad: Count <br/> Tipo de agregación: Average <br/> Ejemplo de valor: 1024 |

#### <a name="file-storage"></a>File Storage

En esta tabla se muestran [métricas de File Storage](https://docs.microsoft.com/azure/azure-monitor/platform/metrics-supported#microsoftstoragestorageaccountsqueueservices).

| Métrica | Descripción |
| ------------------- | ----------------- |
| FileCapacity | La cantidad de almacenamiento de archivos que utiliza la cuenta de almacenamiento. <br/><br/> Unidad: Bytes <br/> Tipo de agregación: Average <br/> Ejemplo de valor: 1024 |
| FileCount   | El número de archivos que hay en la cuenta de almacenamiento. <br/><br/> Unidad: Count <br/> Tipo de agregación: Average <br/> Ejemplo de valor: 1024 |
| FileShareCount | El número de recursos compartidos de archivo que hay en la cuenta de almacenamiento. <br/><br/> Unidad: Count <br/> Tipo de agregación: Average <br/> Ejemplo de valor: 1024 |
| FileShareProvisionedIOPS | El número de IOPS aprovisionadas en un recurso compartido de archivos. Esta métrica solo se puede aplicar a un almacenamiento de archivos Premium. <br/><br/> Unidad: bytes <br/> Tipo de agregación: Average |

### <a name="transaction-metrics"></a>Métricas de transacciones

Se emiten en todas las solicitudes enviadas a una cuenta de almacenamiento de Azure Storage a Azure Monitor. En caso de que no haya actividad en la cuenta de almacenamiento, no habrá ningún dato en las métricas de transacciones del período. Todas las métricas de transacciones están disponibles a nivel tanto de cuenta como de servicio (Blob Storage, Table Storage, Azure Files y Queue Storage). El intervalo de agregación define el intervalo de tiempo en el que se presentan los valores de las métricas. Los intervalos de agregación compatible para todas las métricas de transacciones son PT1H y PT1M.

Azure Storage proporciona las siguientes métricas de transacciones en Azure Monitor.

| Métrica | Descripción |
| ------------------- | ----------------- |
| Transacciones | El número de solicitudes realizadas a un servicio de almacenamiento o la operación de API especificada. Este número incluye solicitudes correctas y con errores, así como las solicitudes que generaron errores. <br/><br/> Unidad: Count <br/> Tipo de agregación: Total <br/> Dimensiones aplicables: ResponseType, GeoType, ApiName y Authentication ([Definición](#metrics-dimensions))<br/> Ejemplo de valor: 1024 |
| Entrada | La cantidad de datos de entrada. Este número incluye la entradas desde un cliente externo en Azure Storage, así como la entrada dentro de Azure. <br/><br/> Unidad: Bytes <br/> Tipo de agregación: Total <br/> Dimensiones aplicables: GeoType, ApiName y Authentication ([definición](#metrics-dimensions)) <br/> Ejemplo de valor: 1024 |
| Salida | La cantidad de datos de salida. Este número incluye la salida de un cliente externo en Azure Storage, así como la salida dentro de Azure. En consecuencia, este número no refleja las salidas facturables. <br/><br/> Unidad: Bytes <br/> Tipo de agregación: Total <br/> Dimensiones aplicables: GeoType, ApiName y Authentication ([definición](#metrics-dimensions)) <br/> Ejemplo de valor: 1024 |
| SuccessServerLatency | El tiempo medio que se usa para que Azure Storage procese una solicitud correcta . Este valor no incluye la latencia de red especificada en SuccessE2ELatency. <br/><br/> Unidad: Milisegundos <br/> Tipo de agregación: Average <br/> Dimensiones aplicables: GeoType, ApiName y Authentication ([definición](#metrics-dimensions)) <br/> Ejemplo de valor: 1024 |
| SuccessE2ELatency | La latencia media de un extremo a otro de las solicitudes correctas realizadas a un servicio de almacenamiento o a la operación de API especificada. Este valor incluye el tiempo de procesamiento requerido dentro de Azure Storage para leer la solicitud, enviar la respuesta y recibir la confirmación de la respuesta. <br/><br/> Unidad: Milisegundos <br/> Tipo de agregación: Average <br/> Dimensiones aplicables: GeoType, ApiName y Authentication ([definición](#metrics-dimensions)) <br/> Ejemplo de valor: 1024 |
| Disponibilidad | El porcentaje de disponibilidad para el servicio de almacenamiento o la operación de API especificada. Para calcular la disponibilidad hay que tomar el número total de solicitudes facturables y dividirlo por el número de solicitudes aplicables, incluidas las solicitudes que generaron errores inesperados. Todos los errores inesperados reducen la disponibilidad del servicio de almacenamiento o de la operación de API especificada. <br/><br/> Unidad: Percent <br/> Tipo de agregación: Average <br/> Dimensiones aplicables: GeoType, ApiName y Authentication ([definición](#metrics-dimensions)) <br/> Ejemplo de valor: 99,99 |

<a id="metrics-dimensions"></a>

## <a name="metrics-dimensions"></a>Dimensiones de métricas

Azure Storage admite los siguientes dimensiones para las métricas en Azure Monitor.

| Nombre de dimensión | Descripción |
| ------------------- | ----------------- |
| **BlobType** | Solo el tipo de blob de las métricas de Blob. Los valores admitidos son **BlockBlob**, **PageBlob** y **Azure Data Lake Storage**. Los blobs en anexos se incluyen en **BlockBlob**. |
| **BlobTier** | Azure Storage ofrece distintos niveles de acceso que le permiten almacenar datos de objeto de blob de la manera más rentable. Obtenga más información en [Nivel de blob de Azure Storage](../blobs/storage-blob-storage-tiers.md). Entre los valores admitidos se incluyen: <br/> <li>**Acceso frecuente**: nivel de acceso frecuente</li> <li>**Acceso esporádico**: nivel de acceso esporádico</li> <li>**Archivo**: nivel de archivo</li> <li>**Premium**: nivel Premium para blob en bloques</li> <li>**P4/P6/P10/P15/P20/P30/P40/P50/P60**: tipos de nivel para blob en páginas premium</li> <li>**Estándar**: tipo de nivel para blob en páginas estándar</li> <li>**Sin nivel**: Tipo de nivel para cuentas de almacenamiento de uso general v1</li> |
| **GeoType** | Transacción de clúster principal o secundario. Los valores disponibles incluyen **Principal** y **Secundario**. Se aplica al Almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) al leer objetos de un inquilino secundario. |
| **ResponseType** | Tipo de respuesta de la transacción. Los valores disponibles son: <br/><br/> <li>**ServerOtherError**: todos los errores del servidor, excepto los descritos. </li> <li>**ServerBusyError**: solicitud autenticada que devolvió un código de estado HTTP 503. </li> <li>**ServerTimeoutError**: solicitud autenticada que ha superado el tiempo de espera y que devolvió un código de estado HTTP 500. El tiempo de espera se superó debido a un error del servidor. </li> <li>**AuthorizationError**: solicitud autenticada errónea debido a un acceso no autorizado de los datos o a un error de autorización. </li> <li>**NetworkError**: solicitud autenticada errónea debido a errores de red. Normalmente se produce cuando un cliente cierra prematuramente una conexión antes de que se haya superado el tiempo de expiración. </li><li>**ClientAccountBandwidthThrottlingError**: la solicitud está limitada por el ancho de banda para superar los [límites de escalabilidad de la cuenta de almacenamiento ](scalability-targets-standard-account.md).</li><li>**ClientAccountRequestThrottlingError**: la solicitud está limitada por la tasa de solicitud para superar los [límites de escalabilidad de la cuenta de almacenamiento ](scalability-targets-standard-account.md).<li>**ClientThrottlingError**: Otro error de limitación del lado del cliente. ClientAccountBandwidthThrottlingError y ClientAccountRequestThrottlingError se excluyen.</li> <li>**ClientTimeoutError**: solicitud autenticada que ha superado el tiempo de espera y que devolvió un código de estado HTTP 500. Si el tiempo de expiración de la red del cliente o el tiempo de expiración de la solicitud se establece en un valor menor de lo que espera el servicio de almacenamiento, es un tiempo de expiración esperado. De lo contrario, se notifica como un error ServerTimeoutError. </li> <li>**ClientOtherError**: todos los errores del lado cliente, excepto los descritos. </li> <li>**Correcto**: solicitud correcta.</li> <li> **SuccessWithThrottling**: solicitud correcta cuando un cliente SMB obtiene una limitación en el primer intento, pero se ejecuta correctamente después de varios reintentos.</li> |
| **ApiName** | El nombre de la operación. Por ejemplo: <br/> <li>**CreateContainer**</li> <li>**DeleteBlob**</li> <li>**GetBlob**</li> Para ver los nombres de todas las operaciones, consulte este [documento](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages). |
| **Autenticación** | Tipos de autenticación que se usan en las transacciones. Los valores disponibles son: <br/> <li>**AccountKey**: la transacción se autentica con la clave de la cuenta de almacenamiento.</li> <li>**SAS**: la transacción se autentica con firmas de acceso compartido.</li> <li>**OAuth**: la transacción se autentica con tokens de acceso de OAuth.</li> <li>**Anonymous**: la transacción se solicita de forma anónima. No incluye las solicitudes preparatorias.</li> <li>**AnonymousPreflight**: la transacción es una solicitud preparatoria.</li> |

En el caso de las dimensiones compatibles con métricas, es preciso especificar el valor de la dimensión para ver los valores correspondientes de las métricas. Por ejemplo, si observa el valor **Transactions** en las respuestas correctas, es preciso filtrar la dimensión **ResponseType** por **Success**. O bien si examina el valor **BlobCount** de Blob en bloques, es preciso filtrar la dimensión **BlobType** por **BlockBlob**.

## <a name="resource-logs-preview"></a>Registros de recursos (versión preliminar)

> [!NOTE]
> Los registros de Azure Storage en Azure Monitor están en versión preliminar pública, además de estar disponibles para pruebas de versión preliminar en todas las regiones de la nube pública. Para inscribirse en la versión preliminar, visite [esta página](https://forms.microsoft.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRxW65f1VQyNCuBHMIMBV8qlUM0E0MFdPRFpOVTRYVklDSE1WUTcyTVAwOC4u).  Esta versión preliminar habilita los registros de blobs (incluido Azure Data Lake Storage Gen2), archivos, colas, tablas, cuentas de almacenamiento Premium en cuentas de almacenamiento de uso general v1 y v2. Las cuentas de almacenamiento clásico no son compatibles.

En la tabla siguiente se indican las propiedades de los registros de recursos de Azure Storage cuando se recopilan en registros de Azure Monitor o Azure Storage. Las propiedades describen la operación, el servicio y el tipo de autorización que se ha usado para realizar la operación.

### <a name="fields-that-describe-the-operation"></a>Campos que describen la operación

```json
{
    "time": "2019-02-28T19:10:21.2123117Z",
    "resourceId": "/subscriptions/12345678-2222-3333-4444-555555555555/resourceGroups/mytestrp/providers/Microsoft.Storage/storageAccounts/testaccount1/blobServices/default",
    "category": "StorageWrite",
    "operationName": "PutBlob",
    "operationVersion": "2017-04-17",
    "schemaVersion": "1.0",
    "statusCode": 201,
    "statusText": "Success",
    "durationMs": 5,
    "callerIpAddress": "192.168.0.1:11111",
    "correlationId": "ad881411-201e-004e-1c99-cfd67d000000",
    "location": "uswestcentral",
    "uri": "http://mystorageaccount.blob.core.windows.net/cont1/blobname?timeout=10"
}
```

| Propiedad | Descripción |
|:--- |:---|
|**time** | Hora universal coordinada (UTC) a la que el almacenamiento ha recibido la solicitud. Por ejemplo: `2018/11/08 21:09:36.6900118`.|
|**resourceId** | Identificador de recurso de la cuenta de almacenamiento. Por ejemplo: `/subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/`<br>`myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/storageAccounts/blobServices/default`|
|**category** | Categoría de la operación solicitada. Por ejemplo: `StorageRead`, `StorageWrite` o `StorageDelete`.|
|**operationName** | Tipo de operación REST realizada. <br> Puede encontrar una lista completa de operaciones en el tema [Operaciones y mensajes de estado registrados de Storage Analytics](https://docs.microsoft.com/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages). |
|**operationVersion** | Versión del servicio de almacenamiento especificada al realizar la solicitud. Es equivalente al valor del encabezado **x-ms-version**. Por ejemplo: `2017-04-17`.|
|**schemaVersion** | Versión del esquema del registro. Por ejemplo: `1.0`.|
|**statusCode** | Código de estado HTTP de la solicitud. Si se interrumpe la solicitud, este valor se podría establecer en `Unknown`. <br> Por ejemplo: `206` |
|**statusText** | Estado de la operación solicitada.  Puede encontrar una lista completa de mensajes de estado en [Operaciones y mensajes de estado registrados por Storage Analytics](https://docs.microsoft.com/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages). En la versión 2017-04-17 y posteriores, el mensaje de estado `ClientOtherError` no se usa. En su lugar, este campo contiene un código de error. Por ejemplo: `SASSuccess`  |
|**durationMs** | Tiempo total, expresado en milisegundos, necesario para realizar la operación solicitada. Incluye el tiempo necesario para leer la solicitud entrante y para enviar la respuesta al solicitante. Por ejemplo: `12`.|
|**callerIpAddress** | Dirección IP del solicitante, incluido el número de puerto. Por ejemplo: `192.100.0.102:4362`. |
|**correlationId** | Identificador que se usa para poner en correlación registros entre recursos. Por ejemplo: `b99ba45e-a01e-0042-4ea6-772bbb000000`. |
|**ubicación** | Ubicación de la cuenta de almacenamiento. Por ejemplo: `North Europe`. |
|**protocolo**|Protocolo que se usa en la operación. Por ejemplo: `HTTP`, `HTTPS`, `SMB` o `NFS`|
| **uri** | Identificador uniforme de recursos que se solicita. Por ejemplo: `http://myaccountname.blob.core.windows.net/cont1/blobname?timeout=10`. |

### <a name="fields-that-describe-how-the-operation-was-authenticated"></a>Campos que describen cómo se ha autenticado la operación

```json
{
    "identity": {
        "authorization": [
            {
                "action": "Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read",
                "principals": [
                    {
                        "id": "fde5ba15-4355-4223-b811-cccccccccccc",
                        "type": "User"
                    }
                ],
                "roleAssignmentId": "ecf75cb8-491c-4a25-ad6e-aaaaaaaaaaaa",
                "roleDefinitionId": "b7e6dc6d-f1e8-4753-8033-ffffffffffff"
            }
        ],
        "requester": {
            "appId": "691458b9-1327-4635-9f55-bbbbbbbbbbbb",
            "audience": "https://storage.azure.com/",
            "objectId": "fde5ba15-4355-4223-b811-cccccccccccc",
            "tenantId": "72f988bf-86f1-41af-91ab-dddddddddddd",
            "tokenIssuer": "https://sts.windows.net/72f988bf-86f1-41af-91ab-eeeeeeeeeeee/"
           },
        "type": "OAuth"
    },
}

```

| Propiedad | Descripción |
|:--- |:---|
|**identity/type** | Tipo de autenticación que se ha usado para realizar la solicitud. Por ejemplo: `OAuth`, `SAS Key`, `Account Key` o `Anonymous` |
|**identity/tokenHash**|Este campo está reservado para uso interno exclusivamente. |
|**authorization/action** | Acción asignada a la solicitud. Por ejemplo: `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read` |
|**authorization/roleAssignmentId** | Identificador de la asignación de roles. Por ejemplo: `4e2521b7-13be-4363-aeda-111111111111`.|
|**authorization/roleDefinitionId** | Identificador de la definición de roles. Por ejemplo: `ba92f5b4-2d11-453d-a403-111111111111"`.|
|**principals/id** | Identificador de la entidad de seguridad. Por ejemplo: `a4711f3a-254f-4cfb-8a2d-111111111111`.|
|**principals/type** | Tipo de la entidad de seguridad. Por ejemplo: `ServicePrincipal`. |
|**requester/appID** | Identificador de la aplicación de Open Authorization (OAuth) que se usa como solicitante. <br> Por ejemplo: `d3f7d5fe-e64a-4e4e-871d-333333333333`.|
|**requester/audience** | Audiencia de OAuth de la solicitud. Por ejemplo: `https://storage.azure.com`. |
|**requester/objectId** | Identificador de objeto de OAuth de la solicitud. En el caso de la autenticación Kerberos, representa al identificador de objeto del usuario autenticado de Kerberos. Por ejemplo: `0e0bf547-55e5-465c-91b7-2873712b249c`. |
|**requester/tenantId** | Identificador de inquilino de OAuth de la identidad. Por ejemplo: `72f988bf-86f1-41af-91ab-222222222222`.|
|**requester/tokenIssuer** | Emisor de token de OAuth. Por ejemplo: `https://sts.windows.net/72f988bf-86f1-41af-91ab-222222222222/`.|
|**requester/upn** | Nombre principal de usuario (UPN) del solicitante. Por ejemplo: `someone@contoso.com`. |
|**requester/userName** | Este campo está reservado para uso interno exclusivamente.|

### <a name="fields-that-describe-the-service"></a>Campos que describen el servicio

```json
{
    "properties": {
        "accountName": "testaccount1",
        "requestUrl": "https://testaccount1.blob.core.windows.net:443/upload?restype=container&comp=list&prefix=&delimiter=%2F&marker=&maxresults=30&include=metadata&_=1551405598426",
        "userAgentHeader": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.140 Safari/537.36 Edge/17.17134",
        "referrerHeader": "blob:https://ms.portal.azure.com/6f50025f-3b88-488d-b29e-3c592a31ddc9",
        "clientRequestId": "",
        "etag": "",
        "serverLatencyMs": 63,
        "serviceType": "blob",
        "operationCount": 0,
        "requestHeaderSize": 2658,
        "requestBodySize": 0,
        "responseHeaderSize": 295,
        "responseBodySize": 2018,
        "contentLengthHeader": 0,
        "requestMd5": "",
        "serverMd5": "",
        "lastModifiedTime": "",
        "conditionsUsed": "",
        "smbTreeConnectID" : "0x3",
        "smbPersistentHandleID" : "0x6003f",
        "smbVolatileHandleID" : "0xFFFFFFFF00000065",
        "smbMessageID" : "0x3b165",
        "smbCreditsConsumed" : "0x3",
        "smbCommandDetail" : "0x2000 bytes at offset 0xf2000",
        "smbFileId" : " 0x9223442405598953",
        "smbSessionID" : "0x8530280128000049",
        "smbCommandMajor" : "0x6",
        "smbCommandMinor" : "DirectoryCloseAndDelete"
    }

}
```

| Propiedad | Descripción |
|:--- |:---|
|**accountName** | El nombre de la cuenta de almacenamiento. Por ejemplo: `mystorageaccount`.  |
|**requestUrl** | Dirección URL que se solicita. Por ejemplo: `http://mystorageaccount.blob.core.windows.net/cont1/blobname?timeout=10`.|
|**userAgentHeader** | Valor del encabezado **User-Agent**, entre comillas. Por ejemplo: `WA-Storage/6.2.0 (.NET CLR 4.0.30319.42000; Win32NT 6.2.9200.0)`.|
|**referrerHeader** | Valor del encabezado **Referrer**. Por ejemplo: `http://contoso.com/about.html`.|
|**clientRequestId** | Valor de encabezado **x-ms-client-request-id** de la solicitud. Por ejemplo: `360b66a6-ad4f-4c4a-84a4-0ad7cb44f7a6`. |
|**etag** | Identificador de la ETag del objeto devuelto, entre comillas. Por ejemplo: `0x8D101F7E4B662C4`.  |
|**serverLatencyMs** | Tiempo total, expresado en milisegundos, necesario para realizar la operación solicitada. Este valor no incluye la latencia de red (el tiempo necesario para leer la solicitud entrante y enviar la respuesta al solicitante). Por ejemplo: `22`. |
|**serviceType** | Servicio asociado a esta solicitud. Por ejemplo: `blob`, `table`, `files` o `queue`. |
|**operationCount** | Número de cada operación registrada implicada en la solicitud. Este recuento comienza con un índice de `0`. Algunas solicitudes necesitan más de una operación, como una solicitud para copiar un blob. La mayoría de las solicitudes solo realizan una operación. Por ejemplo: `1`. |
|**requestHeaderSize** | Tamaño del encabezado de la solicitud expresado en bytes. Por ejemplo: `578`. <br>Si una solicitud no se realiza correctamente, este valor puede estar vacío. |
|**requestBodySize** | Tamaño expresado en bytes de los paquetes de la solicitud leídos por el servicio de almacenamiento. <br> Por ejemplo: `0`. <br>Si una solicitud no se realiza correctamente, este valor puede estar vacío.  |
|**responseHeaderSize** | Tamaño del encabezado de la respuesta expresado en bytes. Por ejemplo: `216`. <br>Si una solicitud no se realiza correctamente, este valor puede estar vacío.  |
|**responseBodySize** | Tamaño de los paquetes de respuesta escritos por el servicio de almacenamiento, en bytes. Si una solicitud no se realiza correctamente, este valor puede estar vacío. Por ejemplo: `216`.  |
|**requestMd5** | Valor del encabezado **Content-MD5** o **x-ms-content-md5** de la solicitud. El valor del hash MD5 especificado en este campo representa el contenido de la solicitud. Por ejemplo: `788815fd0198be0d275ad329cafd1830`. <br>Este campo puede estar vacío.  |
|**serverMd5** | Valor del hash MD5 calculado por el servicio de almacenamiento. Por ejemplo: `3228b3cf1069a5489b298446321f8521`. <br>Este campo puede estar vacío.  |
|**lastModifiedTime** | Hora de la última modificación del objeto devuelto.  Por ejemplo: `Tuesday, 09-Aug-11 21:13:26 GMT`. <br>Este campo está vacío en el caso de operaciones que pueden devolver varios objetos. |
|**conditionsUsed** | Lista separada por punto y coma de pares clave-valor que representan una condición. Las condiciones pueden ser cualquiera de las siguientes: <li> If-Modified-Since <li> If-Unmodified-Since <li> If-Match <li> If-None-Match  <br> Por ejemplo: `If-Modified-Since=Friday, 05-Aug-11 19:11:54 GMT`. |
|**contentLengthHeader** | Valor del encabezado Content-Length de la solicitud enviada al servicio de almacenamiento. Si la solicitud se ha realizado correctamente, este valor es igual a requestBodySize. Si la solicitud no se ha realizado correctamente, este valor puede no ser igual a requestBodySize, o puede estar vacío. |
|**tlsVersion** | Versión de TLS usada en la conexión de la solicitud. Por ejemplo: `TLS 1.2`. |
|**smbTreeConnectID** | **treeConnectId** del Bloque de mensajes del servidor (SMB) establecido en el momento de la conexión del árbol. Por ejemplo: `0x3` |
|**smbPersistentHandleID** | Identificador de manipulador persistente de una solicitud CREATE de SMB2 que sobrevive a las reconexiones de red.  Con referencia en [MS-SMB2](https://docs.microsoft.com/openspecs/windows_protocols/ms-smb2/f1d9b40d-e335-45fc-9d0b-199a31ede4c3) 2.2.14.1 como **SMB2_FILEID.Persistent**. Por ejemplo: `0x6003f` |
|**smbVolatileHandleID** | Identificador de manipulador volátil de una solicitud CREATE de SMB2 que se recicla en las reconexiones de red.  Con referencia en [MS-SMB2](https://docs.microsoft.com/openspecs/windows_protocols/ms-smb2/f1d9b40d-e335-45fc-9d0b-199a31ede4c3) 2.2.14.1 como **SMB2_FILEID.Volatile**. Por ejemplo: `0xFFFFFFFF00000065` |
|**smbMessageID** | Conexión con respecto a **MessageId**. Por ejemplo: `0x3b165` |
|**smbCreditsConsumed** | Entrada o salida consumida por la solicitud, en unidades de 64 k. Por ejemplo: `0x3` |
|**smbCommandDetail** | Más información sobre esta solicitud específica en lugar del tipo general de solicitud. Por ejemplo: `0x2000 bytes at offset 0xf2000` |
|**smbFileId** | **FileId** asociado al archivo o directorio.  Más o menos análogo a un elemento FileId de NTFS. Por ejemplo: `0x9223442405598953` |
|**smbSessionID** | El elemento **SessionId** de SMB2 establecido en el momento de la configuración de la sesión. Por ejemplo: `0x8530280128000049` |
|**smbCommandMajor  uint32** | Valor de **SMB2_HEADER.Command**. Actualmente, se trata de un número entre 0 y 18, ambos incluidos. Por ejemplo: `0x6` |
|**smbCommandMinor** | Subclase de **SmbCommandMajor**, cuando sea conveniente. Por ejemplo: `DirectoryCloseAndDelete` |

## <a name="see-also"></a>Consulte también

- También puede consultar [Supervisión de Azure Storage](monitor-storage.md) para ver una descripción de la supervisión de Azure Storage.
- Para más información sobre la supervisión de recursos de Azure, consulte [Supervisión de recursos de Azure con Azure Monitor](../../azure-monitor/insights/monitor-azure-resource.md).