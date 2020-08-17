---
title: Creación de un grupo básico e incorporación de miembros con Azure Active Directory | Microsoft Docs
description: Aprenda a crear un grupo básico con Azure Active Directory.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: how-to
ms.date: 06/05/2020
ms.author: ajburnle
ms.reviewer: krbain
ms.custom: it-pro, seodec18, contperfq4
ms.collection: M365-identity-device-management
ms.openlocfilehash: 51b257eb833f6c270f2f1b79b2cccd247c8a2d42
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87797668"
---
# <a name="create-a-basic-group-and-add-members-using-azure-active-directory"></a>Creación de un grupo básico e incorporación de miembros con Azure Active Directory
Puede crear un grupo básico con el portal de Azure Active Directory (Azure AD). Para los fines de este artículo, el propietario del recurso (administrador) agrega un grupo básico a un único recurso e incluye miembros específicos (empleados) que necesitan acceder a dicho recurso. Para escenarios más complejos, incluida la creación de reglas y las pertenencias dinámicas, vea la [documentación de administración de usuarios de Azure Active Directory](../users-groups-roles/index.yml).

## <a name="group-and-membership-types"></a>Tipos de grupo y pertenencia
Hay varios tipos de grupo y de pertenencia. La siguiente información explica cada tipo de grupo y pertenencia, así como el motivo por el que se usan, para ayudarle a decidir las opciones que usará al crear un grupo.

### <a name="group-types"></a>Tipos de grupo:
- **Seguridad**. Se usa para administrar el acceso de miembros y del equipo a los recursos compartidos de un grupo de usuarios. Por ejemplo, puede crear un grupo de seguridad para una directiva de seguridad específica. De esta forma, puede conceder una serie de permisos a todos los miembros a la vez, en lugar de tener que agregar permisos a cada miembro individualmente. Un grupo de seguridad puede tener usuarios, dispositivos, grupos y entidades de servicio como miembros y usuarios y entidades de servicio como propietarios. Para más información sobre la administración de acceso a los recursos, vea [Administración de acceso a los recursos con grupos de Azure Active Directory](active-directory-manage-groups.md).
- **Office 365**. Ofrece oportunidades de colaboración al conceder acceso a los miembros a un correo compartido, calendarios, archivos, el sitio de SharePoint y mucho más. Esta opción también permite ofrecer a personas de fuera de su organización acceso al grupo. Un grupo de Office 365 solo puede tener usuarios como miembros. Tanto los usuarios como las entidades de servicio pueden ser propietarios de registros de Office 365. Para más información sobre los Grupos de Office 365, vea [Obtenga más información sobre los grupos de Office 365](https://support.office.com/article/learn-about-office-365-groups-b565caa1-5c40-40ef-9915-60fdb2d97fa2).

### <a name="membership-types"></a>Tipos de pertenencia:
- **Asignado.** Le permite agregar usuarios específicos para que sean miembros de este grupo y para que tengan permisos exclusivos. Para los fines de este artículo, vamos a usar esta opción.
- **Usuario dinámico.** Permite usar reglas de pertenencia dinámicas para agregar y quitar miembros automáticamente. Si los atributos de un miembro cambian, el sistema examina las reglas del grupo dinámico del directorio para ver si el miembro cumple los requisitos de la regla (se agrega) o ya no cumple los requisitos de las reglas (se elimina).
- **Dispositivo dinámico.** Le permite usar reglas de grupo dinámico para agregar y quitar dispositivos automáticamente. Si los atributos de un dispositivo cambian, el sistema examina las reglas del grupo dinámico del directorio para ver si el dispositivo cumple los requisitos de la regla (se agrega) o ya no cumple los requisitos de las reglas (se elimina).

    > [!IMPORTANT]
    > Puede crear un grupo dinámico para dispositivos o usuarios, pero no para ambos. Tampoco se puede crear un grupo de dispositivos basado en los atributos de los propietarios de los dispositivos. Las reglas de pertenencia de dispositivo solo pueden hacer referencia a atribuciones de dispositivos. Para más información sobre cómo crear un grupo dinámico para usuarios y dispositivos, consulte [Creación de un grupo dinámico y comprobación de su estado](../users-groups-roles/groups-create-rule.md).

## <a name="create-a-basic-group-and-add-members"></a>Creación de un grupo básico y adición de miembros
Puede crear un grupo básico y agregar los miembros al mismo tiempo. Para crear un grupo básico y agregar miembros, use el procedimiento siguiente:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta de administrador global para el directorio.

1. Busque y seleccione **Azure Active Directory**.

1. En la página **Active Directory**, seleccione **Grupos** y, a continuación, seleccione **Nuevo grupo**.

    ![Página de Azure AD en la que se muestra Grupos](media/active-directory-groups-create-azure-portal/group-full-screen.png)

1. Aparecerá el panel **Nuevo grupo** y deberá rellenar la información necesaria.

    ![Página del grupo nuevo, rellenada con información de ejemplo](media/active-directory-groups-create-azure-portal/new-group-blade.png)

1. Seleccione un valor de **Tipo de grupo** predefinido. Para obtener más información sobre los tipos de grupo, consulte [Tipos de grupo y pertenencia](#group-types).

1. Cree y agregue un **nombre de grupo.** Agregue un nombre que sea fácil de recordar y que tenga sentido para el grupo. Se realizará una comprobación para determinar si el nombre ya se utiliza para otro grupo. Si el nombre ya está en uso, para evitar duplicarlo, se le pedirá que modifique el nombre del grupo.

1. Agregue una **Dirección de correo electrónico de grupo** o deje la dirección de correo electrónico que se rellena automáticamente.

1. **Descripción del grupo.** Agregue una descripción opcional al grupo.

1. Seleccione un **Tipo de pertenencia** (obligatorio) predefinido. Para obtener más información sobre los tipos de pertenencia, consulte [Tipos de grupo y pertenencia](#membership-types).

1. Seleccione **Crear**. El grupo se crea y está listo para que agregue miembros.

1. Seleccione el área **Miembros** de la página **Grupo** y después empiece a buscar los miembros para agregarlos al grupo en la página **Seleccionar miembros**.

    ![Selección de miembros del grupo durante el proceso de creación del grupo](media/active-directory-groups-create-azure-portal/select-members-create-group.png)

1. Cuando haya terminado de agregar miembros, elija **Seleccionar**.

    La página **Información general del grupo** se actualiza para mostrar el número de miembros que ahora se agregan al grupo.

    ![Página Información general del grupo con el número de miembros resaltado](media/active-directory-groups-create-azure-portal/group-overview-blade-number-highlight.png)

## <a name="turn-on-or-off-group-welcome-email"></a>Activación o desactivación del correo electrónico de bienvenida al grupo

Siempre que se crea un grupo de Office 365, independientemente de que la pertenencia sea estática o dinámica, se envía una notificación de bienvenida a todos los usuarios que se agregan al grupo. Cuando cambian los atributos de un usuario o dispositivo, se procesan todas las reglas de grupo dinámico de la organización para comprobar si hay posibles cambios de pertenencia. Los usuarios que se agregan también reciben la notificación de bienvenida. Este comportamiento se puede desactivar en [Exchange PowerShell](https://docs.microsoft.com/powershell/module/exchange/users-and-groups/Set-UnifiedGroup?view=exchange-ps). 

## <a name="next-steps"></a>Pasos siguientes

- [Administrar el acceso a las aplicaciones SaaS mediante grupos](../users-groups-roles/groups-saasapps.md)
- [Administrar grupos mediante los comandos de PowerShell](../users-groups-roles/groups-settings-v2-cmdlets.md)

