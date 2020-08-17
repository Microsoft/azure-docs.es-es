---
title: Solución de problemas de conectores en Azure Data Factory
description: Obtenga información acerca de la solución de problemas relacionados con conectores en Azure Data Factory.
services: data-factory
author: linda33wj
ms.service: data-factory
ms.topic: troubleshooting
ms.date: 07/20/2020
ms.author: jingwang
ms.reviewer: craigg
ms.custom: has-adal-ref
ms.openlocfilehash: c8edb36345de4516077b3c857cff33389062cc7f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87044550"
---
# <a name="troubleshoot-azure-data-factory-connectors"></a>Solución de problemas de conectores en Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se exploran métodos comunes de solución de problemas de conectores en Azure Data Factory.
  

## <a name="azure-blob-storage"></a>Azure Blob Storage

### <a name="error-code--azurebloboperationfailed"></a>Código de error:  AzureBlobOperationFailed

- **Mensaje**: `Blob operation Failed. ContainerName: %containerName;, path: %path;.`

- **Causa**: La operación de almacenamiento de blobs tuvo un problema.

- **Recomendación:**  compruebe el error en los detalles. Consulte el documento de ayuda de blobs: https://docs.microsoft.com/rest/api/storageservices/blob-service-error-codes. Póngase en contacto con el equipo de almacenamiento si necesita ayuda.


### <a name="error-code--azureblobservicenotreturnexpecteddatalength"></a>Código de error:  AzureBlobServiceNotReturnExpectedDataLength

- **Mensaje**: `Error occurred when trying to fetch the blob '%name;'. This could be a transient issue and you may rerun the job. If it fails again continuously, contact customer support.`


### <a name="error-code--azureblobnotsupportmultiplefilesintosingleblob"></a>Código de error:  AzureBlobNotSupportMultipleFilesIntoSingleBlob

- **Mensaje**: `Transferring multiple files into a single Blob is not supported. Currently only single file source is supported.`


### <a name="error-code--azurestorageoperationfailedconcurrentwrite"></a>Código de error:  AzureStorageOperationFailedConcurrentWrite

- **Mensaje**: `Error occurred when trying to upload a file. It's possible because you have multiple concurrent copy activities runs writing to the same file '%name;'. Check your ADF configuration.`


## <a name="azure-cosmos-db"></a>Azure Cosmos DB

### <a name="error-message-request-size-is-too-large"></a>Mensaje de error: Request size is too large (El tamaño de la solicitud es demasiado grande)

- **Síntomas**: Al copiar datos en Azure Cosmos DB con el tamaño de lote de escritura predeterminado, se produce el error *"**El tamaño de la solicitud es demasiado grande**"* .

- **Causa**: Cosmos DB limita el tamaño de una única solicitud a 2 MB. La fórmula es Tamaño de la solicitud = tamaño de documento único x tamaño de lote de escritura. Si el tamaño del documento es grande, el comportamiento predeterminado producirá un tamaño de solicitud demasiado grande. Puede ajustar el tamaño del lote de escritura.

- **Solución:** En el receptor de la actividad de copia, reduzca el valor del tamaño del lote de escritura (el valor predeterminado es 10000).

### <a name="error-message-unique-index-constraint-violation"></a>Mensaje de error: Unique index constraint violation (Infracción de restricción de índice único)

- **Síntomas**: Al copiar datos en Cosmos DB, se produce el siguiente error:

    ```
    Message=Partition range id 0 | Failed to import mini-batch. 
    Exception was Message: {"Errors":["Encountered exception while executing function. Exception = Error: {\"Errors\":[\"Unique index constraint violation.\"]}... 
    ```

- **Causa**: Hay dos causas posibles:

    - Si utiliza **Insert** como comportamiento de escritura, este error indica que los datos de origen tienen filas/objetos con el mismo identificador.

    - Si usa **Upsert** como comportamiento de escritura y establece otra clave única en el contenedor, este error significa que los datos de origen tienen filas/objetos con identificadores diferentes pero con el mismo valor para la clave única definida.

- **Solución:** 

    - Para la primera causa, establezca **Upsert** como comportamiento de escritura.
    - Para la segunda causa, asegúrese de que cada documento tiene un valor diferente para la clave única definida.

### <a name="error-message-request-rate-is-large"></a>Mensaje de error: La tasa de solicitudes es grande

- **Síntomas**: Al copiar datos en Cosmos DB, se produce el siguiente error:

    ```
    Type=Microsoft.Azure.Documents.DocumentClientException,
    Message=Message: {"Errors":["Request rate is large"]}
    ```

- **Causa**: Las unidades de solicitud utilizadas son mayores que la RU disponible configurada en Cosmos DB. Obtenga información sobre cómo Cosmos DB calcula la RU [aquí](../cosmos-db/request-units.md#request-unit-considerations).

- **Solución:** Hay dos soluciones:

    1. **Aumente la RU del contenedor** a un valor mayor en Cosmos DB, lo que mejorará el rendimiento de la actividad de copia, aunque acarreará un mayor costo en Cosmos DB. 

    2. Reduzca **writeBatchSize** a un valor más pequeño (por ejemplo, 1000) y establezca **parallelCopies** en un valor inferior, como 1, lo que hará que el rendimiento de la ejecución de copia sea peor que el actual, pero no acarreará más costos en Cosmos DB.

### <a name="column-missing-in-column-mapping"></a>Falta una columna en la asignación de columnas

- **Síntomas**: Cuando se importa un esquema para Cosmos DB para la asignación de columnas, faltan algunas columnas. 

- **Causa**: ADF infiere el esquema de los 10 primeros documentos de Cosmos DB. Si algunas columnas o propiedades no tienen valor en esos documentos, ADF no las detectará, por lo que no se mostrarán.

- **Solución:** Puede ajustar la consulta como se muestra a continuación para hacer que la columna aparezca en el conjunto de resultados con un valor vacío (suponga que la columna "impossible" falta en los 10 primeros documentos). Como alternativa, puede agregar manualmente la columna para la asignación.

    ```sql
    select c.company, c.category, c.comments, (c.impossible??'') as impossible from c
    ```

### <a name="error-message-the-guidrepresentation-for-the-reader-is-csharplegacy"></a>Mensaje de error: The GuidRepresentation for the reader is CSharpLegacy (El valor de GuidRepresentation del lector es CSharpLegacy)

- **Síntomas**: Al copiar datos desde Cosmos DB MongoAPI/MongoDB con el campo de UUID, se produce el siguiente error:

    ```
    Failed to read data via MongoDB client.,
    Source=Microsoft.DataTransfer.Runtime.MongoDbV2Connector,Type=System.FormatException,
    Message=The GuidRepresentation for the reader is CSharpLegacy which requires the binary sub type to be UuidLegacy not UuidStandard.,Source=MongoDB.Bson,’“,
    ```

- **Causa**: Hay dos maneras de representar el UUID en BSON: UuidStardard y UuidLegacy. De forma predeterminada, UuidLegacy se usa para leer datos. Se producirá un error si los datos de UUID en MongoDB son UuidStandard.

- **Solución:** En la cadena de conexión de MongoDB, agregue la opción "**uuidRepresentation=standard**". Para más información, consulte [Cadena de conexión de MongoDB](connector-mongodb.md#linked-service-properties).
            

## <a name="azure-data-lake-storage-gen2"></a>Azure Data Lake Storage Gen2

### <a name="error-code--adlsgen2operationfailed"></a>Código de error:  AdlsGen2OperationFailed

- **Mensaje**: `ADLS Gen2 operation failed for: %adlsGen2Message;.%exceptionData;.`

- **Causa**: ADLS Gen2 muestra un error que indica que no se pudo realizar la operación.

- **Recomendación:**  compruebe el mensaje de error detallado de ADLS Gen2. Si se debe a un error transitorio, vuelva a intentarlo. Si necesita más ayuda, póngase en contacto con el Soporte técnico de Azure Storage y proporcione el id. de solicitud en el mensaje de error.

- **Causa**: Cuando el mensaje de error contiene el valor "Forbidden", es posible que la entidad de servicio o la identidad administrada que está usando no tenga permisos suficientes para obtener acceso a ADLS Gen2.

- **Recomendación:**  consulte el documento de ayuda: https://docs.microsoft.com/azure/data-factory/connector-azure-data-lake-storage#service-principal-authentication.

- **Causa**: Cuando el mensaje de error contiene el valor "InternalServerError", el error lo devuelve ADLS Gen2.

- **Recomendación:**  Si se debe a un error transitorio, vuelva a intentarlo. Si el problema persiste, póngase en contacto con el Soporte técnico de Azure Storage y proporcione el id. de solicitud en el mensaje de error.


### <a name="error-code--adlsgen2invalidurl"></a>Código de error:  AdlsGen2InvalidUrl

- **Mensaje**: `Invalid url '%url;' provided, expecting http[s]://<accountname>.dfs.core.windows.net.`


### <a name="error-code--adlsgen2invalidfolderpath"></a>Código de error:  AdlsGen2InvalidFolderPath

- **Mensaje**: `The folder path is not specified. Cannot locate the file '%name;' under the ADLS Gen2 account directly. Please specify the folder path instead.`


### <a name="error-code--adlsgen2operationfailedconcurrentwrite"></a>Código de error:  AdlsGen2OperationFailedConcurrentWrite

- **Mensaje**: `Error occurred when trying to upload a file. It's possible because you have multiple concurrent copy activities runs writing to the same file '%name;'. Check your ADF configuration.`


### <a name="error-code-adlsgen2timeouterror"></a>Código de error: AdlsGen2TimeoutError

- **Mensaje**: `Request to ADLS Gen2 account '%account;' met timeout error. It is mostly caused by the poor network between the Self-hosted IR machine and the ADLS Gen2 account. Check the network to resolve such error.`


## <a name="azure-data-lake-storage-gen1"></a>Azure Data Lake Storage Gen1

### <a name="error-message-the-underlying-connection-was-closed-could-not-establish-trust-relationship-for-the-ssltls-secure-channel"></a>Mensaje de error: Se ha cerrado la conexión subyacente: No se ha podido establecer la relación de confianza para el canal seguro SSL/TLS.

- **Síntomas**: Se produce el siguiente error en la actividad de copia: 

    ```
    Message: Failure happened on 'Sink' side. ErrorCode=UserErrorFailedFileOperation,'Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException,Message=Upload file failed at path STAGING/PLANT/INDIARENEWABLE/LiveData/2020/01/14\\20200114-0701-oem_gibtvl_mannur_data_10min.csv.,Source=Microsoft.DataTransfer.ClientLibrary,''Type=System.Net.WebException,Message=The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel.,Source=System,''Type=System.Security.Authentication.AuthenticationException,Message=The remote certificate is invalid according to the validation procedure.,Source=System,'.
    ```

- **Causa**: Error de validación de certificado durante el protocolo de enlace TLS.

- **Solución:** Solución alternativa: Use una copia almacenada provisionalmente para omitir la validación TLS para ADLS Gen1. Debe reproducir este problema y recopilar el seguimiento de Netmon y, a continuación, implicar al equipo de red en la comprobación de la configuración de red local conforme a [este artículo](self-hosted-integration-runtime-troubleshoot-guide.md#how-to-collect-netmon-trace).


    ![Solución de problemas de ADLS Gen1](./media/connector-troubleshoot-guide/adls-troubleshoot.png)


### <a name="error-message-the-remote-server-returned-an-error-403-forbidden"></a>Mensaje de error: Error en el servidor remoto: (403) Prohibido

- **Síntomas**: Se produce el siguiente error en la actividad de copia: 

    ```
    Message: The remote server returned an error: (403) Forbidden.. 
    Response details: {"RemoteException":{"exception":"AccessControlException""message":"CREATE failed with error 0x83090aa2 (Forbidden. ACL verification failed. Either the resource does not exist or the user is not authorized to perform the requested operation.)....
    ```

- **Causa**: Una posible causa es que la entidad de servicio o la identidad administrada que usa no tiene permiso para acceder a una carpeta o un archivo determinados.

- **Solución:** Conceda los permisos correspondientes en todas las carpetas y subcarpetas que necesita copiar. Consulte [este documento](connector-azure-data-lake-store.md#linked-service-properties).

### <a name="error-message-failed-to-get-access-token-by-using-service-principal-adal-error-service_unavailable"></a>Mensaje de error: No se pudo obtener el token de acceso mediante la entidad de servicio. Error de ADAL: service_unavailable

- **Síntomas**: Se produce el siguiente error en la actividad de copia:

    ```
    Failed to get access token by using service principal. 
    ADAL Error: service_unavailable, The remote server returned an error: (503) Server Unavailable.
    ```

- **Causa**: Cuando el servicio de token de seguridad (STS) propiedad de Azure Active Directory no está disponible, es decir, cuando está demasiado ocupado para administrar las solicitudes, devuelve un error HTTP 503. 

- **Solución:** Vuelva a ejecutar la actividad de copia después de unos minutos.
                  

## <a name="azure-sql-data-warehouseazure-sql-databasesql-server"></a>Azure SQL Data Warehouse, Azure SQL Database, SQL Server

### <a name="error-code--sqlfailedtoconnect"></a>Código de error:  SqlFailedToConnect

- **Mensaje**: `Cannot connect to SQL Database: '%server;', Database: '%database;', User: '%user;'. Check the linked service configuration is correct, and make sure the SQL Database firewall allows the integration runtime to access.`

- **Causa**: si el mensaje de error contiene el valor "SqlException", SQL Database genera el error que indica que se produjo un problema en una operación específica.

- **Recomendación:**  Busque en este documento de referencia el código de error de SQL para obtener más detalles: https://docs.microsoft.com/sql/relational-databases/errors-events/database-engine-events-and-errors. Si necesita más ayuda, póngase en contacto con el soporte técnico de Azure SQL.

- **Causa**: si el mensaje de error es "El cliente con la dirección IP '...' no tiene permiso para acceder al servidor" y está intentando conectarse a Azure SQL Database, normalmente se debe a un problema con el firewall de Azure SQL Database.

- **Recomendación:**  En la configuración del firewall del servidor SQL lógico, habilite la opción "Permitir que los servicios y recursos de Azure tengan acceso a este servidor". Documento de referencia: https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure.


### <a name="error-code--sqloperationfailed"></a>Código de error:  SqlOperationFailed

- **Mensaje**: `A database operation failed. Please search error to get more details.`

- **Causa**: si el mensaje de error contiene el valor "SqlException", SQL Database genera el error que indica que se produjo un problema en una operación específica.

- **Recomendación:**  si el error de SQL no está claro, intente modificar la base de datos al nivel de compatibilidad más reciente de "150". Puede producir errores de SQL de la versión más reciente. Consulte el documento de detalles: https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql-compatibility-level?view=sql-server-ver15#backwardCompat.
        Para resolver problemas relacionados con SQL, busque en este documento de referencia el código de error de SQL para obtener más detalles: https://docs.microsoft.com/sql/relational-databases/errors-events/database-engine-events-and-errors. Si necesita más ayuda, póngase en contacto con el soporte técnico de Azure SQL.

- **Causa**: Si el mensaje de error contiene el valor "PdwManagedToNativeInteropException", normalmente se debe a una falta de coincidencia entre los tamaños de columna de origen y receptor.

- **Recomendación:**  compruebe el tamaño de las columnas de origen y receptor. Si necesita más ayuda, póngase en contacto con el soporte técnico de Azure SQL.

- **Causa**: Si el mensaje de error contiene el valor "InvalidOperationException", normalmente se debe a que los datos de entrada no son válidos.

- **Recomendación:**  para saber en qué fila encuentra el problema, habilite la característica de tolerancia a errores en la actividad de copia, que puede redirigir las filas problemáticas al almacenamiento para poder realizar una investigación más detallada. Documento de referencia: https://docs.microsoft.com/azure/data-factory/copy-activity-fault-tolerance.



### <a name="error-code--sqlunauthorizedaccess"></a>Código de error:  SqlUnauthorizedAccess

- **Mensaje**: `Cannot connect to '%connectorName;'. Detail Message: '%message;'`

- **Causa**: la credencial es incorrecta o la cuenta de inicio de sesión no puede acceder a SQL Database.

- **Recomendación:**  compruebe que la cuenta de inicio de sesión tiene permisos suficientes para acceder a SQL Database.


### <a name="error-code--sqlopenconnectiontimeout"></a>Código de error:  SqlOpenConnectionTimeout

- **Mensaje**: `Open connection to database timeout after '%timeoutValue;' seconds.`

- **Causa**: podría ser un error transitorio de SQL Database.

- **Recomendación:**  Vuelva a intentar actualizar la cadena de conexión del servicio vinculado mediante un valor de tiempo de espera de conexión mayor.


### <a name="error-code--sqlautocreatetabletypemapfailed"></a>Código de error:  SqlAutoCreateTableTypeMapFailed

- **Mensaje**: `Type '%dataType;' in source side cannot be mapped to a type that supported by sink side(column name:'%columnName;') in autocreate table.`

- **Causa**: La tabla de creación automática no puede cumplir el requisito de origen.

- **Recomendación:**  actualice el tipo de columna en "mappings" o cree manualmente la tabla del receptor en el servidor de destino.


### <a name="error-code--sqldatatypenotsupported"></a>Código de error:  SqlDataTypeNotSupported

- **Mensaje**: `A database operation failed. Check the SQL errors.`

- **Causa**: Si el problema se produce en el origen de SQL y el error está relacionado con el desbordamiento de SqlDateTime, el valor de datos está por encima del intervalo de tipo de lógica (1/1/1753 12:00:00 AM-12/31/9999 11:59:59 PM).

- **Recomendación:**  convierta el tipo en una cadena en la consulta SQL de origen o, en la asignación de columnas de la actividad de copia, cambie el tipo de columna a "String".

- **Causa**: Si el problema se produce en el receptor de SQL y el error está relacionado con el desbordamiento de SqlDateTime, el valor de los datos estará por encima del intervalo permitido en la tabla del receptor.

- **Recomendación:**  Actualice el tipo de columna correspondiente al tipo "datetime2" en la tabla del receptor.


### <a name="error-code--sqlinvaliddbstoredprocedure"></a>Código de error:  SqlInvalidDbStoredProcedure

- **Mensaje**: `The specified Stored Procedure is not valid. It could be caused by that the stored procedure doesn't return any data. Invalid Stored Procedure script: '%scriptName;'.`

- **Causa**: El procedimiento almacenado especificado no es válido. Esto podría deberse a que el procedimiento almacenado no devuelve ningún dato.

- **Recomendación:**  valide el procedimiento almacenado con las herramientas de SQL. Asimismo, asegúrese de que el procedimiento almacenado puede devolver datos.


### <a name="error-code--sqlinvaliddbquerystring"></a>Código de error:  SqlInvalidDbQueryString

- **Mensaje**: `The specified SQL Query is not valid. It could be caused by that the query doesn't return any data. Invalid query: '%query;'`

- **Causa**: La consulta SQL especificada no es válida. Esto podría deberse a que la consulta no devuelve ningún dato.

- **Recomendación:**  Valide la consulta SQL con las herramientas de SQL. Asimismo, asegúrese de que la consulta pueda devolver datos.


### <a name="error-code--sqlinvalidcolumnname"></a>Código de error:  SqlInvalidColumnName

- **Mensaje**: `Column '%column;' does not exist in the table '%tableName;', ServerName: '%serverName;', DatabaseName: '%dbName;'.`

- **Causa**: No se puede encontrar la columna. Es posible que la configuración no sea correcta.

- **Recomendación:**  compruebe la columna de la consulta, el valor "structure" en el conjunto de datos y el valor "mappings" en la actividad.


### <a name="error-code--sqlcolumnnamemismatchbycasesensitive"></a>Código de error:  SqlColumnNameMismatchByCaseSensitive

- **Mensaje**: `Column '%column;' in DataSet '%dataSetName;' cannot be found in physical SQL Database. Column matching is case-sensitive. Column '%columnInTable;' appears similar. Check the DataSet(s) configuration to proceed further.`


### <a name="error-code--sqlbatchwritetimeout"></a>Código de error:  SqlBatchWriteTimeout

- **Mensaje**: `Timeouts in SQL write operation.`

- **Causa**: podría ser un error transitorio de SQL Database.

- **Recomendación:**  Inténtelo de nuevo. Si el problema persiste, póngase en contacto con el soporte técnico de Azure SQL.


### <a name="error-code--sqlbatchwritetransactionfailed"></a>Código de error:  SqlBatchWriteTransactionFailed

- **Mensaje**: `SQL transaction commits failed`

- **Causa**: si los detalles de la excepción indican constantemente un tiempo de espera de la transacción, la latencia de red entre el entorno de ejecución de integración y la base de datos supera el umbral predeterminado 30 segundos.

- **Recomendación:**  actualice la cadena de conexión del servicio vinculado SQL con el valor de "tiempo de espera de la conexión" igual a 120 o superior y vuelva a ejecutar la actividad.

- **Causa**: si los detalles de la excepción indican de forma intermitente una interrupción de la conexión de SQL, podría tratarse de un error de red transitorio o de un problema de SQL Database.

- **Recomendación:**  vuelva a intentar la actividad y revise las métricas de SQL Database.


### <a name="error-code--sqlbulkcopyinvalidcolumnlength"></a>Código de error:  SqlBulkCopyInvalidColumnLength

- **Mensaje**: `SQL Bulk Copy failed due to receive an invalid column length from the bcp client.`

- **Causa**: error en la copia masiva de SQL, ya que se ha recibido una longitud de columna del cliente bcp que no es válida.

- **Recomendación:**  para saber en qué fila encuentra el problema, habilite la característica de tolerancia a errores en la actividad de copia, que puede redirigir las filas problemáticas al almacenamiento para poder realizar una investigación más detallada. Documento de referencia: https://docs.microsoft.com/azure/data-factory/copy-activity-fault-tolerance.


### <a name="error-code--sqlconnectionisclosed"></a>Código de error:  SqlConnectionIsClosed

- **Mensaje**: `The connection is closed by SQL Database.`

- **Causa**: SQL Database cierra la conexión SQL cuando la ejecución simultánea excesiva y el servidor cierran la conexión.

- **Recomendación:**  el servidor remoto cerró la conexión SQL. Inténtelo de nuevo. Si el problema persiste, póngase en contacto con el soporte técnico de Azure SQL.


### <a name="error-code--sqlcreatetablefailedunsupportedtype"></a>Código de error:  SqlCreateTableFailedUnsupportedType

- **Mensaje**: `Type '%type;' in source side cannot be mapped to a type that supported by sink side(column name:'%name;') in autocreate table.`


### <a name="error-message-conversion-failed-when-converting-from-a-character-string-to-uniqueidentifier"></a>Mensaje de error: Error de conversión al convertir de una cadena de caracteres a uniqueidentifier

- **Síntomas**: Al copiar datos desde un origen de datos tabulares (como SQL Server) en Azure SQL Data Warehouse mediante copias almacenadas provisionalmente y PolyBase, se produce el siguiente error:

    ```
    ErrorCode=FailedDbOperation,Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException,
    Message=Error happened when loading data into SQL Data Warehouse.,
    Source=Microsoft.DataTransfer.ClientLibrary,Type=System.Data.SqlClient.SqlException,
    Message=Conversion failed when converting from a character string to uniqueidentifier...
    ```

- **Causa**: PolyBase de Azure SQL Data Warehouse no puede convertir una cadena vacía en GUID.

- **Solución:** En el receptor de la actividad de copia, en la configuración de PolyBase, establezca la opción **Use Type default** (Usar tipo predeterminado) en false.

### <a name="error-message-expected-data-type-decimalxx-offending-value"></a>Mensaje de error: Expected data type: DECIMAL(x,x), Offending value (Tipo de datos esperado: DECIMAL(x,x), valor incorrecto)

- **Síntomas**: Al copiar datos desde un origen de datos tabulares (como SQL Server) en SQL Data Warehouse mediante copias almacenadas provisionalmente y PolyBase, se produce el siguiente error:

    ```
    ErrorCode=FailedDbOperation,Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException,
    Message=Error happened when loading data into SQL Data Warehouse.,
    Source=Microsoft.DataTransfer.ClientLibrary,Type=System.Data.SqlClient.SqlException,
    Message=Query aborted-- the maximum reject threshold (0 rows) was reached while reading from an external source: 1 rows rejected out of total 415 rows processed. (/file_name.txt) 
    Column ordinal: 18, Expected data type: DECIMAL(x,x), Offending value:..
    ```

- **Causa**: PolyBase de Azure SQL Data Warehouse no puede insertar una cadena vacía (valor null) en la columna decimal.

- **Solución:** En el receptor de la actividad de copia, en la configuración de PolyBase, establezca la opción **Use Type default** (Usar tipo predeterminado) en false.

### <a name="error-message-java-exception-message-hdfsbridgecreaterecordreader"></a>Mensaje de error: Mensaje de excepción de Java: HdfsBridge::CreateRecordReader

- **Síntomas**: Al copiar datos en Azure SQL Data Warehouse mediante PolyBase se produce el siguiente error:

    ```
    Message=110802;An internal DMS error occurred that caused this operation to fail. 
    Details: Exception: Microsoft.SqlServer.DataWarehouse.DataMovement.Common.ExternalAccess.HdfsAccessException, 
    Message: Java exception raised on call to HdfsBridge_CreateRecordReader. 
    Java exception message:HdfsBridge::CreateRecordReader - Unexpected error encountered creating the record reader.: Error [HdfsBridge::CreateRecordReader - Unexpected error encountered creating the record reader.] occurred while accessing external file.....
    ```

- **Causa**: La posible causa es que el esquema (ancho total de columna) es demasiado grande (más de 1 MB). Compruebe el esquema de la tabla de SQL Data Warehouse de destino sumando el tamaño de todas las columnas:

    - Int -> 4 bytes
    - Bigint -> 8 bytes
    - Varchar(n),char(n),binary(n), varbinary(n) -> n bytes
    - Nvarchar(n), nchar(n) -> n*2 bytes
    - Date -> 6 bytes
    - Datetime/(2), smalldatetime -> 16 bytes
    - Datetimeoffset -> 20 bytes
    - Decimal -> 19 bytes
    - Float -> 8 bytes
    - Money -> 8 bytes
    - Smallmoney -> 4 bytes
    - Real -> 4 bytes
    - Smallint -> 2 bytes
    - Time -> 12 bytes
    - Tinyint -> 1 byte

- **Solución:** Reduzca el ancho de columna a menos de 1 MB.

- También puede deshabilitar PolyBase para usar un enfoque de inserción masiva.

### <a name="error-message-the-condition-specified-using-http-conditional-headers-is-not-met"></a>Mensaje de error: The condition specified using HTTP conditional header(s) is not met (No se cumple la condición especificada mediante encabezados condicionales HTTP)

- **Síntomas**: Al utilizar una consulta SQL para extraer datos de Azure SQL Data Warehouse, se produce el siguiente error:

    ```
    ...StorageException: The condition specified using HTTP conditional header(s) is not met...
    ```

- **Causa**: Se produce un problema en Azure SQL Data Warehouse al consultar la tabla externa en Azure Storage.

- **Solución:** Ejecute la misma consulta en SSMS y compruebe si ve el mismo resultado. En caso afirmativo, abra una incidencia de soporte técnico para Azure SQL Data Warehouse y proporcione el nombre de la base de datos y el servidor de SQL Data Warehouse para solucionar el problema.
            

## <a name="delimited-text-format"></a>Formato de texto delimitado

### <a name="error-code--delimitedtextcolumnnamenotallownull"></a>Código de error:  DelimitedTextColumnNameNotAllowNull

- **Mensaje**: `The name of column index %index; is empty. Make sure column name is properly specified in the header row.`

- **Causa**: cuando establezca "firstRowAsHeader" en la actividad, se usará la primera fila como nombre de columna. Este error significa que la primera fila contiene un valor vacío. Por ejemplo: 'ColumnA,, ColumnB'.

- **Recomendación:**  compruebe la primera fila y corrija el valor si está vacío.


### <a name="error-code--delimitedtextmorecolumnsthandefined"></a>Código de error:  DelimitedTextMoreColumnsThanDefined

- **Mensaje**: `Error found when processing '%function;' source '%name;' with row number %rowCount;: found more columns than expected column count: %columnCount;.`

- **Causa**: el número de columnas de la fila problemática es mayor que el recuento de columnas de la primera fila. Puede deberse a un problema de datos o a una configuración incorrecta de los caracteres de la oferta o el delimitador de columna.

- **Recomendación:**  obtenga el número de filas en el mensaje de error, compruebe la columna de la fila y corrija los datos.

- **Causa**: si el número de columnas esperado es "1" en el mensaje de error, es posible que haya especificado una compresión o configuración de formato incorrecta, lo que ha causado que ADF analice los archivos erróneamente.

- **Recomendación:**  compruebe la configuración de formato para asegurarse de que coincide con los archivos de origen.

- **Causa**: si el origen es una carpeta, es posible que los archivos de la carpeta especificada tengan un esquema diferente.

- **Recomendación:**  asegúrese de que los archivos de la carpeta especificada tienen un esquema idéntico.


### <a name="error-code--delimitedtextincorrectrowdelimiter"></a>Código de error:  DelimitedTextIncorrectRowDelimiter

- **Mensaje**: `The specified row delimiter %rowDelimiter; is incorrect. Cannot detect a row after parse %size; MB data.`


### <a name="error-code--delimitedtexttoolargecolumncount"></a>Código de error:  DelimitedTextTooLargeColumnCount

- **Mensaje**: `Column count reaches limitation when deserializing csv file. Maximum size is '%size;'. Check the column delimiter and row delimiter provided. (Column delimiter: '%columnDelimiter;', Row delimiter: '%rowDelimiter;')`


### <a name="error-code--delimitedtextinvalidsettings"></a>Código de error:  DelimitedTextInvalidSettings

- **Mensaje**: `%settingIssues;`



## <a name="dynamics-365common-data-servicedynamics-crm"></a>Dynamics 365, Common Data Service, Dynamics CRM

### <a name="error-code--dynamicscreateserviceclienterror"></a>Código de error:  DynamicsCreateServiceClientError

- **Mensaje**: `This is a transient issue on dynamics server side. Try to rerun the pipeline.`

- **Causa**: se trata de un problema transitorio en el servidor de Dynamics.

- **Recomendación:**  vuelva a ejecutar la canalización. Si sigue sin funcionar, intente reducir el paralelismo. Si esto no funciona, póngase en contacto con el soporte técnico de Dynamics.



## <a name="json-format"></a>Formato JSON

### <a name="error-code--jsoninvalidarraypathdefinition"></a>Código de error:  JsonInvalidArrayPathDefinition

- **Mensaje**: `Error occurred when deserializing source JSON data. Check whether the JsonPath in JsonNodeReference and JsonPathDefintion is valid.`


### <a name="error-code--jsonemptyjobjectdata"></a>Código de error:  JsonEmptyJObjectData

- **Mensaje**: `The specified row delimiter %rowDelimiter; is incorrect. Cannot detect a row after parse %size; MB data.`


### <a name="error-code--jsonnullvalueinpathdefinition"></a>Código de error:  JsonNullValueInPathDefinition

- **Mensaje**: `Null JSONPath detected in JsonPathDefinition.`


### <a name="error-code--jsonunsupportedhierarchicalcomplexvalue"></a>Código de error:  JsonUnsupportedHierarchicalComplexValue

- **Mensaje**: `The retrieved type of data %data; with value %value; is not supported yet. Please either remove the targeted column '%name;' or enable skip incompatible row to skip the issue rows.`


### <a name="error-code--jsonconflictpartitiondiscoveryschema"></a>Código de error:  JsonConflictPartitionDiscoverySchema

- **Mensaje**: `Conflicting partition column names detected.'%schema;', '%partitionDiscoverySchema;'`


### <a name="error-code--jsoninvaliddataformat"></a>Código de error:  JsonInvalidDataFormat

- **Mensaje**: `Error occurred when deserializing source JSON file '%fileName;'. Check if the data is in valid JSON object format.`


### <a name="error-code--jsoninvaliddatamixedarrayandobject"></a>Código de error:  JsonInvalidDataMixedArrayAndObject

- **Mensaje**: `Error occurred when deserializing source JSON file '%fileName;'. The JSON format doesn't allow mixed arrays and objects.`



## <a name="parquet-format"></a>Formato Parquet

### <a name="error-code--parquetjavainvocationexception"></a>Código de error:  ParquetJavaInvocationException

- **Mensaje**: `An error occurred when invoking java, message: %javaException;.`

- **Causa**: si el mensaje de error contiene "java.lang.OutOfMemory", "Java heap space" y "doubleCapacity", suele ser un problema de administración de memoria en la versión anterior de Integration Runtime.

- **Recomendación:**  si usa Integration Runtime autohospedado y la versión es anterior a 3.20.7159.1, se sugiere actualizar a la versión más reciente.

- **Causa**: si el mensaje de error contiene " java.lang.OutOfMemory", Integration Runtime no tiene suficientes recursos para procesar los archivos.

- **Recomendación:**  limite las ejecuciones simultáneas en Integration Runtime. En el caso de Integration Runtime autohospedado, escale verticalmente a una máquina eficaz con una memoria igual o superior a 8 GB.

- **Causa**: si el mensaje de error contiene "NullPointerReference", es posible que se trata de un error transitorio.

- **Recomendación:**  Inténtelo de nuevo. Si el problema persiste, póngase en contacto con el soporte técnico.


### <a name="error-code--parquetinvalidfile"></a>Código de error:  ParquetInvalidFile

- **Mensaje**: `File is not a valid parquet file.`

- **Causa**: problema del archivo Parquet.

- **Recomendación:**  compruebe que la entrada es un archivo Parquet válido.


### <a name="error-code--parquetnotsupportedtype"></a>Código de error:  ParquetNotSupportedType

- **Mensaje**: `Unsupported Parquet type. PrimitiveType: %primitiveType; OriginalType: %originalType;.`

- **Causa**: el formato Parquet no se admite en Azure Data Factory.

- **Recomendación:**  compruebe los datos de origen. Consulte el documento: https://docs.microsoft.com/azure/data-factory/supported-file-formats-and-compression-codecs.


### <a name="error-code--parquetmisseddecimalprecisionscale"></a>Código de error:  ParquetMissedDecimalPrecisionScale

- **Mensaje**: `Decimal Precision or Scale information is not found in schema for column: %column;.`

- **Causa**: intenta analizar la precisión y la escala del número, pero no se proporciona esa información.

- **Recomendación:**  "Source" no devuelve la precisión y la escala correctas. Compruebe la precisión y la escala de la columna problemática.


### <a name="error-code--parquetinvaliddecimalprecisionscale"></a>Código de error:  ParquetInvalidDecimalPrecisionScale

- **Mensaje**: `Invalid Decimal Precision or Scale. Precision: %precision; Scale: %scale;.`

- **Causa**: el esquema no es válido.

- **Recomendación:**  Compruebe la precisión y la escala de la columna problemática.


### <a name="error-code--parquetcolumnnotfound"></a>Código de error:  ParquetColumnNotFound

- **Mensaje**: `Column %column; does not exist in Parquet file.`

- **Causa**: el esquema de origen no coincide con el esquema de receptor.

- **Recomendación:**  compruebe el valor "mappings" en "activity". Asegúrese de que la columna de origen se puede asignar a la columna de receptor correcta.


### <a name="error-code--parquetinvaliddataformat"></a>Código de error:  ParquetInvalidDataFormat

- **Mensaje**: `Incorrect format of %srcValue; for converting to %dstType;.`

- **Causa**: los datos no se pueden convertir al tipo especificado en mappings.source.

- **Recomendación:**  vuelva a comprobar los datos de origen o especifique el tipo de datos correcto para esta columna en la asignación de columnas de la actividad de copia. Consulte el documento: https://docs.microsoft.com/azure/data-factory/supported-file-formats-and-compression-codecs.


### <a name="error-code--parquetdatacountnotmatchcolumncount"></a>Código de error:  ParquetDataCountNotMatchColumnCount

- **Mensaje**: `The data count in a row '%sourceColumnCount;' does not match the column count '%sinkColumnCount;' in given schema.`

- **Causa**: el número de columnas de origen y el número de columnas de receptor no coinciden.

- **Recomendación:**  vuelva a comprobar que el número de columnas de origen sea igual al número de columnas de receptor en "mapping".


### <a name="error-code--parquetdatatypenotmatchcolumntype"></a>Código de error:  ParquetDataTypeNotMatchColumnType

- **Mensaje**: el tipo de datos %srcType; no coincide con el tipo de columna especificada %dstType; en la columna "%columnIndex;".

- **Causa**: los datos del origen no se pueden convertir al tipo definido en el receptor.

- **Recomendación:**  especifique un tipo correcto en mapping.sink.


### <a name="error-code--parquetbridgeinvaliddata"></a>Código de error:  ParquetBridgeInvalidData

- **Mensaje**: `%message;`

- **Causa**: el valor de datos supera la limitación.

- **Recomendación:**  Inténtelo de nuevo. Si el problema persiste, póngase en contacto con Microsoft.


### <a name="error-code--parquetunsupportedinterpretation"></a>Código de error:  ParquetUnsupportedInterpretation

- **Mensaje**: `The given interpretation '%interpretation;' of parquet format is not supported.`

- **Causa**: escenario no admitido.

- **Recomendación:**  el valor de "ParquetInterpretFor" no debe ser "sparkSql".


### <a name="error-code--parquetunsupportfilelevelcompressionoption"></a>Código de error:  ParquetUnsupportFileLevelCompressionOption

- **Mensaje**: `File level compression is not supported for Parquet.`

- **Causa**: escenario no admitido.

- **Recomendación:**  Quite "CompressionType" de la carga útil.



## <a name="general-copy-activity-error"></a>Error de actividad de copia general

### <a name="error-code--jrenotfound"></a>Código de error:  JreNotFound

- **Mensaje**: `Java Runtime Environment cannot be found on the Self-hosted Integration Runtime machine. It is required for parsing or writing to Parquet/ORC files. Make sure Java Runtime Environment has been installed on the Self-hosted Integration Runtime machine.`

- **Causa**: Integration Runtime autohospedado no puede encontrar Java Runtime. Se requiere Java Runtime para leer el origen específico.

- **Recomendación:**  compruebe el entorno de Integration Runtime y el documento de referencia: https://docs.microsoft.com/azure/data-factory/format-parquet#using-self-hosted-integration-runtime.


### <a name="error-code--wildcardpathsinknotsupported"></a>Código de error:  WildcardPathSinkNotSupported

- **Mensaje**: `Wildcard in path is not supported in sink dataset. Fix the path: '%setting;'.`

- **Causa**: el conjunto de datos del receptor no admite caracteres comodín.

- **Recomendación:**  compruebe el conjunto de valores del receptor y corrija la ruta de acceso sin el valor del carácter comodín.


### <a name="error-code--mappinginvalidpropertywithemptyvalue"></a>Código de error:  MappingInvalidPropertyWithEmptyValue

- **Mensaje**: `One or more '%sourceOrSink;' in copy activity mapping doesn't point to any data. Choose one of the three properties 'name', 'path' and 'ordinal' to reference columns/fields.`


### <a name="error-code--mappinginvalidpropertywithnamepathandordinal"></a>Código de error:  MappingInvalidPropertyWithNamePathAndOrdinal

- **Mensaje**: `Mixed properties are used to reference '%sourceOrSink;' columns/fields in copy activity mapping. Please only choose one of the three properties 'name', 'path' and 'ordinal'. The problematic mapping setting is 'name': '%name;', 'path': '%path;','ordinal': '%ordinal;'.`


### <a name="error-code--mappingduplicatedordinal"></a>Código de error:  MappingDuplicatedOrdinal

- **Mensaje**: `Copy activity 'mappings' has duplicated ordinal value "%Ordinal;". Fix the setting in 'mappings'.`


### <a name="error-code--mappinginvalidordinalforsinkcolumn"></a>Código de error:  MappingInvalidOrdinalForSinkColumn

- **Mensaje**: `Invalid 'ordinal' property for sink column under 'mappings' property. Ordinal: %Ordinal;.`


## <a name="next-steps"></a>Pasos siguientes

Para obtener ayuda para solucionar problemas, pruebe estos recursos:

*  [Blog de Data Factory](https://azure.microsoft.com/blog/tag/azure-data-factory/)
*  [Solicitud de características de Data Factory](https://feedback.azure.com/forums/270578-data-factory)
*  [Vídeos de Azure](https://azure.microsoft.com/resources/videos/index/?sort=newest&services=data-factory)
*  [Página de preguntas y respuestas de Microsoft](https://docs.microsoft.com/answers/topics/azure-data-factory.html)
*  [Foro de Stack Overflow para Data Factory](https://stackoverflow.com/questions/tagged/azure-data-factory)
*  [Información de Twitter sobre Data Factory](https://twitter.com/hashtag/DataFactory)
            
