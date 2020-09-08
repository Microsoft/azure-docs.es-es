---
title: Filtros de seguridad para limitar los resultados mediante Active Directory
titleSuffix: Azure Cognitive Search
description: Privilegios de seguridad en el nivel de documento de los resultados de búsqueda de Azure Cognitive Search, mediante filtros de seguridad y las identidades de Azure Active Directory (AAD).
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/04/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 87337cf22bdb388c5873a2811bb9913c3e7f4d4e
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89019783"
---
# <a name="security-filters-for-trimming-azure-cognitive-search-results-using-active-directory-identities"></a>Filtros de seguridad para limitar los resultados de Azure Cognitive Search mediante las identidades de Active Directory

En este artículo se muestra cómo usar las identidades de seguridad de Azure Active Directory (AAD) junto con los filtros de Azure Cognitive Search para limitar los resultados de búsqueda en función de la pertenencia a grupos de usuarios.

En este artículo se tratan las tareas siguientes:
> [!div class="checklist"]
> - Creación de usuarios y grupos de AAD
> - Asociación del usuario con el grupo que ha creado
> - Almacenamiento en caché de nuevos grupos
> - Indexación de documentos con grupos asociados
> - Emisión de una solicitud de búsqueda con filtro de identificadores de grupo
> 
> [!NOTE]
> Los fragmentos de código de ejemplo de este artículo están escritos en C#. El código fuente completo se puede encontrar [en GitHub](https://github.com/Azure-Samples/search-dotnet-getting-started). 

## <a name="prerequisites"></a>Requisitos previos

El índice de Azure Cognitive Search debe tener un [campo de seguridad](search-security-trimming-for-azure-search.md) para almacenar la lista de identidades de grupo que tengan acceso de lectura al documento. Este caso de uso supone una correspondencia uno a uno entre un elemento protegible (por ejemplo, la solicitud de la universidad de un individuo) y un campo de seguridad que especifica quién tiene acceso a ese elemento (personal de admisión).

Debe tener permisos de administrador de AAD, necesarios en este tutorial para crear usuarios, grupos y asociaciones en AAD.

La solicitud debe registrarse con AAD, tal como se describe en el siguiente procedimiento.

### <a name="register-your-application-with-aad"></a>Registro de la solicitud con AAD

Este paso integra la solicitud con AAD con el fin de aceptar inicios de sesión de cuentas de usuario y grupo. Si no es un administrador de AAD en su organización, podría tener que [crear un nuevo inquilino](../active-directory/develop/quickstart-create-new-tenant.md) para realizar los pasos siguientes.

1. Vaya al [**Portal de registro de aplicaciones**](https://apps.dev.microsoft.com) >  **Converged app (Aplicación convergida)**  > **Agregar una aplicación**.
2. Escriba un nombre para la solicitud y haga clic en **Crear**. 
3. Seleccione la solicitud recién registrada en la página Mis aplicaciones.
4. En la página de registro de la solicitud > **Plataformas** > **Agregar plataforma**, elija **API web**.
5. Todavía en la página de registro de la solicitud, vaya a > **Permisos para Microsoft Graph** > **Agregar**.
6. En Seleccionar permisos, agregue los siguientes permisos delegados y haga clic en **Aceptar**:

   + **Directory.ReadWrite.All**
   + **Group.ReadWrite.All**
   + **User.ReadWrite.All**

Microsoft Graph proporciona una API que permite el acceso mediante programación a AAD a través de una API de REST. El ejemplo de código para este tutorial usa los permisos para llamar a Microsoft Graph API para la creación de grupos, usuarios y asociaciones. Las API también se usan para almacenar en caché identificadores de grupo de caché para filtrar con mayor rapidez.

## <a name="create-users-and-groups"></a>Creación de usuarios y grupos

Si va a agregar la búsqueda a una solicitud establecida, podría tener identificadores de grupo y usuarios existentes en AAD. En este caso, puede omitir los tres pasos siguientes. 

Sin embargo, si no tiene usuarios existentes, puede utilizar Microsoft Graph API para crear las entidades de seguridad. Los siguientes fragmentos de código muestran cómo generar identificadores, que se convierten en valores de datos para el campo de seguridad en el índice de Azure Cognitive Search. En nuestra hipotética solicitud de admisión de la universidad, esto serían los identificadores de seguridad para el personal de admisión.

La pertenencia de usuarios y grupos podría ser muy fluida, especialmente en grandes organizaciones. El código que crea identidades de usuario y grupo debe ejecutarse con la frecuencia suficiente como para recoger los cambios en la pertenencia de la organización. Del mismo modo, el índice de Azure Cognitive Search requiere una programación de actualización similar para reflejar el estado actual de recursos y usuarios permitidos.

### <a name="step-1-create-aad-group"></a>Paso 1: Crear un [grupo de AAD](/graph/api/group-post-groups?view=graph-rest-1.0) 
```csharp
// Instantiate graph client 
GraphServiceClient graph = new GraphServiceClient(new DelegateAuthenticationProvider(...));
Group group = new Group()
{
    DisplayName = "My First Prog Group",
    SecurityEnabled = true,
    MailEnabled = false,
    MailNickname = "group1"
}; 
Group newGroup = await graph.Groups.Request().AddAsync(group);
```
   
### <a name="step-2-create-aad-user"></a>Paso 2: Crear un [usuario de AAD](/graph/api/user-post-users?view=graph-rest-1.0)
```csharp
User user = new User()
{
    GivenName = "First User",
    Surname = "User1",
    MailNickname = "User1",
    DisplayName = "First User",
    UserPrincipalName = "User1@FirstUser.com",
    PasswordProfile = new PasswordProfile() { Password = "********" },
    AccountEnabled = true
};
User newUser = await graph.Users.Request().AddAsync(user);
```

### <a name="step-3-associate-user-and-group"></a>Paso 3: Asociar el usuario y el grupo
```csharp
await graph.Groups[newGroup.Id].Members.References.Request().AddAsync(newUser);
```

### <a name="step-4-cache-the-groups-identifiers"></a>Paso 4: Almacenar los identificadores de grupos en caché
Si lo desea, para reducir la latencia de red, puede almacenar en caché las asociaciones entre usuarios y grupos para que cuando se emita una solicitud de búsqueda, los grupos se devuelven desde la memoria caché, ahorrando un ida y vuelta a AAD. Puede usar la [API de Batch de AAD](/graph/json-batching) para enviar una solicitud HTTP única con varios usuarios y crear la caché.

Microsoft Graph se ha diseñado para controlar un alto volumen de solicitudes. Si se produce un número excesivo de solicitudes, Microsoft Graph produce un error en la solicitud con el código de estado HTTP 429. Para más información, consulte [Guía de limitación de Microsoft Graph](/graph/throttling).

## <a name="index-document-with-their-permitted-groups"></a>Documento de índice con sus grupos permitidos

Las operaciones de consulta en Azure Cognitive Search se ejecutan sobre un índice de Azure Cognitive Search. En este paso, una operación de indexación importa datos que permiten realizar búsquedas en un índice, incluidos los identificadores usados como filtros de seguridad. 

Azure Cognitive Search no autentica identidades de usuario ni proporciona lógica para establecer el contenido que un usuario tiene permiso para ver. El caso de uso para el recorte de seguridad asume que se proporciona la asociación entre un documento confidencial y el identificador del grupo que tiene acceso a ese documento, que se ha importado intacto en un índice de búsqueda. 

En el ejemplo hipotético, el cuerpo de la solicitud PUT en un índice de Azure Cognitive Search incluiría el trabajo o la transcripción de la universidad de un solicitante junto con el identificador de grupo que tiene permiso para ver el contenido. 

En el ejemplo genérico utilizado en el ejemplo de código para este tutorial, la acción del índice podría ser la siguiente:

```csharp
var actions = new IndexAction<SecuredFiles>[]
              {
                  IndexAction.Upload(
                  new SecuredFiles()
                  {
                      FileId = "1",
                      Name = "secured_file_a",
                      GroupIds = new[] { groups[0] }
                  }),
              ...
             };

var batch = IndexBatch.New(actions);

_indexClient.Documents.Index(batch);  
```

## <a name="issue-a-search-request"></a>Emisión de una solicitud de búsqueda

Por motivos de recorte de seguridad, los valores del campo de seguridad en el índice son valores estáticos que se usan para incluir o excluir documentos en los resultados de la búsqueda. Por ejemplo, si el identificador de grupo de admisiones es "A11B22C33D44-E55F66G77-H88I99JKK", todos los documentos de un índice de Azure Cognitive Search que tengan ese identificador en el campo de seguridad se incluyen (o excluyen) en los resultados de búsqueda devueltos al solicitante.

Para filtrar los documentos devueltos en los resultados de búsqueda en función de los grupos de usuario que emiten la solicitud, revise los siguientes pasos.

### <a name="step-1-retrieve-users-group-identifiers"></a>Paso 1: Recuperar identificadores de grupo del usuario

Si los grupos del usuario todavía no estaban almacenados en caché o la memoria caché ha expirado, emita la solicitud de [grupos](/graph/api/directoryobject-getmembergroups?view=graph-rest-1.0).
```csharp
private static void RefreshCacheIfRequired(string user)
{
    if (!_groupsCache.ContainsKey(user))
    {
        var groups = GetGroupIdsForUser(user).Result;
        _groupsCache[user] = groups;
    }
}

private static async Task<List<string>> GetGroupIdsForUser(string userPrincipalName)
{
    List<string> groups = new List<string>();
    var allUserGroupsRequest = graph.Users[userPrincipalName].GetMemberGroups(true).Request();

    while (allUserGroupsRequest != null) 
    {
        IDirectoryObjectGetMemberGroupsRequestBuilder allUserGroups = await allUserGroupsRequest.PostAsync();
        groups = allUserGroups.ToList();
        allUserGroupsRequest = allUserGroups.NextPageRequest;
    }
    return groups;
}
``` 

### <a name="step-2-compose-the-search-request"></a>Paso 2: Crear la solicitud de búsqueda

Suponiendo que tiene pertenencia a grupos del usuario, puede emitir la solicitud de búsqueda con los valores de filtro apropiados.

```csharp
string filter = String.Format("groupIds/any(p:search.in(p, '{0}'))", string.Join(",", groups.Select(g => g.ToString())));
SearchParameters parameters = new SearchParameters()
             {
                 Filter = filter,
                 Select = new[] { "application essays" }
             };

DocumentSearchResult<SecuredFiles> results = _indexClient.Documents.Search<SecuredFiles>("*", parameters);
```
### <a name="step-3-handle-the-results"></a>Paso 3: Controlar los resultados

La respuesta incluye una lista filtrada de documentos, que consta de los que el usuario tiene permiso para ver. Dependiendo de cómo se cree la página de resultados de búsqueda, puede que desee incluir indicaciones visuales para reflejar el conjunto de resultados filtrados.

## <a name="conclusion"></a>Conclusión

En este tutorial, ha aprendido técnicas para usar inicios de sesión de AAD para filtrar documentos en los resultados de Azure Cognitive Search, recortando los resultados de los documentos que no coinciden con el filtro proporcionado en la solicitud.

## <a name="see-also"></a>Consulte también

+ [Control de acceso basado en identidades mediante filtros de Azure Cognitive Search](search-security-trimming-for-azure-search.md)
+ [Filtros de Azure Cognitive Search](search-filters.md)
+ [Control de acceso y seguridad de datos en las operaciones de Azure Cognitive Search](search-security-overview.md)