---
title: Permisos en Azure Advisor
description: Los permisos de Advisor y cómo pueden bloquear la capacidad de configurar suscripciones o posponer o descartar recomendaciones.
ms.topic: article
ms.date: 04/03/2019
ms.openlocfilehash: 3c7b42372ae6de5c2f519b69aec8a154a2f35a82
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87927392"
---
# <a name="permissions-in-azure-advisor"></a>Permisos en Azure Advisor

Azure Advisor proporciona recomendaciones basadas en el uso y la configuración de los recursos y las suscripciones de Azure. Advisor usa los [roles integrados](../role-based-access-control/built-in-roles.md) que se proporcionan en el [control de acceso basado en rol (RBAC de Azure)](../role-based-access-control/overview.md) para administrar el acceso a recomendaciones y características de Advisor. 

## <a name="roles-and-their-access"></a>Roles y su acceso

En la siguiente tabla se definen los roles y el acceso que tienen dentro de Advisor:

| **Rol** | **Ver recomendaciones** | **Editar reglas** | **Editar configuración de la suscripción** | **Editar configuración del grupo de recursos**| **Descartar y posponer recomendaciones**|
|---|:---:|:---:|:---:|:---:|:---:|
|Propietario de la suscripción|**X**|**X**|**X**|**X**|**X**|
|Colaborador de la suscripción|**X**|**X**|**X**|**X**|**X**|
|Lector de la suscripción|**X**|--|--|--|--|
|Propietario del grupo de recursos|**X**|--|--|**X**|**X**|
|Colaborador del grupo de recursos|**X**|--|--|**X**|**X**|
|Lector del grupo de recursos|**X**|--|--|--|--|
|Propietario del recurso|**X**|--|--|--|**X**|
|Colaborador del recurso|**X**|--|--|--|**X**|
|Lector del recurso|**X**|--|--|--|--|

> [!NOTE]
> El acceso para ver recomendaciones depende del acceso al recurso afectado de la recomendación.

## <a name="permissions-and-unavailable-actions"></a>Permisos y acciones no disponibles

La falta de los permisos adecuados puede bloquear su capacidad para realizar acciones en Advisor. Estos son algunos problemas comunes.

### <a name="unable-to-configure-subscriptions-or-resource-groups"></a>No es posible configurar suscripciones o grupos de recursos

Al intentar configurar suscripciones o grupos de recursos en Advisor, puede ver que la opción para incluir o excluir está deshabilitada. Este estado indica que no tiene un nivel de permisos suficiente para ese grupo de recursos o suscripción. Para resolver este problema, aprenda a [conceder acceso a un usuario](../role-based-access-control/quickstart-assign-role-user-portal.md).

### <a name="unable-to-postpone-or-dismiss-a-recommendation"></a>No se puede posponer o descartar una recomendación

Si recibe un error al intentar posponer o descartar una recomendación, es posible que no tenga permisos suficientes. Asegúrese de tener al menos acceso de colaborador a los recursos afectados de la recomendación que va a posponer o descartar. Para resolver este problema, aprenda a [conceder acceso a un usuario](../role-based-access-control/quickstart-assign-role-user-portal.md).

## <a name="next-steps"></a>Pasos siguientes

En este artículo se le ha proporcionado información general de cómo Advisor usa RBAC para controlar los permisos de usuario y cómo resolver problemas comunes. Para más información sobre Advisor, consulte:

- [¿Qué es Azure Advisor?](./advisor-overview.md)
- [Introducción a Azure Advisor](./advisor-get-started.md)
