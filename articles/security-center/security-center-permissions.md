---
title: Permisos en Azure Security Center | Microsoft Docs
description: En este artículo se explica cómo Azure Security Center usa el control de acceso basado en roles para asignar permisos a los usuarios e identifica las acciones permitidas para cada rol.
services: security-center
cloud: na
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: ''
ms.service: security-center
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/02/2020
ms.author: memildin
ms.openlocfilehash: e5728b220f12c3305aa896e64010fc7e5828d6fb
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88067229"
---
# <a name="permissions-in-azure-security-center"></a>Permisos en Azure Security Center

Azure Security Center usa el [control de acceso basado en rol de Azure (Azure RBAC)](../role-based-access-control/role-assignments-portal.md), que proporciona [roles integrados](../role-based-access-control/built-in-roles.md) que se pueden asignar a usuarios, grupos y servicios en Azure.

Security Center evalúa la configuración de los recursos para identificar problemas de seguridad y vulnerabilidades. En Security Center, solo se muestra información relacionada con un recurso cuando tiene asignado el rol de Propietario, Colaborador o Lector a la suscripción o grupo de recursos al que pertenece un recurso.

Además de estos roles, hay dos roles específicos de Security Center:

* **Lector de seguridad**: un usuario que pertenece a este rol tiene derechos de visualización en Security Center. El usuario puede ver las recomendaciones, las alertas, una directiva de seguridad y los estados de seguridad, pero no puede realizar cambios.
* **Administrador de seguridad**: un usuario que pertenece a este rol tiene los mismos derechos que el lector de seguridad, y puede actualizar la directiva de seguridad y descartar las alertas y las recomendaciones.

> [!NOTE]
> Los roles de seguridad Lector de seguridad y Administrador de seguridad solo tienen acceso a Security Center. Los roles de seguridad descritos no tienen acceso a otras áreas de servicio de Azure, como Storage, Web y móvil o Internet de las cosas.
>

## <a name="roles-and-allowed-actions"></a>Roles y acciones permitidas

En la siguiente tabla se muestran los roles y las acciones permitidas en Security Center.

|Acción|Lector de seguridad / <br> Lector |Administrador de seguridad  |Colaborador del grupo de recursos / <br> Propietario del grupo de recursos  |Colaborador de la suscripción  |Propietario de la suscripción  |
|:--- |:---:|:---:|:---:|:---:|:---:|
|Editar directivas de seguridad|-|✔|-|-|✔|
|Agregar o asignar iniciativas (incluidas las normas de cumplimiento normativo)|-|-|-|-|✔|
|Cambiar plan de tarifa de suscripción|-|✔|-|-|✔|
|Habilitar o deshabilitar el aprovisionamiento automático|-|✔|-|✔|✔|
|Aplicar recomendaciones de seguridad en un recurso</br> (y utilizar la [corrección rápida](security-center-remediate-recommendations.md#quick-fix-remediation))|-|-|✔|✔|✔|
|Descartar alertas|-|✔|-|✔|✔|
|Ver alertas y recomendaciones|✔|✔|✔|✔|✔|

> [!NOTE]
> Es recomendable que asigne el rol de menos permisos que los usuarios necesiten para realizar sus tareas. Por ejemplo, asigne el rol Lector a los usuarios que solo necesiten ver información sobre el estado de seguridad de los recursos, pero no llevar a cabo acciones como aplicar recomendaciones o editar directivas.
>
>

## <a name="next-steps"></a>Pasos siguientes
En este artículo se ha explicado cómo Security Center usa RBAC para asignar permisos a los usuarios e identifica las acciones permitidas para cada rol. Ahora que ya está familiarizado con las asignaciones de roles necesarios para supervisar el estado de seguridad de su suscripción, editar directivas de seguridad y aplicar recomendaciones, aprenderá los siguientes conceptos:

- [Establecer directivas de seguridad en Security Center](tutorial-security-policy.md)
- [Administrar recomendaciones de seguridad en Security Center](security-center-recommendations.md)
- [Supervisar el estado de seguridad de los recursos de Azure](security-center-monitoring.md)
- [Responder a alertas de seguridad en Security Center](security-center-managing-and-responding-alerts.md)
- [Supervisar soluciones de seguridad de asociados](security-center-partner-solutions.md)
