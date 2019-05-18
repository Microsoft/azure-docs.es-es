---
title: Configurar alertas de seguridad para los roles de recursos de Azure en PIM - Azure Active Directory | Microsoft Docs
description: Aprenda a configurar alertas de seguridad para los roles de recurso de Azure en Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: pim
ms.date: 04/02/2018
ms.author: rolyon
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: c15e4080308c3e7e2ff54312cd91fa1f3d68668a
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65602433"
---
# <a name="configure-security-alerts-for-azure-resource-roles-in-pim"></a>Configuración de alertas de seguridad para los roles de recurso de Azure en PIM
Azure Active Directory (Azure AD) Privileged Identity Management (PIM) genera alertas cuando no hay actividades sospechosas o no seguras en su entorno. Cuando se desencadena una alerta, se muestra en la página de alertas. 

![Página de alertas](media/pim-resource-roles-configure-alerts/rbac-alerts-page.png)

## <a name="review-alerts"></a>Revisar alertas
Seleccione una alerta para ver un informe que enumera los usuarios o roles que desencadenaron la alerta, así como consejos de solución.

![Informe de alertas](media/pim-resource-roles-configure-alerts/rbac-alert-info.png)

## <a name="alerts"></a>Alertas
| Alerta | Severity | Desencadenador | Recomendación |
| --- | --- | --- | --- |
| **Hay demasiados propietarios asignados a un recurso** |Mediano |Demasiados usuarios tienen el rol de propietario. |Revise los usuarios de la lista y vuelva a asignar algunos a roles con menos privilegios. |
| **Hay demasiados propietarios permanentes asignados a un recurso** |Mediano |Demasiados usuarios están asignados permanentemente a un rol. |Revise los usuarios de la lista y vuelva a asignar algunos para que tengan que solicitar la activación para usar el rol. |
| **Se ha creado un rol duplicado** |Mediano |Varios roles tienen los mismos criterios. |Use solo uno de estos roles. |


### <a name="severity"></a>Severity
* **Alta**: requiere acción inmediata debido a la infracción de una directiva. 
* **Media**: no requiere acción inmediata, pero indica una posible infracción de una directiva.
* **Baja**: no requiere acción inmediata pero sugiere un cambio de directiva preferida.

## <a name="configure-security-alert-settings"></a>Configuración de alertas de seguridad
En la página de alertas, vaya a **Configuración**.
![Configuración](media/pim-resource-roles-configure-alerts/rbac-navigate-settings.png)

Personalice la configuración de las diferentes alertas para que encajen con su entorno y con sus objetivos de seguridad.
![Personalizar la configuración](media/pim-resource-roles-configure-alerts/rbac-alert-settings.png)

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de alertas de seguridad para los roles de recurso de Azure en PIM](pim-resource-roles-configure-alerts.md)
