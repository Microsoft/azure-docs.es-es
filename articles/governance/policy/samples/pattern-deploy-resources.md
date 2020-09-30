---
title: 'Patrón: implementación de recursos con una definición de directiva'
description: Este patrón de Azure Policy proporciona un ejemplo de cómo implementar recursos con una definición de directiva deployIfNotExists.
ms.date: 08/17/2020
ms.topic: sample
ms.openlocfilehash: 0a9eec54954b8963f38b3f19a0d0cabffe1092e5
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89649985"
---
# <a name="azure-policy-pattern-deploy-resources"></a>Patrón de Azure Policy: implementación de recursos

El efecto [deployIfNotExists](../concepts/effects.md#deployifnotexists) permite implementar una [plantilla de Azure Resource Manager](../../../azure-resource-manager/templates/overview.md) (plantilla ARM) al crear o actualizar un recurso que no es compatible. Este enfoque puede preferirse al uso del efecto [deny](../concepts/effects.md#deny), ya que permite que se sigan creando recursos, pero garantiza que se realizan cambios para que sean compatibles.

## <a name="sample-policy-definition"></a>Definición de directiva de ejemplo

Esta definición de directiva usa el operador **field** para evaluar el valor de `type` del recurso creado o actualizado. Cuando ese recurso es un elemento _Microsoft.Network/virtualNetworks_, la directiva busca un monitor de red en la ubicación del recurso nuevo o actualizado. Si no se encuentra ningún monitor de red que coincida, se implementa la plantilla de Resource Manager para crear el recurso que falta.

:::code language="json" source="~/policy-templates/patterns/pattern-deploy-resources.json":::

### <a name="explanation"></a>Explicación

#### <a name="existencecondition"></a>existenceCondition

:::code language="json" source="~/policy-templates/patterns/pattern-deploy-resources.json" range="18-23":::

El bloque **properties.policyRule.then.details** indica a Azure Policy qué buscar relacionado con el recurso creado o actualizado en el bloque **properties.policyRule.if**. En este ejemplo, debe existir un monitor de red en el grupo de recursos **networkWatcherRG** con el **campo** `location` igual a la ubicación del recurso nuevo o actualizado. El uso de la función `field()` permite a **existenceCondition** acceder a las propiedades del recurso nuevo o actualizado, específicamente a la propiedad `location`.

#### <a name="roledefinitionids"></a>roleDefinitionIds

:::code language="json" source="~/policy-templates/patterns/pattern-deploy-resources.json" range="24-26":::

La _matriz_ de propiedades **roleDefinitionIds** del bloque **properties.policyRule.then.details** indica a la definición de directiva qué derechos necesita la identidad administrada para implementar la plantilla de Resource Manager incluida. Esta propiedad debe estar configurada para incluir roles que tengan los permisos necesarios para la implementación de la plantilla, pero se debería usar el concepto de "principio de privilegios mínimos" y tener solo las operaciones necesarias y nada más.

#### <a name="deployment-template"></a>Plantilla de implementación

La parte **deployment** de la definición de directiva tiene un bloque **properties** que define los tres componentes principales:

- **mode**: esta propiedad establece el [modo de implementación](../../../azure-resource-manager/templates/deployment-modes.md) de la plantilla.

- **template**: esta propiedad incluye la propia plantilla. En este ejemplo, el parámetro de plantilla **location** establece la ubicación del nuevo recurso de monitor de red.

  :::code language="json" source="~/policy-templates/patterns/pattern-deploy-resources.json" range="30-44":::
  
- **parameters**: esta propiedad define los parámetros que se proporcionan a la **plantilla**. Los nombres de parámetro deben coincidir con los definidos en **template**. En este ejemplo, el parámetro se denomina **location** para que coincida. El valor de **location** utiliza de nuevo la función `field()` para obtener el valor del recurso evaluado, que es la red virtual del bloque **policyRule.if**.

  :::code language="json" source="~/policy-templates/patterns/pattern-deploy-resources.json" range="45-49":::

## <a name="next-steps"></a>Pasos siguientes

- Consulte otros [patrones y definiciones integradas](./index.md).
- Revise la [estructura de definición de Azure Policy](../concepts/definition-structure.md).
- Vea la [Descripción de los efectos de directivas](../concepts/effects.md).