---
title: Implementación de Azure Policy en suscripciones delegadas a escala
description: Obtenga información sobre cómo Azure Lighthouse permite implementar una definición de directivas y una asignación de directivas en varios inquilinos.
ms.date: 08/12/2020
ms.topic: how-to
ms.openlocfilehash: 27d32a68c1a3806e514533efbae581aa97bc6d0c
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88167290"
---
# <a name="deploy-azure-policy-to-delegated-subscriptions-at-scale"></a>Implementación de Azure Policy en suscripciones delegadas a escala

Como proveedor de servicios, es posible que haya incorporado varios inquilinos de cliente para [Azure Lighthouse](../overview.md). Azure Lighthouse permite a los proveedores de servicios realizar operaciones a escala a través de varios inquilinos a la vez, lo que hace que las tareas de administración sean más eficaces.

En este tema se muestra cómo usar [Azure Policy](../../governance/policy/index.yml) para implementar una definición de directivas y una asignación de directivas en varios inquilinos mediante comandos de PowerShell. En este ejemplo, la definición de directivas garantiza que las cuentas de almacenamiento estén protegidas al permitir solo el tráfico HTTPS.

> [!TIP]
> Aunque en este tema hacemos referencia a los proveedores de servicios y clientes, las [empresas que administran varios inquilinos](../concepts/enterprise.md) pueden usar los mismos procesos.

## <a name="use-azure-resource-graph-to-query-across-customer-tenants"></a>Uso de Azure Resource Graph para consultar entre inquilinos de cliente

Puede usar [Azure Resource Graph](../../governance/resource-graph/index.yml) para realizar consultas en todas las suscripciones de los inquilinos de cliente que administra. En este ejemplo, se identificarán todas las cuentas de almacenamiento de estas suscripciones que no requieran actualmente tráfico HTTPS.  

```powershell
$MspTenant = "insert your managing tenantId here"

$subs = Get-AzSubscription

$ManagedSubscriptions = Search-AzGraph -Query "ResourceContainers | where type == 'microsoft.resources/subscriptions' | where tenantId != '$($mspTenant)' | project name, subscriptionId, tenantId" -subscription $subs.subscriptionId

Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Storage/storageAccounts' | project name, location, subscriptionId, tenantId, properties.supportsHttpsTrafficOnly" -subscription $ManagedSubscriptions.subscriptionId | convertto-json
```

## <a name="deploy-a-policy-across-multiple-customer-tenants"></a>Implementación de una directiva en varios inquilinos de cliente

En el ejemplo siguiente se muestra cómo usar una [plantilla de Azure Resource Manager](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/policy-enforce-https-storage/enforceHttpsStorage.json) para implementar una definición de directivas y una asignación de directivas entre las suscripciones delegadas en varios inquilinos de cliente. Esta definición de directivas requiere que todas las cuentas de almacenamiento usen tráfico HTTPS, lo que impide la creación de nuevas cuentas de almacenamiento que no cumplan con la especificación, y hace que las cuentas de almacenamiento existentes que no tengan esta configuración se marquen como no conformes.

```powershell
Write-Output "In total, there are $($ManagedSubscriptions.Count) delegated customer subscriptions to be managed"

foreach ($ManagedSub in $ManagedSubscriptions)
{
    Select-AzSubscription -SubscriptionId $ManagedSub.subscriptionId

    New-AzSubscriptionDeployment -Name mgmt `
                     -Location eastus `
                     -TemplateUri "https://raw.githubusercontent.com/Azure/Azure-Lighthouse-samples/master/templates/policy-enforce-https-storage/enforceHttpsStorage.json" `
                     -AsJob
}
```

## <a name="validate-the-policy-deployment"></a>Validación de la implementación de directivas

Una vez que haya implementado la plantilla de Azure Resource Manager, puede confirmar que la definición de directivas se ha aplicado correctamente; para ello, intente crear una cuenta de almacenamiento con **EnableHttpsTrafficOnly** establecido en **false** en una de las suscripciones delegadas. Debido a la asignación de directivas, no debería ser capaz de crear esta cuenta de almacenamiento.  

```powershell
New-AzStorageAccount -ResourceGroupName (New-AzResourceGroup -name policy-test -Location eastus -Force).ResourceGroupName `
                     -Name (get-random) `
                     -Location eastus `
                     -EnableHttpsTrafficOnly $false `
                     -SkuName Standard_LRS `
                     -Verbose                  
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado, elimine la definición y la asignación de directivas que creó la implementación.

```powershell
foreach ($ManagedSub in $ManagedSubscriptions)
{
    select-azsubscription -subscriptionId $ManagedSub.subscriptionId

    Remove-AzSubscriptionDeployment -Name mgmt -AsJob

    $Assignment = Get-AzPolicyAssignment | where-object {$_.Name -like "enforce-https-storage-assignment"}

    if ([string]::IsNullOrEmpty($Assignment))
    {
        Write-Output "Nothing to clean up - we're done"
    }
    else
    {

    Remove-AzPolicyAssignment -Name 'enforce-https-storage-assignment' -Scope "/subscriptions/$($ManagedSub.subscriptionId)" -Verbose

    Write-Output "Deployment has been deleted - we're done"
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

- Más información acerca de [Azure Policy](../../governance/policy/index.yml).
- Más información sobre las [experiencias de administración entre inquilinos](../concepts/cross-tenant-management-experience.md).
