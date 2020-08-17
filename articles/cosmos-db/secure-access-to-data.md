---
title: Información sobre la protección del acceso a los datos de Azure Cosmos DB
description: Obtenga información sobre los conceptos de control de acceso en Azure Cosmos DB, incluidas las claves maestras, las claves de solo lectura, los usuarios y los permisos.
author: thomasweiss
ms.author: thweiss
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 01/21/2020
ms.openlocfilehash: 3a9039470c32b89d398dd41e3df99e91c70d913c
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87542643"
---
# <a name="secure-access-to-data-in-azure-cosmos-db"></a>Protección del acceso a los datos de Azure Cosmos DB

En este artículo se proporciona información general sobre la protección del acceso a los datos almacenados en [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).

Azure Cosmos DB usa dos tipos de claves para autenticar usuarios y proporcionar acceso a sus datos y recursos. 

|Tipo de clave|Recursos|
|---|---|
|[Claves maestras](#master-keys) |Se utilizan para los recursos administrativos: cuentas de base de datos, bases de datos, usuarios y permisos|
|[Tokens de recursos](#resource-tokens)|Se usan para recursos de aplicaciones: contenedores, documentos, datos adjuntos, procedimientos almacenados, desencadenadores y UDF|

<a id="master-keys"></a>

## <a name="master-keys"></a>Claves maestras

Las claves maestras proporcionan acceso a todos los recursos administrativos de la cuenta de base de datos. Claves maestras:

- Proporcionan acceso a cuentas, bases de datos, usuarios y permisos. 
- No se pueden usar para proporcionar acceso pormenorizado a contenedores y documentos.
- Se crean durante la creación de una cuenta.
- Se pueden regenerar en cualquier momento.

Cada cuenta consta de dos claves maestras: una principal y una secundaria. El fin de las claves dobles es que pueda regenerar, o distribuir claves, lo que proporciona un acceso continuo a su cuenta y sus datos.

Además de las dos claves maestras de la cuenta de Cosmos DB, hay dos claves de solo lectura. Estas claves de solo lectura permiten operaciones de lectura en la cuenta. Las claves de solo lectura no proporcionan acceso a los recursos de los permisos de lectura.

Las claves maestras principal, secundaria, de solo lectura y de lectura y escritura se pueden recuperar y volver a generar desde Azure Portal. Para ver instrucciones al respecto, consulte [Visualización, copia y regeneración de las claves de acceso](manage-with-cli.md#regenerate-account-key).

:::image type="content" source="./media/secure-access-to-data/nosql-database-security-master-key-portal.png" alt-text="Control de acceso (IAM) en Azure Portal: demostración de la seguridad de bases de datos NoSQL":::

### <a name="key-rotation"></a>Rotación de claves<a id="key-rotation"></a>

El proceso de rotación de la clave maestra es simple. 

1. Vaya a Azure Portal para recuperar la clave secundaria.
2. Reemplace la clave principal por su clave secundaria en la aplicación. Asegúrese de que todos los clientes de Cosmos DB en todas las implementaciones se hayan reiniciado rápidamente y de que comenzarán a usar la clave actualizada.
3. Rote la clave principal en Azure Portal.
4. Valide que la nueva clave principal funcione en todos los recursos. El proceso de rotación de claves puede tardar entre menos de un minuto y unas horas, en función del tamaño de la cuenta de Cosmos DB.
5. Reemplace la clave secundaria por la nueva clave principal.

:::image type="content" source="./media/secure-access-to-data/nosql-database-security-master-key-rotate-workflow.png" alt-text="Rotación de la clave maestra en Azure Portal: demostración de la seguridad de bases de datos NoSQL" border="false":::

### <a name="code-sample-to-use-a-master-key"></a>Ejemplo de código para usar una clave maestra

El fragmento de código de ejemplo ilustra cómo usar un punto de conexión y la clave maestra de la cuenta de Cosmos DB para crear una instancia de DocumentClient y una base de datos:

```csharp
//Read the Azure Cosmos DB endpointUrl and authorization keys from config.
//These values are available from the Azure portal on the Azure Cosmos DB account blade under "Keys".
//Keep these values in a safe and secure location. Together they provide Administrative access to your Azure Cosmos DB account.

private static readonly string endpointUrl = ConfigurationManager.AppSettings["EndPointUrl"];
private static readonly string authorizationKey = ConfigurationManager.AppSettings["AuthorizationKey"];

CosmosClient client = new CosmosClient(endpointUrl, authorizationKey);
```

El fragmento de código de ejemplo ilustra cómo usar un punto de conexión y la clave maestra de la cuenta de Azure Cosmos DB para crear una instancia de un objeto `CosmosClient`:

:::code language="python" source="~/cosmosdb-python-sdk/sdk/cosmos/azure-cosmos/samples/access_cosmos_with_resource_token.py" id="configureConnectivity":::

## <a name="resource-tokens"></a>Tokens de recursos <a id="resource-tokens"></a>

Los tokens de recursos proporcionan acceso a los recursos de la aplicación en una base de datos. Los tokens de recursos:

- Proporcionan acceso a contenedores, claves de partición, documentos, datos adjuntos, procedimientos almacenados, desencadenadores y UDF concretos.
- Se crean cuando a un [usuario](#users) se le conceden [permisos](#permissions) para un recurso concreto.
- Se vuelven a crear cuando una llamada POST, GET o PUT aplica una acción a un recurso de permiso.
- Usan un token de recurso de hash construido específicamente para el usuario, el recurso y el permiso.
- Está limitado por un período de validez personalizable. El intervalo de tiempo válido predeterminado es una hora. Sin embargo, la vigencia del token puede especificarse explícitamente hasta un máximo de cinco horas.
- Proporcionan una alternativa segura a proporcionar la clave maestra.
- Permiten que los clientes lean, escriban y eliminen recursos de la cuenta de Cosmos DB en función de los permisos que se les haya otorgado.

Si desea proporcionar acceso a los recursos de su cuenta de Cosmos DB a un cliente que no es de confianza con la clave maestra, puede usar un token de recurso (mediante la creación de usuarios y permisos de Cosmos DB).  

Los tokens de recurso de Cosmos DB ofrecen una alternativa segura que permite a los clientes leer, escribir y eliminar recursos de la cuenta de Cosmos DB en función de los permisos que se les hayan concedido y sin necesidad de una clave maestra o de solo lectura.

Este es un patrón de diseño típico para solicitar, generar y entregar tokens de recursos a los clientes:

1. Se configura un servicio de nivel intermedio para que una aplicación móvil pueda usarlo para compartir fotos del usuario.
2. Dicho servicio tiene la clave maestra de la cuenta de Cosmos DB.
3. La aplicación fotográfica se instala en los dispositivos móviles de los usuarios finales.
4. Al iniciar sesión, dicha aplicación establece la identidad del usuario con el servicio de nivel intermedio. Este mecanismo de establecimiento de identidad depende únicamente de la aplicación.
5. Una vez establecida la identidad, el servicio de nivel intermedio solicita permisos en función de esta.
6. El servicio de nivel intermedio reenvía un token de recurso a la aplicación de teléfono.
7. La aplicación de teléfono puede seguir usando el token de recurso para obtener acceso directo a los recursos de Cosmos DB con los permisos definidos por el token de recurso y para el intervalo permitido por dicho token.
8. Cuando el token de recurso expira, las solicitudes posteriores reciben un mensaje 401 de excepción no autorizada.  En este punto, la aplicación de teléfono restablece la identidad y solicita un nuevo token de recurso.

    :::image type="content" source="./media/secure-access-to-data/resourcekeyworkflow.png" alt-text="Flujo de trabajo de tokens de recursos de Azure Cosmos DB" border="false":::

La generación y administración de los tokens de recursos las controlan las bibliotecas de cliente de Cosmos DB nativas; sin embargo, si se usa REST, debe construir los encabezados de solicitud o autenticación. Para más información sobre cómo crear encabezados de autenticación para REST, consulte [Control de acceso en recursos de Cosmos DB](/rest/api/cosmos-db/access-control-on-cosmosdb-resources) o el código fuente de nuestros [.NET SDK](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos/src/AuthorizationHelper.cs) o [Node.js SDK](https://github.com/Azure/azure-cosmos-js/blob/master/src/auth.ts).

Para ver un ejemplo de un servicio de nivel intermedio que se usa para generarlo o los tokens de recursos del agente, consulte la [aplicación ResourceTokenBroker](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/xamarin/UserItems/ResourceTokenBroker/ResourceTokenBroker/Controllers).

## <a name="users"></a>Usuarios<a id="users"></a>

Los usuarios de Azure Cosmos DB están asociados a una base de datos de Cosmos.  Cada base de datos puede contener cero, o más, usuarios de Cosmos DB. En el siguiente código de ejemplo se muestra cómo crear un usuario de Cosmos DB mediante el [SDK de .NET v3 de Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage/UserManagement).

```csharp
//Create a user.
Database database = benchmark.client.GetDatabase("SalesDatabase");

User user = await database.CreateUserAsync("User 1");
```

> [!NOTE]
> Cada usuario de Cosmos DB tiene un método ReadAsync() que se puede usar para recuperar la lista de [permisos](#permissions) asociados al usuario.

## <a name="permissions"></a>Permisos<a id="permissions"></a>

Un recurso de permiso está asociado a un usuario y está asignado en el contenedor, así como en el nivel de clave de partición. Cada usuario puede contener cero o más permisos. Un recurso de permiso proporciona acceso a un token de seguridad que el usuario necesita al intentar acceder a un contenedor específico o a los datos de una clave de partición específica. Hay dos niveles de acceso disponibles que puede proporcionar un recurso de permiso:

- Todo: el usuario tiene pleno permiso en el recurso.
- Lectura: el usuario solo puede leer el contenido del recurso, pero no realizar operaciones de escritura, actualización o eliminación en este.

> [!NOTE]
> Para ejecutar procedimientos almacenados, el usuario debe tener el permiso "Todo" en el contenedor donde se va a ejecutar el procedimiento almacenado.

### <a name="code-sample-to-create-permission"></a>Ejemplo de código para crear permisos

El código de ejemplo siguiente muestra cómo crear un recurso de permisos, leer el token de dicho recurso y asociar los permisos al [usuario](#users) creado anteriormente.

```csharp
// Create a permission on a container and specific partition key value
Container container = client.GetContainer("SalesDatabase", "OrdersContainer");
user.CreatePermissionAsync(
    new PermissionProperties(
        id: "permissionUser1Orders",
        permissionMode: PermissionMode.All,
        container: benchmark.container,
        resourcePartitionKey: new PartitionKey("012345")));
```

### <a name="code-sample-to-read-permission-for-user"></a>Ejemplo de código para los permisos de lectura del usuario

En el siguiente fragmento de código se muestra cómo recuperar el permiso asociado al usuario creado anteriormente y crear una instancia de un nuevo elemento CosmosClient en nombre del usuario, en el ámbito de una única clave de partición.

```csharp
//Read a permission, create user client session.
PermissionProperties permissionProperties = await user.GetPermission("permissionUser1Orders")

CosmosClient client = new CosmosClient(accountEndpoint: "MyEndpoint", authKeyOrResourceToken: permissionProperties.Token);
```

## <a name="add-users-and-assign-roles"></a>Incorporación de usuarios y asignación de roles

Para agregar acceso de lectura en la cuenta de Azure Cosmos DB a su cuenta de usuario, necesita que un propietario de la suscripción realice los pasos siguientes en Azure Portal.

1. Vaya a Azure Portal y seleccione su cuenta de Azure Cosmos DB.
2. Haga clic en la pestaña **Control de acceso (IAM)** y, después, haga clic en **+ Add role assignment** (+ Agregar asignación de roles).
3. En el panel **Add role assignment** (Agregar asignación de roles), en el cuadro **Rol**, seleccione **Rol de lector de la cuenta de Cosmos DB**.
4. En el cuadro **Asignar acceso a**, seleccione **usuario de Azure AD, grupo o aplicación**.
5. Seleccione el usuario, el grupo o la aplicación en el directorio al que desea conceder acceso.  Puede buscar en el directorio con los nombres para mostrar, dirección de correo electrónico o identificadores de objeto.
    El usuario, el grupo o la aplicación seleccionados aparecen en la lista de los miembros seleccionados.
6. Haga clic en **Save**(Guardar).

La entidad ahora puede leer los recursos de Azure Cosmos DB.

## <a name="delete-or-export-user-data"></a>Eliminación o exportación de datos de usuario

Azure Cosmos DB le permite buscar, seleccionar, modificar y eliminar cualquier dato personal ubicado en la base de datos o en colecciones. Azure Cosmos DB proporciona API para buscar y eliminar datos personales; sin embargo, es su responsabilidad usar las API y definir la lógica requerida para borrar los datos personales. Cada API multimodelo (SQL, MongoDB, Gremlin, Cassandra, Table) proporciona SDK de diferentes lenguajes que contienen métodos para buscar y eliminar datos personales. También puede habilitar la característica [período de vida (TTL)](time-to-live.md) para eliminar datos automáticamente después de un período especificado, sin incurrir en ningún costo adicional.

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre la seguridad de las bases de datos de Cosmos, consulte [Cosmos DB: seguridad de bases de datos](database-security.md).
- Para aprender a construir tokens de autorización de Azure Cosmos DB, consulte [Access Control on Azure Cosmos DB Resources](/rest/api/cosmos-db/access-control-on-cosmosdb-resources) (Control de acceso en recursos de Azure Cosmos DB).
- Ejemplos de administración de usuarios con usuarios y permisos, [ejemplos de administración de usuarios del SDK de .NET v3](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/UserManagement/UserManagementProgram.cs)
