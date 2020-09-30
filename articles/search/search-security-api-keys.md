---
title: Creación, administración y protección de las claves de API de consulta y administración
titleSuffix: Azure Cognitive Search
description: Las claves de API controlan el acceso al punto de conexión de servicio. Las claves de administración conceden acceso de escritura. Las claves de consulta se pueden crear para el acceso de solo lectura.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 1fa464eca69f50fba7fa125212b85e0f7f63bcc9
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89294788"
---
# <a name="create-and-manage-api-keys-for-an-azure-cognitive-search-service"></a>Creación y administración de claves de API para un servicio de Azure Cognitive Search

Todas las solicitudes que se realizan a un servicio de búsqueda necesitan una clave de API de solo lectura generada de forma específica para el servicio. Dicha clave de API es el único mecanismo para autenticar el acceso del punto de conexión de su servicio de búsqueda y se debe incluir en todas las solicitudes. En las [soluciones REST](search-get-started-postman.md) la clave de API se especifica normalmente en un encabezado de solicitud. En las [soluciones .NET](search-howto-dotnet-sdk.md#core-scenarios), se especifica a menudo una clave como valor de configuración y, posteriormente, se pasa como [Credentials](/dotnet/api/microsoft.azure.search.searchserviceclient.credentials) (clave de administración) o [SearchCredentials](/dotnet/api/microsoft.azure.search.searchserviceclient.searchcredentials) (clave de consulta) en [SearchServiceClient](/dotnet/api/microsoft.azure.search.searchserviceclient).

Las claves se crean con el servicio de búsqueda durante el aprovisionamiento de este. Puede ver y obtener los valores de clave en [Azure Portal](https://portal.azure.com).

![Página del portal, recuperar configuración, sección de claves](media/search-manage/azure-search-view-keys.png)

## <a name="what-is-an-api-key"></a>¿Qué es una clave de API?

Una clave de API es una cadena que se compone de letras y números generados aleatoriamente. A través de [permisos basados en roles](search-security-rbac.md), puede eliminar o leer las claves, pero no se puede reemplazar una clave con una contraseña definida por el usuario ni usar Active Directory como metodología de autenticación principal para acceder a las operaciones de búsqueda. 

Se usan dos tipos de claves para obtener acceso al servicio de búsqueda: administración (lectura y escritura) y consulta (solo lectura).

|Clave|Descripción|límites|  
|---------|-----------------|------------|  
|Administración|Concede derechos completos para todas las operaciones, incluida la capacidad para administrar el servicio, crear y eliminar índices, indexadores y orígenes de datos.<br /><br /> Cuando se crea el servicio, se generan dos claves de administración, a las que se hace referencia como claves *principal* y *secundaria* en el portal. También se pueden volver a generar individualmente a petición. Tener dos claves le permite la rotación de una clave mientras se usa la segunda clave para un acceso continuado al servicio.<br /><br /> Las claves de administración solo se especifican en los encabezados de la solicitud HTTP. No se puede colocar una clave de API de administración en una dirección URL.|Máximo 2 por servicio|  
|Consultar|Conceden acceso de solo lectura a índices y documentos y, normalmente, se distribuyen entre las aplicaciones cliente que emiten solicitudes de búsqueda.<br /><br /> Las claves de consulta se crean bajo petición. Puede crearlas manualmente en el portal o mediante programación con la [API de REST de administración](/rest/api/searchmanagement/).<br /><br /> Las claves de consulta se pueden especificar en un encabezado de solicitud HTTP para operaciones de búsqueda y sugerencias. Como alternativa, puede pasar una clave de consulta como un parámetro en una dirección URL. En función de cómo formule la solicitud la aplicación cliente, puede resultar más fácil pasar la clave como un parámetro de consulta:<br /><br /> `GET /indexes/hotels/docs?search=*&$orderby=lastRenovationDate desc&api-version=2020-06-30&api-key=[query key]`|50 por servicio|  

 Visualmente, no hay distinción entre las claves de administración y de consulta. Ambas claves son cadenas formadas por 32 caracteres alfanuméricos generados aleatoriamente. Si necesita saber qué tipo de clave se especifica en la aplicación, puede [comprobar los valores de las claves en el portal](https://portal.azure.com) o usar la [API de REST](/rest/api/searchmanagement/) para devolver el valor y el tipo de clave.  

> [!NOTE]  
>  Se considera una práctica poco segura pasar datos confidenciales, como una `api-key` en el identificador URI de la solicitud. Por este motivo, Azure Cognitive Search solo acepta una clave de consulta como `api-key` en la cadena de consulta, y debe evitar hacerlo a menos que el contenido del índice deba estar disponible públicamente. Como regla general, se recomienda pasar la `api-key` como un encabezado de solicitud.  

## <a name="find-existing-keys"></a>Encontrar las claves existentes

Puede obtener las claves de acceso en el portal o mediante la [API de REST de administración](/rest/api/searchmanagement/). Para obtener más información, consulte [Administrar claves de API de administración y consulta](search-security-api-keys.md).

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Obtenga la lista de los [servicios de búsqueda](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) de su suscripción.
3. Seleccione el servicio y, en la página de información general, haga clic en **Configuración** >**Claves** para ver las claves de administración y de consulta.

   ![Página del portal, ver configuración, sección de claves](media/search-security-overview/settings-keys.png)

## <a name="create-query-keys"></a>Creación de claves de consulta

Las claves de consulta se usan para el acceso de solo lectura a los documentos dentro de un índice para las operaciones destinadas a una colección de documentos. Las consultas de búsqueda, filtro y sugerencias son todas las operaciones que toman una clave de consulta. Cualquier operación de solo lectura que devuelve definiciones de objetos o datos del sistema, como una definición del índice o el estado del indexador, requiere una clave de administración.

Restringir el acceso y las operaciones en las aplicaciones cliente es esencial para proteger los activos de búsqueda en el servicio. Use siempre una clave de consulta en lugar de una clave de administración para cualquier consulta originada desde una aplicación cliente.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Obtenga la lista de los [servicios de búsqueda](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) de su suscripción.
3. Seleccione el servicio y, en la página de información general, haga clic en **Configuración** >**Claves**.
4. Haga clic en **Administrar claves de consulta**.
5. Use la clave de consulta ya generada para el servicio o cree hasta 50 claves de consulta nuevas. La clave de consulta predeterminada no tiene nombre, pero se puede asignar un nombre a las claves de consulta adicionales para facilitar la administración.

   ![Creación o uso de una clave de consulta](media/search-security-overview/create-query-key.png) 

> [!Note]
> Un ejemplo de código que muestra el uso de claves de consulta puede encontrarse en [Consulta de un índice de Azure Cognitive Search en C#](./search-get-started-dotnet.md).

<a name="regenerate-admin-keys"></a>

## <a name="regenerate-admin-keys"></a>Regeneración de claves de administración

Se crean dos claves de administración para cada servicio para que pueda rotar una clave principal mediante la clave secundaria para la continuidad empresarial.

1. En la página **Configuración** >**Claves**, copie la clave secundaria.
2. Para todas las aplicaciones, actualice la configuración de la clave de API para usar la clave secundaria.
3. Regenere la clave principal
4. Actualice todas las aplicaciones para usar la nueva clave principal.

Si regenera las claves al mismo tiempo sin querer, se producirá un error HTTP 403 Prohibido en todas las solicitudes de cliente que usen esas claves. Sin embargo, no se elimina el contenido ni se le bloquea permanentemente. 

Todavía puede tener acceso al servicio mediante el portal o la capa de administración ([API REST](/rest/api/searchmanagement/), [PowerShell](./search-manage-powershell.md) o Azure Resource Manager). Las funciones de administración están operativas a través de un identificador de suscripción, no un servicio de claves de API; por lo tanto, seguirán estando disponibles incluso si no lo están las claves de API. 

Después de crear nuevas claves a través del portal o la capa de administración, se restaura el acceso al contenido (índices, indexadores, orígenes de datos, las asignaciones de sinónimos) una vez que tiene las nuevas claves y las proporciona en las solicitudes.

## <a name="secure-api-keys"></a>Protección de las claves de API
La seguridad de las claves se garantiza mediante la restricción del acceso a través del portal o de las interfaces de Resource Manager (PowerShell o interfaz de línea de comandos). Como se ha indicado, los administradores de suscripciones pueden ver y volver a generar todas las claves de API. Como medida de precaución, revise las asignaciones de roles para conocer quién tiene acceso a las claves de administración.

+ En el panel de servicio, haga clic en **Control de acceso (IAM)** y, a continuación, en la pestaña **Asignaciones de roles** para ver las asignaciones de roles para su servicio.

Los miembros de los roles siguientes pueden ver y regenerar las claves: Propietario, Colaborador, [Colaboradores de Search Service](../role-based-access-control/built-in-roles.md#search-service-contributor)

> [!Note]
> Para tener acceso basado en la identidad a través de los resultados de búsqueda, puede crear filtros de seguridad para recortar los resultados según la identidad y quitar los documentos para los que el solicitante no debe tener acceso. Para obtener más información, consulte [Filtros de seguridad](search-security-trimming-for-azure-search.md) y [Seguridad con Active Directory](search-security-trimming-for-azure-search-with-aad.md).

## <a name="see-also"></a>Consulte también

+ [Control de acceso basado en rol en Azure Cognitive Search](search-security-rbac.md)
+ [Administración mediante PowerShell](search-manage-powershell.md) 
+ [Artículo de rendimiento y optimización](search-performance-optimization.md)