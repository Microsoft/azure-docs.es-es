---
title: 'Inicio rápido: Creación de una instancia de Azure Firewall con varias direcciones IP públicas: plantilla de Resource Manager'
description: En este inicio rápido, aprenderá a usar una plantilla de Azure Resource Manager para crear una instancia de Azure Firewall con varias direcciones IP públicas.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 08/28/2020
ms.author: victorh
ms.openlocfilehash: 282ce16c926c61008d0ebd70106b161425a15c9e
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89079088"
---
# <a name="quickstart-create-an-azure-firewall-with-multiple-public-ip-addresses---arm-template"></a>Inicio rápido: Creación de una instancia de Azure Firewall con varias direcciones IP públicas: plantilla de Resource Manager

En este inicio rápido, usará una plantilla de Azure Resource Manager para implementar una instancia de Azure Firewall con varias direcciones IP públicas. El firewall implementado tiene reglas de recopilación de reglas NAT que permiten conexiones RDP a dos máquinas virtuales Windows Server 2019.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Para más información acerca de Azure Firewall con varias direcciones IP públicas, consulte [Implementación de una instancia de Azure Firewall con varias direcciones IP públicas mediante Azure PowerShell](deploy-multi-public-ip-powershell.md).

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Ffw-docs-qs%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="review-the-template"></a>Revisión de la plantilla

Esta plantilla crea una instancia de Azure Firewall con dos grupos de direcciones IP públicas, junto con los recursos necesarios para admitir Azure Firewall.

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/fw-docs-qs).

:::code language="json" source="~/quickstart-templates/fw-docs-qs/azuredeploy.json":::

En la plantilla se definen varios recursos de Azure:

- [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.network/networksecuritygroups)
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses)
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks)
- [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines):
- [**Microsoft.Storage/storageAccounts**](/azure/templates/microsoft.storage/storageAccounts)
- [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces)
- [**Microsoft.Network/azureFirewalls**](/azure/templates/microsoft.network/azureFirewalls)
- [**Microsoft.Network/routeTables**](/azure/templates/microsoft.network/routeTables)

## <a name="deploy-the-template"></a>Implementación de la plantilla

Implementación de la plantilla de Resource Manager en Azure:

1. Seleccione **Implementar en Azure** para iniciar sesión en Azure y abrir la plantilla. La plantilla crea una instancia de Azure Firewall, la infraestructura de red y dos máquinas virtuales.

   [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Ffw-docs-qs%2Fazuredeploy.json)

2. En el portal, en la página **Create an Azure Firewall with multiple IP public addresses** (Crear una instancia de Azure Firewall con varias direcciones públicas), escriba o seleccione los valores siguientes:
   - Suscripción: seleccione una de las suscripciones existentes. 
   - Grupo de recursos:  seleccione uno de los grupos de recursos existentes o elija **Crear nuevo** y, luego, **Aceptar**.
   - Ubicación: Seleccionar una ubicación
   - Nombre de usuario de administrador: escriba un nombre de usuario para la cuenta de usuario de administrador. 
   - Contraseña de administrador: escriba una contraseña o clave de administrador.

3. Seleccione **Acepto los términos y condiciones indicadas anteriormente** y, después, seleccione **Comprar**. La implementación puede tardar 10 minutos o más en completarse.

## <a name="validate-the-deployment"></a>Validación de la implementación

En Azure Portal, revise los recursos implementados. Anote las direcciones IP públicas del firewall.  

Use la conexión a Escritorio remoto para conectarse a las direcciones IP públicas del firewall. Las conexiones correctas muestran las reglas NAT de firewall que permiten la conexión con los servidores back-end.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos que ha creado con el firewall, elimine el grupo de recursos. Esta acción quita el firewall y todos los recursos relacionados.

Para eliminar el grupo de recursos, llame al cmdlet `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name "<your resource group name>"
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Implementación y configuración de Azure Firewall en una red híbrida con Azure Portal](tutorial-hybrid-portal.md)