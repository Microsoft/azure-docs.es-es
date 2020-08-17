---
title: Copia de datos hacia y desde Instancia administrada de Azure SQL
description: Obtenga información sobre cómo mover datos hacia y desde Instancia administrada de Azure SQL mediante Azure Data Factory.
services: data-factory
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.custom: seo-lt-2019
ms.date: 08/05/2020
ms.openlocfilehash: 5244d9711886376fe9502e31c227d8d74de4c21a
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87800388"
---
# <a name="copy-data-to-and-from-azure-sql-managed-instance-by-using-azure-data-factory"></a>Copia de datos hacia y desde Instancia administrada de Azure SQL mediante Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se describe el uso de la actividad de copia de Azure Data Factory para copiar datos hacia y desde Instancia administrada de Azure SQL. El documento se basa en el artículo de [introducción a la actividad de copia](copy-activity-overview.md) en el que se describe información general de esta actividad.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector de Instancia administrada de SQL es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)
- [Actividad GetMetadata](control-flow-get-metadata-activity.md)

Puede copiar datos desde Instancia administrada de SQL a cualquier almacén de datos receptor compatible. También puede copiar datos desde cualquier almacén de datos de origen compatible a Instancia administrada de SQL. Consulte la tabla de [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de almacenes de datos que la actividad de copia admite como orígenes y receptores.

En concreto, este conector de Instancia administrada de SQL admite lo siguiente:

- Copia de datos mediante la autenticación con SQL y la autenticación de tokens de aplicaciones de Azure Active Directory (Azure AD) con una entidad de servicio o identidades administradas para los recursos de Azure.
- Como origen, la recuperación de datos mediante una consulta SQL o un procedimiento almacenado.
- Como receptor, la creación automática de la tabla de destino si no existe en función del esquema de origen; la anexión de datos a una tabla o la invocación de un procedimiento almacenado con lógica personalizada durante la copia.

>[!NOTE]
> Actualmente, este conector no admite [Always Encrypted](https://docs.microsoft.com/sql/relational-databases/security/encryption/always-encrypted-database-engine?view=azuresqldb-mi-current) de Instancia administrada de SQL. Como solución alternativa, puede usar un [conector ODBC genérico](connector-odbc.md) y un controlador ODBC de SQL Server mediante un entorno de ejecución de integración autohospedado. Obtenga más información en la sección [Uso de Always Encrypted](#using-always-encrypted). 

## <a name="prerequisites"></a>Requisitos previos

Para acceder al [punto de conexión público](../azure-sql/managed-instance/public-endpoint-overview.md) de Instancia administrada de SQL, puede usar un entorno de ejecución de integración de Azure administrado de Azure Data Factory. Asegúrese de habilitar el punto de conexión público y de permitir también el tráfico de punto de conexión público en el grupo de seguridad de red para que Azure Data Factory pueda conectarse a la base de datos. Para obtener más información, consulte [esta guía](../azure-sql/managed-instance/public-endpoint-configure.md).

Para acceder al punto de conexión privado de Instancia administrada de SQL, configure un [entorno de ejecución de integración autohospedado](create-self-hosted-integration-runtime.md) que pueda acceder a la base de datos. Si aprovisiona el entorno de ejecución de integración autohospedado en la misma red virtual que la instancia administrada, asegúrese de que la máquina del entorno de ejecución de integración está en una subred distinta a la de la instancia administrada. Si aprovisiona el entorno de ejecución de integración autohospedado en una red virtual distinta a la de la instancia administrada, puede usar un emparejamiento de redes virtuales o una conexión de red virtual a red virtual. Para obtener más información, vea [Conexión de una aplicación a Instancia administrada de SQL](../azure-sql/managed-instance/connect-application-instance.md).

## <a name="get-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles sobre las propiedades que se usan para definir entidades de Azure Data Factory específicas del conector de Instancia administrada de SQL.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades se admiten en el servicio vinculado de Instancia administrada de SQL:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en **AzureSqlMI**. | Sí |
| connectionString |Esta propiedad especifica la información de **connectionString** necesaria para conectarse a Instancia administrada de SQL mediante la autenticación de SQL. Para más información, vea los ejemplos siguientes. <br/>El puerto predeterminado es el 1433. Si usa Instancia administrada de SQL con un punto de conexión público, especifique explícitamente el puerto 3342.<br> También puede asignar una contraseña en Azure Key Vault. Si se trata de la autenticación de SQL, extraiga la configuración `password` de la cadena de conexión. Para obtener más información, vea el ejemplo de JSON debajo de la tabla y consulte el artículo [Almacenamiento de credenciales en Azure Key Vault](store-credentials-in-key-vault.md). |Sí |
| servicePrincipalId | Especifique el id. de cliente de la aplicación. | Sí, al utilizar la autenticación de Azure AD con una entidad de servicio |
| servicePrincipalKey | Especifique la clave de la aplicación. Marque este campo como [SecureString](store-credentials-in-key-vault.md) para almacenarlo de forma segura en Azure Data Factory, o bien **para hacer referencia a un secreto almacenado en Azure Key Vault**. | Sí, al utilizar la autenticación de Azure AD con una entidad de servicio |
| tenant | Especifique la información del inquilino, como el nombre de dominio o identificador de inquilino, en el que reside la aplicación. Para recuperarlo, mantenga el puntero del mouse en la esquina superior derecha de Azure Portal. | Sí, al utilizar la autenticación de Azure AD con una entidad de servicio |
| connectVia | Este [entorno de ejecución de integración](concepts-integration-runtime.md) se usa para conectarse al almacén de datos. Puede usar un entorno de ejecución de integración autohospedado o un entorno de ejecución de integración de Azure si la instancia administrada tiene un punto de conexión público y permite que Azure Data Factory acceda a él. Si no se especifica, se usa el valor predeterminado de Azure Integration Runtime. |Sí |

Para ver los distintos tipos de autenticación, consulte las secciones siguientes acerca de requisitos previos y ejemplos de JSON, respectivamente:

- [Autenticación de SQL](#sql-authentication)
- [Autenticación de token de la aplicación de Azure AD: entidad de servicio](#service-principal-authentication)
- [Autenticación de token de la aplicación de Azure AD: identidades administradas de recursos de Azure](#managed-identity)

### <a name="sql-authentication"></a>Autenticación SQL

**Ejemplo 1: uso de la autenticación de SQL**

```json
{
    "name": "AzureSqlMILinkedService",
    "properties": {
        "type": "AzureSqlMI",
        "typeProperties": {
            "connectionString": "Data Source=<hostname,port>;Initial Catalog=<databasename>;Integrated Security=False;User ID=<username>;Password=<password>;"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Ejemplo 2: uso de la autenticación de SQL con una contraseña en Azure Key Vault**

```json
{
    "name": "AzureSqlMILinkedService",
    "properties": {
        "type": "AzureSqlMI",
        "typeProperties": {
            "connectionString": "Data Source=<hostname,port>;Initial Catalog=<databasename>;Integrated Security=False;User ID=<username>;",
            "password": { 
                "type": "AzureKeyVaultSecret", 
                "store": { 
                    "referenceName": "<Azure Key Vault linked service name>", 
                    "type": "LinkedServiceReference" 
                }, 
                "secretName": "<secretName>" 
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="service-principal-authentication"></a>Autenticación de entidad de servicio

Para usar la autenticación de token de aplicación de Azure AD basada en la entidad de servicio, realice estos pasos:

1. Siga los pasos que aparecen en [Aprovisionamiento de un administrador de Azure Active Directory para su instancia administrada](../azure-sql/database/authentication-aad-configure.md#provision-azure-ad-admin-sql-managed-instance).

2. [Cree una aplicación de Azure Active Directory](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal) desde Azure Portal. Anote el nombre de la aplicación y los siguientes valores, que definen el servicio vinculado:

    - Identificador de aplicación
    - Clave de la aplicación
    - Id. de inquilino

3. [Cree inicios de sesión](https://docs.microsoft.com/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current) para la identidad administrada de Azure Data Factory. En SQL Server Management Studio (SSMS), conéctese a la instancia administrada con una cuenta de SQL Server que sea **sysadmin**. En la base de datos **maestra**, ejecute el siguiente script T-SQL:

    ```sql
    CREATE LOGIN [your application name] FROM EXTERNAL PROVIDER
    ```

4. [Cree usuarios de bases de datos independientes](../azure-sql/database/authentication-aad-configure.md#create-contained-users-mapped-to-azure-ad-identities) para la identidad administrada de Azure Data Factory. Conéctese a la base de datos de origen o destino de copia de los datos y ejecute el siguiente script T-SQL: 
  
    ```sql
    CREATE USER [your application name] FROM EXTERNAL PROVIDER
    ```

5. Conceda a la identidad administrada de Data Factory los permisos necesarios, tal como lo haría normalmente para los usuarios de SQL y otros usuarios. Ejecute el código siguiente: Para más opciones, consulte [este documento](https://docs.microsoft.com/sql/t-sql/statements/alter-role-transact-sql?view=azuresqldb-mi-current).

    ```sql
    ALTER ROLE [role name e.g. db_owner] ADD MEMBER [your application name]
    ```

6. En Azure Data Factory, configure un servicio vinculado de Instancia administrada de SQL.

**Ejemplo: uso de la autenticación de la entidad de servicio**

```json
{
    "name": "AzureSqlDbLinkedService",
    "properties": {
        "type": "AzureSqlMI",
        "typeProperties": {
            "connectionString": "Data Source=<hostname,port>;Initial Catalog=<databasename>;",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "<service principal key>"
            },
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="managed-identities-for-azure-resources-authentication"></a><a name="managed-identity"></a> Identidades administradas para la autenticación de recursos de Azure

Una factoría de datos se puede asociar con una [identidad administrada para recursos de Azure](data-factory-service-identity.md) que representa esa factoría de datos concreta. Esta identidad administrada se puede usar para la autenticación de Instancia administrada de SQL. Puede acceder a la factoría designada y copiar datos desde la base de datos o en la base de datos usando esta identidad.

Para usar la autenticación de identidad administrada, siga estos pasos.

1. Siga los pasos que aparecen en [Aprovisionamiento de un administrador de Azure Active Directory para su instancia administrada](../azure-sql/database/authentication-aad-configure.md#provision-azure-ad-admin-sql-managed-instance).

2. [Cree inicios de sesión](https://docs.microsoft.com/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current) para la identidad administrada de Azure Data Factory. En SQL Server Management Studio (SSMS), conéctese a la instancia administrada con una cuenta de SQL Server que sea **sysadmin**. En la base de datos **maestra**, ejecute el siguiente script T-SQL:

    ```sql
    CREATE LOGIN [your Data Factory name] FROM EXTERNAL PROVIDER
    ```

3. [Cree usuarios de bases de datos independientes](../azure-sql/database/authentication-aad-configure.md#create-contained-users-mapped-to-azure-ad-identities) para la identidad administrada de Azure Data Factory. Conéctese a la base de datos de origen o destino de copia de los datos y ejecute el siguiente script T-SQL: 
  
    ```sql
    CREATE USER [your Data Factory name] FROM EXTERNAL PROVIDER
    ```

4. Conceda a la identidad administrada de Data Factory los permisos necesarios, tal como lo haría normalmente para los usuarios de SQL y otros usuarios. Ejecute el código siguiente: Para más opciones, consulte [este documento](https://docs.microsoft.com/sql/t-sql/statements/alter-role-transact-sql?view=azuresqldb-mi-current).

    ```sql
    ALTER ROLE [role name e.g. db_owner] ADD MEMBER [your Data Factory name]
    ```

5. En Azure Data Factory, configure un servicio vinculado de Instancia administrada de SQL.

**Ejemplo: uso de autenticación de identidad administrada**

```json
{
    "name": "AzureSqlDbLinkedService",
    "properties": {
        "type": "AzureSqlMI",
        "typeProperties": {
            "connectionString": "Data Source=<hostname,port>;Initial Catalog=<databasename>;"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea obtener una lista completa de secciones y propiedades disponibles para definir conjuntos de datos, vea el artículo sobre conjuntos de datos. En esta sección se proporciona una lista de las propiedades que admite el conjunto de datos de Instancia administrada de SQL.

Para copiar datos en Instancia administrada de SQL y desde esta, se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del conjunto de datos debe establecerse en **AzureSqlMITable**. | Sí |
| esquema | Nombre del esquema. |No para el origen, sí para el receptor  |
| table | Nombre de la tabla o vista. |No para el origen, sí para el receptor  |
| tableName | Nombre de la tabla o vista con el esquema. Esta propiedad permite la compatibilidad con versiones anteriores. Para la nueva carga de trabajo use `schema` y `table`. | No para el origen, sí para el receptor |

**Ejemplo**

```json
{
    "name": "AzureSqlMIDataset",
    "properties":
    {
        "type": "AzureSqlMITable",
        "linkedServiceName": {
            "referenceName": "<SQL Managed Instance linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, retrievable during authoring > ],
        "typeProperties": {
            "schema": "<schema_name>",
            "table": "<table_name>"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, vea el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades admitidas en el origen y el receptor de Instancia administrada de SQL.

### <a name="sql-managed-instance-as-a-source"></a>Instancia administrada de SQL como origen

Para copiar datos desde Instancia administrada de SQL, se admiten las siguientes propiedades en la sección de origen de la actividad de copia:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del origen de la actividad de copia debe establecerse en **SqlMISource**. | Sí |
| sqlReaderQuery |Esta propiedad usa la consulta SQL personalizada para leer los datos. Un ejemplo es `select * from MyTable`. |No |
| sqlReaderStoredProcedureName |Esta propiedad es el nombre del procedimiento almacenado que lee datos de la tabla de origen. La última instrucción SQL debe ser una instrucción SELECT del procedimiento almacenado. |No |
| storedProcedureParameters |Estos parámetros son para el procedimiento almacenado.<br/>Los valores permitidos son pares de nombre o valor. Los nombres y las mayúsculas y minúsculas de los parámetros deben coincidir con las mismas características de los parámetros de procedimiento almacenado. |No |
| isolationLevel | Especifica el comportamiento de bloqueo de transacción para el origen de SQL. Los valores permitidos son: **ReadCommitted**, **ReadUncommitted**, **RepeatableRead**, **Serializable** y **Snapshot**. Si no se especifica, se utiliza el nivel de aislamiento predeterminado de la base de datos. Vea [este documento](https://docs.microsoft.com/dotnet/api/system.data.isolationlevel) para obtener más detalles. | No |

**Tenga en cuenta los siguientes puntos:**

- Si se especifica **sqlReaderQuery** para **SqlMISource**, la actividad de copia ejecuta esta consulta en el origen de Instancia administrada de SQL para obtener los datos. También puede indicar un procedimiento almacenado mediante la definición de **sqlReaderStoredProcedureName** y **storedProcedureParameters** si el procedimiento almacenado adopta parámetros.
- Si no se especifican las propiedades **sqlReaderQuery** o **sqlReaderStoredProcedureName**, se usan las columnas que se definen en la sección “structure” del JSON del conjunto de datos para construir una consulta. La consulta `select column1, column2 from mytable` se ejecuta en Instancia administrada de SQL. Si la definición del conjunto de datos no tiene la sección "structure", se seleccionan todas las columnas de la tabla.

**Ejemplo: Uso de una consulta SQL**

```json
"activities":[
    {
        "name": "CopyFromAzureSqlMI",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SQL Managed Instance input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "SqlMISource",
                "sqlReaderQuery": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

**Ejemplo: Uso de un procedimiento almacenado**

```json
"activities":[
    {
        "name": "CopyFromAzureSqlMI",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SQL Managed Instance input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "SqlMISource",
                "sqlReaderStoredProcedureName": "CopyTestSrcStoredProcedureWithParameters",
                "storedProcedureParameters": {
                    "stringData": { "value": "str3" },
                    "identifier": { "value": "$$Text.Format('{0:yyyy}', <datetime parameter>)", "type": "Int"}
                }
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

**Definición del procedimiento almacenado**

```sql
CREATE PROCEDURE CopyTestSrcStoredProcedureWithParameters
(
    @stringData varchar(20),
    @identifier int
)
AS
SET NOCOUNT ON;
BEGIN
    select *
    from dbo.UnitTestSrcTable
    where dbo.UnitTestSrcTable.stringData != stringData
    and dbo.UnitTestSrcTable.identifier != identifier
END
GO
```

### <a name="sql-managed-instance-as-a-sink"></a>Instancia administrada de SQL como receptor

> [!TIP]
> Obtenga más información sobre los comportamientos de escritura, las configuraciones y los procedimientos recomendados que se admiten en [Procedimiento recomendado para cargar datos en Instancia administrada de SQL](#best-practice-for-loading-data-into-sql-managed-instance).

Para copiar datos a Instancia administrada de SQL, se admiten las siguientes propiedades en la sección de receptor de la actividad de copia:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del receptor de la actividad de copia debe establecerse en **SqlMISink**. | Sí |
| preCopyScript |Esta propiedad especifica una consulta SQL para que la actividad de copia se ejecute antes de escribir datos en Instancia administrada de SQL. Solo se invoca una vez por cada copia que se ejecuta. Puede usar esta propiedad para limpiar los datos cargados previamente. |No |
| tableOption | Especifica si [se crea automáticamente la tabla de receptores](copy-activity-overview.md#auto-create-sink-tables) según el esquema de origen, si no existe. No se admite la creación automática de tablas cuando el receptor especifica un procedimiento almacenado. Los valores permitidos son: `none` (valor predeterminado), `autoCreate`. |No |
| sqlWriterStoredProcedureName | El nombre del procedimiento almacenado que define cómo se aplican los datos de origen en una tabla de destino. <br/>Este procedimiento almacenado se *invoca por lote*. Para las operaciones que solo se ejecuta una vez y que no tiene nada que ver con los datos de origen, como por ejemplo, eliminar o truncar, use la propiedad `preCopyScript`.<br>Vea el ejemplo de [Invocación del procedimiento almacenado desde el receptor de SQL](#invoke-a-stored-procedure-from-a-sql-sink). | No |
| storedProcedureTableTypeParameterName |Nombre del parámetro del tipo de tabla especificado en el procedimiento almacenado.  |No |
| sqlWriterTableType |Nombre del tipo de tabla que se usará en el procedimiento almacenado. La actividad de copia dispone que los datos que se mueven estén disponibles en una tabla temporal con este tipo de tabla. El código de procedimiento almacenado puede combinar los datos copiados con datos existentes. |No |
| storedProcedureParameters |Parámetros del procedimiento almacenado.<br/>Los valores permitidos son pares de nombre y valor. Los nombres y las mayúsculas y minúsculas de los parámetros deben coincidir con las mismas características de los parámetros de procedimiento almacenado. | No |
| writeBatchSize |Número de filas que se va a insertar en la tabla SQL *por lote*.<br/>Los valores permitidos son enteros para el número de filas. De forma predeterminada, Azure Data Factory determina dinámicamente el tamaño adecuado del lote en función del tamaño de fila.  |No |
| writeBatchTimeout |Esta propiedad especifica el tiempo de espera para que la operación de inserción por lotes se complete antes de que se agote el tiempo de espera.<br/>Los valores permitidos son para el intervalo de tiempo. Un ejemplo es "00:30:00", que es 30 minutos. |No |

**Ejemplo 1: Anexión de datos**

```json
"activities":[
    {
        "name": "CopyToAzureSqlMI",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<SQL Managed Instance output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "SqlMISink",
                "tableOption": "autoCreate",
                "writeBatchSize": 100000
            }
        }
    }
]
```

**Ejemplo 2: Invocación de un procedimiento almacenado durante la copia**

Para más información, vea [Invocación del procedimiento almacenado desde el receptor MI de SQL ](#invoke-a-stored-procedure-from-a-sql-sink).

```json
"activities":[
    {
        "name": "CopyToAzureSqlMI",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<SQL Managed Instance output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "SqlMISink",
                "sqlWriterStoredProcedureName": "CopyTestStoredProcedureWithParameters",
                "storedProcedureTableTypeParameterName": "MyTable",
                "sqlWriterTableType": "MyTableType",
                "storedProcedureParameters": {
                    "identifier": { "value": "1", "type": "Int" },
                    "stringData": { "value": "str1" }
                }
            }
        }
    }
]
```

## <a name="best-practice-for-loading-data-into-sql-managed-instance"></a>Procedimiento recomendado para cargar datos en Instancia administrada de SQL

Al copiar datos en Instancia administrada de SQL, puede requerir un comportamiento de escritura diferente:

- [Anexión](#append-data): mis datos de origen solo tienen registros nuevos.
- [Actualización e inserción](#upsert-data): mis datos de origen tienen inserciones y actualizaciones.
- [Sobrescritura](#overwrite-the-entire-table): quiero volver a cargar toda la tabla de dimensiones cada vez.
- [Escritura con lógica personalizada](#write-data-with-custom-logic): necesito un procesamiento adicional antes de la inserción final en la tabla de destino. 

Consulte las secciones correspondientes sobre cómo configurar estas operaciones en Azure Data Factory y los procedimientos recomendados.

### <a name="append-data"></a>Anexión de datos

La anexión de datos es el comportamiento predeterminado de este conector de receptor de Instancia administrada de SQL. Azure Data Factory realiza una inserción masiva para escribir en la tabla de forma eficaz. Puede configurar el origen y el receptor según corresponda en la actividad de copia.

### <a name="upsert-data"></a>Actualización e inserción de datos

**Opción 1:** si tiene una gran cantidad de datos para copiar, puede cargar de forma masiva todos los registros en una tabla de almacenamiento provisional mediante la actividad de copia y luego ejecutar una actividad de procedimiento almacenado para aplicar una instrucción [MERGE](https://docs.microsoft.com/sql/t-sql/statements/merge-transact-sql?view=azuresqldb-mi-current) o INSERT/UPDATE de una sola vez. 

La actividad de copia actualmente no admite de forma nativa la carga de datos en una tabla temporal de base de datos. Hay una forma avanzada de configurarla con una combinación de varias actividades. Vea [Optimización de escenarios de upsert masivo de SQL Database](https://github.com/scoriani/azuresqlbulkupsert). A continuación se muestra un ejemplo de uso de una tabla permanente como almacenamiento provisional.

Por ejemplo, en Azure Data Factory, puede crear una canalización con una **actividad de copia** encadenada con una **actividad de procedimiento almacenado**. La primera copia datos del almacén de origen en una tabla de almacenamiento provisional de Instancia administrada de Azure SQL, por ejemplo, **UpsertStagingTable**, como nombre de la tabla del conjunto de datos. Luego, la segunda invoca un procedimiento almacenado para combinar datos de origen de la tabla de almacenamiento provisional en la tabla de destino y limpiar la tabla de almacenamiento provisional.

![Upsert](./media/connector-azure-sql-database/azure-sql-database-upsert.png)

En la base de datos, defina un procedimiento almacenado con la lógica MERGE, como en el ejemplo siguiente, al que se señala desde la actividad de procedimiento almacenado anterior. Suponga que el destino es la tabla **Marketing** con tres columnas: **ProfileID**, **State** y **Category**. Realice la operación upsert de acuerdo con la columna **ProfileID**.

```sql
CREATE PROCEDURE [dbo].[spMergeData]
AS
BEGIN
    MERGE TargetTable AS target
    USING UpsertStagingTable AS source
    ON (target.[ProfileID] = source.[ProfileID])
    WHEN MATCHED THEN
        UPDATE SET State = source.State
    WHEN NOT matched THEN
        INSERT ([ProfileID], [State], [Category])
      VALUES (source.ProfileID, source.State, source.Category);
    
    TRUNCATE TABLE UpsertStagingTable
END
```

**Opción 2:** puede optar por [invocar un procedimiento almacenado en la actividad de copia](#invoke-a-stored-procedure-from-a-sql-sink). Este enfoque ejecuta cada lote (según indica la propiedad `writeBatchSize`) en la tabla de origen en lugar de usar la inserción masiva como enfoque predeterminado en la actividad de copia.

### <a name="overwrite-the-entire-table"></a>Sobrescritura de toda la tabla

Puede configurar la propiedad **preCopyScript** en un receptor de la actividad de copia. En este caso, Azure Data Factory ejecuta primero el script para cada actividad de copia que ejecute. Después, ejecuta la copia para insertar los datos. Por ejemplo, para sobrescribir toda la tabla con los datos más recientes, especifique un script para eliminar primero todos los registros antes de realizar la carga masiva de los nuevos datos desde el origen.

### <a name="write-data-with-custom-logic"></a>Escritura de datos con lógica personalizada

Los pasos necesarios para escribir datos con lógica personalizada son similares a los que se describen en la sección [Actualización e inserción de datos](#upsert-data). Si necesita aplicar procesamiento adicional antes de la inserción final de los datos de origen en la tabla de destino, puede cargar en una tabla de almacenamiento provisional y, luego, invocar una actividad de procedimiento almacenado, o bien invocar un procedimiento almacenado en un receptor de actividad de copia para aplicar datos.

## <a name="invoke-a-stored-procedure-from-a-sql-sink"></a><a name="invoke-a-stored-procedure-from-a-sql-sink"></a> Invocación del procedimiento almacenado desde el receptor de SQL

Al copiar datos en Instancia administrada de SQL, también se puede configurar e invocar un procedimiento almacenado especificado por el usuario con parámetros adicionales en cada lote de la tabla de origen. La característica de procedimiento almacenado aprovecha los [parámetros con valores de tabla](https://msdn.microsoft.com/library/bb675163.aspx).

Cuando los mecanismos de copia integrados no prestan el servicio, se puede usar un procedimiento almacenado. Por ejemplo, si quiere aplicar un procesamiento adicional antes de la inserción final de los datos de origen en la tabla de destino. Otros ejemplos de procesamiento adicional son cuando quiere combinar columnas, buscar valores adicionales e insertar datos en más de una tabla.

En el ejemplo siguiente se muestra cómo usar un procedimiento almacenado para realizar una operación UPSERT en una tabla en la base de datos SQL Server. Supongamos que los datos de entrada y la tabla **Marketing** del receptor tienen tres columnas: **ProfileID**, **State** y **Category**. Realice una operación UPSERT en función de la columna **ProfileID** y aplíquela solo a una categoría específica llamada "ProductA".

1. En la base de datos, defina el tipo de tabla con el mismo nombre que **sqlWriterTableType**. El esquema del tipo de tabla es el mismo que el que devuelven los datos de entrada.

    ```sql
    CREATE TYPE [dbo].[MarketingType] AS TABLE(
        [ProfileID] [varchar](256) NOT NULL,
        [State] [varchar](256) NOT NULL,
        [Category] [varchar](256) NOT NULL
    )
    ```

2. En la base de datos, defina el procedimiento almacenado con el mismo nombre que **sqlWriterStoredProcedureName**. Dicho procedimiento administra los datos de entrada del origen especificado y los combina en la tabla de salida. El nombre del parámetro del tipo de tabla del procedimiento almacenado es el mismo que el de **tableName** que se ha definido en el conjunto de datos.

    ```sql
    CREATE PROCEDURE spOverwriteMarketing @Marketing [dbo].[MarketingType] READONLY, @category varchar(256)
    AS
    BEGIN
    MERGE [dbo].[Marketing] AS target
    USING @Marketing AS source
    ON (target.ProfileID = source.ProfileID and target.Category = @category)
    WHEN MATCHED THEN
        UPDATE SET State = source.State
    WHEN NOT MATCHED THEN
        INSERT (ProfileID, State, Category)
        VALUES (source.ProfileID, source.State, source.Category);
    END
    ```

3. En Azure Data Factory, defina la sección del **receptor MI de SQL** en la actividad de copia como se indica a continuación:

    ```json
    "sink": {
        "type": "SqlMISink",
        "sqlWriterStoredProcedureName": "spOverwriteMarketing",
        "storedProcedureTableTypeParameterName": "Marketing",
        "sqlWriterTableType": "MarketingType",
        "storedProcedureParameters": {
            "category": {
                "value": "ProductA"
            }
        }
    }
    ```

## <a name="data-type-mapping-for-sql-managed-instance"></a>Asignación de tipos de datos para Instancia administrada de SQL

Al copiar datos desde y hacia Instancia administrada de SQL, se usan las siguientes asignaciones de los tipos de datos de dicha instancia a los tipos de datos provisionales de Azure Data Factory. Consulte el artículo sobre [asignaciones de tipos de datos y esquema](copy-activity-schema-and-type-mapping.md) para más información sobre cómo la actividad de copia asigna el tipo de datos y el esquema de origen al receptor.

| Tipo de datos de Instancia administrada de SQL | Tipo de datos provisionales de Azure Data Factory |
|:--- |:--- |
| bigint |Int64 |
| binary |Byte[] |
| bit |Boolean |
| char |String, Char[] |
| date |DateTime |
| Datetime |DateTime |
| datetime2 |DateTime |
| Datetimeoffset |DateTimeOffset |
| Decimal |Decimal |
| FILESTREAM attribute (varbinary(max)) |Byte[] |
| Float |Double |
| imagen |Byte[] |
| int |Int32 |
| money |Decimal |
| NCHAR |String, Char[] |
| ntext |String, Char[] |
| NUMERIC |Decimal |
| NVARCHAR |String, Char[] |
| real |Single |
| rowversion |Byte[] |
| smalldatetime |DateTime |
| SMALLINT |Int16 |
| SMALLMONEY |Decimal |
| sql_variant |Object |
| text |String, Char[] |
| time |TimeSpan |
| timestamp |Byte[] |
| TINYINT |Int16 |
| UNIQUEIDENTIFIER |Guid |
| varbinary |Byte[] |
| varchar |String, Char[] |
| Xml |String |

>[!NOTE]
> En el caso de los tipos de datos que se asignan al tipo decimal provisional, la actividad de copia actualmente admite una precisión de hasta 28. Si tiene datos que requieren una precisión mayor que 28, considere la posibilidad de convertir a una cadena en una consulta SQL.

## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para obtener información detallada sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).

## <a name="getmetadata-activity-properties"></a>Propiedades de la actividad GetMetadata

Para información detallada sobre las propiedades, consulte [Actividad de obtención de metadatos](control-flow-get-metadata-activity.md). 

## <a name="using-always-encrypted"></a>Uso de Always Encrypted

Al copiar datos desde o hacia Instancia administrada de Azure SQL con [Always Encrypted](https://docs.microsoft.com/sql/relational-databases/security/encryption/always-encrypted-database-engine?view=azuresqldb-mi-current), use el [conector ODBC genérico](connector-odbc.md) y el controlador ODBC de SQL Server por medio de un entorno de ejecución de integración autohospedado. Este conector de Instancia administrada de Azure SQL de momento no es compatible con Always Encrypted. 

Más concretamente:

1. Configure un entorno de ejecución de integración autohospedado si no tiene uno. Consulte el artículo sobre [Integration Runtime autohospedado](create-self-hosted-integration-runtime.md) para más información.

2. Descargue el controlador ODBC de 64 bits para SQL Server [aquí](https://docs.microsoft.com/sql/connect/odbc/download-odbc-driver-for-sql-server?view=azuresqldb-mi-current) e instálelo en la máquina del entorno de ejecución de integración. Puede encontrar más información sobre el funcionamiento de este controlador en [Uso de Always Encrypted con el controlador ODBC para SQL Server](https://docs.microsoft.com/sql/connect/odbc/using-always-encrypted-with-the-odbc-driver?view=azuresqldb-mi-current#using-the-azure-key-vault-provider).

3. Cree un servicio vinculado con tipo ODBC para conectarse a la base de datos SQL; consulte los ejemplos siguientes:

    - Para usar la **autenticación de SQL**: especifique la cadena de conexión de ODBC como se muestra a continuación y seleccione autenticación **Básica** para establecer el nombre de usuario y la contraseña.

        ```
        Driver={ODBC Driver 17 for SQL Server};Server=<serverName>;Database=<databaseName>;ColumnEncryption=Enabled;KeyStoreAuthentication=KeyVaultClientSecret;KeyStorePrincipalId=<servicePrincipalKey>;KeyStoreSecret=<servicePrincipalKey>
        ```

    - Para usar la **autenticación de Identidad administrada de Data Factory**: 

        1. siga los mismos [requisitos previos](#managed-identity) para crear el usuario de base de datos de la identidad administrada y conceder el rol adecuado en la base de datos.
        2. En el servicio vinculado, especifique la cadena de conexión de ODBC como se muestra a continuación y seleccione autenticación **Anónima** como indica la propia cadena de conexión`Authentication=ActiveDirectoryMsi`.

        ```
        Driver={ODBC Driver 17 for SQL Server};Server=<serverName>;Database=<databaseName>;ColumnEncryption=Enabled;KeyStoreAuthentication=KeyVaultClientSecret;KeyStorePrincipalId=<servicePrincipalKey>;KeyStoreSecret=<servicePrincipalKey>; Authentication=ActiveDirectoryMsi;
        ```

4. Cree el conjunto de datos y la actividad de copia con el tipo ODBC según corresponda. Obtenga más información en el artículo [conector ODBC](connector-odbc.md).

## <a name="next-steps"></a>Pasos siguientes
Consulte los [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver la lista de almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores.
