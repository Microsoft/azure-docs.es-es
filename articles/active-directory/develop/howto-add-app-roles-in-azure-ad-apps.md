---
title: Incorporación de roles de aplicación y su obtención desde un token | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre cómo agregar roles de aplicación en una aplicación registrada en Azure Active Directory, asignar usuarios y grupos a estos roles y recibirlos en la notificación `roles` del token.
services: active-directory
author: kalyankrishna1
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: how-to
ms.date: 07/15/2020
ms.author: kkrishna
ms.reviewer: kkrishna, jmprieur
ms.custom: aaddev
ms.openlocfilehash: 5a2acb08971bc0878c943047c42c9dc2a9525794
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88141438"
---
# <a name="how-to-add-app-roles-in-your-application-and-receive-them-in-the-token"></a>Procedimientos: Agregar roles de aplicación en la aplicación y recibirlos en el token

El control de acceso basado en rol (RBAC) es un mecanismo popular para exigir la autorización en las aplicaciones. Cuando se usa RBAC, un administrador concede permisos a roles y no a usuarios individuales o grupos. El administrador, a continuación, puede asignar roles a diferentes usuarios y grupos para controlar quién tiene acceso a qué contenido y funcionalidad.

Mediante el uso de RBAC con roles de aplicación y notificaciones de rol, los desarrolladores pueden exigir de forma segura la autorización en las aplicaciones con poco esfuerzo por su parte.

Otro enfoque consiste en usar grupos de Azure AD y notificaciones de grupo, como se muestra en el ejemplo [WebApp-GroupClaims-DotNet](https://github.com/Azure-Samples/WebApp-GroupClaims-DotNet). Los grupos de Azure AD y los roles de aplicación no son mutuamente excluyentes, se pueden usar conjuntamente para proporcionar un control de acceso aún más preciso.

## <a name="declare-roles-for-an-application"></a>Declaración de roles para una aplicación

Estos roles de aplicación se definen en [Azure Portal](https://portal.azure.com) en el manifiesto de registro de la aplicación.  Cuando un usuario inicia sesión en la aplicación, Azure AD emite una notificación `roles` por cada rol que se ha concedido al usuario individualmente y por su pertenencia a grupos.  La asignación de usuarios y grupos a los roles se puede realizar mediante la interfaz de usuario del portal o mediante programación con [Microsoft Graph](/graph/azuread-identity-access-management-concept-overview).

### <a name="declare-app-roles-using-azure-portal"></a>Declaración de roles de aplicación mediante Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal.
1. En la lista **Favoritos** o **Todos los directorios**, elija el inquilino de Active Directory en el que quiere registrar la aplicación.
1. En Azure Portal, busque y seleccione **Azure Active Directory**.
1. En el panel **Azure Active Directory**, seleccione **Registros de aplicaciones** para ver una lista de todas las aplicaciones.
1. Seleccione la aplicación para la que desea definir roles de aplicación. A continuación, seleccione **Manifiesto**.
1. Localice la opción de configuración `appRoles` para editar el manifiesto de la aplicación y agregar los roles de aplicación.

     > [!NOTE]
     > Cada definición de roles de aplicación de este manifiesto debe tener un GUID válido diferente en el contexto del manifiesto para la propiedad `id`.
     >
     > La propiedad `value` de cada definición de rol de la aplicación debe coincidir exactamente con las cadenas que se usan en el código de la aplicación. La propiedad `value` no puede contener espacios. Si es así, recibirá un error al guardar el manifiesto.

1. Guarde el manifiesto.

### <a name="examples"></a>Ejemplos

El ejemplo siguiente muestra los distintos `appRoles` que se pueden asignar a `users`.

> [!NOTE]
>El elemento `id` debe ser un GUID único.

```Json
"appId": "8763f1c4-f988-489c-a51e-158e9ef97d6a",
"appRoles": [
    {
      "allowedMemberTypes": [
        "User"
      ],
      "displayName": "Writer",
      "id": "d1c2ade8-98f8-45fd-aa4a-6d06b947c66f",
      "isEnabled": true,
      "description": "Writers Have the ability to create tasks.",
      "value": "Writer"
    }
  ],
"availableToOtherTenants": false,
```

> [!NOTE]
>El elemento `displayName` puede contener espacios.

Puede definir roles de aplicación para `users`, `applications` o ambos. Cuando esté disponible para `applications`, los roles de aplicación aparecen como permisos de aplicación en la sección **Administrar** > **Permisos de API > Agregar un permiso > Mis API > Elegir una API > Permisos de la aplicación**. El ejemplo siguiente muestra un rol de aplicación dirigido a un elemento `Application`.

```Json
"appId": "8763f1c4-f988-489c-a51e-158e9ef97d6a",
"appRoles": [
    {
      "allowedMemberTypes": [
        "Application"
      ],
      "displayName": "ConsumerApps",
      "id": "47fbb575-859a-4941-89c9-0f7a6c30beac",
      "isEnabled": true,
      "description": "Consumer apps have access to the consumer data.",
      "value": "Consumer"
    }
  ],
"availableToOtherTenants": false,
```

El número de roles definidos afecta a los límites que tiene el manifiesto de aplicación. Se describen detalladamente en la página de [límites del manifiesto](./reference-app-manifest.md#manifest-limits).

### <a name="assign-users-and-groups-to-roles"></a>Asignación de usuarios y grupos a roles

Una vez que haya agregado los roles de aplicación en la aplicación, puede asignar usuarios y grupos a estos roles.

1. En el panel **Azure Active Directory**, seleccione **Aplicaciones empresariales** en el menú de navegación izquierdo **Azure Active Directory**.
1. Seleccione **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

     Si no ve la aplicación que desea, utilice los distintos filtros de la parte superior de la lista **Registros de aplicaciones** para restringir la lista o desplácese hacia abajo en la lista para buscar la aplicación.

1. Seleccione la aplicación en la que desea asignar usuarios o grupos de seguridad a los roles.
1. Seleccione el panel **Usuarios y grupos** en el menú de navegación izquierdo de la aplicación.
1. En la parte superior de la lista **Usuarios y grupos**, seleccione el botón **Agregar usuario** para abrir el panel **Agregar asignación**.
1. Seleccione el selector **Usuarios y grupos** del panel **Agregar asignación**.

     Se mostrará una lista de usuarios y grupos de seguridad junto con un cuadro de texto para buscar un usuario o grupo determinado. Esta pantalla permite seleccionar varios usuarios y grupos en una sola operación.

1. Cuando haya terminado la selección de los usuarios y grupos, presione el botón **Seleccionar** en la parte inferior para desplazarse al apartado siguiente.
1. Elija el selector **Seleccionar rol** del panel **Agregar asignación**. Se mostrarán todos los roles que se declararon previamente en el manifiesto de la aplicación.
1. Elija un rol y presione el botón **Seleccionar**.
1. Presione el botón **Asignar** en la parte inferior para finalizar las asignaciones de usuarios y grupos a la aplicación.
1. Confirme que los usuarios y grupos que agregó se muestran en la lista **Usuarios y grupos** actualizada.

### <a name="receive-roles-in-tokens"></a>Recepción de roles en tokens

Cuando los usuarios asignados a los distintos roles de aplicación inicien sesión en la aplicación, los tokens tendrán sus roles asignados en la notificación `roles`.

## <a name="more-information"></a>Más información

- [Incorporación de autorización mediante roles de aplicación y notificaciones de roles a una aplicación web de ASP.NET Core](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/5-WebApp-AuthZ/5-1-Roles)
- [Implementación de la autorización en las aplicaciones con la plataforma de identidad de Microsoft (vídeo)](https://www.youtube.com/watch?v=LRoc-na27l0)
- [Azure Active Directory, ahora con notificaciones de grupo y roles de aplicación](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-Active-Directory-now-with-Group-Claims-and-Application/ba-p/243862)
- [Manifiesto de la aplicación de Azure Active Directory](./reference-app-manifest.md)
- [Tokens de acceso de AAD](access-tokens.md)
- [`id_tokens` de AAD](id-tokens.md)