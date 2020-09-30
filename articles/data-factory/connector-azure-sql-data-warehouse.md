---
title: Copia y transformación de datos en Azure Synapse Analytics
description: Descubra cómo copiar datos en Azure Synapse Analytics y cómo transformarlos en Azure Synapse Analytics mediante Data Factory.
services: data-factory
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 08/28/2020
ms.openlocfilehash: cd14a183ae1434af83c96b7f8d6575186412b534
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89051226"
---
# <a name="copy-and-transform-data-in-azure-synapse-analytics-formerly-azure-sql-data-warehouse-by-using-azure-data-factory"></a>Copia y transformación de datos en Azure Synapse Analytics (antes Azure SQL Data Warehouse) mediante Azure Data Factory

> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que esté usando:"]
>
> - [Versión 1](v1/data-factory-azure-sql-data-warehouse-connector.md)
> - [Versión actual](connector-azure-sql-data-warehouse.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo, se explica cómo usar la actividad de copia en Azure Data Factory para copiar datos desde y hacia Azure Synapse Analytics, y usar Data Flow para transformar datos en Azure Data Lake Storage Gen2. Para información sobre Azure Data Factory, lea el [artículo de introducción](introduction.md).

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector de Azure Synapse Analytics es compatible con estas actividades:

- [Actividad de copia](copy-activity-overview.md) con tabla de [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Asignación de Data Flow](concepts-data-flow-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)
- [Actividad GetMetadata](control-flow-get-metadata-activity.md)

Para la actividad de copia, este conector de Azure Synapse Analytics admite estas funciones:

- Copiar datos mediante la autenticación con SQL y la autenticación de tokens de aplicaciones de Azure Active Directory (Azure AD) con una entidad de servicio o identidades administradas para recursos de Azure.
- Como origen, la recuperación de datos mediante una consulta SQL o un procedimiento almacenado. También puede optar por la copia en paralelo desde un origen de Azure Synapse Analytics; vea la sección [Copia en paralelo desde Synapse Analytics](#parallel-copy-from-synapse-analytics) para obtener detalles.
- Como receptor, cargar de datos mediante [PolyBase](#use-polybase-to-load-data-into-azure-sql-data-warehouse), la [instrucción COPY](#use-copy-statement) (versión preliminar) o Inserción masiva. Se recomienda PolyBase o la instrucción COPY (versión preliminar) para mejorar el rendimiento de la copia. El conector también admite la creación automática de la tabla de destino, si no existe, en función del esquema de origen.

> [!IMPORTANT]
> Si copia los datos mediante Azure Data Factory Integration Runtime, configure una [regla de firewall de nivel de servidor](../azure-sql/database/firewall-configure.md) para que los servicios de Azure puedan acceder al [servidor lógico de SQL](../azure-sql/database/logical-servers.md).
> Si copia los datos mediante un entorno de ejecución de integración autohospedado, configure el firewall para permitir el intervalo IP apropiado. Dicho intervalo incluye la dirección IP del equipo que se usa para conectarse a Azure Synapse Analytics.

## <a name="get-started"></a>Introducción

> [!TIP]
> Para obtener el mejor rendimiento posible, use PolyBase para cargar datos en Azure Synapse Analytics. Encontrará más detalles en la sección [Usar PolyBase para cargar datos en SQL Data Warehouses](#use-polybase-to-load-data-into-azure-sql-data-warehouse). Para un tutorial con un caso de uso, vea [Carga de datos en Azure SQL Data Warehouse mediante Azure Data Factory](load-azure-sql-data-warehouse.md).

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles sobre las propiedades que definen las entidades de Data Factory específicas de un conector de Azure Synapse Analytics.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con un servicio vinculado de Azure Synapse Analytics:

| Propiedad            | Descripción                                                  | Obligatorio                                                     |
| :------------------ | :----------------------------------------------------------- | :----------------------------------------------------------- |
| type                | La propiedad type debe establecerse en **AzureSqlDW**.             | Sí                                                          |
| connectionString    | Especifique la información necesaria para conectarse a la instancia de Azure Synapse Analytics para la propiedad **connectionString**. <br/>Marque este campo como SecureString para almacenarlo de forma segura en Data Factory. También puede colocar la contraseña o clave de la entidad de servicio en Azure Key Vault y, en el caso de la autenticación de SQL, extraer la configuración `password` de la cadena de conexión. Vea el ejemplo de JSON debajo de la tabla y el artículo [Almacenamiento de credenciales en Azure Key Vault](store-credentials-in-key-vault.md) con información detallada. | Sí                                                          |
| servicePrincipalId  | Especifique el id. de cliente de la aplicación.                         | Sí, al utilizar la autenticación de Azure AD con una entidad de servicio. |
| servicePrincipalKey | Especifique la clave de la aplicación. Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | Sí, al utilizar la autenticación de Azure AD con una entidad de servicio. |
| tenant              | Especifique la información del inquilino (nombre de dominio o identificador de inquilino) en el que reside la aplicación. Para recuperarlo, mantenga el puntero del mouse en la esquina superior derecha de Azure Portal. | Sí, al utilizar la autenticación de Azure AD con una entidad de servicio. |
| azureCloudType | Para la autenticación de la entidad de servicio, especifique el tipo de entorno de nube de Azure en el que está registrada la aplicación de Azure AD. <br/> Los valores permitidos son **AzurePublic**, **AzureChina**, **AzureUsGovernment** y **AzureGermany**. De forma predeterminada, se usa el entorno de nube de la factoría de datos. | No |
| connectVia          | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Se puede usar Azure Integration Runtime o un IR autohospedado (si el almacén de datos se encuentra en una red privada). Si no se especifica, se usará Azure Integration Runtime. | No                                                           |

Para ver los distintos tipos de autenticación, consulte las secciones siguientes acerca de requisitos previos y ejemplos de JSON, respectivamente:

- [Autenticación de SQL](#sql-authentication)
- Autenticación de token de la aplicación de Azure AD: [Entidad de servicio](#service-principal-authentication)
- Autenticación de token de la aplicación de Azure AD: [Identidades administradas para recursos de Azure](#managed-identity)

>[!TIP]
>Si recibió un error con código de error como "UserErrorFailedToConnectToSqlServer" y un mensaje como "The session limit for the database is XXX and has been reached" (El límite de sesión de la base de datos es XXX y ya se ha alcanzado), agregue `Pooling=false` a la cadena de conexión e inténtelo de nuevo.

### <a name="sql-authentication"></a>Autenticación SQL

#### <a name="linked-service-example-that-uses-sql-authentication"></a>Ejemplo de servicio vinculado que usa la autenticación de SQL

```json
{
    "name": "AzureSqlDWLinkedService",
    "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
            "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Contraseña en Azure Key Vault:**

```json
{
    "name": "AzureSqlDWLinkedService",
    "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
            "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30",
            "password": {
                "type": "AzureKeyVaultSecret",
                "store": {
                    "referenceName": "<Azure Key Vault linked service name>",
                    "type": "LinkedServiceReference"
                },
                "secretName": "<secretName>"
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

Para usar la autenticación de tokens de aplicaciones de Azure AD basada en una entidad de servicio, realice estos pasos:

1. **[Cree una aplicación de Azure Active Directory](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal)** desde Azure Portal. Anote el nombre de la aplicación y los siguientes valores, que definen el servicio vinculado:

   - Identificador de aplicación
   - Clave de la aplicación
   - Id. de inquilino

2. **[Aprovisione un administrador de Azure Active Directory](../azure-sql/database/authentication-aad-configure.md#provision-azure-ad-admin-sql-database)** para el servidor en Azure Portal, si aún no lo ha hecho. El administrador de Azure AD puede ser un usuario o un grupo de Azure AD. Si concede al grupo con identidad administrada un rol de administrador, omita los pasos 3 y 4. El administrador tendrá acceso completo a la base de datos.

3. **[Cree usuarios de bases de datos independientes](../azure-sql/database/authentication-aad-configure.md#create-contained-users-mapped-to-azure-ad-identities)** para la entidad de servicio. Conéctese al almacenamiento de datos del que desea copiar datos (o en el que desea copiarlos) mediante alguna herramienta como SSMS, con una identidad de Azure AD que tenga al menos permiso para modificar cualquier usuario. Ejecute el T-SQL siguiente:
  
    ```sql
    CREATE USER [your application name] FROM EXTERNAL PROVIDER;
    ```

4. **Conceda a la entidad de servicio los permisos necesarios**, tal como lo haría normalmente para los usuarios de SQL, u otros usuarios. Ejecute el siguiente código, o consulte más opciones [aquí](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-addrolemember-transact-sql?view=sql-server-2017). Si desea usar PolyBase para cargar los datos, infórmese sobre el [permiso de base de datos necesario](#required-database-permission).

    ```sql
    EXEC sp_addrolemember db_owner, [your application name];
    ```

5. En Azure Data Factory, **configure un servicio vinculado de Azure Synapse Analytics**.

#### <a name="linked-service-example-that-uses-service-principal-authentication"></a>Ejemplo de servicio vinculado que usa la autenticación de entidad de servicio

```json
{
    "name": "AzureSqlDWLinkedService",
    "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
            "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;Connection Timeout=30",
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

Una factoría de datos se puede asociar con una [identidad administrada para recursos de Azure](data-factory-service-identity.md) que representa la factoría concreta. Esta identidad administrada se puede usar para la autenticación de Azure Synapse Analytics. Con esta identidad, la fábrica designada puede obtener acceso y copiar datos de la base de datos o en ella.

Para usar la autenticación de identidad administrada, siga estos pasos:

1. **[Aprovisione un administrador de Azure Active Directory](../azure-sql/database/authentication-aad-configure.md#provision-azure-ad-admin-sql-database)** para el servidor en Azure Portal, si aún no lo ha hecho. El administrador de Azure AD puede ser un usuario o un grupo de Azure AD. Si concede al grupo con identidad administrada un rol de administrador, omita los pasos 3 y 4. El administrador tendrá acceso completo a la base de datos.

2. **[Cree usuarios de bases de datos independientes](../azure-sql/database/authentication-aad-configure.md#create-contained-users-mapped-to-azure-ad-identities)** para la identidad administrada de Data Factory. Conéctese al almacenamiento de datos del que desea copiar datos (o en el que desea copiarlos) mediante alguna herramienta como SSMS, con una identidad de Azure AD que tenga al menos permiso para modificar cualquier usuario. Ejecute el siguiente T-SQL.
  
    ```sql
    CREATE USER [your Data Factory name] FROM EXTERNAL PROVIDER;
    ```

3. **Conceda a la identidad administrada de Data Factory los permisos necesarios**, tal como lo haría normalmente para los usuarios de SQL y otros usuarios. Ejecute el siguiente código, o consulte más opciones [aquí](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-addrolemember-transact-sql?view=sql-server-2017). Si desea usar PolyBase para cargar los datos, infórmese sobre el [permiso de base de datos necesario](#required-database-permission).

    ```sql
    EXEC sp_addrolemember db_owner, [your Data Factory name];
    ```

4. En Azure Data Factory, **configure un servicio vinculado de Azure Synapse Analytics**.

**Ejemplo**:

```json
{
    "name": "AzureSqlDWLinkedService",
    "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
            "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;Connection Timeout=30"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md).

Las siguientes propiedades son compatibles con un conjunto d datos de Azure Synapse Analytics:

| Propiedad  | Descripción                                                  | Obligatorio                    |
| :-------- | :----------------------------------------------------------- | :-------------------------- |
| type      | La propiedad **type** del conjunto de datos debe establecerse en **AzureSqlDWTable**. | Sí                         |
| esquema | Nombre del esquema. |No para el origen, sí para el receptor  |
| table | Nombre de la tabla o vista. |No para el origen, sí para el receptor  |
| tableName | Nombre de la tabla o vista con el esquema. Esta propiedad permite la compatibilidad con versiones anteriores. Para la nueva carga de trabajo use `schema` y `table`. | No para el origen, sí para el receptor |

### <a name="dataset-properties-example"></a>Ejemplo de propiedades de un conjunto de datos

```json
{
    "name": "AzureSQLDWDataset",
    "properties":
    {
        "type": "AzureSqlDWTable",
        "linkedServiceName": {
            "referenceName": "<Azure Synapse Analytics linked service name>",
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

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que el origen y el receptor de Azure Synapse Analytics admiten.

### <a name="azure-synapse-analytics-as-the-source"></a>Azure Synapse Analytics como origen

>[!TIP]
>Para cargar datos desde Azure Synapse Analytics de manera eficaz mediante la creación de particiones de datos, vea [Copia en paralelo desde Synapse Analytics](#parallel-copy-from-synapse-analytics).

Para copiar datos desde Azure Synapse Analytics, establezca la propiedad **type** del origen de la actividad de copia en **SqlDWSource**. La sección **source** de la actividad de copia admite las siguientes propiedades:

| Propiedad                     | Descripción                                                  | Obligatorio |
| :--------------------------- | :----------------------------------------------------------- | :------- |
| type                         | La propiedad **type** del origen de la actividad de copia debe establecerse en **SqlDWSource**. | Sí      |
| sqlReaderQuery               | Use la consulta SQL personalizada para leer los datos. Ejemplo: `select * from MyTable`. | No       |
| sqlReaderStoredProcedureName | Nombre del procedimiento almacenado que lee datos de la tabla de origen. La última instrucción SQL debe ser una instrucción SELECT del procedimiento almacenado. | No       |
| storedProcedureParameters    | Parámetros del procedimiento almacenado.<br/>Los valores permitidos son pares de nombre o valor. Los nombres y las mayúsculas y minúsculas de los parámetros deben coincidir con las mismas características de los parámetros de procedimiento almacenado. | No       |
| isolationLevel | Especifica el comportamiento de bloqueo de transacción para el origen de SQL. Los valores permitidos son: **ReadCommitted**, **ReadUncommitted**, **RepeatableRead**, **Serializable** y **Snapshot**. Si no se especifica, se utiliza el nivel de aislamiento predeterminado de la base de datos. Vea [este documento](https://docs.microsoft.com/dotnet/api/system.data.isolationlevel) para obtener más detalles. | No |
| partitionOptions | Especifica las opciones de creación de particiones de datos que se usan para cargar datos desde Azure Synapse Analytics. <br>Los valores permitidos son: **None** (valor predeterminado), **PhysicalPartitionsOfTable** y **DynamicRange**.<br>Cuando se habilita una opción de partición (es decir, no `None`), el grado de paralelismo para cargar datos de forma simultánea desde Azure Synapse Analytics se controla mediante el valor [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) en la actividad de copia. | No |
| partitionSettings | Especifique el grupo de configuración para la creación de particiones de datos. <br>Se aplica si la opción de partición no es `None`. | No |
| ***En`partitionSettings`:*** | | |
| partitionColumnName | Especifique el nombre de la columna de origen **de tipo entero o date/datetime** que va a usar la creación de particiones por rangos para la copia en paralelo. Si no se especifica, el índice o la clave primaria de la tabla se detectan automáticamente y se usan como columna de partición.<br>Se aplica si la opción de partición es `DynamicRange`. Si usa una consulta para recuperar datos de origen, enlace `?AdfDynamicRangePartitionCondition ` en la cláusula WHERE. Para obtener un ejemplo, vea la sección [Copia en paralelo desde una base de datos SQL](#parallel-copy-from-synapse-analytics). | No |
| partitionUpperBound | Valor máximo de la columna de partición para la división del rango de partición. Este valor se usa para decidir el intervalo de particiones, no para filtrar las filas de la tabla. Se crean particiones de todas las filas de la tabla o el resultado de la consulta y se copian. Si no se especifica, la actividad de copia detecta automáticamente el valor.  <br>Se aplica si la opción de partición es `DynamicRange`. Para obtener un ejemplo, vea la sección [Copia en paralelo desde una base de datos SQL](#parallel-copy-from-synapse-analytics). | No |
| partitionLowerBound | Valor mínimo de la columna de partición para la división del rango de partición. Este valor se usa para decidir el intervalo de particiones, no para filtrar las filas de la tabla. Se crean particiones de todas las filas de la tabla o el resultado de la consulta y se copian. Si no se especifica, la actividad de copia detecta automáticamente el valor.<br>Se aplica si la opción de partición es `DynamicRange`. Para obtener un ejemplo, vea la sección [Copia en paralelo desde una base de datos SQL](#parallel-copy-from-synapse-analytics). | No |

**Ejemplo: con la consulta SQL**

```json
"activities":[
    {
        "name": "CopyFromAzureSQLDW",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Azure SQL DW input dataset name>",
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
                "type": "SqlDWSource",
                "sqlReaderQuery": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

**Ejemplo: con el procedimiento almacenado**

```json
"activities":[
    {
        "name": "CopyFromAzureSQLDW",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Azure SQL DW input dataset name>",
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
                "type": "SqlDWSource",
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

**Ejemplo de procedimiento almacenado:**

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

### <a name="azure-synapse-analytics-as-sink"></a><a name="azure-sql-data-warehouse-as-sink"></a> Azure Synapse Analytics como vínculo

Azure Data Factory admite tres maneras de cargar datos en SQL Data Warehouse.

![Opciones de copia de receptor de SQL DataWarehouse](./media/connector-azure-sql-data-warehouse/sql-dw-sink-copy-options.png)

- [Usar PolyBase](#use-polybase-to-load-data-into-azure-sql-data-warehouse)
- [Usar la instrucción COPY (versión preliminar)](#use-copy-statement)
- Usar Inserción masiva

La forma más rápida y escalable de cargar datos es a través de [PolyBase](https://docs.microsoft.com/sql/relational-databases/polybase/polybase-guide) o la [instrucción COPY](https://docs.microsoft.com/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest) (versión preliminar).

Para copiar datos en Azure SQL Data Warehouse, establezca el tipo de receptor de la actividad de copia en **SqlDWSink**. La sección **sink** de la actividad de copia admite las siguientes propiedades:

| Propiedad          | Descripción                                                  | Obligatorio                                      |
| :---------------- | :----------------------------------------------------------- | :-------------------------------------------- |
| type              | La propiedad **type** del receptor de la actividad de copia debe establecerse en **SqlDWSink**. | Sí                                           |
| allowPolyBase     | Indica si se usará PolyBase para cargar datos en SQL Data Warehouse. `allowCopyCommand` y `allowPolyBase` no pueden ser true. <br/><br/>Consulte [Uso de PolyBase para cargar datos en SQL Data Warehouse](#use-polybase-to-load-data-into-azure-sql-data-warehouse) para ver restricciones y más información.<br/><br/>Los valores válidos son **True** y **False** (valor predeterminado). | No.<br/>Se aplica cuando se usa PolyBase.     |
| polyBaseSettings  | Un grupo de propiedades que se pueden especificar cuando el valor de la propiedad `allowPolybase` está establecido en **true**. | No.<br/>Se aplica al usar PolyBase. |
| allowCopyCommand | Indica si se va a usar la [instrucción COPY](https://docs.microsoft.com/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest) (versión preliminar) para cargar datos en SQL Data Warehouse. `allowCopyCommand` y `allowPolyBase` no pueden ser true. <br/><br/>Consulte [Uso de la instrucción COPY para cargar datos en SQL Data Warehouse](#use-copy-statement) para ver restricciones y más información.<br/><br/>Los valores válidos son **True** y **False** (valor predeterminado). | No.<br>Se aplica cuando se usa COPY. |
| copyCommandSettings | Un grupo de propiedades que se pueden especificar cuando el valor de la propiedad `allowCopyCommand` está establecido en TRUE. | No.<br/>Se aplica cuando se usa COPY. |
| writeBatchSize    | Número de filas que se va a insertar en la tabla SQL **por lote**.<br/><br/>El valor que se permite es un **entero** (número de filas). De manera predeterminada, Data Factory determina dinámicamente el tamaño adecuado del lote en función del tamaño de fila. | No.<br/>Se aplica cuando se usa inserción masiva.     |
| writeBatchTimeout | Tiempo que se concede a la operación de inserción por lotes para que finalice antes de que se agote el tiempo de espera.<br/><br/>El valor permitido es **intervalo de tiempo**. Ejemplo: "00:30:00" (30 minutos). | No.<br/>Se aplica cuando se usa inserción masiva.        |
| preCopyScript     | Especifique una consulta SQL para que la actividad de copia se ejecute antes de escribir datos en Azure SQL Data Warehouse en cada ejecución. Esta propiedad se usa para limpiar los datos cargados previamente. | No                                            |
| tableOption | Especifica si [se crea automáticamente la tabla de receptores](copy-activity-overview.md#auto-create-sink-tables) según el esquema de origen, si no existe. Los valores permitidos son: `none` (valor predeterminado), `autoCreate`. |No |
| disableMetricsCollection | Data Factory recopila métricas, como las DWU de SQL Data Warehouse, para la optimización del rendimiento de copia y la obtención de recomendaciones. Si le preocupa este comportamiento, especifique `true` para desactivarlo. | No (el valor predeterminado es `false`) |

#### <a name="sql-data-warehouse-sink-example"></a>Ejemplo de receptor de SQL Data Warehouse

```json
"sink": {
    "type": "SqlDWSink",
    "allowPolyBase": true,
    "polyBaseSettings":
    {
        "rejectType": "percentage",
        "rejectValue": 10.0,
        "rejectSampleValue": 100,
        "useTypeDefault": true
    }
}
```

## <a name="parallel-copy-from-synapse-analytics"></a>Copia en paralelo desde Synapse Analytics

En la actividad de copia, el conector de Azure Synapse Analytics proporciona creación de particiones de datos integrada para copiar los datos en paralelo. Puede encontrar las opciones de creación de particiones de datos en la pestaña **Origen** de la actividad de copia.

![Captura de pantalla de las opciones de partición](./media/connector-sql-server/connector-sql-partition-options.png)

Al habilitar la copia con particiones, la actividad de copia ejecuta consultas en paralelo en el origen de Azure Synapse Analytics para cargar los datos por particiones. El grado en paralelo se controla mediante el valor [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) de la actividad de copia. Por ejemplo, si establece `parallelCopies` en cuatro, Data Factory genera y ejecuta cuatro consultas de forma simultánea de acuerdo con la configuración y la opción de partición que se ha especificado, y cada consulta recupera una porción de datos de Azure Synapse Analytics.

Es recomendable habilitar la copia en paralelo con creación de particiones de datos, especialmente si se cargan grandes cantidades de datos de Azure Synapse Analytics. Estas son algunas configuraciones sugeridas para diferentes escenarios. Cuando se copian datos en un almacén de datos basado en archivos, se recomienda escribirlos en una carpeta como varios archivos (solo especifique el nombre de la carpeta), en cuyo caso el rendimiento es mejor que escribirlos en un único archivo.

| Escenario                                                     | Configuración sugerida                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Carga completa de una tabla grande con particiones físicas.        | **Opción de partición**: particiones físicas de la tabla. <br><br/>Durante la ejecución, Data Factory detecta automáticamente las particiones físicas y copia los datos mediante particiones. |
| Carga completa de una tabla grande, sin particiones físicas, aunque con una columna de tipo entero o datetime para la creación de particiones de datos. | **Opciones de partición**: partición por rangos dinámica.<br>**Columna de partición** (opcional): especifique la columna usada para crear la partición de datos. Si no se especifica, se usa la columna de índice o clave principal.<br/>**Límite de partición superior** y **límite de partición inferior** (opcional): especifique si quiere determinar el intervalo de la partición. No es para filtrar las filas de la tabla, se crean particiones de todas las filas de la tabla y se copian. Si no se especifica, la actividad de copia detecta automáticamente los valores.<br><br>Por ejemplo, si la columna de partición "ID" tiene valores que van de 1 a 100 y establece el límite inferior en 20 y el superior en 80, con la copia en paralelo establecida en 4, Data Factory recupera los datos por 4 particiones: los id. del rango <=20, [21, 50], [51, 80] y >=81, respectivamente. |
| Carga de grandes cantidades de datos mediante una consulta personalizada, sin particiones físicas, aunque con una columna de tipo entero o date/datetime para la creación de particiones de datos. | **Opciones de partición**: partición por rangos dinámica.<br>**Consulta**: `SELECT * FROM <TableName> WHERE ?AdfDynamicRangePartitionCondition AND <your_additional_where_clause>`.<br>**Columna de partición**: especifique la columna usada para crear la partición de datos.<br>**Límite de partición superior** y **límite de partición inferior** (opcional): especifique si quiere determinar el intervalo de la partición. No es para filtrar las filas de la tabla, se crean particiones de todas las filas del resultado de la consulta y se copian. Si no se especifica, la actividad de copia detecta automáticamente el valor.<br><br>Durante la ejecución, Data Factory reemplaza `?AdfRangePartitionColumnName` por el nombre real de la columna y los rangos de valor de cada partición y se los envía a Azure Synapse Analytics. <br>Por ejemplo, si la columna de partición "ID" tiene valores que van de 1 a 100 y establece el límite inferior en 20 y el superior en 80, con la copia en paralelo establecida en 4, Data Factory recupera los datos por 4 particiones: los id. del rango <=20, [21, 50], [51, 80] y >=81, respectivamente. |

Procedimientos recomendados para cargar datos con la opción de partición:

1. Seleccione una columna distintiva como columna de partición (como clave principal o clave única) para evitar la asimetría de datos. 
2. Si la tabla tiene una partición integrada, use la opción de partición "Particiones físicas de tabla" para obtener un mejor rendimiento.
3. Si usa Azure Integration Runtime para copiar datos, puede establecer "[unidades de integración de datos (DIU)](copy-activity-performance-features.md#data-integration-units)" mayores (> 4) para usar más recursos de cálculo. Compruebe los escenarios aplicables allí.
4. "[Grado de paralelismo de copia](copy-activity-performance-features.md#parallel-copy)" controla los números de partición. Si se establece en un número demasiado grande, el rendimiento puede resentirse, de modo que se recomienda establecerlo como (DIU o número de nodos de IR autohospedados) * (2 a 4).
5. Tenga en cuenta que Azure Synapse Analytics puede ejecutar un máximo de 32 consultas en un momento dado; si "Grado de paralelismo de copia" se establece en un número demasiado grande, puede producirse un problema de límite de Synapse.

**Ejemplo: carga completa de una tabla grande con particiones físicas**

```json
"source": {
    "type": "SqlDWSource",
    "partitionOption": "PhysicalPartitionsOfTable"
}
```

**Ejemplo: consulta con partición por rangos dinámica**

```json
"source": {
    "type": "SqlDWSource",
    "query": "SELECT * FROM <TableName> WHERE ?AdfDynamicRangePartitionCondition AND <your_additional_where_clause>",
    "partitionOption": "DynamicRange",
    "partitionSettings": {
        "partitionColumnName": "<partition_column_name>",
        "partitionUpperBound": "<upper_value_of_partition_column (optional) to decide the partition stride, not as data filter>",
        "partitionLowerBound": "<lower_value_of_partition_column (optional) to decide the partition stride, not as data filter>"
    }
}
```

## <a name="use-polybase-to-load-data-into-azure-sql-data-warehouse"></a>Movimiento de datos hacia y desde Azure SQL Data Warehouse mediante Azure Data Factory

Usar [PolyBase](https://docs.microsoft.com/sql/relational-databases/polybase/polybase-guide) es una manera eficaz de cargar grandes cantidades de datos en Azure Synapse Analytics con un alto rendimiento. Verá una gran mejora en el rendimiento mediante el uso de PolyBase en lugar del mecanismo BULKINSERT predeterminado. Para un tutorial con un caso de uso, vea [Carga de datos en Azure SQL Data Warehouse mediante Azure Synapse Analytics](v1/data-factory-load-sql-data-warehouse.md).

- Si los datos de origen están en **Azure Blob, Azure Data Lake Storage Gen1 o Azure Data Lake Storage Gen2** y el **formato es compatible con PolyBase**, puede usar la actividad de copia para invocar directamente PolyBase para permitir que Azure SQL Data Warehouse extraiga los datos del origen. Consulte **[Copia directa con PolyBase](#direct-copy-by-using-polybase)** para obtener detalles.
- Si el formato y el almacenamiento de datos de origen no es compatible originalmente con PolyBase, use en su lugar la característica **[Copia almacenada provisionalmente con PolyBase](#staged-copy-by-using-polybase)** . La característica de copia almacenada provisionalmente también proporciona un mejor rendimiento. Convierte automáticamente los datos en formato compatible con PolyBase, almacena los datos en Azure Blog Storage y luego llama a PolyBase para cargar los datos en SQL Data Warehouse.

> [!TIP]
> Más información en [Prácticas recomendadas para usar PolyBase](#best-practices-for-using-polybase).

Esta configuración de PolyBase es compatible con `polyBaseSettings` en la actividad de copia:

| Propiedad          | Descripción                                                  | Obligatorio                                      |
| :---------------- | :----------------------------------------------------------- | :-------------------------------------------- |
| rejectValue       | Especifica el número o porcentaje de filas que se pueden rechazar antes de que se produzca un error en la consulta.<br/><br/>Más información sobre las opciones de rechazo de PolyBase en la sección Argumentos de [CREATE EXTERNAL TABLE (Transact-SQL)](https://msdn.microsoft.com/library/dn935021.aspx). <br/><br/>Los valores permitidos son 0 (valor predeterminado), 1, 2, etc. | No                                            |
| rejectType        | Especifica si la opción **rejectValue** es un valor literal o un porcentaje.<br/><br/>Los valores permitidos son **Value** (valor predeterminado) y **Percentage**. | No                                            |
| rejectSampleValue | Determina el número de filas que se van a recuperar antes de que PolyBase vuelva a calcular el porcentaje de filas rechazadas.<br/><br/>Los valores permitidos son 1, 2, etc. | Sí, si el valor de **rejectType** es **percentage**. |
| useTypeDefault    | Especifica cómo administrar valores que faltan en archivos de texto delimitado cuando PolyBase recupera datos del archivo de texto.<br/><br/>Más información sobre esta propiedad en la sección de argumentos de [CREATE EXTERNAL FILE FORMAT (Transact-SQL)](https://msdn.microsoft.com/library/dn935026.aspx).<br/><br/>Los valores válidos son **True** y **False** (valor predeterminado).<br><br> | No                                            |

### <a name="direct-copy-by-using-polybase"></a>Copia directa con PolyBase

PolyBase de SQL Data Warehouse admite directamente Azure Blob, Azure Data Lake Storage Gen1 y Azure Data Lake Storage Gen2. Si los datos de origen cumplen los criterios descritos en esta sección, use PolyBase para copiar directamente desde el almacén de datos de origen en Azure Synapse Analytics. De lo contrario, use [Copia almacenada provisionalmente con PolyBase](#staged-copy-by-using-polybase).

> [!TIP]
> Para copiar datos de forma eficaz en SQL Data Warehouse, obtenga más información en [Azure Data Factory makes it even easier and convenient to uncover insights from data when using Data Lake Store with SQL Data Warehouse](https://blogs.msdn.microsoft.com/azuredatalake/2017/04/08/azure-data-factory-makes-it-even-easier-and-convenient-to-uncover-insights-from-data-when-using-data-lake-store-with-sql-data-warehouse/) (Azure Data Factory hace incluso más fácil y cómodo el descubrimiento de información de datos cuando se usa Data Lake Store con SQL Data Warehouse).

Si no se cumplen los requisitos, Azure Data Factory comprobará la configuración y volverá automáticamente al mecanismo BULKINSERT para realizar el movimiento de datos.

1. El **servicio vinculado de origen** tiene los siguientes tipos y métodos de autenticación:

    | Tipo de almacén de datos de origen admitido                             | Tipo de autenticación de origen admitido                        |
    | :----------------------------------------------------------- | :---------------------------------------------------------- |
    | [Azure Blob](connector-azure-blob-storage.md)                | Autenticación de clave de cuenta y autenticación de identidad administrada |
    | [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md) | Autenticación de entidad de servicio                            |
    | [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md) | Autenticación de clave de cuenta y autenticación de identidad administrada |

    >[!IMPORTANT]
    >Si Azure Storage está configurado con el punto de conexión de servicio de red virtual, tiene que utilizar la autenticación de identidad administrada; consulte [Efectos del uso de puntos de conexión de servicio de la red virtual con Azure Storage](../azure-sql/database/vnet-service-endpoint-rule-overview.md#impact-of-using-vnet-service-endpoints-with-azure-storage). Obtenga información sobre las configuraciones necesarias en Data Factory en las secciones [Azure Blob: autenticación de identidad administrada](connector-azure-blob-storage.md#managed-identity) y [Azure Data Lake Storage Gen2: autenticación de identidad administrada](connector-azure-data-lake-storage.md#managed-identity) respectivamente.

2. El **formato de datos de origen** es de **Parquet**, **ORC**, o **texto delimitado**, con las siguientes configuraciones:

   1. La ruta de acceso de la carpeta no contiene el filtro de comodín.
   2. El nombre de archivo está vacío o apunta a un solo archivo. Si especifica un nombre de archivo de comodín en la actividad de copia, solo puede ser `*` o `*.*`.
   3. `rowDelimiter` es **valor predeterminado**, **\n**, **\r\n** o **\r**.
   4. `nullValue` se deja con el valor predeterminado o se establece en **empty string** ("") y `treatEmptyAsNull` se deja con el valor predeterminado o se establece en True.
   5. `encodingName` se deja con el valor predeterminado o se establece en **utf-8**.
   6. `quoteChar`, `escapeChar` y `skipLineCount` no están especificados. Fila de encabezado de omisión de compatibilidad de PolyBase que se puede configurar como `firstRowAsHeader` en ADF.
   7. `compression` puede ser **no compression**, **GZip** o **Deflate**.

3. Si el origen es una carpeta, `recursive` de la actividad de copia se debe establecer en True.

4. `wildcardFolderPath` , `wildcardFilename`, `modifiedDateTimeStart`, `modifiedDateTimeEnd`, `prefix`, `enablePartitionDiscovery` y `additionalColumns` no se especifican.

>[!NOTE]
>Si el origen es una carpeta, observe que PolyBase recupera archivos de la carpeta y todas sus subcarpetas y no recupera datos de los archivos para los cuales el nombre de archivo empieza con un guion bajo (_) o un punto (.), tal como se documenta [aquí, en el argumento LOCATION](https://docs.microsoft.com/sql/t-sql/statements/create-external-table-transact-sql?view=azure-sqldw-latest#arguments-2).

```json
"activities":[
    {
        "name": "CopyFromAzureBlobToSQLDataWarehouseViaPolyBase",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "ParquetDataset",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "AzureSQLDWDataset",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "ParquetSource",
                "storeSettings":{
                    "type": "AzureBlobStorageReadSettings",
                    "recursive": true
                }
            },
            "sink": {
                "type": "SqlDWSink",
                "allowPolyBase": true
            }
        }
    }
]
```

### <a name="staged-copy-by-using-polybase"></a>Copia almacenada provisionalmente con PolyBase

Si los datos de origen no son compatibles de forma nativa con PolyBase, habilite la copia de datos a través de una instancia provisional de Azure Blob Storage  (no puede ser Azure Premium Storage). En este caso, Azure Data Factory convierte automáticamente los datos para satisfacer los requisitos del formato de datos de PolyBase. Después, invoca PolyBase para cargar datos en SQL Data Warehouse. Por último, limpia los datos temporales del almacenamiento de blobs. Consulte [Copia almacenada provisionalmente](copy-activity-performance-features.md#staged-copy) para más información sobre la copia de datos por medio de una instancia de Azure Blob Storage de almacenamiento provisional.

Para utilizar esta característica, cree un [servicio vinculado de Azure Blob Storage](connector-azure-blob-storage.md#linked-service-properties) que haga referencia a la cuenta de Azure Storage con el almacenamiento de blobs interino. Luego especifique las propiedades `enableStaging` y `stagingSettings` para la actividad de copia, tal como se muestra en el código siguiente.

>[!IMPORTANT]
>Si el almacenamiento provisional de Azure Storage está configurado con el punto de conexión de servicio de red virtual, tiene que utilizar la autenticación de identidad administrada; consulte [Efectos del uso de puntos de conexión de servicio de la red virtual con Azure Storage](../azure-sql/database/vnet-service-endpoint-rule-overview.md#impact-of-using-vnet-service-endpoints-with-azure-storage). Obtenga información sobre las configuraciones necesarias en Data Factory en [Azure Blob: autenticación de identidad administrada](connector-azure-blob-storage.md#managed-identity).

```json
"activities":[
    {
        "name": "CopyFromSQLServerToSQLDataWarehouseViaPolyBase",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "SQLServerDataset",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "AzureSQLDWDataset",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "SqlSource",
            },
            "sink": {
                "type": "SqlDWSink",
                "allowPolyBase": true
            },
            "enableStaging": true,
            "stagingSettings": {
                "linkedServiceName": {
                    "referenceName": "MyStagingBlob",
                    "type": "LinkedServiceReference"
                }
            }
        }
    }
]
```

### <a name="best-practices-for-using-polybase"></a>Prácticas recomendadas para usar PolyBase

En las secciones siguientes se describen procedimientos recomendados, además de los que se mencionan en [Procedimientos recomendados para Azure Synapse Analytics](../synapse-analytics/sql/best-practices-sql-pool.md).

#### <a name="required-database-permission"></a>Permiso de base de datos necesario

Para usar PolyBase, el usuario que carga los datos en SQL Data Warehouse debe tener el [permiso "CONTROL"](https://msdn.microsoft.com/library/ms191291.aspx) en la base de datos de destino. Una manera de conseguirlo es agregar el usuario como miembro del rol **db_owner**. Obtenga información sobre cómo hacerlo en la [información general de SQL Data Warehouse](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-manage-security.md#authorization).

#### <a name="row-size-and-data-type-limits"></a>Límites del tipo de datos y del tamaño de fila

Las cargas de PolyBase están limitadas a filas más pequeñas que 1 MB. No se puede usar para cargar en VARCHR(MAX), NVARCHAR(MAX) ni VARBINARY(MAX). Para más información, consulte [Límites de capacidad de SQL Data Warehouse](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-service-capacity-limits.md#loads).

Cuando los datos de origen tienen filas mayores de 1 MB, es aconsejable dividir verticalmente las tablas de origen en otras más pequeñas. Asegúrese de que el tamaño mayor de cada fila no excede el límite. Las tablas más pequeñas se pueden cargar con PolyBase y combinar en Azure Synapse Analytics.

Como alternativa, para los datos con estas columnas anchas, puede usar no PolyBase para cargar los datos de uso de ADF, para ello desactive el valor "Permitir PolyBase".

#### <a name="sql-data-warehouse-resource-class"></a>Clase de recursos de SQL Data Warehouse

Para obtener el mejor rendimiento posible, asigne una clase de recurso mayor al usuario que carga datos en SQL Data Warehouse a través de PolyBase.

#### <a name="polybase-troubleshooting"></a>Solución de problemas de PolyBase

**Carga en una columna decimal**

Si los datos de origen están en formato de texto, u otro almacén compatible no PolyBase (con copias almacenadas provisionalmente y PolyBase), y contienen un valor vacío para cargarse en una columna Decimal de SQL Data Warehouse, puede encontrarse con el siguiente error:

```
ErrorCode=FailedDbOperation, ......HadoopSqlException: Error converting data type VARCHAR to DECIMAL.....Detailed Message=Empty string can't be converted to DECIMAL.....
```

La solución consiste en anular la selección de la opción "**Use type default**" (Usar tipo predeterminado) (como falsa) en el receptor de la actividad de copia -> configuración de PolyBase. "[USE_TYPE_DEFAULT](https://docs.microsoft.com/sql/t-sql/statements/create-external-file-format-transact-sql?view=azure-sqldw-latest#arguments
)" es una configuración nativa PolyBase que especifica cómo administrar los valores que faltan en archivos de texto delimitados cuando PolyBase recupera datos del archivo de texto.

**`tableName` en Azure Synapse Analytics**

En la siguiente tabla se proporcionan ejemplos de cómo especificar la propiedad **tableName** en el conjunto de datos JSON. Se muestran varias combinaciones de esquema y nombres de tabla.

| Esquema de base de datos | Nombre de la tabla | Propiedad JSON **tableName**               |
| --------- | ---------- | ----------------------------------------- |
| dbo       | MyTable    | MyTable o dbo.MyTable o [dbo].[MyTable] |
| dbo1      | MyTable    | dbo1.MyTable o [dbo1].[MyTable]          |
| dbo       | My.Table   | [My.Table] o [dbo].[My.Table]            |
| dbo1      | My.Table   | [dbo1].[My.Table]                         |

Si ve el siguiente error, el problema podría ser el valor especificado para la propiedad **tableName**. Consulte en la tabla anterior la forma correcta de especificar los valores para la propiedad **tableName** de JSON.

```
Type=System.Data.SqlClient.SqlException,Message=Invalid object name 'stg.Account_test'.,Source=.Net SqlClient Data Provider
```

**Columnas con valores predeterminados**

Actualmente, la característica PolyBase en Data Factory solo acepta el mismo número de columnas que la tabla de destino. Un ejemplo sería una tabla con cuatro columnas y que una de ellas esté definida con un valor predeterminado. Los datos de entrada siguen necesitando cuatro columnas. Un conjunto de datos de entrada de tres columnas producirá un error parecido al siguiente mensaje:

```
All columns of the table must be specified in the INSERT BULK statement.
```

El valor NULL es una forma especial del valor predeterminado. Si la columna acepta valores nulos, los datos de entrada del blob para esa columna pueden estar vacíos. Pero no puede faltar del conjunto de datos de entrada. PolyBase insertará valores NULL para los valores que falten en Azure Synapse Analytics.

## <a name="use-copy-statement-to-load-data-into-azure-sql-data-warehouse-preview"></a><a name="use-copy-statement"></a> Usar la instrucción COPY para cargar datos en Azure SQL Data Warehouse (versión preliminar)

La [instrucción COPY](https://docs.microsoft.com/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest) de Azure SQL Data Warehouse (versión preliminar) admite directamente la carga de datos de **Azure Blob y Azure Data Lake Storage Gen2**. Si los datos de origen cumplen los criterios descritos en esta sección, puede optar por usar la instrucción COPY en ADF para cargar datos en Azure SQL Data Warehouse. Azure Data Factory comprueba la configuración y produce un error en la ejecución de la actividad de copia si no se cumplen los criterios.

>[!NOTE]
>Actualmente, Data Factory solo admite la copia de los orígenes compatibles con la instrucción COPY que se mencionan a continuación.

La instrucción COPY es compatible con esta configuración:

1. El **formato y el servicio vinculado de origen** tienen los siguientes tipos y métodos de autenticación:

    | Tipo de almacén de datos de origen admitido                             | Formato admitido           | Tipo de autenticación de origen admitido                         |
    | :----------------------------------------------------------- | -------------------------- | :----------------------------------------------------------- |
    | [Azure Blob](connector-azure-blob-storage.md)                | [Texto delimitado](format-delimited-text.md)             | Autenticación de clave de cuenta, autenticación de firma de acceso compartido, autenticación de entidad de servicio, autenticación de identidad administrada |
    | &nbsp;                                                       | [Parquet](format-parquet.md)                    | Autenticación de clave de cuenta, autenticación de firma de acceso compartido |
    | &nbsp;                                                       | [ORC](format-orc.md)                        | Autenticación de clave de cuenta, autenticación de firma de acceso compartido |
    | [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md) | [Texto delimitado](format-delimited-text.md)<br/>[Parquet](format-parquet.md)<br/>[ORC](format-orc.md) | Autenticación de clave de cuenta, autenticación de entidad de servicio, autenticación de identidad administrada |

    >[!IMPORTANT]
    >Si Azure Storage está configurado con el punto de conexión de servicio de red virtual, tiene que utilizar la autenticación de identidad administrada; consulte [Efectos del uso de puntos de conexión de servicio de la red virtual con Azure Storage](../azure-sql/database/vnet-service-endpoint-rule-overview.md#impact-of-using-vnet-service-endpoints-with-azure-storage). Obtenga información sobre las configuraciones necesarias en Data Factory en las secciones [Azure Blob: autenticación de identidad administrada](connector-azure-blob-storage.md#managed-identity) y [Azure Data Lake Storage Gen2: autenticación de identidad administrada](connector-azure-data-lake-storage.md#managed-identity) respectivamente.

2. Esta es la configuración de formato:

   1. Para **Parquet**: `compression` puede ser **sin compresión**, **Snappy** o **GZip**.
   2. Para **ORC**: `compression` puede ser **sin compresión**, **```zlib```** o **Snappy**.
   3. Para **Texto delimitado**:
      1. `rowDelimiter` se establece explícitamente como **carácter único** o " **\r\n**"; no se admite el valor predeterminado.
      2. `nullValue` se deja con el valor predeterminado o se establece en **cadena vacía** ("").
      3. `encodingName` se deja con el valor predeterminado o se establece en **utf-8 o utf-16**.
      4. `escapeChar` debe ser igual que `quoteChar` y no está vacío.
      5. `skipLineCount` se deja como valor predeterminado o se establece en 0.
      6. `compression` puede ser **sin compresión** o **GZip**.

3. Si el origen es una carpeta, `recursive` de la actividad de copia se debe establecer en True y `wildcardFilename` tiene que ser `*`. 

4. `wildcardFolderPath` , `wildcardFilename` (distintos de `*`), `modifiedDateTimeStart`, `modifiedDateTimeEnd`, `prefix`, `enablePartitionDiscovery` y `additionalColumns` no se especifican.

Esta configuración de la instrucción COPY es compatible con `allowCopyCommand` en la actividad de copia:

| Propiedad          | Descripción                                                  | Obligatorio                                      |
| :---------------- | :----------------------------------------------------------- | :-------------------------------------------- |
| defaultValues | Especifica los valores predeterminados para cada columna de destino en SQL Data Warehouse.  Los valores predeterminados de la propiedad sobrescriben el conjunto de restricciones predeterminado en el almacenamiento de datos, y la columna de identidad no puede tener un valor predeterminado. | No |
| additionalOptions | Opciones adicionales que se pasarán directamente a la instrucción COPY de SQL Data Warehouse en la cláusula "With" en la [instrucción COPY](https://docs.microsoft.com/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest). Incluye el valor entre comillas si es necesario para ajustarlo a los requisitos de la instrucción COPY. | No |

```json
"activities":[
    {
        "name": "CopyFromAzureBlobToSQLDataWarehouseViaCOPY",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "ParquetDataset",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "AzureSQLDWDataset",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "ParquetSource",
                "storeSettings":{
                    "type": "AzureBlobStorageReadSettings",
                    "recursive": true
                }
            },
            "sink": {
                "type": "SqlDWSink",
                "allowCopyCommand": true,
                "copyCommandSettings": {
                    "defaultValues": [
                        {
                            "columnName": "col_string",
                            "defaultValue": "DefaultStringValue"
                        }
                    ],
                    "additionalOptions": {
                        "MAXERRORS": "10000",
                        "DATEFORMAT": "'ymd'"
                    }
                }
            },
            "enableSkipIncompatibleRow": true
        }
    }
]
```

## <a name="mapping-data-flow-properties"></a>Propiedades de Asignación de instancias de Data Flow

Al transformar datos en Asignación de Data Flow, puede leer y escribir en las tablas de Azure Synapse Analytics. Para más información, vea la [transformación de origen](data-flow-source.md) y la [transformación de receptor](data-flow-sink.md) en Asignación de Data Flow.

### <a name="source-transformation"></a>Transformación de origen

La configuración específica de Azure Synapse Analytics está disponible en la pestaña **Opciones de origen** de la transformación de origen.

**Entrada** Seleccione si desea señalar el origen en una tabla (equivale a ```Select * from <table-name>```) o escribir una consulta SQL personalizada.

**Enable Staging** (Habilitar almacenamiento provisional) Se recomienda encarecidamente usar esta opción en cargas de trabajo de producción con orígenes de Synapse DW. Al ejecutar una actividad de flujo de datos con orígenes de Synapse desde una canalización, ADF le solicitará una cuenta de almacenamiento de ubicación de almacenamiento provisional y la usará para la carga de datos almacenados provisionalmente. Es el mecanismo más rápido para cargar datos desde Synapse DW.

**Consultar** Si selecciona Consulta en el campo de entrada, escriba una consulta SQL para el origen. Esta configuración invalidará cualquier tabla que haya elegido en el conjunto de datos. Las cláusulas **Ordenar por** no se admiten aquí, pero puede establecer una instrucción SELECT FROM completa. También puede usar las funciones de tabla definidas por el usuario. **select * from udfGetData()** es un UDF in SQL que devuelve una tabla. Esta consulta genera una tabla de origen que puede usar en el flujo de datos. El uso de consultas también es una excelente manera de reducir las filas para pruebas o búsquedas.

Ejemplo de SQL: ```Select * from MyTable where customerId > 1000 and customerId < 2000```

**Tamaño del lote**: escriba un tamaño de lote para fragmentar datos grandes en lecturas. En los flujos de datos, ADF usará esta configuración para establecer el almacenamiento en caché de columnas de Spark. Se trata de un campo de opción que usará los valores predeterminados de Spark si se deja en blanco.

**Nivel de aislamiento**: El valor predeterminado de los orígenes de SQL en Mapping Data Flow es de lectura no confirmada. Puede cambiar el nivel de aislamiento aquí a uno de estos valores:

- Read Committed
- Read Uncommitted
- Repeatable Read
- Serializable *: ninguno (ignorar el nivel de aislamiento)

![Nivel de aislamiento](media/data-flow/isolationlevel.png "Nivel de aislamiento"):

### <a name="sink-transformation"></a>Transformación de receptor

La configuración específica de Azure Synapse Analytics está disponible en la pestaña **Configuración** de la transformación de receptor.

**Update method** (Método de actualización): determina qué operaciones se permiten en el destino de la base de datos. El valor predeterminado es permitir solamente las inserciones. Para realizar las operaciones update, upsert o delete rows, se requiere una transformación de alteración de filas para etiquetar esas acciones. En el caso de las actualizaciones, upserts y eliminaciones, se debe establecer una o varias columnas de clave para determinar la fila que se va a modificar.

**Acción de tabla**: determina si se deben volver a crear o quitar todas las filas de la tabla de destino antes de escribir.

- None (Ninguna): no se realizará ninguna acción en la tabla.
- Recreate (Volver a crear): se quitará la tabla y se volverá a crear. Obligatorio si se crea una nueva tabla dinámicamente.
- Truncate (Truncar): se quitarán todas las filas de la tabla de destino.

**Enable staging** (Permitir almacenamiento provisional): determina si se debe usar [PolyBase](https://docs.microsoft.com/sql/relational-databases/polybase/polybase-guide?view=sql-server-ver15) al escribir en Azure Synapse Analytics

**Tamaño del lote**: controla el número de filas que se escriben en cada cubo. Los tamaños de lote más grandes mejoran la compresión y la optimización de memoria, pero se arriesgan a obtener excepciones de memoria al almacenar datos en caché.

**Scripts SQL anteriores y posteriores**: escriba scripts de SQL de varias líneas que se ejecutarán antes (preprocesamiento) y después (procesamiento posterior) de que los datos se escriban en la base de datos del receptor.

![Scripts previos y posteriores al procesamiento de SQL](media/data-flow/prepost1.png "Scripts de procesamiento SQL")

## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para obtener información detallada sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).

## <a name="getmetadata-activity-properties"></a>Propiedades de la actividad GetMetadata

Para información detallada sobre las propiedades, consulte [Actividad de obtención de metadatos](control-flow-get-metadata-activity.md).

## <a name="data-type-mapping-for-azure-synapse-analytics"></a>Asignación de tipos de datos para Azure Synapse Analytics

Al copiar datos con Azure Synapse Analytics como origen o destino, se usan las siguientes asignaciones de tipos de datos de Azure Synapse Analytics en los tipos de datos provisionales de Azure Data Factory. Para obtener información acerca de la forma en que la actividad de copia asigna el esquema de origen y el tipo de datos al receptor, consulte [Asignación de esquemas en la actividad de copia](copy-activity-schema-and-type-mapping.md).

>[!TIP]
>Vea el artículo [Tipos de datos de tabla en Azure Synapse Analytics](../synapse-analytics/sql/develop-tables-data-types.md) sobre los tipos de datos admitidos en Azure SQL Data Warehouse y las soluciones alternativas para los no admitidos.

| Tipo de datos de Azure Synapse Analytics    | Tipo de datos provisionales de Data Factory |
| :------------------------------------ | :----------------------------- |
| bigint                                | Int64                          |
| binary                                | Byte[]                         |
| bit                                   | Boolean                        |
| char                                  | String, Char[]                 |
| date                                  | DateTime                       |
| Datetime                              | DateTime                       |
| datetime2                             | DateTime                       |
| Datetimeoffset                        | DateTimeOffset                 |
| Decimal                               | Decimal                        |
| FILESTREAM attribute (varbinary(max)) | Byte[]                         |
| Float                                 | Double                         |
| imagen                                 | Byte[]                         |
| int                                   | Int32                          |
| money                                 | Decimal                        |
| NCHAR                                 | String, Char[]                 |
| NUMERIC                               | Decimal                        |
| NVARCHAR                              | String, Char[]                 |
| real                                  | Single                         |
| rowversion                            | Byte[]                         |
| smalldatetime                         | DateTime                       |
| SMALLINT                              | Int16                          |
| SMALLMONEY                            | Decimal                        |
| time                                  | TimeSpan                       |
| TINYINT                               | Byte                           |
| UNIQUEIDENTIFIER                      | Guid                           |
| varbinary                             | Byte[]                         |
| varchar                               | String, Char[]                 |

## <a name="next-steps"></a>Pasos siguientes

Consulte los [formatos y almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de los almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores.
