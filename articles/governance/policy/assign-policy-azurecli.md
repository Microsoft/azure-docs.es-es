---
title: 'Inicio rápido: Asignación de directivas nuevas con la CLI de Azure'
description: En este inicio rápido, se usa la CLI de Azure para crear una asignación de Azure Policy para identificar recursos no compatibles.
ms.date: 06/29/2020
ms.topic: quickstart
ms.custom: devx-track-azurecli
ms.openlocfilehash: 2c289aa01362fcf0ede24bc1da7e8aef9e9be072
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87496702"
---
# <a name="quickstart-create-a-policy-assignment-to-identify-non-compliant-resources-with-azure-cli"></a>Inicio rápido: Creación de una asignación de directiva para identificar recursos no compatibles mediante la CLI de Azure

El primer paso para entender el cumplimiento en Azure es identificar el estado de sus recursos.
Esta guía de inicio rápido lo guiará por el proceso de creación de una asignación de directiva para identificar las máquinas virtuales que no están usando discos administrados.

Al finalizar este proceso, habrá identificado correctamente máquinas virtuales que no utilizan discos administrados. _No son compatibles_ con la asignación de directiva.

La CLI de Azure se usa para crear y administrar recursos de Azure desde la línea de comandos o en scripts. En esta guía se usa la CLI de Azure para crear una asignación de directiva e identificar recursos no compatibles en el entorno de Azure.

## <a name="prerequisites"></a>Prerrequisitos

- Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

- Este inicio rápido requiere la ejecución de la versión 2.0.76 de la CLI de Azure, o cualquier versión posterior, para instalar y usar la CLI en un entorno local. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

- Registre el proveedor de recursos de Azure Policy Insights mediante la CLI de Azure. El registro del proveedor de recursos garantiza que la suscripción funcionará con él. Para registrar un proveedor de recursos, debe tener permiso para registrar la operación del proveedor de recursos. Esta operación está incluida en los roles Colaborador y Propietario. Para registrar el proveedor de recursos, ejecute el siguiente comando:

  ```azurecli-interactive
  az provider register --namespace 'Microsoft.PolicyInsights'
  ```

  Para más información acerca del registro y la visualización de los proveedores de recursos, consulte [Tipos y proveedores de recursos](../../azure-resource-manager/management/resource-providers-and-types.md)

- Si aún no lo ha hecho, instale [ARMClient](https://github.com/projectkudu/ARMClient). Esta es una herramienta que envía solicitudes HTTP a las API basadas en Azure Resource Manager.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="create-a-policy-assignment"></a>Creación de una asignación de directiva

En esta guía de inicio rápido, creará una asignación de directiva y asignará la definición **Auditoría de máquinas virtuales que no usan discos administrados**. Esta definición de directiva identifica los recursos que no cumplen las condiciones establecidas en la definición de directiva.

Ejecute el siguiente comando para crear una asignación de directiva:

```azurecli-interactive
az policy assignment create --name 'audit-vm-manageddisks' --display-name 'Audit VMs without managed disks Assignment' --scope '<scope>' --policy '<policy definition ID>'
```

El comando anterior usa la siguiente información:

- **Nombre**: el nombre real de la asignación. En este ejemplo se usa _audit-vm-manageddisks_.
- **DisplayName**: nombre para mostrar de la asignación de directiva. En este caso, usará _Auditoría de máquinas virtuales sin discos administrados_.
- **Policy**: identificador de definición de la directiva, según la opción utilizada para crear la asignación. En este caso, es el identificador de la definición de directiva _Auditoría de máquinas virtuales que no usan discos administrados_. Para obtener el identificador de definición de directiva, ejecute este comando: `az policy definition list --query "[?displayName=='Audit VMs that do not use managed disks']"`
- **Scope**: un ámbito determina en qué recursos o agrupación de recursos se aplica la asignación de directiva. Puede abarcar desde una suscripción hasta grupos de recursos. Asegúrese de sustituir &lt;scope&gt; por el nombre del grupo de recursos.

## <a name="identify-non-compliant-resources"></a>Identificación de recursos no compatibles

Para ver los recursos que no son compatibles con esta nueva asignación, ejecute los siguientes comandos para obtener el identificador de asignación de directiva:

```azurecli-interactive
az policy assignment list --query "[?displayName=='Audit VMs without managed disks Assignment'].id"
```

Para más información sobre los identificadores de asignación de directivas, consulte [az policy assignment](/cli/azure/policy/assignment).

A continuación, ejecute el siguiente comando para obtener los identificadores de los recursos no compatibles que se copian en un archivo JSON:

```console
armclient post "/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2019-10-01&$filter=IsCompliant eq false and PolicyAssignmentId eq '<policyAssignmentID>'&$apply=groupby((ResourceId))" > <json file to direct the output with the resource IDs into>
```

Los resultados deben tener una apariencia similar al ejemplo siguiente:

```json
{
    "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest",
    "@odata.count": 3,
    "value": [{
            "@odata.id": null,
            "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
            "ResourceId": "/subscriptions/<subscriptionId>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachineId>"
        },
        {
            "@odata.id": null,
            "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
            "ResourceId": "/subscriptions/<subscriptionId>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachine2Id>"
        },
        {
            "@odata.id": null,
            "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
            "ResourceId": "/subscriptions/<subscriptionName>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachine3ID>"
        }

    ]
}
```

Son comparables a lo que normalmente vería en **Recursos no compatibles**, en la vista de Azure Portal.

## <a name="clean-up-resources"></a>Limpieza de recursos

Para quitar la asignación creada, ejecute el siguiente comando:

```azurecli-interactive
az policy assignment delete --name 'audit-vm-manageddisks' --scope '/subscriptions/<subscriptionID>/<resourceGroupName>'
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, se asigna una definición de directiva para identificar los recursos incompatibles en el entorno de Azure.

Para más información sobre la asignación de directivas para garantizar la compatibilidad de los nuevos recursos, continúe con el tutorial para:

> [!div class="nextstepaction"]
> [Crear y administrar directivas](./tutorials/create-and-manage.md)
