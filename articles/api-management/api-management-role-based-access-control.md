---
title: Uso del control de acceso basado en rol en Azure API Management | Microsoft Docs
description: Obtención de información sobre cómo usar los roles integrados y crear roles personalizados en Azure API Management
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 364cd53e-88fb-4301-a093-f132fa1f88f5
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 06/20/2018
ms.author: apimpm
ms.openlocfilehash: faef2721b48ffab12264c585d2dec55ab9334016
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87015296"
---
# <a name="how-to-use-role-based-access-control-in-azure-api-management"></a>Uso del control de acceso basado en rol en Azure API Management

Azure API Management se basa en el control de acceso basado en rol (RBAC) de Azure para permitir una administración de acceso pormenorizada de servicios y entidades de API Management (por ejemplo, API y directivas). En este artículo se proporciona información general de los roles integrados y personalizados en API Management. Para más información sobre la administración de acceso en Azure Portal, consulte la [introducción sobre la administración de acceso en Azure Portal](../role-based-access-control/overview.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="built-in-roles"></a>Roles integrados

Actualmente, API Management cuenta con tres roles integrados y próximamente agregará otros dos. Estos roles pueden asignarse en distintos ámbitos, como la suscripción, el grupo de recursos y la instancia concreta de API Management. Por ejemplo, si el rol "Lector de servicios de Azure API Management" se asigna a un usuario en el nivel de grupo de recursos, el usuario tendrá acceso de lectura a todas las instancias de API Management incluidas en el grupo de recursos. 

En la tabla siguiente se proporcionan breves descripciones de los roles integrados. Estos roles se pueden asignar a través de Azure Portal u otras herramientas, como Azure [PowerShell](../role-based-access-control/role-assignments-powershell.md), la [CLI de Azure](../role-based-access-control/role-assignments-cli.md) y la [API de REST](../role-based-access-control/role-assignments-rest.md). Para obtener detalles sobre la asignación de roles integrados, consulte [Uso de asignaciones de roles para administrar el acceso a los recursos de la suscripción de Azure](../role-based-access-control/role-assignments-portal.md).

| Role          | Acceso de lectura<sup>[1]</sup> | Acceso de escritura<sup>[2]</sup> | Creación, eliminación y escalado del servicio, VPN y configuración personalizada de dominios | Acceso al portal del editor heredado | Descripción
| ------------- | ---- | ---- | ---- | ---- | ---- 
| Colaborador de servicio de administración de API | ✓ | ✓ | ✓ | ✓ | Superusuario. Tiene acceso CRUD completo a los servicios y a las entidades de API Management (por ejemplo, API y directivas). Tiene acceso al portal del editor heredado. |
| Rol de lector de servicios de API Management | ✓ | | || Tiene acceso de solo lectura a los servicios y a las entidades de API Management. |
| Rol del operador de servicios de API Management | ✓ | | ✓ | | Puede administrar servicios de API Management, pero no entidades.|
| Editor de servicios de API Management<sup>*</sup> | ✓ | ✓ | |  | Puede administrar entidades de API Management, pero no servicios.|
| Administrador de contenido de API Management<sup>*</sup> | ✓ | | | ✓ | Puede administrar el portal del desarrollador. Acceso de solo lectura a servicios y entidades.|

<sup>[1] Acceso de lectura a los servicios y entidades de API Management (por ejemplo, API y directivas)</sup>

<sup>[2] Acceso de escritura a los servicios y identidades de API Management, excepto las operaciones siguientes: creación, eliminación y escalado de instancias; configuración de VPN, y configuración personalizada de dominios</sup>

<sup>\* El rol Service Editor estará disponible cuando se realice la migración de toda la interfaz de usuario de administración desde el portal del editor existente a Azure Portal. El rol Content Manager estará disponible después de que el portal del editor se refactorice, de modo que solo contenga funcionalidades relacionadas con la administración del portal del desarrollador.</sup>  

## <a name="custom-roles"></a>Roles personalizados

Si ninguno de los roles integrados satisface sus necesidades específicas, se pueden crear roles personalizados para proporcionar administración de acceso más pormenorizada para entidades de API Management. Por ejemplo, puede crear un rol personalizado que tenga acceso de solo lectura a un servicio de API Management, pero que tenga acceso de escritura solamente a una API específica. Para más información sobre los roles personalizados, consulte [Roles personalizados para RBAC de Azure](../role-based-access-control/custom-roles.md). 

> [!NOTE]
> Para poder ver una instancia de API Management en Azure Portal, un rol personalizado debe incluir la acción ```Microsoft.ApiManagement/service/read```.

A la hora de crear un rol personalizado, es más fácil comenzar con uno de los roles integrados. Edite los atributos para agregar los elementos **Actions**, **NotActions** o **AssignableScopes** y guarde los cambios como un nuevo rol. El ejemplo siguiente comienza con el rol "Lector de servicios de API Management" y crea un rol personalizado denominado "Calculator API Editor". Puede asignar el rol personalizado a una API concreta. De ese modo, el rol solamente tendrá acceso a esa API. 

```powershell
$role = Get-AzRoleDefinition "API Management Service Reader Role"
$role.Id = $null
$role.Name = 'Calculator API Contributor'
$role.Description = 'Has read access to Contoso APIM instance and write access to the Calculator API.'
$role.Actions.Add('Microsoft.ApiManagement/service/apis/write')
$role.Actions.Add('Microsoft.ApiManagement/service/apis/*/write')
$role.AssignableScopes.Clear()
$role.AssignableScopes.Add('/subscriptions/<subscription ID>/resourceGroups/<resource group name>/providers/Microsoft.ApiManagement/service/<service name>/apis/<api ID>')
New-AzRoleDefinition -Role $role
New-AzRoleAssignment -ObjectId <object ID of the user account> -RoleDefinitionName 'Calculator API Contributor' -Scope '/subscriptions/<subscription ID>/resourceGroups/<resource group name>/providers/Microsoft.ApiManagement/service/<service name>/apis/<api ID>'
```

El artículo [Operaciones del proveedor de recursos de Azure Resource Manager](../role-based-access-control/resource-provider-operations.md#microsoftapimanagement) contiene la lista de permisos que se pueden conceder en el nivel de API Management.

## <a name="video"></a>Vídeo


> [!VIDEO https://channel9.msdn.com/Blogs/AzureApiMgmt/Role-Based-Access-Control-in-API-Management/player]
>
>

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre el control de acceso basado en rol de Azure, consulte los siguientes artículos:
  * [Introducción a la administración de acceso en Azure Portal](../role-based-access-control/overview.md)
  * [Uso de asignaciones de roles para administrar el acceso a los recursos de la suscripción de Azure](../role-based-access-control/role-assignments-portal.md)
  * [Roles personalizados para RBAC de Azure](../role-based-access-control/custom-roles.md)
  * [Operaciones del proveedor de recursos de Azure Resource Manager](../role-based-access-control/resource-provider-operations.md#microsoftapimanagement)
