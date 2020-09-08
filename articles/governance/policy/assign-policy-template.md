---
title: 'Inicio rápido: Asignación de directivas nuevas con plantillas'
description: En este inicio rápido se usa una plantilla de Azure Resource Manager para crear una asignación de directivas para identificar los recursos no compatibles.
ms.date: 08/17/2020
ms.topic: quickstart
ms.custom: subject-armqs
ms.openlocfilehash: 2f012f9cc8f986f8f887096098961e44c4b7dffb
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88689854"
---
# <a name="quickstart-create-a-policy-assignment-to-identify-non-compliant-resources-by-using-an-arm-template"></a>Inicio rápido: Creación de una asignación de directiva para identificar recursos no compatibles mediante una plantilla de Resource Manager

El primer paso para entender el cumplimiento en Azure es identificar el estado de sus recursos.
Este inicio rápido le guía por el proceso de usar una plantilla de Azure Resource Manager para crear una asignación de directiva con el objeto de identificar las máquinas virtuales que no usan discos administrados. Al finalizar este proceso, habrá identificado correctamente máquinas virtuales que no utilizan discos administrados. _No son compatibles_ con la asignación de directiva.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

:::image type="content" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Implementación de la plantilla de Resource Manager para asignar una instancia de Azure Policy a Azure" border="false" link="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-azurepolicy-assign-builtinpolicy-resourcegroup%2Fazuredeploy.json":::

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

En este inicio rápido, creará una asignación de directiva y asignará una definición de directiva integrada denominada _Auditoría de máquinas virtuales que no usan discos administrados_. Para una lista parcial de las directivas integradas disponibles, consulte los [ejemplos de Azure Policy](./samples/index.md).

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-azurepolicy-assign-builtinpolicy-resourcegroup/).

:::code language="json" source="~/quickstart-templates/101-azurepolicy-assign-builtinpolicy-resourcegroup/azuredeploy.json":::

El recurso definido en la plantilla es el siguiente:

- [Microsoft.Authorization/policyAssignments](/azure/templates/microsoft.authorization/policyassignments)

## <a name="deploy-the-template"></a>Implementación de la plantilla

> [!NOTE]
> El servicio Azure Policy es gratuito. Para más información, consulte la [Introducción a Azure Policy](./overview.md).

1. Seleccione la siguiente imagen para iniciar sesión en Azure Portal y abrir la plantilla:

   :::image type="content" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Implementación de la plantilla de Resource Manager para asignar una instancia de Azure Policy a Azure" border="false" link="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-azurepolicy-assign-builtinpolicy-resourcegroup%2Fazuredeploy.json":::

1. Seleccione o escriba los siguientes valores:

   | Nombre | Value |
   |------|-------|
   | Suscripción | Seleccione su suscripción a Azure. |
   | Resource group | Seleccione **Crear**, especifique un nombre y seleccione **Aceptar**. En la captura de pantalla, el nombre del grupo de recursos es _mypolicyquickstart\<Date in MMDD\>rg_. |
   | Location | Seleccione una región. Por ejemplo, **Centro de EE. UU**. |
   | Nombre de la asignación de directiva | Especifique un nombre para la asignación de directiva. Si lo desea, puede usar la definición de directiva en pantalla. Por ejemplo, _Auditoría de máquinas virtuales que no usan discos administrados_. |
   | Nombre del grupo de recursos | Especifique un nombre para el grupo de recursos donde desea asignar la directiva. En este inicio rápido se usa el valor predeterminado **[resourceGroup().name]** . **[resourceGroup()](../../azure-resource-manager/templates/template-functions-resource.md#resourcegroup)** es una función de plantilla que recupera el grupo de recursos. |
   | ID de definición de directiva | Especifique **/providers/Microsoft.Authorization/policyDefinitions/0a914e76-4921-4c19-b460-a2d36003525a**. |
   | Acepto los términos y condiciones indicados anteriormente | (Seleccionar) |

1. Seleccione **Comprar**.

Algunos recursos adicionales:

- Puede encontrar más plantillas de ejemplo en [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Authorization&pageNumber=1&sort=Popular).
- Para ver la referencia de plantilla, vaya a la [referencia de plantilla de Azure](/azure/templates/microsoft.authorization/allversions).
- Para aprender a desarrollar plantillas de Resource Manager, consulte la [documentación de Azure Resource Manager](../../azure-resource-manager/management/overview.md).
- Para información sobre la implementación de nivel de suscripción, consulte [Create resource groups and resources at the subscription level](../../azure-resource-manager/templates/deploy-to-subscription.md) (Creación de grupos de recursos y recursos en el nivel de suscripción).

## <a name="validate-the-deployment"></a>Validación de la implementación

Seleccione **Cumplimiento** en el panel izquierdo de la página. A continuación, busque la asignación de directiva _Auditoría de máquinas virtuales que no usan discos administrados_ que ha creado.

:::image type="content" source="./media/assign-policy-template/policy-compliance.png" alt-text="Página de información general del cumplimiento de directivas" border="false":::

Si hay algún recurso existente no compatible con esta nueva asignación, aparecerá en la pestaña **Recursos no compatibles**.

Para más información, consulte [How compliance works](./how-to/get-compliance-data.md#how-compliance-works) (Funcionamiento del cumplimiento).

## <a name="clean-up-resources"></a>Limpieza de recursos

Para quitar la asignación creada, siga estos pasos:

1. Seleccione **Cumplimiento** (o **Asignaciones**) en el lado izquierdo de página Azure Policy y busque la asignación de directiva _Auditoría de máquinas virtuales que no usan discos administrados_ asignación de directiva que ha creado.

1. Haga clic con el botón derecho en la asignación de directiva _Auditoría de máquinas virtuales que no usan discos administrados_ y seleccione **Eliminar asignación**.

   :::image type="content" source="./media/assign-policy-template/delete-assignment.png" alt-text="Eliminación de una asignación de la página de información general de cumplimiento" border="false":::

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido se asigna una definición de directiva integrada a un ámbito y se evalúa su informe de cumplimiento. La definición de la directiva confirma que todos los recursos del ámbito son compatibles y se identifican cuáles no lo son.

Para más información sobre la asignación de directivas para garantizar la compatibilidad de los nuevos recursos, continúe con el tutorial para:

> [!div class="nextstepaction"]
> [Crear y administrar directivas](./tutorials/create-and-manage.md)