---
title: Control de acceso basado en roles para cuentas de Media Services - Azure | Microsoft Docs
description: En este artículo se describe el control de acceso basado en roles (RBAC) para las cuentas de Azure Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 05/23/2019
ms.author: juliako
ms.custom: seodec18
ms.openlocfilehash: c75b6e67932cfd26a3374eab3f3efa34ceade577
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87504490"
---
# <a name="role-based-access-control-rbac-for-media-services-accounts"></a>Control de acceso basado en roles (RBAC) para cuentas de Media Services

Actualmente, Azure Media Services no define los roles personalizados específicos para el servicio. Para obtener acceso completo a la cuenta de Media Services, los clientes pueden usar las funciones integradas **Propietario** o **Colaborador**. La principal diferencia entre estos roles es que el rol **Propietario** puede controlar quién tiene acceso a un recurso y el rol **Colaborador** no. El rol integrado **Lector** también se puede usar, pero el usuario o la aplicación solo tendrán acceso de lectura a las API de Media Services. 

## <a name="design-principles"></a>Principios de diseño

Uno de los principales principios de diseño de la versión v3 de la API es hacerla más segura. Las API de la versión v3 no devuelven secretos o credenciales en las operaciones **Get** o **List**. Las claves se hacen siempre NULL, se vacían o se sanean de la respuesta. El usuario debe llamar a un método de acción independiente para obtener los secretos o las credenciales. El rol **Lector** no puede llamar a operaciones como, por ejemplo, Asset.ListContainerSas, StreamingLocator.ListContentKeys o ContentKeyPolicies.GetPolicyPropertiesWithSecrets. Al tener acciones independientes, se pueden establecer permisos de seguridad de RBAC más granulares en un rol personalizado, si lo desea.

Para obtener una lista de las operaciones que admite Media Services, haga lo siguiente:

```csharp
foreach (Microsoft.Azure.Management.Media.Models.Operation a in client.Operations.List())
{
    Console.WriteLine($"{a.Name} - {a.Display.Operation} - {a.Display.Description}");
}
```

En el artículo sobre [definiciones de roles integrados](../../role-based-access-control/built-in-roles.md) se explica lo que el rol concede. 

Para obtener más información, consulte los siguientes artículos:

- [Roles de administrador de suscripciones clásico de Azure y roles de administrador de Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md)
- [¿Qué es el control de acceso basado en rol de Azure (RBAC)?](../../role-based-access-control/overview.md)
- [Uso de RBAC para administrar el acceso](../../role-based-access-control/role-assignments-rest.md)
- [Operaciones del proveedor de recursos de Media Services](../../role-based-access-control/resource-provider-operations.md#microsoftmedia)

## <a name="next-steps"></a>Pasos siguientes

- [Desarrollo con API de Media Services v3](media-services-apis-overview.md)
- [Obtener la directiva de clave de contenido mediante Media Services .NET](get-content-key-policy-dotnet-howto.md)
