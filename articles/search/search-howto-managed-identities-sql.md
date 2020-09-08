---
title: Configuración de una conexión a Azure SQL Database mediante una identidad administrada (versión preliminar)
titleSuffix: Azure Cognitive Search
description: Aprenda a configurar una conexión de indexador a Azure SQL Database mediante una identidad administrada (versión preliminar)
manager: luisca
author: markheff
ms.author: maheff
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 05/18/2020
ms.openlocfilehash: 8dabf69af8628bb0b168bfea94af5333df341423
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88924136"
---
# <a name="set-up-an-indexer-connection-to-azure-sql-database-using-a-managed-identity-preview"></a>Configuración de una conexión de indexador a Azure SQL Database mediante una identidad administrada (versión preliminar)

> [!IMPORTANT] 
> La compatibilidad con la configuración de una conexión en un origen de datos mediante una identidad administrada se encuentra actualmente en versión preliminar pública. La funcionalidad de versión preliminar se ofrece sin un Acuerdo de Nivel de Servicio y no es aconsejable usarla para cargas de trabajo de producción.

Esta página describe cómo configurar una conexión de indexador a Azure SQL Database mediante una identidad administrada en lugar de proporcionar credenciales en la cadena de conexión del objeto de origen de datos.

Antes de obtener más información acerca de esta característica, se recomienda que se familiarice con lo que es un indexador y cómo configurar uno para el origen de datos. Se puede encontrar más información en los vínculos siguientes:

* [Información general del indexador](search-indexer-overview.md)
* [Indexador de Azure SQL](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md)

## <a name="set-up-a-connection-using-a-managed-identity"></a>Configuración de una conexión mediante una identidad administrada

### <a name="1---turn-on-system-assigned-managed-identity"></a>1 - Activar la identidad administrada asignada por el sistema

Cuando se habilita una identidad administrada asignada por el sistema, Azure crea una identidad para el servicio de búsqueda que se puede usar para autenticarse en otros servicios de Azure en el mismo inquilino y la misma suscripción. Después, puede usar esta identidad en las asignaciones de control de acceso basado en rol (RBAC) que permiten el acceso a los datos durante la indexación.

![Activar la identidad administrada asignada por el sistema](./media/search-managed-identities/turn-on-system-assigned-identity.png "Activación de la identidad administrada asignada por el sistema")

Después de seleccionar **Guardar**, verá un identificador de objeto que se ha asignado al servicio de búsqueda.

![Id. de objeto](./media/search-managed-identities/system-assigned-identity-object-id.png "Id. de objeto")

### <a name="2---provision-azure-active-directory-admin-for-sql-server"></a>2\. Aprovisionamiento del administrador de Azure Active Directory para SQL Server

Cuando se conecte a la base de datos en el paso siguiente, tendrá que conectarse a una cuenta Azure Active Directory (Azure AD) que tenga acceso de administrador a la base de datos para que el servicio de búsqueda tenga permiso de acceso a la base de datos.

Siga las instrucciones disponibles [aquí](../azure-sql/database/authentication-aad-configure.md?tabs=azure-powershell#provision-azure-ad-admin-sql-database) para conceder al administrador de la cuenta de Azure AD acceso a la base de datos.

### <a name="3---assign-the-search-service-permissions"></a>3\. Asignación de permisos del servicio de búsqueda

Siga los pasos que se indican a continuación para asignar el permiso de servicio de búsqueda necesario para leer la base de datos:

1. Conexión a Visual Studio

    ![Conexión a Visual Studio](./media/search-managed-identities/connect-with-visual-studio.png "Conexión a Visual Studio")

2. Autenticación con la cuenta de Azure AD

    ![Autenticar](./media/search-managed-identities/visual-studio-authentication.png "Authenticate")

3. Ejecute los siguientes comandos:

    Incluya el nombre del servicio de búsqueda entre corchetes.
    
    ```
    CREATE USER [your search service name here] FROM EXTERNAL PROVIDER;
    EXEC sp_addrolemember 'db_datareader', [your search service name here];
    ```

    ![Nueva consulta](./media/search-managed-identities/visual-studio-new-query.png "Nueva consulta")

    ![Ejecutar consulta](./media/search-managed-identities/visual-studio-execute-query.png "Ejecutar consulta")

>[!NOTE]
> Si se cambia la identidad del servicio de búsqueda del paso 1 después de completar ese paso, tiene que quitar la pertenencia a rol y el usuario de la base de datos SQL. A continuación, vuelva a agregar los permisos realizando el paso 3.
> Para eliminar la pertenencia al rol y el usuario, ejecute los siguientes comandos:
> ```
> sp_droprolemember 'db_datareader', [your search service name];
> DROP USER IF EXISTS [your search service name];
> ```

### <a name="4---add-a-role-assignment"></a>4\. Adición de una asignación de roles

En este paso, concederá permiso a su servicio de Azure Cognitive Search para leer los datos de SQL Server.

1. En Azure Portal, vaya a la página de Azure SQL Server.
2. Seleccione **Access Control (IAM)**
3. Seleccione **Agregar** y, a continuación, **Agregar asignación de roles**.

    ![Agregar asignación de roles](./media/search-managed-identities/add-role-assignment-sql-server.png "Agregar asignación de roles")

4. Seleccione el rol de **lector** adecuado.
5. Deje **Asignar acceso a** como **Usuario, grupo o entidad de servicio de Azure AD**.
6. Busque el servicio de búsqueda, selecciónelo y elija **Guardar**.

    ![Adición de una asignación de roles](./media/search-managed-identities/add-role-assignment-sql-server-reader-role.png "Adición de una asignación de roles")

### <a name="5---create-the-data-source"></a>5\. Creación del origen de datos

La [API REST](/rest/api/searchservice/create-data-source), Azure Portal y el [SDK de .NET](/dotnet/api/microsoft.azure.search.models.datasource?view=azure-dotnet) admiten la cadena de conexión de identidad administrada. A continuación se muestra un ejemplo de cómo crear un origen de datos para indexar los datos de la base de datos de Azure SQL mediante la [API REST](/rest/api/searchservice/create-data-source) y una cadena de conexión de identidad administrada. El formato de la cadena de conexión de identidad administrada es el mismo para la API REST, el SDK de .NET y Azure Portal.

Al crear un origen de datos mediante la [API de REST](/rest/api/searchservice/create-data-source), el origen de datos debe tener las siguientes propiedades obligatorias:

* **name** es el nombre único del origen de datos dentro del servicio de búsqueda.
* **type** es `azuresql`.
* **credentials**
    * Cuando se usa una identidad administrada con fines de autenticación, el formato de **credentials** es diferente a cuando no se usa la identidad administrada. Aquí proporcionará un nombre de catálogo o base de datos original y un ResourceId que no tenga ninguna clave o contraseña de cuenta. ResourceId debe incluir el identificador de suscripción de Azure SQL Database, así como el grupo de recursos de SQL Database y el nombre de la base de datos SQL. 
    * Formato de la cadena de conexión de identidad administrada:
        * *Initial Catalog|Database=**nombre de la base de datos**;ResourceId=/subscriptions/**Id. de la suscripción**/resourceGroups/**nombre del grupo de recursos**/providers/Microsoft.Sql/servers/**nombre de SQL Server**/Connection Timeout=**duración del tiempo de espera de conexión**;*
* **container** indica el nombre de la tabla o la vista que desea indexar.

Ejemplo de cómo crear un objeto de origen de datos de Azure SQL mediante la [API de REST](/rest/api/searchservice/create-data-source):

```
POST https://[service name].search.windows.net/datasources?api-version=2020-06-30
Content-Type: application/json
api-key: [admin key]

{
    "name" : "sql-datasource",
    "type" : "azuresql",
    "credentials" : { "connectionString" : "Database=sql-database;ResourceId=/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azure-sql-resource-group/providers/Microsoft.Sql/servers/sql-server-search-demo;Connection Timeout=30;" },
    "container" : { "name" : "my-table" }
} 
```

### <a name="6---create-the-index"></a>6\. Creación del índice

El índice especifica los campos de un documento, los atributos y otras construcciones que conforman la experiencia de búsqueda.

Aquí se muestra cómo crear un índice con un campo `booktitle` que admite búsquedas:   

```
POST https://[service name].search.windows.net/indexes?api-version=2020-06-30
Content-Type: application/json
api-key: [admin key]

{
    "name" : "my-target-index",
    "fields": [
    { "name": "id", "type": "Edm.String", "key": true, "searchable": false },
    { "name": "booktitle", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false }
    ]
}
```

Para obtener más información sobre la creación de índices, vea el artículo de [creación de índices](/rest/api/searchservice/create-index).

### <a name="7---create-the-indexer"></a>7\. Creación del indexador

Un indizador conecta un origen de datos con un índice de búsqueda de destino y proporciona una programación para automatizar la actualización de datos.

Una vez creados el origen de datos y los índices, podrá crear el indexador:

Ejemplo de definición del indexador para un indexador de Azure SQL:

```
POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
Content-Type: application/json
api-key: [admin key]

{
    "name" : "sql-indexer",
    "dataSourceName" : "sql-datasource",
    "targetIndexName" : "my-target-index",
    "schedule" : { "interval" : "PT2H" }
}
```    

Este indizador se ejecutará cada dos horas (el intervalo de programación se establece en PT2H). Para ejecutar un indizador cada 30 minutos, establézcalo en PT30M. El intervalo más breve que se admite es de 5 minutos. La programación es opcional: si se omite, el indizador solo se ejecuta una vez cuando se crea. Sin embargo, puede ejecutarlo a petición en cualquier momento.   

Para más información sobre la API Create Indexer, consulte [Crear indexador](/rest/api/searchservice/create-indexer).

Para más información sobre cómo definir las programaciones del indexador, consulte [Programación de indexadores para Azure Cognitive Search](search-howto-schedule-indexers.md).

## <a name="troubleshooting"></a>Solución de problemas

Si recibe un error cuando el indexador intenta conectarse al origen de datos que indica que el cliente no tiene permiso para acceder al servidor, consulte los [errores comunes del indexador](./search-indexer-troubleshooting.md).

## <a name="see-also"></a>Consulte también

Más información acerca del indexar de Azure SQL:
* [Indexador de Azure SQL](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md)