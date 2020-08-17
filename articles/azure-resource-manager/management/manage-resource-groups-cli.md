---
title: 'Administración de grupos de recursos: CLI de Azure'
description: Use la CLI de Azure para administrar grupos de recursos con Azure Resource Manager. Muestra cómo crear, enumerar y eliminar grupos de recursos.
author: mumian
ms.topic: conceptual
ms.date: 02/11/2019
ms.author: jgao
ms.openlocfilehash: d8cf93d8945d6597f3c7276e83df0b837490c57d
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87326858"
---
# <a name="manage-azure-resource-manager-resource-groups-by-using-azure-cli"></a>Administración de grupos de recursos de Azure Resource Manager mediante la CLI de Azure

Obtenga información sobre cómo utilizar la CLI de Azure con [Azure Resource Manager](overview.md) para administrar los grupos de recursos de Azure. Para administrar los recursos de Azure, vea [Administración de recursos de Azure mediante la CLI de Azure](manage-resources-cli.md).

Otros artículos sobre cómo administrar grupos de recursos:

- [Administración de grupos de recursos de Azure con Azure Portal](manage-resources-portal.md)
- [Administración de grupos de recursos de Azure mediante Azure PowerShell](manage-resources-powershell.md)

## <a name="what-is-a-resource-group"></a>¿Qué es un grupo de recursos?

Un grupo de recursos es un contenedor que almacena los recursos relacionados con una solución de Azure. El grupo de recursos puede incluir todos los recursos de la solución o solo aquellos que se desean administrar como grupo. Para decidir cómo asignar los recursos a los grupos de recursos, tenga en cuenta lo que más conviene a su organización. Por lo general, se recomienda agregar recursos que compartan el mismo ciclo de vida al mismo grupo de recursos para que los pueda implementar, actualizar y eliminar con facilidad como un grupo.

Los grupos de recursos almacenan metadatos acerca de los recursos. Por consiguiente, al especificar la ubicación del grupo de recursos, se especifica el lugar en que se almacenan dichos metadatos. Por motivos de compatibilidad, es posible que sea preciso asegurarse de que los datos se almacenan en una región concreta.

Los grupos de recursos almacenan metadatos acerca de los recursos. Al especificar la ubicación del grupo de recursos, se especifica el lugar en que dichos metadatos se almacenan.

## <a name="create-resource-groups"></a>Crear grupos de recursos

El siguiente script de la CLI crea un grupo de recursos y luego lo muestra.

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the location (i.e. centralus):" &&
read location &&
az group create --name $resourceGroupName --location $location
```

## <a name="list-resource-groups"></a>Enumeración de grupos de recursos

El siguiente script de la CLI enumera los grupos de recursos de su suscripción.

```azurecli-interactive
az group list
```

Para obtener un grupo de recursos:

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group show --name $resourceGroupName
```

## <a name="delete-resource-groups"></a>Eliminación de grupos de recursos

El siguiente script de la CLI elimina un grupo de recursos:

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName
```

Para obtener más información sobre cómo ordena Azure Resource Manager la eliminación de recursos, consulte [Eliminación del grupo de recursos en Azure Resource Manager](delete-resource-group.md).

## <a name="deploy-resources-to-an-existing-resource-group"></a>Implementación de recursos en un grupo de recursos existente

Consulte [Implementar recursos en un grupo de recursos](manage-resources-cli.md#deploy-resources-to-an-existing-resource-group).

## <a name="deploy-a-resource-group-and-resources"></a>Implementación de un grupo de recursos y de recursos

Puede crear un grupo de recursos e implementar recursos en el grupo por medio de una plantilla de Resource Manager. Para más información, consulte [Creación de un grupo de recursos e implementación de recursos](../templates/deploy-to-subscription.md#resource-groups).

## <a name="redeploy-when-deployment-fails"></a>Nueva implementación cuando se produce un error en la implementación

Esta característica también es conocida como *Reversión en caso de error*. Para obtener más información, consulte [Nueva implementación cuando se produce un error en la implementación](../templates/rollback-on-error.md).

## <a name="move-to-another-resource-group-or-subscription"></a>Traslado a otro grupo de recursos o suscripción

Puede mover los recursos de un grupo a otro grupo de recursos. Para más información, vea [Traslado de recursos](manage-resources-cli.md#move-resources).

## <a name="lock-resource-groups"></a>Bloqueo de grupos de recursos

Los bloqueos impiden que otros usuarios de la organización eliminen o modifiquen por error recursos esenciales, como una suscripción de Azure, un grupo de recursos o un recurso. 

El script siguiente bloquea un grupo de recursos de manera que no se pueda eliminar.

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az lock create --name LockGroup --lock-type CanNotDelete --resource-group $resourceGroupName  
```

El script siguiente obtiene todos los bloqueos de un grupo de recursos:

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az lock list --resource-group $resourceGroupName  
```

El script siguiente elimina un bloqueo:

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the lock name:" &&
read lockName &&
az lock delete --name $lockName --resource-group $resourceGroupName
```

Para obtener más información, consulte [Bloqueo de recursos con el Administrador de recursos de Azure](lock-resources.md).

## <a name="tag-resource-groups"></a>Etiquetado de grupos de recursos

Puede aplicar etiquetas a los recursos y grupos de recursos para organizar de manera lógica los recursos. Para obtener información, vea [Uso de etiquetas para organizar los recursos de Azure](tag-resources.md#azure-cli).

## <a name="export-resource-groups-to-templates"></a>Exportación de grupos de recursos a plantillas

Después de configurar el grupo de recursos correctamente, quizás quiera ver la plantilla de Resource Manager para el grupo de recursos. Exportar la plantilla ofrece dos ventajas:

- Automatice las futuras implementaciones de la solución porque la plantilla contiene la infraestructura completa.
- Obtenga más información sobre la sintaxis de la plantilla consultando la notación de objetos JavaScript (JSON) que representa la solución.

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group export --name $resourceGroupName  
```

El script muestra la plantilla en la consola.  Copie el JSON y guárdelo como un archivo.

La característica de exportación de plantillas no admite la exportación de recursos de Azure Data Factory. Para más información sobre cómo exportar recursos de Data Factory, consulte [Copia o clonación de una factoría de datos en Azure Data Factory](https://aka.ms/exportTemplateViaAdf).

Para exportar los recursos creados mediante el modelo de implementación clásica, debe [migrarlos al modelo de implementación de Resource Manager](https://aka.ms/migrateclassicresourcetoarm).

Para más información, consulte [Exportación de uno y varios recursos a la plantilla en Azure Portal](../templates/export-template-portal.md).

## <a name="manage-access-to-resource-groups"></a>Administración del acceso a los grupos de recursos

El [control de acceso basado en rol (RBAC)](../../role-based-access-control/overview.md) es la forma en que se administra el acceso a los recursos en Azure. Para obtener más información, consulte [Administración del acceso a los recursos de Azure mediante RBAC y la CLI de Azure](../../role-based-access-control/role-assignments-cli.md).

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información sobre Azure Resource Manager, consulte [Información general de Azure Resource Manager](overview.md).
- Para obtener información sobre la sintaxis de las plantillas de Resource Manager, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](../templates/template-syntax.md).
- Para obtener información sobre cómo desarrollar plantillas, consulte los [tutoriales paso a paso](../index.yml).
- Para ver los esquemas de plantilla de Azure Resource Manager, vea la [referencia de plantilla](/azure/templates/).
