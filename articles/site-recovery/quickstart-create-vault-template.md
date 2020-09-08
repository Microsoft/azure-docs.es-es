---
title: Inicio rápido para crear un almacén de Azure Recovery Services mediante una plantilla de Azure Resource Manager.
description: En este inicio rápido, aprenderá a crear un almacén de Azure Recovery Services mediante una plantilla de Azure Resource Manager.
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 04/29/2020
ms.openlocfilehash: 62bfe71c397367282c696eecdf55ab9c900c69a1
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88684123"
---
# <a name="quickstart-create-a-recovery-services-vault-using-an-arm-template"></a>Inicio rápido: Creación de un almacén de Recovery Services mediante una plantilla de Resource Manager

En este inicio rápido, se describe cómo instalar un almacén de Recovery Services mediante una plantilla de Azure Resource Manager. El servicio [Azure Site Recovery](site-recovery-overview.md) contribuye a la estrategia de continuidad empresarial y recuperación ante desastres (BCDR) para que las aplicaciones empresariales permanezcan en línea durante las interrupciones planeadas y no planeadas. Site Recovery administra la recuperación ante desastres de máquinas locales y las máquinas virtuales de Azure (VM), lo que incluye su replicación, conmutación por error y recuperación.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-recovery-services-vault-create%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción activa a Azure, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-recovery-services-vault-create/).

:::code language="json" source="~/quickstart-templates/101-recovery-services-vault-create/azuredeploy.json":::

En la plantilla se definen dos recursos de Azure:

- [Microsoft.RecoveryServices vaults](/azure/templates/microsoft.recoveryservices/vaults): crea el almacén.
- [Microsoft.RecoveryServices/vaults/backupstorageconfig](/rest/api/backup/backupresourcestorageconfigs): configura la redundancia de copia de seguridad del almacén.

La plantilla incluye parámetros opcionales para la configuración de las copias de seguridad del almacén. Los valores de redundancia del almacenamiento son: almacenamiento con redundancia local (LRS) o almacenamiento con redundancia geográfica (GRS). Para más información, consulte el apartado[Establecimiento de la redundancia de almacenamiento](../backup/backup-create-rs-vault.md#set-storage-redundancy).

Para ver más plantillas de Azure Recovery Services, consulte [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Recoveryservices&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Implementación de la plantilla

Para implementar la plantilla, se requieren los valores de **Suscripción**, **Grupo de recursos** y **Nombre de almacén**.

1. Para iniciar sesión en Azure y abrir la plantilla, seleccione la imagen **Implementar en Azure**.

   [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-recovery-services-vault-create%2Fazuredeploy.json)

1. Seleccione o escriba los siguientes valores:

   :::image type="content" source="media/quickstart-create-vault-template/create-vault-template.png" alt-text="Plantilla para crear un almacén de Recovery Services.":::

   - **Suscripción**: seleccione su suscripción de Azure.
   - **Grupo de recursos**: seleccione un grupo existente, o bien seleccione **Crear nuevo** para agregar un grupo.
   - **Ubicación**: el valor predeterminado es la ubicación del grupo de recursos y deja de estar disponible cuando se selecciona un grupo de recursos.
   - **Nombre del almacén**: escriba el nombre del almacén.
   - **Change Storage Type** (Cambiar tipo de almacenamiento): El valor predeterminado es **false**. Seleccione **true** solo si necesita cambiar el tipo de almacenamiento del almacén.
   - **Vault Storage Type** (Tipo de almacenamiento del almacén): el valor predeterminado es **GloballyRedundant**. Si el tipo de almacenamiento se ha establecido en **true**, seleccione **LocallyRedundant**.
   - **Location** (Ubicación): el valor predeterminado de la función `[resourceGroup().location]` es la ubicación del grupo de recursos. Para cambiar la ubicación, escriba un valor como **westus**.
   - Seleccione la casilla **Acepto los términos y condiciones indicados anteriormente**.

1. Para comenzar la implementación del almacén, seleccione el botón **Comprar**. Después de que la implementación haya finalizado correctamente, se muestra una notificación.

   :::image type="content" source="media/quickstart-create-vault-template/deployment-success.png" alt-text="La implementación del almacén se realizó correctamente.":::

## <a name="validate-the-deployment"></a>Validación de la implementación

Para confirmar que se ha creado el almacén, use la CLI de Azure o Azure PowerShell.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
echo "Enter the resource group name:" &&
read resourceGroupName &&
echo "Enter the vault name:" &&
read vaultName &&
az backup vault show --name $vaultName --resource-group $resourceGroupName &&
az backup vault backup-properties show --name $vaultName --resource-group $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resouceGroupName = Read-Host -Prompt "Enter the resource group name"
$vaultName = Read-Host -Prompt "Enter the vault name"
$vaultBackupConfig = Get-AzRecoveryServicesVault -Name $vaultName
Get-AzRecoveryServicesVault -ResourceGroupName $resouceGroupName -Name $vaultName
Get-AzRecoveryServicesBackupProperty -Vault $vaultBackupConfig
Write-Host "Press [ENTER] to continue..."
```

---

La siguiente salida es un extracto de la información del almacén:

# <a name="cli"></a>[CLI](#tab/CLI)

```Output
"id": "/subscriptions/<Subscription Id>/resourceGroups/myResourceGroup
         /providers/Microsoft.RecoveryServices/vaults/myVault"
"location": "eastus"
"name": "myVault"
"resourceGroup": "myResourceGroup"

"storageModelType": "GeoRedundant"
"storageType": "GeoRedundant"
"type": "Microsoft.RecoveryServices/vaults/backupstorageconfig"
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```Output
Name              : myVault
Type              : Microsoft.RecoveryServices/vaults
Location          : eastus
ResourceGroupName : myResourceGroup
SubscriptionId    : <Subscription Id>

BackupStorageRedundancy
-----------------------
GeoRedundant
```

---

## <a name="clean-up-resources"></a>Limpieza de recursos

Si planea usar los nuevos recursos, no es necesario realizar ninguna acción. De lo contrario, puede quitar el grupo de recursos y el almacén que se crearon en este inicio rápido. Para eliminar tanto el grupo de recursos como sus recursos, utilice la CLI de Azure o Azure PowerShell.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
echo "Enter the resource group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resouceGroupName = Read-Host -Prompt "Enter the resource group name"
Remove-AzResourceGroup -Name $resouceGroupName
Write-Host "Press [ENTER] to continue..."
```

---

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un almacén de Recovery Services. Para más información sobre la recuperación ante desastres, pase al siguiente artículo del inicio rápido.

> [!div class="nextstepaction"]
> [Configuración de la recuperación ante desastres](azure-to-azure-quickstart.md)
