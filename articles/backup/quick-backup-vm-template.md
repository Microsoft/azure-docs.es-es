---
title: 'Inicio rápido: Copia de seguridad de máquina virtual con una plantilla de Resource Manager'
description: Aprenda a realizar una copia de seguridad de las máquinas virtuales con la plantilla de Azure Resource Manager.
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 05/14/2019
ms.custom: mvc,subject-armqs
ms.openlocfilehash: e64aa10d8ddadd367d04b2b480770a99f3ece1dc
ms.sourcegitcommit: ac7ae29773faaa6b1f7836868565517cd48561b2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88826538"
---
# <a name="quickstart-back-up-a-virtual-machine-in-azure-with-an-arm-template"></a>Inicio rápido: Copia de seguridad de una máquina virtual en Azure con una plantilla de Resource Manager

[Azure Backup](backup-overview.md) puede hacer copias de seguridad de máquinas locales, aplicaciones y máquinas virtuales de Azure. En este artículo se muestra cómo realizar una copia de seguridad de una máquina virtual de Azure con una plantilla de Azure Resource Manager y Azure PowerShell. Este inicio rápido se centra en el proceso de implementación de una plantilla de ARM para crear un almacén de Recovery Services. Para más información sobre el desarrollo de plantillas de Resource Manager, consulte la [documentación de Azure Resource Manager](../azure-resource-manager/index.yml) y la [referencia de plantilla](/azure/templates/microsoft.recoveryservices/allversions).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Un [almacén de Recovery Services](backup-azure-recovery-services-vault-overview.md) es un contenedor lógico que almacena los datos de copia de seguridad de los recursos protegidos, como las máquinas virtuales de Azure. Cuando se ejecuta un trabajo de copia de seguridad, se crea un punto de recuperación en el almacén de Recovery Services. Posteriormente, se puede usar uno de estos puntos de recuperación para restaurar los datos a un momento dado en el tiempo. Como alternativa, puede realizar una copia de seguridad de una máquina virtual mediante [Azure PowerShell](./quick-backup-vm-powershell.md), la [CLI de Azure](quick-backup-vm-cli.md) o [Azure Portal](quick-backup-vm-portal.md).

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-recovery-services-create-vm-and-configure-backup%2Fazuredeploy.json)

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-recovery-services-create-vm-and-configure-backup/). Esta plantilla le permite implementar una máquina virtual de Windows y un almacén de Recovery Services sencillos configurados con la directiva de protección predeterminada.

:::code language="json" source="~/quickstart-templates/101-recovery-services-create-vm-and-configure-backup/azuredeploy.json":::

Los recursos definidos en la plantilla son los siguientes:

- [**Microsoft.Storage/storageAccounts**](/azure/templates/microsoft.storage/storageaccounts)
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses)
- [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.network/networksecuritygroups)
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks)
- [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces)
- [**Microsoft.Compute/virutalMachines**](/azure/templates/microsoft.compute/virtualmachines)
- [**Microsoft.RecoveryServices/vaults**](/azure/templates/microsoft.recoveryservices/2016-06-01/vaults)
- [**Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectedItems**](/azure/templates/microsoft.recoveryservices/vaults/backupfabrics/protectioncontainers/protecteditems)

## <a name="deploy-the-template"></a>Implementación de la plantilla

Para implementar la plantilla, seleccione **Probar** para abrir Azure Cloud Shell y pegue el siguiente script de PowerShell en la ventana del shell. Para pegar el código, haga clic con el botón derecho en la ventana del shell y seleccione **Pegar**.

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter a project name (limited to eight characters) that is used to generate Azure resource names"
$location = Read-Host -Prompt "Enter the location (for example, centralus)"
$adminUsername = Read-Host -Prompt "Enter the administrator username for the virtual machine"
$adminPassword = Read-Host -Prompt "Enter the administrator password for the virtual machine" -AsSecureString
$dnsPrefix = Read-Host -Prompt "Enter the unique DNS Name for the Public IP used to access the virtual machine"

$resourceGroupName = "${projectName}rg"
$templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-recovery-services-create-vm-and-configure-backup/azuredeploy.json"

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri -projectName $projectName -adminUsername $adminUsername -adminPassword $adminPassword -dnsLabelPrefix $dnsPrefix
```

En este inicio rápido, se usa Azure PowerShell para implementar la plantilla de Resource Manager. También se pueden usar para tal fin [Azure Portal](../azure-resource-manager/templates/deploy-portal.md), la [CLI de Azure](../azure-resource-manager/templates/deploy-cli.md) y la [API REST](../azure-resource-manager/templates/deploy-rest.md).

## <a name="validate-the-deployment"></a>Validación de la implementación

### <a name="start-a-backup-job"></a>Inicio de un trabajo de copia de seguridad

La plantilla crea una máquina virtual y le permite crear una copia de seguridad de la máquina virtual. Después de implementar la plantilla, deberá iniciar un trabajo de copia de seguridad. Para más información, consulte [Inicio de un trabajo de copia de seguridad](./quick-backup-vm-powershell.md#start-a-backup-job).

### <a name="monitor-the-backup-job"></a>Supervisión del trabajo de copia de seguridad

Para supervisar el trabajo de copia de seguridad, consulte [Supervisión del trabajo de copia de seguridad](./quick-backup-vm-powershell.md#monitor-the-backup-job).

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya no necesita realizar copias de seguridad de la máquina virtual, puede limpiarla.

- Para intentar restaurar la VM, omita el paso de limpieza.
- Si ha usado una máquina virtual existente, puede omitir el último cmdlet [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para dejar tanto el grupo de recursos como la máquina virtual en su lugar.

Deshabilite la protección, elimine los puntos de restauración y el almacén. A continuación, elimine el grupo de recursos y los recursos de máquina virtual asociados como se indica a continuación:

```powershell
Disable-AzRecoveryServicesBackupProtection -Item $item -RemoveRecoveryPoints
$vault = Get-AzRecoveryServicesVault -Name "myRecoveryServicesVault"
Remove-AzRecoveryServicesVault -Vault $vault
Remove-AzResourceGroup -Name "myResourceGroup"
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha creado un almacén de Recovery Services, ha habilitado la protección en una máquina virtual y ha creado el punto de recuperación inicial.

- [Aprenda](tutorial-backup-vm-at-scale.md) a realizar una copia de seguridad de máquinas virtuales en Azure Portal.
- [Aprenda a](tutorial-restore-disk.md) restaurar rápidamente una máquina virtual.
- [Aprenda a](../azure-resource-manager/templates/template-tutorial-create-first-template.md) crear plantillas de Resource Manager.
