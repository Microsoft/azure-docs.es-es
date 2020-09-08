---
title: 'Inicio rápido: Nueva asignación de directivas con Python'
description: En este inicio rápido, se usa Python para crear una asignación de Azure Policy e identificar los recursos no compatibles.
ms.date: 08/10/2020
ms.topic: quickstart
ms.custom: devx-track-python
ms.openlocfilehash: a160b9bc389bc0c902f9644887aa478f80822e60
ms.sourcegitcommit: 1aef4235aec3fd326ded18df7fdb750883809ae8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88134625"
---
# <a name="quickstart-create-a-policy-assignment-to-identify-non-compliant-resources-using-python"></a>Inicio rápido: Creación de una asignación de directiva para identificar los recursos no compatibles mediante Python

El primer paso para entender el cumplimiento en Azure es identificar el estado de sus recursos. En esta guía de inicio rápido, se crea una asignación de directiva para identificar máquinas virtuales que no usan discos administrados. Cuando haya finalizado, podrá identificar máquinas virtuales que _no estén conformes_.

La biblioteca Python se usa para administrar los recursos de Azure desde la línea de comandos o en scripts. En esta guía se explica cómo usar la biblioteca Python para crear una asignación de directiva.

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="add-the-policy-library"></a>Incorporación de la biblioteca de directiva

Para permitir que Python funcione con Azure Policy, se debe agregar la biblioteca. Esta biblioteca funciona siempre que se pueda usar Python, lo que incluye [bash en Windows 10](/windows/wsl/install-win10) o instalado localmente.

1. Compruebe que está instalada la versión más reciente de Python (al menos la **3.8**). Si aún no está instalada, descárguela en [Python.org](https://www.python.org/downloads/).

1. Compruebe que está instalada la versión más reciente de la CLI de Azure (al menos la **2.5.1**). Si aún no está instalada, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

   > [!NOTE]
   > La CLI de Azure es necesaria para que Python pueda usar la **autenticación basada en la CLI** en los ejemplos siguientes. Para información sobre otras opciones, consulte [Autenticación mediante las bibliotecas de administración de Python](/azure/developer/python/azure-sdk-authenticate).

1. Autentíquese mediante la CLI de Azure.

   ```azurecli
   az login
   ```

1. En el entorno de Python que elija, instale las bibliotecas necesarias para Azure Resource Graph:

   ```bash
   # Add the Python library for Python
   pip install azure-mgmt-policyinsights

   # Add the Resources library for Python
   pip install azure-mgmt-resource

   # Add the CLI Core library for Python for authentication (development only!)
   pip install azure-cli-core
   ```

   > [!NOTE]
   > Si Python está instalado en todos los usuarios, estos comando deben ejecutarse desde una consola con privilegios elevados.

1. Compruebe que se han instalado las bibliotecas. `azure-mgmt-policyinsights` debe tener la versión **0.5.0** o posterior, `azure-mgmt-resource` debe tener la versión **9.0.0** o posterior, y `azure-cli-core` debe tener la versión **2.5.0** o posterior.

   ```bash
   # Check each installed library
   pip show azure-mgmt-policyinsights azure-mgmt-resource azure-cli-core
   ```

## <a name="create-a-policy-assignment"></a>Creación de una asignación de directiva

En este inicio rápido, creará una asignación de directiva y asignará la definición **Auditoría de máquinas virtuales que no usan discos administrados** (`06a78e20-9358-41c9-923c-fb736d382a4d`). Esta definición de directiva identifica los recursos que no cumplen las condiciones establecidas en la definición de directiva.

Ejecute el código siguiente para crear una nueva asignación de directiva:

```python
# Import specific methods and models from other libraries
from azure.common.credentials import get_azure_cli_credentials
from azure.common.client_factory import get_client_from_cli_profile
from azure.mgmt.resource.policy import PolicyClient
from azure.mgmt.resource.policy.models import PolicyAssignment

# Get your credentials from Azure CLI (development only!) and get your subscription list
policyClient = get_client_from_cli_profile(PolicyClient)

# Create details for the assignment
policyAssignmentDetails = PolicyAssignment(display_name="Audit VMs without managed disks Assignment", policy_definition_id="/providers/Microsoft.Authorization/policyDefinitions/06a78e20-9358-41c9-923c-fb736d382a4d", scope="{scope}", description="Shows all virtual machines not using managed disks")

# Create new policy assignment
policyAssignment = policyClient.policy_assignments.create("{scope}", "audit-vm-manageddisks", policyAssignmentDetails)

# Show results
print(policyAssignment)
```

Los comandos anteriores usan la siguiente información:

Detalles de la asignación:
- **display_name**: nombre para mostrar de la asignación de directiva. En este caso, usará _Auditoría de máquinas virtuales sin discos administrados_.
- **policy_definition_id**: la ruta de acceso de la definición de directiva, según la opción utilizada para crear la asignación. En este caso, es el identificador de la definición de directiva _Auditoría de máquinas virtuales que no usan discos administrados_. En este ejemplo, la definición de directiva está integrada y la ruta de acceso no incluye la información del grupo de administración ni de la suscripción.
- **scope**: un ámbito determina en qué recursos o agrupación de recursos se aplica la asignación de directiva. Podría abarcar desde un grupo de administración a un recurso individual. Asegúrese de reemplazar `{scope}` por uno de los siguientes patrones:
  - Grupo de administración: `/providers/Microsoft.Management/managementGroups/{managementGroup}`
  - Suscripción: `/subscriptions/{subscriptionId}`
  - Grupos de recursos: `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}`
  - Recurso: `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/[{parentResourcePath}/]`
- **description**: una explicación más detallada de lo que hace la directiva o de por qué se asigna a este ámbito.

Creación de asignaciones:

- Ámbito: este ámbito determina dónde se guarda la asignación de directiva. El ámbito establecido en los detalles de la asignación debe existir dentro de este ámbito.
- Nombre: el nombre real de la asignación. En este ejemplo se usa _audit-vm-manageddisks_.
- Asignación de directiva: el objeto Python **PolicyAssignment** creado en el paso anterior.

Ahora ya está listo para identificar los recursos no compatibles a fin de conocer el estado de cumplimiento de su entorno.

## <a name="identify-non-compliant-resources"></a>Identificación de recursos sin compatibilidad

Utilice la siguiente información para identificar los recursos que no son compatibles con la asignación de directiva que ha creado. Ejecute el código siguiente:

```python
# Import specific methods and models from other libraries
from azure.common.client_factory import get_client_from_cli_profile
from azure.mgmt.policyinsights._policy_insights_client import PolicyInsightsClient
from azure.mgmt.policyinsights.models import QueryOptions

# Get your credentials from Azure CLI (development only!) and get your subscription list
policyInsightsClient = get_client_from_cli_profile(PolicyInsightsClient)

# Set the query options
queryOptions = QueryOptions(filter="IsCompliant eq false and PolicyAssignmentId eq 'audit-vm-manageddisks'",apply="groupby((ResourceId))")

# Fetch 'latest' results for the subscription
results = policyInsightsClient.policy_states.list_query_results_for_subscription(policy_states_resource="latest", subscription_id="{subscriptionId}", query_options=queryOptions)

# Show results
print(results)
```

Reemplace `{subscriptionId}` por la suscripción en la que desea ver los resultados de cumplimiento de esta asignación de directiva. Para obtener una lista de otros ámbitos y maneras de resumir los datos, consulte [Métodos de estado de las directivas](/python/api/azure-mgmt-policyinsights/azure.mgmt.policyinsights.operations.policystatesoperations#methods).

Los resultados deben tener una apariencia similar al ejemplo siguiente:

```output
{
    'additional_properties': {
        '@odata.nextLink': None
    },
    'odatacontext': 'https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest',
    'odatacount': 12,
    'value': [{data}]
}
```

Los resultados coinciden con lo que ve en la pestaña de **cumplimiento de recursos** de una asignación de directiva en la vista de Azure Portal.

## <a name="clean-up-resources"></a>Limpieza de recursos

Para quitar la asignación creada, ejecute el siguiente comando:

```python
# Import specific methods and models from other libraries
from azure.common.client_factory import get_client_from_cli_profile
from azure.mgmt.resource.policy import PolicyClient

# Get your credentials from Azure CLI (development only!) and get your subscription list
policyClient = get_client_from_cli_profile(PolicyClient)

# Delete the policy assignment
policyAssignment = policyClient.policy_assignments.delete("{scope}", "audit-vm-manageddisks")

# Show results
print(policyAssignment)
```

Reemplace `{scope}` por el mismo ámbito que usó para crear la asignación de directiva.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, se asigna una definición de directiva para identificar los recursos incompatibles en el entorno de Azure.

Para más información sobre la asignación de definiciones de directivas para asegurarse de la compatibilidad de los nuevos recursos, continúe con el tutorial para:

> [!div class="nextstepaction"]
> [Crear y administrar directivas](./tutorials/create-and-manage.md)