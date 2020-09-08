---
title: 'Modificación de los roles de recurso de un paquete de acceso en la administración de derechos de Azure AD: Azure Active Directory'
description: Aprenda a modificar los roles de recurso de un paquete de acceso existente en la administración de derechos de Azure Active Directory.
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 06/18/2020
ms.author: ajburnle
ms.reviewer: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: 56948b700f816c13d35915400658136ffcf48846
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88783593"
---
# <a name="change-resource-roles-for-an-access-package-in-azure-ad-entitlement-management"></a>Modificación de los roles de recurso de un paquete de acceso en la administración de derechos de Azure AD

Como administrador de paquetes de acceso, puede cambiar los recursos de un paquete de acceso en cualquier momento sin preocuparse de aprovisionar el acceso del usuario a los nuevos recursos o quitárselo de los recursos anteriores. En este artículo, se explica cómo se pueden modificar los roles de recursos de un paquete de acceso existente.

Este vídeo proporciona información general acerca de cómo cambiar un paquete de acceso.

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE3LD4Z]

## <a name="check-catalog-for-resources"></a>Comprobación del catálogo de recursos

Si necesita agregar recursos a un paquete de acceso, debe comprobar si los recursos que necesita están disponibles en el catálogo. Si es un administrador de paquetes de acceso, no podrá agregar recursos a un catálogo, ni siquiera aunque sea el propietario. Está limitado a usar los recursos disponibles en el catálogo.

**Rol necesario:** Administrador global, administrador de usuarios, propietario del catálogo o administrador de paquetes de acceso.

1. En Azure Portal, haga clic en **Azure Active Directory** y, luego, en **Gobernanza de identidades**.

1. En el menú de la izquierda, haga clic en **Catálogo** y abra el catálogo.

1. En el menú de la izquierda, haga clic en **Recursos** para ver la lista de recursos de este catálogo.

    ![Lista de recursos de un catálogo](./media/entitlement-management-access-package-resources/catalog-resources.png)

1. Si es un administrador de paquetes de acceso y necesita agregar recursos al catálogo, puede solicitar al propietario del catálogo que los agregue.

## <a name="add-resource-roles"></a>Agregar roles de recursos

Un rol de recursos es una colección de permisos asociados a un recurso. Para hacer que los usuarios puedan solicitar los recursos es necesario agregar roles de recursos al paquete de acceso. Puede agregar roles de recursos de grupos, equipos, aplicaciones y sitios de SharePoint.

**Rol necesario:** Administrador global, administrador de usuarios, propietario del catálogo o administrador de paquetes de acceso.

1. En Azure Portal, haga clic en **Azure Active Directory** y, luego, en **Gobernanza de identidades**.

1. En el menú de la izquierda, haga clic en **Paquetes de acceso** y luego abra el paquete de acceso.

1. En el menú de la izquierda, haga clic en **Roles de recursos**.

1. Haga clic en **Agregar roles de recursos** para abrir la página Agregar roles de recursos al paquete de acceso.

    ![Paquete de acceso - Agregar roles de recursos](./media/entitlement-management-access-package-resources/resource-roles-add.png)

1. En función de si lo que quiere agregar es un grupo, un equipo, una aplicación o un sitio de SharePoint, siga los pasos de una de las siguientes secciones de roles de recursos.

## <a name="add-a-group-or-team-resource-role"></a>Incorporación de un rol de recurso de un grupo o equipo

Puede hacer que la administración de derechos agregue usuarios automáticamente a un grupo o equipo de Microsoft Teams cuando se le asigne un paquete de acceso. 

- Cuando un grupo o equipo forma parte de un paquete de acceso y se asigna un usuario a dicho paquete de acceso, el usuario se agrega a ese grupo o equipo, si aún no está presente.
- Cuando la asignación de paquetes de acceso de un usuario expira, se le quita del grupo o equipo, a menos que en ese momento tenga una asignación a otro paquete de acceso que incluya ese mismo grupo o equipo.

Puede seleccionar cualquier [grupo de seguridad de Azure AD o grupo de Office 365](../fundamentals/active-directory-groups-create-azure-portal.md). Los administradores pueden agregar cualquier grupo a un catálogo; los propietarios de catálogos pueden agregar cualquier grupo al catálogo si son los propietarios del grupo. Tenga en cuenta las siguientes restricciones de Azure AD al seleccionar un grupo:

- Cuando un usuario, incluidos los usuarios invitados, se agrega como miembro a un grupo o equipo, puede ver a los demás miembros de ese grupo o equipo.
- Azure AD no puede cambiar la pertenencia de un grupo que se ha sincronizado desde Windows Server Active Directory con Azure AD Connect o que se ha creado en Exchange Online como grupo de distribución.  
- No se puede actualizar la pertenencia a grupos dinámicos agregando o eliminando a un miembro, por lo que las pertenencias a grupos dinámicos no son adecuadas para su uso con la administración de derechos.

Para obtener más información, consulte [Comparar grupos](/office365/admin/create-groups/compare-groups) y [Grupos de Office 365 y equipos de Microsoft](/microsoftteams/office-365-groups).

1. En la página **Agregar roles de recursos al paquete de acceso**, haga clic en **Grupos y Equipos** para abrir el panel Seleccionar grupos.

1. Seleccione los grupos y equipos que desee incluir en el paquete de acceso.

    ![Paquete de acceso - Agregar roles de recursos - Seleccionar grupos](./media/entitlement-management-access-package-resources/group-select.png)

1. Haga clic en **Seleccionar**.

    Una vez que seleccione el grupo o equipo, aparecerá uno de los siguientes valores en la columna **Subtipo**:

    | Subtipo | Descripción |
    | --- | --- |
    | Seguridad | Se usa para conceder acceso a los recursos de Azure. |
    | Distribución | Se usa para enviar notificaciones a un grupo de personas. |
    | O365 | Grupo de Office 365 que no está habilitado para Teams. Se usa para la colaboración entre usuarios, tanto dentro como fuera de su empresa. |
    | Team | Grupo de Office 365 que sí está habilitado para Teams. Se usa para la colaboración entre usuarios, tanto dentro como fuera de su empresa. |

1. En la lista **Rol**, seleccione **Propietario** o **Miembro**.

    Normalmente se selecciona el rol Miembro. Si selecciona el rol Propietario, permitirá que los usuarios agreguen o quiten a otros miembros o propietarios.

    ![Paquete de acceso: agregar un rol de recurso para un grupo o equipo](./media/entitlement-management-access-package-resources/group-role.png)

1. Haga clic en **Agregar**.

    Cuando se agreguen, los usuarios que tengan asignaciones existentes al paquete de acceso se convertirán automáticamente en miembros de este grupo o equipo.

## <a name="add-an-application-resource-role"></a>Agregar un rol de recursos de la aplicación

Puede hacer que Azure AD asigne automáticamente a los usuarios acceso a una aplicación empresarial de Azure AD, incluidas tanto las aplicaciones SaaS como las aplicaciones de su organización federadas con Azure AD, cuando se asigne a un usuario un paquete de acceso. Para las aplicaciones que se integran con Azure AD a través de un inicio de sesión único federado, Azure AD emitirá los tokens de federación para los usuarios asignados a la aplicación.

Las aplicaciones pueden tener varios roles. Al agregar una aplicación a un paquete de acceso, si esa aplicación tiene más de un rol, deberá especificar el rol adecuado para esos usuarios. Si va a desarrollar aplicaciones, puede consultar más información acerca de cómo se agregan estos roles a las aplicaciones en [Procedimientos para: Configuración de la notificación de rol emitida en el token SAML para aplicaciones empresariales](../develop/active-directory-enterprise-app-role-management.md).

Una vez que un rol de aplicación es parte de un paquete de acceso:

- Cuando se asigna un usuario a ese paquete de acceso, el usuario se agrega a ese rol de aplicación, si todavía no está presente.
- Cuando la asignación de paquetes de acceso de un usuario expira, se le quitará el acceso a la aplicación, a menos que tenga una asignación a otro paquete de acceso que incluya ese mismo rol de aplicación.

Estas son algunas opciones a tener en cuenta cuando seleccione una aplicación:

- Las aplicaciones también pueden tener, además, grupos asignados a sus roles.  Puede elegir agregar un grupo en lugar de un rol de aplicación en un paquete de acceso, pero entonces la aplicación no será visible para el usuario como parte del paquete de acceso en el portal Mi acceso.

1. En la página **Agregar roles de recursos al paquete de acceso**, haga clic en **Aplicaciones** para abrir el panel Seleccionar aplicaciones.

1. Seleccione las aplicaciones que quiere incluir en el paquete de acceso.

    ![Paquete de acceso - Agregar roles de recursos - Seleccionar aplicaciones](./media/entitlement-management-access-package-resources/application-select.png)

1. Haga clic en **Seleccionar**.

1. En la lista **Rol**, seleccione un rol de aplicación.

    ![Paquete de acceso - Agregar rol de recursos para una aplicación](./media/entitlement-management-access-package-resources/application-role.png)

1. Haga clic en **Agregar**.

    Cuando se agregue, se otorgará acceso a esta aplicación a los usuarios con asignaciones existentes para el paquete de acceso.

## <a name="add-a-sharepoint-site-resource-role"></a>Agregar un rol de recursos de sitio de SharePoint

Azure AD puede asignar automáticamente a los usuarios acceso a un sitio de SharePoint Online o una colección de sitios de SharePoint Online cuando se asignan a un paquete de acceso.

1. En la página **Agregar roles de recursos a un paquete de acceso**, haga clic en **Sitios de SharePoint** para abrir el panel Select SharePoint Online sites (Seleccionar sitios de SharePoint Online).

1. Seleccione los sitios de SharePoint Online que quiere incluir en el paquete de acceso.

    ![Paquete de acceso - Agregar roles de recursos - Select SharePoint Online sites (Seleccionar sitios de SharePoint Online)](./media/entitlement-management-access-package-resources/sharepoint-site-select.png)

1. Haga clic en **Seleccionar**.

1. En la lista **Rol**, seleccione un rol de sitio de SharePoint Online.

    ![Paquete de acceso - Agregar un rol de recursos para un sitio de SharePoint Online](./media/entitlement-management-access-package-resources/sharepoint-site-role.png)

1. Haga clic en **Agregar**.

    Cuando se agregue, se otorgará acceso a este sitio de SharePoint Online a los usuarios con asignaciones existentes para el paquete de acceso.

## <a name="remove-resource-roles"></a>Eliminar roles de recursos

**Rol necesario:** Administrador global, administrador de usuarios, propietario del catálogo o administrador de paquetes de acceso.

1. En Azure Portal, haga clic en **Azure Active Directory** y, luego, en **Gobernanza de identidades**.

1. En el menú de la izquierda, haga clic en **Paquetes de acceso** y luego abra el paquete de acceso.

1. En el menú de la izquierda, haga clic en **Roles de recursos**.

1. En la lista de roles de recursos, busque el rol de recurso que quiere quitar.

1. Haga clic en el botón de puntos suspensivos **...** y luego en **Eliminar rol de recurso**.

    Cuando se elimine, se revocará automáticamente el acceso a este rol de recurso de los usuarios con asignaciones existentes para el paquete de acceso.

## <a name="when-changes-are-applied"></a>Cuándo se aplican los cambios

En administración de derechos, Azure AD procesará los cambios de forma masiva para la asignación y los recursos de los paquetes de acceso varias veces al día. Por lo tanto, si realiza una asignación o cambia los roles de recursos del paquete de acceso, puede tardar hasta 24 horas para que dicho cambio se lleve a cabo en Azure AD, además del tiempo que se tarde en propagar los cambios a otras aplicaciones SaaS conectadas o Microsoft Online Services. Si el cambio solo afecta a unos pocos objetos, probablemente tardará solo unos minutos en aplicarse en Azure AD, tras lo cual otros componentes de Azure AD detectarán dicho cambio y actualizarán las aplicaciones SaaS. Si el cambio afecta a miles de objetos tardará más tiempo. Por ejemplo, si tiene un paquete de acceso con 2 aplicaciones y 100 asignaciones de usuario y decide agregar un rol de sitio de SharePoint al paquete de acceso, podría haber un retraso hasta que todos los usuarios formen parte de ese rol de sitio de SharePoint. Puede supervisar el progreso a través de los registros de auditoría de Azure AD, el registro de aprovisionamiento de Azure AD y los registros de auditoría del sitio de SharePoint.

Cuando se quita un miembro de un equipo, también se quita del grupo de Office 365. Puede pasar algún tiempo hasta que se elimine la funcionalidad de chat del equipo. Para más información, consulte [Pertenencia a grupos](/microsoftteams/office-365-groups#group-membership).

## <a name="next-steps"></a>Pasos siguientes

- [Creación de un grupo básico e incorporación de miembros con Azure Active Directory](../fundamentals/active-directory-groups-create-azure-portal.md)
- [Cómo: Configuración de la notificación de rol emitida en el token SAML para aplicaciones empresariales](../develop/active-directory-enterprise-app-role-management.md)
- [Introducción a SharePoint Online](/sharepoint/introduction)