---
title: Registro de Azure Storage Analytics
description: Aprenda a registrar detalles sobre las solicitudes tramitadas en Azure Storage.
author: normesta
ms.service: storage
ms.subservice: common
ms.topic: conceptual
ms.date: 07/23/2020
ms.author: normesta
ms.reviewer: fryu
ms.custom: monitoring
ms.openlocfilehash: 061c7f6a45b8667b7fd03d62bee67c695bec5e68
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87276795"
---
# <a name="azure-storage-analytics-logging"></a>Registro de Azure Storage Analytics

Storage Analytics registra información detallada sobre las solicitudes correctas y erróneas realizadas a un servicio de almacenamiento. Esta información se puede utilizar para supervisar solicitudes concretas y para diagnosticar problemas con un servicio de almacenamiento. Las solicitudes se registran en función de la mejor opción.

 El registro de Storage Analytics no está habilitado de forma predeterminada en la cuenta de almacenamiento. Puede habilitarlo en [Azure Portal](https://portal.azure.com/); para más información, consulte [Supervisión de una cuenta de almacenamiento en Azure Portal](/azure/storage/storage-monitor-storage-account). También puede habilitar Storage Analytics mediante programación a través de la API de REST o la biblioteca de cliente. Use las operaciones [Obtener las propiedades de Blob service](https://docs.microsoft.com/rest/api/storageservices/Blob-Service-REST-API), [Obtener las propiedades de Queue service](https://docs.microsoft.com/rest/api/storageservices/Get-Queue-Service-Properties) y [Obtener las propiedades de Table service](https://docs.microsoft.com/rest/api/storageservices/Get-Table-Service-Properties) para habilitar Storage Analytics en cada servicio.

 Las entradas del registro se crean solo si se presentan solicitudes al punto de conexión de servicio. Por ejemplo, si una cuenta de almacenamiento tiene actividad en el punto de conexión de Blob service, pero no en los puntos de conexión de Table service o Queue service, solo se crearán los registros correspondientes a Blob service.

> [!NOTE]
>  Actualmente, el registro de Storage Analytics está disponible solo para los servicios Blob, Queue y Table service. El registro de Storage Analytics también está disponible para las cuentas [BlockBlobStorage](../blobs/storage-blob-create-account-block-blob.md) de rendimiento Premium. Sin embargo, no está disponible para las cuentas de uso general v2 con rendimiento Premium.

## <a name="requests-logged-in-logging"></a>Solicitudes registradas en el registro
### <a name="logging-authenticated-requests"></a>Registrar solicitudes de autenticación

 Se registran los siguientes tipos de solicitudes autenticadas:

- Solicitudes correctas
- Solicitudes erróneas, incluidos errores de tiempo de espera, de limitación, de red, de autorización y de otro tipo
- Solicitudes que usan una firma de acceso compartido (SAS) u OAuth, tanto las correctas como las erróneas
- Solicitudes de datos de análisis

  Las solicitudes realizadas por el propio Storage Analytics, como la creación o eliminación del registro, no se registran. Puede encontrar una lista completa de los datos registrados en los temas [Operaciones y mensajes de estado registrados por Storage Analytics](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages) y [Formato del registro de Storage Analytics](/rest/api/storageservices/storage-analytics-log-format).

### <a name="logging-anonymous-requests"></a>Registrar solicitudes anónimas

 Se registran los siguientes tipos de solicitudes anónimas:

- Solicitudes correctas
- Errores del servidor
- Errores de tiempo de espera de cliente y servidor
- Solicitudes GET erróneas con el código de error 304 (No modificado)

  El resto de solicitudes anónimas erróneas no se registran. Puede encontrar una lista completa de los datos registrados en los temas [Operaciones y mensajes de estado registrados por Storage Analytics](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages) y [Formato del registro de Storage Analytics](/rest/api/storageservices/storage-analytics-log-format).

## <a name="how-logs-are-stored"></a>Cómo se almacenan los registros

Todos los registros se almacenan en blobs en bloques, en un contenedor denominado `$logs`, que se crea automáticamente cuando Storage Analytics se habilita en una cuenta de almacenamiento. El contenedor `$logs` se encuentra en el espacio de nombres del blob de la cuenta de almacenamiento, por ejemplo: `http://<accountname>.blob.core.windows.net/$logs`. Este contenedor no se puede eliminar una vez habilitado Storage Analytics, aunque su contenido sí se puede eliminar. Si usa la herramienta de exploración del almacenamiento para ir directamente al contenedor, verá todos los blobs que contienen los datos de registro.

> [!NOTE]
>  El contenedor `$logs` no se muestra cuando se realiza una operación para enumerar los contenedores, como la operación Lista de contenedores. Se debe tener acceso a él directamente. Por ejemplo, puede utilizar la operación Mostrar blobs para acceder a los blobs en el contenedor `$logs`.

A medida que se registran las solicitudes, Storage Analytics cargará los resultados intermedios como bloques. Periódicamente, Storage Analytics confirmará estos bloques para que estén disponibles como blobs. Hasta que los datos de registro aparezcan en los blobs del contenedor **$logs**, puede transcurrir hasta una hora, dada la frecuencia con la que el servicio de almacenamiento vacía la escritura de registros. Es posible que haya entradas duplicadas en los registros creados durante la misma hora. Puede determinar si un registro es un duplicado comprobando el número **RequestId** y **Operation**.

Si tiene un gran volumen de datos de registro con varios archivos correspondientes a cada hora, puede usar los metadatos del blob para averiguar qué datos contiene el registro (examinando para ello los campos de metadatos del blob). Esto resulta útil también porque, a veces, puede haber un retraso mientras los datos se escriben en los archivos de registro, y es que los metadatos del blob indican el contenido del blob de forma más precisa que el nombre del blob.

La mayoría de las herramientas de exploración del almacenamiento permiten ver los metadatos de los blobs, si bien esta información se puede consultar también con PowerShell o mediante programación. El siguiente fragmento de código de PowerShell es un ejemplo de filtrado de la lista de blobs de registro por nombre para especificar una hora, y por metadatos para identificar únicamente aquellos registros que contengan operaciones de escritura (**write**).  

 ```powershell
 Get-AzStorageBlob -Container '$logs' |  
 Where-Object {  
     $_.Name -match 'table/2014/05/21/05' -and   
     $_.ICloudBlob.Metadata.LogType -match 'write'  
 } |  
 ForEach-Object {  
     "{0}  {1}  {2}  {3}" –f $_.Name,   
     $_.ICloudBlob.Metadata.StartTime,   
     $_.ICloudBlob.Metadata.EndTime,   
     $_.ICloudBlob.Metadata.LogType  
 }  
 ```  

Para obtener información sobre cómo mostrar los blobs mediante programación, vea [Enumerar recursos de blob](https://msdn.microsoft.com/library/azure/hh452233.aspx) y [Configurar y recuperar propiedades y metadatos de recursos de blob](https://msdn.microsoft.com/library/azure/dd179404.aspx).  

### <a name="log-naming-conventions"></a>Convenciones de nomenclatura de los registros

 Cada registro se escribirá con el siguiente formato:

 `<service-name>/YYYY/MM/DD/hhmm/<counter>.log`

 En la siguiente tabla se describe cada atributo del nombre del registro:

|Atributo|Descripción|
|---------------|-----------------|
|`<service-name>`|El nombre del servicio de almacenamiento. Por ejemplo: `blob`, `table` o `queue`|
|`YYYY`|El año de cuatro dígitos del registro. Por ejemplo: `2011`|
|`MM`|El mes de dos dígitos del registro. Por ejemplo: `07`|
|`DD`|El día de dos dígitos del registro. Por ejemplo: `31`|
|`hh`|La hora de dos dígitos que indica la hora inicial de los registros, en formato UTC de 24 horas. Por ejemplo: `18`|
|`mm`|El número de dos dígitos que indica el minuto inicial de los registros. **Nota:**  Este valor no se admite en la versión actual de Storage Analytics, y su valor será siempre `00`.|
|`<counter>`|Un contador basado en cero con seis dígitos que indica el número de blobs del registro generados para el servicio de almacenamiento en un período de tiempo de una hora. Este contador se inicia en `000000`. Por ejemplo: `000001`|

 El siguiente es un nombre completo de registro de ejemplo que combina los ejemplos anteriores:

 `blob/2011/07/31/1800/000001.log`

 El siguiente es un URI de ejemplo que se puede utilizar para obtener acceso al registro anterior:

 `https://<accountname>.blob.core.windows.net/$logs/blob/2011/07/31/1800/000001.log`

 Cuando se registra una solicitud de almacenamiento, el nombre del registro resultante depende de la hora en la que se completó la operación solicitada. Por ejemplo, si una solicitud para obtener un blob finalizó a las 6:30 P.M. del 31/7/2011, el registro se escribirá con el siguiente prefijo: `blob/2011/07/31/1800/`

### <a name="log-metadata"></a>Metadatos del registro

 Todos los blobs del registro se almacenan con metadatos que se pueden utilizar para identificar los datos de registro que contiene el blob. En la siguiente tabla se describe cada atributo de metadatos:

|Atributo|Descripción|
|---------------|-----------------|
|`LogType`|Describe si el registro contiene información relacionada con las operaciones de lectura, escritura o eliminación. Este valor puede incluir un tipo o una combinación de los tres, separados por comas.<br /><br /> Ejemplo 1: `write`<br /><br /> Ejemplo 2: `read,write`<br /><br /> Ejemplo 3: `read,write,delete`|
|`StartTime`|La primera hora de una entrada en el registro, en formato `YYYY-MM-DDThh:mm:ssZ`. Por ejemplo: `2011-07-31T18:21:46Z`|
|`EndTime`|La última hora de una entrada en el registro, en formato `YYYY-MM-DDThh:mm:ssZ`. Por ejemplo: `2011-07-31T18:22:09Z`|
|`LogVersion`|La versión del formato del registro.|

 En la siguiente lista se muestran los metadatos de ejemplo completos utilizando los ejemplos anteriores:

-   `LogType=write`
-   `StartTime=2011-07-31T18:21:46Z`
-   `EndTime=2011-07-31T18:22:09Z`
-   `LogVersion=1.0`

## <a name="enable-storage-logging"></a>Habilitar el registro de almacenamiento

El registro de almacenamiento se puede habilitar con Azure Portal, PowerShell y Storage SDK.

### <a name="enable-storage-logging-using-the-azure-portal"></a>Habilitar el registro de almacenamiento con Azure Portal  

En Azure Portal, use la hoja **Configuración de diagnóstico (clásica)** para controlar el registro de almacenamiento, accesible desde la sección **Supervisión (clásica)** de la **hoja Menú** de una cuenta de almacenamiento.

Puede especificar los servicios de almacenamiento que quiera registrar, así como el período de retención (en días) de los datos registrados.  

### <a name="enable-storage-logging-using-powershell"></a>Habilitar el registro de almacenamiento con PowerShell  

 Para usar PowerShell en el equipo local para configurar el registro de almacenamiento en la cuenta de almacenamiento, use el cmdlet **Get-AzStorageServiceLoggingProperty** de Azure PowerShell para recuperar la configuración actual y el cmdlet **Set-AzStorageServiceLoggingProperty** para cambiar la configuración actual.  

 Los cmdlets que controlan el registro de almacenamiento usan un parámetro **LoggingOperations**, que es una cadena que contiene una lista separada por comas de los tipos de solicitud que se van a registrar. Los tres tipos de solicitud posibles son **read**, **write** y **delete**. Para desactivar el registro, use el valor **none** en el parámetro **LoggingOperations**.  

 Con el siguiente comando se activa el registro de solicitudes de lectura, escritura y eliminación en el servicio Queue service de la cuenta de almacenamiento predeterminada, con un período de retención establecido en cinco días:  

```powershell
Set-AzStorageServiceLoggingProperty -ServiceType Queue -LoggingOperations read,write,delete -RetentionDays 5  
```  

 Con el siguiente comando se desactiva el registro en el servicio Table service de la cuenta de almacenamiento predeterminada:  

```powershell
Set-AzStorageServiceLoggingProperty -ServiceType Table -LoggingOperations none  
```  

 Para más información sobre cómo configurar los cmdlets de Azure PowerShell para que funcionen con su suscripción de Azure y cómo seleccionar la cuenta de almacenamiento predeterminada que quiere usar, vea: [Cómo instalar y configurar Azure PowerShell](https://azure.microsoft.com/documentation/articles/install-configure-powershell/).  

### <a name="enable-storage-logging-programmatically"></a>Habilitar el registro de almacenamiento mediante programación  

 Aparte de usar Azure Portal o los cmdlets de Azure PowerShell para controlar el registro de almacenamiento, también se puede recurrir a una de las API de Azure Storage. Por ejemplo, si está utilizando un lenguaje .NET, puede usar la Biblioteca del cliente de almacenamiento.  

# <a name="net-v12-sdk"></a>[\.SDK de .NET, versión 12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/Monitoring.cs" id="snippet_EnableDiagnosticLogs":::

# <a name="net-v11-sdk"></a>[\.SDK de .NET, versión 11](#tab/dotnet11)

```csharp
var storageAccount = CloudStorageAccount.Parse(connStr);  
var queueClient = storageAccount.CreateCloudQueueClient();  
var serviceProperties = queueClient.GetServiceProperties();  

serviceProperties.Logging.LoggingOperations = LoggingOperations.All;  
serviceProperties.Logging.RetentionDays = 2;  

queueClient.SetServiceProperties(serviceProperties);  
```  

---


 Para obtener más información sobre cómo usar un lenguaje .NET para configurar el registro de almacenamiento, vea [Referencia de biblioteca de cliente de almacenamiento](https://msdn.microsoft.com/library/azure/dn261237.aspx).  

 Para obtener información general sobre cómo configurar el registro de almacenamiento con la API de REST, vea [Habilitar y configurar Storage Analytics](https://msdn.microsoft.com/library/azure/hh360996.aspx).  

## <a name="download-storage-logging-log-data"></a>Descargar datos de registro del registro de almacenamiento

 Para ver y analizar los datos de registro, conviene descargar en un equipo local los blobs que contengan los datos de registro de su interés. Muchas herramientas de exploración del almacenamiento permiten descargar blobs de la cuenta de almacenamiento; también puede usar la herramienta de la línea de comandos para copiar de Azure [AzCopy](storage-use-azcopy-v10.md), suministrada por el equipo de Azure Storage, para descargar los datos de registro.  
 
>[!NOTE]
> El contenedor `$logs` no está integrado con Event Grid, por lo que no recibirá notificaciones cuando se escriban en los archivos de registro. 

 Para estar seguro de que descarga los datos de registro de su interés y no descarga los mismos datos de registro más de una vez:  

-   Use la convención de nomenclatura de fecha y hora en los blobs que contienen datos de registro para realizar un seguimiento de los blobs que ya ha descargado para analizarlos y, así, evitar volver a descargar los mismos datos más de una vez.  

-   Use los metadatos de los blobs que contienen datos de registro para identificar el período específico en el que el blob contiene los datos de registro y, así, detectar el blob exacto que necesita descargar.  

Para empezar a trabajar con AzCopy, consulte [Introducción a AzCopy](storage-use-azcopy-v10.md). 

En el siguiente ejemplo se muestra cómo descargar los datos de registro de Queue service correspondientes a las horas 9 A.M., 10 A.M. y 11 A.M. del 20 de mayo de 2014.

```
azcopy copy 'https://mystorageaccount.blob.core.windows.net/$logs/queue' 'C:\Logs\Storage' --include-path '2014/05/20/09;2014/05/20/10;2014/05/20/11' --recursive
```

Para obtener más información sobre cómo descargar archivos específicos, consulte [Descarga de archivos específicos](https://docs.microsoft.com/azure/storage/common/storage-use-azcopy-blobs?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#download-specific-files).

Cuando haya descargado los datos de registro, puede ver las entradas de registro en los archivos. Estos archivos de registro usan un formato de texto delimitado que muchas herramientas de lectura de registros son capaces de analizar, como el Analizador de mensajes de Microsoft; para obtener más información, vea la guía [Supervisión, diagnóstico y solución de problemas de Microsoft Azure Storage](storage-monitoring-diagnosing-troubleshooting.md). Cada herramienta tiene diferentes recursos para aplicar formato, filtrar, ordenar y buscar contenido en los archivos de registro. Para obtener más información sobre el contenido y el formato de los archivos de registro del registro de almacenamiento, vea [Formato de registros de Storage Analytics](/rest/api/storageservices/storage-analytics-log-format) y [Operaciones registradas y mensajes de estado de Storage Analytics](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages).

## <a name="next-steps"></a>Pasos siguientes

* [Formato del registro de Storage Analytics](/rest/api/storageservices/storage-analytics-log-format)
* [Operaciones y mensajes de estado registrados por Storage Analytics](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages)
* [Métricas de Storage Analytics (clásica)](storage-analytics-metrics.md)
