---
title: 'Administración de unidades administrativas (versión preliminar): Azure AD | Microsoft Docs'
description: Uso de unidades administrativas para una delegación más detallada de permisos en Azure Active Directory
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.topic: overview
ms.subservice: users-groups-roles
ms.workload: identity
ms.date: 07/27/2020
ms.author: curtand
ms.reviewer: elkuzmen
ms.custom: oldportal;it-pro;
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5854ef57a08218fee4479eb9e075824f4a1ff781
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89376390"
---
# <a name="administrative-units-management-in-azure-active-directory-preview"></a>Administración de unidades administrativas en Azure Active Directory (versión preliminar)

En este artículo se describen las unidades administrativas en Azure Active Directory (Azure AD). Una unidad administrativa es un recurso de Azure AD que puede ser un contenedor para otros recursos de Azure AD. En esta versión preliminar, una unidad administrativa solo puede contener usuarios y grupos.

Las unidades administrativas permiten conceder permisos de administrador que están restringidos a un departamento, a una región o a otro segmento de la organización que se defina. Puede usar las unidades administrativas para delegar permisos en los administradores regionales o establecer directivas de forma pormenorizada. Por ejemplo, un administrador de cuentas de usuario podría actualizar la información del perfil, restablecer las contraseñas y asignar las licencias para los usuarios solo en su unidad administrativa.

Por ejemplo, delegar en especialistas de soporte técnico regionales el rol [Administrador del departamento de soporte técnico](directory-assign-admin-roles.md#helpdesk-administrator), restringido solo a administrar a los usuarios de la región en cuestión.

## <a name="deployment-scenario"></a>Escenario de implementación

Restringir el ámbito administrativo a través de las unidades administrativas puede resultar útil en organizaciones compuestas de divisiones independientes de cualquier tipo. Considere el ejemplo de una gran universidad que se compone de muchas escuelas autónomas (escuela de negocios, escuela de ingeniería, etc.), donde cada una de las cuales cuenta con un equipo de administradores de TI, que controlan el acceso, administran los usuarios y establecen las directivas para su escuela. Un administrador central podría:

- Crear un rol con permisos administrativos solo sobre los usuarios de Azure AD en la unidad administrativa de la escuela de negocios.
- Crear una unidad administrativa para la escuela de negocios.
- Completar la unidad administrativa solo con los alumnos y el personal de la escuela de negocios.
- Agregar el equipo de TI de la escuela de negocios al rol con su ámbito.

## <a name="license-requirements"></a>Requisitos de licencia

El uso de unidades administrativas requiere una licencia de Azure Active Directory Premium para cada administrador de unidad administrativa y licencias de Azure Active Directory Free para los miembros de las unidades administrativas. Para obtener más información, consulte [Introducción a Azure AD Premium](../fundamentals/active-directory-get-started-premium.md).

## <a name="manage-administrative-units"></a>Administración de unidades administrativas

En esta versión preliminar, puede administrar las unidades administrativas mediante Azure Portal, cmdlets y scripts de PowerShell o Microsoft Graph. Para información más detallada, puede consultar nuestra documentación:

- [Crear, quitar, rellenar y agregar roles a las unidades administrativas](roles-admin-units-manage.md): procedimientos completos.
- [Trabajar con las unidades administrativas](/powershell/azure/active-directory/working-with-administrative-units?view=azureadps-2.0): trabajo con las unidades administrativas mediante PowerShell.
- [Compatibilidad de una unidad administrativa con Graph](/graph/api/resources/administrativeunit?view=graph-rest-beta): documentación detallada sobre Microsoft Graph para las unidades administrativas.

### <a name="planning-your-administrative-units"></a>Planeamiento de las unidades administrativas

Las unidades administrativas se pueden usar para agrupar recursos de Azure AD de manera lógica. Por ejemplo, en el caso de una organización cuyo departamento de TI está disperso a nivel global, puede que tenga sentido crear unidades administrativas que definan esos límites geográficos. En otro escenario en el que una organización multinacional tiene distintas "suborganizaciones" que son semiautónomas en las operaciones, cada suborganización puede estar representada por una unidad administrativa.

Los requisitos únicos de una organización lo guiarán por los criterios según los cuales crear unidades administrativas. Las unidades administrativas son una forma habitual de definir la estructura en todos los servicios de Microsoft 365. Se recomienda que prepare las unidades administrativas teniendo en mente su uso en los servicios de Microsoft 365. Puede aprovechar al máximo las unidades administrativas cuando puede asociar recursos comunes en Microsoft 365 bajo una unidad administrativa.

Puede esperar que la creación de las unidades administrativas en la organización pase por las fases siguientes:

1. Adopción inicial: la organización empezará a crear unidades administrativas en función de los criterios iniciales y el número de unidades administrativas aumentará a medida que se perfeccionan los criterios.
1. Eliminación: una vez que los criterios estén bien definidos, se eliminarán las unidades administrativas que ya no se requieran.
1. Estabilización: La estructura de la organización está bien definida y el número de unidades administrativas no va a cambiar significativamente en períodos de duración breves.

## <a name="currently-supported-scenarios"></a>Escenarios admitidos actualmente

Los Administradores globales o los Administradores de roles con privilegios pueden usar el portal de Azure AD para crear unidades administrativas, agregar usuarios como miembros de las unidades administrativas y, luego, asignar personal de TI a los roles de administrador con ámbito de unidad administrativa. Los administradores con ámbito de unidad administrativa pueden usar el centro de administración de Microsoft 365 para la administración básica de los usuarios de sus unidades administrativas.

Además, los grupos se pueden agregar como miembros de la unidad administrativa y un administrador de grupo con ámbito de unidad de administración puede administrarlos mediante PowerShell, Microsoft Graph y el portal de Azure AD.

En la tabla siguiente se describe la compatibilidad actual con los escenarios de unidades administrativas.

### <a name="administrative-unit-management"></a>Administración de unidades administrativas

Permisos |   MS Graph/PowerShell   | Portal de Azure AD | Centro de administración de Microsoft 365
----------- | ----------------------- | --------------- | -----------------
Crear y eliminar unidades administrativas   |    Compatible    |   Compatible   |    No compatible
Agregar y quitar miembros de las unidades administrativas de manera individual    |   Compatible    |   Compatible   |    No compatible
Agregar o quitar en masa miembros de las unidades administrativas con un archivo .csv   |    No compatible     |  Compatible   |    Sin planes de compatibilidad
Asignar administradores con ámbito de unidad administrativa  |     Compatible    |   Compatible    |   No compatible
Agregar y quitar miembros de unidades administrativas de manera dinámica según los atributos | No compatible | No compatible | No compatible

### <a name="user-management"></a>Administración de usuarios

Permisos |   MS Graph/PowerShell   | Portal de Azure AD | Centro de administración de Microsoft 365
----------- | ----------------------- | --------------- | -----------------
Administración con ámbito de unidad administrativa de las propiedades de usuario, contraseñas y licencias   |    Compatible     |  Compatible   |   Compatible
Bloqueo y desbloqueo con ámbito de unidad administrativa de los inicios de sesión de usuario    |   Compatible   |    Compatible   |    Compatible
Administración con ámbito de unidad administrativa de las credenciales de MFA de usuario   |    Compatible   |   Compatible   |   No compatible

### <a name="group-management"></a>Administración de grupos

Permisos |   MS Graph/PowerShell   | Portal de Azure AD | Centro de administración de Microsoft 365
----------- | ----------------------- | --------------- | -----------------
Administración con ámbito de unidad administrativa de los miembros y las propiedades de un grupo     |  Compatible   |    Compatible    |  No compatible
Administración con ámbito de unidad administrativa de las licencias de grupo   |    Compatible  |    Compatible   |   No compatible

> [!NOTE]
>
> Los administradores con un ámbito de unidad administrativa no pueden administrar las reglas de pertenencia a grupos dinámicos.

Las unidades administrativas solo aplican el ámbito a los permisos de administración. No impiden que los miembros o los administradores usen sus [permisos de usuario predeterminados](../fundamentals/users-default-permissions.md) para examinar otros usuarios, grupos o recursos fuera de la unidad administrativa. En el centro de administración de Microsoft 365, se filtran los usuarios que están fuera de las unidades administrativas de un administrador con ámbito, pero se pueden examinar otros usuarios en el portal de Azure AD, PowerShell y otros servicios de Microsoft.

## <a name="next-steps"></a>Pasos siguientes

- [Administración de unidades administrativas](roles-admin-units-manage.md)
- [Administración de usuarios en unidades administrativas](roles-admin-units-add-manage-users.md)
- [Administración de grupos en unidades administrativas](roles-admin-units-add-manage-groups.md)
- [Asignación de roles con ámbito a una unidad administrativa](roles-admin-units-assign-roles.md)
