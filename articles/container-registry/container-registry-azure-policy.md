---
title: Cumplimiento mediante Azure Policy
description: Asignación de directivas integradas en Azure Policy para auditar el cumplimiento de los registros de contenedor de Azure
ms.topic: article
ms.date: 06/11/2020
ms.openlocfilehash: 26c56616bcc411063d0ebfda28ba1e6fdf44c7fb
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89291029"
---
# <a name="audit-compliance-of-azure-container-registries-using-azure-policy"></a>Auditoría del cumplimiento de los registros de contenedor de Azure mediante Azure Policy

[Azure Policy](../governance/policy/overview.md) es un servicio de Azure que se usa para crear, asignar y administrar directivas. Dichas directivas aplican distintas reglas y efectos a los recursos, con el fin de que estos sigan siendo compatibles con los estándares corporativos y los acuerdos de nivel de servicio.

En este artículo se presentan las directivas integradas para Azure Container Registry. Use estas directivas para auditar los registros nuevos y existentes para el cumplimiento.

No se aplica ningún cargo por el uso de Azure Policy.

## <a name="built-in-policy-definitions"></a>Definiciones de directiva integradas

Las definiciones de directivas integradas siguientes son específicas de Azure Container Registry:

[!INCLUDE [azure-policy-reference-policies-container-registry](../../includes/policy/reference/bycat/policies-container-registry.md)]

Vea también la definición de directiva de red integrada: [Container Registry debe usar un punto de conexión de servicio de red virtual.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc4857be7-912a-4c75-87e6-e30292bcdf78)

## <a name="assign-policies"></a>Asignación de directivas

* Asigne directivas mediante el uso de [Azure Portal](../governance/policy/assign-policy-portal.md), la [CLI de Azure](../governance/policy/assign-policy-azurecli.md), una [plantilla de Resource Manager](../governance/policy/assign-policy-template.md) o el SDK de Azure Policy.
* Limite el ámbito de una asignación de directiva a un grupo de recursos, una suscripción o un [grupo de administración de Azure](../governance/management-groups/overview.md). Las asignaciones de directivas del registro de contenedor se aplican a los registros de contenedor nuevos y existentes dentro del ámbito.
* Habilite o deshabilite la [aplicación de directivas](../governance/policy/concepts/assignment-structure.md#enforcement-mode) en cualquier momento.

> [!NOTE]
> Después de asignar o actualizar una directiva, la asignación tarda algún tiempo en aplicarse a los recursos del ámbito definido. Consulte información sobre [desencadenadores de evaluación de directivas](../governance/policy/how-to/get-compliance-data.md#evaluation-triggers).

## <a name="review-policy-compliance"></a>Revisión del cumplimiento de directivas

Acceda a la información de cumplimiento generada por las asignaciones de directivas mediante Azure Portal, las herramientas de la línea de comandos de Azure o los SDK de Azure Policy. Para detalles, consulte [Obtención de datos de cumplimiento de los recursos de Azure](../governance/policy/how-to/get-compliance-data.md).

Cuando un recurso no es compatible, hay muchos motivos posibles para ello. Para determinar el motivo o encontrar al responsable del cambio, consulte [Determinación del incumplimiento](../governance/policy/how-to/determine-non-compliance.md).

### <a name="policy-compliance-in-the-portal"></a>Cumplimiento de directivas en el portal:

1. Seleccione **Todos los servicios** y busque **Directiva**.
1. Seleccione **Cumplimiento**.
1. Use los filtros para limitar los estados de cumplimiento o para buscar las directivas.

    ![Cumplimiento de directivas en el portal](./media/container-registry-azure-policy/azure-policy-compliance.png)
    
1. Seleccione una directiva para revisar los eventos y los detalles de cumplimiento de los agregados. Si lo desea, seleccione un registro específico para el cumplimiento de los recursos.

### <a name="policy-compliance-in-the-azure-cli"></a>Cumplimiento de directivas en la CLI de Azure

También puede usar la CLI de Azure para obtener datos de cumplimiento. Por ejemplo, use el comando [az policy assignment list](/cli/azure/policy/assignment#az-policy-assignment-list) en la CLI para obtener los identificadores de directiva de las directivas de Azure Container Registry que se aplican:

```azurecli
az policy assignment list --query "[?contains(displayName,'Container Registries')].{name:displayName, ID:id}" --output table
```

Salida del ejemplo:

```
Name                                                                                   ID
-------------------------------------------------------------------------------------  --------------------------------------------------------------------------------------------------------------------------------
Container Registries should not allow unrestricted network access           /subscriptions/<subscriptionID>/providers/Microsoft.Authorization/policyAssignments/b4faf132dc344b84ba68a441
Container Registries should be encrypted with a Customer-Managed Key (CMK)  /subscriptions/<subscriptionID>/providers/Microsoft.Authorization/policyAssignments/cce1ed4f38a147ad994ab60a
```

Luego, ejecute [az policy state list](/cli/azure/policy/state#az-policy-state-list) para devolver el estado de cumplimiento con formato JSON para todos los recursos de un identificador de directiva específico:

```azurecli
az policy state list \
  --resource <policyID>
```

O bien ejecute [az policy state list](/cli/azure/policy/state#az-policy-state-list) para devolver el estado de cumplimiento con formato JSON de un recurso de registro específico, como *myregistry*:

```azurecli
az policy state list \
 --resource myregistry \
 --namespace Microsoft.ContainerRegistry \
 --resource-type registries \
 --resource-group myresourcegroup
```

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre las [definiciones](../governance/policy/concepts/definition-structure.md) y [efectos](../governance/policy/concepts/effects.md) de Azure Policy.

* Creación de una [definición de directiva personalizada](../governance/policy/tutorials/create-custom-policy-definition.md).

* Más información sobre las [funcionalidades de gobernanza](../governance/index.yml) en Azure.
