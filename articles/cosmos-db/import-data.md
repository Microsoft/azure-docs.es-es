---
title: 'Tutorial: Herramienta de migración de base de datos para Azure Cosmos DB'
description: 'Tutorial: Obtenga información sobre cómo usar las herramientas de migración de datos de código abierto de Azure Cosmos DB para importar datos a Azure Cosmos DB desde varios orígenes, incluidos archivos MongoDB, SQL Server, Table Storage, Amazon DynamoDB, CSV y JSON. Conversión de CSV a JSON.'
author: deborahc
ms.service: cosmos-db
ms.topic: tutorial
ms.date: 08/31/2020
ms.author: dech
ms.openlocfilehash: 9992d6f1f9f1d0aad6f451d6a974f4df9f655881
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89255994"
---
# <a name="tutorial-use-data-migration-tool-to-migrate-your-data-to-azure-cosmos-db"></a>Tutorial: Use la herramienta de migración de datos para migrar los datos a Azure Cosmos DB

En este tutorial, se muestra cómo usar la herramienta de migración de datos de Azure Cosmos DB, que puede importar datos desde diversos orígenes a contenedores y tablas de Azure Cosmos. Puede importar archivos JSON, archivos CSV, SQL, MongoDB, Azure Table Storage, Amazon DynamoDB e incluso colecciones de SQL API de Azure Cosmos DB. Puede migrar esos datos a colecciones y tablas para usarlos con Azure Cosmos DB. También se puede utilizar la herramienta de migración de datos al migrar de una colección de partición única a una colección de varias particiones de la API de SQL.

¿Qué API se va a usar con Azure Cosmos DB?

* **[API de SQL ](documentdb-introduction.md)** : para importar los datos puede usar cualquiera de las opciones de origen disponibles en la herramienta de migración de datos.
* **[Table API](table-introduction.md)** : para importar los datos, puede usar la herramienta de migración de datos o AzCopy. Para más información, consulte [Importación de datos para su uso con Table API de Azure Cosmos DB](table-import.md).
* **[API de Azure Cosmos DB para MongoDB](mongodb-introduction.md)** : la herramienta de migración de datos no es compatible actualmente con la API de Azure Cosmos DB para MongoDB, independientemente de que sea origen o destino. Si quiere migrar los datos dentro o fuera de las colecciones en Azure Cosmos DB, consulte [Migración de los datos a una cuenta de la API de MongoDB para Azure Cosmos DB](mongodb-migrate.md) (Azure Cosmos DB: importar datos en la API de MongoDB), donde encontrará obtener instrucciones para hacerlo. De todos modos, todavía puede usar la herramienta de migración de datos para exportar datos desde MongoDB a las colecciones de API de SQL de Azure Cosmos DB, para así poder usarlos con la API de SQL.
* **[Gremlin API](graph-introduction.md)** : la herramienta de migración de datos no se admite como herramienta de importación para las cuentas de Gremlin API en este momento.

En este tutorial se describen las tareas siguientes:

> [!div class="checklist"]
> * Instalación de la herramienta de migración de datos
> * Importación de datos de orígenes de datos diferentes
> * Exportación desde Azure Cosmos DB a JSON

## <a name="prerequisites"></a><a id="Prerequisites"></a>Requisitos previos

Antes de seguir las instrucciones de este artículo, asegúrese de realizar los siguientes pasos:

* **Instale** [Microsoft .NET Framework 4.51](https://www.microsoft.com/download/developer-tools.aspx) o superior.

* **Aumento del rendimiento:** la duración de la migración de datos depende de la cantidad de rendimiento configurado para una sola colección o un conjunto de colecciones. Asegúrese de aumentar el rendimiento para migraciones de datos más grandes. Después de haber completado la migración, reduzca el rendimiento para ahorrar costos. Para más información acerca de cómo aumentar el rendimiento en Azure Portal, consulte [Niveles de rendimiento](performance-levels.md) y [planes de tarifa](https://azure.microsoft.com/pricing/details/cosmos-db/) de Azure Cosmos DB.

* **Creación de recursos de Azure Cosmos DB:** antes de comenzar la migración de datos, cree previamente todas las colecciones desde Azure Portal. Para migrar a una cuenta de Azure Cosmos DB que tiene un rendimiento de nivel de base de datos, proporcione una clave de partición al crear los contenedores de Azure Cosmos DB.

> [!IMPORTANT]
> Para asegurarse de que la herramienta de migración de datos usa la Seguridad de la capa de transporte (TLS) 1.2 al conectarse a sus cuentas de Azure Cosmos, use .NET Framework versión 4.7 o siga las instrucciones que se encuentran en [este artículo](https://docs.microsoft.com/dotnet/framework/network-programming/tls).

## <a name="overview"></a><a id="Overviewl"></a>Información general

La herramienta de migración de datos es una solución de código abierto que importa datos a Azure Cosmos DB desde una variedad de orígenes, como:

* Archivos JSON
* MongoDB
* SQL Server
* Archivos CSV
* Almacenamiento de tablas de Azure
* Amazon DynamoDB
* HBase
* Contenedores de Azure Cosmos

Aunque la herramienta de importación incluye una interfaz gráfica de usuario (dtui.exe), también se puede controlar desde la línea de comandos (dt.exe). De hecho, hay una opción para mostrar el comando asociado después de configurar una importación a través de la interfaz de usuario. Puede transformar datos tabulares de origen (por ejemplo, archivos de SQL Server o CSV) para crear relaciones jerárquicas (subdocumentos) durante la importación. Siga leyendo para más información acerca de las opciones de origen, los comandos de ejemplo para importar desde cada origen, las opciones de destino y la visualización de los resultados de la importación.

## <a name="installation"></a><a id="Install"></a>Instalación

El código fuente de la herramienta de migración está disponible en GitHub en [este repositorio](https://github.com/azure/azure-documentdb-datamigrationtool). Puede descargar y compilar la solución de manera local o bien [descargar un binario precompilado](https://aka.ms/csdmtool) y, después, ejecutar cualquiera de las siguientes:

* **Dtui.exe**: versión de interfaz gráfica de la herramienta
* **Dt.exe**: versión de línea de comandos de la herramienta

## <a name="select-data-source"></a>Selección de origen de datos

Una vez que haya instalado la herramienta, es el momento de importar los datos. ¿Qué tipo de datos desea importar?

* [Archivos JSON](#JSON)
* [MongoDB](#MongoDB)
* [Archivos de exportación de MongoDB](#MongoDBExport)
* [SQL Server](#SQL)
* [Archivos CSV](#CSV)
* [Azure Table Storage](#AzureTableSource)
* [Amazon DynamoDB](#DynamoDBSource)
* [Blob](#BlobImport)
* [Contenedores de Azure Cosmos](#SQLSource)
* [HBase](#HBaseSource)
* [Importación masiva de Azure Cosmos DB](#SQLBulkTarget)
* [Importación de registros secuenciales de Azure Cosmos DB](#SQLSeqTarget)

## <a name="import-json-files"></a><a id="JSON"></a>Importación de archivos JSON

La opción del importador de origen de archivos JSON permite importar uno o varios archivos JSON de un único documento o archivos JSON que tengan cada uno una matriz de documentos JSON. Cuando se agregan carpetas que contienen archivos JSON que se van a importar, existe la opción de buscar archivos en subcarpetas de forma recursiva.

:::image type="content" source="./media/import-data/jsonsource.png" alt-text="Captura de pantalla de opciones de origen de archivos JSON: herramientas de migración de base de datos":::

La cadena de conexión tiene el formato siguiente:

`AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>`

* `<CosmosDB Endpoint>` es el URI del punto de conexión. Puede encontrar este valor en Azure Portal. Vaya a la cuenta de Azure Cosmos. Abra el panel **Introducción** y copie el valor **URI**.
* `<AccountKey>` es la "Contraseña" o **CLAVE PRINCIPAL**. Puede encontrar este valor en Azure Portal. Vaya a la cuenta de Azure Cosmos. Abra el panel **Cadenas de conexión** o **Claves** y copie el valor de "Contraseña" o **CLAVE PRINCIPAL**.
* `<CosmosDB Database>` es el nombre de la base de datos de CosmosDB.

Ejemplo: `AccountEndpoint=https://myCosmosDBName.documents.azure.com:443/;AccountKey=wJmFRYna6ttQ79ATmrTMKql8vPri84QBiHTt6oinFkZRvoe7Vv81x9sn6zlVlBY10bEPMgGM982wfYXpWXWB9w==;Database=myDatabaseName`

> [!NOTE]
> Use el comando Verify para asegurarse de que se puede tener acceso a la cuenta de Cosmos DB especificada en el campo de la cadena de conexión.

Estos son algunos ejemplos de la línea de comandos para importar archivos JSON:

```console
#Import a single JSON file
dt.exe /s:JsonFile /s.Files:.\Sessions.json /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:Sessions /t.CollectionThroughput:2500

#Import a directory of JSON files
dt.exe /s:JsonFile /s.Files:C:\TESessions\*.json /t:DocumentDBBulk /t.ConnectionString:" AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:Sessions /t.CollectionThroughput:2500

#Import a directory (including sub-directories) of JSON files
dt.exe /s:JsonFile /s.Files:C:\LastFMMusic\**\*.json /t:DocumentDBBulk /t.ConnectionString:" AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:Music /t.CollectionThroughput:2500

#Import a directory (single), directory (recursive), and individual JSON files
dt.exe /s:JsonFile /s.Files:C:\Tweets\*.*;C:\LargeDocs\**\*.*;C:\TESessions\Session48172.json;C:\TESessions\Session48173.json;C:\TESessions\Session48174.json;C:\TESessions\Session48175.json;C:\TESessions\Session48177.json /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:subs /t.CollectionThroughput:2500

#Import a single JSON file and partition the data across 4 collections
dt.exe /s:JsonFile /s.Files:D:\\CompanyData\\Companies.json /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:comp[1-4] /t.PartitionKey:name /t.CollectionThroughput:2500
```

## <a name="import-from-mongodb"></a><a id="MongoDB"></a>Importación desde MongoDB

> [!IMPORTANT]
> Si va a importar a una cuenta de Cosmos configurada con la API de Azure Cosmos DB para MongoDB, siga estas [instrucciones](mongodb-migrate.md).

Con la opción del importador de origen de MongoDB puede importar desde una colección de MongoDB individual y, opcionalmente, filtrar documentos mediante una consulta o modificar la estructura del documento con una proyección.  

:::image type="content" source="./media/import-data/mongodbsource.png" alt-text="Captura de pantalla de las opciones de origen de MongoDB":::

La cadena de conexión tiene el formato estándar de MongoDB:

`mongodb://<dbuser>:<dbpassword>@<host>:<port>/<database>`

> [!NOTE]
> Utilice el comando Verify para asegurarse de que se puede tener acceso a la instancia de MongoDB especificada en el campo de la cadena de conexión.

Escriba el nombre de la colección desde la que se importarán los datos. Opcionalmente, puede especificar o facilitar un archivo para una consulta como, por ejemplo `{pop: {$gt:5000}}`, o una proyección, por ejemplo `{loc:0}`, para filtrar los datos que se van a importar y darles forma.

Estos son algunos ejemplos de la línea de comandos para importar desde MongoDB:

```console
#Import all documents from a MongoDB collection
dt.exe /s:MongoDB /s.ConnectionString:mongodb://<dbuser>:<dbpassword>@<host>:<port>/<database> /s.Collection:zips /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:BulkZips /t.IdField:_id /t.CollectionThroughput:2500

#Import documents from a MongoDB collection which match the query and exclude the loc field
dt.exe /s:MongoDB /s.ConnectionString:mongodb://<dbuser>:<dbpassword>@<host>:<port>/<database> /s.Collection:zips /s.Query:{pop:{$gt:50000}} /s.Projection:{loc:0} /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:BulkZipsTransform /t.IdField:_id/t.CollectionThroughput:2500
```

## <a name="import-mongodb-export-files"></a><a id="MongoDBExport"></a>Importación de archivos de exportación de MongoDB

> [!IMPORTANT]
> Si va a importar a una cuenta de Azure Cosmos DB compatible con MongoDB, siga estas [instrucciones](mongodb-migrate.md).

La opción del importador de origen de archivos JSON de exportación de MongoDB permite importar uno o más archivos JSON generados a partir de la utilidad mongoexport.  

:::image type="content" source="./media/import-data/mongodbexportsource.png" alt-text="Captura de pantalla de las opciones de origen de exportación de MongoDB":::

Cuando se agregan carpetas que contienen archivos JSON de exportación de MongoDB para importarlos, tiene la opción de buscar archivos en subcarpetas de forma recursiva.

A continuación se muestra un ejemplo de la línea de comandos para importar desde archivos JSON de exportación de MongoDB:

```console
dt.exe /s:MongoDBExport /s.Files:D:\mongoemployees.json /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:employees /t.IdField:_id /t.Dates:Epoch /t.CollectionThroughput:2500
```

## <a name="import-from-sql-server"></a><a id="SQL"></a>Importación desde SQL Server

La opción del importador de origen SQL permite importar desde una base de datos SQL Server individual y, opcionalmente, filtrar los registros que se pueden importar utilizando una consulta. Además, puede modificar la estructura del documento especificando un separador de anidamiento (más en un momento determinado).  

:::image type="content" source="./media/import-data/sqlexportsource.png" alt-text="Captura de pantalla de opciones de origen de SQL: herramientas de migración de base de datos":::

El formato de la cadena de conexión es el formato de la cadena de conexión SQL estándar.

> [!NOTE]
> Utilice el comando Verify para asegurarse de que se puede tener acceso a la instancia de SQL Server especificada en el campo de la cadena de conexión.

La propiedad de separador de anidamiento se utiliza para crear relaciones jerárquicas (documentos secundarios) durante la importación. Considere la siguiente consulta SQL:

`select CAST(BusinessEntityID AS varchar) as Id, Name, AddressType as [Address.AddressType], AddressLine1 as [Address.AddressLine1], City as [Address.Location.City], StateProvinceName as [Address.Location.StateProvinceName], PostalCode as [Address.PostalCode], CountryRegionName as [Address.CountryRegionName] from Sales.vStoreWithAddresses WHERE AddressType='Main Office'`

Que devuelve los resultados siguientes (parciales):

:::image type="content" source="./media/import-data/sqlqueryresults.png" alt-text="Captura de pantalla de los resultados de consultas SQL":::

Tenga en cuenta los alias como Address.AddressType y Address.Location.StateProvinceName. Al especificar el separador de anidamiento ".", la herramienta de importación crea los subdocumentos Address y Address.Location durante la importación. Este es un ejemplo de un documento resultante en Azure Cosmos DB:

*{ "id": "956", "Name": "Finer Sales and Service", "Address": { "AddressType": "Main Office", "AddressLine1": "#500-75 O'Connor Street", "Location": { "City": "Ottawa", "StateProvinceName": "Ontario" }, "PostalCode": "K4B 1S2", "CountryRegionName": "Canada" } }*

Estos son algunos ejemplos de la línea de comandos para importar desde SQL Server:

```console
#Import records from SQL which match a query
dt.exe /s:SQL /s.ConnectionString:"Data Source=<server>;Initial Catalog=AdventureWorks;User Id=advworks;Password=<password>;" /s.Query:"select CAST(BusinessEntityID AS varchar) as Id, * from Sales.vStoreWithAddresses WHERE AddressType='Main Office'" /t:DocumentDBBulk /t.ConnectionString:" AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:Stores /t.IdField:Id /t.CollectionThroughput:2500

#Import records from sql which match a query and create hierarchical relationships
dt.exe /s:SQL /s.ConnectionString:"Data Source=<server>;Initial Catalog=AdventureWorks;User Id=advworks;Password=<password>;" /s.Query:"select CAST(BusinessEntityID AS varchar) as Id, Name, AddressType as [Address.AddressType], AddressLine1 as [Address.AddressLine1], City as [Address.Location.City], StateProvinceName as [Address.Location.StateProvinceName], PostalCode as [Address.PostalCode], CountryRegionName as [Address.CountryRegionName] from Sales.vStoreWithAddresses WHERE AddressType='Main Office'" /s.NestingSeparator:. /t:DocumentDBBulk /t.ConnectionString:" AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:StoresSub /t.IdField:Id /t.CollectionThroughput:2500
```

## <a name="import-csv-files-and-convert-csv-to-json"></a><a id="CSV"></a>Importación de archivos CSV y conversión a JSON

La opción del importador de origen de archivos CSV le permite importar uno o varios archivos CSV. Cuando se agregan carpetas que contienen archivos CSV que se van a importar, existe la opción de buscar archivos en subcarpetas de forma recursiva.

:::image type="content" source="media/import-data/csvsource.png" alt-text="Captura de pantalla de opciones de origen de CSV: de CSV a JSON":::

De forma similar a lo que sucede con el origen SQL, la propiedad de separador de anidamiento se utiliza para crear relaciones jerárquicas (documentos secundarios) durante la importación. Tenga en cuenta las siguientes filas de datos y la fila de encabezado CSV:

:::image type="content" source="./media/import-data/csvsample.png" alt-text="Captura de pantalla de registros de ejemplo de CSV: de CSV a JSON":::

Tenga en cuenta los alias como DomainInfo.Domain_Name y RedirectInfo.Redirecting. Al especificar un separador de anidamiento de ".", la herramienta de importación crea subdocumentos DomainInfo y RedirectInfo durante la importación. Este es un ejemplo de un documento resultante en Azure Cosmos DB:

*{ "DomainInfo": { "Domain_Name": "ACUS.GOV", "Domain_Name_Address": "https:\//www.ACUS.GOV" }, "Federal Agency": "Administrative Conference of the Estados Unidos", "RedirectInfo": { "Redirecting": "0", "Redirect_Destination": "" }, "id": "9cc565c5-ebcd-1c03-ebd3-cc3e2ecd814d" }*

La herramienta de importación intenta inferir la información de tipo de los valores sin comillas de los archivos CSV (los valores entre comillas se tratan siempre como cadenas).  Los tipos se identifican en el siguiente orden: número, fecha y hora, booleano.  

Hay otras dos cosas que debe saber sobre la importación de archivos CSV:

1. De forma predeterminada, los valores sin comillas se recortan siempre en tabuladores y espacios, mientras que los valores entre comillas se mantienen como son. Este comportamiento se puede invalidar con la casilla Recortar valores entre comillas o la opción de línea de comandos /s.TrimQuoted.
2. De forma predeterminada, un valor null sin comillas se trata como un valor null. Este comportamiento se puede invalidar (es decir, tratar los valores null sin comillas como si fueran cadenas "null") con la casilla Tratar NULL sin comillas como cadena o la opción de línea de comandos /s.NoUnquotedNulls.

Este es un ejemplo de línea de comandos para la importación CSV:

```console
dt.exe /s:CsvFile /s.Files:.\Employees.csv /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:Employees /t.IdField:EntityID /t.CollectionThroughput:2500
```

## <a name="import-from-azure-table-storage"></a><a id="AzureTableSource"></a>Importación de almacenamiento de tablas de Azure

La opción del importador de código fuente de Azure Table Storage permite importar desde una tabla individual de Azure Table Storage. Si lo desea, puede filtrar las entidades de tabla que se van a importar.

Puede generar datos que se importen de Azure Table Storage a tablas y entidades de Azure Cosmos DB para usarlos con Table API. Los datos importados también se pueden enviar a colecciones y documentos para su uso con la API de SQL. Sin embargo, Table API solo está disponible como destino en la utilidad de línea de comandos. No se puede exportar a Table API mediante el uso de la interfaz de usuario de la herramienta de migración de datos. Para más información, consulte [Importación de datos para su uso con Table API de Azure Cosmos DB](table-import.md).

:::image type="content" source="./media/import-data/azuretablesource.png" alt-text="Captura de pantalla de las opciones de origen de Azure Table Storage":::

El formato de la cadena de conexión de almacenamiento de tablas de Azure es:

`DefaultEndpointsProtocol=<protocol>;AccountName=<Account Name>;AccountKey=<Account Key>;`

> [!NOTE]
> Utilice el comando Verify para asegurarse de que se puede tener acceso a la instancia de almacenamiento de tablas de Azure especificada en el campo de la cadena de conexión.

Escriba el nombre de la tabla de Azure desde la que se importarán los datos. Opcionalmente, puede especificar un [filtro](../vs-azure-tools-table-designer-construct-filter-strings.md).

La opción del importador de origen de almacenamiento de tablas de Azure tiene las siguientes opciones adicionales:

1. Incluir campos internos
   1. All: incluir todos los campos internos (PartitionKey, RowKey y Timestamp)
   2. None: excluir todos los campos internos
   3. RowKey: incluir sólo el campo RowKey
2. Seleccionar columnas
   1. Los filtros de almacenamiento de tablas de Azure no admiten proyecciones. Si desea importar sólo las propiedades específicas de la entidad de tablas de Azure, agréguelas a la lista Seleccionar columnas. Se omiten todas las demás propiedades de la entidad.

A continuación se muestra un ejemplo de línea de comandos para importar desde Azure Table Storage:

```console
dt.exe /s:AzureTable /s.ConnectionString:"DefaultEndpointsProtocol=https;AccountName=<Account Name>;AccountKey=<Account Key>" /s.Table:metrics /s.InternalFields:All /s.Filter:"PartitionKey eq 'Partition1' and RowKey gt '00001'" /s.Projection:ObjectCount;ObjectSize  /t:DocumentDBBulk /t.ConnectionString:" AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:metrics /t.CollectionThroughput:2500
```

## <a name="import-from-amazon-dynamodb"></a><a id="DynamoDBSource"></a>Importar desde Amazon DynamoDB

La opción del importador de origen Amazon DynamoDB le permite importar desde una sola tabla de Amazon DynamoDB. Este puede filtrar las entidades de tabla que se van a importar. Se proporcionan varias plantillas para que configurar una importación resulte lo más fácil posible.

:::image type="content" source="./media/import-data/dynamodbsource1.png" alt-text="Captura de pantalla de opciones de origen de Amazon DynamoD: herramientas de migración de base de datos":::

:::image type="content" source="./media/import-data/dynamodbsource2.png" alt-text="Captura de pantalla de opciones de origen de Amazon DynamoD: herramientas de migración de base de datos":::

El formato de la cadena de conexión de Amazon DynamoDB es:

`ServiceURL=<Service Address>;AccessKey=<Access Key>;SecretKey=<Secret Key>;`

> [!NOTE]
> Use el comando Verify para asegurarse de que se puede tener acceso a la instancia de Amazon DynamoDB especificada en el campo de la cadena de conexión.

A continuación se muestra un ejemplo de línea de comandos para importar desde Amazon DynamoDB:

```console
dt.exe /s:DynamoDB /s.ConnectionString:ServiceURL=https://dynamodb.us-east-1.amazonaws.com;AccessKey=<accessKey>;SecretKey=<secretKey> /s.Request:"{   """TableName""": """ProductCatalog""" }" /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<Azure Cosmos DB Endpoint>;AccountKey=<Azure Cosmos DB Key>;Database=<Azure Cosmos database>;" /t.Collection:catalogCollection /t.CollectionThroughput:2500
```

## <a name="import-from-azure-blob-storage"></a><a id="BlobImport"></a>Importación desde Azure Blob Storage

El archivo JSON, archivo de exportación de MongoDB y opciones de importador de origen de archivo CSV permiten importar uno o más archivos del almacenamiento de blobs de Azure. Después de especificar una dirección URL de contenedor de blobs y una clave de cuenta, proporcione una expresión regular para seleccionar los archivos para importar.

:::image type="content" source="./media/import-data/blobsource.png" alt-text="Captura de pantalla de opciones de origen de archivos Blob":::

Este es el ejemplo de línea de comandos para importar archivos JSON desde Azure Blob Storage:

```console
dt.exe /s:JsonFile /s.Files:"blobs://<account key>@account.blob.core.windows.net:443/importcontainer/.*" /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:doctest
```

## <a name="import-from-a-sql-api-collection"></a><a id="SQLSource"></a>Importación desde una colección de API de SQL

La opción del importador de origen de Azure Cosmos DB permite importar datos de uno o varios contenedores de Azure Cosmos y, si lo desea, filtrar los documentos mediante una consulta.  

:::image type="content" source="./media/import-data/documentdbsource.png" alt-text="Captura de pantalla de opciones de origen de Azure Cosmos DB":::

El formato de la cadena de conexión de Azure Cosmos DB es:

`AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;`

Puede recuperar la cadena de conexión de la cuenta de Azure Cosmos DB desde la página de claves de Azure Portal, como se describe en [Administración de una cuenta de Azure Cosmos DB](manage-account.md). Sin embargo, el nombre de la base de datos se debe anexar a la cadena de conexión con el formato siguiente:

`Database=<CosmosDB Database>;`

> [!NOTE]
> Use el comando Verify para asegurarse de que se puede tener acceso a la instancia de Azure Cosmos DB especificada en el campo de la cadena de conexión.

Para realizar la importación desde un solo contenedor de Azure Cosmos, escriba el nombre de la colección desde la que se van a importar los datos. Para realizar la importación desde varios contenedores de Azure Cosmos, especifique una expresión regular que coincida con uno o varios nombres de colección (por ejemplo, collection01 | collection02 | collection03). Opcionalmente, puede especificar o facilitar un archivo para una consulta para filtrar los datos que se van a importar y darles forma.

> [!NOTE]
> Dado que el campo de colecciones acepta expresiones regulares, si la importación se va a realizar de una sola colección cuyo nombre contenga caracteres de expresión regular, dichos caracteres deben incluir una secuencia de escape.

La opción del importador de origen de Azure Cosmos DB tiene las siguientes opciones avanzadas:

1. Incluir campos internos: especifica si se debe o no incluir propiedades del sistema de documentos de Azure Cosmos DB en la exportación (por ejemplo, _rid, _ts).
2. Número de reintentos en caso de error: especifica el número de veces que se intentará la conexión a Azure Cosmos DB en caso de errores transitorios (por ejemplo, interrupciones de conectividad de red).
3. Intervalo de reintento: especifica el tiempo de espera entre los reintentos de restablecimiento de conexión a Azure Cosmos DB en caso de errores transitorios (por ejemplo, interrupción de la conectividad de red).
4. Modo de conexión: especifica el modo de conexión para utilizar con Azure Cosmos DB. Las opciones disponibles son DirectTcp, DirectHttps y puerta de enlace. Los modos de conexión directa son más rápidos, mientras que el modo de puerta de enlace es compatible con el firewall, ya que sólo usa el puerto 443.

:::image type="content" source="./media/import-data/documentdbsourceoptions.png" alt-text="Captura de pantalla de opciones avanzadas de origen de Azure Cosmos DB":::

> [!TIP]
> El modo de conexión predeterminado de la herramienta de importación es DirectTcp. Si experimenta problemas de firewall, cambie al modo de conexión puerta de enlace, ya que sólo requiere el puerto 443.

Estos son algunos ejemplos de línea de comandos para importar desde Azure Cosmos DB:

```console
#Migrate data from one Azure Cosmos container to another Azure Cosmos containers
dt.exe /s:DocumentDB /s.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /s.Collection:TEColl /t:DocumentDBBulk /t.ConnectionString:" AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:TESessions /t.CollectionThroughput:2500

#Migrate data from more than one Azure Cosmos container to a single Azure Cosmos container
dt.exe /s:DocumentDB /s.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /s.Collection:comp1|comp2|comp3|comp4 /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:singleCollection /t.CollectionThroughput:2500

#Export an Azure Cosmos container to a JSON file
dt.exe /s:DocumentDB /s.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /s.Collection:StoresSub /t:JsonFile /t.File:StoresExport.json /t.Overwrite /t.CollectionThroughput:2500
```

> [!TIP]
> La herramienta de importación de datos de Azure Cosmos DB también admite la importación de datos desde el [Emulador de Azure Cosmos DB](local-emulator.md). Al importar datos desde un emulador local, establezca el punto de conexión en `https://localhost:<port>`.

## <a name="import-from-hbase"></a><a id="HBaseSource"></a>Importar desde HBase

La opción del importador de origen de HBase le permite importar datos de una tabla HBase y, opcionalmente, filtrar los datos. Se proporcionan varias plantillas para que configurar una importación resulte lo más fácil posible.

:::image type="content" source="./media/import-data/hbasesource1.png" alt-text="Captura de pantalla de opciones de origen de HBase":::

:::image type="content" source="./media/import-data/hbasesource2.png" alt-text="Captura de pantalla de opciones de origen de HBase":::

El formato de la cadena de conexión de HBase Stargate es:

`ServiceURL=<server-address>;Username=<username>;Password=<password>`

> [!NOTE]
> Use el comando Verify para asegurarse de que se puede tener acceso a la instancia de HBase especificada en el campo de la cadena de conexión.

A continuación se muestra un ejemplo de línea de comandos para importar desde HBase:

```console
dt.exe /s:HBase /s.ConnectionString:ServiceURL=<server-address>;Username=<username>;Password=<password> /s.Table:Contacts /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:hbaseimport
```

## <a name="import-to-the-sql-api-bulk-import"></a><a id="SQLBulkTarget"></a>Importación a la API de SQL (importación en bloque)

El importador masivo de Azure Cosmos DB permite importar desde cualquier opción de origen disponible con un procedimiento almacenado de Azure Cosmos DB para que resulte más eficaz. La herramienta admite la importación a un contenedor de Azure Cosmos con una sola partición. También admite la importación con particiones en la cual se dividen los datos en más de un contenedor de Azure Cosmos con una sola partición. Para más información sobre la creación de particiones de datos, consulte [Partición y escalado de datos en Azure Cosmos DB](partition-data.md). La herramienta crea, ejecuta y elimina el procedimiento almacenado de las colecciones de destino.  

:::image type="content" source="./media/import-data/documentdbbulk.png" alt-text="Captura de pantalla de opciones de importación masiva de Azure Cosmos DB":::

El formato de la cadena de conexión de Azure Cosmos DB es:

`AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;`

La cadena de conexión de la cuenta de Azure Cosmos DB se puede obtener en la página Claves de Azure Portal, como se describe en [Administración de una cuenta de Azure Cosmos DB](manage-account.md); sin embargo, el nombre de la base de datos se debe anexar a la cadena de conexión con el siguiente formato:

`Database=<CosmosDB Database>;`

> [!NOTE]
> Use el comando Verify para asegurarse de que se puede tener acceso a la instancia de Azure Cosmos DB especificada en el campo de la cadena de conexión.

Para realizar la importación en una sola colección, escriba el nombre de la colección desde la que se van a importar los datos y haga clic en el botón Agregar. Para importar en varias colecciones, escriba el nombre de cada colección individualmente o use la siguiente sintaxis para especificar varias colecciones: *collection_prefix*[índice inicial - índice final]. Si va a especificar varias colecciones a través de esta sintaxis, tenga en cuenta las directrices siguientes:

1. Solo se admiten patrones de nombre de intervalo entero. Por ejemplo, si se especifica colección [0-3], se crean las siguientes colecciones: colección0, colección1, colección2, colección3.
2. Puede usar una sintaxis abreviada: colección[3] crea el mismo conjunto de colecciones que el paso 1.
3. Se pueden proporcionar varias sustituciones. Por ejemplo, colección[0-1] [0-9] genera 20 nombres de colección con ceros delante (colección01,... 02... 03).

Una vez especificados los nombres de las colecciones, elija la capacidad de proceso deseada de la colección (de 400 RU a 10 000 RU). Para conseguir el mejor rendimiento en la importación, elija una mayor capacidad de proceso. Para más información sobre los niveles de rendimiento, consulte [Niveles de rendimiento en Azure Cosmos DB](performance-levels.md).

> [!NOTE]
> La configuración del nivel de rendimiento solo se aplica a la creación de la colección. Si la colección especificada ya existe, su capacidad de proceso no se modificará.

Cuando se importa a varias colecciones, la herramienta de importación admite el particionamiento basado en hash. En este escenario, especifique la propiedad de documento que desea usar como clave de partición. (Si la clave de partición se deja en blanco, los documentos se particionan aleatoriamente entre las colecciones de destino).

También puede especificar qué campo del origen de la importación debe utilizarse como la propiedad de identificador de documentos de Azure Cosmos DB durante la importación. Si los documentos no tienen esta propiedad, la herramienta de importación genera un GUID como el valor de la propiedad de identificador.

Hay una serie de opciones avanzadas disponibles durante la importación. En primer lugar, mientras que la herramienta incluye un procedimiento predeterminado almacenado de importación masiva (BulkInsert.js), puede especificar su propio procedimiento almacenado de importación:

 :::image type="content" source="./media/import-data/bulkinsertsp.png" alt-text="Captura de pantalla de la opción de importación de insert sproc masiva de Azure Cosmos DB":::

Además, al importar los tipos de fecha (por ejemplo, desde SQL Server o MongoDB), puede elegir entre tres opciones de importación:

 :::image type="content" source="./media/import-data/datetimeoptions.png" alt-text="Captura de pantalla de opciones de importación de fecha y hora de Azure Cosmos DB":::

* Cadena: conservar como un valor de cadena
* Época: conservar como un valor de número de tiempo
* Ambos: conservar los valores de cadena y de número. Esta opción crea un subdocumento, por ejemplo, "date_joined": { "Value": "2013-10-21T21:17:25.2410000Z", "Epoch": 1382390245 }

El importador masivo de Azure Cosmos DB tiene las siguientes opciones avanzadas adicionales:

1. Tamaño del lote: el tamaño de lote predeterminado de la herramienta es 50.  Si los documentos que desea importar son grandes, considere la posibilidad de reducir el tamaño del lote. Si los documentos que desea importar son pequeños, considere la posibilidad de aumentar el tamaño del lote.
2. Tamaño máximo de script (bytes): el valor predeterminado máximo del script en la herramienta es de 512 KB.
3. Deshabilitar generación de identificador automática: si cada documento que se va a importar contiene un campo de identificador, seleccione esta opción para aumentar el rendimiento. No se importan los documentos que no contengan un campo de identificador exclusivo.
4. Actualización de documentos existentes: la opción predeterminada es no reemplazar los documentos existentes por conflictos de identificador. Si selecciona esta opción, permite que se sobrescriban los documentos existentes con identificadores que coinciden entre sí. Esta característica es útil para las migraciones de datos programadas que actualizan los documentos existentes.
5. Número de reintentos en caso de error: especifica el número de veces que se intentará la conexión a Azure Cosmos DB durante errores transitorios (por ejemplo, interrupciones de conectividad de red).
6. Intervalo de reintento: especifica el tiempo de espera entre los reintentos de restablecimiento de conexión a Azure Cosmos DB en caso de errores transitorios (por ejemplo, interrupción de la conectividad de red).
7. Modo de conexión: especifica el modo de conexión para utilizar con Azure Cosmos DB. Las opciones disponibles son DirectTcp, DirectHttps y puerta de enlace. Los modos de conexión directa son más rápidos, mientras que el modo de puerta de enlace es compatible con el firewall, ya que sólo usa el puerto 443.

:::image type="content" source="./media/import-data/docdbbulkoptions.png" alt-text="Captura de pantalla de opciones avanzadas de importación masiva de Azure Cosmos DB":::

> [!TIP]
> El modo de conexión predeterminado de la herramienta de importación es DirectTcp. Si experimenta problemas de firewall, cambie al modo de conexión puerta de enlace, ya que sólo requiere el puerto 443.

## <a name="import-to-the-sql-api-sequential-record-import"></a><a id="SQLSeqTarget"></a>Importación a la API de SQL (importación de registros secuenciales)

El importador de registros secuenciales de Azure Cosmos DB permite importar desde una opción de origen disponible según cada registro. Puede elegir esta opción si va a importar a una colección existente que ha alcanzado su cuota de procedimientos almacenados. La herramienta admite la importación a un solo contenedor de Azure Cosmos (con una sola partición o con varias). También admite la importación con particiones en la cual se dividen los datos en más de un contenedor de Azure Cosmos con una sola partición o con varias. Para más información sobre la creación de particiones de datos, consulte [Partición y escalado de datos en Azure Cosmos DB](partition-data.md).

:::image type="content" source="./media/import-data/documentdbsequential.png" alt-text="Captura de pantalla de opciones de importación de registros secuenciales de Azure Cosmos DB":::

El formato de la cadena de conexión de Azure Cosmos DB es:

`AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;`

Puede recuperar la cadena de conexión de la cuenta de Azure Cosmos DB desde la página de claves de Azure Portal, como se describe en [Administración de una cuenta de Azure Cosmos DB](manage-account.md). Sin embargo, el nombre de la base de datos se debe anexar a la cadena de conexión con el formato siguiente:

`Database=<Azure Cosmos database>;`

> [!NOTE]
> Use el comando Verify para asegurarse de que se puede tener acceso a la instancia de Azure Cosmos DB especificada en el campo de la cadena de conexión.

Para realizar la importación en una sola colección, escriba el nombre de la colección en la que se van a importar los datos y, después, haga clic en el botón Agregar. Para importar a más de una colección, escriba el nombre de cada colección individualmente. También puede usar la siguiente sintaxis para especificar más de una colección: *collection_prefix*[índice inicial - índice final]. Si va a especificar varias colecciones a través de esta sintaxis, tenga en cuenta las instrucciones siguientes:

1. Solo se admiten patrones de nombre de intervalo entero. Por ejemplo, si se especifica colección [0-3], se crean las siguientes colecciones: colección0, colección1, colección2, colección3.
2. Puede usar una sintaxis abreviada: colección[3] crea el mismo conjunto de colecciones que el paso 1.
3. Se pueden proporcionar varias sustituciones. Por ejemplo, colección[0-1] [0-9] crea 20 nombres de colección con ceros delante (colección01,... 02... 03).

Una vez especificados los nombres de las colecciones, elija la capacidad de proceso deseada de la colección (de 400 RU a 250 000 RU). Para conseguir el mejor rendimiento en la importación, elija una mayor capacidad de proceso. Para más información sobre los niveles de rendimiento, consulte [Niveles de rendimiento en Azure Cosmos DB](performance-levels.md). Cualquier importación a colecciones con una capacidad de proceso de más de 10 000 RU requiere una clave de partición. Si elige tener más de 250 000 RU, debe registrar una solicitud en el portal para que le aumenten su cuenta.

> [!NOTE]
> La configuración del rendimiento solo se aplica a la creación de colecciones o bases de datos. Si la colección especificada ya existe, su capacidad de proceso no se modificará.

Cuando se importa a varias colecciones, la herramienta de importación admite el particionamiento basado en hash. En este escenario, especifique la propiedad de documento que desea usar como clave de partición. (Si la clave de partición se deja en blanco, los documentos se particionan aleatoriamente entre las colecciones de destino).

También puede especificar qué campo del origen de la importación debe utilizarse como la propiedad de identificador de documentos de Azure Cosmos DB durante la importación. (Si los documentos no tienen esta propiedad, la herramienta de importación genera un GUID como el valor de la propiedad de identificador).

Hay una serie de opciones avanzadas disponibles durante la importación. En primer lugar, al importar los tipos de fecha (por ejemplo, desde SQL Server o MongoDB), puede elegir entre tres opciones de importación:

 :::image type="content" source="./media/import-data/datetimeoptions.png" alt-text="Captura de pantalla de opciones de importación de fecha y hora de Azure Cosmos DB":::

* Cadena: conservar como un valor de cadena
* Época: conservar como un valor de número de tiempo
* Ambos: conservar los valores de cadena y de número. Esta opción crea un subdocumento, por ejemplo, "date_joined": { "Value": "2013-10-21T21:17:25.2410000Z", "Epoch": 1382390245 }

El importador de registros secuenciales de Azure Cosmos DB tiene estas opciones avanzadas adicionales:

1. Número de solicitudes paralelas: la opción predeterminada es dos. Si los documentos que desea importar son pequeños, considere la posibilidad de aumentar el número de solicitudes paralelas. Si este número es demasiado elevado, podrá haber limitaciones en la velocidad de la importación.
2. Deshabilitar generación de identificador automática: si cada documento que se va a importar contiene un campo de identificador, seleccione esta opción para aumentar el rendimiento. No se importan los documentos que no contengan un campo de identificador exclusivo.
3. Actualización de documentos existentes: la opción predeterminada es no reemplazar los documentos existentes por conflictos de identificador. Si selecciona esta opción, permite que se sobrescriban los documentos existentes con identificadores que coinciden entre sí. Esta característica es útil para las migraciones de datos programadas que actualizan los documentos existentes.
4. Número de reintentos en caso de error: especifica el número de veces que se intentará la conexión a Azure Cosmos DB durante errores transitorios (por ejemplo, interrupciones de conectividad de red).
5. Intervalo de reintento: especifica el tiempo de espera entre los reintentos de restablecimiento de conexión a Azure Cosmos DB durante errores transitorios (por ejemplo, interrupción de la conectividad de red).
6. Modo de conexión: especifica el modo de conexión para utilizar con Azure Cosmos DB. Las opciones disponibles son DirectTcp, DirectHttps y puerta de enlace. Los modos de conexión directa son más rápidos, mientras que el modo de puerta de enlace es compatible con el firewall, ya que sólo usa el puerto 443.

:::image type="content" source="./media/import-data/documentdbsequentialoptions.png" alt-text="Captura de pantalla de opciones avanzadas de importación de registros secuenciales de Azure Cosmos DB":::

> [!TIP]
> El modo de conexión predeterminado de la herramienta de importación es DirectTcp. Si experimenta problemas de firewall, cambie al modo de conexión puerta de enlace, ya que sólo requiere el puerto 443.

## <a name="specify-an-indexing-policy"></a><a id="IndexingPolicy"></a>Especificación de una directiva de indexación

Si permite que la herramienta de migración cree colecciones de API de SQL de Azure Cosmos DB durante la importación, puede especificar la directiva de indexación de esas colecciones. En la sección de opciones avanzadas de las opciones de Importación masiva de Azure Cosmos DB y Registro secuencial de Azure Cosmos DB, vaya a la sección de la directiva de indexación.

:::image type="content" source="./media/import-data/indexingpolicy1.png" alt-text="Captura de pantalla de opciones avanzadas de política de indexación de Azure Cosmos DB":::

Mediante la opción avanzada Directiva de indexación, puede seleccionar un archivo correspondiente, especificar manualmente una directiva de indexación o seleccionar de un conjunto de plantillas predeterminadas (haciendo clic con el botón derecho en el cuadro de texto de la que desee).

La herramienta proporciona las plantillas de directiva siguientes:

* Predeterminada. Esta directiva es mejor al realizar consultas de igualdad en cadenas. También funciona si usa consultas ORDER BY, de intervalo y de igualdad para números. Esta directiva tiene una sobrecarga de almacenamiento de índices menor que la de intervalo.
* Intervalo. Esta directiva es la mejor cuando usa consultas ORDER BY, de intervalo o igualdad tanto en números como en cadenas. Esta directiva tiene una mayor sobrecarga de almacenamiento de índice que la Predeterminada o Hash.

:::image type="content" source="./media/import-data/indexingpolicy2.png" alt-text="Captura de pantalla de opciones avanzadas de política de indexación de Azure Cosmos DB":::

> [!NOTE]
> Si no especifica una directiva de indexación, se aplica la directiva predeterminada. Para más información sobre las directivas de indexación, consulte [Directivas de indexación de Azure Cosmos DB](index-policy.md).

## <a name="export-to-json-file"></a>Exportación a archivos JSON

El exportador JSON de Azure Cosmos DB permite exportar cualquiera de las opciones de origen disponibles a un archivo JSON que contiene una matriz de documentos JSON. La herramienta controlará la exportación en su lugar. O bien, puede elegir ver el comando resultante de la migración y ejecutarlo usted mismo. El archivo JSON resultante puede almacenarse localmente o en el almacenamiento de blobs de Azure.

:::image type="content" source="./media/import-data/jsontarget.png" alt-text="Captura de pantalla de opción de exportación de archivo local JSON de Azure Cosmos DB":::

:::image type="content" source="./media/import-data/jsontarget2.png" alt-text="Captura de pantalla de opción de exportación de Azure Blob Storage de JSON de Azure Cosmos DB":::

Opcionalmente, puede optar por adornar el JSON resultante. Esta acción aumentará el tamaño del documento resultante al mismo tiempo que el contenido será más legible.

* Exportación JSON estándar

  ```JSON
  [{"id":"Sample","Title":"About Paris","Language":{"Name":"English"},"Author":{"Name":"Don","Location":{"City":"Paris","Country":"France"}},"Content":"Don's document in Azure Cosmos DB is a valid JSON document as defined by the JSON spec.","PageViews":10000,"Topics":[{"Title":"History of Paris"},{"Title":"Places to see in Paris"}]}]
  ```

* Exportación JSON adornada

  ```JSON
    [
     {
    "id": "Sample",
    "Title": "About Paris",
    "Language": {
      "Name": "English"
    },
    "Author": {
      "Name": "Don",
      "Location": {
        "City": "Paris",
        "Country": "France"
      }
    },
    "Content": "Don's document in Azure Cosmos DB is a valid JSON document as defined by the JSON spec.",
    "PageViews": 10000,
    "Topics": [
      {
        "Title": "History of Paris"
      },
      {
        "Title": "Places to see in Paris"
      }
    ]
    }]
  ```

Este es el ejemplo de línea de comandos para importar el archivo JSON en Azure Blob Storage:

```console
dt.exe /ErrorDetails:All /s:DocumentDB /s.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB database_name>" /s.Collection:<CosmosDB collection_name>
/t:JsonFile /t.File:"blobs://<Storage account key>@<Storage account name>.blob.core.windows.net:443/<Container_name>/<Blob_name>"
/t.Overwrite
```

## <a name="advanced-configuration"></a>Configuración avanzada

En la pantalla Configuración avanzada, especifique la ubicación del archivo de registro en que desee que se escriban los errores. Las siguientes reglas se aplican a esta página:

1. Si no se proporciona un nombre de archivo, todos los errores se devuelven en la página de resultados.
2. Si se proporciona un nombre de archivo sin un directorio, el archivo se crea (o sobrescribe) en el directorio del entorno actual.
3. Si selecciona un archivo existente, este se sobrescribe, ya que no hay opción de anexar.
4. A continuación, elija si desea registrar todos los mensajes de error, los críticos o ninguno. Por último, decida con qué frecuencia se actualiza el progreso en el mensaje de transferencia en pantalla.

   :::image type="content" source="./media/import-data/AdvancedConfiguration.png" alt-text="Captura de la pantalla de configuración avanzada":::

## <a name="confirm-import-settings-and-view-command-line"></a>Confirmación de las opciones de importación y visualización de la línea de comandos

1. Después de especificar la información de origen, la de destino y la configuración avanzada, revise el resumen de la migración y vea o copie el comando resultante de la migración si lo desea. (Copiar el comando es útil para automatizar las operaciones de importación).

    :::image type="content" source="./media/import-data/summary.png" alt-text="Captura de la pantalla de resumen":::

    :::image type="content" source="./media/import-data/summarycommand.png" alt-text="Captura de la pantalla de resumen":::

2. Una vez que esté satisfecho con las opciones de origen y de destino, haga clic en **Importar**. El tiempo transcurrido, la cantidad de elementos transferidos y la información sobre los errores (si no se ha especificado un nombre de archivo en la configuración avanzada) se actualizarán mientras la importación esté curso. Una vez que haya finalizado, puede exportar los resultados (por ejemplo, para tratar los errores de importación).

    :::image type="content" source="./media/import-data/viewresults.png" alt-text="Captura de pantalla de opción de exportación de JSON de Azure Cosmos DB":::

3. También puede iniciar una nueva importación restableciendo todos los valores o conservando la configuración existente. (Por ejemplo, puede decidir mantener la información de la cadena de conexión, la elección de origen y destino, etc).

    :::image type="content" source="./media/import-data/newimport.png" alt-text="Captura de pantalla de opción de exportación de JSON de Azure Cosmos DB":::

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha realizado las tareas siguientes:

> [!div class="checklist"]
> * Instalación de la herramienta de migración de datos
> * Importación de datos de orígenes de datos diferentes
> * Exportación desde Azure Cosmos DB a JSON

Ahora puede pasar al siguiente tutorial y obtener más información sobre cómo consultar los datos con Azure Cosmos DB.

> [!div class="nextstepaction"]
>[Consulta de datos](../cosmos-db/tutorial-query-sql-api.md)
