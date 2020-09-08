---
title: Cumplimiento mediante Azure Policy
description: Asignación de directivas integradas en Azure Policy para auditar el cumplimiento de los recursos de Azure SignalR Service.
author: JialinXin
ms.service: signalr
ms.topic: conceptual
ms.date: 06/17/2020
ms.author: jixin
ms.openlocfilehash: 018033d3a6123948191a7261f5a1ee2ae526e25a
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89295033"
---
# <a name="audit-compliance-of-azure-signalr-service-resources-using-azure-policy"></a>Auditoría del cumplimiento de los recursos de Azure SignalR Service mediante Azure Policy

[Azure Policy](../governance/policy/overview.md) es un servicio de Azure que se usa para crear, asignar y administrar directivas. Dichas directivas aplican distintas reglas y efectos a los recursos, con el fin de que estos sigan siendo compatibles con los estándares corporativos y los acuerdos de nivel de servicio.

En este artículo se presentan las directivas integradas (versión preliminar) para Azure SignalR Service. Use estas directivas para auditar los recursos nuevos y existentes de SignalR para el cumplimiento.

No se aplica ningún cargo por el uso de Azure Policy.

## <a name="built-in-policy-definitions"></a>Definiciones de directiva integradas

Las definiciones de directivas integradas siguientes son específicas de Azure SignalR Service:

[!INCLUDE [azure-policy-reference-policies-signalr](../../includes/policy/reference/bycat/policies-signalr.md)]

## <a name="assign-policy-definitions"></a>Asignación de definiciones de directiva

* Asigne definiciones de directiva mediante el uso de [Azure Portal](../governance/policy/assign-policy-portal.md), la [CLI de Azure](../governance/policy/assign-policy-azurecli.md), una [plantilla de Resource Manager](../governance/policy/assign-policy-template.md) o los SDK de Azure Policy.
* Limite el ámbito de una asignación de directiva a un grupo de recursos, una suscripción o un [grupo de administración de Azure](../governance/management-groups/overview.md). Las asignaciones de directivas de SignalR se aplican a los recursos nuevos y existentes de SignalR dentro del ámbito.
* Habilite o deshabilite la [aplicación de directivas](../governance/policy/concepts/assignment-structure.md#enforcement-mode) en cualquier momento.

> [!NOTE]
> Después de asignar o actualizar una directiva, la asignación tarda algún tiempo en aplicarse a los recursos del ámbito definido. Consulte información sobre [desencadenadores de evaluación de directivas](../governance/policy/how-to/get-compliance-data.md#evaluation-triggers).

## <a name="review-policy-compliance"></a>Revisión del cumplimiento de directivas

Acceda a la información de cumplimiento generada por las asignaciones de directivas mediante Azure Portal, las herramientas de la línea de comandos de Azure o los SDK de Azure Policy. Para detalles, consulte [Obtención de datos de cumplimiento de los recursos de Azure](../governance/policy/how-to/get-compliance-data.md).

Cuando un recurso no es compatible, hay muchos motivos posibles para ello. Para determinar el motivo o encontrar al responsable del cambio, consulte [Determinación del incumplimiento](../governance/policy/how-to/determine-non-compliance.md).

### <a name="policy-compliance-in-the-portal"></a>Cumplimiento de directivas en el portal:

1. Seleccione **Todos los servicios** y busque **Directiva**.
1. Seleccione **Cumplimiento**.
1. Uso de los filtros para limitar los estados de cumplimiento o para buscar directivas
   
    [ ![Cumplimiento de directivas en el portal](./media/signalr-howto-azure-policy/azure-policy-compliance.png) ](./media/signalr-howto-azure-policy/azure-policy-compliance.png#lightbox)
2. Seleccione una directiva para revisar los eventos y los detalles de cumplimiento de los agregados. Si lo desea, seleccione un recurso de SignalR específico para el cumplimiento de los recursos.

### <a name="policy-compliance-in-the-azure-cli"></a>Cumplimiento de directivas en la CLI de Azure

También puede usar la CLI de Azure para obtener datos de cumplimiento. Por ejemplo, use el comando [az policy assignment list](/cli/azure/policy/assignment#az-policy-assignment-list) en la CLI para obtener los identificadores de directiva de las directivas de Azure SignalR Service que se aplican:

```azurecli
az policy assignment list --query "[?contains(displayName,'SignalR')].{name:displayName, ID:id}" --output table
```

Salida del ejemplo:

```
Name                                                                                   ID
-------------------------------------------------------------------------------------  --------------------------------------------------------------------------------------------------------------------------------
[Preview]: Azure SignalR Service should use private links  /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.Authorization/policyAssignments/<assignmentId>
```

A continuación, ejecute [az policy state list](/cli/azure/policy/state#az-policy-state-list) para devolver el estado de cumplimiento con formato JSON para todos los recursos de un grupo de recursos específico:

```azurecli
az policy state list --g <resourceGroup>
```

O bien ejecute [az policy state list](/cli/azure/policy/state#az-policy-state-list) para devolver el estado de cumplimiento con formato JSON de un recurso de SignalR específico:

```azurecli
az policy state list \
 --resource /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.SignalRService/SignalR/<resourceName> \
 --namespace Microsoft.SignalRService \
 --resource-group <resourceGroup>
```

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre las [definiciones](../governance/policy/concepts/definition-structure.md) y [efectos](../governance/policy/concepts/effects.md) de Azure Policy

* Creación de una [definición de directiva personalizada](../governance/policy/tutorials/create-custom-policy-definition.md)

* Más información sobre las [funcionalidades de gobernanza](../governance/index.yml) en Azure


<!-- LINKS - External -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/