---
title: Mecanismos de autenticación con la instrucción COPY
description: Describe los mecanismos de autenticación para la carga masiva de datos
services: synapse-analytics
author: kevinvngo
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql-dw
ms.date: 07/10/2020
ms.author: kevin
ms.reviewer: jrasnick
ms.openlocfilehash: 05dd1f1d429b59c4d621b63c6b78a1fc00e8d4dd
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87528470"
---
# <a name="securely-load-data-using-synapse-sql"></a>Carga de datos de forma segura mediante el uso de Synapse SQL

En este artículo se resaltan los mecanismos de autenticación segura para la instrucción [COPY](https://docs.microsoft.com/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest) y se muestran ejemplos al respecto. La instrucción COPY es la forma más flexible y segura de cargar datos de forma masiva en Synapse SQL.
## <a name="supported-authentication-mechanisms"></a>Mecanismos de autenticación compatibles

En la siguiente matriz se describen los métodos de autenticación compatibles tanto con cada tipo de archivo como con una cuenta de almacenamiento. Esto se aplica a la ubicación de almacenamiento de origen y a la ubicación del archivo de error.

|                          |                CSV                |              Parquet              |                ORC                |
| :----------------------: | :-------------------------------: | :-------------------------------: | :-------------------------------: |
|  **Azure Blob Storage**  | SAS/MSI/SERVICE PRINCIPAL/KEY/AAD |              SAS/KEY              |              SAS/KEY              |
| **Azure Data Lake Gen2** | SAS/MSI/SERVICE PRINCIPAL/KEY/AAD | SAS/MSI/SERVICE PRINCIPAL/KEY/AAD | SAS/MSI/SERVICE PRINCIPAL/KEY/AAD |

## <a name="a-storage-account-key-with-lf-as-the-row-terminator-unix-style-new-line"></a>A. Clave de cuenta de almacenamiento con LF como terminador de fila (nueva línea de estilo Unix)


```sql
--Note when specifying the column list, input field numbers start from 1
COPY INTO target_table (Col_one default 'myStringDefault' 1, Col_two default 1 3)
FROM 'https://adlsgen2account.dfs.core.windows.net/myblobcontainer/folder1/'
WITH (
    FILE_TYPE = 'CSV'
    ,CREDENTIAL=(IDENTITY= 'Storage Account Key', SECRET='<Your_Account_Key>')
    --CREDENTIAL should look something like this:
    --CREDENTIAL=(IDENTITY= 'Storage Account Key', SECRET='x6RWv4It5F2msnjelv3H4DA80n0QW0daPdw43jM0nyetx4c6CpDkdj3986DX5AHFMIf/YN4y6kkCnU8lb+Wx0Pj+6MDw=='),
    ,ROWTERMINATOR='0x0A' --0x0A specifies to use the Line Feed character (Unix based systems)
)
```
> [!IMPORTANT]
>
> - Use el valor hexadecimal (0x0A) para especificar el carácter de avance de línea/nueva línea. Tenga en cuenta que la instrucción COPY interpretará la cadena "\n" como "\r\n" (nueva línea de retorno de carro).

## <a name="b-shared-access-signatures-sas-with-crlf-as-the-row-terminator-windows-style-new-line"></a>B. Firmas de acceso compartido (SAS) con CRLF como terminador de fila (nueva línea con estilo de Windows)
```sql
COPY INTO target_table
FROM 'https://adlsgen2account.dfs.core.windows.net/myblobcontainer/folder1/'
WITH (
    FILE_TYPE = 'CSV'
    ,CREDENTIAL=(IDENTITY= 'Shared Access Signature', SECRET='<Your_SAS_Token>')
    --CREDENTIAL should look something like this:
    --CREDENTIAL=(IDENTITY= 'Shared Access Signature', SECRET='?sv=2018-03-28&ss=bfqt&srt=sco&sp=rl&st=2016-10-17T20%3A14%3A55Z&se=2021-10-18T20%3A19%3A00Z&sig=IEoOdmeYnE9%2FKiJDSFSYsz4AkNa%2F%2BTx61FuQ%2FfKHefqoBE%3D'),
    ,ROWTERMINATOR='\n'-- COPY command automatically prefixes the \r character when \n (newline) is specified. This results in carriage return newline (\r\n) for Windows based systems.
)
```

> [!IMPORTANT]
>
> - No especifique ROWTERMINATOR como "\r\n", lo que se interpretará como "\r\r\n" y puede dar lugar a problemas de análisis.

## <a name="c-managed-identity"></a>C. Identidad administrada

La autenticación de Identidad administrada es necesaria cuando la cuenta de almacenamiento está conectada a una red virtual. 

### <a name="prerequisites"></a>Requisitos previos

1. Instale Azure PowerShell mediante esta [guía](/powershell/azure/install-az-ps?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).
2. Si tiene una cuenta de uso general v1 o de Blob Storage, primero debe actualizar a Uso general v2 mediante esta [guía](../../storage/common/storage-account-upgrade.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).
3. Debe activar **Permitir que los servicios de Microsoft de confianza accedan a esta cuenta de almacenamiento** en el menú de configuración **Firewalls y redes virtuales** de la cuenta de Azure Storage. Consulte [esta guía](../../storage/common/storage-network-security.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#exceptions) para obtener más información.
#### <a name="steps"></a>Pasos

1. En PowerShell, **registre el servidor SQL lógico** con Azure Active Directory (AAD):

   ```powershell
   Connect-AzAccount
   Select-AzSubscription -SubscriptionId your-subscriptionId
   Set-AzSqlServer -ResourceGroupName your-database-server-resourceGroup -ServerName your-database-servername -AssignIdentity
   ```

2. Cree una **cuenta de almacenamiento de uso general v2** con esta [guía](../../storage/common/storage-account-create.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).

   > [!NOTE]
   > Si tiene una cuenta de uso general v1 o de Blob Storage, **primero debe actualizar a Uso general v2** mediante esta [guía](../../storage/common/storage-account-upgrade.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).

3. En la cuenta de almacenamiento, vaya a **Control de acceso (IAM)** y seleccione **Agregar asignación de roles**. Asigne los roles de Azure **Lector, Colaborador o Propietario de datos de Storage Blob** a su servidor de SQL Server.

   > [!NOTE]
   > Solo los miembros con el privilegio Propietario pueden realizar este paso. Para conocer los distintos roles integrados de Azure, consulte esta [guía](../../role-based-access-control/built-in-roles.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).
   
    > [!IMPORTANT]
    > Especifique los roles de Azure **Propietario, Colaborador o Lector** de los **datos de Storage Blob**. Estos roles son diferentes de los roles integrados de Azure de Propietario, Colaborador y Lector. 

    ![Concesión de permiso de RBAC para carga](./media/quickstart-bulk-load-copy-tsql-examples/rbac-load-permissions.png)

4. Ya puede ejecutar la instrucción COPY especificando "Identity administrada":

    ```sql
    COPY INTO dbo.target_table
    FROM 'https://myaccount.blob.core.windows.net/myblobcontainer/folder1/*.txt'
    WITH (
        FILE_TYPE = 'CSV',
        CREDENTIAL = (IDENTITY = 'Managed Identity'),
    )
    ```

## <a name="d-azure-active-directory-authentication-aad"></a>D. Autenticación con Azure Active Directory (AAD)
#### <a name="steps"></a>Pasos

1. En la cuenta de almacenamiento, vaya a **Control de acceso (IAM)** y seleccione **Agregar asignación de roles**. Asigne los roles de Azure **Lector, Colaborador o Propietario de datos de Storage Blob** al usuario de AAD. 

    > [!IMPORTANT]
    > Especifique los roles de Azure **Propietario, Colaborador o Lector** de los **datos de Storage Blob**. Estos roles son diferentes de los roles integrados de Azure de Propietario, Colaborador y Lector.

    ![Concesión de permiso de RBAC para carga](./media/quickstart-bulk-load-copy-tsql-examples/rbac-load-permissions.png)

2. Para configurar la autenticación de Azure AD, consulte la siguiente [documentación](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication-configure?tabs=azure-powershell#create-an-azure-ad-administrator-for-azure-sql-server). 

3. Conéctese a su grupo de SQL mediante Active Directory, donde ahora puede ejecutar la instrucción COPY sin especificar ninguna credencial:

    ```sql
    COPY INTO dbo.target_table
    FROM 'https://myaccount.blob.core.windows.net/myblobcontainer/folder1/*.txt'
    WITH (
        FILE_TYPE = 'CSV'
    )
    ```


## <a name="e-service-principal-authentication"></a>E. Autenticación de la entidad de servicio
#### <a name="steps"></a>Pasos

1. [Creación de una aplicación de Azure Active Directory (AAD)](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#create-an-azure-active-directory-application)
2. [Obtención del identificador de la aplicación](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#get-values-for-signing-in)
3. [Obtención de la clave de autenticación](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#create-a-new-application-secret)
4. [Obtención de la versión V1 del punto de conexión de token de OAuth 2.0](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-service-to-service-authenticate-using-active-directory?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#step-4-get-the-oauth-20-token-endpoint-only-for-java-based-applications)
5. [Asignación de permisos de lectura, escritura y ejecución a una aplicación de AAD](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-service-to-service-authenticate-using-active-directory?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#step-3-assign-the-azure-ad-application-to-the-azure-data-lake-storage-gen1-account-file-or-folder) en la cuenta de almacenamiento
6. Ya puede ejecutar la instrucción COPY:

    ```sql
    COPY INTO dbo.target_table
    FROM 'https://myaccount.blob.core.windows.net/myblobcontainer/folder0/*.txt'
    WITH (
        FILE_TYPE = 'CSV'
        ,CREDENTIAL=(IDENTITY= '<application_ID>@<OAuth_2.0_Token_EndPoint>' , SECRET= '<authentication_key>')
        --CREDENTIAL should look something like this:
        --,CREDENTIAL=(IDENTITY= '92761aac-12a9-4ec3-89b8-7149aef4c35b@https://login.microsoftonline.com/72f714bf-86f1-41af-91ab-2d7cd011db47/oauth2/token', SECRET='juXi12sZ6gse]woKQNgqwSywYv]7A.M')
    )
    ```

> [!IMPORTANT]
>
> - Use la versión **V1** del punto de conexión de token de OAuth 2.0

## <a name="next-steps"></a>Pasos siguientes

- Consulte el artículo sobre la [instrucción COPY](https://docs.microsoft.com/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest#syntax) para ver la sintaxis detallada
- Consulte el artículo con [información general sobre la carga de datos](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/design-elt-data-loading#what-is-elt) para ver los procedimientos recomendados de la carga.
