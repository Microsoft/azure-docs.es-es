---
title: 'Inicio rápido: Asignación de nuevas directivas con la API REST'
description: En este inicio rápido, se usa la API REST para crear una asignación de Azure Policy para identificar recursos no compatibles.
ms.date: 08/10/2020
ms.topic: quickstart
ms.openlocfilehash: 04880ef013060bc5ff12618af6a9156295a26a88
ms.sourcegitcommit: 1aef4235aec3fd326ded18df7fdb750883809ae8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88137096"
---
# <a name="quickstart-create-a-policy-assignment-to-identify-non-compliant-resources-with-rest-api"></a>Inicio rápido: Creación de una asignación de directiva para identificar recursos no compatibles mediante la API REST

El primer paso para entender el cumplimiento en Azure es identificar el estado de sus recursos.
Esta guía de inicio rápido lo guiará por el proceso de creación de una asignación de directiva para identificar las máquinas virtuales que no están usando discos administrados.

Al finalizar este proceso, habrá identificado correctamente máquinas virtuales que no utilizan discos administrados. _No son compatibles_ con la asignación de directiva.

La API REST se usa para crear y administrar los recursos de Azure. En esta guía se usa la API REST para crear una asignación de directiva e identificar recursos no compatibles en el entorno de Azure.

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

- Si aún no lo ha hecho, instale [ARMClient](https://github.com/projectkudu/ARMClient). Se trata de una herramienta que envía solicitudes HTTP a las API REST basadas en Azure Resource Manager. Como alternativa, puede usar la característica "Pruébelo" de la documentación de REST o herramientas como [Invoke-RestMethod](/powershell/module/microsoft.powershell.utility/invoke-restmethod) o [Postman](https://www.postman.com)de PowerShell.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="create-a-policy-assignment"></a>Creación de una asignación de directiva

En este inicio rápido, creará una asignación de directiva y asignará la definición **Auditoría de máquinas virtuales que no usan discos administrados** (`06a78e20-9358-41c9-923c-fb736d382a4d`). Esta definición de directiva identifica los recursos que no cumplen las condiciones establecidas en la definición de directiva.

Ejecute el siguiente comando para crear una asignación de directiva:

   - URI DE LA API REST

     ```http
     PUT https://management.azure.com/{scope}/providers/Microsoft.Authorization/policyAssignments/audit-vm-manageddisks?api-version=2019-09-01
     ```

   - Cuerpo de la solicitud

     ```json
     {
       "properties": {
         "displayName": "Audit VMs without managed disks Assignment",
         "description": "Shows all virtual machines not using managed disks",
         "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/06a78e20-9358-41c9-923c-fb736d382a4d",
       }
     }
     ```

El punto de conexión anterior y el cuerpo de la solicitud usan la siguiente información:

URI de la API REST:
- **Scope**: un ámbito determina en qué recursos o agrupación de recursos se aplica la asignación de directiva. Podría abarcar desde un grupo de administración a un recurso individual. Asegúrese de reemplazar `{scope}` por uno de los siguientes patrones:
  - Grupo de administración: `/providers/Microsoft.Management/managementGroups/{managementGroup}`
  - Suscripción: `/subscriptions/{subscriptionId}`
  - Grupos de recursos: `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}`
  - Recurso: `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/[{parentResourcePath}/]{resourceType}/{resourceName}`
- **Nombre**: el nombre real de la asignación. En este ejemplo se usa _audit-vm-manageddisks_.

Cuerpo de la solicitud:
- **DisplayName**: nombre para mostrar de la asignación de directiva. En este caso, usará _Auditoría de máquinas virtuales sin discos administrados_.
- **Description**: una explicación más detallada de lo que hace la directiva o de por qué se asigna a este ámbito.
- **policyDefinitionId**: identificador de definición de directiva, según la opción utilizada para crear la asignación. En este caso, es el identificador de la definición de directiva _Auditoría de máquinas virtuales que no usan discos administrados_.

## <a name="identify-non-compliant-resources"></a>Identificación de recursos sin compatibilidad

Para ver los recursos que no son compatibles en esta nueva asignación, ejecute el siguiente comando para obtener los identificadores de recursos de los recursos no compatibles que se generan en un archivo JSON:

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/policyDefinitions/06a78e20-9358-41c9-923c-fb736d382a4d/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2019-10-01&$filter=IsCompliant eq false and PolicyAssignmentId eq 'audit-vm-manageddisks'&$apply=groupby((ResourceId))"
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
            "ResourceId": "/subscriptions/<subscriptionName>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachine3Id>"
        }

    ]
}
```

Son comparables a lo que normalmente vería en **Recursos no compatibles**, en la vista de Azure Portal.

## <a name="clean-up-resources"></a>Limpieza de recursos

Para quitar la asignación creada, ejecute el siguiente comando:

```http
DELETE https://management.azure.com/{scope}/providers/Microsoft.Authorization/policyAssignments/audit-vm-manageddisks?api-version=2019-09-01
```

Reemplace `{scope}` por el ámbito que utilizó cuando creó por primera vez la asignación de directiva.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, se asigna una definición de directiva para identificar los recursos incompatibles en el entorno de Azure.

Para más información sobre la asignación de directivas para garantizar la compatibilidad de los nuevos recursos, continúe con el tutorial para:

> [!div class="nextstepaction"]
> [Crear y administrar directivas](./tutorials/create-and-manage.md)
