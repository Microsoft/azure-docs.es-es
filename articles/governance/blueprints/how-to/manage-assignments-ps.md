---
title: Administración de asignaciones con PowerShell
description: Obtenga información sobre cómo administrar las asignaciones de planos técnicos con el módulo oficial de PowerShell de Azure Blueprints, Az.Blueprint.
ms.date: 08/27/2020
ms.topic: how-to
ms.openlocfilehash: 3bcb3731bd1270497945fa86406d08b2f9750c85
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89051413"
---
# <a name="how-to-manage-assignments-with-powershell"></a>Administración de asignaciones con PowerShell

Una asignación de plano técnico se puede administrar mediante módulo de Azure PowerShell **Az.Blueprint**. El módulo admite capturar, crear, actualizar y quitar las asignaciones. El módulo también puede recuperar detalles sobre las definiciones de los planos técnicos existentes. En este artículo se explica cómo instalar el módulo y empezar a usarlo.

## <a name="add-the-azblueprint-module"></a>Adición del módulo Az.Blueprint

A fin de habilitar Azure PowerShell para administrar asignaciones de planos técnicos, debe agregar el módulo. Este módulo se puede usar con PowerShell y PowerShell Core instalados localmente, con [Azure Cloud Shell](https://shell.azure.com) o con la [imagen de Docker de Azure PowerShell](https://hub.docker.com/r/azuresdk/azure-powershell/).

### <a name="base-requirements"></a>Requisitos básicos

El módulo de Azure Blueprints requiere el software siguiente:

- Azure PowerShell 1.5.0 o posterior. Si todavía no está instalado, siga [estas instrucciones](/powershell/azure/install-az-ps).
- PowerShellGet 2.0.1 o una versión posterior. Si no está instalado o actualizado, siga [estas instrucciones](/powershell/scripting/gallery/installing-psget).

### <a name="install-the-module"></a>Instalación del módulo

El módulo de Azure Blueprints para PowerShell es **Az.Blueprint**.

1. Ejecute el siguiente comando como **administrador** en un símbolo del sistema de PowerShell:

   ```azurepowershell-interactive
   # Install the Azure Blueprints module from PowerShell Gallery
   Install-Module -Name Az.Blueprint
   ```

   > [!NOTE]
   > Si **Az.Accounts** está ya instalado, puede que sea necesario usar `-AllowClobber` para forzar la instalación.

1. Asegúrese de que el módulo se haya importado y que sea la versión correcta (0.2.6):

   ```azurepowershell-interactive
   # Get a list of commands for the imported Az.Blueprint module
   Get-Command -Module 'Az.Blueprint' -CommandType 'Cmdlet'
   ```

## <a name="get-blueprint-definitions"></a>Obtención de las definiciones de planos técnicos

El primer paso para trabajar con una asignación a menudo es obtener una referencia a una definición de plano técnico.
El cmdlet `Get-AzBlueprint` obtiene una o varias definiciones de planos técnicos. El cmdlet puede obtener las definiciones de planos técnicos de un grupo de administración con `-ManagementGroupId {mgId}` o una suscripción con `-SubscriptionId {subId}`. El parámetro **Name** obtiene una definición del plano técnico, pero se debe utilizar con **ManagementGroupId** o **SubscriptionId**. **Version** puede utilizarse con **Name** para explicitar con más detalle qué definición de plano técnico se devuelve. En lugar de **Version**, el conmutador `-LatestPublished` toma la versión publicada más recientemente.

En el ejemplo siguiente se usa `Get-AzBlueprint` para obtener todas las versiones de una definición de plano técnico denominada "101-blueprints-definition-subscription" de una suscripción específica que se representa como `{subId}`:

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Get all versions of the blueprint definition in the specified subscription
$blueprints = Get-AzBlueprint -SubscriptionId '{subId}' -Name '101-blueprints-definition-subscription'

# Display the blueprint definition object
$blueprints
```

La salida de ejemplo para una definición de plano técnico con varias versiones tiene este aspecto:

```output
Name                 : 101-blueprints-definition-subscription
Id                   : /subscriptions/{subId}/providers/Microsoft.Blueprint/blueprints/101
                       -blueprints-definition-subscription
DefinitionLocationId : {subId}
Versions             : {1.0, 1.1}
TimeCreated          : 2019-02-25
TargetScope          : Subscription
Parameters           : {storageAccount_storageAccountType, storageAccount_location,
                       allowedlocations_listOfAllowedLocations, [Usergrouporapplicationname]:Reader_RoleAssignmentName}
ResourceGroups       : ResourceGroup
```

Los [parámetros de plano técnico](../concepts/parameters.md#blueprint-parameters) en la definición del plano técnico se pueden ampliar para ofrecer más información.

```azurepowershell-interactive
$blueprints.Parameters
```

```output
Key                                                    Value
---                                                    -----
storageAccount_storageAccountType                      Microsoft.Azure.Commands.Blueprint.Models.PSParameterDefinition
storageAccount_location                                Microsoft.Azure.Commands.Blueprint.Models.PSParameterDefinition
allowedlocations_listOfAllowedLocations                Microsoft.Azure.Commands.Blueprint.Models.PSParameterDefinition
[Usergrouporapplicationname]:Reader_RoleAssignmentName Microsoft.Azure.Commands.Blueprint.Models.PSParameterDefinition
```

## <a name="get-blueprint-assignments"></a>Obtención de asignaciones de planos técnicos

Si la asignación de plano técnico ya existe, puede obtener una referencia a ella con el cmdlet `Get-AzBlueprintAssignment`. El cmdlet toma **SubscriptionId** y **Name** como parámetros opcionales. Si no se especifica **SubscriptionId**, se usa el contexto de la suscripción actual.

En el ejemplo siguiente se usa `Get-AzBlueprintAssignment` para obtener una asignación de plano técnico única denominada "Assignment-lock-resource-groups" de una suscripción específica representada como `{subId}`:

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Get the blueprint assignment in the specified subscription
$blueprintAssignment = Get-AzBlueprintAssignment -SubscriptionId '{subId}' -Name 'Assignment-lock-resource-groups'

# Display the blueprint assignment object
$blueprintAssignment
```

La salida de ejemplo para una asignación de plano técnico tiene este aspecto:

```output
Name              : Assignment-lock-resource-groups
Id                : /subscriptions/{subId}/providers/Microsoft.Blueprint/blueprintAssignme
                    nts/Assignment-lock-resource-groups
Scope             : /subscriptions/{subId}
LastModified      : 2019-02-19
LockMode          : AllResourcesReadOnly
ProvisioningState : Succeeded
Parameters        :
ResourceGroups    : ResourceGroup
```

## <a name="create-blueprint-assignments"></a>Creación de asignaciones de planos técnicos

Si aún no existe la asignación de plano técnico, puede crearla con el cmdlet `New-AzBlueprintAssignment`. Este cmdlet utiliza los siguientes parámetros:

- **Name** [obligatorio]
  - Especifica el nombre de la asignación de plano técnico.
  - Debe ser único y no existir ya en **SubscriptionId**.
- **Blueprint** [obligatorio]
  - Especifica la definición de plano técnico para asignar.
  - Use `Get-AzBlueprint` para obtener el objeto de referencia.
- **Location** [obligatorio]
  - Especifica la región para la identidad administrada asignada por el sistema y el objeto de implementación de suscripción en que se creará.
- **Subscription** (opcional)
  - Especifica la suscripción en que se implementa la asignación.
  - Si no se proporciona, el valor predeterminado es el contexto de la suscripción actual.
- **Lock** (opcional)
  - Define el [bloqueo de recursos de planos técnicos](../concepts/resource-locking.md) que se utilizará para los recursos implementados.
  - Opciones admitidas: _None_, _AllResourcesReadOnly_, _AllResourcesDoNotDelete_
  - Si no se proporciona, el valor predeterminado es _None_.
- **SystemAssignedIdentity** (opcional)
  - Selecciónelo para crear una identidad administrada asignada por el sistema para la asignación y para implementar los recursos.
  - Valor predeterminado para el conjunto de parámetros "identity".
  - No se puede usar con **UserAssignedIdentity**.
- **UserAssignedIdentity** (opcional)
  - Especifica la identidad administrada asignada por el usuario que se usará para la asignación y para implementar los recursos.
  - Parte del conjunto de parámetros "identity".
  - No se puede usar con **SystemAssignedIdentity**.
- **Parameter** (opcional)
  - Una [tabla hash](/powershell/module/microsoft.powershell.core/about/about_hash_tables) de pares clave/valor para establecer [parámetros dinámicos](../concepts/parameters.md#dynamic-parameters) en la asignación de plano técnico.
  - El valor predeterminado para un parámetro dinámico es **defaultValue** en la definición.
  - Si un parámetro no se proporciona y no tiene ningún valor **defaultValue**, el parámetro no es opcional.

    > [!NOTE]
    > **Parameter** no es compatible con secureStrings.

- **ResourceGroupParameter** (opcional)
  - Un [tabla hash](/powershell/module/microsoft.powershell.core/about/about_hash_tables) de artefactos de un grupo de recursos
  - Cada marcador de posición del artefacto del grupo de recursos tiene un par clave/valor para establecer dinámicamente los parámetros **Name** y **Location** en dicho artefacto del grupo de recursos.
  - Si no se proporciona un parámetro para el grupo de recursos y no tiene ningún valor **defaultValue**, el parámetro del grupo de recursos no es opcional.
- **AssignmentFile** (opcional)
  - La ruta de acceso a la representación de un archivo JSON de una asignación de plano técnico
  - Este parámetro forma parte de un conjunto de parámetros de PowerShell que únicamente incluye **Name**, **Blueprint** y **SubscriptionId**, además de los parámetros comunes.

### <a name="example-1-provide-parameters"></a>Ejemplo 1: Suministro de parámetros

En el ejemplo siguiente se crea una nueva asignación de la versión "1.1" de la definición de plano técnico "my-blueprint" capturada con `Get-AzBlueprint`, establece la ubicación del objeto de asignación y la identidad aministrada en "westus2", bloquea los recursos con _AllResourcesReadOnly_ y establece las tablas hash para **Parameter** y **ResourceGroupParameter** en la suscripción específica que se representa como `{subId}`:

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Get version '1.1' of the blueprint definition in the specified subscription
$bpDefinition = Get-AzBlueprint -SubscriptionId '{subId}' -Name 'my-blueprint' -Version '1.1'

# Create the hash table for Parameters
$bpParameters = @{storageAccount_storageAccountType='Standard_GRS'}

# Create the hash table for ResourceGroupParameters
# ResourceGroup is the resource group artifact placeholder name
$bpRGParameters = @{ResourceGroup=@{name='storage_rg';location='westus2'}}

# Create the new blueprint assignment
$bpAssignment = New-AzBlueprintAssignment -Name 'my-blueprint-assignment' -Blueprint $bpDefinition `
    -SubscriptionId '{subId}' -Location 'westus2' -Lock AllResourcesReadOnly `
    -Parameter $bpParameters -ResourceGroupParameter $bpRGParameters
```

La salida de ejemplo para crear asignación de plano técnico tiene este aspecto:

```output
Name              : my-blueprint-assignment
Id                : /subscriptions/{subId}/providers/Microsoft.Blueprint/blueprintAssi
                    gnments/my-blueprint-assignment
Scope             : /subscriptions/{subId}
LastModified      : 2019-03-13
LockMode          : AllResourcesReadOnly
ProvisioningState : Creating
Parameters        : {storageAccount_storageAccountType}
ResourceGroups    : ResourceGroup
```

### <a name="example-2-use-a-json-assignment-definition-file"></a>Ejemplo 2: Uso de un archivo de definición de asignación JSON

En el ejemplo siguiente se crea prácticamente la misma asignación que en el [ejemplo 1](#example-1-provide-parameters). En lugar de pasar parámetros al cmdlet, en el ejemplo se muestra el uso de un archivo de definición de asignación JSON y el parámetro **AssignmentFile**. Además, la propiedad **excludedPrincipals** está configurada como parte de **locks**. No hay un parámetro de PowerShell para **excludedPrincipals** y la propiedad solo se puede configurar mediante el archivo de definición de asignación JSON.

```json
{
  "identity": {
    "type": "SystemAssigned"
  },
  "location": "westus2",
  "properties": {
    "description": "Assignment of the 101-blueprint-definition-subscription",
    "blueprintId": "/subscriptions/{subId}/providers/Microsoft.Blueprint/blueprints/101-blueprints-definition-subscription",
    "locks": {
      "mode": "AllResourcesReadOnly",
      "excludedPrincipals": [
          "7be2f100-3af5-4c15-bcb7-27ee43784a1f",
          "38833b56-194d-420b-90ce-cff578296714"
      ]
    },
    "parameters": {
      "storageAccount_storageAccountType": {
        "value": "Standard_GRS"
      }
    },
    "resourceGroups": {
      "ResourceGroup": {
        "name": "storage_rg",
        "location": "westus2"
      }
    }
  }
}
```

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Create the new blueprint assignment
$bpAssignment = New-AzBlueprintAssignment -Name 'my-blueprint-assignment' -SubscriptionId '{subId}' `
    -AssignmentFile '.\assignment.json'
```

Para un ejemplo del archivo de definición de asignación JSON para una identidad administrada asignada por el usuario, consulte el cuerpo de la solicitud en [Ejemplo: Asignación con la identidad administrada asignada por el usuario](/rest/api/blueprints/assignments/createorupdate#examples) para API REST.

## <a name="update-blueprint-assignments"></a>Actualización de asignaciones de planos técnicos

A veces es necesario actualizar una asignación de plano técnico que ya se ha creado. El cmdlet `Set-AzBlueprintAssignment` se ocupa de ello. El cmdlet toma la mayor parte de los mismos parámetros que el cmdlet `New-AzBlueprintAssignment`, lo que permite que se actualice todo lo que se estableció en la asignación. Las excepciones son los parámetros _Name_, _Blueprint_ y _SubscriptionId_. Solo se actualizan los valores proporcionados.

Para comprender qué ocurre cuando se actualiza una asignación de plano técnico, consulte las [reglas para actualizar asignaciones](./update-existing-assignments.md#rules-for-updating-assignments).

- **Name** [obligatorio]
  - Especifica el nombre de la asignación de plano técnico que se va a actualizar.
  - Se utiliza para buscar la asignación para actualizar, no para cambiar la asignación.
- **Blueprint** [obligatorio]
  - Especifica la definición del plano técnico de la asignación de plano técnico.
  - Use `Get-AzBlueprint` para obtener el objeto de referencia.
  - Se utiliza para buscar la asignación para actualizar, no para cambiar la asignación.
- **Location** (opcional)
  - Especifica la región para la identidad administrada asignada por el sistema y el objeto de implementación de suscripción en que se creará.
- **Subscription** (opcional)
  - Especifica la suscripción en que se implementa la asignación.
  - Si no se proporciona, el valor predeterminado es el contexto de la suscripción actual.
  - Se utiliza para buscar la asignación para actualizar, no para cambiar la asignación.
- **Lock** (opcional)
  - Define el [bloqueo de recursos de planos técnicos](../concepts/resource-locking.md) que se utilizará para los recursos implementados.
  - Opciones admitidas: _None_, _AllResourcesReadOnly_, _AllResourcesDoNotDelete_
- **SystemAssignedIdentity** (opcional)
  - Selecciónelo para crear una identidad administrada asignada por el sistema para la asignación y para implementar los recursos.
  - Valor predeterminado para el conjunto de parámetros "identity".
  - No se puede usar con **UserAssignedIdentity**.
- **UserAssignedIdentity** (opcional)
  - Especifica la identidad administrada asignada por el usuario que se usará para la asignación y para implementar los recursos.
  - Parte del conjunto de parámetros "identity".
  - No se puede usar con **SystemAssignedIdentity**.
- **Parameter** (opcional)
  - Una [tabla hash](/powershell/module/microsoft.powershell.core/about/about_hash_tables) de pares clave/valor para establecer [parámetros dinámicos](../concepts/parameters.md#dynamic-parameters) en la asignación de plano técnico.
  - El valor predeterminado para un parámetro dinámico es **defaultValue** en la definición.
  - Si un parámetro no se proporciona y no tiene ningún valor **defaultValue**, el parámetro no es opcional.

    > [!NOTE]
    > **Parameter** no es compatible con secureStrings.

- **ResourceGroupParameter** (opcional)
  - Un [tabla hash](/powershell/module/microsoft.powershell.core/about/about_hash_tables) de artefactos de un grupo de recursos
  - Cada marcador de posición del artefacto del grupo de recursos tiene un par clave/valor para establecer dinámicamente los parámetros **Name** y **Location** en dicho artefacto del grupo de recursos.
  - Si no se proporciona un parámetro para el grupo de recursos y no tiene ningún valor **defaultValue**, el parámetro del grupo de recursos no es opcional.

En el ejemplo siguiente se actualiza la asignación de versión "1.1" de la definición de plano técnico "my-blueprint" capturada con `Get-AzBlueprint` cambiando el modo de bloqueo:

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Get version '1.1' of the blueprint definition in the specified subscription
$bpDefinition = Get-AzBlueprint -SubscriptionId '{subId}' -Name 'my-blueprint' -Version '1.1'

# Update the existing blueprint assignment
$bpAssignment = Set-AzBlueprintAssignment -Name 'my-blueprint-assignment' -Blueprint $bpDefinition `
    -SubscriptionId '{subId}' -Lock AllResourcesDoNotDelete
```

La salida de ejemplo para crear asignación de plano técnico tiene este aspecto:

```output
Name              : my-blueprint-assignment
Id                : /subscriptions/{subId}/providers/Microsoft.Blueprint/blueprintAssi
                    gnments/my-blueprint-assignment
Scope             : /subscriptions/{subId}
LastModified      : 2019-03-13
LockMode          : AllResourcesDoNotDelete
ProvisioningState : Updating
Parameters        : {storageAccount_storageAccountType}
ResourceGroups    : ResourceGroup
```

## <a name="remove-blueprint-assignments"></a>Eliminación de asignaciones de planos técnicos

Cuando llega el momento de quitar una asignación de plano técnico, el cmdlet `Remove-AzBlueprintAssignment` se ocupa de ello. El cmdlet toma **Name** o **InputObject** para especificar la asignación que se va a quitar. **SubscriptionId** es _obligatorio_ y se debe proporcionar en todos los casos.

En el ejemplo siguiente se recupera una asignación de plano técnico existente con `Get-AzBlueprintAssignment` y, a continuación, se quita la suscripción específica que se representa como `{subId}`:

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Get the blueprint assignment in the specified subscription
$blueprintAssignment = Get-AzBlueprintAssignment -Name 'Assignment-lock-resource-groups'

# Remove the existing blueprint assignment
Remove-AzBlueprintAssignment -InputObject $blueprintAssignment -SubscriptionId '{subId}'
```

## <a name="code-example"></a>Ejemplo de código

Reuniendo todos estos pasos, en el ejemplo siguiente se obtiene la definición del plano técnico y, continuación, se crea, actualiza y quita una asignación de plano técnico de la suscripción específica que se representa como `{subId}`:

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

#region GetBlueprint
# Get version '1.1' of the blueprint definition in the specified subscription
$bpDefinition = Get-AzBlueprint -SubscriptionId '{subId}' -Name 'my-blueprint' -Version '1.1'
#endregion

#region CreateAssignment
# Create the hash table for Parameters
$bpParameters = @{storageAccount_storageAccountType='Standard_GRS'}

# Create the hash table for ResourceGroupParameters
# ResourceGroup is the resource group artifact placeholder name
$bpRGParameters = @{ResourceGroup=@{name='storage_rg';location='westus2'}}

# Create the new blueprint assignment
$bpAssignment = New-AzBlueprintAssignment -Name 'my-blueprint-assignment' -Blueprint $bpDefinition `
    -SubscriptionId '{subId}' -Location 'westus2' -Lock AllResourcesReadOnly `
    -Parameter $bpParameters -ResourceGroupParameter $bpRGParameters
#endregion CreateAssignment

# Wait for the blueprint assignment to finish deployment prior to the next steps

#region UpdateAssignment
# Update the existing blueprint assignment
$bpAssignment = Set-AzBlueprintAssignment -Name 'my-blueprint-assignment' -Blueprint $bpDefinition `
    -SubscriptionId '{subId}' -Lock AllResourcesDoNotDelete
#endregion UpdateAssignment

# Wait for the blueprint assignment to finish deployment prior to the next steps

#region RemoveAssignment
# Remove the existing blueprint assignment
Remove-AzBlueprintAssignment -InputObject $bpAssignment -SubscriptionId '{subId}'
#endregion
```

## <a name="next-steps"></a>Pasos siguientes

- Información acerca del [ciclo de vida del plano técnico](../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../concepts/resource-locking.md).
- Puede consultar la información de [solución de problemas generales](../troubleshoot/general.md) para resolver los problemas durante la asignación de un plano técnico.