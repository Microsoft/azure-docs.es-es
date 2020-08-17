---
title: Creación de una puerta de enlace NAT - plantilla de Resource Manager
titleSuffix: Azure Virtual Network NAT
description: En este inicio rápido se muestra cómo crear una puerta de enlace NAT mediante la plantilla de Azure Resource Manager.
services: load-balancer
documentationcenter: na
author: asudbring
manager: KumudD
Customer intent: I want to create a NAT gateway by using an Azure Resource Manager template so that I can provide outbound connectivity for my virtual machines.
ms.service: virtual-network
ms.subservice: nat
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/09/2020
ms.author: allensu
ms.custom: subject-armqs
ms.openlocfilehash: fc4804070e0fa4ca6e9e54dcf6e04aafcc17f91a
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88053905"
---
# <a name="create-a-nat-gateway---resource-manager-template"></a>Creación de una puerta de enlace NAT - plantilla de Resource Manager

Introducción a Virtual Network NAT mediante una plantilla de Azure Resource Manager.  Esta plantilla implementa una red virtual, un recurso de puerta de enlace NAT y una máquina virtual con Ubuntu. La máquina virtual con Ubuntu se implementa en una subred que está asociada al recurso de puerta de enlace NAT.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="create-a-nat-gateway-and-supporting-resources"></a>Creación de una puerta de enlace NAT y recursos auxiliares

Esta plantilla está configurada para crear lo siguiente: 

* Virtual network 
* Recurso de puerta de enlace NAT
* Máquina virtual con Ubuntu

La máquina virtual con Ubuntu se implementa en una subred que está asociada al recurso de puerta de enlace NAT.

### <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-nat-gateway-1-vm/azuredeploy.json).

:::code language="json" source="~/quickstart-templates/101-nat-gateway-1-vm/azuredeploy.json" range="1-335" highlight="256-282":::

En la plantilla se definen nueve recursos de Azure:

**Microsoft.Network**

* **[Microsoft.Network/natGateways](https://docs.microsoft.com/azure/templates/microsoft.network/natgateways)** : Crea un recurso de puerta de enlace NAT.

* **[Microsoft.Network/networkSecurityGroups](https://docs.microsoft.com/azure/templates/microsoft.network/networksecuritygroups)** : Crea un grupo de seguridad de red.

    * **[Microsoft.Network/networkSecurityGroups/securityRules](https://docs.microsoft.com/azure/templates/microsoft.network/networksecuritygroups/securityrules)** : Crea una regla de seguridad.

* **[Microsoft.Network/publicIPAddresses](https://docs.microsoft.com/azure/templates/microsoft.network/publicipaddresses)** : Crea una dirección IP pública.

* **[Microsoft.Network/publicIPPrefixes](https://docs.microsoft.com/azure/templates/microsoft.network/publicipprefixes)** : Crea un prefijo de dirección IP pública.

* **[Microsoft.Network/virtualNetworks](https://docs.microsoft.com/azure/templates/microsoft.network/virtualnetworks)** : Crea una red virtual.

    * **[Microsoft.Network/virtualNetworks/subnets](https://docs.microsoft.com/azure/templates/microsoft.network/virtualnetworks/subnets)** : Crea una subred de red virtual.

* **[Microsoft.Network/networkinterfaces](https://docs.microsoft.com/azure/templates/microsoft.network/networkinterfaces)** : Crea una interfaz de red.

**Microsoft.Compute**

* **[Microsoft.Compute/virtualMachines](https://docs.microsoft.com/azure/templates/Microsoft.Compute/virtualMachines)** : Crea una máquina virtual.

### <a name="deploy-the-template"></a>Implementación de la plantilla

**CLI de Azure**

```azurecli-interactive
read -p "Enter the location (i.e. westcentralus): " location
resourceGroupName="myResourceGroupNAT"
templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-nat-gateway-1-vm/azuredeploy.json" 

az group create \
--name $resourceGroupName \
--location $location

az group deployment create \
--resource-group $resourceGroupName \
--template-uri  $templateUri
```

**Azure PowerShell**

```azurepowershell-interactive
$location = Read-Host -Prompt "Enter the location (i.e. westcentralus)"
$templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-nat-gateway-1-vm/azuredeploy.json"

$resourceGroupName = "myResourceGroupNAT"

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri
```

**Azure Portal**

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-nat-gateway-1-vm%2Fazuredeploy.json)

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Seleccione **Grupos de recursos** en el panel izquierdo.

3. Seleccione el grupo de recursos que creó en la sección anterior. El nombre predeterminado del grupo de recursos es **myResourceGroupNAT**.

4. Compruebe que los recursos siguientes se han creado en el grupo de recursos:

    ![Grupo de recursos de Virtual Network NAT](./media/quick-create-template/nat-gateway-template-rg.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

**CLI de Azure**

Cuando ya no lo necesite, puede usar el comando [az group delete](/cli/azure/group#az-group-delete) para quitar el grupo de recursos y todos los recursos que contiene.

```azurecli-interactive 
  az group delete \
    --name myResourceGroupNAT
```

**Azure PowerShell**

Cuando ya no lo necesite, puede usar el comando [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup?view=latest) para quitar el grupo de recursos y todos los recursos que contiene.

```azurepowershell-interactive 
Remove-AzResourceGroup -Name myResourceGroupNAT
```

**Azure Portal**

Cuando ya no los necesite, elimine el grupo de recursos, la puerta de enlace de NAT y todos los recursos relacionados. Seleccione el grupo de recursos **myResourceGroupNAT** que contiene la puerta de enlace de NAT y, luego, seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado lo siguiente:

* Recurso de puerta de enlace NAT
* Virtual network
* Máquina virtual con Ubuntu

La máquina virtual se implementa en una subred de red virtual asociada a la puerta de enlace NAT. 

Para más información sobre Virtual Network NAT y Azure Resource Manager, continúe con los artículos siguientes.

* Lea [Información general de Virtual Network NAT](nat-overview.md).
* Consulte los [recursos de puerta de enlace NAT](nat-gateway-resource.md).
* Obtenga más información sobre [Azure Resource Manager](../azure-resource-manager/management/overview.md).
