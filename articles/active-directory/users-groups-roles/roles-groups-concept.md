---
title: Uso de grupos en la nube para administrar asignaciones de roles en Azure Active Directory | Microsoft Docs
description: Versión preliminar de los roles personalizados de Azure AD para delegar la administración de identidades. Administre roles de Azure en Azure Portal, PowerShell o Graph API.
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
ms.openlocfilehash: 68d40e9dbc2c4da6b298f7245318c95de983edcc
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87530153"
---
# <a name="use-cloud-groups-to-manage-role-assignments-in-azure-active-directory-preview"></a>Uso de grupos en la nube para administrar asignaciones de roles en Azure Active Directory (versión preliminar)

Azure Active Directory (Azure AD) presenta una versión preliminar pública en la que puede asignar un grupo en la nube a roles integrados de Azure AD. Con esta característica, puede usar grupos para conceder acceso de administrador en Azure AD con un esfuerzo mínimo de los administradores de roles globales y con privilegios.

En este ejemplo: Contoso ha contratado a personas en distintas zonas geográficas para administrar y restablecer las contraseñas de los empleados de su organización de Azure AD. En lugar de solicitar a un administrador de rol con privilegios o a un administrador global que asigne el rol de administrador del departamento de soporte técnico a cada persona individualmente, puede crear un grupo Contoso_Helpdesk_Administrators y asignarle el rol. Cuando los usuarios se unen al grupo, se les asigna el rol de forma indirecta. Después, el flujo de trabajo de gobernanza existente puede encargarse del proceso de aprobación y la auditoría de la pertenencia al grupo para asegurarse de que solo los usuarios legítimos son miembros del grupo, a fin de asignarles el rol de administrador del departamento de soporte técnico.

## <a name="how-this-feature-works"></a>Funcionamiento de esta característica

Cree un grupo de seguridad o de Office 365 con la propiedad "isAssignableToRole" establecida en "true". También puede habilitar esta propiedad al crear un grupo en Azure Portal; para ello, active **Los roles de Azure AD se pueden asignar a un grupo**. En cualquier caso, puede asignar el grupo a uno o varios roles de Azure AD de la misma manera que asigna roles a los usuarios. Se puede crear un máximo de 200 grupos a los que se puedan asignar roles en una sola organización de Azure AD (inquilino).

Si no desea que los miembros del grupo tengan acceso permanente al rol, puede usar Azure AD Privileged Identity Management. Asigne un grupo como miembro apto de un rol de Azure AD. Después, cada miembro del grupo es apto para que se active la asignación del rol al que está asignado el grupo. A continuación, pueden activar su asignación de roles durante un tiempo fijo.

> [!Note]
> Debe tener la versión actualizada de Privileged Identity Management para poder asignar un grupo a un rol de Azure AD mediante PIM. Podría estar en una versión anterior de PIM porque su organización de Azure AD aprovecha la API de Privileged Identity Management. Póngase en contacto con el alias pim_preview@microsoft.com para trasladar la organización y actualizar la API. Obtenga más información en el artículo sobre [roles y características de Azure AD en PIM](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/azure-ad-roles-features).

## <a name="why-we-enforce-creation-of-a-special-group-for-assigning-it-to-a-role"></a>Por qué se exige la creación de un grupo especial para asignarlo a un rol

Si se asigna un rol a un grupo, cualquier administrador de TI que pueda administrar la pertenencia a grupos también podría administrar indirectamente la pertenencia a ese rol. Por ejemplo, supongamos que se asigna un grupo Contoso_User_Administrators al rol de administrador de cuentas de usuario. Un administrador de Exchange que puede modificar la pertenencia a grupos podría agregarse a sí mismo al grupo Contoso_User_Administrators y, de esta forma, convertirse en administrador de cuentas de usuario. Como puede ver, un administrador podría elevar sus privilegios de una forma no prevista.

Azure AD permite proteger un grupo asignado a un rol mediante una nueva propiedad denominada isAssignableToRole para los grupos. Solo los grupos en la nube que tenían la propiedad isAssignableToRole establecida en "true" en el momento de la creación pueden asignarse a un rol. Esta propiedad es inalterable; una vez que se crea un grupo con esta propiedad establecida en "true", ya no se puede modificar. No se puede establecer la propiedad en un grupo existente.
Hemos diseñado cómo se asignan los grupos a los roles para evitar que se produzca esa posible vulneración:

- Solo los administradores globales y los administradores de roles con privilegios pueden crear un grupo al que se pueden asignar roles (con la propiedad "isAssignableToRole" habilitada).
- No puede ser un grupo dinámico de Azure AD; es decir, debe tener un tipo de pertenencia "Asignado". El rellenado automático de grupos dinámicos podría dar lugar a la adición de una cuenta no deseada al grupo y, por tanto, a la asignación del rol.
- De forma predeterminada, solo los administradores de roles con privilegios y los administradores globales puede administrar la pertenencia de un grupo al que se pueden asignar roles, pero puede delegar la administración de grupos a los que se pueden asignar roles mediante la adición de propietarios del grupo.
- Para evitar la elevación de privilegios, solo un administrador de autenticación con privilegios o un administrador global pueden cambiar las credenciales de los miembros y propietarios de un grupo al que se pueden asignar roles.
- Sin anidamiento. No se puede agregar un grupo como miembro de un grupo al que se pueden asignar roles.

## <a name="limitations"></a>Limitaciones

Los siguientes escenarios no se admiten en este momento:  

- Asignación de grupos en la nube a roles personalizados de Azure AD
- Asignación de grupos en la nube a roles de Azure AD (integrados o personalizados) a través de una unidad administrativa
- Asignación de grupos locales a roles de Azure AD (integrados o personalizados)

## <a name="known-issues"></a>Problemas conocidos

- No se puede crear o modificar un grupo dinámico cuando el rol se asigna a través de un grupo.
- La característica **Habilitar el lanzamiento preconfigurado para el inicio de sesión de usuarios administrados** no admite la asignación a través de un grupo.
- *Solo clientes con licencia de Azure AD P2*: no asigne un grupo como activo a un rol a través de Azure AD y Privileged Identity Management. Esto generará problemas en los que los usuarios no puedan ver sus asignaciones de roles activas en PIM, así como la incapacidad de quitar esa asignación de PIM. Las asignaciones válidas no se ven afectadas en este escenario. Si intenta realizar esta asignación, es posible que vea un comportamiento inesperado como el siguiente:
  - Es posible que la hora de finalización de la asignación de roles no se muestre correctamente.
  - En el portal de PIM, en **Mis roles** se puede mostrar solo una asignación de roles, independientemente del número de métodos por los que se concede la asignación (mediante uno o varios grupos y directamente).
- *Solo clientes con licencia de Azure AD P2*: incluso después de eliminar el grupo, aún se muestra un miembro apto del rol en la interfaz de usuario de PIM. Funcionalmente, no hay ningún problema. Se trata simplemente de un problema de caché de Azure Portal.  
- El centro de administración de Exchange todavía no reconoce la pertenencia al rol mediante el grupo, pero funcionará el cmdlet de PowerShell.
- El portal de Azure Information Protection (el portal clásico) todavía no reconoce la pertenencia al rol mediante el grupo. Puede [migrar a la plataforma de etiquetado de confidencialidad unificada](https://docs.microsoft.com/azure/information-protection/configure-policy-migrate-labels) y, a continuación, usar el Centro de seguridad y cumplimiento de Office 365 a fin de utilizar las asignaciones de grupo para administrar roles.

Estamos solucionando estos problemas.

## <a name="required-license-plan"></a>Plan de licencia necesario

El uso de esta característica requiere tener una licencia de Azure AD Premium P1 disponible en la organización de Azure AD. Para usar también Privileged Identity Management en la activación del rol Just-in-Time, es necesario disponer de una licencia de Azure AD Premium P2. Para obtener la licencia correcta para sus requisitos, consulte [Comparación de las características con disponibilidad general de los planes Gratis y Prémium](../fundamentals/active-directory-whatis.md#what-are-the-azure-ad-licenses).

## <a name="next-steps"></a>Pasos siguientes

- [Creación de un grupo al que se pueden asignar roles](roles-groups-create-eligible.md)
- [Asignación de un rol a un grupo al que se pueden asignar roles](roles-groups-assign-role.md)
