---
title: 'Preguntas frecuentes sobre Azure Security Center: preguntas sobre permisos'
description: Estas preguntas más frecuentes responden a cuestiones sobre permisos de Azure Security Center, un producto que ayuda a evitar, detectar y responder a amenazas.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: be2ab6d5-72a8-411f-878e-98dac21bc5cb
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/25/2020
ms.author: memildin
ms.openlocfilehash: 88d6207bf44d1ec04fde4f74ac9295412ea88fbc
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87824620"
---
# <a name="permissions"></a>Permisos

## <a name="how-do-permissions-work-in-azure-security-center"></a>¿Cómo funcionan los permisos en Azure Security Center?

Azure Security Center usa el [control de acceso basado en rol de Azure (Azure RBAC)](../role-based-access-control/role-assignments-portal.md), que proporciona [roles integrados](../role-based-access-control/built-in-roles.md) que se pueden asignar a usuarios, grupos y servicios en Azure.

Security Center evalúa la configuración de los recursos para identificar problemas de seguridad y vulnerabilidades. En Security Center, solo se muestra información relacionada con un recurso cuando tiene asignado el rol de Propietario, Colaborador o Lector a la suscripción o grupo de recursos al que pertenece un recurso.

Consulte [Permisos en Azure Security Center](security-center-permissions.md) para más información sobre los roles y las acciones permitidas en Security Center.



## <a name="who-can-modify-a-security-policy"></a>¿Quién puede modificar una directiva de seguridad?

Para modificar una directiva de seguridad, debe ser administrador de seguridad o propietario/colaborador de esa suscripción.

Para obtener información sobre cómo configurar una directiva de seguridad, consulte [Establecimiento de directivas de seguridad en el Centro de seguridad de Azure](tutorial-security-policy.md).