---
title: 'Inicio rápido: Creación de una instancia de Azure Firewall y grupos de IP: plantilla de Resource Manager'
description: En este inicio rápido, aprenderá a usar las plantillas de Azure Resource Manager para crear una instancia de Azure Firewall y grupos de IP.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 08/28/2020
ms.author: victorh
ms.openlocfilehash: 7966cbe44fc8984cf3c3d9e2d08a11bf599ff158
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89079139"
---
# <a name="quickstart-create-an-azure-firewall-and-ip-groups---arm-template"></a>Inicio rápido: Creación de una instancia de Azure Firewall y grupos de IP: plantilla de Resource Manager

En este inicio rápido, se va a usar una plantilla de Azure Resource Manager para implementar una instancia de Azure Firewall con grupos de IP de ejemplo usados en una regla de red y una regla de aplicación. Un grupo de IP es un recurso de nivel superior que le permite definir y agrupar direcciones IP, intervalos y subredes en un único objeto. Resulta útil para administrar direcciones IP en reglas de Azure Firewall. Puede especificar manualmente las direcciones IP o importarlas desde un archivo.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-azurefirewall-create-with-ipgroups-and-linux-jumpbox%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="review-the-template"></a>Revisión de la plantilla

Esta plantilla crea una instancia de Azure Firewall y grupos de IP, junto con los recursos necesarios para admitir Azure Firewall.

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-azurefirewall-create-with-ipgroups-and-linux-jumpbox).

:::code language="json" source="~/quickstart-templates/101-azurefirewall-create-with-ipgroups-and-linux-jumpbox/azuredeploy.json":::

En la plantilla se definen varios recursos de Azure:

- [**Microsoft.Network/ipGroups**](/azure/templates/microsoft.network/ipGroups)
- [**Microsoft.Storage/storageAccounts**](/azure/templates/microsoft.storage/storageAccounts)
- [**Microsoft.Network/routeTables**](/azure/templates/microsoft.network/routeTables)
- [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.network/networksecuritygroups)
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks)
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses)
- [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces)
- [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines):
- [**Microsoft.Network/azureFirewalls**](/azure/templates/microsoft.network/azureFirewalls)

## <a name="deploy-the-template"></a>Implementación de la plantilla

Implementación de la plantilla de Resource Manager en Azure:

1. Seleccione **Implementar en Azure** para iniciar sesión en Azure y abrir la plantilla. La plantilla crea una instancia de Azure Firewall, la infraestructura de red y dos máquinas virtuales.

   [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-azurefirewall-create-with-ipgroups-and-linux-jumpbox%2Fazuredeploy.json)

2. En el portal, en la página **Create an Azure Firewall with IpGroups** (Crear una instancia de Azure Firewall con grupos de IP), escriba o seleccione los valores siguientes:
   - Suscripción: seleccione una de las suscripciones existentes. 
   - Grupo de recursos:  seleccione uno de los grupos de recursos existentes o elija **Crear nuevo** y, luego, **Aceptar**.
   - Ubicación: Seleccionar una ubicación
   - Nombre de la red virtual: escriba un nombre para la nueva red virtual. 
   - Nombre del grupo de IP 1: escriba el nombre del grupo de IP 1. 
   - Nombre del grupo de IP 2: escriba el nombre del grupo de IP 2. 
   - Nombre de usuario de administrador: escriba un nombre de usuario para la cuenta de usuario de administrador. 
   - Autenticación: seleccione sshPublicKey o contraseña. 
   - Contraseña de administrador: escriba una contraseña o clave de administrador.

3. Seleccione **Acepto los términos y condiciones indicadas anteriormente** y, después, seleccione **Comprar**. La implementación puede tardar 10 minutos o más en completarse.

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

En Azure Portal, revise los recursos implementados, especialmente las reglas de firewall que usan grupos de IP.

:::image type="content" source="media/quick-create-ipgroup-template/ipgroups.png" alt-text="Grupos de IP.":::

:::image type="content" source="media/quick-create-ipgroup-template/network-rule.png" alt-text="Reglas de red."::::

Para información sobre la sintaxis y las propiedades de JSON de un firewall en una plantilla, consulte la [referencia de plantilla de Microsoft.Network/azureFirewalls](/azure/templates/Microsoft.Network/2019-11-01/azureFirewalls).

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos que ha creado con el firewall, elimine el grupo de recursos. Esta acción quita el firewall y todos los recursos relacionados.

Para eliminar el grupo de recursos, llame al cmdlet `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name "<your resource group name>"
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Implementación y configuración de Azure Firewall en una red híbrida con Azure Portal](tutorial-hybrid-portal.md)