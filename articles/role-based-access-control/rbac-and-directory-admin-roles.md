---
title: Roles de administrador de la suscripción clásica, roles de Azure y roles de Azure AD
description: 'Describe los diferentes roles de Azure: roles de administrador de la suscripción clásica, roles de Azure y roles de Azure Active Directory (Azure AD)'
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.assetid: 174f1706-b959-4230-9a75-bf651227ebf6
ms.service: role-based-access-control
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.date: 07/07/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.custom: it-pro;
ms.openlocfilehash: 8809b5fdce382b7c8f913823ca8e176226d5837b
ms.sourcegitcommit: c52e50ea04dfb8d4da0e18735477b80cafccc2cf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/08/2020
ms.locfileid: "89536181"
---
# <a name="classic-subscription-administrator-roles-azure-roles-and-azure-ad-roles"></a>Roles de administrador de la suscripción clásica, roles de Azure y roles de Azure AD

Si no está familiarizado con Azure, le resultará un poco difícil de entender todos los distintos roles en Azure. En este artículo se explican los siguientes roles y cuándo se usa cada uno:
- Roles de administrador de suscripciones clásicas
- Roles de Azure
- Roles de Azure Active Directory (Azure AD)

## <a name="how-the-roles-are-related"></a>¿Cómo están relacionados los roles?

Para entender mejor los roles en Azure, es útil conocer algo de la historia. Cuando se publicó inicialmente Azure, el acceso a los recursos se administraba con solo tres roles de administrador: administrador de cuentas, administrador de servicios y coadministrador. Posteriormente se agregó el control de acceso basado en rol de Azure (Azure RBAC). RBAC de Azure es un nuevo sistema de autorización que proporciona una administración de acceso detallada a los recursos de Azure. Azure RBAC incluye muchos roles integrados, puede asignarse en distintos ámbitos y le permite crear sus propios roles personalizados. Para administrar recursos de Azure AD, como usuarios, grupos y dominios, hay varios roles de Azure AD.

En el siguiente diagrama se muestra una visión general de cómo se relacionan los roles de administrador de la suscripción clásica, los roles de Azure y los roles de Azure AD.

![Los distintos roles en Azure](./media/rbac-and-directory-admin-roles/rbac-admin-roles.png)


## <a name="classic-subscription-administrator-roles"></a>Roles de administrador de suscripciones clásicas

Administrador de cuenta, administrador de servicios y coadministrador son las tres variantes de roles de administrador de suscripciones clásicas de Azure. Los administradores de suscripción clásica tienen acceso completo a la suscripción de Azure. Se pueden administrar los recursos con Azure Portal, con las API de Azure Resource Manager o bien con las API del modelo de implementación clásica. La cuenta que se utiliza para suscribirse a Azure se establece automáticamente como administrador de cuenta y administrador de servicio. A continuación, se pueden agregar coadministradores adicionales. Tanto el administrador de servicios como los coadministradores tienen un acceso equivalente al de los usuarios a los que se les ha asignado el rol Propietario (un rol de Azure) en el ámbito de la suscripción. En la tabla siguiente se describen las diferencias entre estos tres roles administrativos de la suscripción clásica.

| Administrador de suscripciones clásicas | Límite | Permisos | Notas |
| --- | --- | --- | --- |
| Administrador de cuenta | 1 por cuenta de Azure | <ul><li>Administración de la facturación en [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)</li><li>Administrar todas las suscripciones en una cuenta</li><li>Crear nuevas suscripciones</li><li>Cancelar suscripciones</li><li>Cambiar la facturación de una suscripción</li><li>Cambiar el administrador de servicios</li></ul> | Desde un punto de vista conceptual, el propietario de la facturación de la suscripción.<br>El administrador de cuenta no tiene acceso a Azure Portal. |
| Administrador de servicios | 1 por cada suscripción de Azure | <ul><li>Administrar servicios en [Azure Portal](https://portal.azure.com)</li><li>Cancelación de la suscripción</li><li>Asignar a usuarios al rol de coadministrador</li></ul> | De forma predeterminada, en una nueva suscripción, el administrador de cuenta es también el administrador de servicios.<br>El administrador de servicios tiene el acceso equivalente a un usuario al que se le asigna el rol de propietario en el ámbito de la suscripción.<br>El administrador de servicios tiene permiso total de acceso a Azure Portal. |
| Coadministrador | 200 por suscripción | <ul><li>Mismos privilegios de acceso que el administrador de servicios, pero no puede cambiar la asociación de suscripciones a directorios de Azure</li><li>Asignar usuarios al rol de coadministrador, pero no puede cambiar el administrador de servicios</li></ul> | El coadministrador tiene el acceso equivalente a un usuario al que se le asigna el rol de propietario en el ámbito de la suscripción. |

En Azure Portal, puede administrar coadministradores o ver el administrador de servicios mediante la pestaña **Administradores clásicos**.

![Administradores clásicos de la suscripción de Azure en Azure Portal](./media/rbac-and-directory-admin-roles/subscription-view-classic-administrators.png)

En Azure Portal, puede ver o cambiar el administrador de servicios o ver el administrador de cuenta en la hoja de propiedades de la suscripción.

![Administrador de cuenta y administrador de servicios en Azure Portal](./media/rbac-and-directory-admin-roles/account-admin.png)

Para más información, consulte el artículo sobre los [Administradores clásicos de la suscripción de Azure](classic-administrators.md).

### <a name="azure-account-and-azure-subscriptions"></a>Cuenta de Azure y suscripciones de Azure

Una cuenta de Azure representa una relación de facturación. Una cuenta de Azure es una identidad de usuario, una o varias suscripciones de Azure y un conjunto asociado de recursos de Azure. La persona que crea la cuenta es el administrador de cuenta para todas las suscripciones creadas en esa cuenta. Esa persona también es el administrador de servicios predeterminado de la suscripción.

Las suscripciones de Azure le ayudan a organizar el acceso a los recursos de Azure. También le ayudan a controlar cómo se informa, factura y paga el uso de recursos. Cada suscripción puede tener una configuración de facturación y pago diferente, por lo que puede tener varias suscripciones y planes diferentes por oficina, departamento o proyecto, entre otros. Cada servicio pertenece a una suscripción y el identificador de la suscripción puede ser necesario para las operaciones de programación.

Cada suscripción está asociada a un directorio de Azure AD. Para encontrar el directorio al que está asociada la suscripción, abra **Suscripciones** en Azure Portal y, a continuación, seleccione una suscripción para ver el directorio.

Tanto las cuentas como las suscripciones se administran en [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade).

## <a name="azure-roles"></a>Roles de Azure

RBAC de Azure es un sistema de autorización basado en [Azure Resource Manager](../azure-resource-manager/management/overview.md) que proporciona administración de acceso específico a los recursos de Azure como, por ejemplo, proceso y almacenamiento. RBAC de Azure incluye más de 70 roles integrados. Existen cuatro roles de Azure fundamentales. Las tres primeras se aplican a todos los tipos de recursos:

| Rol de Azure | Permisos | Notas |
| --- | --- | --- |
| [Propietario](built-in-roles.md#owner) | <ul><li>Acceso total a todos los recursos</li><li>Delegar el acceso a otros usuarios</li></ul> | Al administrador de servicios y a los coadministradores se les asigna el rol de propietario en el ámbito de suscripción.<br>Se aplica a todos los tipos de recursos. |
| [Colaborador](built-in-roles.md#contributor) | <ul><li>Crear y administrar todos los tipos de recursos de Azure</li><li>Creación de un inquilino en Azure Active Directory</li><li>No se puede conceder acceso a otros usuarios</li></ul> | Se aplica a todos los tipos de recursos. |
| [Lector](built-in-roles.md#reader) | <ul><li>Ver recursos de Azure</li></ul> | Se aplica a todos los tipos de recursos. |
| [Administrador de acceso de usuario](built-in-roles.md#user-access-administrator) | <ul><li>Administrar el acceso de usuarios a los recursos de Azure</li></ul> |  |

Los demás roles integrados permiten la administración de recursos específicos de Azure. Por ejemplo, el rol de [colaborador de máquina virtual](built-in-roles.md#virtual-machine-contributor) permite al usuario crear y administrar máquinas virtuales. Para ver una lista de todos los roles integrados, consulte [Roles integrados de Azure](built-in-roles.md).

Azure RBAC solo es compatible con Azure Portal y con las API de Azure Resource Manager. Los usuarios, los grupos y las aplicaciones a los que se asignan roles de Azure no pueden usar las [API del modelo de implementación clásica de Azure](../azure-resource-manager/management/deployment-models.md).

En Azure Portal, las asignaciones de roles mediante Azure RBAC aparecen en la hoja **Control de acceso (IAM)** . Esta hoja se puede encontrar en todo el portal, como en grupos de administración, suscripciones, grupos de recursos y distintos recursos.

![Hoja de control de acceso (IAM) en Azure Portal](./media/rbac-and-directory-admin-roles/access-control-role-assignments.png)

Al hacer clic en la pestaña **Roles**, verá la lista de roles integrados y personalizados.

![Roles integrados en Azure Portal](./media/rbac-and-directory-admin-roles/roles-list.png)

Para más información, consulte [Incorporación o eliminación de asignaciones de roles mediante Azure Portal](role-assignments-portal.md).

## <a name="azure-ad-roles"></a>Roles de Azure AD

Los roles de Azure AD se utilizan para administrar los recursos de Azure AD en un directorio como crear o editar usuarios, asignar roles administrativos a otros usuarios, restablecer contraseñas de usuario, administrar licencias de usuario y administrar dominios. En la tabla siguiente se describen algunos de los roles de Azure AD más importantes.

| Rol de Azure AD | Permisos | Notas |
| --- | --- | --- |
| [Administrador global](../active-directory/users-groups-roles/directory-assign-admin-roles.md#company-administrator-permissions) | <ul><li>Administrar el acceso a todas las características administrativas en Azure Active Directory, así como los servicios que se federan con Azure Active Directory</li><li>Asignar roles de administrador a otros usuarios</li><li>Restablecer la contraseña de cualquier usuario y de todos los demás administradores</li></ul> | La persona que se suscribe al inquilino de Azure Active Directory se convierte en un administrador global. |
| [Administrador de usuarios](../active-directory/users-groups-roles/directory-assign-admin-roles.md#user-administrator) | <ul><li>Crear y administrar todos los aspectos de usuarios y grupos</li><li>Administrar incidencias de soporte técnico</li><li>Supervisar el estado del servicio</li><li>Cambiar las contraseñas de los usuarios, de los administradores del departamento de soporte técnico y de otros administradores de usuario</li></ul> |  |
| [Administrador de facturación](../active-directory/users-groups-roles/directory-assign-admin-roles.md#billing-administrator) | <ul><li>Realizar compras</li><li>Administrar suscripciones</li><li>Administrar incidencias de soporte técnico</li><li>Supervisa el mantenimiento del servicio</li></ul> |  |

En Azure Portal, puede ver la lista de roles de Azure AD en la hoja **Roles y administradores**. Para ver una lista de todos los roles de Azure AD, consulte [Permisos de roles de administrador en Azure Active Directory](../active-directory/users-groups-roles/directory-assign-admin-roles.md).

![Roles de Azure AD en Azure Portal](./media/rbac-and-directory-admin-roles/directory-admin-roles.png)

## <a name="differences-between-azure-roles-and-azure-ad-roles"></a>Diferencias entre los roles de Azure y los de Azure AD

En un nivel superior, los roles de Azure controlan los permisos para administrar recursos de Azure, mientras que los roles de Azure AD controlan los permisos para administrar los recursos de Azure Active Directory. En la tabla siguiente se comparan algunas de las diferencias.

| Roles de Azure | Roles de Azure AD |
| --- | --- |
| Administración del acceso de usuarios a los recursos de Azure | Administrar el acceso a recursos de Azure Active Directory |
| Admite los roles personalizados | Admite los roles personalizados |
| El ámbito se puede especificar en varios niveles (grupo de administración, suscripción, grupo de recursos, recurso). | El ámbito está en el nivel de inquilino |
| Se puede acceder a la información de roles en Azure Portal, CLI de Azure, Azure PowerShell, plantillas de Azure Resource Manager, API REST | Se puede acceder a la información de roles en el portal de administración de Azure, el centro de administración de Microsoft 365, Microsoft Graph y AzureAD PowerShell |

### <a name="do-azure-roles-and-azure-ad-roles-overlap"></a>¿Se superponen los roles de Azure y los de Azure AD?

De forma predeterminada, los roles de Azure y los de Azure AD no abarcan Azure y Azure AD. Sin embargo, si un administrador global eleva su acceso mediante la elección del modificador **Administración del acceso para los recursos de Azure** en Azure Portal, se le otorgará el rol [Administrador de acceso de usuario](built-in-roles.md#user-access-administrator) (un rol de Azure) en todas las suscripciones para un inquilino en particular. El rol de administrador de accesos de usuario permite conceder a otros usuarios acceso a recursos de Azure. Este modificador puede ser útil para recuperar el acceso a una suscripción. Para más información, consulte [Elevación de los privilegios de acceso para administrar todas las suscripciones y los grupos de administración de Azure](elevate-access-global-admin.md).

Varios roles de Azure AD abarcan Azure AD y Microsoft Office 365, como los roles de administrador global y administrador de usuarios. Por ejemplo, si es miembro del rol de administrador global, tiene funcionalidades de administrador global en Azure AD y Office 365, como realizar cambios en Microsoft Exchange y Microsoft SharePoint. Sin embargo, de forma predeterminada, el administrador global no tiene acceso a los recursos de Azure.

![Los roles de Azure RBAC frente a los de Azure AD](./media/rbac-and-directory-admin-roles/azure-office-roles.png)

## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es el control de acceso basado en rol de Azure (RBAC)?](overview.md)
- [Permisos de roles de administrador en Azure Active Directory](../active-directory/users-groups-roles/directory-assign-admin-roles.md)
- [Azure classic subscription administrators](classic-administrators.md) (Administradores clásicos de la suscripción de Azure)
