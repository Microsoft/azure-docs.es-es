---
title: Etapas de implementación de un plano técnico
description: Conozca los pasos relacionados con la seguridad y los artefactos que recorren los servicios de Azure Blueprints al crear una asignación de plano técnico.
ms.date: 08/27/2020
ms.topic: conceptual
ms.openlocfilehash: 73d5d67cf77d4ccc48517c895630b8299081e2e0
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89049781"
---
# <a name="stages-of-a-blueprint-deployment"></a>Etapas de implementación de un plano técnico

Cuando se implementa un plano técnico, el servicio de Azure Blueprints lleva a cabo una serie de acciones para implementar los recursos definidos en el plano. En este artículo se proporcionan detalles sobre lo que implica cada paso.

La implementación del plano técnico se desencadena mediante la asignación de un plano a una suscripción o [la actualización de una asignación existente](../how-to/update-existing-assignments.md). Durante la implementación, Azure Blueprints realiza los siguientes pasos generales:

> [!div class="checklist"]
> - Concesión a Azure Blueprints de derechos de propietario
> - Creación del objeto de asignación del plano técnico
> - Opcional: Azure Blueprints crea la identidad administrada **asignada por el sistema**
> - Implementación de artefactos del plano técnico por la identidad administrada
> - Revocación de derechos de la identidad administrada **asignada por el sistema** y del servicio Azure Blueprints

## <a name="azure-blueprints-granted-owner-rights"></a>Concesión a Azure Blueprints de derechos de propietario

Cuando se usa una [identidad administrada asignada por el sistema](../../../active-directory/managed-identities-azure-resources/overview.md), a la entidad de servicio de Azure Blueprints se le conceden derechos de propietario en la suscripción o suscripciones asignadas. El rol concedido permite a Azure Blueprints crear y revocar después la identidad administrada **asignada por el sistema**. Si usa una identidad administrada **asignada por el usuario**, la entidad de servicio de Azure Blueprints no obtiene y no necesita derechos de propietario sobre la suscripción.

Los derechos se conceden automáticamente si la asignación se realiza a través del portal. Sin embargo, si la asignación se realiza a través de la API REST, la concesión de derechos debe hacerse con una llamada API independiente. El valor de AppId de Azure Blueprints es `f71766dc-90d9-4b7d-bd9d-4499c4331c3f`, pero la entidad de servicio varía según el inquilino. Use la [Graph API de Azure Active Directory](../../../active-directory/develop/active-directory-graph-api.md) y el punto de conexión REST [servicePrincipals](/graph/api/resources/serviceprincipal) para obtener la entidad de servicio. Después, conceda a Azure Blueprints el rol _Propietario_ a través del [Portal](../../../role-based-access-control/role-assignments-portal.md), la [CLI de Azure](../../../role-based-access-control/role-assignments-cli.md), [Azure PowerShell](../../../role-based-access-control/role-assignments-powershell.md), la [API REST](../../../role-based-access-control/role-assignments-rest.md) o una [plantilla de Azure Resource Manager](../../../role-based-access-control/role-assignments-template.md).

El servicio Azure Blueprints no implementa directamente los recursos.

## <a name="the-blueprint-assignment-object-is-created"></a>Creación del objeto de asignación del plano técnico

Un usuario, grupo o entidad de servicio asigna un plano técnico a una suscripción. El objeto de asignación existe en el nivel de suscripción donde se asignó el plano técnico. Los recursos creados por la implementación no se generan en el contexto de la entidad de implementación.

Al crear la asignación del plano técnico, se selecciona el tipo de [identidad administrada](../../../active-directory/managed-identities-azure-resources/overview.md). El valor predeterminado es una identidad administrada **asignada por el sistema**. Se puede elegir una identidad administrada **asignada por el usuario**. Cuando se usa una identidad administrada **asignada por el usuario**, deben definirse y concederse los permisos antes de la creación de la asignación del plano técnico. Los roles integrados de [Propietario](../../../role-based-access-control/built-in-roles.md#owner) y [Operador de Azure Blueprints](../../../role-based-access-control/built-in-roles.md#blueprint-operator) tienen el permiso `blueprintAssignment/write` necesario para crear una asignación que utilice una identidad administrada **asignada por el usuario**.

## <a name="optional---azure-blueprints-creates-system-assigned-managed-identity"></a>Opcional: Azure Blueprints crea la identidad administrada asignada por el sistema

Cuando se selecciona [identidad administrada asignada por el sistema](../../../active-directory/managed-identities-azure-resources/overview.md) durante la asignación, Azure Blueprints crea la identidad y concede a la identidad administrada el rol de [propietario](../../../role-based-access-control/built-in-roles.md#owner). Si se [actualiza una asignación existente](../how-to/update-existing-assignments.md), Azure Blueprints utiliza la identidad administrada creada anteriormente.

La identidad administrada relacionada con la asignación del plano técnico se usa para implementar o volver a implementar los recursos definidos en el plano técnico. Este diseño evita que las asignaciones interfieran de forma inadvertida entre sí.
Este diseño también admite la característica del [bloqueo de recursos](./resource-locking.md) mediante el control de la seguridad de cada recurso implementado desde el plano técnico.

## <a name="the-managed-identity-deploys-blueprint-artifacts"></a>Implementación de artefactos del plano técnico por la identidad administrada

La identidad administrada, a continuación, desencadena las implementaciones de Resource Manager de los artefactos en el plano en el [orden secuenciación](./sequencing-order.md) definido. El orden se puede ajustar para garantizar que los artefactos dependientes de otros artefactos se implementan en el orden correcto.

Un error de acceso de una implementación suele ser el resultado del nivel de acceso concedido a la identidad administrada. El servicio Azure Blueprints administra el ciclo de vida de la seguridad de la identidad administrada **asignada por el sistema**. Sin embargo, el usuario es responsable de administrar los derechos y el ciclo de vida de una identidad administrada **asignada por el usuario**.

## <a name="blueprint-service-and-system-assigned-managed-identity-rights-are-revoked"></a>Revocación de derechos de identidad administrada asignada por el sistema y del servicio Blueprints

Una vez que se completan las implementaciones, Azure Blueprints revoca los derechos de la identidad administrada **asignada por el sistema** de la suscripción. A continuación, el servicio Azure Blueprints revoca sus derechos de la suscripción. La eliminación de derechos evita que Azure Blueprints se convierta en un propietario permanente en una suscripción.

## <a name="next-steps"></a>Pasos siguientes

- Descubra cómo utilizar [parámetros estáticos y dinámicos](./parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](./sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](./resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../how-to/update-existing-assignments.md).
- Puede consultar la información de [solución de problemas generales](../troubleshoot/general.md) para resolver los problemas durante la asignación de un plano técnico.
