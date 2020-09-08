---
title: Uso de tokens de recursos de Azure Cosmos DB con el SDK de Gremlin
description: Aprenda a crear tokens de recursos y a usarlos para acceder a la base de datos de grafos.
author: luisbosquez
ms.author: lbosq
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: how-to
ms.date: 09/06/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: bb2f948a49badf2578957b137d185c26607923b7
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88997207"
---
# <a name="use-azure-cosmos-db-resource-tokens-with-the-gremlin-sdk"></a>Uso de tokens de recursos de Azure Cosmos DB con el SDK de Gremlin

En este artículo se explica cómo usar los [tokens de recursos de Azure Cosmos DB](secure-access-to-data.md) para acceder a la base de datos de grafos mediante el SDK de Gremlin.

## <a name="create-a-resource-token"></a>Creación de un token de recursos

El SDK de Apache TinkerPop no tiene una API para crear tokens de recursos. El término *token de recurso* es un concepto de Azure Cosmos DB. Para crear tokens de recursos, descargue el [SDK de Azure Cosmos DB](sql-api-sdk-dotnet.md). Si la aplicación necesita crear tokens de recursos y usarlos para tener acceso a la base de datos de grafos, necesita dos SDK independientes.

La jerarquía del modelo de objetos sobre los tokens de recursos se muestra en el esquema siguiente:

- **Cuenta de Azure Cosmos DB**: entidad de nivel superior con DNS asociado con él (por ejemplo, `contoso.gremlin.cosmos.azure.com`)
  - **Base de datos de Azure Cosmos DB**
    - **User**
      - **Permiso**
        - **Token**: una propiedad del objeto Permission que denota las acciones se permiten o se deniegan

Un token de recursos utiliza el formato siguiente: `"type=resource&ver=1&sig=<base64 string>;<base64 string>;"`. Esta cadena es opaca para los clientes y debe usarse tal cual, sin modificación ni interpretación.

```csharp
// Notice that document client is created against .NET SDK endpoint, rather than Gremlin.
DocumentClient client = new DocumentClient(
  new Uri("https://contoso.documents.azure.com:443/"), 
  "<master key>", 
  new ConnectionPolicy 
  {
    EnableEndpointDiscovery = false, 
    ConnectionMode = ConnectionMode.Direct 
  });

  // Read specific permission to obtain a token.
  // The token isn't returned during the ReadPermissionReedAsync() call.
  // The call succeeds only if database id, user id, and permission id already exist. 
  // Note that <database id> is not a database name. It is a base64 string that represents the database identifier, for example "KalVAA==".
  // Similar comment applies to <user id> and <permission id>.
  Permission permission = await client.ReadPermissionAsync(UriFactory.CreatePermissionUri("<database id>", "<user id>", "<permission id>"));

  Console.WriteLine("Obtained token {0}", permission.Token);
}
```

## <a name="use-a-resource-token"></a>Uso de un token de recursos
Puede usar tokens de recursos directamente como una propiedad de "contraseña" al construir la clase GremlinServer.

```csharp
// The Gremlin application needs to be given a resource token. It can't discover the token on its own.
// You can obtain the token for a given permission by using the Azure Cosmos DB SDK, or you can pass it into the application as a command line argument or configuration value.
string resourceToken = GetResourceToken();

// Configure the Gremlin server to use a resource token rather than a master key.
GremlinServer server = new GremlinServer(
  "contoso.gremlin.cosmosdb.azure.com",
  port: 443,
  enableSsl: true,
  username: "/dbs/<database name>/colls/<collection name>",

  // The format of the token is "type=resource&ver=1&sig=<base64 string>;<base64 string>;".
  password: resourceToken);

  using (GremlinClient gremlinClient = new GremlinClient(server, new GraphSON2Reader(), new GraphSON2Writer(), GremlinClient.GraphSON2MimeType))
  {
      await gremlinClient.SubmitAsync("g.V().limit(1)");
  }
```

La misma estrategia funciona en todos los SDK de TinkerPop Gremlin.

```java
Cluster.Builder builder = Cluster.build();

AuthProperties authenticationProperties = new AuthProperties();
authenticationProperties.with(AuthProperties.Property.USERNAME,
    String.format("/dbs/%s/colls/%s", "<database name>", "<collection name>"));

// The format of the token is "type=resource&ver=1&sig=<base64 string>;<base64 string>;".
authenticationProperties.with(AuthProperties.Property.PASSWORD, resourceToken);

builder.authProperties(authenticationProperties);
```

## <a name="limit"></a>Límite

Con una sola cuenta de Gremlin, puede emitir un número ilimitado de tokens. Sin embargo, solo puede usar hasta 100 tokens simultáneamente en un plazo de una hora. Si una aplicación supera el límite de tokens por hora, se deniega una solicitud de autenticación y recibe el mensaje de error que indica que se ha superado el límite de tokens de recursos permitido de 100 que se puede usar simultáneamente. No funciona si se cierran las conexiones activas que utilizan tokens específicos para liberar ranuras para tokens nuevos. El motor de base de datos de Gremlin para Azure Cosmos DB realiza un seguimiento de los tokens únicos durante la hora inmediatamente anterior a la solicitud de autenticación.

## <a name="permission"></a>Permiso

Un error común que las aplicaciones encuentran mientras usan tokens de recursos es que hay permisos insuficientes en el encabezado de autorización para la solicitud correspondiente. Se recomienda volver a intentarlo con otro encabezado de autorización. Este error se devuelve cuando el recorrido a través de Gremlin intenta escribir un borde o un vértice, pero el token de recurso solo concede permisos de *lectura*. Inspeccione el recorrido por si contiene alguno de los siguientes pasos: *.addV()* , *.addE()* , *.drop()* o *.property()* .

## <a name="next-steps"></a>Pasos siguientes
* [Control de acceso basado en roles de Azure (RBAC de Azure)](role-based-access-control.md) en Azure Cosmos DB
* [Información sobre la protección del acceso a los datos](secure-access-to-data.md) en Azure Cosmos DB
