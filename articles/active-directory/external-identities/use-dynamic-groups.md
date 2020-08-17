---
title: Grupos dinámicos y colaboración B2B - Azure Active Directory | Microsoft Docs
description: Se muestra cómo usar grupos dinámicos de Azure AD con colaboración B2B de Active Directory de Azure
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 02/28/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6adf7e6c5abbba3c018f9a03b5167aec7537c704
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87907622"
---
# <a name="dynamic-groups-and-azure-active-directory-b2b-collaboration"></a>Grupos dinámicos y colaboración B2B de Azure Active Directory

## <a name="what-are-dynamic-groups"></a>¿Qué son los grupos dinámicos?
La configuración dinámica de pertenencia a grupos de seguridad de Azure Active Directory (Azure AD) está disponible en [Azure Portal](https://portal.azure.com). Los administradores pueden establecer reglas para rellenar los grupos que se crean en Azure AD en función de los atributos de usuario (por ejemplo, userType, department o country/region). Los miembros se agregan automáticamente a un grupo de seguridad, o se quitan de este, según sus atributos. Estos grupos pueden proporcionar acceso a aplicaciones o recursos en la nube (documentos y sitios de SharePoint) y para asignar licencias a miembros. Obtenga más información sobre los grupos dinámicos en [Grupos dedicados en Azure Active Directory](../active-directory-accessmanagement-dedicated-groups.md).

La [licencia Azure AD Premium P1 o P2](https://azure.microsoft.com/pricing/details/active-directory/) apropiada se necesita para crear y usar grupos dinámicos. Obtenga más información en el artículo [Creación de reglas de pertenencia dinámica a grupos basadas en atributos en Azure Active Directory](../users-groups-roles/groups-dynamic-membership.md).

## <a name="creating-an-all-users-dynamic-group"></a>Creación del grupo dinámico «Todos los usuarios»
Puede crear un grupo que contenga todos los usuarios de un inquilino mediante una regla de pertenencia. Cuando se agreguen o eliminen usuarios del inquilino en el futuro, la pertenencia al grupo se ajustará automáticamente.

1. Inicie la sesión en [Azure Portal](https://portal.azure.com) con una cuenta a la que se haya asignado el rol de administrador global o administrador de usuarios en la organización.
1. Seleccione **Azure Active Directory**.
2. Seleccione **Administrar**, **Grupos** y, a continuación, **Nuevo grupo**.
1. En la página **Nuevo grupo**, en **Tipo de grupo**, seleccione **Seguridad**. Escriba un **Nombre de grupo** y una **Descripción del grupo** para el nuevo grupo. 
2. En **Tipo de pertenencia**, seleccione **Usuario dinámico** y, a continuación, **Agregar una consulta dinámica**. 
4. Encima del cuadro de texto **Sintaxis de regla**, seleccione **Editar**. En la página **Editar sintaxis de la regla**, escriba la siguiente expresión en el cuadro de texto:

   ```
   user.objectId -ne null
   ```
1. Seleccione **Aceptar**. La regla aparece en el cuadro Sintaxis de regla:

   ![Sintaxis de la regla para el grupo dinámico «Todos los usuarios»](media/use-dynamic-groups/all-user-rule-syntax.png)

1.  Seleccione **Guardar**. En el nuevo grupo dinámico se incluirán ahora los usuarios invitados de B2B y los usuarios miembros.


1. Seleccione **Crear** en la página **Nuevo grupo** para crear el grupo.

## <a name="creating-a-group-of-members-only"></a>Creación de un grupo de miembros solamente

Si desea que el grupo excluya los usuarios invitados e incluya solo los miembros de su inquilino, cree un grupo dinámico tal como se describió anteriormente, pero en el cuadro **Sintaxis de regla**, escriba la siguiente expresión:

```
(user.objectId -ne null) and (user.userType -eq "Member")
```

En la siguiente imagen se muestra la sintaxis de la regla para un grupo dinámico modificado, para que incluya solo miembros y excluya invitados.

![Se muestra una regla en la que el tipo de usuario es igual a miembro](media/use-dynamic-groups/all-member-user-rule-syntax.png)

## <a name="creating-a-group-of-guests-only"></a>Creación de un grupo de invitados solamente

Puede que también encuentre esto útil para crear un grupo dinámico que contenga solo usuarios invitados, de manera que pueda aplicar directivas (como directivas de acceso condicional de Azure AD) para ellos. Cree un grupo dinámico tal como se describió anteriormente, pero en el cuadro **Sintaxis de regla**, escriba la siguiente expresión:

```
(user.objectId -ne null) and (user.userType -eq "Guest")
```

En la siguiente imagen se muestra la sintaxis de la regla de un grupo dinámico modificado, para que incluya solo invitados y excluya miembros.

![Se muestra una regla en la que el tipo de usuario es igual a invitado](media/use-dynamic-groups/all-guest-user-rule-syntax.png)

## <a name="next-steps"></a>Pasos siguientes

- [Propiedades de usuario de la colaboración B2B](user-properties.md)
- [Incorporación de usuarios de colaboración B2B a un rol](add-guest-to-role.md)
- [Acceso condicional para usuarios de colaboración B2B](conditional-access.md)

