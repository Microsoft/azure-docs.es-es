---
title: Permisos en Azure Active Directory | Microsoft Docs
description: Obtenga información sobre los permisos en Azure Active Directory y cómo utilizarlos.
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: 6c0dc122-2cd8-4d70-be5a-3943459d308e
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: ryanwi
ms.reviewer: jesakowi, justhu
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6fb4342e024d826c65ed33184aaf33012d09190a
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65545194"
---
# <a name="permissions-and-consent-in-the-azure-active-directory-v10-endpoint"></a>Permisos y consentimiento en el punto de conexión v1.0 de Azure Active Directory

[!INCLUDE [active-directory-develop-applies-v1](../../../includes/active-directory-develop-applies-v1.md)]

Azure Active Directory (Azure AD) hace un uso extensivo de los permisos para los flujos de OAuth y OpenID Connect (OIDC). Cuando la aplicación recibe un token de acceso de Azure AD, el token de acceso incluye notificaciones que describen los permisos que tiene la aplicación con respecto a un recurso determinado.

Los *permisos*, también conocidos como *ámbitos*, facilitan la autorización del recurso, ya que este solo tiene que comprobar que el token contiene el permiso adecuado para cualquier API a la que llame la aplicación.

## <a name="types-of-permissions"></a>Tipos de permisos

Azure AD define dos tipos de permisos:

* **Permisos delegados**: se utilizan en aplicaciones que tienen un usuario con la sesión iniciada. Para estas aplicaciones, el usuario o un administrador dan su consentimiento para los permisos que la aplicación requiere y a la aplicación se le delega el permiso para actuar como el usuario que inició sesión al realizar llamadas a una API. En función de la API de que se trate, es posible que el usuario no tenga la capacidad de dar su consentimiento a la API directamente y, en su lugar, tenga que [pedir a un administrador que proporcione el "consentimiento del administrador"](/azure/active-directory/develop/active-directory-devhowto-multi-tenant-overview).
* **Permisos de aplicación**: los usan las aplicaciones que se ejecutan sin la presencia de un usuario con la sesión iniciada; por ejemplo, las aplicaciones que se ejecutan como demonios o servicios en segundo plano. Los permisos de aplicación solo puede [autorizarlos un administrador](/azure/active-directory/develop/active-directory-v2-scopes#requesting-consent-for-an-entire-tenant), porque suelen ser muy eficaces y permitir el acceso a los datos más allá de los límites de usuario o a datos que deberían estar restringidos a los administradores.

Los permisos efectivos son los permisos que la aplicación tendrá al realizar solicitudes a una API. 

* En el caso de los permisos delegados, los permisos efectivos de la aplicación serán la intersección con menos privilegios de los permisos delegados que se le han concedido a la aplicación (mediante consentimiento) y los privilegios del usuario que tiene iniciada la sesión actualmente. La aplicación nunca puede tener más privilegios que el usuario que tiene la sesión iniciada. Dentro de las organizaciones, los privilegios del usuario que tiene la sesión iniciada pueden determinarse mediante directivas o pertenencia a uno o varios roles de administrador. Para información sobre qué roles de administrador pueden consentir los permisos delegados, consulte [Permisos de roles de administrador en Azure AD](../users-groups-roles/directory-assign-admin-roles.md).
    Por ejemplo, suponga que a su aplicación se le ha concedido el permiso delegado `User.ReadWrite.All` en Microsoft Graph. Este permiso concede a su aplicación de forma nominal un permiso para leer y actualizar el perfil de cada usuario de una organización. Si el usuario que inició sesión es un administrador global, la aplicación podrá actualizar el perfil de cada usuario de la organización. Sin embargo, si el usuario con la sesión iniciada no pertenece a un rol de administrador, la aplicación podrá actualizar solo el perfil del usuario que tiene la sesión iniciada. No podrá actualizar los perfiles de otros usuarios de la organización, porque el usuario para el que tiene permiso para actuar en su nombre no tiene tales privilegios.
* Para los permisos de aplicación, los permisos efectivos de la aplicación son el nivel completo de privilegios que concede el permiso. Por ejemplo, una aplicación que tiene el permiso de aplicación `User.ReadWrite.All` puede actualizar el perfil de cada usuario de la organización.

## <a name="permission-attributes"></a>Atributos de los permisos
Los permisos de Azure AD tienen un número de propiedades que ayudan a los usuarios, administradores o desarrolladores de aplicaciones a tomar decisiones fundamentadas sobre a qué concede acceso el permiso.

> [!NOTE]
> Puede ver los permisos que una entidad de servicio o aplicación de Azure AD exponen mediante Azure Portal o PowerShell. Pruebe este script para ver los permisos expuestos por Microsoft Graph.
> ```powershell
> Connect-AzureAD
> 
> # Get OAuth2 Permissions/delegated permissions
> (Get-AzureADServicePrincipal -filter "DisplayName eq 'Microsoft Graph'").OAuth2Permissions
> 
> # Get App roles/application permissions
> (Get-AzureADServicePrincipal -filter "DisplayName eq 'Microsoft Graph'").AppRoles
> ```

| Nombre de propiedad | DESCRIPCIÓN | Ejemplo |
| --- | --- | --- |
| `ID` | Es un valor de GUID que identifica este permiso de forma única. | 570282fd-fa5c-430d-a7fd-fc8dc98a9dca |
| `IsEnabled` | Indica si este permiso está disponible para su uso. | true |
| `Type` | Indica si este permiso requiere el consentimiento del usuario o el consentimiento del administrador. | Usuario |
| `AdminConsentDescription` | Es una descripción que se muestra a los administradores durante las experiencias de consentimiento de administración. | Permite que la aplicación lea el correo electrónico de los buzones del usuario. |
| `AdminConsentDisplayName` | Es el nombre descriptivo que se muestra a los administradores durante las experiencias de consentimiento de administración. | Leer correo de usuario |
| `UserConsentDescription` | Es una descripción que se muestra a los usuarios durante una experiencia de consentimiento del usuario. |  Permite que la aplicación lea el correo electrónico del buzón. |
| `UserConsentDisplayName` | Es el nombre descriptivo que se muestra a los usuarios durante la experiencia de consentimiento de usuario. | Leer tu correo |
| `Value` | Es la cadena que se usa para identificar el permiso durante los flujos de autorización de OAuth 2.0. `Value` también puede combinarse con la cadena del URI del identificador de la aplicación para formar un nombre de permiso completo. | `Mail.Read` |

## <a name="types-of-consent"></a>Tipos de consentimiento

Las aplicaciones de Azure AD dependen del consentimiento para poder acceder a los recursos o las API necesarios. Hay una serie de tipos de consentimiento que la aplicación puede necesitar saber para que se pueda ejecutar correctamente. Si va a definir permisos, también debe saber cómo accederán los usuarios a la aplicación o la API.

* **Consentimiento de usuario estático**: se concede automáticamente durante el [flujo de autorización de OAuth 2.0](v1-protocols-oauth-code.md#request-an-authorization-code) al especificar el recurso con el que la aplicación desea interactuar. En el escenario del consentimiento de usuario estático, la aplicación debe haber especificado a todos los permisos que necesita en la configuración de la aplicación en Azure Portal. Si al usuario o al administrador, según corresponda, no se le concede permiso a esta aplicación, Azure AD pedirá al usuario que proporcione su consentimiento en este momento. 

    Obtenga más información sobre el registro de una aplicación de Azure AD que solicita acceso a un conjunto estático de API.
* **Consentimiento de usuario dinámico**: es una característica del modelo de aplicación v2 de Azure AD. En este escenario, la aplicación solicita un conjunto de permisos que necesita en el [flujo de autorización de OAuth 2.0 para aplicaciones v2](/azure/active-directory/develop/active-directory-v2-scopes#requesting-individual-user-consent). Si el usuario aún no ha dado su consentimiento, se le pedirá en este momento. [Obtenga más información sobre el consentimiento dinámico](/azure/active-directory/develop/active-directory-v2-compare#incremental-and-dynamic-consent).

    > [!IMPORTANT]
    > El consentimiento dinámico puede resultar conveniente, pero presenta un gran desafío para los permisos que requieren el consentimiento del administrador, ya que la experiencia de consentimiento de administrador no conoce los permisos en el momento del consentimiento. Si necesita permisos con privilegios de administrador o si su aplicación usa el consentimiento dinámico, debe registrar todos los permisos en Azure Portal (no solo el subconjunto de permisos que requieren el consentimiento del administrador). Esto permite a los administradores de inquilinos dar su consentimiento en nombre de todos los usuarios.
  
* **Consentimiento del administrador**: es necesario si la aplicación necesita acceder a determinados permisos con privilegios elevados. El consentimiento de administrador garantiza que los administradores dispongan de algunos controles adicionales antes de autorizar a las aplicaciones o a los usuarios a acceder a datos con privilegios elevados de la organización. [Obtenga más información sobre cómo conceder el permiso de administrador](/azure/active-directory/develop/active-directory-v2-scopes#using-the-admin-consent-endpoint).

## <a name="best-practices"></a>Procedimientos recomendados

### <a name="client-best-practices"></a>Procedimientos recomendados de cliente

- Solicite únicamente los permisos que necesita la aplicación. Las aplicaciones con demasiados permisos corren el riesgo de exponer los datos del usuario si se ven comprometidos.
- Elija entre permisos delegados y permisos de aplicación en función del escenario que admite la aplicación.
    - Utilice siempre permisos delegados si la llamada se está realizando en nombre del usuario.
    - Use solo los permisos de aplicación si la aplicación no es interactiva y no realiza llamadas en nombre de ningún usuario específico. Los permisos de aplicación cuentan con altos privilegios y solo deben utilizarse cuando sea absolutamente necesario.
- Cuando se utiliza una aplicación basada en el punto de conexión v2.0, establezca siempre los permisos estáticos (aquellos especificados en el registro de aplicación) como el superconjunto de los permisos dinámicos que solicita en tiempo de ejecución (aquellos especificados en el código y enviados como parámetros de consulta en la solicitud de autorización) para que escenarios como el consentimiento de administrador funcionen correctamente.

### <a name="resourceapi-best-practices"></a>Procedimientos recomendados de recursos y API

- Los recursos que exponen las API deben definir los permisos que son específicos para los datos o las acciones que protegen. Seguir este procedimiento recomendado ayuda a garantizar que los clientes no terminarán con permisos para acceder a datos que no necesitan y que los usuarios estén bien informados de los datos para los que dan el consentimiento.
- Los recursos deben definir explícitamente los permisos `Read` y `ReadWrite` por separado.
- Los recursos deben marcar todos los permisos que permiten acceder a los datos más allá de los límites del usuario como permisos `Admin`.
- Los recursos deben seguir el patrón de nomenclatura `Subject.Permission[.Modifier]`, donde:
  - `Subject` se corresponde con el tipo de datos que están disponibles,
  - `Permission` se corresponde con la acción que un usuario puede realizar en esos datos y
  - `Modifier` se utiliza opcionalmente para describir las especializaciones de otro permiso.
    
    Por ejemplo:
  - Mail.Read: permite a los usuarios leer correos.
  - Mail.ReadWrite: permite a los usuarios leer o escribir correos.
  - Mail.ReadWrite.All: permite que un administrador o usuario tenga acceso a todo el correo de la organización.
