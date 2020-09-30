---
title: Incorporación de roles de aplicación y su obtención desde un token | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre cómo agregar roles de aplicación en una aplicación registrada en Azure Active Directory, asignar usuarios y grupos a estos roles y recibirlos en la notificación "roles" del token.
services: active-directory
author: kalyankrishna1
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: how-to
ms.date: 09/15/2020
ms.author: kkrishna
ms.reviewer: kkrishna, jmprieur
ms.custom: aaddev
ms.openlocfilehash: 02fe3c1ebc893dea259abcda3ea1d13ab96d68d5
ms.sourcegitcommit: 7374b41bb1469f2e3ef119ffaf735f03f5fad484
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2020
ms.locfileid: "90706020"
---
# <a name="how-to-add-app-roles-in-your-application-and-receive-them-in-the-token"></a>Procedimientos: Agregar roles de aplicación en la aplicación y recibirlos en el token

El control de acceso basado en rol (RBAC) es un mecanismo popular para exigir la autorización en las aplicaciones. Cuando se usa RBAC, un administrador concede permisos a roles y no a usuarios individuales o grupos. El administrador, a continuación, puede asignar roles a diferentes usuarios y grupos para controlar quién tiene acceso a qué contenido y funcionalidad.

Mediante el uso de RBAC con roles de aplicación y notificaciones de rol, los desarrolladores pueden exigir de forma segura la autorización en las aplicaciones con menos esfuerzo por su parte.

Otro enfoque consiste en usar grupos de Azure AD y notificaciones de grupo, como se muestra en el código de ejemplo [WebApp-GroupClaims-DotNet](https://github.com/Azure-Samples/WebApp-GroupClaims-DotNet) en GitHub. Los grupos de Azure AD y los roles de aplicación no son mutuamente excluyentes, se pueden usar conjuntamente para proporcionar un control de acceso aún más preciso.

## <a name="declare-roles-for-an-application"></a>Declaración de roles para una aplicación

Los roles de aplicación se definen en [Azure Portal](https://portal.azure.com).  Cuando un usuario inicia sesión en la aplicación, Azure AD emite una notificación `roles` por cada rol que se ha concedido al usuario individualmente y por su pertenencia a grupos.  La asignación de usuarios y grupos a los roles se puede realizar mediante la interfaz de usuario del portal o mediante programación con [Microsoft Graph](/graph/azuread-identity-access-management-concept-overview).

Para crear un rol de aplicación:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En el menú superior, seleccione el filtro**Directorio + suscripción** y, a continuación, elija el inquilino de Azure Active Directory que contiene el registro de la aplicación al que desea agregar un rol de aplicación.
1. Busque y seleccione **Azure Active Directory**.
1. En **Administrar**, seleccione **Registros de aplicaciones** y, a continuación, seleccione la aplicación en la que desea definir los roles de aplicación.
1. Seleccione **Roles de aplicación | Versión preliminar** y, a continuación, seleccione **Crear rol de aplicación**.

   :::image type="content" source="media/howto-add-app-roles-in-azure-ad-apps/app-roles-overview-pane.png" alt-text="Panel de roles de aplicación durante el registro de una aplicación en Azure Portal":::
1. En el panel **Crear rol de aplicación**, escriba la configuración del rol. La tabla que sigue a la imagen describe cada configuración y sus parámetros.

    :::image type="content" source="media/howto-add-app-roles-in-azure-ad-apps/app-roles-create-context-pane.png" alt-text="Panel contextual de creación de roles de aplicación durante el registro de una aplicación en Azure Portal":::

    | Campo | Descripción | Ejemplo |
    |-------|-------------|---------|
    | **Nombre para mostrar** | Nombre para mostrar para el rol de aplicación que aparece en las experiencias de asignación de la aplicación y de consentimiento del administrador. Este valor puede incluir espacios.  | `Survey Writer` |
    | **Tipos de miembros permitido** | Especifica si este rol de aplicación puede asignarse a usuarios, aplicaciones o ambos. | `Users/Groups` |
    | **Valor** | Especifica el valor de la notificación de roles que debería esperar la aplicación en los tokens de acceso y autenticación. El valor debe coincidir exactamente con la cadena a la que se hace referencia en el código de la aplicación. El valor no puede contener espacios. | `Survey.Create` |
    | **Descripción** | Una descripción más detallada del rol de aplicación que se muestra durante la asignación de aplicaciones de administración y las experiencias de consentimiento. | `Writers can create surveys.` |
    | **¿Quiere habilitar este rol de aplicación?** | Si está habilitada el rol de aplicación. Para eliminar un rol de aplicación, desactive esta casilla y aplique el cambio antes de intentar la operación de eliminación. | *Activada* |

1. Seleccione **Aplicar** para guardar los cambios.

> [!NOTE]
> El número de roles que agrega se cuenta para los límites definidos para un manifiesto de aplicación. Para obtener información sobre estos límites, consulte la sección [Límites de manifiesto](./reference-app-manifest.md#manifest-limits) de la referencia del [manifiesto de aplicación de Azure Active Directory](reference-app-manifest.md).

## <a name="assign-users-and-groups-to-roles"></a>Asignación de usuarios y grupos a roles

Una vez que haya agregado los roles de aplicación en la aplicación, puede asignar usuarios y grupos a estos roles.

1. En **Azure Active Directory**, en Azure Portal, seleccione **Aplicaciones empresariales** en el menú de navegación izquierdo.
1. Seleccione **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

     Si la aplicación no aparece en la lista, use los filtros de la parte superior de la lista **Todas las aplicaciones** para restringir la lista o desplácese hacia abajo en la lista para localizar la aplicación.

1. Seleccione la aplicación en la que desea asignar usuarios o grupos de seguridad a los roles.
1. En **Administrar**, seleccione **Usuarios y grupos**.
1. Seleccione **Agregar miembro** para abrir el panel **Agregar asignación**.
1. Seleccione el selector **Usuarios y grupos** del panel **Agregar asignación**.

     Se muestra una lista de usuarios y grupos de seguridad. Puede buscar un usuario o grupo determinados y seleccionar varios usuarios y grupos que aparecen en la lista.

1. Una vez que haya seleccionado usuarios y grupos, seleccione el botón **Seleccionar** para continuar.
1. En el panel **Agregar asignación**, seleccione **Seleccionar rol**. Se muestran todos los roles que ha definido para la aplicación.
1. Elija un rol y seleccione el botón **Seleccionar**.
1. Seleccione el botón **Asignar** para finalizar las asignaciones de usuarios y grupos a la aplicación.
1. Confirme que los usuarios y grupos que agregó se muestran en la lista **Usuarios y grupos**.

## <a name="receive-roles-in-tokens"></a>Recepción de roles en tokens

Cuando los usuarios asignados a los distintos roles de aplicación inicien sesión en la aplicación, los tokens tendrán sus roles asignados en la notificación `roles`.

## <a name="web-api-application-permissions"></a>Permisos de la aplicación de API web

Puede definir roles de aplicación que tengan como destino **Usuarios/Grupos**, **Aplicaciones** o **Ambos**. Al seleccionar **Aplicaciones** o **Ambos**, el rol de aplicación aparece como permiso de aplicación en los **permisos de la API** de la aplicación cliente.

Para seleccionar el permiso de aplicación en un registro de aplicación cliente después de definir un rol que tenga como destino **Aplicaciones** o **Ambos**:

1. En Azure Portal, seleccione **Azure Active Directory**, **Registros de aplicaciones** y elija el registro de la aplicación cliente.
1. En **Administrar**, seleccione **Permisos de API**.
1. Seleccione **Agregar un permiso** > **Mis API**.
1. Seleccione la aplicación a la que agregó el rol de aplicación y, a continuación, seleccione **Permisos de aplicación**.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre los roles de aplicación, vea los recursos siguientes:

- Ejemplo de código: [Incorporación de autorización mediante roles de aplicación y notificaciones de roles a una aplicación web de ASP.NET Core](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/5-WebApp-AuthZ/5-1-Roles) (GitHub)
- Vídeo: [Implementación de la autorización en las aplicaciones con la plataforma de identidad de Microsoft](https://www.youtube.com/watch?v=LRoc-na27l0) (1:01:15)
- [Manifiesto de la aplicación de Azure Active Directory](./reference-app-manifest.md)
- [Tokens de acceso de Azure AD](access-tokens.md)
- [Azure AD`id_tokens`](id-tokens.md)
