---
title: Cómo aparecen las aplicaciones en el panel de acceso | Microsoft Docs
description: Solucionar por qué una aplicación no aparece en el panel de acceso
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/11/2017
ms.author: mimart
ms.reviewr: japere
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8fa8ea75cc7fda05326c802c25a91d025b66b5ce
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65784409"
---
# <a name="how-applications-appear-on-the-access-panel"></a>Cómo aparecen las aplicaciones en el panel de acceso

El panel de acceso es un portal basado en web que permite que un usuario con una cuenta profesional o educativa de Azure Active Directory (Azure AD) vea e inicie las aplicaciones basadas en la nube a las que el administrador de Azure AD le ha concedido acceso. Estas aplicaciones se configuran en nombre del usuario en el portal de Azure AD. El administrador puede aprovisionar la aplicación para el usuario directamente o para un grupo del que el usuario forma parte con lo cual la aplicación aparecerá en el panel de acceso del usuario.

## <a name="general-issues-to-check-first"></a>Problemas generales para comprobar primero

-   Si se ha quitado una aplicación a un usuario o a un grupo al que este pertenece, intente volver a iniciar y cerrar sesión en el panel de acceso del usuario después de unos minutos para ver si la aplicación se ha quitado.

-   Si se ha quitado una licencia de un usuario o grupo del que el usuario es miembro, los cambios pueden tardar en aplicarse, en función del tamaño y la complejidad del grupo. Espere un poco más de tiempo antes de iniciar sesión en el panel de acceso.

## <a name="problems-related-to-assigning-applications-to-users"></a>Problemas relacionados con la asignación de aplicaciones a los usuarios

Un usuario puede ver una aplicación en su panel de acceso si ya se le había asignado previamente a él. A continuación, se muestran algunas maneras de comprobarlo:

-   [Comprobación de si un usuario está asignado a la aplicación](#check-if-a-user-is-assigned-to-the-application)

-   [Comprobar si un usuario posee una licencia relacionada con la aplicación](#check-if-a-user-is-under-a-license-related-to-the-application)


### <a name="check-if-a-user-is-assigned-to-the-application"></a>Comprobar si un usuario está asignado a la aplicación

Para comprobar si un usuario está asignado a la aplicación, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

6. **Busque** el nombre de la aplicación en cuestión.

7. Haga clic en **Usuarios y grupos**.

8. Compruebe si el usuario está asignado a la aplicación.

   * Si desea quitar el usuario de la aplicación, **haga clic en la fila** del usuario y seleccione **eliminar**.

### <a name="check-if-a-user-is-under-a-license-related-to-the-application"></a>Comprobar si un usuario posee una licencia relacionada con la aplicación

Para comprobar las licencias asignadas de un usuario, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Usuarios y grupos** en el menú de navegación.

5. Haga clic en **Todos los usuarios**.

6. **Busque** el usuario en el que está interesado y **haga clic en la fila** para seleccionarlo.

7. Haga clic en **Licencias** para ver qué licencias tiene asignadas actualmente el usuario.

   * Si el usuario está asignado a una licencia de Office, esto permitirá que las aplicaciones de Office aparezcan en el panel de acceso del usuario.

## <a name="problems-related-to-assigning-applications-to-groups"></a>Problemas relacionados con la asignación de aplicaciones a grupos

Puede que un usuario vea una aplicación en su Panel de acceso porque forme parte de un grupo al que se ha asignado esa aplicación. A continuación, se muestran algunas maneras de comprobarlo:

-   [Comprobación de la pertenencia a grupos de un usuario](#check-a-users-group-memberships)

-   [Comprobar si un usuario es miembro de un grupo asignado a una licencia](#check-if-a-user-is-a-member-of-a-group-assigned-to-a-license)

### <a name="check-a-users-group-memberships"></a>Comprobación de la pertenencia a grupos de un usuario

Para comprobar la pertenencia de un grupo, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Usuarios y grupos** en el menú de navegación.

5. Haga clic en **Todos los usuarios**.

6. **Busque** el usuario en el que está interesado y **haga clic en la fila** para seleccionarlo.

7. Haga clic en **Grupos**.

8. Compruebe si el usuario forma parte de un grupo al que se ha asignado la aplicación.

   * Si desea quitar el usuario del grupo, **haga clic en la fila** del grupo y seleccione Eliminar.

### <a name="check-if-a-user-is-a-member-of-a-group-assigned-to-a-license"></a>Comprobar si un usuario es miembro de un grupo asignado a una licencia

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Usuarios y grupos** en el menú de navegación.

5. Haga clic en **Todos los usuarios**.

6. **Busque** el usuario en el que está interesado y **haga clic en la fila** para seleccionarlo.

7. Haga clic en **Grupos**.

8. Haga clic en la fila de un grupo específico.

9. Haga clic en **Licencias** para ver qué licencias tiene asignadas el grupo.

   * Si el grupo está asignado a una licencia de Office, esto permitirá que determinadas aplicaciones de Office aparezcan en el panel de acceso del usuario.


## <a name="if-these-troubleshooting-steps-do-not-the-resolve-the-issue"></a>Si después de seguir estos pasos, el problema no se ha resuelto,

abra una incidencia de soporte técnico con la información siguiente si está disponible:

-   Id. de error de correlación

-   UPN (dirección de correo electrónico del usuario)

-   Id. de inquilino

-   Tipo de explorador

-   Zona horaria y hora/período de tiempo durante el que se produce el error

-   Seguimientos de Fiddler

## <a name="next-steps"></a>Pasos siguientes
[Administración de aplicaciones con Azure Active Directory](what-is-application-management.md)
