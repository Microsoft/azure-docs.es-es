---
title: 'Preguntas frecuentes sobre la solución de problemas de roles asignados a grupos en la nube: Azure Active Directory | Microsoft Docs'
description: Asigne un rol de Azure AD a un grupo al que se pueden asignar roles en Azure Portal, PowerShell o Graph API.
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: article
ms.date: 07/27/2020
ms.author: curtand
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 901815ba40459bd50562e557a0a766474c731ce0
ms.sourcegitcommit: 5f7b75e32222fe20ac68a053d141a0adbd16b347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87475907"
---
# <a name="troubleshooting-roles-assigned-to-cloud-groups"></a>Solución de problemas de roles asignados a grupos en la nube

Estas son algunas preguntas y sugerencias para solucionar problemas comunes para la asignación de roles a grupos en Azure Active Directory (Azure AD).

**P:** soy administrador de grupos, pero no veo el modificador **Los roles de Azure AD se pueden asignar a un grupo**.

**R:** solo los administradores de roles con privilegios o los administradores globales pueden crear un grupo apto para la asignación de roles. Solo los usuarios de esos roles ven este control.

**P:** ¿quién puede modificar la pertenencia de grupos asignados a roles de Azure AD?

**R:** de forma predeterminada, solo los administradores de roles con privilegios y los administradores globales administran la pertenencia de un grupo al que se pueden asignar roles, pero puede delegar la administración de grupos a los que se pueden asignar roles agregando propietarios del grupo.

**P.** : soy administrador del departamento de soporte técnico en mi organización, pero no puedo actualizar la contraseña de un usuario que sea lector del directorio. ¿Por qué ocurre eso?

**R.** : el usuario podría haber conseguido un lector del directorio por medio de un grupo al que se pueden asignar roles. Todos los miembros y propietarios de un grupo al que se pueden asignar roles están protegidos. Solo los usuarios de los roles Administrador de autenticación con privilegios o Administrador global pueden restablecer las credenciales de un usuario protegido.

**P:** no puedo actualizar la contraseña de un usuario. No se les ha asignado ningún rol con privilegios superior. ¿Por qué está sucediendo?

**R:** el usuario podría ser propietario de un grupo al que se pueden asignar roles. Protegemos a los propietarios de grupos a los que se pueden asignar roles para evitar la elevación de privilegios. Un ejemplo podría ser si se asigna un grupo Contoso_Security_Admins al rol Administrador de seguridad, donde Bob es el propietario del grupo y Alice es el administrador de contraseñas de la organización. Si no estuviera presente esta protección, Alice podría restablecer las credenciales de Bob y asumir su identidad. A continuación, Alice podría agregarse a sí misma o a cualquier persona al grupo Contoso_Security_Admins para convertirse en Administrador de seguridad de la organización. Para averiguar si un usuario es el propietario de un grupo, obtenga la lista de objetos pertenecientes a dicho usuario y compruebe si alguno de los grupos tiene isAssignableToRole establecido en true. En caso afirmativo, ese usuario está protegido y el comportamiento es por diseño. Consulte estos documentos para obtener objetos en propiedad:

- [Get-AzureADUserOwnedObject](https://docs.microsoft.com/powershell/module/azuread/get-azureaduserownedobject?view=azureadps-2.0)  
- [Enumeración de ownedObjects](https://docs.microsoft.com/graph/api/user-list-ownedobjects?view=graph-rest-1.0&tabs=http)

**P:** ¿puedo crear una revisión de acceso en grupos que puedan asignarse a roles de Azure AD (en concreto, grupos con la propiedad isAssignableToRole establecida en true)?  

**R:**  Sí, puede hacerlo. Si está en la versión más reciente de la revisión de acceso, se dirige a sus revisores a Mi acceso de forma predeterminada y solo los administradores globales pueden crear revisiones de acceso en grupos asignables del rol. Sin embargo, si está en la versión anterior de la revisión de acceso, se dirige a sus revisores al Panel de acceso de forma predeterminada, tanto los administradores globales como el administrador de usuarios pueden crear revisiones de acceso en grupos asignables del rol. La nueva experiencia se implementará en todos los clientes el 28 de julio de 2020, pero si desea actualizar antes, realice una solicitud en la página de las [revisiones de acceso de Azure AD sobre la experiencia del revisor actualizada en la suscripción a Mi acceso](https://forms.microsoft.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR5dv-S62099HtxdeKIcgO-NUOFJaRDFDWUpHRk8zQ1BWVU1MMTcyQ1FFUi4u).

**P:** ¿puedo crear un paquete de acceso e incluir en él grupos que puedan asignarse a roles de Azure AD?

**R:**  Sí, puede hacerlo. El administrador global y el administrador de usuarios tienen la capacidad de incluir cualquier grupo en un paquete de acceso. En el caso del administrador global, no se produce ningún cambio. Sin embargo, hay un ligero cambio en los permisos del rol Administrador de usuarios. Para incluir un grupo asignable del rol en un paquete de acceso, debe ser administrador de usuarios y también propietario del grupo asignable del rol. Esta es la tabla completa donde se muestra quién puede crear un paquete de acceso en la administración de licencias Enterprise:

Rol de directorio de Azure AD | Rol de administración de derechos | Puede agregar grupos de seguridad\* | Puede agregar grupos de Office 365\* | Puede agregar aplicaciones | Puede agregar sitios de SharePoint Online
----------------------- | --------------------------- | ----------------------- | ------------------------- | ----------- |  -----------------------------
Administrador global | N/D | ✔️ | ✔️ | ✔️  | ✔️
Administrador de usuarios  | N/D  | ✔️  | ✔️  | ✔️
Administrador de Intune | Propietario del catálogo | ✔️  | ✔️  | &nbsp;  | &nbsp;
Administrador de Exchange  | Propietario del catálogo  | &nbsp; | ✔️  | &nbsp;  | &nbsp;
Administrador del servicio Teams | Propietario del catálogo  | &nbsp; | ✔️  | &nbsp;  | &nbsp;
Administrador de SharePoint | Propietario del catálogo | &nbsp; | ✔️  | &nbsp;  | ✔️ 
Administrador de aplicaciones | Propietario del catálogo  | &nbsp;  | &nbsp; | ✔️  | &nbsp;
Administrador de aplicaciones en la nube | Propietario del catálogo  | &nbsp;  | &nbsp; | ✔️  | &nbsp;
Usuario | Propietario del catálogo | Solo si es propietario del grupo | Solo si es propietario del grupo | Solo si es propietario de la aplicación  | &nbsp;

\*Al grupo no se le pueden asignar roles; es decir, isAssignableToRole = false. Si a un grupo se le pueden asignar roles, la persona que crea el paquete de acceso también debe ser propietaria del grupo asignable del rol.

**P:** no encuentro la opción "Quitar asignación" en "Roles asignados". ¿Cómo elimino la asignación de roles a un usuario?

**R:** esta respuesta solo es aplicable a las organizaciones Azure AD Premium P1.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y abra **Azure Active Directory**.
1. Seleccione usuarios y abra un perfil de usuario.
1. Seleccione **Roles asignados**.
1. Seleccione el icono de engranaje. Se abre un panel que puede proporcionar esta información. Hay un botón "Quitar" junto a las asignaciones directas. Para quitar la asignación indirecta de roles, quite el usuario del grupo al que se le ha asignado el rol.

**P:** ¿cómo veo todos los grupos a los que se pueden asignar roles?

**R:** Siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y abra **Azure Active Directory**.
1. Seleccione **Grupos** > **Todos los grupos**.
1. Seleccione **Agregar filtros**.
1. Filtre para **Rol asignable**.

**P:** ¿cómo sabe qué rol se asigna directa o indirectamente a una entidad de seguridad?

**R:** Siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y abra **Azure Active Directory**.
1. Seleccione usuarios y abra un perfil de usuario.
1. Seleccione **Roles asignados** y, a continuación:

    - En las organizaciones con licencia Azure AD Premium P1: seleccione el icono de engranaje. Se abre un panel que puede proporcionar esta información.
    - En las organizaciones con licencia Azure AD Premium P2: encontrará información sobre licencias directas y heredadas en la columna **Pertenencia**.

**P:** ¿por qué exigimos la creación de un nuevo grupo en la nube para asignarlo al rol?  

**R:** si asigna un grupo existente a un rol, el propietario del grupo existente podría agregar otros miembros a este grupo sin saber los nuevos miembros que tendrán el rol. Dado que los grupos a los que se pueden asignar roles son eficaces, ponemos muchas restricciones para protegerlos. No quiere cambios en el grupo inesperados para la persona que administra el grupo.

## <a name="next-steps"></a>Pasos siguientes

- [Uso de grupos en la nube para administrar asignaciones de roles](roles-groups-concept.md)
- [Creación de un grupo al que se pueden asignar roles](roles-groups-create-eligible.md)
