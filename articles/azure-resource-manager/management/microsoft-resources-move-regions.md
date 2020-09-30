---
title: Movimiento de recursos de Microsoft.Resources entre regiones
description: Se muestra cómo mover los recursos que se encuentran en el espacio de nombres Microsoft.Resources a nuevas regiones.
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: 098e5c36969da12966d793b6e1eddd4f79701ead
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88950812"
---
# <a name="move-microsoftresources-resources-to-new-region"></a>Movimiento de recursos de Microsoft.Resources a nueva región

Es posible que tenga que mover un recurso existente a una nueva región. En este artículo se muestra cómo mover dos tipos de recursos: templateSpecs y deploymentScripts, que se encuentran en el espacio de nombres Microsoft.Resources.

## <a name="move-template-specs-to-new-region"></a>Movimiento de especificaciones de plantilla a una nueva región

Si tiene una [especificación de plantilla](../templates/template-specs.md) en una región y desea moverla a una nueva, puede exportar la especificación de plantilla y volver a implementarla.

1. Use el comando para exportar una especificación de plantilla existente. Para los valores de parámetro, proporcione los valores que coincidan con la especificación de plantilla que quiere exportar.

   Para Azure PowerShell, use:

   ```azurepowershell
   Export-AzTemplateSpec `
     -ResourceGroupName demoRG `
     -Name demoTemplateSpec `
     -Version 1.0 `
     -OutputFolder c:\export
   ```

   Para la CLI de Azure, utilice:

   ```azurecli
   az template-specs export \
     --resource-group demoRG \
     --name demoTemplateSpec \
     --version 1.0 \
     --output-folder c:\export
   ```

1. Use la especificación de plantilla exportada para crear una especificación de plantilla. En los ejemplos siguientes se muestra `westus` para la nueva región, pero puede proporcionar la región que quiera.

   Para Azure PowerShell, use:

   ```azurepowershell
   New-AzTemplateSpec `
     -Name movedTemplateSpec `
     -Version 1.0 `
     -ResourceGroupName newRG `
     -Location westus `
     -TemplateJsonFile c:\export\1.0.json
   ```

   Para la CLI de Azure, utilice:

   ```azurecli
   az template-specs create \
     --name movedTemplateSpec \
     --version "1.0" \
     --resource-group newRG \
     --location "westus" \
     --template-file "c:\export\demoTemplateSpec.json"
   ```

## <a name="move-deployment-scripts-to-new-region"></a>Movimiento de scripts de implementación a una nueva región

1. Seleccione el grupo de recursos que contiene el script de implementación que quiere trasladar a una nueva región.

1. [Exportación de la plantilla](../templates/export-template-portal.md) Al exportar, seleccione el script de implementación y los demás recursos necesarios.

1. En la plantilla exportada, elimine las siguientes propiedades:

   * tenantId
   * principalId
   * clientId

1. En la plantilla exportada, la región del script de implementación tiene un valor fijo.

   ```json
   "location": "westus2",
   ```

   Cambie la plantilla para permitir establecer la ubicación mediante un parámetro. Para más información, consulte [Establecimiento de la ubicación del recurso en la plantilla de Resource Manager](../templates/resource-location.md).

   ```json
   "location": "[parameters('location')]",
   ```

1. [Implemente la plantilla exportada](../templates/deploy-powershell.md) y especifique una nueva región para el script de implementación.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre cómo mover recursos a un nuevo grupo de recursos o a una nueva suscripción, consulte [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](move-resource-group-and-subscription.md).
* Para información sobre el movimiento de los recursos a una nueva región, consulte [Movimiento de recursos de Azure entre regiones](move-region.md).
