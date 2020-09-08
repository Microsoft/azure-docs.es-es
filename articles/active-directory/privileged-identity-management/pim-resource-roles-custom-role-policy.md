---
title: Uso de roles personalizados de Azure en PIM (Azure AD) | Microsoft Docs
description: Aprenda a usar los roles personalizados de Azure AD en Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: pim
ms.date: 11/08/2019
ms.author: curtand
ms.collection: M365-identity-device-management
ms.openlocfilehash: 24b7845ec66a85e6ced4f1df9caec409a94016bf
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88782607"
---
# <a name="use-azure-custom-roles-in-privileged-identity-management"></a>Uso de roles personalizados de Azure AD en Privileged Identity Management

Puede que tenga que aplicar una configuración estricta de Privileged Identity Management (PIM) a algunos usuarios de un rol con privilegios de la organización de Azure Active Directory (Azure AD) y, al mismo tiempo, proporcionar una mayor autonomía a otros. Imagine un escenario en el que su organización contrata varios a asociados para que le ayuden en el desarrollo de una aplicación que se ejecutará en una suscripción de Azure.

Como administrador de recursos, quiere que los empleados puedan obtener acceso sin necesidad de aprobación. Sin embargo, todos los asociados contratados deben obtener una aprobación cuando soliciten acceso a los recursos de la organización.

Siga los pasos que se describen en la siguiente sección para configurar las opciones de destino de Privileged Identity Management para los roles de recursos de Azure.

## <a name="create-the-custom-role"></a>Creación del rol personalizado

Para crear un rol personalizado para un recurso, siga los pasos descritos en [Roles personalizados de Azure](../../role-based-access-control/custom-roles.md).

Cuando cree un rol personalizado, incluya un nombre descriptivo para que pueda recordar fácilmente qué rol integrado pretende duplicar.

> [!NOTE]
> Asegúrese de que el rol personalizado sea un duplicado del rol integrado que quiere duplicar, y que su ámbito coincida con el rol integrado.

## <a name="apply-pim-settings"></a>Aplicación de la configuración de PIM

Cuando se haya creado el rol en la organización de Azure AD, vaya a la página **Privileged Identity Management - Recursos de Azure** de Azure Portal. Seleccione el recurso al cual se aplica el rol.

![Panel "Privileged Identity Management - Recursos de Azure"](media/pim-resource-roles-custom-role-policy/aadpim-manage-azure-resource-some-there.png)

[Configure los valores de rol de Privileged Identity Management](pim-resource-roles-configure-role-settings.md) que se deberían aplicar a estos miembros del rol.

Por último, [asigne roles](pim-resource-roles-assign-roles.md) al grupo distinto de miembros que quiere establecer como destino de esta configuración.

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de las opciones de rol de recursos de Azure en Privileged Identity Management](pim-resource-roles-configure-role-settings.md)
- [Roles personalizados en Azure](../../role-based-access-control/custom-roles.md)