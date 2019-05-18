---
title: Modos de implementación de Azure Resource Manager | Microsoft Docs
description: Descripción de cómo especificar si desea usar un modo de implementación completa o incremental con Azure Resource Manager.
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/14/2019
ms.author: tomfitz
ms.openlocfilehash: b4d3a9c5e985949d813b638806f60bc86f5a163a
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65789266"
---
# <a name="azure-resource-manager-deployment-modes"></a>Modos de implementación de Azure Resource Manager

Al implementar los recursos, debe especificar si la implementación es una actualización incremental o una actualización completa.  La diferencia principal entre estos dos modos es la forma en que Resource Manager controla los recursos existentes en el grupo de recursos que no están en la plantilla. El modo predeterminado es el incremental.

En ambos modos, Resource Manager intenta crear todos los recursos especificados en la plantilla. Si el recurso ya existe en el grupo de recursos y su configuración es igual, no se realizará ninguna operación en ese recurso. Si cambia los valores de propiedad de un recurso, el recurso se actualiza con esos nuevos valores. Si intenta actualizar la ubicación o el tipo de un recurso existente, la implementación produce un error. En su lugar, implemente un nuevo recurso con la ubicación o escriba la que necesite.

## <a name="complete-mode"></a>Modo completo

En el modo completo, Resource Manager **elimina** los recursos que existen en el grupo de recursos pero que no se especifican en la plantilla. Los recursos especificados en la plantilla que están sin implementar porque una [condición](resource-group-authoring-templates.md#condition) se evalúa como false no se eliminan.

Hay algunas diferencias en cómo los tipos de recursos para controlar las eliminaciones de modo completo. Los recursos principales se eliminan automáticamente cuando no están en una plantilla implementada en modo completo. Por el contrario, algunos recursos secundarios no se eliminan automáticamente cuando no están en la plantilla. Sin embargo, estos recursos secundarios se eliminan si se elimina el recurso primario. 

Por ejemplo, si el grupo de recursos contiene una zona DNS (Microsoft.Network/tipo de recurso dnsZones) y un registro CNAME (Microsoft.Network/dnsZones/tipo de recurso CNAME), la zona DNS estará en el recurso principal del registro CNAME. Si la implementación se realiza en modo completo y no se incluye la zona DNS en la plantilla, se eliminará tanto la zona DNS como el registro CNAME. Si incluye la zona DNS en la plantilla, pero no incluyen el registro CNAME, no se elimina el registro CNAME. 

Para ver una lista del modo en que los diferentes tipos de recursos administran las eliminaciones, consulte [Eliminación de recursos de Azure en implementaciones en modo completo](complete-mode-deletion.md).

Si el grupo de recursos es [bloqueado](resource-group-lock-resources.md), el modo completo, no elimina los recursos.

> [!NOTE]
> Solo las plantillas de nivel de raíz admiten el modo de implementación completa. En cuanto a [ plantillas vinculadas o anidadas ](resource-group-linked-templates.md), debe usar el modo incremental. 
>
> [Las implementaciones de nivel de suscripción](deploy-to-subscription.md) no son compatibles con el modo completo.
>
> Actualmente, el portal no admite el modo completo.
>

## <a name="incremental-mode"></a>Modo incremental

En el modo incremental, Resource Manager **deja sin modificar** los recursos que existen en el grupo de recursos pero que no se especifican en la plantilla.

Sin embargo, al volver a implementar un recurso existente en el modo incremental, el resultado es una diferente. Especifique todas las propiedades para el recurso, no solo a los que va a actualizar. Un malentendido frecuente es pensar en las propiedades que no se especifican son permanecen sin cambios. Si no especifica determinadas propiedades, Resource Manager interpreta la actualización como que sobrescribe esos valores.

## <a name="example-result"></a>Resultado de ejemplo

Para ilustrar la diferencia entre el modo incremental y el completo, considere el escenario siguiente.

El **grupo de recursos** contiene:

* Recurso A
* Recurso B
* Recurso C

La **plantilla** contiene:

* Recurso A
* Recurso B
* Recurso D

Cuando se implementa en modo **incremental**, el grupo de recursos contiene:

* Recurso A
* Recurso B
* Recurso C
* Recurso D

Cuando se implementa en modo **completo**, se elimina el recurso C. El grupo de recursos contiene:

* Recurso A
* Recurso B
* Recurso D

## <a name="set-deployment-mode"></a>Establecimiento del modo de implementación

Para establecer el modo de implementación al implementar con PowerShell, use el parámetro `Mode`.

```azurepowershell-interactive
New-AzResourceGroupDeployment `
  -Mode Complete `
  -Name ExampleDeployment `
  -ResourceGroupName ExampleResourceGroup `
  -TemplateFile c:\MyTemplates\storage.json
```

Para establecer el modo de implementación al implementar con la CLI de Azure, use el parámetro `mode`.

```azurecli-interactive
az group deployment create \
  --name ExampleDeployment \
  --mode Complete \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters storageAccountType=Standard_GRS
```

En el siguiente ejemplo se muestra un conjunto de plantillas vinculadas al modo de implementación incremental:

```json
"resources": [
  {
      "apiVersion": "2017-05-10",
      "name": "linkedTemplate",
      "type": "Microsoft.Resources/deployments",
      "properties": {
          "mode": "Incremental",
          <nested-template-or-external-template>
      }
  }
]
```

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre la creación de plantillas del Administrador de recursos, consulte [Creación de plantillas del Administrador de recursos de Azure](resource-group-authoring-templates.md).
* Para obtener más información sobre la implementación de recursos, consulte [Implementación de una aplicación con la plantilla del Administrador de recursos de Azure](resource-group-template-deploy.md).
* Para ver las operaciones de un proveedor de recursos, consulte [API de REST de Azure](/rest/api/).
