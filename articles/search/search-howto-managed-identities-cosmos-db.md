---
title: Configuración de una conexión a una cuenta de Cosmos DB mediante una identidad administrada (versión preliminar)
titleSuffix: Azure Cognitive Search
description: Aprenda a configurar una conexión de indexador a una cuenta de Cosmos DB mediante una identidad administrada (versión preliminar)
manager: luisca
author: markheff
ms.author: maheff
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 05/18/2020
ms.openlocfilehash: df37b7f1c5b1ed35b6c3779eea470b2fb0936ecf
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88936663"
---
# <a name="set-up-an-indexer-connection-to-a-cosmos-db-database-using-a-managed-identity-preview"></a>Configuración de una conexión de indexador a una base de datos de Cosmos DB mediante una identidad administrada (versión preliminar)

> [!IMPORTANT] 
> La compatibilidad con la configuración de una conexión en un origen de datos mediante una identidad administrada se encuentra actualmente en versión preliminar pública. La funcionalidad de versión preliminar se ofrece sin un Acuerdo de Nivel de Servicio y no es aconsejable usarla para cargas de trabajo de producción.

En esta página se describe cómo configurar una conexión de indexador a una base de datos de Azure Cosmos DB mediante una identidad administrada en lugar de proporcionar credenciales en la cadena de conexión del objeto de origen de datos.

Antes de obtener más información acerca de esta característica, se recomienda comprender qué es un indexador y cómo configurar un indexador para el origen de datos. Se puede encontrar más información en los vínculos siguientes:
* [Información general del indexador](search-indexer-overview.md)
* [Indexador de Azure Cosmos DB](search-howto-index-cosmosdb.md)

## <a name="set-up-a-connection-using-a-managed-identity"></a>Configuración de una conexión mediante una identidad administrada

### <a name="1---turn-on-system-assigned-managed-identity"></a>1 - Activar la identidad administrada asignada por el sistema

Cuando se habilita una identidad administrada asignada por el sistema, Azure crea una identidad para el servicio de búsqueda que se puede usar para autenticarse en otros servicios de Azure en el mismo inquilino y la misma suscripción. Después, puede usar esta identidad en las asignaciones de control de acceso basado en rol (RBAC) que permiten el acceso a los datos durante la indexación.

![Activar la identidad administrada asignada por el sistema](./media/search-managed-identities/turn-on-system-assigned-identity.png "Activar la identidad administrada asignada por el sistema")

Después de seleccionar **Guardar**, verá un identificador de objeto que se ha asignado al servicio de búsqueda.

![Id. de objeto](./media/search-managed-identities/system-assigned-identity-object-id.png "Id. de objeto")
 
### <a name="2---add-a-role-assignment"></a>2 - Agregar una asignación de roles

En este paso, concederá permiso a su servicio de Azure Cognitive Search para leer los datos de la base de datos de Cosmos DB.

1. En Azure Portal, navegue hasta la cuenta de Cosmos DB que contiene los datos que quiere indexar.
2. Seleccione **Access Control (IAM)**
3. Seleccione **Agregar** y, a continuación, **Agregar asignación de roles**.

    ![Agregar asignación de roles](./media/search-managed-identities/add-role-assignment-cosmos-db.png "Agregar asignación de roles")

4. Seleccione el **Rol de lector de cuentas de Cosmos DB**.
5. Deje **Asignar acceso a** como **Usuario, grupo o entidad de servicio de Azure AD**.
6. Busque el servicio de búsqueda, selecciónelo y elija **Guardar**.

    ![Agregue la asignación del rol Lector y acceso a datos](./media/search-managed-identities/add-role-assignment-cosmos-db-account-reader-role.png "Agregar la asignación del rol Lector y acceso a datos").

### <a name="3---create-the-data-source"></a>3 - Crear el origen de datos

La [API REST](/rest/api/searchservice/create-data-source), Azure Portal y el [SDK de .NET](/dotnet/api/microsoft.azure.search.models.datasource?view=azure-dotnet) admiten la cadena de conexión de identidad administrada. A continuación se muestra un ejemplo de cómo crear un origen de datos para indexar los datos de Cosmos DB mediante la [API REST](/rest/api/searchservice/create-data-source) y una cadena de conexión de identidad administrada. El formato de la cadena de conexión de identidad administrada es el mismo para la API REST, el SDK de .NET y Azure Portal.

Si se utilizan identidades administradas para autenticarse, las **credenciales** no incluirán una clave de cuenta.

```
POST https://[service name].search.windows.net/datasources?api-version=2020-06-30
Content-Type: application/json
api-key: [Search service admin key]

{
    "name": "cosmos-db-datasource",
    "type": "cosmosdb",
    "credentials": {
        "connectionString": "Database=sql-test-db;ResourceId=/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/cosmos-db-resource-group/providers/Microsoft.DocumentDB/databaseAccounts/my-cosmos-db-account/;"
    },
    "container": { "name": "myCollection", "query": null },
    "dataChangeDetectionPolicy": {
        "@odata.type": "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
        "highWaterMarkColumnName": "_ts"
    }
}
```    

El cuerpo de la solicitud contiene la definición del origen de datos, que debe incluir los siguientes campos:

| Campo   | Descripción |
|---------|-------------|
| **name** | Necesario. Elija un nombre para representar el objeto de origen de datos. |
|**type**| Necesario. Debe ser `cosmosdb`. |
|**credentials** | Necesario. <br/><br/>Al conectarse mediante una identidad administrada, el formato de las **credenciales** debe ser: *Database=[nombre-base-de-datos];ResourceId=[cadena-id-recurso];(ApiKind=[clase-api];)*<br/> <br/>Formato de ResourceId: *ResourceId=/subscriptions/**id. de la suscripción**/resourceGroups/**nombre del grupo de recursos**/providers/Microsoft.DocumentDB/databaseAccounts/**nombre de la cuenta de Cosmos DB**/;*<br/><br/>En las colecciones de SQL, la cadena de conexión no requiere un elemento ApiKind.<br/><br/>Para las colecciones de MongoDB, agregue **ApiKind=MongoDb** a la cadena de conexión. <br/><br/>En el caso de los gráficos de Gremlin y las tablas de Cassandra, regístrese para la [versión preliminar del indexador validada](https://aka.ms/azure-cognitive-search/indexer-preview) para obtener acceso a la versión preliminar e información sobre cómo dar formato a las credenciales.<br/>|
| **container** | Contiene los siguientes elementos: <br/>**name**: Necesario. Especifique el identificador de la colección de la base de datos que se va a indexar.<br/>**query**: Opcional. Puede especificar una consulta para acoplar un documento JSON arbitrario en un esquema plano que Azure Cognitive Search pueda indizar.<br/>En MongoDB API, Gremlin API y Cassandra API, no se admiten consultas. |
| **dataChangeDetectionPolicy** | Recomendado |
|**dataDeletionDetectionPolicy** | Opcional |

### <a name="4---create-the-index"></a>4 - Crear el índice

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

### <a name="5---create-the-indexer"></a>5 - Crear el indexador

Un indexador conecta un origen de datos con un índice de búsqueda de destino y proporciona una programación para automatizar la actualización de datos.

Una vez creados el índice y el origen de datos, ya podrá crear el indexador.

Ejemplo de definición del indexador:

```http
    POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "cosmos-db-indexer",
      "dataSourceName" : "cosmos-db-datasource",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" }
    }
```

Este indizador se ejecutará cada dos horas (el intervalo de programación se establece en PT2H). Para ejecutar un indizador cada 30 minutos, establézcalo en PT30M. El intervalo más breve que se admite es de 5 minutos. La programación es opcional: si se omite, el indizador solo se ejecuta una vez cuando se crea. Sin embargo, puede ejecutarlo a petición en cualquier momento.   

Para más información sobre la API Create Indexer, consulte [Crear indexador](/rest/api/searchservice/create-indexer).

Para más información sobre cómo definir las programaciones del indexador, consulte [Programación de indexadores para Azure Cognitive Search](search-howto-schedule-indexers.md).

## <a name="see-also"></a>Consulte también

Más información sobre los indexadores de Cosmos DB:
* [Indexador de Azure Cosmos DB](search-howto-index-cosmosdb.md)