---
title: 'Inicio rápido: Dirección del tráfico web mediante una plantilla de Resource Manager'
titleSuffix: Azure Application Gateway
description: En esta guía de inicio rápido, obtendrá información sobre cómo utilizar una plantilla de Resource Manager para crear una instancia de Azure Application Gateway que dirija el tráfico web a las máquinas virtuales de un grupo de back-end.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: quickstart
ms.date: 08/27/2020
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: f492521469281615d4182294feaefffff688bd21
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88961693"
---
# <a name="quickstart-direct-web-traffic-with-azure-application-gateway---arm-template"></a>Inicio rápido: Dirección del tráfico web con Azure Application Gateway: plantilla de ARM

En este inicio rápido, va a usar una plantilla de Azure Resource Manager (plantilla de ARM) para crear una instancia de Azure Application Gateway. A continuación, pruebe la puerta de enlace de aplicaciones para asegurarse de que funciona correctamente.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

También se puede completar este inicio rápido mediante [Azure Portal](quick-create-portal.md), [Azure PowerShell](quick-create-powershell.md) o la [CLI de Azure](quick-create-cli.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fag-docs-qs%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="review-the-template"></a>Revisión de la plantilla

Para simplificar, en esta plantilla se crea una configuración sencilla con una dirección IP de front-end pública, un cliente de escucha básica que hospeda un único sitio en la puerta de enlace de aplicaciones, una regla de enrutamiento de solicitudes básica y dos máquinas virtuales que se usan con el grupo de back-end.

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/ag-docs-qs/).

:::code language="json" source="~/quickstart-templates/ag-docs-qs/azuredeploy.json":::

En la plantilla se definen varios recursos de Azure:

- [**Microsoft.Network/applicationgateways**](/azure/templates/microsoft.network/applicationgateways)
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses): uno para la puerta de enlace de aplicaciones y dos para las máquina virtuales.
- [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.network/networksecuritygroups)
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks)
- [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines): dos máquinas virtuales
- [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces): dos para las máquinas virtuales
- [**Microsoft.Compute/virtualMachine/extensions**](/azure/templates/microsoft.compute/virtualmachines/extensions): para configurar las páginas web e IIS

## <a name="deploy-the-template"></a>Implementación de la plantilla

Implementación de la plantilla de Resource Manager en Azure:

1. Seleccione **Implementar en Azure** para iniciar sesión en Azure y abrir la plantilla. La plantilla crea una puerta de enlace de aplicaciones, la infraestructura de red y dos máquinas virtuales en el grupo de back-end que ejecuta IIS.

   [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fag-docs-qs%2Fazuredeploy.json)

2. Seleccione o cree el grupo de recursos, y escriba el nombre de usuario y la contraseña del administrador de la máquina virtual.
3. Seleccione **Revisar y crear** y, luego, **Crear**.

   La implementación puede tardar 20 minutos o más en completarse.

## <a name="validate-the-deployment"></a>Validación de la implementación

Aunque no es necesario instalar IIS para crear la puerta de enlace de aplicaciones, se instaló para comprobar si Azure la creó correctamente. Use IIS para probar la puerta de enlace de aplicaciones:

1. Busque la dirección IP pública de la puerta de enlace de aplicaciones en la página de **información general**.![ Registre la dirección IP pública de la puerta de enlace de aplicaciones](./media/application-gateway-create-gateway-portal/application-gateway-record-ag-address.png). O bien, puede seleccionar **Todos los recursos**, escribir *myAGPublicIPAddress* en el cuadro de búsqueda y luego seleccionarlo en los resultados de la búsqueda. Azure muestra la dirección IP pública en la página de **información general**.
2. Copie la dirección IP pública y, luego, péguela en la barra de direcciones del explorador para ir a esa dirección IP.
3. Compruebe la respuesta. Una respuesta válida corrobora que la puerta de enlace de aplicaciones se ha creado correctamente y puede conectarse correctamente con el back-end.

   ![Prueba de la puerta de enlace de aplicaciones](./media/application-gateway-create-gateway-portal/application-gateway-iistest.png)

   Actualice el explorador varias veces y debería ver las conexiones a myVM1 y a myVM2.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos que ha creado con la puerta de enlace de aplicaciones, elimine el grupo de recursos. De esta forma, se elimina la puerta de enlace de aplicaciones y todos los recursos relacionados.

Para eliminar el grupo de recursos, llame al cmdlet `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Administrar el tráfico web con Application Gateway mediante la CLI de Azure](./tutorial-manage-web-traffic-cli.md)
