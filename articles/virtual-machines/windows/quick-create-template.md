---
title: 'Inicio rápido: Implementación de una plantilla de Resource Manager para crear una VM Windows'
description: En esta guía de inicio rápido, aprenderá a usar una plantilla de Resource Manager para crear una máquina virtual Windows.
author: cynthn
ms.service: virtual-machines-windows
ms.topic: quickstart
ms.workload: infrastructure
ms.date: 06/04/2020
ms.author: cynthn
ms.custom: subject-armqs
ms.openlocfilehash: ded59b2f517c5b109dfd00bde2fb73f8351bf821
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88649676"
---
# <a name="quickstart-create-a-windows-virtual-machine-using-an-arm-template"></a>Inicio rápido: Creación de una máquina virtual Windows con una plantilla de Resource Manager

En este inicio rápido se muestra como usar una plantilla de Azure Resource Manager para implementar una máquina virtual Windows en Azure.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-vm-simple-windows%2fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-vm-simple-windows/).

:::code language="json" source="~/quickstart-templates/101-vm-simple-windows/azuredeploy.json":::


En la plantilla se definen varios recursos:

- [**Microsoft.Network/virtualNetworks/subnets**](/azure/templates/Microsoft.Network/virtualNetworks/subnets): cree una subred.
- [**Microsoft.Storage/storageAccounts**](/azure/templates/Microsoft.Storage/storageAccounts): cree una cuenta de almacenamiento.
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/Microsoft.Network/publicIPAddresses): cree una dirección IP pública.
- [**Microsoft.Network/networkSecurityGroups**](/azure/templates/Microsoft.Network/networkSecurityGroups): cree un grupo de seguridad de red.
- [**Microsoft.Network/virtualNetworks**](/azure/templates/Microsoft.Network/virtualNetworks): cree una red virtual.
- [**Microsoft.Network/networkInterfaces**](/azure/templates/Microsoft.Network/networkInterfaces): cree una NIC.
- [**Microsoft.Compute/virtualMachines**](/azure/templates/Microsoft.Compute/virtualMachines): cree una máquina virtual.



## <a name="deploy-the-template"></a>Implementación de la plantilla

1. Seleccione la imagen siguiente para iniciar sesión en Azure y abrir una plantilla. La plantilla crea un almacén de claves y un secreto.

    [![Implementación en Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-vm-simple-windows%2fazuredeploy.json)

1. Seleccione o escriba los siguientes valores. Utilice los valores predeterminados, si están disponibles.

    - **Suscripción**: seleccione una suscripción de Azure.
    - **Grupo de recursos**: seleccione un grupo de recursos existente del menú desplegable, o bien seleccione **Crear nuevo**, escriba un nombre único para el grupo de recursos y, a continuación, haga clic en **Aceptar**.
    - **Ubicación**: seleccione una ubicación.  Por ejemplo, **Centro de EE. UU**.
    - **Nombre de usuario de administrador**: proporcione un nombre de usuario, como *usuarioAzure*.
    - **Contraseña de administrador**: proporcione una contraseña para usarla en la cuenta de administrador. La contraseña debe tener al menos 12 caracteres de largo y cumplir con los [requisitos de complejidad definidos](faq.md#what-are-the-password-requirements-when-creating-a-vm).
    - **DNS label prefix** (Prefijo de etiqueta DNS): escriba un identificador único para usarlo como parte de la etiqueta DNS.
    - **Windows OS version** (Versión del sistema operativo Windows): seleccione la versión de Windows que quiere ejecutar en la VM.
    - **Tamaño de VM**: seleccione el [tamaño](../sizes.md) que se usará para la VM.
    - **Ubicación**: el valor predeterminado es la misma ubicación que el grupo de recursos, si ya existe.
1. Seleccione **Revisar + crear**. Una vez finalizada la validación, seleccione **Crear** para crear e implementar la VM.


Azure Portal se usa para implementar la plantilla. Además de Azure Portal, también puede usar Azure PowerShell, la CLI de Azure y API REST. Para obtener información sobre otros métodos de implementación, consulte [Implementación de plantillas](../../azure-resource-manager/templates/deploy-powershell.md).

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

Puede usar Azure Portal para comprobar la VM y otros recursos que se crearon. Una vez finalizada la implementación, seleccione **Ir al grupo de recursos** para ver la VM y otros recursos.


## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, elimine el grupo de recursos, lo que eliminará la VM y todos los recursos del grupo de recursos. 

1. Seleccione el **Grupo de recursos**.
1. En la página del grupo de recursos, seleccione **Eliminar**.
1. Cuando se le solicite, escriba el nombre del grupo de recursos y, después, seleccione **Eliminar**.


## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha implementado una máquina virtual simple mediante una plantilla de Resource Manager. Para más información acerca de las máquinas virtuales de Azure, continúe con el tutorial de máquinas virtuales Linux.


> [!div class="nextstepaction"]
> [Tutoriales de máquinas virtuales Windows de Azure](./tutorial-manage-vm.md)