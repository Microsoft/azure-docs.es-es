---
title: 'Inicio rápido: Protección del centro de conectividad virtual mediante Azure Firewall Manager (plantilla de Resource Manager)'
description: En este inicio rápido, aprenderá a proteger un centro virtual mediante Azure Firewall Manager.
services: firewall-manager
author: vhorne
ms.service: firewall-manager
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 08/28/2020
ms.author: victorh
ms.openlocfilehash: 050dfa737ad04127f8c72158fbecb2e8e70e05c1
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89075348"
---
# <a name="quickstart-secure-your-virtual-hub-using-azure-firewall-manager---arm-template"></a>Inicio rápido: Protección de un centro virtual mediante Azure Firewall Manager: plantilla de Resource Manager

En este inicio rápido, usará una plantilla de Azure Resource Manager para proteger el centro virtual mediante Azure Firewall Manager. El firewall implementado tiene una regla de aplicación que permite conexiones a `www.microsoft.com`. Para probar el firewall, se implementan dos máquinas virtuales con Windows Server 2019. Para realizar la conexión al servidor de cargas de trabajo, se usa un servidor de saltos. Desde el servidor de cargas de trabajo, solo puede conectarse a `www.microsoft.com`.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Para más información sobre Azure Firewall Manager, consulte [¿Qué es Azure Firewall Manager?](overview.md)

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Ffwm-docs-qs%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="review-the-template"></a>Revisión de la plantilla

Esta plantilla crea un centro virtual protegido mediante Azure Firewall Manager, junto con los recursos necesarios para admitir el escenario.

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/fwm-docs-qs/).

:::code language="json" source="~/quickstart-templates/fwm-docs-qs/azuredeploy.json":::

En la plantilla se definen varios recursos de Azure:

- [**Microsoft.Network/virtualWans**](/azure/templates/microsoft.network/virtualWans)
- [**Microsoft.Network/virtualHubs**](/azure/templates/microsoft.network/virtualHubs)
- [**Microsoft.Network/firewallPolicies**](/azure/templates/microsoft.network/firewallPolicies)
- [**Microsoft.Network/azureFirewalls**](/azure/templates/microsoft.network/azureFirewalls)
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks)
- [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines):
- [**Microsoft.Storage/storageAccounts**](/azure/templates/microsoft.storage/storageAccounts)
- [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces)
- [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.network/networksecuritygroups)
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses)
- [**Microsoft.Network/routeTables**](/azure/templates/microsoft.network/routeTables)

## <a name="deploy-the-template"></a>Implementación de la plantilla

Implementación de la plantilla de Resource Manager en Azure:

1. Seleccione **Implementar en Azure** para iniciar sesión en Azure y abrir la plantilla. La plantilla crea una instancia de Azure Firewall, una WAN virtual y un centro virtual, la infraestructura de red y dos máquinas virtuales.

   [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Ffwm-docs-qs%2Fazuredeploy.json)

2. En el portal, en la página **Centros virtuales protegidos**, escriba o seleccione los valores siguientes:
   - Suscripción: seleccione una de las suscripciones existentes. 
   - Grupo de recursos:  seleccione uno de los grupos de recursos existentes o elija **Crear nuevo** y, luego, **Aceptar**.
   - Ubicación: Seleccionar una ubicación
   - Nombre de usuario de administrador: escriba un nombre de usuario para la cuenta de usuario de administrador. 
   - Contraseña de administrador: escriba una contraseña o clave de administrador.

3. Seleccione **Revisar y crear** y, luego, **Crear**. La implementación puede tardar 10 minutos o más en completarse.

## <a name="validate-the-deployment"></a>Validación de la implementación

Ahora, pruebe las reglas de firewall para confirmar que funcionan según lo previsto.

1. En Azure Portal, revise la configuración de red de la máquina virtual **Workload-Srv** y anote la dirección IP privada.
2. Conecte un escritorio remoto a la máquina virtual **Jump-Srv** e inicie sesión. Desde ahí, abra una conexión del escritorio remoto a la dirección IP privada de **Workload-Srv**.

3. Abra Internet Explorer y vaya a `www.microsoft.com`.
4. Seleccione **Aceptar** > **Cerrar** en las alertas de seguridad de Internet Explorer.

   Debería ver la página principal de Microsoft.

5. Vaya a `www.google.com`.

   El firewall debería bloquearle.

Con ello, ha comprobado que las reglas de firewall funcionan:

- Puede navegar al FQDN permitido pero no a ningún otro.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos que ha creado con el firewall, elimine el grupo de recursos. Esta acción quita el firewall y todos los recursos relacionados.

Para eliminar el grupo de recursos, llame al cmdlet `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name "<your resource group name>"
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre los proveedores de seguridad asociados](trusted-security-partners.md)
