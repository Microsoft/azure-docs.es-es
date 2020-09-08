---
title: 'Solución de un problema con Privileged Identity Management: Azure Active Directory | Microsoft Docs'
description: Aprenda a solucionar errores del sistema con roles de Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.subservice: pim
ms.date: 10/18/2019
ms.author: curtand
ms.collection: M365-identity-device-management
ms.openlocfilehash: b02b8f3b94d191c7439aadf5a56ec0e2a1c38e28
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88782505"
---
# <a name="troubleshoot-a-problem-with-privileged-identity-management"></a>Solución de un problema con Privileged Identity Management

¿Tiene un problema con Privileged Identity Management (PIM) de Azure Active Directory (Azure AD)? La siguiente información puede ayudarle a conseguir que funcione de nuevo.

## <a name="access-to-azure-resources-denied"></a>Denegación de acceso a recursos de Azure

### <a name="problem"></a>Problema

Como administrador de acceso de usuario o propietario activo de un recurso de Azure, puede ver el recurso dentro de Privileged Identity Management pero no realizar acciones tales como crear una asignación válida o ver una lista de asignaciones de roles en la página de información general sobre recursos. Cualquiera de estas acciones produce un error de autorización.

### <a name="cause"></a>Causa

Este problema puede ocurrir cuando el rol de administrador de acceso de usuario de la entidad de servicio de PIM se quitó accidentalmente de la suscripción. Para que el servicio Privileged Identity Management pueda acceder a los recursos de Azure, la entidad de servicio MS-PIM debe tener siempre asignado el [rol de administrador de acceso de usuario](../../role-based-access-control/built-in-roles.md#user-access-administrator) a través de la suscripción de Azure.

### <a name="resolution"></a>Solución

Asigne el rol de administrador de acceso de usuario al nombre de entidad de seguridad de servicio de Privileged Identity Management (MS–PIM) en el nivel de suscripción. Esta asignación debe permitir que el servicio Privileged Identity Management acceda a los recursos de Azure. El rol puede asignarse en el nivel de grupo de administración o el nivel de suscripción, según sus requisitos. Para obtener más información sobre las entidades de servicio, consulte el artículo sobre [asignación de una aplicación a un rol](../develop/howto-create-service-principal-portal.md#assign-a-role-to-the-application).

## <a name="next-steps"></a>Pasos siguientes

- [Requisitos de licencia para usar Privileged Identity Management](subscription-requirements.md)
- [Protección del acceso con privilegios para las implementaciones híbridas y en la nube en Azure AD](../users-groups-roles/directory-admin-roles-secure.md?toc=%2fazure%2factive-directory%2fprivileged-identity-management%2ftoc.json)
- [Implementación de Privileged Identity Management](pim-deployment-plan.md)