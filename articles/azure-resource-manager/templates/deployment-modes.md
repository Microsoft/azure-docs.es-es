---
title: Modos de implementación
description: Descripción de cómo especificar si desea usar un modo de implementación completa o incremental con Azure Resource Manager.
ms.topic: conceptual
ms.date: 07/22/2020
ms.openlocfilehash: e584acd4af1dc6adb5f5d383acd5d16da0815f32
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87371590"
---
# <a name="azure-resource-manager-deployment-modes"></a>Modos de implementación de Azure Resource Manager

Al implementar los recursos, debe especificar si la implementación es una actualización incremental o una actualización completa. La diferencia entre estos dos modos es la forma en que Resource Manager controla los recursos existentes en el grupo de recursos que no están en la plantilla.

En ambos modos, Resource Manager intenta crear todos los recursos especificados en la plantilla. Si el recurso ya existe en el grupo de recursos y su configuración es igual, no se realizará ninguna operación en ese recurso. Si cambia los valores de propiedad de un recurso, el recurso se actualiza con esos nuevos valores. Si intenta actualizar la ubicación o el tipo de un recurso existente, la implementación produce un error. En su lugar, implemente un nuevo recurso con la ubicación o escriba la que necesite.

El modo predeterminado es el incremental.

## <a name="complete-mode"></a>Modo completo

En el modo completo, Resource Manager **elimina** los recursos que existen en el grupo de recursos pero que no se especifican en la plantilla.

> [!NOTE]
> Use siempre la [operación what-if](template-deploy-what-if.md) antes de implementar una plantilla en el modo completo. What-if muestra qué recursos se crearán, eliminarán o modificarán. Use what-if para evitar la eliminación accidental de recursos.

Si la plantilla incluye un recurso que no se ha implementado porque la [condición](conditional-resource-deployment.md) se evalúa como false, el resultado depende de la versión de API REST que use para implementar la plantilla. Si usa una versión anterior a 2019-05-10, el recurso **no se elimina**. Con 2019-05-10 o posterior, el recurso **se elimina**. Las versiones más recientes de Azure PowerShell y la CLI de Azure eliminan el recurso.

Tenga cuidado al usar el modo completo con [bucles de copia](copy-resources.md). Se eliminan todos los recursos que no se especifican en la plantilla después de resolver el bucle de copia.

Si implementa en [más de un grupo de recursos en una plantilla](cross-scope-deployment.md), se pueden eliminar los recursos del grupo de recursos especificado en la operación de implementación. Los recursos de los grupos de recursos secundarios no se eliminarán.

Hay algunas diferencias en la forma en que los diferentes tipos de recursos controlan las eliminaciones en modo completo. Los recursos principales se eliminan automáticamente cuando no están en una plantilla implementada en modo completo. Por el contrario, algunos recursos secundarios no se eliminan automáticamente cuando no están en la plantilla. A pesar de ello, estos recursos secundarios sí se eliminan si también se elimina el recurso principal.

Por ejemplo, si el grupo de recursos contiene una zona DNS (Microsoft.Network/tipo de recurso dnsZones) y un registro CNAME (Microsoft.Network/dnsZones/tipo de recurso CNAME), la zona DNS estará en el recurso principal del registro CNAME. Si la implementación se realiza en modo completo y no se incluye la zona DNS en la plantilla, se eliminará tanto la zona DNS como el registro CNAME. Si se incluye la zona DNS en la plantilla, pero no se incluye el registro CNAME, el registro CNAME no se eliminará.

Para ver una lista del modo en que los diferentes tipos de recursos administran las eliminaciones, consulte [Eliminación de recursos de Azure en implementaciones en modo completo](complete-mode-deletion.md).

Si el grupo de recursos está [bloqueado](../management/lock-resources.md), el modo completo no elimina los recursos.

> [!NOTE]
> Solo las plantillas de nivel de raíz admiten el modo de implementación completa. En cuanto a [ plantillas vinculadas o anidadas ](linked-templates.md), debe usar el modo incremental.
>
> Las [implementaciones de nivel de suscripción](deploy-to-subscription.md) no son compatibles con el modo completo.
>
> Actualmente, el portal no admite el modo completo.
>

## <a name="incremental-mode"></a>Modo incremental

En el modo incremental, Resource Manager **deja sin modificar** los recursos que existen en el grupo de recursos pero que no se especifican en la plantilla. Los recursos de la plantilla **se agregan** al grupo de recursos.

> [!NOTE]
> Al volver a implementar un recurso existente en el modo incremental, se aplican todas las propiedades de nuevo. Las **propiedades no se agregan de manera incremental**. Suele pensarse erróneamente que las propiedades que no se especifican en la plantilla se mantienen sin cambios. Si no se especifican determinadas propiedades, Resource Manager interpreta la implementación como que sobrescribe esos valores. Las propiedades que no se incluyen en la plantilla se restablecen a los valores predeterminados. Especifique todos los valores no predeterminados del recurso, no solo los que vaya a actualizar. La definición del recurso en la plantilla siempre contiene el estado final del recurso. No puede representar una actualización parcial de un recurso existente.
>
> En raras ocasiones, las propiedades que se especifican para un recurso se implementan realmente como un recurso secundario. Por ejemplo, al proporcionar los valores de configuración del sitio para una aplicación web, esos valores se implementan en el tipo de recurso secundario `Microsoft.Web/sites/config`. Si vuelve a implementar la aplicación web y especifica un objeto vacío para los valores de configuración del sitio, el recurso secundario no se actualiza. Sin embargo, si proporciona nuevos valores de configuración de sitio, se actualiza el tipo de recurso secundario.

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
az deployment group create \
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
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2017-05-10",
      "name": "linkedTemplate",
      "properties": {
          "mode": "Incremental",
          <nested-template-or-external-template>
      }
  }
]
```

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre la creación de plantillas del Administrador de recursos, consulte [Creación de plantillas del Administrador de recursos de Azure](template-syntax.md).
* Para obtener más información sobre la implementación de recursos, consulte [Implementación de una aplicación con la plantilla del Administrador de recursos de Azure](deploy-powershell.md).
* Para ver las operaciones de un proveedor de recursos, consulte [API de REST de Azure](/rest/api/).
