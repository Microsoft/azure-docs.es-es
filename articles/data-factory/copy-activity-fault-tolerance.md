---
title: Tolerancia a errores de la actividad de copia en Azure Data Factory
description: Obtenga información sobre cómo agregar una tolerancia a errores para la actividad de copia en Azure Data Factory omitiendo los datos incompatibles.
services: data-factory
documentationcenter: ''
author: dearandyxu
manager: ''
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 06/22/2020
ms.author: yexu
ms.openlocfilehash: 4a0529248c58f7fa7f962d9d1432411c351c7bdd
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89440650"
---
#  <a name="fault-tolerance-of-copy-activity-in-azure-data-factory"></a>Tolerancia a errores de la actividad de copia en Azure Data Factory
> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](v1/data-factory-copy-activity-fault-tolerance.md)
> * [Versión actual](copy-activity-fault-tolerance.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Al copiar datos desde el almacén de origen al de destino, la actividad de copia de Azure Data Factory proporciona cierto nivel de tolerancia a errores para evitar la interrupción ocasionada por errores en medio del movimiento de datos. Por ejemplo, imagine que está copiando millones de filas del almacén de origen al de destino y que se ha creado una clave principal en la base de datos de destino, pero la base de datos de origen no tiene ninguna clave principal definida. Cuando copie filas duplicadas del origen al destino, aparecerá un error de infracción de clave primaria en la base de datos de destino. En este momento, la actividad de copia ofrece dos formas de controlar estos errores: 
- Puede anular la actividad de copia cuando se produzca un error. 
- Puede continuar copiando el resto habilitando la tolerancia a errores para omitir los datos incompatibles. Por ejemplo, omitir la fila duplicada en este caso. Además, puede registrar los datos omitidos habilitando el registro de sesión dentro de la actividad de copia. 

## <a name="copying-binary-files"></a>Copia de archivos binarios 

ADF admite los siguientes escenarios de tolerancia a errores al copiar archivos binarios. Puede anular la actividad de copia o seguir copiando el resto en los escenarios siguientes:

1. Los archivos que ADF copiará están siendo eliminados por otras aplicaciones al mismo tiempo.
2. Algunas carpetas o archivos determinados no permiten que ADF tenga acceso porque los ACL de esos archivos o carpetas requieren un nivel de permiso superior al de la información de conexión configurada en ADF.
3. No se comprueba que uno o varios archivos sean coherentes entre el almacén de origen y el de destino si habilita la configuración de comprobación de coherencia de datos en ADF.

### <a name="configuration"></a>Configuración 
Al copiar archivos binarios entre los almacenes, puede habilitar la tolerancia a errores de la siguiente manera: 

```json
"typeProperties": { 
    "source": { 
        "type": "BinarySource", 
        "storeSettings": { 
            "type": "AzureDataLakeStoreReadSettings", 
            "recursive": true 
            } 
    }, 
    "sink": { 
        "type": "BinarySink", 
        "storeSettings": { 
            "type": "AzureDataLakeStoreWriteSettings" 
        } 
    }, 
    "skipErrorFile": { 
        "fileMissing": true, 
        "fileForbidden": true, 
        "dataInconsistency": true 
    }, 
    "validateDataConsistency": true, 
    "logStorageSettings": { 
        "linkedServiceName": { 
            "referenceName": "ADLSGen2", 
            "type": "LinkedServiceReference" 
            }, 
        "path": "sessionlog/" 
     } 
} 
```
Propiedad | Descripción | Valores permitidos | Obligatorio
-------- | ----------- | -------------- | -------- 
skipErrorFile | Un grupo de propiedades para especificar los tipos de errores que desea omitir durante el movimiento de datos. | | No
fileMissing | Uno de los pares clave-valor dentro del contenedor de propiedades de skipErrorFile para determinar si desea omitir los archivos que se están eliminando en otras aplicaciones mientras ADF realiza la copia. <br/> -True: desea copiar el resto omitiendo los archivos que eliminan otras aplicaciones. <br/> -False: desea anular la actividad de copia una vez que se eliminan los archivos del almacén de origen en medio del movimiento de datos. <br/>Tenga en cuenta que esta propiedad está establecida en true como valor predeterminado. | True (valor predeterminado) <br/>False | No
fileForbidden | Uno de los pares clave-valor dentro del contenedor de propiedades de skipErrorFile para determinar si desea omitir los archivos concretos, cuando los ACL de esos archivos o carpetas requieren un nivel de permiso superior al de la conexión configurada en ADF. <br/> -True: desea copiar el resto omitiendo los archivos. <br/> -False: desea anular la actividad de copia una vez que recibe el problema de los permisos en las carpetas o los archivos. | True <br/>False (valor predeterminado) | No
dataInconsistency | Uno de los pares clave-valor dentro del contenedor de propiedades de skipErrorFile para determinar si desea omitir los datos incoherentes entre el almacén de origen y el de destino. <br/> -True: desea copiar el resto omitiendo los datos incoherentes. <br/> - False: desea anular la actividad de copia una vez que se encuentren datos incoherentes. <br/>Tenga en cuenta que esta propiedad solo es válida cuando se establece validateDataConsistency como True. | True <br/>False (valor predeterminado) | No
logStorageSettings  | Un grupo de propiedades que puede especificarse cuando quiere registrar los nombres de los objetos omitidos. | &nbsp; | No
linkedServiceName | El servicio vinculado de [Azure Blob Storage](connector-azure-blob-storage.md#linked-service-properties) o [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#linked-service-properties) para almacenar los archivos de registro de sesión. | Nombre de un servicio vinculado de tipo `AzureBlobStorage` o `AzureBlobFS`, que hace referencia a la instancia que usa para almacenar el archivo de registro. | No
path | Ruta de acceso de los archivos de registro. | Especifique la ruta de acceso que se utiliza para almacenar los archivos de registro. Si no se proporciona una ruta de acceso, el servicio creará un contenedor para usted. | No

> [!NOTE]
> A continuación se indican los requisitos previos de la habilitación de la tolerancia a errores en la actividad de copia al copiar archivos binarios.
> Para omitir archivos concretos cuando se eliminan del almacén de origen:
> - Los conjuntos de datos de origen y de receptor deben tener un formato binario y no se puede especificar el tipo de compresión. 
> - Los tipos de almacén de datos admitidos son Azure Blob Storage, Azure Data Lake Storage Gen1, Azure Data Lake Storage Gen2, Azure File Storage, Sistema de archivos, FTP, SFTP, Amazon S3, Google Cloud Storage y HDFS.
> - Solo si se especifican varios archivos en el conjunto de datos de origen, que puede ser una carpeta, un carácter comodín o una lista de archivos, la actividad de copia puede omitir los archivos de error concretos. Si se especifica un solo archivo en el conjunto de datos de origen que se va a copiar en el destino, en el caso de que se produzca algún error en la actividad de copia, esta no se llevará a cabo.
>
> Para omitir archivos concretos cuando su acceso está prohibido desde el almacén de origen:
> - Los conjuntos de datos de origen y de receptor deben tener un formato binario y no se puede especificar el tipo de compresión. 
> - Los tipos de almacén de datos admitidos son Azure Blob Storage, Azure Data Lake Storage Gen1, Azure Data Lake Storage Gen2, Azure File Storage, SFTP, Amazon S3 y HDFS.
> - Solo si se especifican varios archivos en el conjunto de datos de origen, que puede ser una carpeta, un carácter comodín o una lista de archivos, la actividad de copia puede omitir los archivos de error concretos. Si se especifica un solo archivo en el conjunto de datos de origen que se va a copiar en el destino, en el caso de que se produzca algún error en la actividad de copia, esta no se llevará a cabo.
>
> Para omitir archivos concretos cuando se compruebe que no son coherentes entre el almacén de origen y el de destino:
> - Puede obtener más detalles del documento de coherencia de datos [aquí](https://docs.microsoft.com/azure/data-factory/copy-activity-data-consistency).

### <a name="monitoring"></a>Supervisión 

#### <a name="output-from-copy-activity"></a>Salida de la actividad de copia
Puede obtener el número de archivos que se van a leer, escribir y omitir a través de la salida de cada ejecución de la actividad de copia. 

```json
"output": {
            "dataRead": 695,
            "dataWritten": 186,
            "filesRead": 3,  
            "filesWritten": 1, 
            "filesSkipped": 2, 
            "throughput": 297,
            "logPath": "https://myblobstorage.blob.core.windows.net//myfolder/a84bf8d4-233f-4216-8cb5-45962831cd1b/",
            "dataConsistencyVerification": 
           { 
                "VerificationResult": "Verified", 
                "InconsistentData": "Skipped" 
           } 
        }

```

#### <a name="session-log-from-copy-activity"></a>Registro de sesión de la actividad de copia

Si configura para registrar los nombres de archivo omitidos, puede encontrar el archivo de registro en esta ruta de acceso: `https://[your-blob-account].blob.core.windows.net/[path-if-configured]/copyactivity-logs/[copy-activity-name]/[copy-activity-run-id]/[auto-generated-GUID].csv`. 

Los archivos de registro deben ser archivos CSV. El esquema del archivo de registro es el siguiente:

Columna | Descripción 
-------- | -----------  
Timestamp | Marca de tiempo en la que ADF omite el archivo.
Nivel | Nivel de registro de este elemento. Estará en el nivel "Warning" para el elemento que muestra la omisión del archivo.
OperationName | Comportamiento operativo de la actividad de copia de ADF en cada archivo. Será "FileSkip" para especificar el archivo que se va a omitir.
OperationItem | Nombres de archivo que se van a omitir.
Message | Más información para ilustrar por qué se omite el archivo.

El ejemplo de un archivo de registro es el siguiente: 
```
Timestamp,Level,OperationName,OperationItem,Message 
2020-03-24 05:35:41.0209942,Warning,FileSkip,"bigfile.csv","File is skipped after read 322961408 bytes: ErrorCode=UserErrorSourceBlobNotExist,'Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException,Message=The required Blob is missing. ContainerName: https://transferserviceonebox.blob.core.windows.net/skipfaultyfile, path: bigfile.csv.,Source=Microsoft.DataTransfer.ClientLibrary,'." 
2020-03-24 05:38:41.2595989,Warning,FileSkip,"3_nopermission.txt","File is skipped after read 0 bytes: ErrorCode=AdlsGen2OperationFailed,'Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException,Message=ADLS Gen2 operation failed for: Operation returned an invalid status code 'Forbidden'. Account: 'adlsgen2perfsource'. FileSystem: 'skipfaultyfilesforbidden'. Path: '3_nopermission.txt'. ErrorCode: 'AuthorizationPermissionMismatch'. Message: 'This request is not authorized to perform this operation using this permission.'. RequestId: '35089f5d-101f-008c-489e-01cce4000000'..,Source=Microsoft.DataTransfer.ClientLibrary,''Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException,Message=Operation returned an invalid status code 'Forbidden',Source=,''Type=Microsoft.Azure.Storage.Data.Models.ErrorSchemaException,Message='Type=Microsoft.Azure.Storage.Data.Models.ErrorSchemaException,Message=Operation returned an invalid status code 'Forbidden',Source=Microsoft.DataTransfer.ClientLibrary,',Source=Microsoft.DataTransfer.ClientLibrary,'." 
```
En el registro anterior, puede ver que bigfile.csv se ha omitido debido a que otra aplicación ha eliminado este archivo cuando ADF lo estaba copiando. Y 3_nopermission.tx se ha omitido porque ADF no puede acceder a él debido a un problema de permisos.


## <a name="copying-tabular-data"></a>Copia de datos tabulares 

### <a name="supported-scenarios"></a>Escenarios admitidos
La actividad de copia admite tres escenarios para detectar, omitir y registrar datos tabulares incompatibles:

- **Incompatibilidad entre el tipo de datos de origen y el tipo nativo de receptor**. 

    Por ejemplo: Copie datos desde un archivo CSV en Blob Storage a una base de datos SQL con una definición de esquema que contenga tres columnas de tipo INT. Las filas del archivo CSV que contienen datos numéricos, como 123 456 789, se copian correctamente en el almacén de receptor. Pero las filas que contienen valores no numéricos, como 123 456 abc, se detectan como incompatibles y se omiten.

- **Error de coincidencia en el número de columnas entre el origen y el receptor**.

    Por ejemplo: Copie datos desde un archivo CSV en Blob Storage a una base de datos SQL con una definición de esquema que contenga seis columnas. Las filas del archivo CSV que contiene seis columnas se copian correctamente en el almacén de receptor. Las filas del archivo CSV que contienen más de seis columnas se detectan como incompatibles y se omiten.

- **Infracción de clave principal al escribir en SQL Server, Azure SQL Database o Azure Cosmos DB**.

    Por ejemplo: Copie datos desde un servidor SQL a una base de datos SQL. Se define una clave principal en la base de datos SQL de receptor, pero no se define en el servidor SQL de origen. Las filas duplicadas que existen en el origen no se pueden copiar en el receptor. La actividad de copia solo copia la primera fila de los datos de origen en el receptor. Las filas de origen subsiguientes que contienen el valor de clave principal duplicado se detectan como incompatibles y se omiten.

>[!NOTE]
>- Para cargar datos en Azure Synapse Analytics (antes SQL Data Warehouse) mediante PolyBase, defina la configuración de tolerancia a errores nativa de PolyBase especificando directivas de rechazo a través de "[polyBaseSettings](connector-azure-sql-data-warehouse.md#azure-sql-data-warehouse-as-sink)" en la actividad de copia. Aún puede habilitar la redirección de filas incompatibles de PolyBase a Blob o ADLS de la forma habitual, como se muestra a continuación.
>- Esta característica no se aplica cuando la actividad de copia está configurada para invoca el mecanismo [Unload de Amazon Redshift](connector-amazon-redshift.md#use-unload-to-copy-data-from-amazon-redshift).
>- Esta característica no se aplica cuando la actividad de copia se configura para invocar un [procedimiento almacenado desde un receptor de SQL](https://docs.microsoft.com/azure/data-factory/connector-azure-sql-database#invoke-a-stored-procedure-from-a-sql-sink).

### <a name="configuration"></a>Configuración
En el ejemplo siguiente se proporciona una definición JSON para configurar la omisión de las filas incompatibles en la actividad de copia:

```json
"typeProperties": { 
    "source": { 
        "type": "AzureSqlSource" 
    }, 
    "sink": { 
        "type": "AzureSqlSink" 
    }, 
    "enableSkipIncompatibleRow": true, 
    "logStorageSettings": { 
    "linkedServiceName": { 
        "referenceName": "ADLSGen2", 
        "type": "LinkedServiceReference" 
        }, 
    "path": "sessionlog/" 
    } 
}, 
```

Propiedad | Descripción | Valores permitidos | Obligatorio
-------- | ----------- | -------------- | -------- 
enableSkipIncompatibleRow | Especifica si se deben omitir las filas incompatibles durante la copia o no. | True<br/>False (valor predeterminado) | No
logStorageSettings | Un grupo de propiedades que puede especificarse cuando quiere registrar las filas incompatibles. | &nbsp; | No
linkedServiceName | Servicio vinculado de [Azure Blob Storage](connector-azure-blob-storage.md#linked-service-properties) o [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#linked-service-properties) para almacenar el registro que contiene las filas que se omiten. | Nombre de un servicio vinculado de tipo `AzureBlobStorage` o `AzureBlobFS`, que hace referencia a la instancia que usa para almacenar el archivo de registro. | No
path | La ruta de acceso de los archivos de registro que contiene las filas que se omiten. | Especifique la ruta de acceso que quiere usar para registrar los datos incompatibles. Si no se proporciona una ruta de acceso, el servicio creará un contenedor para usted. | No

### <a name="monitor-skipped-rows"></a>Supervisar filas omitidas
Una vez finalizada la ejecución de la actividad de copia, puede ver el número de filas omitidas en la salida de la actividad de copia:

```json
"output": {
            "dataRead": 95,
            "dataWritten": 186,
            "rowsCopied": 9,
            "rowsSkipped": 2,
            "copyDuration": 16,
            "throughput": 0.01,
            "logPath": "https://myblobstorage.blob.core.windows.net//myfolder/a84bf8d4-233f-4216-8cb5-45962831cd1b/",
            "errors": []
        },

```

Si configura el registro de las filas incompatibles, puede encontrar el archivo de registro en esta ruta de acceso: `https://[your-blob-account].blob.core.windows.net/[path-if-configured]/copyactivity-logs/[copy-activity-name]/[copy-activity-run-id]/[auto-generated-GUID].csv`. 

Los archivos de registro serán los archivos CSV. El esquema del archivo de registro es el siguiente:

Columna | Descripción 
-------- | -----------  
Timestamp | Marca de tiempo en la que ADF omite las filas incompatibles
Nivel | Nivel de registro de este elemento. Estará en el nivel de "Advertencia" si este elemento muestra las filas omitidas.
OperationName | Comportamiento operativo de la actividad de copia de ADF en cada fila. Será "TabularRowSkip" para especificar que se ha omitido la fila incompatible determinada.
OperationItem | Filas omitidas del almacén de datos de origen.
Message | Más información para ilustrar el motivo de la incompatibilidad de esta fila concreta.


A continuación se muestra un ejemplo del contenido del archivo de registro:

```
Timestamp, Level, OperationName, OperationItem, Message
2020-02-26 06:22:32.2586581, Warning, TabularRowSkip, """data1"", ""data2"", ""data3""," "Column 'Prop_2' contains an invalid value 'data3'. Cannot convert 'data3' to type 'DateTime'." 
2020-02-26 06:22:33.2586351, Warning, TabularRowSkip, """data4"", ""data5"", ""data6"",", "Violation of PRIMARY KEY constraint 'PK_tblintstrdatetimewithpk'. Cannot insert duplicate key in object 'dbo.tblintstrdatetimewithpk'. The duplicate key value is (data4)." 
```

En el archivo de registro de ejemplo anterior, puede ver que se ha omitido una fila "data1, data2, data3" debido a un problema de conversión de tipo del almacén de origen al de destino. Otra fila "data4, data5, data6" se ha omitido debido a un problema de infracción de clave principal del almacén de origen al de destino. 


## <a name="copying-tabular-data-legacy"></a>Copia de datos tabulares (heredado):

A continuación se presenta la manera heredada de habilitar la tolerancia a errores para copiar solo los datos tabulares. Si va a crear una nueva canalización o actividad, le recomendamos que comience desde [aquí](#copying-tabular-data) en su lugar.

### <a name="configuration"></a>Configuración
En el ejemplo siguiente se proporciona una definición JSON para configurar la omisión de las filas incompatibles en la actividad de copia:

```json
"typeProperties": {
    "source": {
        "type": "BlobSource"
    },
    "sink": {
        "type": "SqlSink",
    },
    "enableSkipIncompatibleRow": true,
    "redirectIncompatibleRowSettings": {
         "linkedServiceName": {
              "referenceName": "<Azure Storage or Data Lake Store linked service>",
              "type": "LinkedServiceReference"
            },
            "path": "redirectcontainer/erroroutput"
     }
}
```

Propiedad | Descripción | Valores permitidos | Obligatorio
-------- | ----------- | -------------- | -------- 
enableSkipIncompatibleRow | Especifica si se deben omitir las filas incompatibles durante la copia o no. | True<br/>False (valor predeterminado) | No
redirectIncompatibleRowSettings | Un grupo de propiedades que puede especificarse cuando quiere registrar las filas incompatibles. | &nbsp; | No
linkedServiceName | Servicio vinculado de [Azure Storage](connector-azure-blob-storage.md#linked-service-properties) o [Azure Data Lake Store](connector-azure-data-lake-store.md#linked-service-properties) para almacenar el registro que contiene las filas que se omiten. | Nombres de un servicio vinculado de tipo `AzureStorage` o `AzureDataLakeStore`, que hace referencia a la instancia que quiere usar para almacenar el archivo de registro. | No
path | La ruta de acceso del archivo de registro que contiene las filas que se omiten. | Especifique la ruta de acceso que quiere usar para registrar los datos incompatibles. Si no se proporciona una ruta de acceso, el servicio creará un contenedor para usted. | No

### <a name="monitor-skipped-rows"></a>Supervisar filas omitidas
Una vez finalizada la ejecución de la actividad de copia, puede ver el número de filas omitidas en la salida de la actividad de copia:

```json
"output": {
            "dataRead": 95,
            "dataWritten": 186,
            "rowsCopied": 9,
            "rowsSkipped": 2,
            "copyDuration": 16,
            "throughput": 0.01,
            "redirectRowPath": "https://myblobstorage.blob.core.windows.net//myfolder/a84bf8d4-233f-4216-8cb5-45962831cd1b/",
            "errors": []
        },

```
Si configura el registro de las filas incompatibles, encontrará el archivo de registro en esta ruta de acceso: `https://[your-blob-account].blob.core.windows.net/[path-if-configured]/[copy-activity-run-id]/[auto-generated-GUID].csv`. 

Los archivos de registro solo pueden ser archivos csv. Los datos originales que se omitan se registrarán con una coma como delimitador de columna, si es necesario. Se agregan dos columnas más "ErrorCode" y "ErrorMessage" a los datos de origen originales en el archivo de registro, en las que podrá ver la causa principal de la incompatibilidad. El nombre de estas dos columnas aparece entre comillas dobles. 

A continuación se muestra un ejemplo del contenido del archivo de registro:

```
data1, data2, data3, "UserErrorInvalidDataValue", "Column 'Prop_2' contains an invalid value 'data3'. Cannot convert 'data3' to type 'DateTime'."
data4, data5, data6, "2627", "Violation of PRIMARY KEY constraint 'PK_tblintstrdatetimewithpk'. Cannot insert duplicate key in object 'dbo.tblintstrdatetimewithpk'. The duplicate key value is (data4)."
```

## <a name="next-steps"></a>Pasos siguientes
Consulte los restantes artículos acerca de la actividad de copia:

- [Información general de la actividad de copia](copy-activity-overview.md)
- [Rendimiento de la actividad de copia](copy-activity-performance.md)


