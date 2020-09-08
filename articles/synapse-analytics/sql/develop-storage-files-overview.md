---
title: Acceso a los archivos que se encuentran en el almacenamiento en SQL a petición (versión preliminar)
description: Describe la consulta de archivos de almacenamiento mediante recursos de SQL a petición (versión preliminar) en Synapse SQL.
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 04/19/2020
ms.author: v-stazar
ms.reviewer: jrasnick, carlrab
ms.openlocfilehash: 2a0751f12f33a36d9e0003977bcf40b66d715615
ms.sourcegitcommit: 25bb515efe62bfb8a8377293b56c3163f46122bf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87986957"
---
# <a name="access-external-storage-in-synapse-sql-on-demand"></a>Acceso al almacenamiento externo en Synapse SQL (a petición)

En este documento se describe la forma en que los usuarios pueden leer datos de los archivos almacenados en Azure Storage en Synapse SQL (a petición). Los usuarios tienen las siguientes opciones para acceder al almacenamiento:

- La función [OPENROWSET](develop-openrowset.md), que habilita las consultas ad hoc en los archivos de Azure Storage.
- Una [tabla externa](develop-tables-external-tables.md), que es una estructura de datos predefinida basada en un conjunto de archivos externos.

El usuario puede utilizar [diferentes métodos de autenticación](develop-storage-files-storage-access-control.md) como la autenticación de paso a través de Azure AD (el valor predeterminado para las entidades de seguridad de Azure AD) y la autenticación de SAS (el valor predeterminado para las entidades de seguridad de SQL).

## <a name="query-files-using-openrowset"></a>Consulta de archivos mediante OPENROWSET

OPENROWSET permite a los usuarios consultar archivos externos en Azure Storage si tienen acceso al almacenamiento. El usuario que está conectado al punto de conexión de Synapse SQL a petición debe usar la siguiente consulta para leer el contenido de los archivos en Azure Storage:

```sql
SELECT * FROM
 OPENROWSET(BULK 'https://<storage_account>.dfs.core.windows.net/<container>/<path>/*.parquet', format= 'parquet') as rows
```

El usuario puede acceder al almacenamiento mediante las siguientes reglas de acceso:

- Usuario de Azure AD: `OPENROWSET` utilizará la identidad de Azure AD del autor de la llamada para acceder a Azure Storage o para acceder al almacenamiento con acceso anónimo.
- Usuario de SQL: `OPENROWSET` accederá al almacenamiento con acceso anónimo o se puede suplantar mediante el token de SAS o la identidad administrada del área de trabajo.

### <a name="impersonation"></a>[Suplantación](#tab/impersonation)

Las entidades de seguridad de SQL también pueden usar OPENROWSET para consultar directamente los archivos protegidos con tokens de SAS o la identidad administrada del área de trabajo. Si un usuario de SQL ejecuta esta función, un usuario avanzado con el permiso `ALTER ANY CREDENTIAL` debe crear una credencial, cuyo ámbito sea el servidor, que coincida con la dirección URL de la función (mediante el nombre de almacenamiento y el contenedor) y el permiso REFERENCES concedido para esta credencial al autor de la llamada de la función OPENROWSET:

```sql
EXECUTE AS somepoweruser

CREATE CREDENTIAL [https://<storage_account>.dfs.core.windows.net/<container>]
 WITH IDENTITY = 'SHARED ACCESS SIGNATURE', SECRET = 'sas token';

GRANT REFERENCES CREDENTIAL::[https://<storage_account>.dfs.core.windows.net/<container>] TO sqluser
```

Si no hay ninguna CREDENCIAL en el nivel de servidor que coincida con la dirección URL, o bien el usuario de SQL no tiene permiso REFERENCES para esta credencial, se devolverá el error. Las entidades de seguridad de SQL no se pueden suplantar con una identidad de Azure AD.

### <a name="direct-access"></a>[Acceso directo](#tab/direct-access)

No se necesita ninguna configuración adicional para permitir que los usuarios de Azure AD tengan acceso a los archivos mediante sus identidades.
Cualquier usuario puede acceder al almacenamiento de Azure que permite el acceso anónimo (no se necesita configuración adicional).

---

> [!NOTE]
> Esta versión de OPENROWSET está diseñada para realizar una exploración de datos rápida y sencilla mediante la autenticación predeterminada. Para sacar provecho de la suplantación o de la identidad administrada, use OPENROWSET con DATASOURCE como se describe en la siguiente sección.

## <a name="query-data-sources-using-openrowset"></a>Consulta de orígenes de datos mediante OPENROWSET

OPENROWSET permite al usuario consultar los archivos colocados en algunos orígenes de datos externos:

```sql
SELECT * FROM
 OPENROWSET(BULK 'file/path/*.parquet',
 DATASOURCE = MyAzureInvoices,
 FORMAT= 'parquet') as rows
```

El usuario que ejecuta esta consulta tiene que tener acceso a los archivos. Los usuarios se tienen que suplantar mediante el [token de SAS](develop-storage-files-storage-access-control.md?tabs=shared-access-signature) o la [ identidad administrada del área de trabajo](develop-storage-files-storage-access-control.md?tabs=managed-identity) si no pueden acceder directamente a los archivos mediante su [identidad de Azure AD](develop-storage-files-storage-access-control.md?tabs=user-identity) o [acceso anónimo](develop-storage-files-storage-access-control.md?tabs=public-access).

### <a name="impersonation"></a>[Suplantación](#tab/impersonation)

`DATABASE SCOPED CREDENTIAL` especifica cómo acceder a los archivos en el origen de datos al que se hace referencia (actualmente SAS e identidad administrada). El usuario avanzado con el permiso `CONTROL DATABASE` tendría que crear `DATABASE SCOPED CREDENTIAL` que se usará para acceder al almacenamiento y `EXTERNAL DATA SOURCE` que especifica la dirección URL del origen de datos y la credencial que se debe usar:

```sql
EXECUTE AS somepoweruser;

-- Create MASTER KEY if it doesn't exists in database
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'some very strong password';

CREATE DATABASE SCOPED CREDENTIAL AccessAzureInvoices
 WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
 SECRET = '******srt=sco&amp;sp=rwac&amp;se=2017-02-01T00:55:34Z&amp;st=201********' ;

CREATE EXTERNAL DATA SOURCE MyAzureInvoices
 WITH ( LOCATION = 'https://<storage_account>.dfs.core.windows.net/<container>/<path>/' ,
 CREDENTIAL = AccessAzureInvoices) ;
```

El autor de la llamada debe tener uno de los siguientes permisos para ejecutar la función OPENROWSET:

- Uno de los permisos para ejecutar OPENROWSET:
  - `ADMINISTER BULK OPERATIONS` permite iniciar sesión para ejecutar la función OPENROWSET.
  - `ADMINISTER DATABASE BULK OPERATIONS` permite a los usuarios cuyo ámbito sea la base de datos ejecutar la función OPENROWSET.
- `REFERENCES DATABASE SCOPED CREDENTIAL` a la credencial a la que se hace referencia en `EXTERNAL DATA SOURCE`.

### <a name="direct-access"></a>[Acceso directo](#tab/direct-access)

El usuario puede crear EXTERNAL DATA SOURCE sin CREDENCIAL que haga referencia al almacenamiento de acceso público, o bien usar la autenticación de paso a través de Azure AD:

```sql
CREATE EXTERNAL DATA SOURCE MyAzureInvoices
 WITH ( LOCATION = 'https://<storage_account>.dfs.core.windows.net/<container>/<path>') ;
```
---
## <a name="external-table"></a>EXTERNAL TABLE

El usuario con los permisos para leer la tabla puede acceder a los archivos externos mediante una EXTERNAL TABLE que se crea sobre el conjunto de carpetas y archivos de Azure Storage.

El usuario que tenga [permisos para crear una tabla externa](https://docs.microsoft.com/sql/t-sql/statements/create-external-table-transact-sql?view=sql-server-ver15#permissions) (por ejemplo CREATE TABLE y ALTER ANY CREDENTIAL o REFERENCES DATABASE SCOPED CREDENTIAL) puede usar el siguiente script para crear una tabla sobre el origen de datos de Azure Storage:

```sql
CREATE EXTERNAL TABLE [dbo].[DimProductexternal]
( ProductKey int, ProductLabel nvarchar, ProductName nvarchar )
WITH
(
LOCATION='/DimProduct/year=*/month=*' ,
DATA_SOURCE = AzureDataLakeStore ,
FILE_FORMAT = TextFileFormat
) ;
```

El usuario que lee los datos de esta tabla tienen que poder acceder a los archivos. Los usuarios se tienen que suplantar mediante el [token de SAS](develop-storage-files-storage-access-control.md?tabs=shared-access-signature) o la [ identidad administrada del área de trabajo](develop-storage-files-storage-access-control.md?tabs=managed-identity) si no pueden acceder directamente a los archivos mediante su [identidad de Azure AD](develop-storage-files-storage-access-control.md?tabs=user-identity) o [acceso anónimo](develop-storage-files-storage-access-control.md?tabs=public-access).

### <a name="impersonation"></a>[Suplantación](#tab/impersonation)

DATABASE SCOPED CREDENTIAL especifica cómo acceder a los archivos en el origen de datos al que se hace referencia. Los usuarios con el permiso CONTROL DATABASE necesitarán crear DATABASE SCOPED CREDENTIAL, que se usará para acceder al almacenamiento, y EXTERNAL DATA SOURCE, que especifica la dirección URL del origen de datos y la credencial que se debe usar:

```sql
EXECUTE AS somepoweruser;

-- Create MASTER KEY if it doesn't exists in database
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'some very strong password';

CREATE DATABASE SCOPED CREDENTIAL cred
 WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
 SECRET = '******srt=sco&sp=rwac&se=2017-02-01T00:55:34Z&st=201********' ;

CREATE EXTERNAL DATA SOURCE AzureDataLakeStore
 WITH ( LOCATION = 'https://<storage_account>.dfs.core.windows.net/<container>/<path>' ,
 CREDENTIAL = cred
 ) ;
```

### <a name="direct-access"></a>[Acceso directo](#tab/direct-access)

El usuario puede crear EXTERNAL DATA SOURCE sin CREDENCIAL que haga referencia al almacenamiento de acceso público, o bien usar la autenticación de paso a través de Azure AD:

```sql
CREATE EXTERNAL DATA SOURCE MyAzureInvoices
 WITH ( LOCATION = 'https://<storage_account>.dfs.core.windows.net/<container>/<path>') ;
```
---

### <a name="read-external-files-with-external-table"></a>Lectura de archivos externos con EXTERNAL TABLE

EXTERNAL TABLE permite leer datos de los archivos a los que se hace referencia a través de un origen de datos mediante la instrucción SELECT de SQL estándar:

```sql
SELECT *
FROM dbo.DimProductsExternal
```

El autor de la llamada debe tener los siguientes permisos para leer datos:
- Permiso `SELECT` SOBRE la tabla externa
- Permiso `REFERENCES DATABASE SCOPED CREDENTIAL` si `DATA SOURCE` tiene `CREDENTIAL`

## <a name="permissions"></a>Permisos

En la tabla siguiente se enumeran los permisos necesarios para las operaciones enumeradas anteriormente.

| Consultar | Permisos necesarios|
| --- | --- |
| OPENROWSET(BULK) sin origen de datos | `ADMINISTER BULK OPERATIONS`, `ADMINISTER DATABASE BULK OPERATIONS` o el inicio de sesión de SQL deben tener REFERENCES CREDENTIAL::\<URL> para el almacenamiento protegido con SAS |
| OPENROWSET(BULK) con origen de datos sin credencial | `ADMINISTER BULK OPERATIONS` o `ADMINISTER DATABASE BULK OPERATIONS` |
| OPENROWSET(BULK) con origen de datos con credencial | `REFERENCES DATABASE SCOPED CREDENTIAL` y `ADMINISTER BULK OPERATIONS` o `ADMINISTER DATABASE BULK OPERATIONS` |
| CREATE EXTERNAL DATA SOURCE | `ALTER ANY EXTERNAL DATA SOURCE` y `REFERENCES DATABASE SCOPED CREDENTIAL` |
| CREATE EXTERNAL TABLE | `CREATE TABLE`, `ALTER ANY SCHEMA`, `ALTER ANY EXTERNAL FILE FORMAT` y `ALTER ANY EXTERNAL DATA SOURCE`. |
| SELECT FROM EXTERNAL TABLE | `SELECT TABLE` y `REFERENCES DATABASE SCOPED CREDENTIAL` |
| CETAS | Para crear una tabla: `CREATE TABLE`, `ALTER ANY SCHEMA`, `ALTER ANY DATA SOURCE` y `ALTER ANY EXTERNAL FILE FORMAT`. Para leer datos: `ADMINISTER BULK OPERATIONS` o `REFERENCES CREDENTIAL` o `SELECT TABLE` por cada tabla, vista o función en el permiso de consulta + lectura/escritura en el almacenamiento |

## <a name="next-steps"></a>Pasos siguientes

Ya está preparado para continuar con los siguientes artículos de procedimientos:

- [Consulta de datos en almacenamiento](query-data-storage.md)

- [Consulta de archivo CSV](query-single-csv-file.md)

- [Consulta de archivos Parquet](query-parquet-files.md)

- [Consulta de archivos JSON](query-json-files.md)

- [Consulta de carpetas y varios archivos](query-folders-multiple-csv-files.md)

- [Uso de las funciones de partición y metadatos](query-specific-files.md)

- [Consulta de tipos anidados](query-parquet-nested-types.md)

- [Creación y uso de vistas en SQL a petición (versión preliminar) en Azure Synapse Analytics](create-use-views.md)
