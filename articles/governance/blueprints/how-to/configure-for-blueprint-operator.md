---
title: Configuración del entorno para un operador de plano técnico
description: Aprenda a configurar el entorno de Azure para su uso con el rol integrado de Azure en el operador del plano técnico.
ms.date: 05/06/2020
ms.topic: how-to
ms.openlocfilehash: 9390958507a135497c4c48a260a44bc0f4492ffa
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87499813"
---
# <a name="configure-your-environment-for-a-blueprint-operator"></a>Configuración del entorno para un operador de plano técnico

La administración de las definiciones del plano técnico y las asignaciones del plano técnico se pueden conceder a distintos equipos. Es habitual que un arquitecto o equipo de gobernanza sea el responsable de la administración del ciclo de vida de las definiciones del plano técnico, y que un equipo de operaciones sea el responsable de administrar las asignaciones de esas definiciones de plano técnico controladas centralmente.

El control de acceso basado en rol (RBAC) integrado del **operador de plano técnico** está diseñado específicamente para su uso en este tipo de escenario. El rol permite a los equipos de los tipos de operaciones administrar la asignación de las definiciones del plano técnico de las organizaciones, pero no concede la capacidad de modificarlas. Para ello, se requiere cierta configuración en el entorno de Azure, y en este artículo se explican los pasos necesarios.

## <a name="grant-permission-to-the-blueprint-operator"></a>Concesión de permiso al operador de plano técnico

El primer paso consiste en conceder el rol de **operador de plano técnico** a la cuenta o al grupo de seguridad (recomendado) que va a asignar los planos. Esta acción debe realizarse en el nivel más alto de la jerarquía de grupos de administración que abarque todos los grupos de administración y las suscripciones a las que el equipo de operaciones debe tener acceso de asignación de planos. Se recomienda seguir el principio de privilegios mínimos al conceder los permisos.

1. (Recomendado) [Crear un grupo de seguridad y agregar miembros](../../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md)

1. [Agregue una asignación de rol](../../../role-based-access-control/role-assignments-portal.md#add-a-role-assignment) de **operador de plano técnico** a la cuenta o el grupo de seguridad

## <a name="user-assign-managed-identity"></a>Identidad administrada asignada por el usuario

Una definición de plano técnico puede usar identidades administradas asignadas por el sistema o por el usuario. Sin embargo, al usar el rol de **operador de plano técnico**, la definición del plano técnico debe configurarse para usar una identidad administrada asignada por el usuario. Además, la cuenta o el grupo de seguridad a quien se concede el rol de **operador de plano técnico** debe tener concedido el rol de **operador de identidad administrada** en la identidad administrada asignada por el usuario. Sin este permiso, las asignaciones de plano técnico generan errores debido a la falta de permisos.

1. [Cree una identidad administrada asignada por el usuario](../../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md#create-a-user-assigned-managed-identity) para que la use un plano técnico asignado.

1. [Agregue una asignación de roles](../../../role-based-access-control/role-assignments-portal.md#add-a-role-assignment) de **operador de identidad administrada** a la cuenta o al grupo de seguridad. Limite la asignación de roles a la nueva identidad administrada asignada por el usuario.

1. Como **operador de plano técnico**, [asigne un plano](../create-blueprint-portal.md#assign-a-blueprint) que use la nueva identidad administrada asignada por el usuario.

## <a name="next-steps"></a>Pasos siguientes

- Información acerca del [ciclo de vida del plano técnico](../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../concepts/resource-locking.md).
- Puede consultar la información de [solución de problemas generales](../troubleshoot/general.md) para resolver los problemas durante la asignación de un plano técnico.